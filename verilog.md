## function vs task 的完整比較

| 項目 | `function` | `task` |
|------|-----------|--------|
| **輸入** | 只能有 input | input、output、inout 都可以 |
| **輸出** | 只能 1 個（透過函數名回傳） | 可以多個（透過 output 參數） |
| **呼叫方式** | `y = func(...)`（用等號接） | `task(...)`（直接呼叫，沒等號） |
| **可以有延遲嗎？**（`#10`、`@`、`wait`） | ❌ 不行 | ✅ 可以 |
| **執行時間** | 0 時間（不消耗模擬時間） | 可消耗時間 |
| **可以呼叫 task 嗎？** | ❌ 不行 | ✅ 可以呼叫 task 和 function |

## case / casez / casex 重點

- **沒給初始值時(訊號是 x):**
  - `casex`:**所有 pattern 都會匹配**,但只執行第一條就停,後面不會跑 ⚠️ → bug 被掩蓋
  - `casez`:x 不被當不在乎 → 不匹配,走 default ✅ 安全
  - `case`: x 嚴格比對 → 不匹配,走 default ✅ 安全

- **don't care 符號:**
  - `case`:不支援
  - `casez`:`?`、`z` 算 don't care
  - `casex`:`?`、`z`、`x` 都算 don't care(連訊號本身的 x 也算 → 危險)

- **結論:用 `casez` + `?`,別用 `casex`**

---

### 範例一:有給初值(假設 in = `4'b0100`)

**case** → 完全比對 `4'b0100`,沒這條 → 走 default,`out = 2'd0`

    case (in)               // in = 4'b0100
        4'b1000: out = 2'd3;
        4'b0100: out = 2'd2; // ✅ 完全匹配
        4'b0010: out = 2'd1;
        4'b0001: out = 2'd0;
        default: out = 2'd0;
    endcase
    // out = 2'd2

**casez** → 第二條 `4'b01??` 中,`out = 2'd2`

    casez (in)              // in = 4'b0100
        4'b1???: out = 2'd3;
        4'b01??: out = 2'd2; // ✅ 中
        4'b001?: out = 2'd1;
        4'b0001: out = 2'd0;
        default: out = 2'd0;
    endcase
    // out = 2'd2

**casex** → 第二條 `4'b01xx` 中,`out = 2'd2`

    casex (in)              // in = 4'b0100
        4'b1xxx: out = 2'd3;
        4'b01xx: out = 2'd2; // ✅ 中
        4'b001x: out = 2'd1;
        4'b0001: out = 2'd0;
        default: out = 2'd0;
    endcase
    // out = 2'd2

→ 正常值下,三種行為一樣 ✅

---

### 範例二:沒給初值(in = `4'bxxxx`)

**case** → x 嚴格比對,沒一條中 → 走 default,`out = 2'd0` ✅

    case (in)               // in = 4'bxxxx
        4'b1000: out = 2'd3;
        4'b0100: out = 2'd2;
        4'b0010: out = 2'd1;
        4'b0001: out = 2'd0;
        default: out = 2'd0; // ✅ 走這條
    endcase
    // out = 2'd0(安全)

**casez** → x 不被當不在乎,沒一條中 → 走 default,`out = 2'd0` ✅

    casez (in)              // in = 4'bxxxx
        4'b1???: out = 2'd3;
        4'b01??: out = 2'd2;
        4'b001?: out = 2'd1;
        4'b0001: out = 2'd0;
        default: out = 2'd0; // ✅ 走這條
    endcase
    // out = 2'd0(安全)

**casex** → 所有條都匹配,停在第一條 → `out = 2'd3` ⚠️

    casex (in)              // in = 4'bxxxx
        4'b1xxx: out = 2'd3; // ✅ 中這條就停,out = 2'd3
        4'b01xx: out = 2'd2; // 也匹配,但走不到
        4'b001x: out = 2'd1; // 也匹配,但走不到
        4'b0001: out = 2'd0; // 也匹配,但走不到
        default: out = 2'd0; // 永遠走不到 ⚠️
    endcase
    // out = 2'd3(bug 被掩蓋)
