# 📌 本次新增錯題(貼進「學習總整理」對應位置)

---

## A. 加進「一、做過的題目」(接在現有題目後面)

### 🟡 Mux 9-to-1(16-bit,case+default)

**卡點**
- 以為 `4'h0`~`4'h8` 是輸入的位寬,其實是 **sel 的值**(sel 是 4-bit)
- a, b, c, ..., i 不是 4-bit,**每一個都是 16-bit**(`input [15:0] a, b, c, ...` 是「每個都 16-bit」)
- default 想寫全 1 寫成 `4{1}` → 錯
  - `{N{x}}` 要**雙層大括號 + N 在外**:`{4{1'b1}}`
  - 而且 out 是 16-bit,要重複 **16 次** 不是 4 次

**頓悟點**
- `4'h0` 是 sel 的值(sel 是 4-bit),跟 a~i 位寬無關
- default 全 1 三種等價寫法:`{16{1'b1}}`、`16'hFFFF`、`16'b1111_1111_1111_1111`
- default 輸出什麼**是設計者決定**,不是「一定要 1111」——只要有 default 就不會 latch,題目要求全 1 才寫全 1

---

### 🔴 KMAP 卡諾圖化簡(連串題)

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

### 🔴 Karnaugh map → 4-to-1 mux 實作(不准用閘)

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

### 🟢 D Latch(D 鎖存器,故意產生 latch)

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

### 🟢 同步 reset + 主動高 + 8-bit DFF + 各種變化

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

### 🟡 byteena(byte enable)16-bit DFF

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

### 🔴 100-bit ripple carry(自己用 generate)

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

### 🟢 Adder & signed overflow

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

### 🟢 Testbench 動態次數(repeat)

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

### 🟢 多 module 撞名(`top_module` 在多題都叫一樣)

**卡點**
- HDLBits 用其他題的 module(如 mt2015_q4a, q4b)組合,**那些題的 module 都叫 `top_module`**
- 三個 `top_module` 同檔 → 撞名報錯

**頓悟點**
- **複製貼進來後手動改名**:把貼進來的副本 `module top_module` 那行改成 `mod_a`、`mod_b`
- 原題 HDLBits 不用改(也改不了),改的是你貼進編輯框的副本
- 然後主 `top_module` 實例化改名後的 `mod_a`/`mod_b`

---

## B. 補進「二、核心觀念整理」

### 🔧 數字進位前綴(各種寫法)

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

### 🔧 重複運算子 `{N{x}}`

```verilog
{4{1'b1}} = 4'b1111      // 1 重複 4 次
{16{1'b1}} = 16'hFFFF    // 16-bit 全 1
{3{4'hA}} = 12'hAAA      // 1010 重複 3 次
```

- **格式**:**`{N{內容}}`**——N 在外,內容在內,**雙層大括號**
- N **必須常數**(編譯時決定位寬)
- N **要對應 output 的位寬**(out 是 16-bit 就重複 16 次)
- 內容寫 `1'b1` 明確位寬,別寫 `1`(會被當 32-bit)

### 🔧 K-map 化簡通則

- **SOP**:圈 **1**,1→x, 0→~x,項內 `&`、項間 `|`
- **POS**:圈 **0**,**0→x, 1→~x**(跟 SOP 相反!),項內 `|`、項間 `&`
- 圈大小必須是 2 的次方(1, 2, 4, 8, 16)
- 可繞行(左右、上下相連)
- 圈越大化簡越多;**先找 8 格 → 4 格 → 2 格**
- don't care 全當 1 用(SOP)/ 當 0 用(POS),圈大圈
- **棋盤格 pattern = XOR**(odd parity),K-map 圈不出來,本質是 `^`
- K-map 軸**是 Gray code 順序 00,01,11,10**,後兩欄容易看反

### 🔧 DFF / Latch / reset 完整對照

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
    if (reset) q <= 8'b0;
    else       q <= d;
```

**DFF + 非同步 + active-high reset**:
```verilog
always @(posedge clk or posedge reset)
    if (reset) q <= 8'b0;
    else       q <= d;
```

**reset 重置值不一定是 0**:可以 `q <= 8'h34;` 等任意值
**負邊觸發**:`@(negedge clk)` 代替 `@(posedge clk)`

### 🔧 signed overflow(8-bit)

- 判斷:`(a[7] == b[7]) && (s[7] != a[7])`(同號相加 + 結果反號)
- 二補數還原:**signed = unsigned − 2^N**
- 8-bit 範圍 -128~127,16-bit -32768~32767

### 🔧 波形圖讀法(Yours vs Ref + Mismatch)

- 4 區:Inputs / Yours / Ref / Mismatch
- **Mismatch 凸起 = 你跟 Ref 不同;平的 = 一樣 ✅**
- 逐欄(同一時段)對照 Yours 和 Ref 找差異
- DFF 看 **clk 上升緣**那一刻 d 的值決定 q

---

## C. 補進「三、最常踩雷 Top」

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
