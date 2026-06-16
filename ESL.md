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

