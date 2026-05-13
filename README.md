# 小麥種子資料集：分群與迴歸分析

## 專案簡介

本專案使用 UCI Machine Learning Repository 的 **Seeds 資料集**，以 X 光量測 210 顆小麥核仁的七項幾何特徵，分別探索兩種機器學習方法：

- **K-Means 分群**：從零手刻 Lloyd's Algorithm，評估其在無監督情境下辨識三種小麥品種的能力
- **簡單線性迴歸**：從零手刻封閉解迴歸，預測核仁緊密度（compactness）與其他特徵間的線性關係

---

## 資料集說明

**來源：** [UCI Machine Learning Repository – Seeds](https://archive.ics.uci.edu/dataset/236/seeds)

| 欄位 | 特徵名稱 | 說明 |
|------|----------|------|
| 1 | `area` | 核仁面積 A |
| 2 | `perimeter` | 核仁周長 P |
| 3 | `compactness` | 緊密度 C = 4πA/P² |
| 4 | `length` | 核仁長度 |
| 5 | `width` | 核仁寬度 |
| 6 | `asymmetry coefficient` | 不對稱係數 |
| 7 | `length of kernel groove` | 核仁槽長度 |
| 8 | `class` | 品種標籤（1、2、3） |

共 210 筆觀測，三種小麥品種各 70 筆。

---

## 專案結構

```
homework/
├── assignment4_O.ipynb   # 主要分析 Notebook
├── seeds.tsv             # 原始資料集（Tab 分隔）
└── README_assignment4.md # 本說明文件
```

---

## 主要分析內容

### 一、分群分析（Clustering）

#### 1. 手刻 KMeansManual 類別
從零實作 Lloyd's Algorithm，不依賴 scikit-learn：
- `assign_labels`：計算每個資料點到各群中心的歐氏距離，指派最近的群標籤
- `compute_cluster_centers`：對各群資料點取平均，更新群中心
- `compute_inertia`：計算所有資料點到其群中心距離的平方和
- `fit`：迭代執行直到群標籤不再改變

#### 2. 資料標準化
使用 `StandardScaler` 對七個特徵進行 Z-score 標準化，消除量綱影響。

#### 3. 資料視覺化（三種投影方式）
- **特徵對散佈圖**：直接觀察兩兩特徵關係，面積×周長呈明顯線性相關且類別可分
- **主成分分析（PCA）**：前兩個主成分解釋大部分變異，三類幾乎不重疊
- **高斯隨機投影**：以隨機矩陣保留距離近似性，結果與 PCA 相近但無計算共變異矩陣的開銷

#### 4. 手肘法決定最佳群數
對 k=1 至 30 繪製慣性曲線，k=3 出現明顯轉折，對應三種小麥品種。

#### 5. 分群評估
| 指標 | 數值 | 說明 |
|------|------|------|
| Rand 指數 | ~0.90 | 與真實標籤的一致性 |
| 最佳排列準確率 | ~0.92 | 最優標籤對應下的分類準確率 |

K-Means 在此資料集上表現良好，顯示三群結構十分穩健。

---

### 二、迴歸分析（Regression）

#### 1. 手刻 SimpleLinearRegression 類別
以封閉解實作一元線性迴歸 y = α + βx：
- 斜率 β = Σ(xᵢ−x̄)(yᵢ−ȳ) / Σ(xᵢ−x̄)²
- 截距 α = ȳ − β·x̄

#### 2. 特徵選擇
以皮爾森相關係數找出與 `compactness` 相關性最高的特徵作為輸入。

#### 3. 模型評估結果
| 指標 | 數值 |
|------|------|
| R²（訓練集） | ~0.57 |
| RMS（測試集） | ~0.0147 |
| MAE（測試集） | ~0.0121 |
| MAPE（測試集） | ~1.4% |

#### 4. 結論
單一特徵能解釋約 57% 的變異，整體趨勢捕捉尚可但仍有改進空間。引入多元特徵或採用穩健迴歸可有效提升精度。

---

## 使用環境

```
Python 3.x
numpy
pandas
scikit-learn
matplotlib
```

---

## 執行方式

1. 確認 `seeds.tsv` 與 `Wheat_Seeds_Analysis_KMeans_Regression.ipynb` 在同一目錄
2. 在 Jupyter 環境中開啟 `Wheat_Seeds_Analysis_KMeans_Regression.ipynb`
3. 依序執行所有 cell（Kernel → Restart & Run All）
