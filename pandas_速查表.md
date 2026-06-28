# Pandas 速查表

> 給自己的查找用筆記。不用背,用到再翻。
> 範例資料假設是一個 `df`,有兩欄:`mRNA`(字串)、`mRNAs_count`(數字)、`miRNAs`(字串,可能一格多值)。

---

## 目錄

1. [讀檔 / 看資料](#1-讀檔--看資料)
2. [取欄位 / 取列](#2-取欄位--取列)
3. [篩選(條件過濾)](#3-篩選條件過濾)
4. [排序](#4-排序)
5. [分組統計 groupby](#5-分組統計-groupby)
6. [字串處理](#6-字串處理)
7. [新增 / 修改欄位](#7-新增--修改欄位)
8. [處理空值](#8-處理空值)
9. [去重 / 不重複值](#9-去重--不重複值)
10. [輸出存檔](#10-輸出存檔)
11. [list 寫法 vs pandas 寫法對照](#11-list-寫法-vs-pandas-寫法對照)
12. [最常踩的雷](#12-最常踩的雷)
13. [先熟這幾個就夠用](#13-先熟這幾個就夠用)

---

## 1. 讀檔 / 看資料

每次都從這裡開始,先確認讀對了再做事。

```python
import pandas as pd

df = pd.read_csv("input.csv")        # 讀 CSV
df = pd.read_csv("input.csv", sep="\t")   # 讀 TSV(用 tab 分隔)

df.head()        # 看前 5 列(最常用,確認讀對)
df.head(10)      # 看前 10 列
df.tail()        # 看後 5 列

df.shape         # (列數, 欄數),例如 (1000, 3)
df.columns       # 有哪些欄位名
df.dtypes        # 每欄的型別
df.info()        # 型別 + 有沒有空值,綜合資訊
df.describe()    # 數字欄的統計摘要(平均、最大、最小...)
```

---

## 2. 取欄位 / 取列

```python
# --- 取欄位 ---
df["mRNAs_count"]              # 取單欄 → 回傳 Series(一欄)
df[["miRNAs", "mRNAs_count"]] # 取多欄 → 回傳 DataFrame(注意兩層中括號)

# --- 取列(用位置 iloc) ---
df.iloc[0]        # 第 0 列
df.iloc[0:5]      # 前 5 列(第 0~4)
df.iloc[-1]       # 最後一列

# --- 取某一格 ---
df.iloc[0]["miRNAs"]     # 第 0 列的 miRNAs 欄
df.at[0, "miRNAs"]       # 同上,更快的寫法
```

> 取單欄出來叫 **Series**(一欄資料 + 索引);取多欄出來還是 **DataFrame**(表格)。

---

## 3. 篩選(條件過濾)

**這是 `filter_by_count` 的核心。**

```python
# 單一條件
df[df["mRNAs_count"] >= 5]        # 保留 count >= 5 的列
df[df["mRNAs_count"] == 1]        # 等於 1
df[df["miRNAs"] == "cel-miR-1"]   # 字串相等

# 多條件:and 用 & , or 用 | , 每個條件要用 () 包起來
df[(df["mRNAs_count"] >= 5) & (df["mRNAs_count"] <= 10)]   # 介於 5~10
df[(df["mRNAs_count"] == 1) | (df["mRNAs_count"] == 2)]    # 1 或 2

# 取反:not 用 ~
df[~(df["mRNAs_count"] >= 5)]     # 不要 count >= 5 的

# 某欄的值在一組清單裡(很好用)
df[df["miRNAs"].isin(["cel-miR-1", "cel-miR-2"])]

# 篩完通常加 reset_index 把索引重排
df[df["mRNAs_count"] >= 5].reset_index(drop=True)
```

> **原理**:`df["mRNAs_count"] >= 5` 會對整欄逐格比較,產生一排 True/False;
> 外面的 `df[...]` 再把 True 的列留下來。你可以想成「一個 mask 遮罩」。

---

## 4. 排序

```python
df.sort_values("mRNAs_count")                     # 由小到大
df.sort_values("mRNAs_count", ascending=False)    # 由大到小
df.sort_values(["miRNAs", "mRNAs_count"])         # 先按 miRNAs 再按 count
df.sort_values("mRNAs_count").reset_index(drop=True)   # 排完重排索引
```

---

## 5. 分組統計 groupby

**reverse 那段的核心。** 概念:把資料「按某欄分堆」,再對每堆做計算。

```python
# 每組幾筆
df.groupby("miRNAs")["mRNA"].count()

# 每組加總 / 平均 / 最大
df.groupby("miRNAs")["mRNAs_count"].sum()
df.groupby("miRNAs")["mRNAs_count"].mean()
df.groupby("miRNAs")["mRNAs_count"].max()

# 一次算多個(最實用,你已經在用這個)
df.groupby("miRNAs").agg(
    mRNA_count=("mRNA", "count"),                    # 數個數
    mRNA_list=("mRNA", lambda x: ",".join(x))        # 串成字串
).reset_index()
```

`agg` 裡的格式是:`新欄名=("要算的舊欄", "怎麼算")`。
常用的「怎麼算」:`"count"`、`"sum"`、`"mean"`、`"max"`、`"min"`、`"nunique"`(不重複個數)。

```python
# 如果要「不重複」的個數和清單(去掉重複的 mRNA)
df.groupby("miRNAs").agg(
    mRNA_count=("mRNA", "nunique"),
    mRNA_list=("mRNA", lambda x: ",".join(sorted(set(x))))
).reset_index()
```

---

## 6. 字串處理

**拆 miRNAs 那段用的。** 對「整欄字串」操作,前面都要加 `.str`。

```python
df["miRNAs"].str.split(",")        # 用逗號拆開(每格變成一個 list)
df["miRNAs"].str.strip()           # 去頭尾空白
df["miRNAs"].str.replace(",", "")  # 取代
df["miRNAs"].str.upper()           # 轉大寫
df["miRNAs"].str.lower()           # 轉小寫
df["miRNAs"].str.contains("miR")   # 是否包含某字串 → True/False
df["miRNAs"].str.len()             # 每格字串長度

# explode:把「一格多值」攤平成多列(你已經在用)
df["miRNAs"] = df["miRNAs"].str.split(",")
df = df.explode("miRNAs")          # 一格 ["a","b"] → 變成兩列 a 和 b
df["miRNAs"] = df["miRNAs"].str.strip()
```

`explode` 前後對照:

```
拆開前:
  mRNA    miRNAs
  geneA   a, b, c

explode 後:
  mRNA    miRNAs
  geneA   a
  geneA   b
  geneA   c
```

---

## 7. 新增 / 修改欄位

```python
df["double"] = df["mRNAs_count"] * 2          # 用既有欄算出新欄
df["flag"] = df["mRNAs_count"] >= 5           # 算出一欄 True/False
df["total"] = df["a"] + df["b"]               # 兩欄相加

# 用條件給不同的值(np.where:類似 if-else)
import numpy as np
df["level"] = np.where(df["mRNAs_count"] >= 5, "high", "low")

# 改欄位名
df = df.rename(columns={"舊名": "新名"})

# 刪欄位
df = df.drop(columns=["不要的欄"])
```

---

## 8. 處理空值

```python
df.isna()                 # 哪些格是空值 → 整表 True/False
df["x"].isna()            # 某欄哪些是空值
df.isna().sum()           # 每欄有幾個空值(很常用來檢查)

df.dropna()               # 丟掉「有空值」的列
df.dropna(subset=["x"])   # 只看 x 欄有空值的才丟
df.fillna(0)              # 把空值填成 0
df["x"].fillna("未知")     # 某欄空值填字串
```

---

## 9. 去重 / 不重複值

```python
df["miRNAs"].unique()        # 不重複的值(回傳陣列)
df["miRNAs"].nunique()       # 有幾個不重複的值
df["miRNAs"].value_counts()  # 每個值各出現幾次(超好用)

df.drop_duplicates()                    # 丟掉完全重複的列
df.drop_duplicates(subset=["miRNAs"])   # 只看 miRNAs 欄重複就丟
```

---

## 10. 輸出存檔

```python
df.to_csv("output.csv", index=False)                       # 存 CSV,不要索引欄
df.to_csv("output.csv", index=False, encoding="utf-8-sig") # 中文不亂碼(Excel 友善)

# 搭配 pathlib 組路徑(你的 pipeline 用法)
from pathlib import Path
df.to_csv(job_dir / "output" / "output.csv", index=False, encoding="utf-8-sig")
```

> `index=False` 幾乎一定要加,不然會多出一欄沒用的編號。

---

## 11. list 寫法 vs pandas 寫法對照

給習慣用 list 思考的你。同一件事,兩種寫法,先用看得懂的那種。

### 篩選出 count >= 5 的資料

```python
# --- 純 list / for 寫法(你熟的) ---
result = []
for i in range(len(df)):
    if df.iloc[i]["mRNAs_count"] >= 5:
        result.append(df.iloc[i])

# --- pandas 寫法(一行) ---
result = df[df["mRNAs_count"] >= 5]
```

### 算某欄總和

```python
# --- list 寫法 ---
total = 0
for x in df["mRNAs_count"]:
    total += x

# --- pandas 寫法 ---
total = df["mRNAs_count"].sum()
```

### 每個值出現幾次

```python
# --- list / dict 寫法 ---
counts = {}
for x in df["miRNAs"]:
    counts[x] = counts.get(x, 0) + 1

# --- pandas 寫法 ---
counts = df["miRNAs"].value_counts()
```

> 兩邊結果一樣。pandas 只是把迴圈藏在底層用 C 跑,資料量大時快很多。
> **學習階段用 list 刻一遍懂原理,正式跑大資料用 pandas。**

---

## 12. 最常踩的雷

1. **多條件篩選**:`and / or` 不能用,要用 `&` `|`,而且每個條件包 `()`。
   ```python
   df[(df["a"] >= 5) & (df["b"] < 10)]   # ✅
   df[df["a"] >= 5 and df["b"] < 10]     # ❌ 會報錯
   ```

2. **存檔多一欄編號**:忘了加 `index=False`。

3. **字串操作忘了 `.str`**:
   ```python
   df["miRNAs"].str.split(",")   # ✅
   df["miRNAs"].split(",")       # ❌
   ```

4. **read_csv 路徑反斜線**:Windows 用 `/` 或在字串前加 `r`。
   ```python
   pd.read_csv(r"C:\python\input.csv")   # ✅
   pd.read_csv("C:\python\input.csv")    # ❌ \p 可能出錯
   ```

5. **改了 df 卻沒存回去**:很多操作回傳新的,不會改原本的。
   ```python
   df.dropna()        # ❌ 沒效果(結果丟掉了)
   df = df.dropna()   # ✅ 存回去
   ```

---

## 13. 先熟這幾個就夠用

不用全會。八成狀況靠這幾個組合就能解決,你的 pipeline 也是它們拼出來的:

| 動作 | 語法 |
|------|------|
| 讀檔 | `pd.read_csv("input.csv")` |
| 看一眼 | `df.head()` |
| 取欄位 | `df["欄名"]` |
| 篩選 | `df[df["欄名"] >= n]` |
| 分組統計 | `df.groupby("欄名").agg(...)` |
| 存檔 | `df.to_csv("out.csv", index=False)` |

其他的,要用的時候回來查這份就好。
