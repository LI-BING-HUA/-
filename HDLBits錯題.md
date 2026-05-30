# 📚 Verilog / HDLBits 學習總整理(完整版)

---

難度:🔴 觀念類(理解卡關) ｜ 🟡 細節類(粗心 / 邊界) ｜ 🟢 語法類(寫法不熟)

---

## 📋 題目目錄

### Verilog Language
- 🟢 Vectors - Vector concatenation operator
- 🟢 Vectors - Vector reversal 1
- 🟢 Vectors - Replication operator
- 🟢 Modules: Hierarchy - Modules and Vectors
- 🔴 Modules: Hierarchy - Adder-subtractor
- 🟢 Procedures - Priority encoder
- 🔴 Procedures - Avoiding latches
- 🟢 More verilog features - Combinational for-loop: Vector reversal
- 🟡 More verilog features - Combinational for-loop: 255-bit popcount
- 🟡 More verilog features - Generate for-loop: 100-bit binary adder 2
- 🔴 More verilog features - Generate for-loop: 100-digit BCD adder

### Circuits - Combinational Logic
- 🟡 Basic Gates - Two-bit equality
- 🟡 Basic Gates - Gates and Vectors
- 🟢 Multiplexers - 9-to-1 multiplexer
- 🔴 Arithmetic - Signed addition overflow
- 🔴 Karnaugh Map - Minimum SOP and POS
- 🔴 Karnaugh Map - K-map implemented with a multiplexer

### Circuits - Sequential Logic
- 🟡 Latches and Flip-Flops - DFF with reset value
- 🟡 Latches and Flip-Flops - DFF with byte enable
- 🟢Latches and Flip-Flops - D Latch
- 🔴Latches and Flip-Flops - Detect both edges
- 🔴Latches and Flip-Flops - Edge capture register
- 🔴Latches and Flip-Flops - Dual-edge triggered flip-flop
- 🟡Counters - Slow decade counter
- 🔴Counters - Counter 1-12
- 🔴Counters - Counter 1000
- 🔴Counters - 4-digit decimal counter
- 🟡Counters - 12-hour clock
- 🔴Shift Registers - Left/right arithmetic shift by 1 or 8
- 🟡Shift Registers - shift register 1
- 🟡Shift Registers - shift register 2
- 🟢Shift Registers - 3-input LUT
- 🟢More Circuits - Rule 90
- 🔴More Circuits - Rule 110

### 附錄
- 🔧 核心觀念整理(精簡版)
- ⚠️ 最常踩雷

---

## Verilog Language - Vectors - Vector concatenation operator
<img width="1589" height="324" alt="image" src="https://github.com/user-attachments/assets/48eaebea-89a5-4322-82d4-956b96526d9f" />

```verilog
{a, b, c, d, e, f, 2'b11}     // ✅ 正確
{a, b, c, d, e, f, 11}        // ⚠️ 危險!
```

11 沒位寬,Verilog 會把它當成「預設 32-bit 整數」。所以:
- 你以為的 11 = 想要的 2 個 bit(11)
- Verilog 實際當成 32'd11 = 32'b00000000_00000000_00000000_00001011(32-bit!)

### Write your solution here
```verilog
module top_module (
    input  [4:0] a, b, c, d, e, f,
    output [7:0] w, x, y, z
);
    assign {w, x, y, z} = {a, b, c, d, e, f, 2'b11};
endmodule
```

---

## Verilog Language - Vectors - Vector reversal 1
<img width="582" height="88" alt="image" src="https://github.com/user-attachments/assets/43bf5dbe-1193-47da-8369-7718898a3166" />

for 迴圈 / if 不能直接寫在 module 裡,要放在:
- always @(*)(行為描述)
- generate 區塊(合成時展開)

### Write your solution here
```verilog
module top_module (
    input  [7:0] in,
    output reg [7:0] out
);
    integer i;
    always @(*) begin
        for (i = 0; i < 8; i = i + 1)
            out[i] = in[7-i];
    end
endmodule
```

---

## Verilog Language - Vectors - Replication operator
<img width="1601" height="214" alt="image" src="https://github.com/user-attachments/assets/d93740aa-7efe-4780-b4dc-63a039e7b34d" />

### Verilog 拼接:雙層大括號注意

❌ 錯誤:`{24{in[7]}, in}` ← 少一個 `{}`

✅ 正確:`{{24{in[7]}}, in}` ← **外層** + **內層** 兩組 `{}`

### Write your solution here
```verilog
module top_module (
    input  [7:0]  in,
    output [31:0] out
);
    assign out = {{24{in[7]}}, in};
endmodule
```

---

## Verilog Language - Modules : Hierachy - Modules and Vectors
<img width="1305" height="424" alt="image" src="https://github.com/user-attachments/assets/4de56fcf-d60b-40dc-a946-2614e8963d02" />

- 看到 always → ` output reg [7:0] q`

- Mux 想到 case ✅

### Write your solution here
```verilog
module top_module (
    input        clk,
    input  [7:0] d,
    input  [1:0] sel,
    output reg [7:0] q
);
    wire [7:0] w1, w2, w3;
    my_dff8 my_dff8_1(.clk(clk), .d(d),  .q(w1));
    my_dff8 my_dff8_2(.clk(clk), .d(w1), .q(w2));
    my_dff8 my_dff8_3(.clk(clk), .d(w2), .q(w3));
    always @(*) begin
        case (sel)
            2'd0: q = d;
            2'd1: q = w1;
            2'd2: q = w2;
            2'd3: q = w3;
        endcase
    end
endmodule
```


---

## Verilog Language - Modules : Hierachy - Adder-subtractor
<img width="759" height="487" alt="image" src="https://github.com/user-attachments/assets/46db646a-33fe-4ff9-83e0-ff0e2ae6f732" />

`module add16 ( input[15:0] a, input[15:0] b, input cin, output[15:0] sum, output cout );`

技巧 : assign wxor = {32{sub}} ^ b;

### 兩種「組合輸出」寫法(選一個,不可混用)

**方案 A:直接接 output(這題用這個)**
```verilog
add16 a1(... .sum(sum[15:0]));     // 直接接 sum 的低 16-bit
add16 a2(... .sum(sum[31:16]));    // 直接接 sum 的高 16-bit
// 結束,不用 assign sum
```

**方案 B:中間 wire 再拼接**
```verilog
wire [15:0] ws1, ws2;
add16 a1(... .sum(ws1));
add16 a2(... .sum(ws2));
assign sum = {ws2, ws1};           // 最後拼接
```

**選一個,別混用** → 混用就是 multiple drivers。

### Write your solution here
```verilog
module top_module (
    input  [31:0] a, b,
    input         sub,
    output [31:0] sum
);
    wire w1;
    wire [31:0] wxor;
    assign wxor = {32{sub}} ^ b;
    add16 add16_1(.a(a[15:0]),  .b(wxor[15:0]),  .cin(sub), .cout(w1), .sum(sum[15:0]));
    add16 add16_2(.a(a[31:16]), .b(wxor[31:16]), .cin(w1),  .cout(),   .sum(sum[31:16]));
endmodule
```

---

## Verilog Language - Procedures - Priority encoder
<img width="1572" height="172" alt="image" src="https://github.com/user-attachments/assets/2b1afb15-0cd5-4873-937c-22546aaa11f9" />

|  | `case` | `casez` | `casex` |
|---|--------|---------|---------|
| **完全匹配** | 才算中 | 才算中 | 才算中 |
| **`?` 當 don't care** | ❌ 不支援 | ✅ 支援 | ✅ 支援 |
| **`z` 當 don't care** | ❌ 不支援 | ✅ 支援 | ✅ 支援 |
| **`x` 當 don't care** | ❌ 不支援 | ❌ 不支援 | ⚠️ 支援(危險) |

### ⚠️ casex 為什麼危險?

訊號未初始化會是 `x`,假設 `sel = 3'bxxx`:

```verilog
casex (sel)
    3'b001: a = 1;     // sel=xxx,因為 x 被當 don't care → 第一條就匹配!
    3'b010: a = 2;
    3'b100: a = 3;
endcase
```

**第一條直接匹配** → 走 `a=1` → **bug 被掩蓋**,你以為功能正常,其實訊號是垃圾值。

### Write your solution here
```verilog
module top_module (
    input [3:0] in,
    output reg [1:0] pos  );
    always @(*) begin
            casez(in)
                4'b???1: pos = 2'd0;
                4'b??10: pos = 2'd1;
                4'b?100: pos = 2'd2;
                4'b1000: pos = 2'd3;
                default: pos = 2'd0;
            endcase
    end
endmodule
```

---

## Verilog Language - Procedures - Avoiding latches

避免組合邏輯 latch 有兩招:
- case 加 default — 適合 output 少(1 個)
- 進 case 前先給所有 output 預設值 — 適合 output 多(這題 4 個)

### Write your solution here
```verilog
module top_module (
    input [15:0] scancode,
    output reg left,
    output reg down,
    output reg right,
    output reg up  ); 
always @(*) begin
    up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0;// ← 先把 4 個都設 0
    case (scancode)
        16'he06b: left  = 1'b1;
        16'he072: down  = 1'b1;
        16'he074: right = 1'b1;
        16'he075: up    = 1'b1;
    endcase
end
endmodule
```
---

## Verilog Language - More verilog features - Combinational for-loop : Vector reversal 
<img width="605" height="50" alt="image" src="https://github.com/user-attachments/assets/ee586388-0ad8-40d3-a3d7-c5308dcf690e" />

## 🔧 generate 通式

### 基本架構

```verilog
genvar i;                            // ← genvar(不是 integer)
generate
    for (i = 0; i < N; i = i + 1) begin : 標籤名   // 只一行也要 begin/end + 標籤
        // 要重複的東西(實例化、assign、邏輯閘)
    end
endgenerate
```
### 三個語法要點

| 規則 | 為什麼 |
|------|--------|
| 用 `genvar` 不是 `integer` | generate 專用變數,合成時展開 |
| 必須有 `begin : 標籤名` | 否則合成器不知道怎麼編號每個迭代 |
| 用 `assign` 不是 `=` | 在 generate 區塊外(不在 always 裡) |

### Write your solution here
```verilog
module top_module( 
    input [99:0] in,
    output [99:0] out
);
    genvar i;
    generate
        for (i = 0; i < 100; i = i + 1) begin : rev
            assign out[i] = in[99 - i];
        end
    endgenerate
endmodule
```

---

## Verilog Language - More verilog features - Combinational for-loop : 255-bit population count
<img width="1350" height="36" alt="image" src="https://github.com/user-attachments/assets/7e079dc5-0fe7-4ff0-8414-e1bbb2104c6d" />

### Write your solution here
```verilog
module top_module( 
    input [254:0] in,
    output reg [7:0] out );
    integer i;
    always @(*) begin
        out = 0;
        for (i = 0;i < 255; i = i + 1) begin
            //if (in[i] == 1)
            out = out + in[i];
        end 
    end
    //assign out = $countones(in);      SystemVerilog 內建函式
endmodule
```
---

## Verilog Language - More verilog features - Generate for-loop : 100-bit binary adder 2
<img width="1523" height="113" alt="image" src="https://github.com/user-attachments/assets/5d478d84-4f01-4ced-9f79-bb73736e7359" />


### Write your solution here
```verilog
module top_module( 
    input [99:0] a, b,
    input cin,
    output [99:0] cout,
    output [99:0] sum );
    full_adder full_adder0(.a(a[0]), .b(b[0]), .cin(cin), .cout(cout[0]), .sum(sum[0]));
    genvar i;
    generate
        for (i = 1; i < 100; i = i + 1) begin : add_chain
            full_adder full_adderu(.a(a[i]), .b(b[i]), .cin(cout[i - 1]), .cout(cout[i]), .sum(sum[i]));
        end
    endgenerate
endmodule

module full_adder (
    input  a, b, cin,
    output sum, cout
);
    assign sum  = a ^ b ^ cin;         
    assign cout = ((a ^ b) & cin) | (a & b);
endmodule
```

---

## Verilog Language - More verilog features - Generate for-loop : 100-digit BCD adder
<img width="1580" height="381" alt="image" src="https://github.com/user-attachments/assets/d76e2e25-a700-4e3b-a70f-9cca5a97ef1f" />

### 句型

```verilog
a[start +: width]    // 從 start 往高位取 width 個 bit
a[start -: width]    // 從 start 往低位取 width 個 bit
```

**起點可變、寬度固定**(寬度必須是常數)
### 對照表

| 寫法 | 等價於 |
|------|--------|
| `a[0 +: 4]` | `a[3:0]` |
| `a[4 +: 4]` | `a[7:4]` |
| `a[8 +: 4]` | `a[11:8]` |
| `a[7 -: 4]` | `a[7:4]` |
| `a[3 -: 4]` | `a[3:0]` |

### Write your solution here
```verilog
module top_module (
    input  [399:0] a, b,
    input          cin,
    output         cout,
    output [399:0] sum
);
    wire [99:0] carry;          // 99 條進位線(中間進位用)
    
    genvar i;
    generate
        for (i = 0; i < 100; i = i + 1) begin : bcd_loop
            if (i == 0)
                bcd_fadd fa (
                    .a   ( a[3:0]   ),
                    .b   ( b[3:0]   ),
                    .cin ( cin      ),          // 第 0 個接外部 cin
                    .cout( carry[0] ),
                    .sum ( sum[3:0] )
                );
            else if (i == 99)
                bcd_fadd fa (
                    .a   ( a[i*4 +: 4]   ),
                    .b   ( b[i*4 +: 4]   ),
                    .cin ( carry[i-1]    ),
                    .cout( cout          ),     // 最後一個接 cout(最終進位)
                    .sum ( sum[i*4 +: 4] )
                );
            else
                bcd_fadd fa (
                    .a   ( a[i*4 +: 4]   ),
                    .b   ( b[i*4 +: 4]   ),
                    .cin ( carry[i-1]    ),     // 中間的接前一個 carry
                    .cout( carry[i]      ),
                    .sum ( sum[i*4 +: 4] )
                );
        end
    endgenerate
endmodule
```

---

## Circuits - Combinational Logic - Basic Gates - Two-bit equality

第一次寫 `assign z = ~(A ^ B);` → 錯(`~(A^B)` 是 2-bit,z 只 1-bit,被截掉只剩最低位 → A=10,B=00 誤判 z=1)

### Write your solution here
```verilog
module top_module ( input [1:0] A, input [1:0] B, output z ); 
    assign z = &(~(A ^ B));
    //assign z = A == B;
    //assign z = A[1] == B[1] && A[2] == B[2];
endmodule

```

---

## Circuits - Combinational Logic - Basic Gates - Gates and Vectors
<img width="1578" height="405" alt="image" src="https://github.com/user-attachments/assets/d14961f1-6960-47a3-a146-ae1c040cbf29" />

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
## 邏輯 vs 位元運算子

| 類型 | 運算子 | 結果寬度 | 用途 |
|------|--------|---------|------|
| **邏輯** | `&&` `\|\|` `!` `==` `!=` | **永遠 1-bit** | 判斷真假(if、三元、條件) |
| **位元** | `&` `\|` `~` `^` | **跟輸入同寬** | 逐位運算(算 bit 值) |


**重點 : 題目out_both, out_any沒用滿 port [3:0], 所以不用管題目寫的左右**

### Write your solution here
```verilog
module top_module( 
    input [3:0] in,
    output [2:0] out_both,
    output [3:1] out_any,
    output [3:0] out_different );
    assign out_both = in[3:1] & in[2:0];
    assign out_any = in[3:1] | in[2:0];
    assign out_different = in[3:0] ^ {in[0], in[3:1]};
endmodule
```

---

## Circuits - Combinational Logic - Multiplexers - 9-to-1 multiplexer
<img width="1451" height="43" alt="image" src="https://github.com/user-attachments/assets/c4ca2d1a-a4ac-48bf-80ae-b21ad8e0bc53" />

default 全 1 等價寫法:
  ```verilog
  out = {16{1'b1}};            // 重複運算子
  out = 16'hFFFF;              // hex 全 F
  out = 16'b1111_1111_1111_1111;  // 二進位寫死
  out = '1;                    // SystemVerilog 簡潔寫法:'1 = 全 1(自動填滿位寬)
  ```
default 全 0 等價寫法:
  ```verilog
  out = 16'b0;
  out = 16'h0;
  out = '0;                    // SystemVerilog 簡潔寫法:'0 = 全 0
  ```

### Write your solution here
```verilog
module top_module( 
    input [15:0] a, b, c, d, e, f, g, h, i,
    input [3:0] sel,
    output reg [15:0] out );
    always @(*) begin
		case (sel)
			4'h0: out = a;
			4'h1: out = b;
			4'h2: out = c;
			4'h3: out = d;
			4'h4: out = e;
			4'h5: out = f;
			4'h6: out = g;
			4'h7: out = h;
			4'h8: out = i;
            default: out = {16{1'b1}};
		endcase
	end
endmodule
```

---

## Circuits - Combinational Logic - Arithmetic - Signed addition overflow
<img width="1521" height="74" alt="image" src="https://github.com/user-attachments/assets/58da8010-8902-4671-8c95-b3d4a5cf185f" />

## 🔧 二補數還原公式

### 核心公式

```
signed = unsigned − 2^N    (當 MSB = 1 時)
```

- **N** = 位寬(幾 bit)
- **2^N** = 該位寬的總範圍大小
- **MSB = 1 才需要減**(=0 時 signed 跟 unsigned 一樣)

### 不同位寬對照

| 位寬 | 2^N | signed 範圍 |
|------|-----|-------------|
| 4-bit | 16 | −8 ~ +7 |
| 8-bit | **256** | −128 ~ +127 |
| 16-bit | 65536 | −32768 ~ +32767 |
| 32-bit | 2^32 | ±2^31 |

### 快速判正負(看 hex 第一字)

| Hex 第一字 | Binary 開頭 | 正負 |
|-----------|------------|------|
| 0~7 | `0xxx` | **正數** ✅ |
| 8~F | `1xxx` | **負數** ⚠️ |

### 範例(8-bit)

| hex | unsigned | signed | 計算 |
|------|----------|--------|------|
| `8'h00` | 0 | 0 | MSB=0,直接讀 |
| `8'h7F` | 127 | +127 | MSB=0,直接讀 |
| `8'h80` | 128 | **−128** | 128 − 256 |
| `8'h90` | 144 | **−112** | 144 − 256 |
| `8'hFF` | 255 | **−1** | 255 − 256 |

### 💡 黃金口訣

> **全 1 = −1**(任何位寬都成立!)
> - `4'b1111` = −1
> - `8'hFF` = −1
> - `16'hFFFF` = −1
> - `32'hFFFFFFFF` = −1

### Write your solution here
```verilog
module top_module (
    input [7:0] a,
    input [7:0] b,
    output [7:0] s,
    output overflow
); //
 
    assign s = a + b;
    assign overflow = (a[7] == b[7]) && (s[7] != a[7]);
endmodule
```

---

## Circuits - Combinational Logic - Karnaugh Map to Circuit - Minimum SOP and POS
<img width="1595" height="171" alt="image" src="https://github.com/user-attachments/assets/db842cf7-a60f-4be0-aba2-034e283e7698" />

### 💡 重點:SOP vs POS 規則完全相反

| 形式 | 圈什麼 | 0 對應 | 1 對應 | 項內 | 項間 |
|------|--------|--------|--------|------|------|
| **SOP** | 圈 **1** | `~x` | `x` | `&` | `\|` |
| **POS** | 圈 **0** | `x` | `~x` | `\|` | `&` |

### SOP 的精神

> **每個 minterm「點亮」一個位置**
> 該位 = 1 才匹配 → 用 `|` 串,「只要任一個成立 → 1」

### POS 的精神

> **每個 maxterm「壓黑」一個位置**
> 該位 = 0 才強制壓 0 → 用 `&` 串,「只要任一個 = 0 → 整體 = 0」

### Write your solution here
```verilog
module top_module (
    input  a, b, c, d,
    output out_sop,
    output out_pos
);
    // SOP:圈 1 的位置寫成 minterm,用 | 串
    assign out_sop = (~a & ~b &  c &  d)
                   | (~a &  b &  c &  d)
                   | ( a &  b &  c &  d)
                   | ( a & ~b &  c &  d)
                   | (~a & ~b &  c & ~d);

    // POS:圈 0 的位置寫成 maxterm,用 & 串
    assign out_pos = ( a |  b |  c |  d)
                   & ( a | ~b |  c |  d)
                   & (~a | ~b |  c |  d)
                   & (~a |  b |  c |  d)
                   & ( a |  b |  c | ~d)
                   & ( a | ~b |  c | ~d)
                   & (~a | ~b |  c | ~d)
                   & (~a |  b |  c | ~d)
                   & ( a | ~b | ~c |  d)
                   & (~a | ~b | ~c |  d)
                   & (~a |  b | ~c |  d);
endmodule
```

---

## Circuits - Combinational Logic - Karnaugh Map to Circuit - K-map implemented with a multiplexer
<img width="1579" height="799" alt="image" src="https://github.com/user-attachments/assets/cc3f1c07-8a25-4f35-aff4-9a416d83a092" />

```
ab=00 → 走 mux_in[0]
ab=01 → 走 mux_in[1]
ab=10 → 走 mux_in[2]
ab=11 → 走 mux_in[3]
```

### 核心步驟:K-map 拆 4 欄

把 K-map 按 **ab 欄拆開**,每欄對應一條 mux_in:

| mux 編號(binary) | 對應 K-map 哪欄 |
|------------------|----------------|
| mux_in[0] (ab=00) | K-map 第 1 欄 |
| mux_in[1] (ab=01) | K-map 第 2 欄 |
| mux_in[2] (ab=10) | K-map **第 4 欄** ⚠️ |
| mux_in[3] (ab=11) | K-map **第 3 欄** ⚠️ |

每條 mux_in 是「**ab 固定後,只剩 c, d 的子函數**」。

### Write your solution here
```verilog
module top_module (
    input c,
    input d,
    output [3:0] mux_in
); 
    assign mux_in[0] = d ? 1 : c;
    assign mux_in[1] = 0;
    assign mux_in[2] = ~d ? 1 : 0;
    assign mux_in[3] = ~d ? 0 : c;
endmodule
```

---

## Circuits - Sequemtial Logic - Latches and Flip-Flops - DFF with reset value
<img width="1525" height="78" alt="image" src="https://github.com/user-attachments/assets/6484f43e-c294-411f-98da-3019e22ba4e7" />

**reset 四種組合**:

| reset 類型 | 敏感列表 | if 判斷 |
|-----------|---------|---------|
| **同步 + active-high** | `@(posedge clk)` | `if (reset)` |
| 同步 + active-low | `@(posedge clk)` | `if (~reset)` |
| 非同步 + active-high | `@(posedge clk or posedge reset)` | `if (reset)` |
| 非同步 + active-low | `@(posedge clk or negedge reset)` | `if (~reset)` |

**關鍵字翻譯**:
- **同步**(synchronous)→ 敏感列表**只有 clk**,reset 寫在 if 裡
- **非同步**(async)→ 多加 `or posedge/negedge reset`
- **主動高**(active-high)→ `if (reset)`(不加 `~`)
- **主動低**(active-low)→ `if (~reset)`

**C/Python**:

| 前綴 | 進位 | 例 |
|------|------|-----|
| (無) | 十進位 | `52` |
| `0b` | 二進位 | `0b110100` |
| `0o` | 八進位 | `0o64` |
| `0x` | 十六進位 | `0x34` |

**Verilog**:

| 符號 | 進位 | 例 |
|------|------|-----|
| `'b` | binary | `8'b00110100` |
| `'o` | octal | `8'o64` |
| `'d` | decimal | `8'd52` |
| `'h` | hex | `8'h34` |

### Write your solution here
```verilog
module top_module (
    input clk,
    input reset,
    input [7:0] d,
    output reg [7:0] q
);
    always @(negedge clk) begin
        if (reset)
            q <= 8'h34;
        else
            q <= d;
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Latches and Flip-Flops - DFF with byte enable
<img width="1558" height="199" alt="image" src="https://github.com/user-attachments/assets/9bd38fa1-7ac8-4c4c-b902-caf61884e45d" />

- **byte enable**:每個 bit 控制一個位元組要不要更新
  - `byteena[1]` 控制 d[15:8] (上位元組)
  - `byteena[0]` 控制 d[7:0] (下位元組)
  - 該位=1 → 更新、=0 → 鎖住保持
- 寫法用「**if 沒 else**」實現「開關關閉時保持」:

### Write your solution here
```verilog
module top_module (
    input         clk,
    input         resetn,         // 同步 reset, active-low
    input  [1:0]  byteena,
    input  [15:0] d,
    output reg [15:0] q
);
    always @(posedge clk) begin
        if (!resetn) begin
            q <= 16'b0;            // reset 時整個清 0
        end
        else begin
            if (byteena[1])        // 上 byte 開關
                q[15:8] <= d[15:8];
            if (byteena[0])        // 下 byte 開關
                q[7:0]  <= d[7:0];
            // 注意:沒 else!byteena=0 時就「不賦值」 → q 保持不變
        end
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Latches and Flip-Flops - D Latch
<img width="925" height="239" alt="image" src="https://github.com/user-attachments/assets/f88b882c-70ef-4a1b-af2c-308ea83164b1" />

- **沒 else 是故意的** → 產生 latch(這次是要的功能,不是 bug)
- Quartus 警告「latch inferred」是預期的,可無視
- 平常組合邏輯沒 else = 不小心 latch(要避免);這題沒 else = 故意 latch(要的)

### Write your solution here
```verilog
module top_module (
    input d, 
    input ena,
    output q);
    always @(*) begin
        if (ena)
    		q <= d;
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Latches and Flip-Flops - Detect both edges
<img width="860" height="304" alt="image" src="https://github.com/user-attachments/assets/80c53bff-cc64-40fa-90eb-bd2f8de84b96" />

## 邊緣偵測公式速查
### 三種偵測方式
| 偵測什麼 | 公式 |
|---------|------|
| 上升緣(0→1) | `in & ~in_prev` |
| 下降緣(1→0) | `~in & in_prev` |
| **任意邊緣**(值變了) | **`in ^ in_prev`** ⭐ |

### 真值表(以 1-bit 為例)

| in_prev | in | `in & ~in_prev` | `~in & in_prev` | `in ^ in_prev` |
|---------|----|------------------|------------------|----------------|
| 0 | 0 | 0 | 0 | 0 |
| **0** | **1** | **1** ⭐ | 0 | **1** ⭐ |
| **1** | **0** | 0 | **1** ⭐ | **1** ⭐ |
| 1 | 1 | 0 | 0 | 0 |

**`^`(XOR)= 「值變了」**——同時抓上升和下降緣

### 應用

- **按鈕觸發**:上升緣(按下那瞬間)
- **訊號釋放**:下降緣(放開瞬間)
- **狀態變化偵測**:任意邊緣(動了就觸發)

### Write your solution here
```verilog
module top_module (
    input clk,
    input [7:0] in,
    output [7:0] anyedge
);
    reg [7:0] in_prev;
    always @(posedge clk) begin
    	in_prev <= in;
        anyedge <= in ^ in_prev;
    end
endmodule

```

---

## Circuits - Sequemtial Logic - Latches and Flip-Flops - Edge capture register
<img width="1563" height="650" alt="image" src="https://github.com/user-attachments/assets/4544724d-42fb-402f-92aa-54a08a3b3836" />

### 📋 規則對照(4 種情況)

| 情況 | reset | in 變化 | 公式行為 | out 結果 |
|------|-------|---------|---------|---------|
| A | 1 | 任何 | 走 if 分支 | 全清 0 |
| B | 0 | 1→0(下降緣) | `~in & in_prev` 該位 = 1,OR 累積 | 那位變 1 |
| C | 0 | 0→1(上升緣) | `~in & in_prev = 0`,OR 0 不影響 | 維持 |
| D | 0 | 沒變 | `~in & in_prev = 0`,OR 0 不影響 | 維持 |

### 🎯 從 0 推答案的 SOP(5 步驟)

1. **讀題 → 拆出規則**(下降緣觸發、維持、reset 清、其他保持)
2. **逐 cycle 套規則**(用「現在 in」和「上 cycle in」判斷)
3. **手動算每 cycle 的 out**(根據規則,不套公式)
4. **從 trace 數據找模式**:
   - 每 cycle 都「看上 cycle 的 in」→ 需要 `in_prev`
   - 每 cycle 都「逐位比 1→0」→ `~in & in_prev`
   - 每 cycle 都「舊燈不熄 + 新燈加入」→ `out |`
   - reset 直接清 0,跳過其他邏輯 → `if-else`
5. **翻譯成 Verilog**

### 💡 應用場景

這設計叫 **"Capture Register" / "Sticky Bit"**,實際應用:
- **錯誤旗標**(Error flag):瞬間錯誤閃一下就抓住,維持到工程師清除
- **Interrupt flag**:中斷一觸發就記住,軟體讀過再清
- **過熱警告**:過熱訊號閃一下,維持警告直到處理
- **ECC 紀錄**:記憶體 bit 翻轉事件

**核心精神**:**「事件曾發生過」的標記**,跟「邊緣脈衝」不同

### 🔧 相關公式速查

| 偵測什麼 | 公式 |
|---------|------|
| 上升緣(0→1) | `in & ~in_prev` |
| 下降緣(1→0) | `~in & in_prev` |
| 任意邊緣 | `in ^ in_prev` |
| **下降緣 + 記憶**(這題) | **`out \| (~in & in_prev)`** |
| **上升緣 + 記憶** | `out \| (in & ~in_prev)` |
| **任意邊緣 + 記憶** | `out \| (in ^ in_prev)` |

### Write your solution here
```verilog
module top_module (
    input         clk,
    input  [31:0] in,
    input         reset,
    output reg [31:0] out
);
    reg [31:0] in_prev;
    
    always @(posedge clk) begin
        in_prev <= in;
        if (reset)
            out <= 32'b0;
        else
            out <= out | (~in & in_prev);
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Latches and Flip-Flops - Dual-edge triggered flip-flop
<img width="1562" height="331" alt="image" src="https://github.com/user-attachments/assets/0d5440d5-e609-4405-bef7-5e33024e7b0c" />

**Verilog 不支援同時 `@(posedge clk or negedge clk)` 的 FF**

**兩個 always 各自抓一邊,XOR 合併輸出**
```verilog
// ❌ 大忌!q 被兩個 always 賦值 → multiple driver 錯誤
always @(posedge clk) q <= d;
always @(negedge clk) q <= d;     // ❌ 編譯錯
```

**Verilog 規則**:**一個變數只能在一個 always 區塊賦值**(否則 multiple driver)。

#### ✅ 正解:**兩個各自的 reg + XOR 合併**

### Write your solution here
```verilog
module top_module (
    input clk,
    input d,
    output q
);
    reg q_neg, q_pos;
    always @(posedge clk) begin
    	q_pos <= d;
    end
    always @(negedge clk) begin
    	q_neg <= d;
    end
    assign q = clk ? q_pos : q_neg;
endmodule
```

---

## Circuits - Sequemtial Logic - Counters - Slow decade counter
<img width="1586" height="319" alt="image" src="https://github.com/user-attachments/assets/dd229814-5403-4456-b56a-34b1e7b02e10" />

### 🪜 4 階梯 SOP 對照(清、停、轉、跑)

| 階梯 | 對應寫法 |
|------|---------|
| 1. **reset** | `if (reset) q <= 0` |
| 2. **enable**(slowena) | `else if (slowena) ...`(slowena=0 跳過 → q 保持)|
| 3. **邊界**(q==9) | `if (q == 9) q <= 0` |
| 4. **正常** | `else q <= q + 1` |

### Write your solution here
```verilog
module top_module (
    input clk,
    input slowena,
    input reset,
    output [3:0] q);
    always @(posedge clk) begin
        if (reset)
			q <= 0;
        else if (slowena) begin
            if (q == 9)
                q <= 0;
            else
            	q <= q + 1;
        end
        else
            q <= q;
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Counters - Counter 1-12
<img width="1575" height="796" alt="image" src="https://github.com/user-attachments/assets/451b935b-5b73-499d-9cb4-c89ccc625732" />

### 🎯 count4 的「3 種動作」(必懂!)

| load | enable | Q 動作 |
|------|--------|--------|
| **1** | 任何 | **Q ← d**(強制覆蓋,load 優先)|
| 0 | **1** | **Q ← Q + 1**(count4 自己加 1)⭐ |
| 0 | 0 | Q 保持 |

**關鍵觀念**:
- **+1 是 count4 自己做的事**,你不寫 +1 邏輯
- **load = 「強制覆蓋」**,不是 +1!d 才是「覆蓋成什麼值」
- **load 跟 +1 是互斥動作**,不會同時發生

### Write your solution here
```verilog
module top_module (
    input clk,
    input reset,
    input enable,
    output [3:0] Q,
    output c_enable,
    output c_load,
    output [3:0] c_d
); 
    assign c_enable = enable;                                // count4!現在准不准動?
    assign c_load = reset || (Q == 4'd12 && enable == 1'b1); // count4!現在要不要強制塞 1 進去?
    assign c_d = 4'd1;                                       // count4!要塞什麼值進去?
    
    count4 the_counter (
        .clk    (clk),
        .enable (c_enable),
        .load   (c_load),
        .d      (c_d),
        .Q      (Q)
    );

endmodule
```

---

## Circuits - Sequemtial Logic - Counters - Counter 1000(BCD 1000Hz → 1Hz 分頻器)
<img width="1581" height="497" alt="image" src="https://github.com/user-attachments/assets/0cec9557-8228-4c39-b17a-9224b1fc6a62" />

### 觀念:用三個 mod-10 計數器串成 mod-1000
- 1000 = 10 × 10 × 10,所以串三個「數 0~9」的 BCD 計數器
- 注意 wire bit 數

### 進位邏輯(這個要背)
```verilog
c_enable[0] = 1'b1;                                             // 最快那個永遠 enable
c_enable[1] = (c0 == 4'd9);                                     // 個位滿 9
c_enable[2] = (c0 == 4'd9) && (c1 == 4'd9);                     // 個十都滿
OneHertz    = (c0 == 4'd9) && (c1 == 4'd9) && (c2 == 4'd9 );    // 千位輸出
```

### Write your solution here
```verilog
module top_module (
    input clk,
    input reset,
    output OneHertz,
    output [2:0] c_enable
); 
    wire [3:0] c0, c1, c2;
	assign c_enable[0] = 1'b1;
	assign c_enable[1] = (c0 == 4'd9);
	assign c_enable[2] = (c0 == 4'd9) && (c1 == 4'd9);
	assign OneHertz    = (c0 == 4'd9) && (c1 == 4'd9) && (c2 == 4'd9 );
    
    bcdcount counter0 (clk, reset, c_enable[0], c0);
    bcdcount counter1 (clk, reset, c_enable[1], c1);
    bcdcount counter2 (clk, reset, c_enable[2], c2);
endmodule
```

---

## Circuits - Sequemtial Logic - Counters - 4-digit decimal counter
<img width="1549" height="381" alt="image" src="https://github.com/user-attachments/assets/055e0bb9-df87-4ca3-b4cd-800eb3310a72" />

### 觀念:四個 BCD 串接(個、十、百、千位)
- ena[1]、ena[2]、ena[3] 是進位 enable(沒有 ena[0],個位永遠數)
- 子模組需要 enable port,沒有就要自己加

### 我踩的雷
- ❌ 子模組沒 enable port,沒辦法控制「只在下面滿時動」
- ❌ output 沒加 reg,被 always 賦值就會報錯
- ❌ 想用 q_ 同時自己數又用子模組 → 又是 multiple drivers

### Write your solution here
```verilog
module top_module (
    input clk,
    input reset,   // Synchronous active-high reset
    output reg [3:1] ena,
    output reg [15:0] q);

    assign ena[1] = (q[3:0] == 4'd9);
    assign ena[2] = (q[3:0] == 4'd9) && (q[7:4] == 4'd9);
    assign ena[3] = (q[3:0] == 4'd9) && (q[7:4] == 4'd9) && (q[11:8] == 4'd9);
    
    Countbcd u1(.clk(clk), .reset(reset), .enable(1'b1), .q(q[3:0]));
    Countbcd u2(.clk(clk), .reset(reset), .enable(ena[1]), .q(q[7:4]));
    Countbcd u3(.clk(clk), .reset(reset), .enable(ena[2]), .q(q[11:8]));
    Countbcd u4(.clk(clk), .reset(reset), .enable(ena[3]), .q(q[15:12]));
endmodule

module Countbcd (
    input clk,
    input reset,        // Synchronous active-high reset
    input enable,
    output reg [3:0] q);
    always @(posedge clk) begin
        if (reset)
            q <= 4'b0;
        else if (enable) begin
            if (q == 4'd9)
                q <= 1'b0;
            else
                q <= q + 1'b1;
        end
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Counters - 12-hour clock
<img width="1568" height="551" alt="image" src="https://github.com/user-attachments/assets/3dc2df08-4c4d-495f-9d42-7796f78bd08f" />

### 觀念:這是 BCD,不是二進位!
- BCD = 每 4 bit 代表一個十進位數字,只准用 0~9
- 顯示 12 點要寫 `8'h12`,**不能寫 `8'd12`(那是 0c)**
- bit pattern「分兩半看」是題目指定

### BCD 的進位寫法
```verilog
// 個位滿 9 要進位歸零,不能直接 +1(會變 a)
(v[3:0] == 4'h9) ? {v[7:4]+4'h1, 4'h0} : v + 8'h1
```

### 時鐘的轉折點(全部要 if 攔截)
| 值     | 為什麼特殊       | 行為                    |
|--------|----------------|------------------------|
| 個位 9 | BCD 規則        | 進位歸零               |
| 59     | 滿了            | 進到上一層              |
| 11     | 12 小時制       | 變 12,翻 pm           |
| 12     | 12 小時制       | 變 1(不是 13)         |

### 我踩的雷
- ❌ 用 `8'd12` 想表示 12 點 → 實際是 `8'h0c`,顯示變成 c
- ❌ 直接 `ss + 8'd1` → 個位 9 變 a(BCD 不允許 a~f)
- ❌ 最後 `else` 沒加 ena → ena=0 時時鐘還在走(不會停)
- ❌ output 沒加 reg
- ❌ 以為 `'h59` 換十進位是 89,糾結很久(其實是約定問題,看法不同)

### 關鍵心法
- **BCD 一律用 `'h` 寫**(`8'h59` 一看就是 5 和 9)
- **每個分支都要擋 ena**(最後的 else 容易漏)
- 「分兩半看」是題目指定 BCD
- always 沒賦值 → register 自動 hold(這就是「暫停」的原理)

### Write your solution here
```verilog
module top_module(
    input clk,
    input reset,
    input ena,
    output pm,
    output [7:0] hh,
    output [7:0] mm,
    output [7:0] ss); 
    
    always @(posedge clk) begin
        if (reset) begin
            pm <= 1'b0;
            hh <= 8'h12;
            mm <= 8'h0;
            ss <= 8'h0;
        end
        else if (ena && hh == 8'h11 && mm == 8'h59 && ss == 8'h59) begin
            pm <= ~pm;
        	hh <= 8'h12;
            mm <= 8'h0;
            ss <= 8'h0;
        end
        else if (ena && hh == 8'h12 && mm == 8'h59 && ss == 8'h59) begin
            hh <= 8'h1;
            mm <= 8'h0;
            ss <= 8'h0;
        end
        else if (ena && mm == 8'h59 && ss == 8'h59) begin
            hh <= (hh[3:0]==4'h9) ? {hh[7:4]+4'h1,4'h0} : hh+8'h1;
            mm <= 8'h0;
            ss <= 8'h0;
        end
        else if (ena && ss == 8'h59) begin
            mm <= (mm[3:0]==4'h9) ? {mm[7:4]+4'h1,4'h0} : mm+8'h1;
            ss <= 8'h0;
        end
        else if (ena)
            ss <= (ss[3:0]==4'h9) ? {ss[7:4]+4'h1,4'h0} : ss+8'h1;  
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Shift Registers - Left/right arithmetic shift by 1 or 8
<img width="1572" height="565" alt="image" src="https://github.com/user-attachments/assets/4dc3fe9c-120f-42c3-b572-1ee451035637" />

### 觀念:算術右移要補符號位,不補 0
- `>>>` 算術右移(補符號位)、`>>` 邏輯右移(補 0)
- **但是!** `>>>` 只有對 `signed` 變數才會補符號位

### 我踩的雷
- ❌ q 沒宣告成 signed,用 `>>>` 還是補 0 → 算術右移失效
- ❌ 以為 `>>>` 自動會補符號位

### 兩種修法
```verilog
// 方法 A:宣告 signed
output reg signed [63:0] q;
q <= q >>> 1;

// 方法 B:手動拼接補符號位(更穩)
q <= {q[63], q[63:1]};            // 右移 1,補 1 個 q[63]
q <= {{8{q[63]}}, q[63:8]};       // 右移 8,補 8 個 q[63]
```

### 關鍵心法
- 移位運算子整理:
  | 運算子 | 行為              | 何時用                |
  |--------|------------------|----------------------|
  | `>>`   | 補 0              | 邏輯右移              |
  | `>>>`  | 補符號位(需 signed)| 算術右移(有號除法)  |
  | `<<`   | 補 0              | 左移(邏輯/算術一樣)  |
  | `{}`   | 補你指定的東西    | 萬能,但移動量必須是常數    |

- **「能不能補非 0」要看 signed 宣告**,光靠 `>>>` 不夠
- 拼接 `{}` 是最保險的萬用解,但動量必須是常數

### Write your solution here
```verilog
module top_module(
    input clk,
    input load,
    input ena,
    input [1:0] amount,
    input [63:0] data,
    output reg signed [63:0] q); 
    always @(posedge clk) begin
        if (load)
            q <= data;
        else if(ena)
            case(amount)
            	2'b00: q <= q <<< 1;
                2'b01: q <= q <<< 8;
                2'b10: q <= q >>> 1; // q <= {q[63], q[63:1]}
                2'b11: q <= q >>> 8; // q <= {{8{q[63]}}, q[63:8]}
            endcase
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Shift Registers - shift register 1
<img width="874" height="333" alt="image" src="https://github.com/user-attachments/assets/29a090ee-d360-4103-85b2-b7ff205f4b9d" />

### 觀念:reset 有個圈圈 → active-low
- 圖上 FF 的 R 接腳前面有**小圓圈** → 代表 **0 才 reset**
- 訊號名稱 `resetn` 的「n」也是提示(negative-true)

### 我踩的雷
- ❌ 看到 reset 就反射寫 `if (resetn) q <= 0;` → 邏輯反了
- ✅ 應該寫 `if (!resetn) q <= 0;` 或 `if (resetn == 0)`

### 關鍵心法
- **圖上看到圈圈 = 反向 = active-low**,寫 code 要加 `!`
- 訊號名稱含 `n`、`_n`、`_b` 通常都是 active-low
- 看波形時也要注意:有些訊號平常是 1、變 0 才作用

### Write your solution here
```verilog
module top_module (
    input clk,
    input resetn,   // synchronous reset
    input in,
    output out);
    reg w1, w2, w3;
    always @(posedge clk)begin
        if (~resetn) begin
            w1  <= 0;
        	w2  <= 0;
        	w3  <= 0;
        	out <= 0;
        end
        else begin
            w1  <= in;
        	w2  <= w1;
       		w3  <= w2;
        	out <= w3;
        end
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Shift Registers - shift register 2
<img width="1562" height="953" alt="image" src="https://github.com/user-attachments/assets/e3961c41-436c-4ff5-b2d6-0aa64b082246" />

### 我踩的雷
- ❌ **順序看反了**:w 餵給 m0、Q[0] 在最左邊
- ✅ 實際是:**w 進到 m_{n-1}(最高位 Q_{n-1})**,往下移到 Q_0

### Write your solution here
```verilog
module top_module (
    input [3:0] SW,
    input [3:0] KEY,
    output [3:0] LEDR
); 
    MUXDFF m0(.clk(KEY[0]), .w(KEY[3]), .R(SW[3]), .E(KEY[1]), .L(KEY[2]), .Q(LEDR[3]));
    MUXDFF m1(.clk(KEY[0]), .w(LEDR[3]), .R(SW[2]), .E(KEY[1]), .L(KEY[2]), .Q(LEDR[2]));
    MUXDFF m2(.clk(KEY[0]), .w(LEDR[2]), .R(SW[1]), .E(KEY[1]), .L(KEY[2]), .Q(LEDR[1]));
    MUXDFF m3(.clk(KEY[0]), .w(LEDR[1]), .R(SW[0]), .E(KEY[1]), .L(KEY[2]), .Q(LEDR[0]));
    
endmodule

module MUXDFF (
    input clk,
    input w, R, E, L,
    output reg Q
);
    wire w1, w2;
    assign w1 = E ? w : Q;
    assign w2 = L ? R : w1;
    always @(posedge clk) begin
		Q <= w2;
    end
endmodule
```

---

## Circuits - Sequemtial Logic - Shift Registers - 3-input LUT
<img width="1568" height="242" alt="image" src="https://github.com/user-attachments/assets/f075f162-d06d-4d99-8b7c-386a3c4ba332" />

### 經典技巧
```verilog
assign Z = Q[{A, B, C}];   // 用拼接的值當 index
```
- `{A,B,C}` 拼成 3'bxxx 3-bit 數(0~7), 且當你拿一串 bit 當索引用,Verilog 只會把它當二進位整數解讀
- `Q[...]` 用那個數當索引 → **合成出來就是 8-to-1 mux**
- 一行頂八行 case

### Write your solution here
```verilog
module top_module (
    input clk,
    input enable,
    input S,
    input A, B, C,
    output Z ); 
    
    reg [7:0] Q;
    
    always @(posedge clk) begin
    	if (enable)
            Q <= {Q[6:0], S};
        else
            Q <= Q;
    end
    assign Z = Q[{A, B, C}];
endmodule
```

---

## Circuits - Sequemtial Logic - More Circuits - Rule 90
<img width="1570" height="718" alt="image" src="https://github.com/user-attachments/assets/5df66462-e717-40e6-8fb3-45def603eb60" />

**每個 clock = 一個時間步（圖上的一「列」）。**

```
時間 ↓ (clock 推，直向)        橫向 (for 跑每一格)
t=0   1                        ──────────────►
t=1   10                       clock 管「第幾列」
t=2   101                      for   管「一列裡每一格」
t=3   1000
```

## load 的角色（≈ 可指定載入值的 reset）
 
- `load` 是**輸入訊號**，值由 testbench 餵進來，你的 code 不能也不需要知道它是幾。
- 只要做到：**load=1 的那拍，把當下的 data 搬進 q**。
- `load` **每個 clock 都要判斷**，不是只在開頭觸發一次——它隨時可能再被拉高，要求重新載入。

### Write your solution here
```verilog
module top_module(
    input clk,
    input load,
    input [511:0] data,
    output reg [511:0] q ); 
    
    integer i;

    always @(posedge clk) begin
        if (load)
            q <= data;
        else begin
            for (i = 0; i < 512; i = i + 1) begin
                if (i == 0)
                    q[i] <= q[i + 1] ^ 1'b0;
                else if (i == 511)
                    q[i] <= 1'b0 ^ q[i - 1];
                else
                    q[i] <= q[i + 1] ^ q[i - 1];
            end
        end
    end
endmodule
```

### Rule 90（移位版，最簡潔）
```verilog
module top_module(
    input clk,
    input load,
    input [511:0] data,
    output reg [511:0] q
);
    always @(posedge clk) begin
        if (load)
            q <= data;
        else
            q <= {1'b0, q[511:1]} ^ {q[510:0], 1'b0};  // 左鄰 XOR 右鄰
    end
endmodule
```

# Rule 90 — 未化簡 SOP 版 Verilog
 
```verilog
module top_module(
    input clk,
    input load,
    input [511:0] data,
    output reg [511:0] q
);
    wire [511:0] L = {1'b0, q[511:1]};   // 左鄰
    wire [511:0] C = q;                   // 中
    wire [511:0] R = {q[510:0], 1'b0};   // 右鄰
    always @(posedge clk) begin
        if (load)
            q <= data;
        else
            // 未化簡 SOP：把 4 個 next=1 的 minterm OR 起來（含 C）
            q <= (~L & ~C &  R)    // 001
               | (~L &  C &  R)    // 011
               | ( L & ~C & ~R)    // 100
               | ( L &  C & ~R);   // 110
    end
endmodule
```

---

## Circuits - Sequemtial Logic - More Circuits - Rule 110
<img width="1562" height="647" alt="image" src="https://github.com/user-attachments/assets/345116dd-0d17-46af-9c52-28e6941c9865" />

→ 化簡：`next = (中 | 右) & ~(左 & 中 & 右)`

```verilog
// 左鄰居：整條往「右」移一格，最高位補 0（補的 0 = 邊界 q[512]=0）
{1'b0, q[511:1]}
 
// 右鄰居：整條往「左」移一格，最低位補 0（補的 0 = 邊界 q[-1]=0）
{q[510:0], 1'b0}
```

### Write your solution here
```verilog
module top_module(
    input clk,
    input load,
    input [511:0] data,
    output reg [511:0] q
);
    wire [511:0] L = {1'b0, q[511:1]};   // 左鄰
    wire [511:0] C = q;                   // 中
    wire [511:0] R = {q[510:0], 1'b0};   // 右鄰

    always @(posedge clk) begin
        if (load)
            q <= data;
        else
            // 直接把 5 個 next=1 的情況 OR 起來,完全照真值表
            q <= ( L & C & ~R)   // 110
               | ( L & ~C & R)   // 101
               | (~L & C & R)    // 011
               | (~L & C & ~R)   // 010
               | (~L & ~C & R);  // 001
    end
endmodule
```

### Rule 110（移位版）
```verilog
module top_module(
    input clk,
    input load,
    input [511:0] data,
    output reg [511:0] q
);
    wire [511:0] L = {1'b0, q[511:1]};   // 左鄰
    wire [511:0] C = q;                   // 中
    wire [511:0] R = {q[510:0], 1'b0};    // 右鄰
    always @(posedge clk) begin
        if (load) q <= data;
        else      q <= (C | R) & ~(L & C & R);
    end
endmodule
```

---

## Testbench 動態次數(repeat)

| 寫法 | N 可變數? | 用途 |
|------|-----------|------|
| `{N{x}}` 拼接 | ❌ 必須常數 | **電路**位寬要固定 |
| `repeat (N)` | ✅ 變數 OK | **執行**重複 N 次 |
| `for (...)` | ✅ | 更彈性 |

- **`{}` 是結構(電路位寬)、`repeat`/`for` 是行為(動作幾次)**,別混
- testbench 印 N 個值用 `repeat (N) $display(...);`
- 印 `$time` 用 **`%0g`**(避免醜空格),印變數用 **`%0d`**

---

# 🔧 核心觀念整理(精簡版)

## 賦值 & latch
- **`<=`**:時序邏輯(`@(posedge clk)`);**`=`**:組合邏輯(`@(*)`)
- **組合沒 else = latch**(要避免);**時序沒 else = 保持原值**(正常)
- 避免 latch:case 加 default,或進 case 前先給所有 output 預設值

## case 系列
- 三者都「從上往下,第一個匹配就停」
- **永遠用 `casez` + `?`**,別用 `casex`(x 當 don't care 會掩蓋 bug)

## 邏輯 vs 位元運算子

| 類型 | 運算子 | 結果寬度 | 用途 |
|------|--------|---------|------|
| **邏輯** | `&&` `\|\|` `!` `==` `!=` | 永遠 1-bit | 判斷真假 |
| **位元** | `&` `\|` `~` `^` | 跟輸入同寬 | 逐位運算 |

- **reduction**:`&a`(全 1?)`|a`(不為 0?)`^a`(奇偶性)
- **加減法器經典**:`{N{sub}} ^ b` + `cin = sub`

## 數字進位

**Verilog 寫法**(`位寬'進位 數值`):
- `'b` binary / `'o` octal / `'d` decimal / **`'h` hex**(硬體最常用)
- 位寬宣告 `[N:0]` 的 N 是十進位

**易雷**:
- `0x34`(C)→ Verilog 寫 **`8'h34`**
- `4'h10` 是 16,4-bit 裝不下 → 用 `4'hA`~`4'hF`
- **SystemVerilog**:`'0` = 全 0、`'1` = 全 1(自動配合位寬)

## 重複運算子 `{N{x}}`
- 雙層大括號:**`{N{內容}}`**,N 在外
- N **必須常數**(動態重複用 `repeat`)
- 內容寫 `1'b1` 明確位寬

## part-select(`+:` / `-:`)
```verilog
a[start +: width]    // 從 start 往高位取 width 個
a[start -: width]    // 從 start 往低位取 width 個
```
- 起點變數、寬度固定
- generate 內變數切片**唯一寫法**(Verilog 不准 `a[變數:變數]`)

## generate 通式

```verilog
genvar i;
generate
    for (i = 0; i < N; i = i + 1) begin : 標籤
        // 用三元處理邊界:i==0 ? cin : carry[i-1]
    end
endgenerate
```
- 必須 **`genvar`**(不是 integer)
- 必須 **`begin : 標籤`**
- 用 **`assign`**(不是 `=`)

## K-map 化簡

| 形式 | 圈什麼 | 0 對應 | 1 對應 | 項內 | 項間 |
|------|--------|--------|--------|------|------|
| **SOP** | 圈 1 | `~x` | `x` | `&` | `\|` |
| **POS** | 圈 0 | `x` | `~x` | `\|` | `&` |

- 圈大小必須 2 的次方,可繞行
- **棋盤格 = XOR**(奇偶校驗)
- K-map 軸是 **Gray code(00,01,11,10)**,後兩欄(ab=11 vs 10)易看反

## DFF / Latch / reset

| 類型 | 寫法 |
|------|------|
| **D Latch** | `always @(*) if (ena) q <= d;`(沒 else = latch) |
| **DFF** | `always @(posedge clk) q <= d;` |
| **同步 reset** | `@(posedge clk)`,reset 寫 if 裡 |
| **非同步 reset** | `@(posedge clk or posedge reset)` |
| **active-high** | `if (reset)` |
| **active-low** | `if (~reset)` |

- reset 重置值不一定是 0(可以 `q <= 8'h34;`)
- 負邊觸發:`@(negedge clk)`

## Signed overflow(8-bit)
- 判斷:**`(a[7] == b[7]) && (s[7] != a[7])`**(同號相加 + 結果反號)
- 二補數還原:**signed = unsigned − 2^N**

## module 實例化
- **named connection**:`.port(signal)` 優於 positional
- **每條 wire 恰好一個 driver**(避免 multiple drivers)
- 實例名**不要跟 module 同名**

## testbench
- `%0d` 印變數、`%0g` 印 `$time`
- 動態次數用 **`repeat (N)`**(N 可變數,`{N{x}}` 不行)

---

# ⚠️ 最常踩雷 Top 20

| # | 雷 |
|---|-----|
| 1 | 組合邏輯沒涵蓋所有路徑 → **latch** |
| 2 | `output reg` 忘了寫(always 驅動時) |
| 3 | `casex` 害人 → 一律 `casez` + `?` |
| 4 | 變數索引切片要用 **`a[i*4 +: 4]`**,不是 `a[i+3:i]` |
| 5 | 實例名跟 module 同名 |
| 6 | wire 沒驅動 / 重複驅動(multiple drivers) |
| 7 | **`4'h10` 不存在**(裝不下)→ 用 `4'hA`~`4'hF` |
| 8 | **`0x34` 是 C 語法**,Verilog 寫 `8'h34` |
| 9 | **`{N{x}}` 的 N 必須常數**,動態用 `repeat` |
| 10 | **同步 vs 非同步 reset**:差敏感列表有無 `or posedge reset` |
| 11 | **主動高 vs 主動低**:`if (reset)` vs `if (~reset)` |
| 12 | **POS 跟 SOP 規則完全相反**(0→x、項內 `\|`) |
| 13 | **signed overflow 必須看 s[7]** |
| 14 | **K-map 軸是 Gray code**,後兩欄(ab=11 vs 10)易看反 |
| 15 | BCD ≠ 二進位(超過 9 就進位) |
| 16 | 多 module 同名 → **改名為 `mod_a`/`mod_b`** |
| 17 | **`'0`/`'1`** 比 `{N{1'b1}}` 簡潔(SystemVerilog) |
| 18 | **加減法器**:`{N{sub}} ^ b` + `cin = sub` |
| 19 | **D Latch 故意沒 else**(平常避免、這題要) |
| 20 | **時序沒 else ≠ latch**(=保持原值,正常) |
| 21 | `~(A^B)` 是多 bit,1-bit 接收會被截斷 → 用 `&` reduction 或 `==` |
| 22 | 沒看 port 位寬:HDLBits 有些題用縮減位寬(`[3:1]`、`[2:0]`)處理邊界 |
| 23 | 拼接 `{}` 內常數必須有位寬(`{a, 11}` ⚠️ 11 變 32-bit) |
| 24 | gate primitive 只接訊號,不能放 `a&b` 運算式 |
