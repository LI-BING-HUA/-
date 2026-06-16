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
