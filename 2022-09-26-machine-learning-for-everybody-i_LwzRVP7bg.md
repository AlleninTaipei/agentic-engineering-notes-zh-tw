# Machine Learning for Everybody: 從資料準備到分類, 回歸與非監督學習

> 影片: [Machine Learning for Everybody - Full Course](https://www.youtube.com/watch?v=i_LwzRVP7bg)  
> 頻道: freeCodeCamp.org  
> 講者與課程作者: Kylie Ying  
> 原始網址: https://www.youtube.com/watch?v=i_LwzRVP7bg  
> 發布日期: 2022-09-26  
> 片長: 3:53:53  
> Video ID: `i_LwzRVP7bg`  
> 內容依據: YouTube 創作者提供的英文字幕 (`en`)

## 摘要

這門課以三個公開 dataset 與 Google Colab notebooks, 介紹 machine learning 的基本工作流程. 內容從 features, labels, training, validation 與 testing 開始, 接著實作 K-Nearest Neighbors, Naive Bayes, Logistic Regression, Support Vector Machine 與 neural networks. 後半部轉向 linear regression, regression neural network, K-Means clustering 與 Principal Component Analysis.

課程主線可以整理為:

```text
理解問題與資料
  -> 區分 classification, regression 或 unsupervised task
  -> 探索與清理 features
  -> 切分 train / validation / test
  -> 只用 training data 建立 preprocessing
  -> 訓練多個 baseline models
  -> 用適合問題的 metrics 比較
  -> 在 untouched test set 做最後評估
```

它適合第一次接觸 machine learning 的讀者, 因為概念解釋和 notebook 操作交錯進行. 課程不涵蓋 production deployment, data versioning, experiment tracking, distribution shift, model monitoring 或 responsible AI, 因此不能視為完整的 ML engineering 課程.

## 實作環境與三個 datasets

[00:00:58](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=58s)

課程使用 Google Colab, NumPy, pandas, Matplotlib, Seaborn, scikit-learn, imbalanced-learn 與 TensorFlow. 三條實作主線是:

| Dataset | Task | 主要方法 |
| --- | --- | --- |
| MAGIC Gamma Telescope | Gamma / hadron binary classification | KNN, Naive Bayes, Logistic Regression, SVM, neural network |
| Seoul Bike Sharing Demand | Hourly rental count regression | Linear Regression, regression neural network |
| Seeds / wheat kernels | Unsupervised grouping and visualization | K-Means, PCA |

影片從 UCI Machine Learning Repository 下載資料, 在 Colab 讀取 CSV 或 text files, 再透過 notebook 逐步探索, 轉換與建模. 影片說明中的 Colab links 可作為對照實作, 但 package APIs 與 notebook runtime 可能已在 2022 年後改變.

## Machine learning 的位置

[00:08:45](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=525s)

課程將 machine learning 定義為讓 computer 從 data 中學習 patterns 並做 predictions, 而不是由 programmer 明確寫出每一條 decision rule.

它同時區分三個相近領域:

- Artificial intelligence: 讓 machines 執行具有 intelligent behavior 的廣泛領域.
- Machine learning: AI 的一部分, 以 data 和 algorithms 解決 prediction 或 pattern discovery 問題.
- Data science: 從 data 中找出 patterns 與 insights, 可能使用 machine learning, 也可能使用 statistics 或其他分析方法.

課程介紹三類 machine learning:

| 類型 | Training signal | 典型問題 |
| --- | --- | --- |
| Supervised learning | 每個 input 有對應 label | Classification, regression |
| Unsupervised learning | 沒有 target labels | Clustering, dimensionality reduction |
| Reinforcement learning | Environment 回傳 rewards 或 penalties | Agent 依 interaction 學習 policy |

本課實際聚焦 supervised 與 unsupervised learning, reinforcement learning 只做概念介紹.

## Features 與 labels

[00:12:26](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=746s)

每一筆 sample 可表示為 feature vector `x`, 多筆 samples 組成 feature matrix `X`. Supervised learning 的預期輸出則是 target 或 label `y`.

課程將 feature types 分成:

### Qualitative features

- Nominal data: categories 沒有自然順序, 例如 country. 常用 one-hot encoding 將每個 category 轉成獨立 binary feature.
- Ordinal data: categories 具有順序, 例如 rating levels. 可以用 ordered numbers 表示, 但數字間距是否真的相等仍需依 domain 判斷.

### Quantitative features

- Discrete data: 可數的數量, 常以 integers 表示.
- Continuous data: length, temperature 等可落在連續尺度上的 values.

Encoding 不是純格式轉換. 若錯誤地為 nominal categories 指定 1, 2, 3, model 可能把不存在的順序與距離視為有效 signal.

## Classification 與 regression

[00:17:23](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=1043s)

Classification 預測 discrete classes:

- Binary classification: spam / not spam, gamma / hadron.
- Multiclass classification: cat / dog / lizard 或多種 plant species.

Regression 預測 continuous numerical value, 例如 temperature, house price 或 rental bike count. 兩者都屬 supervised learning, 差異在 target type 與 evaluation metrics.

## Training, validation 與 testing

[00:19:57](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=1197s)

Model training 將 predictions 與 known labels 比較, 透過 loss 量化差距, 再調整 model parameters. 若所有資料都參與 training, 就無法知道 model 能否 generalize 到 unseen data, 因此課程將 dataset 分成:

| Split | 用途 | 是否用於更新 model |
| --- | --- | --- |
| Training set | Fit parameters | 是 |
| Validation set | 選擇 model 與 hyperparameters | 否, 但會間接影響 model selection |
| Test set | 最後一次估計 generalization | 否 |

常見比例可以是 60/20/20 或 80/10/10, 但選擇取決於 dataset size, class distribution, time dependency 與 evaluation needs. Test set 應在 model 和 hyperparameters 確定後才使用, 否則會逐漸成為另一個 validation set.

### Loss functions

影片以三種 loss 說明 prediction error:

```text
L1 loss:                |y - y_hat|
L2 loss:                (y - y_hat)^2
Binary cross-entropy:   binary probability prediction 的常見 loss
```

L1 對 error 線性增加, L2 對較大的 error 給予更高 penalty. Binary cross-entropy 適合輸出 class probability 的 binary classifier.

## Data preparation

[00:30:57](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=1857s)

MAGIC dataset 的 target 原本是 `g` 與 `h`, notebook 將它轉成 numerical labels. 課程接著完成四個步驟:

1. 使用 histograms 比較兩個 classes 的 feature distributions.
2. Shuffle 後切分 training, validation 與 test sets.
3. 使用 `StandardScaler` 標準化 numerical features.
4. 使用 `RandomOverSampler` 平衡 training set 的 classes.

Oversampling 只套用 training set 是重要原則. 若 validation 或 test samples 被重複, evaluation distribution 便不再代表自然收到的資料.

### 編者補充: 避免 preprocessing leakage

影片的 helper function 對傳入的每個 split 呼叫 `fit_transform`. 更穩健的流程應只在 training features 上 fit scaler, 再使用同一組 mean 與 standard deviation transform validation 和 test data:

```python
scaler.fit(X_train)
X_train = scaler.transform(X_train)
X_valid = scaler.transform(X_valid)
X_test = scaler.transform(X_test)
```

Categorical encoders, feature selectors, imputers 與 PCA 也應遵循同樣原則. 任何從 validation 或 test distribution 學得的 preprocessing parameters 都可能造成 leakage.

## Classification metrics

Accuracy 是 correct predictions 佔所有 predictions 的比例, 但 class imbalance 下可能具有誤導性. 課程也使用 precision, recall 與 F1 score:

```text
Precision = TP / (TP + FP)
Recall    = TP / (TP + FN)
F1        = 2 * Precision * Recall / (Precision + Recall)
```

- Precision 關心被 model 判成 positive 的 samples 有多少是真的 positive.
- Recall 關心所有真實 positive samples 中有多少被找出.
- F1 在 precision 與 recall 之間取 harmonic mean.

實際選擇要依 error cost. Medical screening 可能更重視 recall, spam blocking 或高成本自動 action 則可能更在意 precision.

## K-Nearest Neighbors

[00:44:43](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=2683s)

KNN 對新 sample 計算與 training samples 的距離, 找出最近的 `k` 個 neighbors, 再以 majority vote 決定 class. 它不建立複雜 parametric model, prediction 主要依賴保存的 training data.

關鍵取捨包括:

- `k` 太小時容易受到 noise 與 outlier 影響.
- `k` 太大時可能抹去 local structure.
- Distance-based method 對 feature scale 敏感, 因此 standardization 很重要.
- Dataset 或 feature dimensions 增大時, prediction cost 與 curse of dimensionality 會變明顯.

[00:52:42](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=3162s)

Notebook 使用 scikit-learn 的 `KNeighborsClassifier`, 以 validation set 比較不同 `n_neighbors`, 最後在 test set 輸出 classification report.

## Naive Bayes

[01:08:43](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=4123s)

Naive Bayes 使用 Bayes theorem 計算給定 features 時某個 class 的 posterior probability:

```text
P(C | X) = P(X | C) * P(C) / P(X)
```

"Naive" 來自 conditional independence assumption, 也就是在已知 class 後將 features 視為彼此獨立. 這個假設常不完全成立, 但 model 在 text classification 或某些結構簡單的問題仍可能成為有效 baseline.

[01:17:30](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=4650s)

MAGIC features 是 continuous values, notebook 因此使用 Gaussian Naive Bayes. 短實作也展示 scikit-learn classifiers 共享 `fit`, `predict` 與 evaluation pattern.

## Logistic Regression

[01:19:22](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=4762s)

Logistic Regression 將 linear combination 傳入 sigmoid function, 將輸出壓縮到 0 至 1:

```text
z = w_0 + w_1*x_1 + ... + w_n*x_n
sigmoid(z) = 1 / (1 + exp(-z))
```

輸出可解讀為 model probability estimate, 再使用 threshold 轉成 class. 雖然名稱包含 regression, 它在這裡是 classification algorithm.

影片以簡短 scikit-learn implementation 建立 baseline. 實務上還應檢查 regularization, class weights, threshold, probability calibration 與 feature interactions.

## Support Vector Machine

[01:29:13](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=5353s)

SVM 尋找能分隔 classes 且 margin 最大的 decision boundary. 離 boundary 最近的 training samples 是 support vectors, 它們對 boundary 位置具有直接影響.

當資料不能由直線或 hyperplane 分隔時, kernel trick 可在不明確建立所有高維 features 的情況下計算 transformed space 中的 similarity. 常見 kernels 包括 linear, polynomial 與 radial basis function.

[01:37:54](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=5874s)

Notebook 以 scikit-learn `SVC` 實作. SVM 對 feature scaling 與 hyperparameters `C`, `gamma`, kernel choice 敏感, 大型 datasets 下 training cost 也可能成為限制.

## Neural networks

[01:39:44](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=5984s)

Neural network 由 layers, neurons, weights, biases 與 activation functions 組成. 每個 neuron 將 inputs 做 weighted sum, 加上 bias, 再通過 activation:

```text
z = w_1*x_1 + w_2*x_2 + ... + w_n*x_n + b
a = activation(z)
```

若所有 layers 都只有 linear operations, 多層 network 仍可化簡成一個 linear transformation. ReLU, sigmoid 等 nonlinear activations 讓 network 能表示更複雜的 functions.

Training loop 是:

```text
Forward pass
  -> Calculate loss
  -> Backpropagate gradients
  -> Update weights with an optimizer
  -> Repeat for multiple epochs
```

Learning rate 控制每次 parameter update 的大小. 過小會使 convergence 緩慢, 過大可能讓 optimization oscillate 或 diverge.

## TensorFlow classification network

[01:47:57](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=6477s)

課程使用 TensorFlow/Keras `Sequential` model 建立 binary classifier:

```text
Input features
  -> Dense layer with ReLU
  -> Dense layer with ReLU
  -> Single sigmoid output
```

[01:49:50](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=6590s)

Model 使用 binary cross-entropy, Adam optimizer 與 accuracy metric. Notebook 追蹤 training/validation loss 和 accuracy, 並比較 nodes, dropout probability, learning rate 與 batch size.

影片示範的 neural network 和 SVM 在 MAGIC test set 上得到相近的約 87% accuracy. 這個結果提醒讀者, neural network 不會因模型較複雜就自動優於傳統方法. Dataset size, feature representation, latency, interpretability 與 maintenance cost 都應納入比較.

### Validation 使用上的注意事項

影片先在 `model.fit` 使用 `validation_split`, 又使用獨立 validation set 選擇最低 loss 的 model. 講者也在影片中指出, 直接傳入既有 validation data 會更一致. 若反覆依 validation performance 挑選大量 configurations, 最終仍應只用 untouched test set 報告結果.

## Linear regression

[02:10:12](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=7812s)

Simple linear regression 以直線描述 feature `x` 與 continuous target `y` 的關係:

```text
y_hat = b_0 + b_1*x
```

Multiple linear regression 將多個 features 納入:

```text
y_hat = b_0 + b_1*x_1 + ... + b_n*x_n
```

Residual 是 observed value 與 prediction 的差. Model fitting 通常尋找能降低 residual-based objective 的 coefficients.

課程也介紹 linear regression 的主要 assumptions:

- Linearity: features 與 target 的關係可由 linear form 合理描述.
- Independence: observations 或 errors 不應彼此依賴.
- Normality: 常指 residuals 在 inference context 下近似 normal distribution.
- Homoscedasticity: residual variance 在 fitted values 範圍內大致穩定.

這些 assumptions 是否重要, 取決於目標是 prediction 還是 statistical inference. Residual plots, time order, group structure 與 domain knowledge 都應參與診斷.

### Regression metrics

| Metric | 意義 | 特性 |
| --- | --- | --- |
| MAE | Absolute errors 的平均 | 與 target 同單位, 對 outliers 較不敏感 |
| MSE | Squared errors 的平均 | 對大 error 給予較高 penalty |
| RMSE | MSE 的平方根 | 與 target 同單位, 仍強調大 error |
| R-squared | 相較只預測 target mean 所解釋的 variance | 不代表 causal relationship, 也不保證 out-of-sample performance |
| Adjusted R-squared | 對 feature count 加入修正 | 減少單純加入 variables 對 R-squared 的機械性提升 |

## Seoul bike demand regression

[02:34:54](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=9294s)

Notebook 使用 Seoul Bike Sharing Demand dataset, 以 hour, temperature, humidity, wind, visibility, radiation, rain, snow 等 features 預測 hourly bike count. 實作包含資料清理, feature plots, train/validation/test split, scaling 與 linear model evaluation.

時間相關資料若隨機 shuffle, 可能讓未來 distribution 的資訊進入 training set. 影片定位為入門示範, 但真正 forecasting 應依時間切分, 並考慮 seasonality, trend, holiday 與 autocorrelation.

## 以單一 neuron 理解 regression

[02:57:44](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=10664s)

沒有 nonlinear activation 的單一 neuron, 本質上執行 weighted linear combination, 因此可以表示 linear regression. 這個對照把傳統 linear model 與 neural network 放在同一個 computational view 下.

[03:00:15](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=10815s)

接著課程建立 regression neural network, 將 output layer 改為 single linear output, 並以 regression loss 訓練. 這再次顯示 output activation 與 loss 必須符合 target type, binary classifier 的 sigmoid output 不適合無界 continuous prediction.

## K-Means clustering

[03:13:13](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=11593s)

K-Means 將 samples 分配到 `k` 個 clusters. 基本迴圈是:

```text
Initialize k centroids
  -> Assign each sample to nearest centroid
  -> Recompute each centroid from assigned samples
  -> Repeat until assignments or centroids stabilize
```

它適合近似 spherical, distance-based clusters, 但對 feature scale, initialization, outliers 與 `k` 的選擇敏感. Cluster IDs 沒有語意, 因此不能直接將 cluster number 0 與 ground-truth class number 0 比較, 必須先建立 correspondence.

## Principal Component Analysis

[03:23:46](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=12226s)

PCA 將高維資料投影到較少的 orthogonal components. First principal component 是資料 variance 最大的方向, 後續 components 在與前面 components orthogonal 的限制下依序保留剩餘 variance.

另一個等價觀點是, projection 選擇能最小化 reconstruction residual 的 lower-dimensional subspace. 實際計算涉及 linear algebra, eigenvectors, eigenvalues 或 singular value decomposition, 影片不展開數學推導.

PCA 常用於 visualization, compression, noise reduction 或 preprocessing, 但 components 是原始 features 的 linear combinations, 可能降低 interpretability. PCA 也必須只在 training data 上 fit, 避免 evaluation leakage.

## Seeds dataset 的 K-Means 與 PCA

[03:33:54](https://www.youtube.com/watch?v=i_LwzRVP7bg&t=12834s)

最後一個 notebook 使用 wheat kernels 的 area, perimeter, compactness, length, width, asymmetry 與 groove length. 示範包含:

1. 以 scatter plots 觀察不同 feature pairs.
2. 使用 `KMeans(n_clusters=3)` 對 samples 分群.
3. 比較 cluster assignments 與已知 wheat classes.
4. 使用 PCA 將七個 features 降為兩個 components.
5. 在 PCA space 視覺化 K-Means 與 ground truth.

這個展示讓 clustering 結果容易理解, 但 `k=3` 是由已知三個 wheat varieties 提供. 真正 unlabeled problem 通常不知道 cluster count, 還需使用 domain knowledge, silhouette score, stability 或其他方法評估 `k`.

## 可重複使用的 ML workflow

以下是根據整門課整理的實務流程, 不是影片逐字提供的單一 checklist.

### Problem framing

- Target 是 discrete class, continuous value, 還是根本沒有 label?
- Prediction 將如何被使用, false positive 與 false negative 的成本是什麼?
- 是否真的需要 ML, 或 deterministic rule 已足夠?

### Data preparation

- 確認 sample, feature, label 與 measurement process.
- 先決定 split strategy, 再 fit scaler, encoder, imputer, PCA 或 feature selection.
- Oversampling, undersampling 與 augmentation 只作用於 training data.
- 時間, user, device 或 organization 相關資料需避免跨 split leakage.

### Modeling

- 先建立簡單 baseline, 再增加 model complexity.
- 依 dataset size, feature scale, nonlinearity, latency 與 interpretability 選擇 algorithm.
- Hyperparameter search 使用 validation set 或 cross-validation, 不碰 test set.

### Evaluation

- Classification 不只看 accuracy, 也檢查 precision, recall, F1, confusion matrix 與 threshold.
- Regression 同時查看 MAE, RMSE, R-squared, residuals 與不同 slices 的 error.
- 最後在 untouched test set 評估一次, 並保存 data, code, random seed 與 environment.

## 來源與限制

- 本筆記依據 creator-provided English captions 整理, 不是逐字稿. 影片中的口語重複, typing mistakes 與除錯過程已濃縮, 但保留對理解 workflow 有幫助的錯誤與修正.
- 課程提供三份 Google Colab notebooks 與公開 UCI datasets, 因此主要實作可重現. 連結, package versions 與 notebook output 可能已在 2022 年後改變.
- 影片是 absolute beginner course, 數學推導刻意簡化. 它沒有深入 probability, linear algebra, calculus, optimization theory, statistical inference 或 uncertainty estimation.
- `StandardScaler` 的示範對每個 split 分別 `fit_transform`, 可能造成不一致 preprocessing. 正確流程應在 training data fit, 再 transform validation 與 test data.
- Bike regression 使用隨機資料切分作教學示範. 若問題是預測未來 demand, 應改用 time-based split, 否則可能高估 generalization.
- K-Means 實作使用已知的三個 wheat classes 選擇 `k=3`, 因此不代表在完全未知結構下能自動發現正確群數.
- 課程沒有涵蓋 data governance, deployment, monitoring, drift, fairness, privacy, security 或 incident response. 任何高風險或 production use case 都需要額外驗證與治理.
