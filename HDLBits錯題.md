# 📚 Verilog / HDLBits 學習總整理(完整版)

---
難度:🔴 看答案才懂 ｜ 🟡 卡很久但自己解出 ｜ 🟢 順順寫出
## 🟢Verilog Language - Vectors - Vector concatenation operator
<img width="1589" height="324" alt="image" src="https://github.com/user-attachments/assets/48eaebea-89a5-4322-82d4-956b96526d9f" />

```verilog
{a, b, c, d, e, f, 2'b11}     // ✅ 正確
{a, b, c, d, e, f, 11}        // ⚠️ 危險!
```

11 沒位寬,Verilog 會把它當成「預設 32-bit 整數」。所以:
- 你以為的 11 = 想要的 2 個 bit(11)
- Verilog 實際當成 32'd11 = 32'b00000000_00000000_00000000_00001011(32-bit!)


## 🟢Verilog Language - Vectors - Vector reversal 1
<img width="582" height="88" alt="image" src="https://github.com/user-attachments/assets/43bf5dbe-1193-47da-8369-7718898a3166" />

for 迴圈不能直接寫在 module 裡,要放在:
- always @(*)(行為描述)
- generate 區塊(合成時展開)

## 🟢Verilog Language - Vectors - Replication operator
<img width="1601" height="214" alt="image" src="https://github.com/user-attachments/assets/d93740aa-7efe-4780-b4dc-63a039e7b34d" />

- **`{`**24{in[7]}, in**`}`** ❌ 語法錯!少一個 **`{`**
- **`{`****`{`**24{in[7]}**`}`**, in**`}`** ✅ 正確,雙層

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

## 🟢 Equality (2-bit A==B 比較,2015 midterm 1k)

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

## 🔴 Gatesv — 相鄰位元 both/any/different(卡超久)

**卡點**
- 被題目「左鄰 / 右鄰」繞暈,左右是取同個 index 去看(i+1 還是 i-1)
- 沒看清楚 **port 位寬**,自己宣告成 `[3:0]` 害後面要補東西繞彎
- 誤以為 out_any 是「跟右邊所有位元」比(其實是「相鄰一個」)

**頓悟點**
- **本質就是「相鄰兩位做運算」**,「左/右」只是題目解釋的講法,程式碼一樣
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
  - 不用管題目寫的左右

---

## 🟡 Mux 9-to-1(16-bit,case + default)

**題目**:9 個 16-bit 輸入(a~i),sel 是 4-bit,選一個輸出到 16-bit 的 out。sel ≥ 9 時 out 全 1。

**卡點**
- 以為 `4'h0`~`4'h8` 是輸入位寬,其實是 **sel 的值**(sel 是 4-bit)
- a, b, c, ..., i 不是 4-bit,**每一個都是 16-bit**(`input [15:0] a, b, c, ...` 是「每個都 16-bit」)
- default 想寫全 1 寫成 `4{1}` → 錯
  - `{N{x}}` 要**雙層大括號 + N 在外**:`{4{1'b1}}`
  - 而且 out 是 16-bit,要重複 **16 次** 不是 4 次

**頓悟點**
- `4'h0` 是 sel 的值(sel 是 4-bit),跟 a~i 位寬無關
- default 全 1 等價寫法:
  ```verilog
  out = {16{1'b1}};            // 重複運算子
  out = 16'hFFFF;              // hex 全 F
  out = 16'b1111_1111_1111_1111;  // 二進位寫死
  out = '1;                    // SystemVerilog 簡潔寫法:'1 = 全 1(自動填滿位寬)
  ```
- default 全 0 等價寫法:
  ```verilog
  out = 16'b0;
  out = 16'h0;
  out = '0;                    // SystemVerilog 簡潔寫法:'0 = 全 0
  ```
- **`'0` / `'1` 是 SystemVerilog 寫法**:
  - `'0` = 全部填 0(自動配合 LHS 位寬)
  - `'1` = 全部填 1(自動配合 LHS 位寬)
  - 不用寫位寬,給 8-bit / 16-bit / 32-bit 都自動對齊
  - 寫起來最簡潔,大型訊號特別好用
- default 輸出什麼**是設計者決定**,不是「一定要 1111」——只要有 default 就不會 latch,題目要求全 1 才寫全 1

---

## 🔴 KMAP 卡諾圖化簡(連串題)

**卡點**
- 不會圈大圈,只會列 minterm
- 漏看大圈,化簡項數太多
- **POS 的 maxterm 規則寫反**:把 SOP 規則套到 POS

**頓悟點**

**SOP vs POS 規則完全相反**(背這個):

| 形式 | 圈什麼 | 0 對應 | 1 對應 | 項內串接 | 項間串接 |
|------|--------|--------|--------|---------|---------|
| **SOP** | 圈 **1** | `~x` | `x` | `&` | `\|` |
| **POS** | 圈 **0** | `x` | `~x` | `\|` | `&` |

- **棋盤格 pattern = XOR**:1 和 0 交錯時,K-map 圈不出大塊,本質是 `a^b^c^d`(odd parity)
- **don't care 全當 1 用**(SOP 時)幫助圈大圈
- HDLBits **不要求最簡**,只檢查邏輯等價 → SOP 列所有 minterm 一定過,但式子長
- 化簡步驟:先找 8 格 → 4 格 → 2 格 → 每個 1 至少被蓋一次,圈可重疊
- **`assign out = a | ~(a | b | ~c)` 範例**:用 De Morgan 展開 = `a + b'c`,有時用 NOR 形式比直接化簡更簡

**K-map Gray code 雷**:
- 橫軸/縱軸是 **Gray code 順序 00, 01, 11, 10**(不是 binary)
- **第 3 欄是 ab=11,第 4 欄是 ab=10** ⚠️
- 從 K-map 抽欄對應到 mux 編號(binary 順序)時最容易混

---

## 🔴 Karnaugh map → 4-to-1 mux 實作(不准用閘)

**卡點**
- 不懂題目要幹嘛(外面 mux 給你,你要算 mux_in[0..3])
- K-map 後兩欄 Gray code 看反(ab=11 vs ab=10)
- 用 `~d` 當條件時,三元方向寫反

**頓悟點**
- **K-map 拆 4 欄,每欄是「ab 固定後 c,d 的子函數」**:
  - mux_in[0] = ab=00 那欄的化簡(對應第 1 欄)
  - mux_in[1] = ab=01 那欄(第 2 欄)
  - mux_in[2] = ab=10 那欄(**第 4 欄!Gray code**)
  - mux_in[3] = ab=11 那欄(**第 3 欄!Gray code**)
- **三元運算子 `? :` 算 2-to-1 mux**,不算邏輯閘 → 題目限制下合法
- `~d ? 0 : c` ≡ `d ? c : 0`(等價,兩個都是 `c & d`)
- 同一邏輯**用 d? 或 ~d? 兩種角度寫,結果一樣**

---

## 🟢 D Latch(D 鎖存器,故意產生 latch)

**卡點**
- 第一次遇到「**故意要 latch**」的題目,跟平常「避免 latch」相反

**頓悟點**

**D Latch vs DFF 差別**:

| | D Latch | DFF |
|--|---------|-----|
| 觸發 | **電平**(level) | **邊緣**(edge) |
| 寫法 | `always @(*)` | `always @(posedge clk)` |
| ena=1 時 | q 跟著 d(透明) | (不適用) |
| ena=0 時 | q 鎖住舊值 | (不適用) |

```verilog
module top_module (input d, input ena, output reg q);
    always @(*) begin
        if (ena) q <= d;   // 沒 else!ena=0 時 q 自然保持 → latch
    end
endmodule
```

- **沒 else 是故意的** → 產生 latch(這次是要的功能,不是 bug)
- Quartus 警告「latch inferred」是預期的,可無視
- 平常組合邏輯沒 else = 不小心 latch(要避免);這題沒 else = 故意 latch(要的)

---

## 🟢 同步 reset + 主動高 + 8-bit DFF + 各種變化

**卡點**
- 把同步 reset 寫成非同步(`@(posedge clk or posedge reset)`)
- 主動高寫成 `if (~reset)` 反相
- `0x34` 不知道是什麼

**頓悟點**

**reset 四種組合**(全部要會):

| reset 類型 | 敏感列表 | if 判斷 |
|-----------|---------|---------|
| **同步 + active-high** | `@(posedge clk)` | `if (reset)` |
| 同步 + active-low | `@(posedge clk)` | `if (~reset)` |
| 非同步 + active-high | `@(posedge clk or posedge reset)` | `if (reset)` |
| 非同步 + active-low | `@(posedge clk or negedge reset)` | `if (~reset)` |

**關鍵字翻譯**:
- **同步**(synchronous)→ 敏感列表**只有 clk**,reset 寫在 if 裡
- **非同步**(async)→ 敏感列表多加 `or posedge/negedge reset`
- **主動高**(active-high)→ `if (reset)`(不加 `~`)
- **主動低**(active-low)→ `if (~reset)`

**reset 重置值不是 0**:
- `0x34`(C 語法)= **hex 34** = `8'h34`(Verilog 寫法)= `8'b00110100` = `8'd52`
- Verilog **不認 `0x` 前綴**,要寫 `8'h34`

---

## 🟡 byteena(byte enable)16-bit DFF

**卡點**
- 不懂 byteena 在幹嘛
- 不知道時序邏輯沒 else 不會 latch

**頓悟點**
- **byte enable**:每個 bit 控制一個位元組要不要更新
  - `byteena[1]` 控制 d[15:8](上位元組)
  - `byteena[0]` 控制 d[7:0](下位元組)
  - 該位=1 → 更新、=0 → 鎖住保持
- 寫法用「**if 沒 else**」實現「開關關閉時保持」:
  ```verilog
  always @(posedge clk) begin
      if (byteena[1]) q[15:8] <= d[15:8];   // 沒 else,關閉時 q[15:8] 不動
      if (byteena[0]) q[7:0]  <= d[7:0];
  end
  ```
- **時序邏輯沒 else ≠ latch**,而是 flip-flop 保持原值(複習舊觀念)
- 實際應用:CPU 寫入記憶體時只改部分 byte

---

## 🔴 100-bit ripple carry(自己用 generate)

**卡點**(同一個 ripple carry 觀念又踩 5 次語法雷)
1. `carry[0] = cin;` 沒加 assign(module 內賦值必須用 assign 或 always)
2. `for(...):add_loop` 標籤位置錯 → 應該 `for (...) begin : add_loop ... end`
3. 迴圈從 `i = 1` 開始 → 漏 bit0,應該 `i = 0`
4. `.cout(carry[i-1])` 方向反 → 應該 `carry[i+1]`(進位往上傳)
5. `assign carry[100] = cout;` 方向反 → 應該 `assign cout = carry[100];`

**頓悟點**
- 用三元運算子處理邊界(替代 assign carry[0]=cin 和最後一條):
  ```verilog
  .cin(  i == 0 ? cin  : carry[i]),
  .cout( i == N-1 ? cout : carry[i+1])
  ```
  迴圈內統一寫法,**省掉外面兩條 assign**,寫得更乾淨
- **觀念懂(會用 generate、拆 add1、carry 線)和語法熟練度是兩件事**,初學會犯細節錯,多寫就熟

---

## 🟢 Adder & signed overflow

**卡點**
- 不知道 overflow 怎麼判斷,亂寫 `a[7]+b[7]>1`、`a[6]+b[6]>1`
- 不知道 8-bit signed 怎麼還原(`90` 算成 -113)

**頓悟點**

**Signed overflow 判斷(考試重點)**:
```verilog
assign overflow = (a[7] == b[7]) && (s[7] != a[7]);
// 同號相加 + 結果反號 → overflow
```
- **必須看 s[7]**!沒看結果無法判斷
- 規則:**同號相加結果變反號 → overflow**
  - 正+正=負 ✓ overflow
  - 負+負=正 ✓ overflow
  - 一正一負 → 永不 overflow

**二補數還原**(8-bit signed):

```
signed = unsigned - 2^N(N = bit 數)
```

- 4-bit:減 **16**(2^4)
- 8-bit:減 **256**(2^8)
- 16-bit:減 **65536**(2^16)

例:`90`(hex)= unsigned 144,signed = 144 - 256 = **-112**

**快速判正負**:hex 第一字 ≥ 8 → 負,< 8 → 正
- `7f`、`70` 第一字 < 8 → 正
- `80`、`90`、`ff` ≥ 8 → 負
- **全 1 = -1**(`ff`、`ffff`、`ffffffff` 都是 -1,記起來)

---

## 🟢 Testbench 動態次數(repeat)

**卡點**
- 想用 `{i{0}}` 重複 i 個 0,N 是變數不行

**頓悟點**

| 寫法 | N 可變數? | 用途 |
|------|-----------|------|
| `{N{x}}` 拼接 | ❌ 必須常數 | **電路**位寬要固定 |
| `repeat (N)` | ✅ 變數 OK | **執行**重複 N 次 |
| `for (...)` | ✅ | 更彈性 |

- **`{}` 是結構(電路位寬)、`repeat`/`for` 是行為(動作幾次)**,別混
- testbench 印 N 個值用 `repeat (N) $display(...);`
- 印 `$time` 用 **`%0g`**(避免醜空格),印變數用 **`%0d`**

---

## 🟢 多 module 撞名(`top_module` 在多題都叫一樣)

**卡點**
- HDLBits 用其他題的 module(如 mt2015_q4a, q4b)組合,**那些題的 module 都叫 `top_module`**
- 三個 `top_module` 同檔 → 撞名報錯

**頓悟點**
- **複製貼進來後手動改名**:把貼進來的副本 `module top_module` 那行改成 `mod_a`、`mod_b`
- 原題 HDLBits 不用改(也改不了),改的是你貼進編輯框的副本
- 然後主 `top_module` 實例化改名後的 `mod_a`/`mod_b`

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

## 🔧 數字進位前綴(各種寫法)

**C/Python 前綴**:
- (無) = 十進位
- `0b` = 二進位:`0b110100`
- `0o` = 八進位:`0o64`
- `0x` = 十六進位:`0x34`

**Verilog 寫法**(不用 `0x`!):
```
位寬'進位 數值
```
- `'b` 二進位:`8'b00110100`
- `'o` 八進位:`8'o64`
- `'d` 十進位:`8'd52`
- `'h` 十六進位:**`8'h34`**(硬體最常用)

**Hex 範圍寫法**:
- 0~9 → `4'h0`~`4'h9`(用 h 或 d 都可,值一樣)
- 10~15 → **`4'hA`~`4'hF`**(一個字母 = 4-bit,合法)
- ⚠️ **`4'h10` 是 16,4-bit 裝不下**!10~15 要用 `4'hA`~`4'hF` 或 `4'd10`~`4'd15`
- 全部統一用 `4'd` 最安全

**SystemVerilog 無位寬常數**(超好用):
- **`'0`** = 全部填 0(自動配合 LHS 位寬,任何寬度都對)
- **`'1`** = 全部填 1(自動配合 LHS 位寬)
- **`'x`** = 全部 x
- **`'z`** = 全部 z
- 例:`out = '1;`(out 16-bit → 自動變 16-bit 全 1;out 32-bit → 自動 32-bit 全 1)
- 比寫 `{16{1'b1}}` 簡潔太多

## 🔧 重複運算子 `{N{x}}`

```verilog
{4{1'b1}} = 4'b1111      // 1 重複 4 次
{16{1'b1}} = 16'hFFFF    // 16-bit 全 1
{3{4'hA}} = 12'hAAA      // 1010 重複 3 次
```

- **格式**:**`{N{內容}}`**——N 在外,內容在內,**雙層大括號**
- N **必須常數**(編譯時決定位寬)
- N **要對應 output 的位寬**(out 是 16-bit 就重複 16 次)
- 內容寫 `1'b1` 明確位寬,別寫 `1`(會被當 32-bit)
- 大型訊號**用 `'0` / `'1`** 更簡潔

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

## 🔧 gate / dataflow / behavioral —「接一條線」也有三層級

| 層級 | 寫法 | 接線範例 |
|------|------|---------|
| gate-level | gate primitive | `buf(out, in);` |
| dataflow | assign | `assign out = in;` |
| behavioral | always | `always @(*) out = in;` |

## 🔧 gate-level vs RTL(設計層級)

- 層級:switch → gate → dataflow(RTL,業界 99%)→ behavioral → HLS
- gate-level 邏輯上更直接但規模不可行、難維護、綁死製程、封死合成器優化
- 結論:用 RTL 思維,心裡知道合成出什麼閘即可,不必手畫電路
- 真值表 = 邏輯式 = 電路圖,會列真值表就能寫 RTL(SOP:輸出為 1 的情況用 & 串、再 | 起來)

## 🔧 K-map 化簡通則

- **SOP**:圈 **1**,1→x, 0→~x,項內 `&`、項間 `|`
- **POS**:圈 **0**,**0→x, 1→~x**(跟 SOP 相反!),項內 `|`、項間 `&`
- 圈大小必須是 2 的次方(1, 2, 4, 8, 16)
- 可繞行(左右、上下相連)
- 圈越大化簡越多;**先找 8 格 → 4 格 → 2 格**
- don't care 全當 1 用(SOP)/ 當 0 用(POS),圈大圈
- **棋盤格 pattern = XOR**(odd parity),K-map 圈不出來,本質是 `^`
- K-map 軸**是 Gray code 順序 00,01,11,10**,後兩欄容易看反

## 🔧 DFF / Latch / reset 完整對照

**D Latch(電平觸發)**:
```verilog
always @(*) if (ena) q <= d;     // 沒 else = latch 行為
```

**DFF(邊緣觸發)**:
```verilog
always @(posedge clk) q <= d;
```

**DFF + 同步 + active-high reset**:
```verilog
always @(posedge clk)
    if (reset) q <= '0;
    else       q <= d;
```

**DFF + 非同步 + active-high reset**:
```verilog
always @(posedge clk or posedge reset)
    if (reset) q <= '0;
    else       q <= d;
```

**reset 重置值不一定是 0**:可以 `q <= 8'h34;` 等任意值
**負邊觸發**:`@(negedge clk)` 代替 `@(posedge clk)`

## 🔧 signed overflow(8-bit)

- 判斷:`(a[7] == b[7]) && (s[7] != a[7])`(同號相加 + 結果反號)
- 二補數還原:**signed = unsigned − 2^N**
- 8-bit 範圍 -128~127,16-bit -32768~32767

## 🔧 波形圖讀法(Yours vs Ref + Mismatch)

- 4 區:Inputs / Yours / Ref / Mismatch
- **Mismatch 凸起 = 你跟 Ref 不同;平的 = 一樣 ✅**
- 逐欄(同一時段)對照 Yours 和 Ref 找差異
- DFF 看 **clk 上升緣**那一刻 d 的值決定 q

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
11. **沒看 port 位寬**:HDLBits 有些題用縮減位寬(`[3:1]`、`[2:0]`)處理邊界
12. **K-map 軸是 Gray code(00,01,11,10)**,後兩欄(ab=11 vs 10)容易看反
13. **`4'h10` 不存在**(4-bit 裝不下 16)→ 10~15 用 `4'hA`~`4'hF` 或 `4'd10`~`4'd15`
14. **`0x34` 是 C 語法,Verilog 要寫 `8'h34`**
15. **`{N{x}}` 的 N 必須是常數**,要動態重複用 `repeat (N)`
16. **同步 vs 非同步 reset**:差在敏感列表有沒有 `or posedge reset`
17. **主動高 vs 主動低**:`if (reset)` vs `if (~reset)`,別反相搞錯
18. **多 module 同名(top_module)→ 改名再貼**(`mod_a`、`mod_b`)
19. **POS 規則跟 SOP 完全相反**(0→x、1→~x、項內 `|`)
20. **signed overflow 必須看 s[7]**,光看輸入位元抓不到
21. **D Latch 故意沒 else**(平常避免 latch、但這題要 latch)
22. **時序邏輯沒 else ≠ latch**(=保持原值,正常)
23. **gate primitive 只接訊號**,不能放 `a&b` 這種運算式
24. **`'0` / `'1`**(SystemVerilog)= 自動填滿位寬全 0/1,比 `{N{1'b1}}` 簡潔
