# ESL重點

## Moore's Law（經濟預測定律）

The number of transistors on a chip roughly doubles every 18 months…

while the chip price remains the same

## Traditional Abstract Levels

<img width="407" height="223" alt="image" src="https://github.com/user-attachments/assets/2dcde310-4e4b-4f1f-9b37-44a82015b54b" />

## Terminology…

### 1. Instruction Accurate Simulation（指令精確）
The ISS executes instructions in the correct order with correct data to produce correct results, but gives **no timing information at all**.

### 2. Cycle Count Simulation（週期計數）
Each instruction is profiled with an execution latency in clock cycles, then in instruction-accurate order the execution times are **accumulated** to estimate total time.

### 3. Cycle Callable Simulation（可逐週期呼叫）
The simulator is invoked **on every clock cycle**.

### 4. Cycle Accurate Simulation（週期精確）
The execution time is accurate **to the end of each clock cycle**.

## 計算範例：strcopy 迴圈的 Cycle Count vs Cycle Accurate
### 程式在做什麼（字串複製迴圈）
 
```
4. LDRB r2, [r1],#1   ; 從來源讀一個 byte 進 r2
5. STRB r2, [r0],#1   ; 把 r2 寫到目的地
6. CMP  r2, #0        ; 檢查是不是 0（字串結尾）
7. BNE  strcopy       ; r2≠0 就跳回第 4 行繼續
8. MOV  pc, lr        ; r2=0（複製完）就返回
```
執行序列：`(4,5,6,7)(4,5,6,7)(4,5,6,7), 8`
### 投影片給的延遲表
LD（LDRB）= 2、ST（STRB）= 3、BNE = 2、**其他沒講到的 = 1**（CMP=1、MOV=1）

### Cycle Count（死板加總，用固定延遲）
一圈 = LDRB + STRB + CMP + BNE = 2 + 3 + 1 + 2 = 8
 
```
(2+3+1+2) + (2+3+1+2) + (2+3+1+2) + 1 = 8+8+8+1 = 25
```
（三個括號=三圈，最後 +1 = 返回的 MOV）→ **25 cycles**

### Cycle Accurate（考慮管線重疊 + 分支預測）
| 圈 | 序列 | 多抓的行→猜 | 實際(r2) | 對錯 | BNE |
|----|------|-----------|---------|------|-----|
| 1 | (4,5,6,7,8) | 抓8→猜不跳（第一次預設） | r2≠0 該跳 | 錯 | 2 |
| 2 | (4,5,6,7) | （無→4）猜跳 | r2≠0 該跳 | 對 | 1 |
| 3 | (4,5,6,7,4) | 抓4→猜跳 | r2=0 不跳 | 錯 | 2 |
 
口訣：**多出來那一行 = 猜錯被丟掉的證據；沒多餘 = 猜對。**
每圈用 LDRB=2、STRB=1、CMP=1、BNE=(2/1/2)：
```
(2+1+1+2) + (2+1+1+1) + (2+1+1+2) + 1 = 6+5+6+1 = 18
```
→ **18 cycles**

## What Is A Transaction?（transaction 的定義與階層）
### 一句話總結（考點）
**Bus transaction = addressed data read/write，是最基本的溝通單位；多個 bus transaction 可階層式組合成更高層的 task，故 transaction 是 hierarchical。**

## Reasons for Using TLM（使用 TLM 的理由）
速度數字（易考陷阱）：**ESL（TLM）至少比 RTL 快 1000 倍，但不能比真正硬體慢超過 1000 倍**。意義 = 夠快能跑軟體、又夠早能用，用軟體來驗證硬體。

<img width="407" height="229" alt="image" src="https://github.com/user-attachments/assets/2b92e2a8-f1b3-418d-a370-44bb97be40a3" />

## TLM Formulation – Component Basics
| 組成 | 功能 | 細分 |
|------|------|------|
| **Data structure** | 元件記住的資料 | control register（控制暫存器）、state registers（狀態暫存器）、variables、constants |
| **FSM with computational operations** | 行為/運算（大腦） | Mealy machine，每個 state 寫運算敘述；更新 data structure 但**不改 constants** |
| **Port** | 資料溝通接口 | **Master**＝FSM 主動發出 bus transaction；**Slave**＝依收到的 transaction 讀/寫暫存器 |
| **Interrupt** | 事件通知接口 | **Input**＝把 FSM 帶到特定 ISR state；**Output**＝FSM 透過它發出中斷 |
 
記法：Master 主動發、Slave 被動收。
<img width="859" height="522" alt="image" src="https://github.com/user-attachments/assets/9d2f1e1b-88e2-4293-8d55-da417f3cb278" />

## Use Cases, Coding Styles and Mechanisms
<img width="407" height="222" alt="image" src="https://github.com/user-attachments/assets/3b0c3c83-6d9b-43f0-8bc5-a26fe5bb9d85" />

## TLM-2 Core Interfaces - Transport
<img width="442" height="266" alt="image" src="https://github.com/user-attachments/assets/60b2e4f3-ce80-44c4-a8b1-d1aeed2c04ec" />

## Model of Computation
<img width="419" height="176" alt="image" src="https://github.com/user-attachments/assets/e5d21f89-30f3-4dc2-a94d-ab3da8ffbf3a" />

### Core Points

MoC (Model of Computation) = the mathematical foundation of computing devices.

MoC is also called the Theory of Computation — this "alias" is the most common true/false trap.

MoC seeks to understand three things: 

(1) what problems computers can solve, 

(2) how efficiently they can be solved, and 

(3) the inherent limitations of computation.

Origin: scientists have tried to build a machine to automate mathematical computing since the early 20th century.

Development path: questioning the nature of mathematical reasoning and formal systems → breakthroughs in logic + design of abstract machines → modern computational complexity theory.

## Church-Turing Thesis
<img width="439" height="184" alt="image" src="https://github.com/user-attachments/assets/97594f7b-2009-47e6-9227-fc4d536b7e25" />

## A Multi-Core, Multi-Thread Application
<img width="369" height="248" alt="image" src="https://github.com/user-attachments/assets/a42f4662-09dd-48f9-a403-3e306db45cd9" />

順序是 : 

1. USB綠色箭頭到API
2. 然後API再進RAM綠色箭頭
3. 接著OS從ARM把東西送到DSP Core
4. 產生ISR AP2藍色箭頭流程
5. 然後RAM指向AP2
6. 最後AP2再到LCD Disolay綠色箭頭

## A Race Condition Bug
<img width="367" height="247" alt="image" src="https://github.com/user-attachments/assets/9ec2b514-562d-450f-b76c-adea2a2e497d" />

### Race Condition（事件順序 bug）
- **Definition**: concurrent actions (ARM write, DSP read) share RAM; result depends on timing. No timing guarantee → reader gets stale data.
- **Conditions for it to happen**: shared resource + concurrency + write-before-read dependency + no sync (with delay) → read happens before data arrives.
- **ICM is the key**: ICM bridges AHB0/AHB1. ARM's cross-core write passes through ICM; its delay makes data arrive late, so DSP reads the old value → AP1→AP2 broken
- **Fix**: synchronization guarantees write-arrived before read (e.g., DSP reads only after write-complete Interrupt, or a handshake/flag). Synchronization coordinates the timing/ordering of concurrent parties so dependent operations run in the correct order.

## Parallel Computing
<img width="413" height="31" alt="image" src="https://github.com/user-attachments/assets/73e89478-a649-4e7f-8abe-4424363aec6e" />

## Example Parallel Architectures
<img width="173" height="142" alt="image" src="https://github.com/user-attachments/assets/62c11f53-aab5-4b2b-bf73-3fe839b69fb6" />
<img width="567" height="419" alt="image" src="https://github.com/user-attachments/assets/e3589aa6-89c3-41ca-b9b3-9095a280c8d0" />

## Distributed Computing
<img width="419" height="47" alt="image" src="https://github.com/user-attachments/assets/c1a2da68-8244-471f-b142-74b7bc7302d8" />

## Cheating Husbands Puzzle
<img width="473" height="221" alt="image" src="https://github.com/user-attachments/assets/03e1c4e9-42a7-4f25-84be-6fceedd1aa1a" />

## Before ESL Verification
<img width="456" height="196" alt="image" src="https://github.com/user-attachments/assets/dc491619-f41d-4a23-9a4d-80c4a28bcf53" />

## System Design Complexity II
<img width="1411" height="754" alt="image" src="https://github.com/user-attachments/assets/b8d97752-30d7-4f13-b57c-812ae3012803" />
<img width="871" height="825" alt="image" src="https://github.com/user-attachments/assets/5da6a717-c08f-46c0-a9b9-d9b94ddfcaa6" />

## Quick Review 2-Way Set Associative Cache
<img width="472" height="290" alt="image" src="https://github.com/user-attachments/assets/60dcad22-96d3-4b3e-bd2e-c7b291951563" />

## System Design Flow
<img width="407" height="238" alt="image" src="https://github.com/user-attachments/assets/4a9e48b9-1e50-4ae4-89ce-bf91a3d0925f" />

### HW ↔ SW/FW 之間的雙箭頭
Issue: HW and SW/FW implementation must be co-designed and constantly coordinated; in a traditional flow they're developed separately and only integrated late, so HW/SW partition and interface mismatches surface too late.

How ESL solves it: ESL provides a shared virtual platform (TLM) so HW and SW can be developed in parallel and integrated/verified early, instead of waiting until the end.

### 左邊那條藍色拱形大箭頭
Issue: The arched arrow represents the problem of verifying that the final integrated system stays consistent with the original algorithm design. In a traditional flow you only find out after integration is complete, and drift is hard to trace back.

How ESL solves it: ESL builds a golden model (the TLM / virtual platform) that exists from the algorithm stage onward. Because this model is ready early and fast enough, every later stage (architecture, HW/SW implementation, integration) can be checked against it, ensuring the result stays faithful to the original algorithm. The final integrated system can be validated directly against the golden model, so deviations are caught early instead of at the very end.

## Kahn Process Network
<img width="463" height="199" alt="image" src="https://github.com/user-attachments/assets/cfd68afb-1910-4ba3-a1f6-2dcd8d5681d7" />
<img width="583" height="503" alt="image" src="https://github.com/user-attachments/assets/7d1bc906-2912-412e-b33e-161855a660a1" />

## Synchronous Data Flow (SDF)（計算題大戶）
<img width="434" height="266" alt="image" src="https://github.com/user-attachments/assets/6799e854-8580-4ac6-891f-37b7386d68b5" />

### 定義
- **SDF = KPN 的限制版**：每個 node 每次 firing 的 produce/consume 數量**固定** → 可**靜態排程（static scheduling）**。（KPN 不固定、不能靜態排程）
- 圖：**Node(actor)=運算**，**Edge=FIFO queue**；每條 edge 有 **produce rate + consume rate**，可有 **initial data（初始 token，黑點）**。
- 形式：3-tuple **(N, E, E_{p,c,i})**：N 節點、E 邊、p 生產率、c 消耗率、i 初始資料。
### 1. 算每個 node fire 幾次（核心計算）
每條 edge 收支平衡：**(來源 fire 次數 × produce) = (目的 fire 次數 × consume)**。
解聯立方程式得每個 node 的 fire 次數。例：Y 形圖得 A:8、B:3、C:6、D:3、E:6（全平衡 = consistent）。
 
### 2. Consistent vs Inconsistent
- **Consistent**：存在 periodic schedule，跑完一輪所有 FIFO **回到初始狀態**（token 不殘留/不短缺）。例：**ABCC**（A1次B1次C2次）。
- **Inconsistent**：找不到 periodic schedule，token 會累積（例：C fire 兩次後仍剩 1 token 消化不掉）→ no periodic schedule。
### 3. Topology Matrix（判斷一致性的正式方法）
- **列(row)=edge，行(column)=node**；元素 (i,j)=node j 每 fire 一次在 edge i 上放/拿幾個 token。
- **produce 正、consume（輸入通道）負**。
- 範例（A→B、A→C、B→C）：
```
        A    B    C
A→B  [  1   -1    0 ]
A→C  [  2    0   -1 ]
B→C  [  0    2   -1 ]
```
 
### 4. Rank 判斷（必背規則）
> **存在 periodic schedule 的必要條件：topology matrix 的 rank = s − 1**（s = 節點數）。
- rank = s−1 → **consistent**（有週期排程）
- rank = s（≠ s−1）→ **inconsistent**（無週期排程）
- 例：3 節點，rank=2 → consistent；改權重使 rank=3>2 → inconsistent。

## SDF 練習題（含解答）
 
判斷準則：每條 edge「來源 fire 次數 × produce = 目的 fire 次數 × consume」。
解得出共同整數解（不矛盾）→ **consistent**；某 node 被要求 fire 不同次數（矛盾無解）→ **inconsistent**。
（topology matrix：列=edge、行=node，produce 正、consume 負；consistent ⇔ rank = 節點數−1。row 上下順序不影響 rank，但每個數字的「位置+正負」要對。）
 
### 第 1 題
 
<img width="1360" height="360" alt="SDF_q1" src="https://github.com/user-attachments/assets/860c97c6-4d88-4f0b-9487-7852efede7e2" />

方程式：A→B：`2·a = 3·b` → 最小整數解 **a=3, b=2**
schedule = **A A B A B**（A 三次、B 兩次）→ **consistent**
 
### 第 2 題
 
<img width="1360" height="640" alt="SDF_q2" src="https://github.com/user-attachments/assets/93b8b225-5626-4ace-a42c-0962e504cf10" />

邊：A→B(1/1)、A→C(3/1)、B→C(3/1)
方程式：
- A→B：`1·a = 1·b` → a=b
- A→C：`3·a = 1·c` → c=3a
- B→C：`3·b = 1·c` → c=3b（與上一致）
最小解 **a=1, b=1, c=3** → schedule **A B C C C** → **consistent**
 
### 第 3 題（有解但 B 要兩次 → 仍是 consistent）
 
<img width="1360" height="640" alt="SDF_q3" src="https://github.com/user-attachments/assets/4a49f2bb-7652-4b27-b29e-1de988c6d469" />

邊：A→B(2/1)、A→C(1/1)、B→C(1/2)
方程式：
- A→B：`2·a = 1·b` → b=2a
- A→C：`1·a = 1·c` → c=a
- B→C：`1·b = 2·c` → b=2c（與上一致：2a=2c, c=a ✓）
最小解 **a=1, b=2, c=1** → schedule **A B B C** → **consistent**（⚠️ 易錯：B 要兩次很正常，有整數解就是 consistent，不是 inconsistent）
 
Topology matrix：
```
        A    B    C
A→B  [  2   -1    0 ]
A→C  [  1    0   -1 ]
B→C  [  0    1   -2 ]
```
3 節點，rank = 2 = 3−1 → 與 consistent 一致 ✓

## 考古

**Q1**

Q: Moore's Law

A: The number of transistors on a chip roughly doubles every 18 months, while the chip price remains the same.

**Q2 (a)**

Q: Given a Digital Signal Processing (DSP) application as an example, why we say at system design level we can accurately give a cycle accurate design spec?

A: Because a DSP has a fixed sampling rate.

**Q2 (b)**

Q: Please discuss why low power design has to do with memory access analysis and how memory access analysis depends on data flow analysis.

A: On data flow analysis, we use memory access. These occupy 80% in data flow, so we design low power system.

**Q3**

Q: Please explain the following terminologies in respect to Instruction Set Simulator (ISS):

(a) Instruction accurate

A: The ISS executes instructions in the correct order with correct data to produce correct results, but gives **no timing information at all**.

(b) Cycle count

A: Each instruction is profiled with an execution latency in clock cycles, then in instruction-accurate order the execution times are **accumulated** to estimate total time.

(c) Cycle callable

A: The simulator is invoked **on every clock cycle**.

(d) Cycle accurate

A: The execution time is accurate **to the end of each clock cycle**.

**Q6 (a)**
 
Q: What operations does a bus transaction perform? In other words, what is a bus transaction?
 
A: Addressed data read and write.
 
**Q6 (b)**
 
Q: Describe, among all parties involved in executing a task, what 'synchronization' is (one or at most two sentences).
 
A: Synchronization coordinates the timing and ordering of the concurrent parties executing a task, so that dependent operations (e.g., write-before-read) happen in the correct order and race conditions are avoided.
