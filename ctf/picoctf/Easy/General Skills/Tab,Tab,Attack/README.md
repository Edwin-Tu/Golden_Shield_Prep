# picoCTF Reverse Engineering Writeup
## 題目：Addadshashanammu

---

## 一、題目概述

這題是 picoCTF 中很經典的入門逆向題。

它的設計重點不是複雜的反組譯，而是要你學會：

- 解壓縮題目檔案
- 在多層目錄中尋找可疑檔案
- 辨認原始碼檔與可執行檔
- 直接從原始碼中讀出 flag

這類題目很適合剛接觸 Reverse Engineering 的人，因為它同時練到：

- Linux 基本操作
- 檔案搜尋能力
- 題型判斷能力
- 原始碼閱讀能力

---

## 二、你這次的實際解題歷程

你一開始先查看目前資料夾內容：

```bash
ls
```

輸出：

```text
Addadshashanammu.zip  README.md  desktop.ini
```

這代表題目主要檔案是：

```text
Addadshashanammu.zip
```

---

## 三、Step 1：先解壓縮題目檔案

你執行：

```bash
unzip Addadshashanammu.zip
```

系統輸出顯示：

```text
creating: Addadshashanammu/
creating: Addadshashanammu/Almurbalarammi/
creating: Addadshashanammu/Almurbalarammi/Ashalmimilkala/
creating: Addadshashanammu/Almurbalarammi/Ashalmimilkala/Assurnabitashpi/
creating: Addadshashanammu/Almurbalarammi/Ashalmimilkala/Assurnabitashpi/Maelkashishi/
creating: Addadshashanammu/Almurbalarammi/Ashalmimilkala/Assurnabitashpi/Maelkashishi/Onnissiralis/
creating: Addadshashanammu/Almurbalarammi/Ashalmimilkala/Assurnabitashpi/Maelkashishi/Onnissiralis/Ularradallaku/
extracting: Addadshashanammu/Almurbalarammi/Ashalmimilkala/Assurnabitashpi/Maelkashishi/Onnissiralis/Ularradallaku/fang-of-haynekhtnamet.c
inflating: Addadshashanammu/Almurbalarammi/Ashalmimilkala/Assurnabitashpi/Maelkashishi/Onnissiralis/Ularradallaku/fang-of-haynekhtnamet
```

### 分析重點

這一步透露了幾個重要資訊：

1. 題目故意把檔案藏在很多層資料夾底下  
2. 最深處有兩個重要檔案：
   - `fang-of-haynekhtnamet.c`
   - `fang-of-haynekhtnamet`

這通常代表：

- `.c` 是 C 原始碼
- 沒副檔名的那個通常是編譯後的可執行檔

---

## 四、Step 2：逐層進入目錄

你接著一路用 `cd` 進入資料夾：

```bash
cd Addadshashanammu/
cd Almurbalarammi/
cd Ashalmimilkala/
cd Assurnabitashpi/
cd Maelkashishi/
cd Onnissiralis/
cd Ularradallaku/
```

最後用：

```bash
ls -a
```

看到：

```text
.  ..  fang-of-haynekhtnamet  fang-of-haynekhtnamet.c
```

### 分析重點

這代表你已經成功走到題目真正藏東西的最深層位置。

這裡出現兩個檔案時，逆向題常見的做法有兩條路：

### 路線 A：先看原始碼
如果有 `.c`、`.py`、`.java` 這類可直接閱讀的檔案，
通常先看原始碼最快。

### 路線 B：分析執行檔
如果只有二進位檔，
才會進一步考慮：

- `strings`
- `file`
- `objdump`
- `gdb`
- Ghidra

你這題因為有 `.c`，所以直接看原始碼是最有效率的。

---

## 五、Step 3：查看 C 原始碼

你執行：

```bash
cat fang-of-haynekhtnamet.c
```

看到內容：

```c
#include <stdio.h>

int main(){
printf("*ZAP!* picoCTF{l3v3l_up!_t4k3_4_r35t!_fc588427}\n");
}
```

---

## 六、Step 4：找出 flag

程式非常簡單，`main()` 中直接：

```c
printf("*ZAP!* picoCTF{l3v3l_up!_t4k3_4_r35t!_fc588427}\n");
```

這表示：

- 程式執行後會直接印出 flag
- 而且 flag 已經硬編碼（hardcoded）在原始碼裡

所以最終 flag 就是：

```text
picoCTF{l3v3l_up!_t4k3_4_r35t!_fc588427}
```

---

## 七、最終答案

```text
picoCTF{l3v3l_up!_t4k3_4_r35t!_fc588427}
```

---

## 八、這題真正想教你的技能

這題表面上很簡單，但其實在訓練你幾個很重要的 Reverse 基本功。

### 1. 不要一看到逆向就先想很複雜
很多初學者看到「Reverse Engineering」就會馬上想到：

- 反組譯
- 組語
- debugger
- Ghidra

但這題告訴你：

> **先看題目給了什麼，再決定工具。**

如果題目已經把 `.c` 原始碼給你，最優先就是直接讀原始碼。

---

### 2. 熟悉 Linux 基本操作
這題練到很多基礎指令：

- `ls`：列出檔案
- `unzip`：解壓縮
- `cd`：切換資料夾
- `cat`：查看檔案內容
- `ls -a`：查看包含隱藏資訊在內的完整清單

這些都是 CTF 非常常用的基本功。

---

### 3. 題目可能把答案藏在很深的路徑
這題用了很多層目錄名稱，目的就是讓你習慣：

- 觀察解壓後的目錄結構
- 不要怕一路往下找
- 題目的重點有時不在演算法，而在「找到檔案」

---

### 4. 學會辨認檔案類型
看到：

```text
fang-of-haynekhtnamet.c
fang-of-haynekhtnamet
```

你就要有這種反應：

- `.c`：原始碼，先看
- 無副檔名：很可能是 Linux 可執行檔

這種判斷能力在 Reverse 題很重要。

---

## 九、如果沒有原始碼，還能怎麼做？

雖然這題直接看 `.c` 就能解，但你也可以補充學會：

### 方法 1：直接執行程式
如果檔案可執行，可以試：

```bash
./fang-of-haynekhtnamet
```

可能就會直接印出 flag。

---

### 方法 2：看字串
如果不能直接讀原始碼，也可以試：

```bash
strings fang-of-haynekhtnamet
```

很多簡單題的 flag 會直接出現在可執行檔字串中。

---

### 方法 3：先確認檔案型態
可以用：

```bash
file fang-of-haynekhtnamet
```

判斷它是不是：

- ELF executable
- script
- text file

---

## 十、這題的通用解題流程

你以後遇到類似題目，可以照這套流程走：

### Step 1：先看有什麼檔案
```bash
ls
```

### Step 2：如果是壓縮檔就先解壓
```bash
unzip 題目.zip
```

### Step 3：觀察解壓後目錄結構
```bash
ls
cd 資料夾
ls -a
```

### Step 4：找可疑檔案
優先注意：

- `.c`
- `.cpp`
- `.py`
- `.java`
- `.js`
- 無副檔名執行檔

### Step 5：如果有原始碼，先看原始碼
```bash
cat 檔名
```

### Step 6：找關鍵字
像這些都很值得注意：

- `printf`
- `flag`
- `picoCTF`
- `main`
- `password`
- `strcmp`

### Step 7：確認 flag
如果程式直接印出或比對 flag，就可直接整理答案。

---

## 十一、初學者常見錯誤

### 1. 一開始就想用很重的工具
這題完全不需要 Ghidra、IDA、gdb。

如果你一開始就走太重的路，反而浪費時間。

---

### 2. 沒注意到已經有原始碼
有些人看到執行檔就只盯著執行檔，
卻忽略同資料夾裡其實有 `.c`。

這題最重要的判斷就是：

> **先看可直接閱讀的檔案。**

---

### 3. 看到很深的資料夾就以為很複雜
其實這題很多層目錄只是障眼法。  
真正有價值的資訊在最底層，而且原始碼只有幾行。

---

## 十二、這題學到的逆向觀念總結

這題雖然簡單，但它是很好的 Reverse 入門練習，因為它讓你熟悉：

- 如何從壓縮檔開始分析
- 如何在多層目錄中定位目標檔案
- 如何辨認原始碼與執行檔
- 如何直接從原始碼找 flag
- 如何避免把題目想得過度複雜

---

## 十三、一句話總結

> **這題的核心不是破解，而是先找到最底層的 C 原始碼，然後直接從 `printf()` 中讀出 flag。**

---

## 十四、精簡版解法

```bash
unzip Addadshashanammu.zip
cd Addadshashanammu/Almurbalarammi/Ashalmimilkala/Assurnabitashpi/Maelkashishi/Onnissiralis/Ularradallaku
cat fang-of-haynekhtnamet.c
```

看到：

```c
printf("*ZAP!* picoCTF{l3v3l_up!_t4k3_4_r35t!_fc588427}\n");
```

所以 flag 是：

```text
picoCTF{l3v3l_up!_t4k3_4_r35t!_fc588427}
```
