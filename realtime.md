# 即時系統 期末考懶人包

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

> 🆕 **補充（為什麼大 T 會震盪）**：T 大 = 隔很久才量測+修正一次，系統反應慢半拍、容易「修正過頭」，狀態在目標值上下來回擺盪（如同每 3 秒才看一次路在開車 → 蛇行）。所以 T 要小到在 rise time 內至少取樣 10～20 次（R/T ≥ 10），否則不穩定。

**Digital Controller 的三個假設（Summary 重點）**

假設 1：感測器資料無雜訊

因為控制律計算 u(k) 是直接用 y(k) 代入公式的。
如果 y(k) 有雜訊 → 算出來的 u(k) 也會亂掉 → 控制輸出錯誤
所以必須先假設感測資料是乾淨的，否則需要加 Kalman Filter 來估計真實狀態。

假設 2：感測資料直接代表 plant 狀態

因為控制律需要知道 plant 現在的狀態，但感測器量到的不一定就是狀態本身。
例如：量到的是電壓，但真正要控制的是溫度 → 還需要轉換。
如果這個假設不成立 → 需要額外的狀態估計器（state estimator）。

假設 3：代表 plant 動態的參數已知

因為控制律的 Kp、Ki、Kd 是在設計時就決定好的，是根據 plant 的模型算出來的。
如果 plant 的參數在執行中改變（例如飛機燃料消耗導致重量改變）→ 原本的控制律就不準了。

> ⚠️ 任一假設不成立 → 需加入**狀態估計（State Estimation）**

| 假設 | 對應問題 | 不成立時要加什麼 |
|--------|------|------|
| 無雜訊 | 感測器量不準 | Kalman Filter |
| 資料 = 狀態 | 量到的不是你要的 | State Estimator |
| 參數已知 | Plant 會變化 | Adaptive Control |

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
| **Tardiness** |  遲到了多少，max(0, 完成時間 - deadline)<br> 準時或提早完成 → Tardiness = 0<br> 超過 deadline 才完成 → Tardiness = 超過了多少 ms |
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

**hard deadline兩個重要觀念：**

- 提早完成 hard deadline 沒有好處（準時就好）
- 要讓一串 jobs 的 response time jitter（抖動）越小越好（每次完成時間要穩定）

| | **Hard** | **Soft** |
|--|----------|----------|
| 定義 | 未達成 = fatal fault | 未達成是 undesirable，但不致命 |
| Usefulness 曲線 | 超過 deadline 後**驟降**（可負） | 超過後**逐漸遞減** |
| 驗證要求 | 需嚴格保證（Guaranteed） | 統計滿足即可（Best-effort） |
| 例子 | 火車煞車、飛行控制 | 電話網路、多媒體串流 |

> 🆕 **補充（Guaranteed vs Best-effort / Admission request）**：
> - **Guaranteed（保證）**＝使用者要求時序品質一定要被保證 → timing constraint 是 **hard**。
> - **Best-effort（盡力）**＝系統盡量做、但允許偶爾低於規格 → timing constraint 是 **soft**。
> - 當應用程式要**新增一個 hard 任務**時，必須先送一個 **admission request（入場申請）** 給 scheduler；scheduler 檢查「加進來後大家還趕得上嗎」→ 趕得上才 **admit（接受）**，否則 **拒絕**。（老師手寫「Hard task 要先 pass」就是指這個；和 Ch5 偶發 job 的 **acceptance test** 是同一個概念。）

**Usefulness 三種類型（Job A, B, C）**
- Job A（Hard）：到 deadline 立刻掉到 0
- Job B（Soft，陡降）：過 deadline 後快速遞減
- Job C（Soft，緩降）：過 deadline 後緩慢遞減

### Hard Timing Constraint 的三種規格方式
1. **Deterministic**：每次 deadline 都必須達成（如：每次 ≤ 50ms）
   - 🆕 **補充（還有一種變體寫法）**：「連續五次中最多一次的 response time 可以超過 50ms」也是 deterministic（容許少數失誤，但仍是死規定、不用機率）。
2. **Probabilistic**：以機率表達（如：P(response > 50ms) < 0.2）
3. **Usefulness function**：以效用函數表達（如：usefulness ≥ 0.8）

|  | Deterministic | Probabilistic | Usefulness |
|--------|------|------|------|
| 嚴格程度 | 最嚴 | 中等 | 最彈性 |
| 驗證方式 | 最壞情況分析 | 統計分析 | 效用函數計算 |
| 典型應用 | 飛控、煞車 | 電話網路 | 多媒體 |

### 系統驗證三步驟
1. **Consistency**：確認 timing constraints 規格正確 ( 規格寫對了嗎？       → Consistency )
2. **Feasibility**：確認每個元件在硬體/軟體資源下可行 ( 單個元件跑得完嗎？   → Feasibility  )
3. **Schedulability**：確認整體系統行為符合 timing constraints ( 全部一起跑沒問題嗎？ → Schedulability )

> 🆕 **補充（Scheduler 的「正確性」定義）**：一個 scheduler 是 **correct（正確）** 的，定義為——它**絕不在任何 job 的 release time 之前**就排它執行（每個 job 的開始時間都 ≥ 該 job 的 release time）。英文填空常考：A scheduler works correctly if it never schedules any job **before** its release time.

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

> 🆕「從 Φ 開始，每 p 做一次，每次最多做 e，要在 D 內做完」
>
> 📌 課本完整記法：(Φ, p, e, D)，如果 Φ=0 且 D=p 就省略 → 只寫 (p, e)。

（例：(10, 3) 代表 phase 0、週期 10、執行 3、deadline 10）

> 🆕 **補充（Task vs Job — 一定要分清楚）**：
> - **Task = 規格/設定**（每隔 p 生一個、每個做 e），本身**不會被執行**。像「鬧鐘設定：每天 7 點響」。
> - **Job = task 按週期生出來的一個個實體**，**真正被排程、被 CPU 跑的是 job**。像「今天 7 點響的那一次」。

**範例：** 三個 tasks，週期 3、4、10，執行時間 1、1、3
- U = 1/3 + 1/4 + 3/10 = 0.33 + 0.25 + 0.30 = **0.88**
- H = LCM(3,4,10) = **60**
- N = 60/3 + 60/4 + 60/10 = 20 + 15 + 6 = **41 jobs**
<img width="1520" height="1280" alt="hyperperiod_job_count_white_text" src="https://github.com/user-attachments/assets/74214e99-63be-47ce-ae91-2ceaee570160" />

> 🆕 **補充（看圖記 N 和 U）**：把每個 job 畫成方塊（寬度 = 執行時間 e），攤在 0~60：
> - **方塊「數量」= N**（20+15+6=41 個）；週期越短刻度越密、job 越多。
> - **方塊「總寬度」= 忙碌時間** = 20·1 + 15·1 + 6·3 = **53 格**；53/60 = **0.88 = U**。
> - 所以 U 的真義：**60 格裡有 53 格在忙、7 格空閒**（空閒留給 aperiodic / 背景工作）。

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

> 🆕 **補充（兩種圖長怎樣）**
>
> **Precedence graph**：只有實線箭頭＝先後限制。圈是 job，Jᵢ→Jₖ 代表「Jᵢ 做完，Jₖ 才能開始」。
>
> <img width="1520" height="1000" alt="precedence_graph_basic" src="https://github.com/user-attachments/assets/1409e4fa-c6d3-4bc8-b614-d330c7703ab7" />
>
> **Task graph**：是 **extended precedence graph**——一樣有實線先後箭頭，**再加上虛線的 data-dependency 邊**（共用資料、讀最新值、不用等）與其他應用資訊。
>
> <img width="1520" height="1120" alt="task_graph_with_data_dependency" src="https://github.com/user-attachments/assets/d649930f-03ba-4d54-9df7-5205e97ea610" />
>
> **一句話**：Precedence graph = 只畫「要等」；Task graph = 再加「共用資料但不用等」的虛線（資訊更完整的版本）。

### 🆕 練習題 3.1（用最壞情況把偶發流近似成週期任務）

**題目**：一串 sporadic job，到達間隔（interrelease time）均勻分布在 **9~11**，執行時間均勻分布在 **1~3**。
(a) 若用一個週期任務來模型化，參數是多少？ (b) 比較它的 utilization 與偶發流的平均 utilization。

**(a) 取最壞情況** → 週期取「最短間隔」、執行取「最長時間」：
- p = 最短間隔 = **9**
- e = 最長執行 = **3**
- → **T = (9, 3)**（D = p = 9）

**(b) 比較 utilization：**
- 週期模型（最壞）：u = e/p = 3/9 ≈ **0.333**
- 偶發流平均：平均執行 (1+3)/2 = 2、平均間隔 (9+11)/2 = 10 → u = 2/10 = **0.2**

**結論**：週期模型用最壞情況抓 → 0.333；實際平均只用 0.2。多預留的 ≈0.13 常空著浪費 → 這就是題幹說的「periodic model 太不精準、造成處理器 underutilization（資源浪費）」。

> 記法：**週期取最短間隔、執行取最長時間（都往最壞想）；保險值 3/9 比實際平均 2/10 高，差額 = 浪費。**
