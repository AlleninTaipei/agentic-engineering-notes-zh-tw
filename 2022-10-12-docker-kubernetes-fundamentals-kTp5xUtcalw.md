# Docker 與 Kubernetes 基礎實作: 從容器到工作負載編排

> [!info] 來源
> - 影片: [Docker Containers and Kubernetes Fundamentals - Full Hands-On Course](https://www.youtube.com/watch?v=kTp5xUtcalw)
> - 頻道: freeCodeCamp.org
> - 發布日期: 2022-10-12
> - 片長: 5:56:36
> - Video ID: `kTp5xUtcalw`
> - 內容依據: YouTube 英文原始語言自動字幕 (`en-orig`) 與影片章節
> - 筆記語言: 繁體中文

## 課程摘要

這是一門從零開始的容器與 Kubernetes 實作課。前半段說明 microservices、cloud native、Docker image、資料持久化、Docker Compose 與 registry; 後半段以本機 Kubernetes 叢集練習 namespaces、Pods、workloads、更新策略、Services、儲存、設定、健康檢查、儀表板與水平擴縮。

課程的核心思路是: 先把應用程式及其執行環境封裝成可攜式 image, 再以宣告式 manifest 描述希望叢集維持的狀態。Kubernetes 負責排程、修復、更新與連線, 但不取代建置流程、資料庫或應用層服務。

## 從單體系統走向 cloud native

### Microservices 的價值與代價

課程先比較單體與微服務架構 ([00:05:02](https://www.youtube.com/watch?v=kTp5xUtcalw&t=302s))。單體系統通常作為單一單元建置、部署與擴充; microservices 則把系統拆成責任較單一的服務, 透過 HTTP 或 gRPC 等輕量協定溝通。

| 面向 | 單體系統 | Microservices |
| --- | --- | --- |
| 部署 | 整套系統一起部署 | 各服務獨立部署 |
| 擴充 | 複製整套應用 | 只擴充有需求的服務 |
| 技術選擇 | 通常共享技術堆疊 | 服務可選不同語言與資料庫 |
| 故障範圍 | 容易影響整體 | 可隔離, 但分散式故障點變多 |
| 複雜度 | 程式內耦合 | 網路、部署、安全與資料一致性複雜 |

既有單體系統可使用 Strangler pattern 漸進拆分: 先在舊系統前建立 facade, 將特定功能逐步導向新服務, 等遷移完成後再移除舊實作與過渡層。

影片提醒, microservices 不是灑在既有系統上的「魔法粉末」。拆分後會增加 API latency、暫時性錯誤、跨服務測試、安全管理與 domino effect 等風險。團隊需要 CI/CD、重試策略、監控及明確指標, 並以小步驟驗證變更。

### Cloud native 是工作方式, 不只是部署位置

Cloud native 結合 containers、microservices、service mesh、immutable infrastructure 與 declarative APIs ([00:13:56](https://www.youtube.com/watch?v=kTp5xUtcalw&t=836s))。其目標是以自動化、小批量且頻繁的部署提高速度與敏捷性。

在 immutable infrastructure 的思維下, 團隊不直接修補長期存在的執行個體, 而是建造包含更新的新版本, 以新版本取代舊版本。課程引用 CNCF trail map, 建議依序建立容器化、CI/CD、orchestration、observability 與 service mesh 能力, 不要同時導入所有工具。

## Docker 基礎

### Container 與 image

Container 是部署單位, 包含程式、runtime、系統函式庫與工具 ([00:23:01](https://www.youtube.com/watch?v=kTp5xUtcalw&t=1381s))。相較於 VM 虛擬化硬體並各自啟動完整 OS, container 共用 host kernel, 因此通常啟動更快、體積更小。

Container image 由唯讀 layers 組成, 執行時最上層再加入可寫層。Docker 會快取已存在的 layers, 拉取新版本時只下載缺少的部分。Registry 則是集中存放及分發 images 的服務。

常見流程如下:

```bash
docker build -t my-app:v1 .
docker run -d -p 8080:80 --name my-app my-app:v1
docker ps
docker logs my-app
docker exec -it my-app sh
docker stop my-app
docker rm my-app
```

課程強調 container 是短暫且可替換的。重要資料不能只留在 container 的可寫層, 否則刪除 container 時資料也會消失。

### Dockerfile 的建置觀念

Dockerfile 描述如何建立 image。典型步驟是選擇 `FROM` base image, 設定 `WORKDIR`, 以 `COPY` 加入檔案, 用 `RUN` 安裝相依套件, 再以 `CMD` 或 `ENTRYPOINT` 指定啟動程序。

實務上應利用 layer cache, 先複製較少變動的 dependency manifest 並安裝相依套件, 最後才複製經常變動的原始碼。也可使用 multi-stage build, 在較大的 builder stage 編譯, 只把執行所需產物複製到精簡的 final image。

### 資料持久化

課程介紹 bind mount 與 Docker volume ([01:07:03](https://www.youtube.com/watch?v=kTp5xUtcalw&t=4023s)):

- Bind mount 將 host 的明確路徑掛入 container, 適合本機開發與直接編輯檔案。
- Volume 由 Docker 管理儲存位置, 與 container 生命週期分離, 較適合保留應用資料。

```bash
docker volume create app-data
docker run -v app-data:/var/lib/app my-app:v1
docker volume ls
```

刪除 container 不會自動刪除 volume。清理環境時要分別確認 containers、images 與 volumes, 避免誤以為 `docker rm` 已移除所有資料。

## 使用 Docker Compose 管理多容器應用

Docker Compose 用一份 YAML 描述多個 services、networks、volumes 與其他設定 ([01:17:03](https://www.youtube.com/watch?v=kTp5xUtcalw&t=4623s))。Compose v2 是 Docker CLI plugin, 命令形式為 `docker compose`, 不再是舊版的 `docker-compose`。

```bash
docker compose build
docker compose up -d
docker compose ps
docker compose logs -f backend
docker compose exec backend sh
docker compose down
```

同一份 Compose 檔內的 services 預設可用 service name 作為 hostname。例如 `backend` 可用 `db:5432` 存取名為 `db` 的 service。若要限制流量, 可將 frontend、backend 與 database 放入不同 networks。

重要設定包括:

- `depends_on`: 控制 container 啟動順序, 但不能取代完整的應用就緒檢查。
- `environment` 與 `.env`: 注入設定值。
- Named volumes: 在 services 間共享或保留資料。
- Resource requests/limits: 約束 CPU 與記憶體使用。
- Restart policy: 決定程序失敗或 host 重啟後是否重啟 container。
- Project name: 在同一目錄啟動多個隔離的 Compose 專案。

Compose 適合本機開發、測試及不需要完整 orchestrator 的小型工作負載。當系統需要跨節點排程、自動修復、進階 rollout 或彈性擴縮時, 才進一步使用 Kubernetes。

## Registry 與 image 發布

Container registry 是 image 的中央儲存庫 ([01:47:18](https://www.youtube.com/watch?v=kTp5xUtcalw&t=6438s))。發布到 Docker Hub 時, image 名稱通常包含帳號或組織名稱:

```bash
docker login
docker build -t my-org/my-app:v1 .
docker push my-org/my-app:v1
docker pull my-org/my-app:v1
```

Tags 用來區分版本, 但課程沒有把 tag 描述為不可變識別碼。部署流程若要求精確可重現性, 還需要規範 tag 更新方式或使用 image digest。

## Kubernetes 架構與操作模型

Kubernetes 是 vendor-neutral 的 container orchestrator, 提供 service discovery、load balancing、rollout、rollback、健康監控、設定與 secrets 管理 ([01:56:58](https://www.youtube.com/watch?v=kTp5xUtcalw&t=7018s))。它不負責編譯程式, 也不直接提供資料庫、cache 或 message bus 等應用層服務。

叢集分成 control plane 與 worker nodes:

- API server 是所有客戶端與叢集溝通的入口。
- `etcd` 保存叢集狀態, 作為控制平面的資料來源。
- Controller managers 持續比較 desired state 與 actual state。
- Scheduler 為尚未指派節點的 Pods 選擇 node。
- 每個 node 上的 kubelet 依指派結果啟動並監督 Pods。
- Container runtime 實際執行 containers。

`kubectl` 透過 kubeconfig 中的 context 連到 API server。Context 組合 cluster、user 與 namespace, 因此執行變更前應先確認目標:

```bash
kubectl config current-context
kubectl config get-contexts
kubectl config use-context docker-desktop
kubectl cluster-info
```

### Imperative 與 declarative

Imperative commands 適合學習、快速測試與故障排除; declarative manifests 則可重現、可審查並可納入 source control。

```bash
# Imperative
kubectl create deployment my-nginx --image=nginx

# Declarative
kubectl apply -f deployment.yaml
```

Kubernetes manifest 通常包含 `apiVersion`、`kind`、`metadata` 與 `spec`。可用官方文件、編輯器範本, 或 `--dry-run=client -o yaml` 產生起始骨架。

## Namespaces、Pods 與 selectors

Namespace 是叢集內的邏輯分組, 可用於 Dev、Test、Prod 等環境 ([02:20:38](https://www.youtube.com/watch?v=kTp5xUtcalw&t=8438s))。刪除 namespace 會連同其中資源一起刪除, 因此很適合建立短期實驗環境, 也必須謹慎操作。

```bash
kubectl get namespaces
kubectl create namespace demo
kubectl get pods -n demo
kubectl get pods --all-namespaces
kubectl delete namespace demo
```

Pod 是 Kubernetes 的原子工作負載, 包含一個或多個共享網路與 volumes 的 containers ([02:38:36](https://www.youtube.com/watch?v=kTp5xUtcalw&t=9516s))。Pod 本身不提供持續副本管理, 因此正式應用通常透過 Deployment 等較高階 workload 建立。

```bash
kubectl get pods -o wide
kubectl describe pod my-app
kubectl logs my-app
kubectl exec -it my-app -- sh
```

Labels 是自訂的 key-value metadata。Selectors 用 labels 找到目標資源, 例如 Service 選取 Pods, 或 node selector 限制 Pod 只能排到特定 nodes。若 labels 與 selector 不一致, Service 將沒有 endpoints。

### Init containers 與多容器 Pod

Init containers 在主要 app container 前依序執行且必須成功完成, 可用來等待相依服務、取得設定或準備檔案。它能把基礎設施準備邏輯移出主要應用程式。

同一 Pod 中的 containers 共享 IP 與 volumes, 可使用 `localhost` 加不同 ports 溝通。課程介紹三種 helper container patterns ([03:07:51](https://www.youtube.com/watch?v=kTp5xUtcalw&t=11271s)):

- Sidecar: 補充主要 container 的功能, 例如轉存 logs。
- Adapter: 將輸出轉換成外部系統可理解的格式。
- Ambassador: 代理主要程式與外部服務的連線。

## Workloads 如何管理 Pods

Workload 是在 Kubernetes 上執行應用的資源。不同 workload 對 Pod 生命週期提供不同保證 ([03:19:45](https://www.youtube.com/watch?v=kTp5xUtcalw&t=11985s))。

| Workload | 主要用途 | 關鍵特性 |
| --- | --- | --- |
| ReplicaSet | 維持指定 Pod 副本數 | 自動補回失敗副本, 通常由 Deployment 管理 |
| Deployment | 一般無狀態服務 | 管理 ReplicaSets, 支援更新與 rollback |
| DaemonSet | 每個合適 node 執行一份 | 適合 log collector、監控代理 |
| StatefulSet | 需要穩定身分與儲存的服務 | Pods 有固定序號, 依序建立及反序刪除 |
| Job | 執行到成功完成的工作 | 適合一次性或批次任務 |
| CronJob | 依排程建立 Jobs | 使用 cron 格式定時執行 |

StatefulSet 通常搭配 headless Service 與 PersistentVolumeClaims。課程也提醒, 在 Kubernetes 內自行經營資料庫仍有相當複雜度, 使用雲端受管資料庫有時是更合適的選擇。

## 更新與 rollback

Deployment 支援 Recreate 與 RollingUpdate ([04:05:15](https://www.youtube.com/watch?v=kTp5xUtcalw&t=14715s)):

- Recreate 先停止所有舊 Pods, 再建立新版本, 可能產生停機時間。
- RollingUpdate 逐步以新 Pods 取代舊 Pods。
- `maxSurge` 控制更新期間可超出期望數量的 Pods。
- `maxUnavailable` 控制更新期間可暫時不可用的 Pods。

```bash
kubectl rollout status deployment/my-app
kubectl rollout history deployment/my-app
kubectl rollout undo deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=2
```

若新舊版本無法同時工作, 可使用 blue-green pattern。Blue 與 Green 各維持一套 Deployment, 驗證新版本後再修改 Service selector 切換流量。這需要足夠資源同時運行兩套版本, 資料庫 schema 遷移仍可能需要額外處理。

## Services 與網路入口

Pod IP 是短暫的, Pod 重建後可能改變。Service 提供持久的虛擬 IP、DNS name、Pod selection 與負載分配 ([04:21:13](https://www.youtube.com/watch?v=kTp5xUtcalw&t=15673s))。

| 類型 | 可見範圍 | 適合情境 |
| --- | --- | --- |
| ClusterIP | 叢集內部 | 微服務間的穩定入口, 預設類型 |
| NodePort | 叢集內外 | 透過 node IP 與高位 port 暴露服務 |
| LoadBalancer | 叢集外部 | 由雲端 provider 建立 L4 load balancer |
| Ingress | HTTP/HTTPS 路由 | 以 L7 host/path rules 共用外部入口 |

Service 的 `port` 是 Service 接收流量的 port, `targetPort` 是 Pod 實際接收流量的 port。NodePort 另有 `nodePort`, 課程說明其預設範圍為 30000 至 32767。

## 儲存與持久化

Kubernetes containers 預設也是短暫且無狀態的。課程介紹靜態與動態供應方式 ([04:44:03](https://www.youtube.com/watch?v=kTp5xUtcalw&t=17043s)):

```text
靜態: 管理員建立 PersistentVolume -> 使用者建立 PVC -> Pod 掛載 PVC
動態: 管理員建立 StorageClass -> 使用者建立 PVC -> provisioner 建立 volume
```

PersistentVolume (PV) 是叢集層級的儲存資源, PersistentVolumeClaim (PVC) 是 workload 對儲存的請求。StorageClass 則描述可動態供應的儲存類別, 不需管理員預先建立固定容量 PV。

必須特別檢查:

- Access mode, 例如 ReadWriteOnce、ReadOnlyMany、ReadWriteMany。
- Reclaim policy。`Delete` 可能在 claim 釋放後刪除資料, `Retain` 則保留資源等待人工處理。
- StatefulSet 刪除後, PVCs 不一定隨之刪除, 必須另行確認。
- `hostPath` 只適合單節點本機測試, 不適合作為多節點正式儲存方案。

## ConfigMaps 與 Secrets

ConfigMap 將非敏感設定與 Pod manifest 解耦 ([05:03:48](https://www.youtube.com/watch?v=kTp5xUtcalw&t=18228s))。值可注入為環境變數, 也可掛載成檔案。環境變數在 container 啟動時注入, ConfigMap 更新後通常需要重新啟動 workload 才會取得新值; 以 volume 掛載時則能反映檔案更新, 但應用必須以讀檔方式取得設定。

Kubernetes Secret 的用法與 ConfigMap 類似, 但 manifest 的 `data` 通常是 base64 編碼。Base64 不是加密。敏感資料需要搭配 RBAC、etcd encryption at rest, 或外部 secret manager, 不能因資源名稱叫 Secret 就假設資料已安全加密。

## 健康檢查與可觀測性

Kubernetes 知道 container process 是否存在, 卻不一定知道應用是否真正可用。Probes 將應用狀態納入控制迴路 ([05:22:24](https://www.youtube.com/watch?v=kTp5xUtcalw&t=19344s)):

| Probe | 回答的問題 | 失敗後的主要行為 |
| --- | --- | --- |
| Startup | 應用是否已完成啟動? | 啟動期間保護較慢的應用 |
| Readiness | 現在能否接收流量? | 暫停將流量送到該 Pod |
| Liveness | 應用是否仍健康運作? | 重新啟動 container |

Probe 可執行 container 內命令、檢查 TCP socket, 或發出 HTTP GET。設定過於積極會造成健康應用被反覆重啟, 因此要根據真實啟動時間與暫時性故障設定 initial delay、period 及 failure threshold。

課程示範三種觀察叢集的介面 ([05:30:46](https://www.youtube.com/watch?v=kTp5xUtcalw&t=19846s)):

- Kubernetes Dashboard: 部署於叢集內的 Web UI, 需要審慎處理暴露方式與權限。
- Lens: 安裝在本機的桌面介面, 可切換 kubeconfig contexts。
- K9s: 在 terminal 中快速瀏覽資源、logs、shell、YAML 與 port forwarding。

無論使用哪種介面, `kubectl describe`、events 與 logs 仍是重要的故障排除證據。

## Horizontal Pod Autoscaler

HPA 根據 metrics 自動調整 Deployment 等 workload 的 Pod 數量 ([05:47:36](https://www.youtube.com/watch?v=kTp5xUtcalw&t=20856s))。課程以 CPU utilization 為例, 設定最少與最多 replicas。要讓百分比型 CPU 指標有意義, Pods 必須設定 resource requests; 叢集也需要可提供資料的 metrics server。

```bash
kubectl autoscale deployment my-app \
  --cpu-percent=50 --min=1 --max=4
kubectl get hpa
kubectl delete hpa my-app
```

刪除 HPA 不會自動把 Deployment 縮回原始副本數, workload 會維持當下 replicas, 需要另行調整。

## 建議的實作學習順序

1. 以單一 Dockerfile 建置並執行 image。
2. 練習 logs、exec、ports、bind mounts 與 volumes。
3. 用 Docker Compose 組合 frontend、backend 與 database。
4. 將 image 推送到 registry, 理解 tags 與 pull 流程。
5. 在本機 Kubernetes 建立 namespace 並確認 context。
6. 以 Deployment 取代裸 Pod, 再以 ClusterIP Service 建立服務發現。
7. 練習 RollingUpdate、rollback 及 blue-green traffic switch。
8. 加入 ConfigMap、Secret、PVC 與 probes。
9. 使用 `describe`、logs、K9s 或 Lens 觀察每次變更。
10. 最後才加入 metrics server 與 HPA, 驗證 requests、limits 與 scaling 行為。

## 來源與時效限制

本筆記依英文原始語言自動字幕與影片章節整理, 並非逐字稿。字幕對講者姓名、產品名稱及部分命令有辨識誤差, 本文只在上下文足以確認時校正。課程發布於 2022 年, Docker Desktop、Compose、Kubernetes、Lens、metrics server 及各 CLI 的安裝方式與預設行為可能已改變。本文保留課程的概念與示範流程, 實際操作前應再查閱目前版本的官方文件。
