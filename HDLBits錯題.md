# 📚 Verilog / HDLBits 學習總整理

> 本筆記整理自學習過程中問過的所有題目與觀念
> 過關標準:不看筆記、不問人,自己從零寫出來 ✅
> 難度:🔴 看答案才懂 ｜ 🟡 卡很久但自己解出 ｜ 🟢 順順寫出

---

# 一、做過的題目

## 🟡 Adder100in (32-bit / 16-bit ripple carry adder)

**題目**:用 add16(內含 16 個 add1)組成 32-bit 加法器

**卡點**
- 一開始連 module 怎麼拆都不懂
- `add16` 自己也寫了 → 報錯 "module cannot be declared more than once"

**頓悟點**
- 結構:top → 2 個 add16 → 每個 16 個 add1,像積木層層疊
- 看到題目寫 "you are given a module add16" → 那個不用自己寫,寫了反而衝突
- `add1` 全加器公式:`sum = a^b^cin`、`cout = (a&b) | (cin&(a^b))` 或 `(a&b)|(a&cin)|(b&cin)`
- `.cin(0)` 建議寫 `.cin(1'b0)`(明確 1-bit)

---

## 🟢 Mux2to1 / 條件選擇 (sel_b1 & sel_b2)

**題目**:assign 版 + always 版各做一個 2-to-1 選擇

**頓悟點**
- 判斷用邏輯符號 `&&`,1-bit 訊號意圖明確
- `assign out = (sel_b1 && sel_b2) ? b : a;`
- always 版要 `output reg`,組合邏輯用 `=` 不用 `<=`
- 同一邏輯可用三元 / if-else 兩種寫法表達

---

## 🟢 簡單條件輸出 (cpu_overheated / keep_driving)

**卡點**:`keep_driving = gas_tank_empty;` 邏輯錯

**頓悟點**
- 到達目的地後,不管油箱狀態都不該繼續開 → else 那條應該是常數,不是 `gas_tank_empty`
- 想清楚「這個情況下答案是不是固定值」

---

## 🟡 Vector reversal (位元反向)

**頓悟點**
- 短向量:`assign out = {in[0], in[1], ... in[7]};`(concatenation)
- 長向量:generate for + `assign out[i] = in[N-1-i];`
- 核心公式:`out[i] = in[N-1-i]`

---

## 🟡 Popcount255 (數有幾個 1)

**卡點**
- `out` 沒宣告 reg、組合邏輯誤用 `<=`
- `out` 沒初始化 → 產生 latch、累加錯誤
- 寫成 `out = out + 1`(無條件加)→ 結果永遠是 255

**頓悟點**
- `always @(*)` 開頭先 `out = 0`,否則 latch
- for 迴圈只是「重複動作」,判斷邏輯要寫進去
- `out = out + in[i]`(Verilog 把 1-bit 當數值,是 1 加 1、是 0 加 0)比 `if` 簡潔
- 進階:`$countones(in)` 或全部 `in[0]+in[1]+...` 相加

---

## 🟡 Parity (奇偶校驗)

**頓悟點**
- Even parity bit = 所有資料位的 XOR
- `assign parity = ^in;`(reduction XOR 一行搞定)
- XOR 性質:成對的 1 互相消掉,剩下的就是「奇數個 1」
- reduction:`&a` 全 1、`|a` 不為 0、`^a` 奇偶性;前面加 `~` 變 NAND/NOR/XNOR

---

## 🟡 Adder100 (100-bit ripple carry,自己寫 full_adder)

**卡點**
- 第 0 個全加器 `.cin()` 留空沒接 → 結果全錯
- 實例名跟 module 同名 `full_adder full_adder(...)`

**頓悟點**
- 第 0 個 `.cin(cin)` 接外部進位,其他接 `cout[i-1]`
- 實例名取不同名(如 `fa`),別跟 module 撞名
- 這題 cout 是 100-bit → 可直接借 output cout 當進位線,不用另開 wire
- `cout = ((a^b)&cin) | (a&b)` 建議加括號讓優先順序明確

---

## 🔴 Bcdadd100 — BCD 100 位加法器

**卡點**
- 以為 BCD 就是普通二進位 → 錯!BCD 每位只能 0~9
- `a[i*4 +: 4]` 切片語法不會讀
- 不懂為什麼 `bcd_fadd` 要做「修正」

**頓悟點**
- BCD `7+5`:純二進位 = `1100`(12),但 BCD 不能輸出 12 → 修正成 `sum=0010`(2)、`cout=1`,跟小學「寫 2 進 1」一樣
- 進位條件是「**超過 9**」,不是「4-bit 用盡(超過 15)」
- `a[i*4 +: 4]` = 從 bit `i*4` 開始往上取 4 個 bit(Verilog 不准 `a[變數:變數]`,所以用 `+:` 起點可變、寬度固定)
- `bcd_fadd` 是黑盒子,修正它處理好了,我只要串接力

---

### 🟢 Equality (2-bit A==B 比較,2015 midterm 1k)
 
**卡點**
- 第一版寫 `assign z = ~(A ^ B);` → 錯(`~(A^B)` 是 2-bit,z 只 1-bit,被截掉只剩最低位 → A=10,B=00 誤判 z=1)
**頓悟點**
- 逐位運算結果是多 bit,但 z 只要 1-bit → **少了「把兩位比較結果合併」那一步**
- 三種解法:
  1. `assign z = &(~(A ^ B));` ← reduction AND:每位相同?→ 全部都相同?(我用這個)
  2. `assign z = (A == B);` ← 最簡單,== 比較整個向量,輸出天生 1-bit
  3. `assign z = (A[1]==B[1]) && (A[0]==B[0]);` ← 逐位展開
- 通用觀念:位元運算(`~`/`^`)結果多 bit,要收成 1-bit 判斷得用 reduction 或 `==`

---

### 🔴 Gatesv — 相鄰位元 both/any/different(卡超久)
 
**卡點**
- 被題目「左鄰 / 右鄰」繞暈,一直搞錯方向(i+1 還是 i-1)
- 沒看清楚 **port 位寬**,自己宣告成 `[3:0]` 害後面要補東西繞彎
- 誤以為 out_any 是「跟右邊所有位元」比(其實是「相鄰一個」)
**頓悟點**
- **本質就是「相鄰兩位做運算」**,「左/右」只是題目解釋的講法,程式碼一樣
- 三個輸出**切法完全相同**,只差運算子:`&`(both)/ `|`(any)/ `^`(different)
- **port 位寬要照官方**:`out_both [2:0]`、`out_any [3:1]`、`out_different [3:0]` ← 題目把「沒鄰居那位」直接從 port 拿掉,所以題目沒寫錯
- out_different 唯一特殊:要**繞行**(最高位鄰居接回 in[0])
- 三種解法(各 3 行,效果相同):

  **① 切片(官方最簡潔,用縮減 port)**
  ```verilog
  assign out_both      = in[3:1] & in[2:0];
  assign out_any       = in[3:1] | in[2:0];
  assign out_different = in ^ {in[0], in[3:1]};
  ```
 
  **② 移位(用滿 port [3:0],邊界位算垃圾但不檢查)**
  ```verilog
  assign out_both      = (in >> 1) & in;
  assign out_any       = (in >> 1) | in;
  assign out_different = in ^ {in[0], in[3:1]};   // 繞行不能用移位
  ```
 
  **③ 拼接(用滿 port [3:0])**
  ```verilog
  assign out_both      = in & {1'b0, in[3:1]};
  assign out_any       = in | {1'b0, in[3:1]};
  assign out_different = in ^ {in[0], in[3:1]};
  ```
 
  **重點**:
  -「滿位要考慮方向、沒滿位不用」
  - `in >> 1` ≡ `{1'b0, in[3:1]}`(移位補 0 = 拼接補 0,完全等價)
  - **out_different 三版都一樣**:繞行(最高位接回 in[0])只能用拼接 `{in[0],in[3:1]}`,移位只會補 0 不會繞
  - 版本①用官方縮減 port(`[2:0]`/`[3:1]`)最乾淨;②③用滿 port `[3:0]`,沒鄰居那位算垃圾值但測試不檢查
- 通用觀念:**HDLBits 有些題用「縮減的 port 位寬」處理邊界,務必看清楚 module 宣告**;「相鄰位元」題=錯開一位的兩段做運算

---

# 二、核心觀念整理

## 🔧 賦值與 latch

- **`<=` vs `=`**:時序邏輯(`always @(posedge clk)`)用 `<=`;組合邏輯(`always @(*)`)用 `=`
- **單一賦值時**兩者合成結果相同;**多賦值時**完全不同(blocking 循序、non-blocking 同時更新)
- **latch 怎麼來**:`always @(*)` 有路徑沒賦值 → latch
- **時序邏輯沒 else 是正常的**:會合成 flip-flop with enable(保持原值),不是 latch
- **避免 latch 標準技巧**:`always @(*)` 進 case/if 前,先給所有 output 預設值

## 🔧 case 系列

- **執行規則**:三者都是「從上往下,第一個匹配就停」
- **don't care**:`case` 不支援;`casez` 用 `?`/`z`;`casex` 用 `?`/`z`/`x`
- **危險**:訊號是 `x`(沒初始化)時,`casex` 所有 pattern 都匹配 → 停在第一條,bug 被掩蓋
- **結論**:用 `casez` + `?`,永遠別用 `casex`
- priority encoder 經典用 `casez`

## 🔧 運算子

- **邏輯 vs 位元**:`&&`/`||`/`!`(輸出 1-bit,判斷用)vs `&`/`|`/`~`(逐位,位元操作用)
- 1-bit 時兩者結果相同,但用對符號意圖清楚、改位寬不會炸、過 lint
- **判斷一律用邏輯符號**(`&&`、`||`、`!`、`==`、`!=`)
- **reduction(單元)**:`&a`、`|a`、`^a` 一行處理整個向量
- **XOR = 可程式化反相器**:`ctrl ^ data`,ctrl=1 反轉、ctrl=0 不變(加減法器用)
- **1-bit 控制多 bit**:`{32{sub}} ^ b`(replication + XOR)

## 🔧 位寬與數值

- 不同位寬可混合運算,窄的自動補 0 擴展
- **1-bit + 1 = wrap**:`1'b1 + 1'b1 = 1'b0`(進位被截掉),等同 `~a`
- wrap around 是 feature → 二補數負數:`-1` 在 4-bit = `1111`、32-bit = `FFFFFFFF`
- **part-select**:`a[i*4 +: 4]` = 起點 `i*4` 往上取 4 個(變數索引唯一寫法,因為不准 `a[變數:變數]`)
- 加法位寬要夠,否則溢位 wrap

## 🔧 宣告型別

- **`[大:小]`(如 `[15:0]`)**:左 MSB、右 LSB,符合二進位直覺,99% 用這個
- **`[小:大]`(如 `[0:15]`)**:很少用,陣列才常見
- **陣列**:`reg [7:0] mem [0:1023];`(bit 用 `[大:小]`,陣列用 `[小:大]`)
- 陣列大小一定要寫範圍,不能只寫數字;`mem[1024]` 只有 SystemVerilog 才行
- **testbench 型別**:input 用 `reg`(自己驅動),output 用 `wire`(被 DUT 驅動);DUT 內則相反
- wire 可隱式宣告但別省,建議 `` `default_nettype none `` 強制宣告(抓錯字、位寬截斷)

## 🔧 module 與實例化

- **assign**:在當前 module 描述邏輯運算
- **實例化**:拿現成 module 來用,`模組名 實例名 (.port(signal))`
- **實例化本身就是有效電路**,不需額外 assign/always 啟動
- **每條 wire 要恰好一個 driver**(assign / 實例 output / always 三選一),不能重複也不能沒有
- **named connection**(`.port(signal)`)優於 positional(按順序),不易接錯、改 port 順序不壞
- output 不用特別 assign 才輸出;裡面有人驅動,外面就看得到;已被驅動就別再 assign(multiple drivers)

## 🔧 parameter / localparam

- 位置:module 內訊號宣告區,或 `#( )` 內(`#( )` 只能放 parameter)
- **parameter**:可從外部 override(可調參數,如位寬)
- **localparam**:不可外部修改(內部固定常數,如 FSM 狀態編號)
- 能用 localparam 就用,保護內部邏輯
- 想用 parameter 決定 port 位寬 → 必須寫在 `#( )` 裡(順序問題)
- 學習階段寫 module 內即可;做可重用 IP 才需要 `#( )`

## 🔧 begin 區塊標籤

- begin...end 預設沒名字;**裡面要宣告變數就必須加標籤**(`begin : label`)
- 不宣告變數則標籤可有可無
- SystemVerilog 可 `for (int i...)` 直接宣告,免標籤

## 🔧 function / task

- **function**:只 1 個回傳值(透過函數名),只能 input,`y = func(...)`,0 時間,不能有延遲
- **task**:可多個 output,input/output/inout 都行,`task(...)`,可消耗時間,可有延遲
- function 的 output = 函數名本身,位寬寫在名字前(`function [3:0] min_fun;`),內部對函數名賦值
- RTL 多用 function(組合邏輯);task 多用在 testbench

## 🔧 testbench 與時間

- **`timescale 1ns/1ps`**:`#N` = N 個 ns,模擬精度到 1 ps;精度太粗會四捨五入
- **多個 `initial` 並行**:時間 0 同時開跑、各跑各的(不是循序)
- 典型分工:stimulus(產生輸入)/ `$monitor`(監看)/ `$finish`(timeout 保險絲)各一個 initial
- **`$monitor`**:值有變就印,整個 tb 只能一個有效,要在訊號變化前(獨立 initial)啟動
- `$display` 印一次、`$monitor` 持續、現代多用 VCD 波形(`$dumpfile`/`$dumpvars`)
- **格式**:`%g` 印 `$time`(無醜空格);`%d` 印變數(`%0d` 緊貼);`%b` 二進位、`%h` 十六進位

## 🔧 gate-level vs RTL(設計層級)

- 層級:switch → gate → dataflow(RTL,業界 99%)→ behavioral → HLS
- gate-level 邏輯上更直接但規模不可行、難維護、綁死製程、封死合成器優化
- 結論:用 RTL 思維,心裡知道合成出什麼閘即可,不必手畫電路
- 真值表 = 邏輯式 = 電路圖,會列真值表就能寫 RTL(SOP:輸出為 1 的情況用 & 串、再 | 起來)

## 🔧 跑馬燈 / 計數器分頻(範例電路)

- Counter:每 clk 加 1,數到 MAX 歸 0(像碼表,一直動)
- State:平常不動,只在 `cnt == MAX` 後的 clk 邊緣旋轉一次(one-hot `{state[2:0],state[3]}`)
- always 看的是「當下值」不是「下一個值」→ cnt==MAX 那刻 state 還沒動,下個邊緣才動
- Counter 當分頻器,把高速 clk 變成人眼可見速度(MAX_CNT 決定快慢)
- 效果:光點輪流亮,不會閃爍/全暗,直接「跳」到下一顆

---

# 三、最常踩雷 Top 速查

1. 組合邏輯忘記初始化 / 沒涵蓋所有路徑 → **latch**
2. `always @(*)` 用 `<=`、`always @(posedge clk)` 用 `=` → 慣例錯
3. `casex` 害人 → 一律 `casez` + `?`
4. 實例化第 0 個進位忘了接外部 cin
5. `output reg` 忘了寫(always 驅動時)
6. 變數索引切片寫成 `a[i+3:i]` → 要用 `a[i*4 +: 4]`
7. 實例名跟 module 同名
8. wire 沒人驅動(`z`)或被重複驅動(multiple drivers)
9. positional connection 接錯腳位
10. BCD ≠ 二進位(超過 9 就進位)

### 🔧 gate / dataflow / behavioral —「接一條線」也有三層級
 
| 層級 | 寫法 | 接線範例 |
|------|------|---------|
| gate-level | gate primitive | `buf(out, in);` |
| dataflow | assign | `assign out = in;` |
| behavioral | always | `always @(*) out = in;` |
