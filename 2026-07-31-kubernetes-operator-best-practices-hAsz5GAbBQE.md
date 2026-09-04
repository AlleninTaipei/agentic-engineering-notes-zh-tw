# Kubernetes Operator Best Practices: Resource Version, Retry, Concurrency 與 Predicate

- 影片: [Kubernetes Operator Best Practices - Kubebuilder Deep Dive](https://www.youtube.com/watch?v=hAsz5GAbBQE)
- 頻道: freeCodeCamp.org
- 課程來源: Kubesimplify
- 發布日期: 2026-07-31
- 片長: 02:20:30
- Video ID: `hAsz5GAbBQE`
- 內容依據: YouTube 英文原語自動字幕 (`en-orig`)

## 摘要

這門課以 Kubebuilder 與 controller-runtime 為背景, 集中處理 Kubernetes operator 在 production 中常見的三類問題: 多個 actor 同時更新同一 resource 所造成的 conflict, reconcile 工作序列化造成的 throughput 限制, 以及 controller 自己更新 status 後再次觸發 reconcile 的循環.

三者背後其實是同一個設計問題: controller 必須在事件驅動, eventual consistency 與 concurrent updates 的環境中維持 idempotency. Operator 不能假設讀到的 object 永遠是最新版, 不能假設只有自己會寫入, 也不能把每個 object event 都當成需要執行完整 workflow 的業務變更.

課程透過 custom resource 與模擬 EC2 provisioning 的 controller 示範問題, 再逐步加入 conflict-aware retry, concurrent reconcile workers 與 generation-based predicate filtering.

## ResourceVersion 與 Generation 解決不同問題

[00:00:00](https://www.youtube.com/watch?v=hAsz5GAbBQE&t=0s)

Kubernetes object 同時具有 `metadata.resourceVersion` 與 `metadata.generation`, 兩者不能互換.

| 欄位 | 代表意義 | 何時改變 | Controller 的典型用途 |
| --- | --- | --- | --- |
| `resourceVersion` | API server 儲存中 object 的特定版本 | Object 發生任何持久化更新時 | Optimistic concurrency 與 watch |
| `generation` | Desired state 的世代 | 通常在 `spec` 改變時 | 判斷使用者意圖是否改變 |

課程用實際修改 object 的方式說明差異. 更新 `spec` 時, `generation` 與 `resourceVersion` 都會改變. 若只是更新 label, annotation 或 status, `resourceVersion` 仍會改變, 但 `generation` 通常不變.

因此:

- `resourceVersion` 回答的是「我正要寫回的 object 是否仍是剛才讀到的版本?」
- `generation` 回答的是「需要 reconcile 的 desired state 是否出現新版本?」

Controller 常把已處理的 generation 寫入 `status.observedGeneration`. 使用者與其他 controller 就能分辨 status 是否對應目前的 spec, 而不是只看到一個可能已過期的 `Ready` condition.

## Conflict 是保護資料, 不是偶發雜訊

[00:03:48](https://www.youtube.com/watch?v=hAsz5GAbBQE&t=228s)

課程建立一個 node pool 與 EC2 provisioning 情境. Reconciler 讀取 custom resource 後, 可能花一段時間呼叫外部 cloud API. 在它準備把 public IP 或進度寫回 status 前, 使用者或另一個 controller 已經修改同一 object.

事件順序可以簡化為:

```text
Controller A reads resourceVersion=1
  -> User or Controller B updates the object
  -> API server stores resourceVersion=2
  -> Controller A attempts Update with resourceVersion=1
  -> API server rejects the stale write with Conflict
```

如果 API server 接受 A 的舊副本, B 的更新就可能被覆蓋. Conflict error 是 Kubernetes optimistic concurrency control 的必要結果, 不是單純需要隱藏的錯誤.

這也說明為何長時間外部操作會擴大衝突窗口. Reconciler 在 `Get` 與 `Update` 之間做愈多工作, object 被別人修改的機會就愈高.

## Conflict Retry 必須重新讀取與重新套用意圖

[00:26:57](https://www.youtube.com/watch?v=hAsz5GAbBQE&t=1617s)

最粗略的處理方式是讓 `Reconcile` 回傳 error, 交給 controller-runtime 的 rate-limited queue 重新排程整個流程. 這能恢復工作, 但也可能重做昂貴或具有 side effect 的外部操作.

課程建議在 object update 周圍使用 Kubernetes client-go 提供的 `RetryOnConflict`. 核心模式不是反覆送出同一份 stale object, 而是每次重試都重新取得最新版, 再把目前 controller 負責的變更套用上去:

```go
err := retry.RetryOnConflict(retry.DefaultRetry, func() error {
    latest := &examplev1.EC2Instance{}

    if err := r.Get(ctx, req.NamespacedName, latest); err != nil {
        return err
    }

    latest.Status.PublicIP = publicIP
    return r.Status().Update(ctx, latest)
})
```

程式片段是依課程模式整理的簡化版本, 不是影片原始碼的逐字轉錄.

正確 retry 有三個關鍵:

1. 在 retry closure 裡重新 `Get`, 不沿用舊 object.
2. 重新套用 mutation intent, 不用舊 object 覆蓋最新狀態.
3. 只針對 conflict 做 bounded retry, 其他 error 依類型處理.

`RetryOnConflict` 的 backoff 可設定 duration, factor, jitter 與 steps. Jitter 能降低多個 controllers 同時衝突後又在相同時間重試的 synchronized retry storm. 重試次數耗盡仍可能失敗, 所以程式必須留下 log, metric 或告警, 不能把 retry 當成成功保證.

## Error 應按語意分類

課程區分幾種結果:

- `NotFound`: Object 已刪除時通常直接結束, 再重試也不會恢復它.
- `Conflict`: 重新取得最新版並嘗試合併自己的變更.
- Transient dependency error: 回傳 error 或明確 requeue, 讓 rate limiter 控制下一次執行.
- Persistent error: 在有限重試後暴露 failure signal, 避免無限安靜重跑.

Reconciler 的回傳值不是一般 function error handling 而已, 它同時控制 work queue. 任意回傳 error 可能使整段 reconcile 再執行一次. 因此呼叫外部 API, 建立 cloud resource 或付款等 side effect 必須具有 idempotency key 或可觀察的外部狀態.

## 多個 Workers 提升不同 Objects 的 Throughput

[01:11:48](https://www.youtube.com/watch?v=hAsz5GAbBQE&t=4308s)

單一 reconcile worker 一次只能處理一個 queue item. 若每個 object 都要等待緩慢的外部 API, 其他互不相關的 objects 也會排隊. Controller-runtime 可透過 controller options 設定最大並行 reconcile 數:

```go
func (r *EC2InstanceReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&examplev1.EC2Instance{}).
        WithOptions(controller.Options{
            MaxConcurrentReconciles: 10,
        }).
        Complete(r)
}
```

程式片段是依課程概念整理. 實際 import path 與 API 形式應依專案使用的 controller-runtime 版本確認.

增加 workers 適合讓不同 objects 並行處理. 它不代表同一 object 可以安全地由多個 goroutines 任意修改. Work queue 通常會避免同一 key 同時被多個 workers 處理, 但其他 controllers, users 與 external systems 仍可能產生 concurrency, 所以 conflict handling 依然必要.

Workers 數量不是愈多愈好. 應根據下列限制量測:

- API server client-side rate limits.
- External provider quotas 與 latency.
- Controller CPU, memory 與 goroutine 數.
- 平均 reconcile duration 與 queue depth.
- Downstream database 或 service capacity.
- Object 數量與事件到達率.

高 worker count 可能只把瓶頸推向 API server, 形成 throttling, 更多 conflict 或外部服務壓力. 課程示範 `10` 個 workers 是教學設定, 不是通用 production default.

## Status Update 可能製造自我觸發循環

[01:41:59](https://www.youtube.com/watch?v=hAsz5GAbBQE&t=6119s)

Controller watch 某種 resource 時, object 的任何更新都可能產生 update event. 若 reconcile 每次都無條件更新 status, status update 又可能把同一 key 放回 queue:

```text
Spec change triggers Reconcile
  -> Reconcile updates Status
  -> Status update triggers Reconcile
  -> Reconcile writes the same Status again
  -> repeat
```

這會浪費 worker, API server 與 etcd 資源. 若 reconcile 還包含外部 side effect, 風險不只是效能下降.

第一層防線是 idempotency 與 change detection. 在寫入前比較 desired status 與 current status, 沒有實質差異時不要送出 `Status().Update`.

第二層是 event filtering. 若 controller 只需要在 spec 改變時啟動, 可以使用 `GenerationChangedPredicate`:

```go
func (r *EC2InstanceReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(
            &examplev1.EC2Instance{},
            builder.WithPredicates(predicate.GenerationChangedPredicate{}),
        ).
        Complete(r)
}
```

它比較 update event 的 old 與 new generation. Status-only update 不會增加 generation, 因此可在進入 work queue 前被過濾.

## Predicate 不是萬用的 Infinite Loop 修補

`GenerationChangedPredicate` 只有在「controller 的正確性只依賴 spec change」時才合適. 如果 controller 也必須回應 annotation, label, deletion timestamp, child resource 或外部狀態變化, 單獨使用此 predicate 可能漏掉必要事件.

更完整的設計可能組合:

- Generation change.
- 特定 label 或 annotation change.
- Create 與 delete events.
- Owned resource changes.
- Periodic resync 或 explicit requeue.

Predicate 的角色是降低無關事件進入 queue 的比例, 不是取代 reconcile idempotency. 即使事件被過濾, controller 仍應容許 duplicate, delayed 與 out-of-order observations.

## Production Operator 的設計檢查表

以下為依課程內容整理的實務檢查表.

### State 與 Ownership

- 哪個 controller 擁有 `spec`, `status` 與個別 metadata fields?
- 是否只更新自己負責的 fields?
- Status 是否記錄 `observedGeneration`?
- 是否需要使用 status subresource, Patch 或 Server-Side Apply 降低衝突範圍?

### Retry 與 Side Effects

- Conflict retry 是否每次重新讀取最新版?
- Retry closure 是否只包含可安全重複的 mutation?
- External API operation 是否具 idempotency?
- Retry 是否 bounded, 有 backoff, jitter 與可觀測失敗?

### Concurrency 與 Capacity

- `MaxConcurrentReconciles` 是否由 workload measurements 決定?
- 是否監測 queue depth, reconcile latency, error rate 與 throttling?
- Controller 是否遵守 API server 與 external provider rate limit?
- Shared client, cache 或 in-memory state 是否 thread-safe?

### Event Filtering

- 哪些 events 真正代表 desired state 改變?
- Status-only update 是否需要再次 reconcile?
- Predicate 是否會忽略 deletion, metadata 或 owned resource changes?
- 即使 filter 失效, reconcile 是否仍然 idempotent?

## 核心結論

可靠的 Kubernetes operator 不只是把 API calls 放進 `Reconcile`. 它必須接受 object cache 可能過期, 多個 actors 可能同時寫入, events 可能重複, 外部操作可能比 object version 存活更久, 而提高 concurrency 會放大既有 race 與 rate-limit 問題.

這門課提供的實用順序是:

```text
Understand resourceVersion and generation
  -> Reproduce a stale-object conflict
  -> Retry with a fresh read and bounded backoff
  -> Classify errors by meaning
  -> Add workers based on measured demand
  -> Filter irrelevant events
  -> Keep Reconcile idempotent throughout
```

`RetryOnConflict`, `MaxConcurrentReconciles` 與 `GenerationChangedPredicate` 各自解決不同層次的問題. 把它們一起使用並不自動形成可靠 controller. Field ownership, external side-effect safety, observability 與容量控制仍需要明確設計.

## 來源與信心限制

- 本筆記依據 YouTube 英文自動字幕與影片章節整理, 不是逐字稿.
- 影片由 Kubesimplify 提供給 freeCodeCamp.org, 屬實作導向教學, 但筆記未取得或執行影片中的完整 source repository.
- 自動字幕多次把 Kubernetes, kubectl, kube-apiserver 與 Kubebuilder 名稱誤辨, 本文只在技術上下文足以確認時修正.
- 課程示範使用當時的 Go 與 controller-runtime API. Library signatures 可能隨版本改變, 實作前應核對目前專案版本的官方文件.
- 本文的簡化程式片段與 production checklist 屬編者整理, 用來呈現模式與風險, 不宣稱是影片原始碼.
- 影片透過 sleep 與手動更新模擬長任務和 conflict. 這能說明機制, 但不是 production load test 或 reliability evidence.
