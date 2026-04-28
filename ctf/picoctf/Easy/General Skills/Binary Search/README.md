# picoCTF Binary Search Game 解題教學

## 題目簡介

本題是 picoCTF 的 Binary Search Game。題目會透過 SSH 連線進入遠端主機，系統會隨機產生一個介於 `1 ~ 1000` 的數字，玩家需要在限定次數內猜出正確答案。

遊戲會根據輸入的數字給出提示：

- `Higher! Try again.`：代表答案比目前猜的數字還要大
- `Lower! Try again.`：代表答案比目前猜的數字還要小
- `Congratulations!`：代表猜中答案，並顯示 flag

本題的核心解法是 **Binary Search（二分搜尋法）**。

---

## 相關檔案

本題提供的程式檔案為：

```bash
guessing_game.sh
```

程式內容會隨機產生一個 `1 ~ 1000` 之間的目標數字：

```bash
target=$(( (RANDOM % 1000) + 1 ))
```

並設定玩家最多只能猜 `10` 次：

```bash
MAX_GUESSES=10
guess_count=0
```

如果玩家猜中正確數字，程式會從 `/challenge/metadata.json` 讀取 flag 並印出：

```bash
flag=$(cat /challenge/metadata.json | jq -r '.flag')
echo "Here's your flag: $flag"
```

---

## 程式邏輯分析

### 1. 產生隨機答案

程式會使用 Bash 的 `$RANDOM` 變數產生隨機數字，並透過取餘數 `% 1000` 限制範圍，再加上 `1`，使答案範圍落在：

```text
1 ~ 1000
```

---

### 2. 限制猜測次數

程式設定最多只能猜 10 次：

```bash
while (( guess_count < MAX_GUESSES )); do
```

每次輸入有效數字後，猜測次數都會加 1：

```bash
(( guess_count++ ))
```

如果超過 10 次仍未猜中，連線會結束：

```bash
echo "Sorry, you've exceeded the maximum number of guesses."
exit 1
```

---

### 3. 判斷輸入是否合法

如果輸入的不是數字，程式會要求重新輸入：

```bash
if ! [[ "$guess" =~ ^[0-9]+$ ]]; then
    echo "Please enter a valid number."
    continue
fi
```

這代表輸入英文字母、符號或空白都不會被接受。

---

### 4. 根據大小給提示

程式會比較玩家輸入的數字與目標數字：

```bash
if (( guess < target )); then
    echo "Higher! Try again."
elif (( guess > target )); then
    echo "Lower! Try again."
else
    echo "Congratulations! You guessed the correct number: $target"
fi
```

因此我們可以根據提示不斷縮小答案範圍。

---

## 解題觀念：Binary Search（二分搜尋）

因為答案範圍是 `1 ~ 1000`，而且最多只能猜 10 次，所以不能用從 1 慢慢猜到 1000 的方式。

正確作法是使用二分搜尋：

1. 先猜範圍中間值
2. 如果提示 `Higher`，代表答案在右半邊
3. 如果提示 `Lower`，代表答案在左半邊
4. 重複縮小範圍，直到猜中答案

由於：

```text
2^10 = 1024
```

所以在 `1 ~ 1000` 的範圍內，只要每次都使用二分搜尋，理論上 10 次內一定可以猜到答案。

---

## 實際操作紀錄

### SSH 連線

使用以下指令連線到 picoCTF 遠端主機：

```bash
ssh -p 59765 ctf-player@atlas.picoctf.net
```

第一次連線時，系統會詢問是否信任該主機：

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

輸入：

```text
yes
```

接著輸入密碼後，進入遊戲畫面：

```text
Welcome to the Binary Search Game!
I'm thinking of a number between 1 and 1000.
```

---

## 猜測過程

實際猜測紀錄如下：

| 次數 | 猜測數字 | 系統提示 | 判斷結果 |
|---:|---:|---|---|
| 1 | 500 | Higher | 答案大於 500 |
| 2 | 700 | Higher | 答案大於 700 |
| 3 | 800 | Lower | 答案小於 800 |
| 4 | 750 | Lower | 答案小於 750 |
| 5 | 725 | Higher | 答案大於 725 |
| 6 | 740 | Higher | 答案大於 740 |
| 7 | 745 | Lower | 答案小於 745 |
| 8 | 74 | Higher | 答案大於 74，這次輸入應該是誤打 |
| 9 | 743 | Correct | 猜中答案 |

雖然第 8 次輸入 `74` 應該是誤打，但仍然在 10 次限制內成功猜中答案。

---

## 答案範圍縮小過程

根據提示可以逐步縮小範圍：

```text
500 → Higher  → 答案 > 500
700 → Higher  → 答案 > 700
800 → Lower   → 答案 < 800
750 → Lower   → 答案 < 750
725 → Higher  → 答案 > 725
740 → Higher  → 答案 > 740
745 → Lower   → 答案 < 745
```

因此答案範圍會被縮小到：

```text
741 ~ 744
```

最後猜測：

```text
743
```

成功猜中。

---

## 成功取得 Flag

猜中後系統回傳：

```text
Congratulations! You guessed the correct number: 743
Here's your flag: picoCTF{g00d_gu355_bee04a2a}
```

最後取得的 flag 為：

```text
picoCTF{g00d_gu355_bee04a2a}
```

---

## 解題重點整理

- 題目限制最多只能猜 10 次
- 答案範圍是 `1 ~ 1000`
- 不能用暴力法一個一個猜
- 必須使用二分搜尋快速縮小範圍
- `Higher` 代表答案更大
- `Lower` 代表答案更小
- 猜中後即可取得 flag

---

## 心得

這題主要是訓練 Binary Search 的基礎觀念。雖然題目看起來只是猜數字遊戲，但因為有猜測次數限制，所以必須用有效率的方法解題。

透過這題可以理解：

- 如何根據提示縮小搜尋範圍
- 為什麼二分搜尋比線性搜尋更有效率
- 在有限次數內如何設計最佳猜測策略

這也是 CTF 中常見的思考方式：不要盲目嘗試，而是要根據程式邏輯與回饋資訊，逐步推理出正確答案。
