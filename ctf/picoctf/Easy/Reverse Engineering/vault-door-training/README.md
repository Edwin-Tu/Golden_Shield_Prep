# picoCTF Reverse Engineering Writeup
## 題目：VaultDoorTraining

---

## 題目類型

- Reverse Engineering
- Source Code Inspection
- Java

---

## 題目核心

這題的重點非常直接：

> **密碼直接寫在 Java 原始碼裡。**

也就是說，這題不是要你爆破，也不是要你真正逆向複雜邏輯，
而是要你先學會：

- 看懂程式流程
- 找出使用者輸入如何被處理
- 找到真正被比對的密碼字串

---

## 題目原始碼重點

題目中的關鍵程式如下：

```java
String userInput = scanner.next();
String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
if (vaultDoor.checkPassword(input)) {
    System.out.println("Access granted.");
} else {
    System.out.println("Access denied!");
}
```

接著在 `checkPassword()` 中：

```java
public boolean checkPassword(String password) {
    return password.equals("w4rm1ng_Up_w1tH_jAv4_000iPnsaWOY");
}
```

---

## 程式流程分析

### 1. 使用者輸入整段字串

程式要求你輸入：

```text
Enter vault password:
```

你輸入的內容預期格式其實是：

```text
picoCTF{某段密碼}
```

---

### 2. 程式把外層 `picoCTF{` 和最後的 `}` 拿掉

這一行很重要：

```java
String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
```

它的意思是：

- 從 `picoCTF{` 後面開始取
- 取到最後一個 `}` 的前一個字元為止

也就是說：

```text
picoCTF{abc123}
```

會被切成：

```text
abc123
```

---

### 3. 真正比對的是裡面的內容

後面呼叫：

```java
vaultDoor.checkPassword(input)
```

而 `checkPassword()` 裡寫死了正確密碼：

```java
"w4rm1ng_Up_w1tH_jAv4_000iPnsaWOY"
```

所以只要你輸入的內容去掉外殼後，剛好等於這串字，就會通過。

---

## 正確解法

因為程式真正要比對的是：

```text
w4rm1ng_Up_w1tH_jAv4_000iPnsaWOY
```

而你輸入時又必須保留題目格式，所以最終應輸入：

```text
picoCTF{w4rm1ng_Up_w1tH_jAv4_000iPnsaWOY}
```

---

## 最終 Flag

```text
picoCTF{w4rm1ng_Up_w1tH_jAv4_000iPnsaWOY}
```

---

## 為什麼這題能這樣解？

因為題目犯了一個很典型的安全問題：

> **把密碼直接寫在前端 / 原始碼 / 可讀程式碼裡**

雖然這裡是 Java 原始碼，不是網頁前端，
但概念一樣：

- 只要使用者拿得到原始碼
- 就可以直接看到密碼
- 驗證機制等於失效

---

## 這題學到的重點

### 1. 不要急著猜答案，先看流程
遇到 Reverse 題，不一定一開始就要跑 debugger。  
先做這些事通常更快：

- 找 `main()`
- 找輸入點
- 找驗證函式
- 找比較邏輯

---

### 2. 注意字串切割
這題最容易忽略的是：

```java
substring("picoCTF{".length(), userInput.length()-1)
```

這表示：

- 程式不會直接拿整串 `picoCTF{...}` 去比
- 它只會比中間那段內容

所以最後答案雖然是 flag 格式，
但你要理解「真正的密碼」其實是大括號中的內容。

---

### 3. 硬編碼密碼（hardcoded password）是常見弱點
只要看到這種：

```java
password.equals("某個固定字串")
```

通常就代表：

- 題目要你直接讀出密碼
- 不需要做複雜運算
- 重點在程式閱讀能力

---

## 精簡版解題步驟

### Step 1：打開原始碼
找到 `checkPassword()`。

### Step 2：找到比對字串
看到：

```java
return password.equals("w4rm1ng_Up_w1tH_jAv4_000iPnsaWOY");
```

### Step 3：補回 flag 外殼
因為主程式會把 `picoCTF{` 和 `}` 去掉再比，
所以正確輸入是：

```text
picoCTF{w4rm1ng_Up_w1tH_jAv4_000iPnsaWOY}
```

---

## 一句話總結

這題不是在「破解密碼」，而是在考你能不能：

> **從原始碼中直接找出硬編碼的密碼，並理解輸入格式怎麼被切割。**
