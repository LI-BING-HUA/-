# ⚡ Verilog 心法速查(一頁版)

## 🔴 最常踩(每次寫都檢查)

**reg / wire / 記憶**
- assign 配 wire、always 配 reg
- reg ≠ 記憶!記憶看敏感列表:`@(posedge clk)`有、`@(*)`沒有
- `always @(*)`+reg 和 assign 都是組合無記憶

**Multiple Driver(一訊號只能一個驅動者)**
- assign / always / 子模組輸出,三選一,不可混
- 衝突就插中間 wire 分開
- generate 每圈打同一條 → 撞多次;該用「一條 assign + 來源全 OR」

**晚一拍(時序核心)**
- 組合(assign、`@(*)`)= clk 一到就反映,當拍生效
- always 的 `<=` = 要等邊緣結束才寫入 → 晚一拍
- next_state 同理:這拍算、下拍才變(「讀到1當下還是A」就是這個)

## 🔴 計數器 / datapath

**計數器每次用前歸零**(最容易忘)
- 不能只靠 reset 顧開頭;每次重新使用都要乾淨
- 封包 count、splat fall_clk 都栽在這

**datapath 跟 state 分塊寫**
- count / shift_reg / 方向reg / accumulator / prev / flag → 各自獨立塊
- 判準:更新條件不同就分塊(每拍變 vs 看條件變、不同邊緣、不同時脈)
- 純 counter(沒 FSM)才合一塊

## 🔴 FSM

- **優先級逐層攔截**:if/else if 鏈,高優先(reset/fall/邊界)寫前面;別用獨立 if(會互相覆蓋)
- **要記的歷史 → 編進狀態(展開)或放 reg(摺疊)**;題目要 Moore 用展開版
- **trap state**(死狀態):`next_state = 自己` 無條件(別寫條件,藏 latch)
- one-hot:用 `state[索引]` 讀位元,別用 `==` 整值(防非法輸入)

## 🟡 數字 / 位寬

- **真計數用 `'d`、BCD 用 `'h`**(fall_clk 寫 5'd21,別寫 5'h15 會看成15)
- 位寬要夠:`[3:0]` 配 `3'd8` 會截成 0
- `||` `&&` 是邏輯(結果1bit),不能列舉多值:要 `state==DL || state==DR`(不是 `state==(DL||DR)`)
- 邏輯運算子 `&&|| !` 結果永遠1bit;位元 `&|~^` 跟輸入同寬

## 🟡 reset / latch / 移位

- reset 同步:敏感列表只有 clk;非同步:加 `or posedge reset`
- active-low(resetn、有圈圈)→ `if(!resetn)`
- 組合 latch:case 加 default 或進 case 前給預設值
- `>>>` 算術右移要 signed 才補符號位;`{}` 拼接最保險(動量需常數)

## 🟢 實作 / 工具

- assign 驅動的 output 別宣告 reg
- Vivado 紅波浪線常誤報,真錯看 Tcl Console / elaborate.log
- 用別人 tb:內部訊號名要跟 tb 的階層存取一致(top.cnt → 你要叫 cnt)
- 模擬慢:分頻門檻改小,上板再改回
- XDC:看得懂就好,port 名要跟 code 完全一致
