# 📚 Verilog / HDLBits 學習總整理(完整版)

---
難度:🔴 觀念類(理解卡關) ｜ 🟡 細節類(粗心 / 邊界) ｜ 🟢 語法類(寫法不熟)
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

for 迴圈不能直接寫在 module 裡,要放在:
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
