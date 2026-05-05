# 即時系統 Ch1–Ch3 期中考懶人包

---

## Ch1：Typical Real-Time Applications

### 核心定義
- **Real-time system**：必須在時限內完成服務（不一定要快，但要**準時**）
- **Embedded system**：隱藏在更大系統內部，用戶通常感知不到

### Digital Process Control（數位程序控制）

**Sampled Data System 架構**
```
Sensor → A/D → [控制律計算] → D/A → Actuator → Plant（回饋）
```
- y(t)：量測到的 plant 狀態
- r(t)：期望狀態
- u(t)：控制輸出 = f(y(t), r(t))

**PID 控制器公式**
```
u(k) = Kp·e(k) + Ki·Σe(n) + Kd·(e(k) - e(k-1))
```
- Kp：比例（Proportional）
- Ki：積分（Integral）
- Kd：微分（Derivative）

**取樣週期 T 的選擇**

| T 大小 | 優點 | 缺點 |
|--------|------|------|
| 小 T | 近似連續行為，精準 | 佔用大量處理器時間 |
| 大 T | 省處理器資源 | 系統震盪（oscillation） |

- 黃金比例：**10 ≤ R/T ≤ 20**（R = rise time）
- R/T < 10 → 控制輸出震盪，系統不穩定

**Digital Controller 的三個假設（Summary 重點）**
1. 感測器資料無雜訊，能精確估計狀態變數
2. 感測器資料直接代表 plant 狀態
3. 所有代表 plant 動態的參數已知

> ⚠️ 任一假設不成立 → 需加入**狀態估計（State Estimation）**

**進階控制**
- **Kalman Filter**：每個取樣週期做矩陣乘加 + 一次矩陣反轉，過濾雜訊估計真實狀態
- **Deadbeat Control**：純離散時間控制，無連續時間等效

**Multi-rate System（多速率系統）**
- 不同狀態變數動態不同（如轉速 vs. 溫度）→ 需要不同取樣週期
- 各週期最好保持**調和（harmonic）關係**

---

### 其他 RT 應用（概略即可）

| 應用 | 特色 |
|------|------|
| High-Level Command & Control（ATC） | 階層式控制，越高層 deadline 越寬鬆 |
| Signal Processing（雷達） | Fourier Transform，10³~10⁵ 次乘加 |
| Real-Time Database | 資料物件需反映真實世界狀態（如飛機位置） |
| Telephony & Multimedia | MPEG 壓縮、影音同步（lip sync） |

---

## Ch2：Hard vs. Soft Real-Time Systems

### 基本名詞

| 名詞 | 定義 |
|------|------|
| **Job** | 系統排程執行的最小工作單位 |
| **Task** | 一組相關 jobs 的集合 |
| **Release time (r)** | job 可以開始執行的最早時間點，釋放時間可抖動|
| **Response time** | release time → 完成時間（≠ execution time） |
| **Completion time** | job 完成執行的時間點 |
| **Relative deadline (D)** | 最大允許的 response time |
| **Absolute deadline (d)** | = release time + relative deadline |
| **Feasible interval** | (rᵢ, dᵢ] |
| **Tardiness** | 遲到了多少，max(0, 完成時間 - deadline)

- 準時或提早完成 → Tardiness = 0
- 超過 deadline 才完成 → Tardiness = 超過了多少 ms|
| **Slack time** | deadline 前的剩餘空閒時間 |

<img width="805" height="424" alt="image" src="https://github.com/user-attachments/assets/eb06cab8-b824-4cdc-8e25-f264139926df" />

假設 CPU 同時有兩個 job 要跑：

- J0 release: 20ms，execution time: 75ms  
- J1 release: 50ms，execution time: 75ms ← 插進來了！
時間軸：
<img width="617" height="199" alt="image" src="https://github.com/user-attachments/assets/3cd564a7-bd95-4569-b87f-b7347dfa66d2" />

**J1 的計算：**
- Release time = 50ms
- 但 CPU 在忙 J0，要等到 95ms 才能開始跑
- Waiting time = 95 − 50 = **45ms**
- Execution time = **75ms**
- 完成時間 = 95 + 75 = **170ms**
- Response time = 170 − 50 = **120ms**

> Response time(120) = Waiting(45) + Execution(75) ✅

簡單說：**CPU 只有一個，同時來兩個 job，後來的那個就要排隊等。**
### Hard vs. Soft Deadline 比較

| | **Hard** | **Soft** |
|--|----------|----------|
| 定義 | 未達成 = fatal fault | 未達成是 undesirable，但不致命 |
| Usefulness 曲線 | 超過 deadline 後**驟降**（可負） | 超過後**逐漸遞減** |
| 驗證要求 | 需嚴格保證（Guaranteed） | 統計滿足即可（Best-effort） |
| 例子 | 火車煞車、飛行控制 | 電話網路、多媒體串流 |

**Usefulness 三種類型（Job A, B, C）**
- Job A（Hard）：到 deadline 立刻掉到 0
- Job B（Soft，陡降）：過 deadline 後快速遞減
- Job C（Soft，緩降）：過 deadline 後緩慢遞減

### Hard Timing Constraint 的三種規格方式
1. **Deterministic**：每次 deadline 都必須達成（如：每次 ≤ 50ms）
2. **Probabilistic**：以機率表達（如：P(response > 50ms) < 0.2）
3. **Usefulness function**：以效用函數表達（如：usefulness ≥ 0.8）

### 系統驗證三步驟
1. **Consistency**：確認 timing constraints 規格正確
2. **Feasibility**：確認每個元件在硬體/軟體資源下可行
3. **Schedulability**：確認整體系統行為符合 timing constraints

---

## Ch3：A Reference Model of Real-Time Systems

### 系統三要素
1. **Workload model**：描述系統支援的應用
2. **Resource model**：描述可用的系統資源
3. **Algorithms**：定義系統如何在所有時間點使用資源

### 資源分類

| 類型 | 別名 | 例子 |
|------|------|------|
| **Processor（主動資源）** | Server / Active resource | CPU、傳輸鏈路、磁碟、DB server |
| **Resource（被動資源）** | Passive resource | 記憶體、mutex、sequence number、DB lock |

> 同類型 processor：功能相同、可互換（如 SMP 中的 CPU）  
> 不同類型 processor：CPU、傳輸鏈路、磁碟 互不相同

### Job 的四個參數
1. **Temporal**：release time (r)、absolute deadline (d)、relative deadline (D)
2. **Interconnection**：precedence constraints
3. **Functional**：job 要做什麼
4. **Resource**：需要哪些資源

### Release Time 的三種類型

| 類型 | 說明 |
|------|------|
| **Fixed** | 精確已知 |
| **Jittered** | 在 [rᵢ⁻, rᵢ⁺] 區間內 |
| **Sporadic / Aperiodic** | 隨機，用機率分布 A(x) 表示 |

### Execution Time
- 執行時間落在 [eᵢ⁻, eᵢ⁺] 區間
- 驗證時通常取 **eᵢ = eᵢ⁺**（最壞情況，safe but conservative）
- 影響因素：條件分支、cache、壓縮（如 MPEG）

### Periodic Task Model（最重要！）

| 符號 | 意義 |
|------|------|
| Tᵢ | 第 i 個 periodic task |
| pᵢ | 週期（period）|
| eᵢ | 最大執行時間 |
| Dᵢ | relative deadline（通常 Dᵢ = pᵢ）|
| Φᵢ | phase（第一個 job 的 release time）|
| uᵢ = eᵢ/pᵢ | task 的 utilization |
| U = Σuᵢ | 系統總 utilization |
| H = LCM(p₁,p₂,...,pₙ) | Hyperperiod |
| N = Σ(H/pᵢ) | Hyperperiod 內的 job 總數 |

**範例：** 三個 tasks，週期 3、4、10，執行時間 1、1、3
- U = 1/3 + 1/4 + 3/10 = 0.33 + 0.25 + 0.30 = **0.88**
- H = LCM(3,4,10) = **60**
- N = 60/3 + 60/4 + 60/10 = 20 + 15 + 6 = **41 jobs**

### 任務類型比較

| 類型 | Release time | Deadline |
|------|-------------|----------|
| **Periodic** | 固定週期 | 通常 Hard |
| **Sporadic** | 隨機 | **Hard** |
| **Aperiodic** | 隨機 | Soft 或無 |

### Precedence Constraints
- **Jᵢ < Jk**：Jᵢ 必須完成，Jk 才能開始執行
- **Immediate predecessor**：中間沒有其他 job
- **Independent jobs**：Jᵢ 和 Jk 互不影響
- 用 **Precedence graph**（有向圖 G = (J, <)）表示

### Data Dependency vs. Precedence
- **Precedence**：執行順序的強制約束
- **Data dependency**：透過共享資料溝通，**不一定有 precedence 關係**
  - 例：導航 job（更新位置）和飛行管理 job（讀取位置）是 data dependency，非 precedence

---

## 快速複習表

| 概念 | 核心重點 |
|------|---------|
| PID 公式 | u(k) = Kp·e(k) + Ki·Σe(n) + Kd·(e(k)-e(k-1)) |
| 取樣週期黃金比例 | 10 ≤ R/T ≤ 20 |
| Hard deadline | 沒趕上 = fatal，需嚴格驗證 |
| Soft deadline | 沒趕上只是 undesirable，統計驗證 |
| Tardiness | 完成時間 - deadline（準時完成 = 0）|
| Utilization | uᵢ = eᵢ/pᵢ，總 U = Σuᵢ |
| Hyperperiod | H = LCM(所有週期) |
| Sporadic vs Aperiodic | Sporadic = Hard deadline；Aperiodic = Soft/無 deadline |
| 驗證三步驟 | Consistency → Feasibility → Schedulability |
