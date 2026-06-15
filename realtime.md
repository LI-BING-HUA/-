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

### 🆕 練習題 3.2（把程式流程畫成 task / precedence graph）

**題目**：一天作息的 pseudocode（job 名稱斜體）：start→breakfast→office；10AM 若有課→teach，否則→help；teach/help 完→lunch；睡到 2PM→sleep；若有 seminar（主題有趣→listen，否則→read），否則→write office；seminar 結束→social hour→discuss→jog→dinner→work more→endtheday。
(a) 畫 task graph 表示 job 間相依。 (b) 用多張 precedence graph 表示所有可能路徑。

**(a) Task graph（一張圖含所有分支）**

<img width="2720" height="1680" alt="daily_routine_task_graph_white_labels" src="https://github.com/user-attachments/assets/f0ab6cae-7661-41d7-86f8-194b8afdf455" />


- 必做主線：start→breakfast→office→…→lunch→sleep→…→social→discuss→jog→dinner→work more→end
- 分支①（office 後）：**teach / help** 二選一 → 會合於 **lunch**
- 分支②③（sleep 後）：**listen / read / write** 三選一 → 會合於 **social hour**
- ⚠️ **write office → social hour 為題目未明寫的合理假設**（圖中以虛線表示）：因為最後有 `endtheday` 這個總終點，所有路徑都應能走到它，否則「沒 seminar」那天走不到結束。考試應在圖旁**註明此假設**。

**(b) 需要幾張 precedence graph？**

precedence graph 一張只能畫**一條確定路徑**（不能有 if/else 選擇）。分支組合數 = 張數：
- 分支①：teach / help → 2 種
- 分支②③：listen / read / write → 3 種
- → **2 × 3 = 6 張**，每張都是一條直線（把分岔拿掉只留實走的那條）。

> 一句話：**task graph 一張容得下所有 if/else 分岔；precedence graph 每張只能是走完的一條路 → 有幾種分支組合就畫幾張（這題 2×3=6）。**
> 申論加分點：題目模糊（如 write 之後沒明寫）時，**主動寫出你的假設**，閱卷老師會知道你想清楚了。

### 🆕 練習題 3.3（pipe 管線：生產者-消費者）

**題目**：符號 `job1 | job2` 代表一個 **pipe（管線）**：job1 產生的結果被 job2 **逐步（一點一點）消費**。例：job1 每辨識一個手寫字就放進 buffer，job2 每當一個字進 buffer 就立刻讀取並顯示（一次一個字）。請畫 precedence graph 表示這個生產者-消費者關係。

**關鍵差別：pipe ≠ 一般 precedence**
- 一般 `J1 < J2`：J1 **整個做完**，J2 才能開始 → 不能重疊
- pipe `J1 | J2`：J1 產一段、J2 立刻消費一段 → **生產與消費可同時進行（重疊）**

**畫法（把兩個 job 各切成對應小段，逐段配對）**
<img width="2720" height="1200" alt="pipe_producer_consumer_precedence" src="https://github.com/user-attachments/assets/81971fc5-742b-45b5-bcce-a5d2ff625367" />

- 上排橫箭頭 J1,1→J1,2→J1,3…：J1 一個字一個字產生
- 下排橫箭頭 J2,1→J2,2→J2,3…：J2 一個字一個字消費
- 直向箭頭 **J1,k → J2,k**：第 k 段的 precedence（J1 產出第 k 個字後，J2 才能消費第 k 個字）

> 一句話：**pipe = 切成小段、只有「對應段之間」有先後（J1,k → J2,k）；因為 J2,k 只等 J1,k、不等 J1 全做完，所以生產者和消費者能重疊執行。**

### 🆕 練習題 3.4（飛控系統 task graph：同步 vs 不同步）

**題目（Figure 1-3 飛控系統）**：每 1/180 秒一個 cycle。① 驗證感測資料/選資料源（每 cycle）；② 30-Hz avionics（鍵盤、座標轉換、追蹤更新，每6 cyc）；③ 30-Hz 外迴路控制律（pitch/roll/yaw outer，每6 cyc）；④ 90-Hz 內迴路控制律（inner pitch、inner roll+coll，每2 cyc，**用 30-Hz 計算+avionics 的輸出**）；⑤ inner yaw 控制律（**用 90-Hz 的輸出**）；⑥ output commands；⑦ built-in-test；⑧ 等下一個 cycle。
(a) 假設 producer/consumer **不同步**（用最新值，不等）畫 task graph。 (b) 假設**同步**（必須等 producer 完成）重畫。

**頻率換算**：180 Hz=基準（每 cycle）；30 Hz=每6 cyc；90 Hz=每2 cyc。

**資料流（題目只明寫兩段相依）**：
1. (30-Hz 外迴路 + avionics) → 90-Hz 內迴路
2. 90-Hz 內迴路 → inner yaw →（最後）output commands

**判斷獨立的鐵則**：題目只在「using outputs … as input」那兩句講相依，其餘沒講 = 獨立 = 並排。
- avionics ‖ 外迴路：題目沒說誰用誰 → **獨立，並排，無箭頭**
- built-in-test：不產生也不用控制資料 → **獨立，不連線**

**(a) 不同步 → data-dependency 虛線**（各頻率各跑各的，用上游最新值，不等）
<img width="2720" height="1840" alt="flight_control_taskgraph_a_no_sync" src="https://github.com/user-attachments/assets/5a48704d-1af8-44db-99f3-7ff13f67cd1d" />

**(b) 同步 → precedence 實線箭頭**（consumer 必須等 producer 這次做完）
<img width="2720" height="1840" alt="flight_control_taskgraph_b_sync" src="https://github.com/user-attachments/assets/cc593471-6406-4d77-b81a-c49a99664acf" />

> 一句話：**兩張圖節點、連線位置完全一樣，只差線的樣式——(a) 全虛線（不等、讀最新 = data dependency）、(b) 全實線箭頭（要等 = precedence）。** 同層並排=獨立（沒箭頭），先後只存在於上下層之間（有箭頭）。

---

## Ch4：Commonly Used Approaches to Scheduling（排程三大方法）

### 三大排程方法總覽

| 方法 | 怎麼決定誰先跑 | 特色 | 適用 |
|------|--------------|------|------|
| **Clock-driven（時鐘驅動）** | 排程**離線算好**、存表，固定時間點照表執行 | 所有參數要事先已知；run-time 負擔最小；確定性高 | 硬即時、參數固定（→ Ch5 frame size） |
| **Weighted round-robin** | FIFO 排隊，每個 job 輪流跑一個 time slice，可加權重 | 不需排序的 priority queue；適合 pipeline | 高速網路訊息傳輸 |
| **Priority-driven（優先權驅動）** | 給每個 job 優先權，高的先跑；**event-driven** | 又叫 greedy / list / work-conserving；**資源絕不無故閒置** | 大多數系統（EDF、LST 屬此類） |

> 🔑 **Priority-driven 的本質**：**只要有 job 等著、處理器就不會閒置**（greedy）。排程決策只在「job release」或「job 完成」這兩種**事件**發生時做。

> 🆕 **生活化比喻（秒懂三方法）**：
> - **Clock-driven = 營養午餐時間表**：菜單和時間「前一天就排好印出來」，時間到照表端菜。一切事先算好存成表，執行時不用臨場思考 → 負擔最小、最穩定，但所有事要事先知道。
> - **Weighted round-robin = 排隊玩遊戲機**：每個人輪流玩一小段（time slice），時間到換下一個，沒玩完排到隊尾等下一輪。VIP（weight 大）一輪多玩幾次。公平分時、不需排序。
> - **Priority-driven = 急診室檢傷分類**：每個病人有「緊急程度（優先權）」，最急的先看；只要有醫生空著又有病人等，**醫生絕不發呆**（這就是 greedy）。而且只在「有新病人來」或「看完一個」時才重新決定下一個看誰。
>
> 一句話：**Clock=照表做、Round-robin=輪流玩、Priority=最急先做且絕不閒置。**

### Weighted Round-Robin 重點
- 每個 job 排 FIFO，每次跑一個 **time slice**，沒做完就排到隊尾等下一輪。
- n 個 job → 每個分到 1/n 處理器時間（又叫 **processor-sharing**）。
- 加權：weight 大的 job 一輪拿較多 slice。
- ⚠️ **不適合有 precedence 的 job 鏈**（每個都被分時 → 一條鏈的 response time 被拖很長）。
- ✅ **適合 pipeline**（如 UNIX pipe）：producer 產一段、successor 就能消費，兩者可並行。

### Priority-driven 排程範例觀念
 
**List scheduling**：把 ready job 依優先權排進 queue，最高優先的先上可用處理器。
 
**常見優先權指派（4 種，常考選擇/填空）**
 
| 規則 | 全名 | 白話 | 比喻 | 依據 |
|------|------|------|------|------|
| **FIFO** | First In First Out | **先來的先做** | 便利商店排隊 | release time |
| **LIFO** | Last In First Out | **後來的先做** | 疊盤子（最後放的先拿）| release time |
| **SETF** | Shortest Execution Time First | **做得快的先做** | 讓只買一瓶水的先結 | execution time |
| **LETF** | Longest Execution Time First | **做得久的先做** | 讓大宗採購的先結 | execution time |
 
> 記法：**FIFO/LIFO 看「誰先到」（release time）；SETF/LETF 看「誰做得快/久」（execution time）。**
 
**Preemptive（可搶佔）vs Non-preemptive（不可搶佔）**
- 可搶佔＝做到一半可被更急的踢下來、等下再回來；不可搶佔＝一開始就要做完才換人。
- **一般情況沒有定論誰比較好**，但有兩個確定的結論：
  - 🔹 **特例（所有 job 同 release time + 忽略搶佔成本）**：**可搶佔的 makespan ≤ 不可搶佔**。因為大家同時來時，可搶佔較靈活、能讓處理器盡量不閒置 → 整批較快做完。（注意：只在「同時來」成立，release 錯開就不保證，見 EDF 反例。）
  - 🔹 **兩處理器**：non-preemptive 的最小 makespan **≤ 4/3 ×** preemptive 的最小 makespan。意思是就算可搶佔較好，不可搶佔最多也只慢 1/3，差距有上限。
> **makespan = 最後一個 job 完成的時間 = 整批全部做完要多久。**
 
### Dynamic vs Static 系統
- **Dynamic（動態）**：ready job 放共同 priority queue，處理器空了就抓隊頭 → job 可在不同處理器間 **migrate（遷移）**。平均反應較快。
- **Static（靜態）**：job 事先**分配綁定**到固定處理器，只有重組/故障時才搬。
- ⚠️ 重點結論：動態平均較快，但**最壞情況可能更差**，且目前**只有靜態系統有可靠的時序驗證技術** → **大多數硬即時系統用 static**。


### 🎯 Effective Release Time / Deadline（有效時間）— 計算重點！
 
**這在幹嘛？** 工作有順序（A 做完 → B 才能做）。但題目給的時間有時會**打架**：規定 B 要等 A，卻又說 B 可以 1 點開始、A 要 2 點才開始 → B 不可能 1 點開始！所以要把時間**修成合理的** = 有效時間。
（「前面的人」= 前驅 predecessor；「後面的人」= 後繼 successor）
 
**規則1：開始時間（release）→ 取「比較晚」的**
- 白話：**你不能比前面的人還早開始**（要等他做完）。
- 前面沒人 → 用自己的開始時間
- 前面有人 → **max(自己的, 前面每個人的有效開始)**
- 🟢 比喻：你想 1 點跑接力，但前一棒 2 點才交棒 → 你只能 **2 點**才跑（取晚的）。

**規則2：截止時間（deadline）→ 取「比較早」的**
- 白話：**你不能比後面的人還晚做完**（他要接你的成果）。
- 後面沒人 → 用自己的截止時間
- 後面有人 → **min(自己的, 後面每個人的有效截止)**
- 🟢 比喻：你以為週五交就好，但同事要接你的報告、他週三就得交 → 你**週三**前就得給他（取早的）。
 
**課本例**：B 開始時間自己寫 1，但前面的 A 是 2 → B 有效開始 = max(1,2) = **2**；B 截止自己寫 12，但後面的人最早要 8 → B 有效截止 = min(12,…,8) = **8**。
 
**算完有什麼好處？（單處理器 + 可搶佔）**
 
- **Q1：「不用管順序」是什麼意思？** 本來排工作要同時盯「時間」和「順序」很煩；算完有效時間後，把工作當成「各做各的」，只看時間、直接用 EDF 排就好。
- **Q2：為什麼能不管順序？** 因為順序已變成時間了——前面的人 deadline 被改早（EDF 自動先排他）、後面的人 release 被改晚（自動不會太早做）→ 順序自動滿足。
- **Q3：排出來順序還是錯怎麼辦？** 只發生在「兩工作有效時間**完全一樣**」被排反時 → **swap 對調位置**。因為時間一樣，前後兩段對誰都來得及，對調後順序對了又沒人 miss。
> 一句話：**算有效時間 → 只看時間用 EDF 排 → 時間相同被排反就 swap。**

### 🎯 EDF 與 LST（兩個重要演算法）

| 演算法 | 優先權依據 | 口訣 |
|--------|-----------|------|
| **EDF（Earliest Deadline First）** | deadline 越早 → 優先權越高 | 誰快到期誰先做 |
| **LST（Least Slack Time）** | slack 越小 → 優先權越高（slack = d − t − 剩餘執行時間）| 誰最沒空檔誰先做 |

**EDF 最佳性（Theorem 4.1）**：在**單處理器 + 可搶佔 + job 不爭用資源**下，**EDF 是最佳的**——只要存在可行排程，EDF 一定排得出來。（證明靠：任何可行排程都能透過 swap 轉成 EDF 排程。）

### ⚠️ EDF / LST 何時「不再」最佳（常考反例！）

**死記框架（背這三行就夠）：**
1. **EDF/LST 最佳條件 = 單 CPU + 可搶佔**
2. **不可搶佔會壞** → 急件來了**抢不了**正在做的 → 急件 miss
3. **多處理器會壞** → EDF 只看 deadline、**大塊頭太晚做** → 大塊頭 miss（改 LST 有時能救，但 LST 多處理器也不保證）

| 失效情況（只有 2 個）| 一句話原因 |
|------|-----------|
| ① 不可搶佔 | 急件抢不了正在做的 → 急件 miss |
| ② 多處理器 | 只看 deadline、大塊頭太晚做 → 大塊頭 miss |
 
> 一句話：**EDF/LST 只在「單 CPU + 可搶佔」最佳；一旦「不可搶佔」或「多處理器」就可能 miss。**

### Graham 異常（Anomaly）— 反直覺，常考
 
**priority-driven 多處理器系統的詭異現象**：照常理「給系統加好料」應該更快，但 Graham 異常說——
 
> **增加處理器、減少執行時間、放寬 precedence（順序限制）→ 反而可能讓某些 job 變更晚完成、甚至 miss deadline！**
 
- 三種「好心反而壞事」的動作：**加 CPU、縮短 e、拿掉順序限制**
- 原因：priority-driven 是 greedy（看當下誰優先就排），改變條件會讓「誰先誰後」整個重排，連鎖效應下反而更糟。
- 這就是最早那份解答 **4.5 那幾題在算的東西**（給你改條件、要你排排程圖看誰 miss）。
> 一句話：**Graham 異常 = priority-driven 多處理器下，加 CPU／減 e／放寬順序，反而可能害 job 更晚完成。**（反直覺，記住「好心做壞事」這三個動作。）
 
> 🆕 **別搞混：Graham 異常 vs EDF 多處理器失效**（兩個不同的事）
>
> | | 在講什麼 | 可搶佔能解嗎 |
> |--|---------|------------|
> | **Graham 異常** | 加 CPU／減 e／放寬順序 → 反而更糟 | 可搶佔會**緩解**，但 greedy+precedence 仍可能出事 |
> | **EDF 多處理器失效** | EDF 在 >1 CPU 不是最佳（J1J2J3 反例）| ❌ **可搶佔也救不了**（那個反例本身就是可搶佔的）|

> 一句話總結 Ch4：**三大方法（clock/round-robin/priority）；priority-driven 是 greedy 永不閒置；有效時間 release 取 max、deadline 取 min；EDF/LST 只在單 CPU 可搶佔最佳，多處理器或不可搶佔會失效；硬即時多用 static。**

### 🆕 練習題 4.1（綜合題：有效時間 + EDF + level priority）⭐
 
**題目（Figure 4P-1）**：
(a) 求有效 release/deadline (b) 求 EDF 排程 (c) 用 level priority（level 越高越優先）排程。
 <img width="476" height="232" alt="image" src="https://github.com/user-attachments/assets/d2b66106-3b82-467b-a51f-e12978955f02" />

**(a) 有效時間**（release 從前往後取 max、deadline 從後往前取 min）
 
| Job | 有效 r | 有效 d |
|-----|-------|-------|
| J1 | 0 | 4 |
| J2 | 1 | 4 |
| J3 | 3 | 5 |
| J4 | 1 | 4 |
| J5 | 3 | 5 |
| J6 | 3 | 10 |
| J7 | 1 | 12 |
| J8 | 3 | 12 |
| J9 | 3 | 12 |
 
**(b) EDF 排程**（用有效時間、可忽略順序、挑 d 最小）
```
J1 → J4 → J2 → J5 → J3 → J6 → J7 → J8 → J9
```
全部趕上有效 deadline → feasible ✅
⚠️ 陷阱：J2(d4) 雖急，但前驅 J4 沒做完不能先排 → 所以 J4 在 J2 前；同理 J5 在 J3 前。
 
**(c) Level priority 排程**（level = 到「無後繼 job」的最長路徑；用**給定 release**、**手動遵守順序**）
 
level 算法（後繼中最大 level + 1）：
 
| level | jobs |
|-------|------|
| 3 | J4 |
| 2 | J1, J7, J5 |
| 1 | J2, J8 |
| 0 | J3, J6, J9 |
 
排程（每刻挑 ready 中 level 最高）：
```
J1 → J4 → J7 → J5 → J2 → J8 → J3 → J6 → J9
```
 
**(b) vs (c) 對照**：
- EDF：J1 J4 J2 J5 J3 J6 J7 J8 J9（看 deadline）
- level：J1 J4 **J7 J5 J2 J8** J3 J6 J9（看路徑長度 → J7、J8 被提前）
> 重點：**同一組 job、不同優先權指派 → 不同排程**。EDF 用有效時間＋忽略順序；level priority 用原始 release＋手動遵守順序（見上方 ⭐ 對照框）。同 level 時挑誰都算對。
