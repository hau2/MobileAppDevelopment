---
sidebar_position: 10
---

Tuyệt vời lớp ơi 👏👏👏
Các em học rất chắc nền, và hôm nay chúng ta bước sang một chủ đề **rất thú vị, quan trọng và “đẳng cấp” hơn**:
👉 **Bài 10 – Closures & Phạm vi biến (Scope) trong Swift.**

---

# 🧩 BÀI 10: CLOSURES & PHẠM VI BIẾN (SCOPE) TRONG SWIFT

---

## 🎯 **Mục tiêu bài học**

Sau bài này, bạn sẽ:

1. Hiểu khái niệm **closure** và cách nó khác với hàm thông thường.
2. Biết cách khai báo, truyền, và sử dụng closures trong Swift.
3. Hiểu rõ **phạm vi biến (scope)** – khi nào biến “tồn tại” hoặc “mất đi”.
4. Biết **capture values** trong closure – cơ chế nền tảng của SwiftUI, UIKit animation, và async code.

---

## 🧠 **1. Closures là gì?**

> **Closure** là một “hàm vô danh” — tức là **hàm không có tên**, có thể **truyền như một giá trị**, **lưu trữ trong biến**, hoặc **trả về từ hàm khác**.

Nói dễ hiểu:
Closures = **Function + Khả năng “nhớ” môi trường xung quanh nó**.

---

## ⚙️ **2. Cú pháp cơ bản của Closure**

Cấu trúc tổng quát:

```swift
{ (parameters) -> ReturnType in
    // code thực thi
}
```

### Ví dụ đơn giản:

```swift
let greeting = { (name: String) -> String in
    return "Xin chào \(name)!"
}

print(greeting("Cường"))
```

✅ Kết quả: `Xin chào Cường!`

---

## ✂️ **3. Viết gọn Closure (Trailing Closure Syntax)**

Swift cho phép rút gọn cú pháp **rất mạnh**:

```swift
// Cách 1: Đầy đủ
let square = { (x: Int) -> Int in
    return x * x
}

// Cách 2: Rút gọn
let squareShort: (Int) -> Int = { x in x * x }

print(squareShort(5)) // 25
```

---

## 🔁 **4. Closure làm tham số cho hàm**

Closures thường được **truyền vào hàm khác** – ví dụ như `map`, `filter`, `sorted`, hoặc khi làm animation, completion handler.

### Ví dụ:

```swift
func operateOnNumbers(a: Int, b: Int, operation: (Int, Int) -> Int) -> Int {
    return operation(a, b)
}

let sum = operateOnNumbers(a: 3, b: 4, operation: { (x, y) in x + y })
print(sum) // 7
```

---

### 🔹 Rút gọn hơn nữa (Trailing Closure)

```swift
let product = operateOnNumbers(a: 3, b: 4) { $0 * $1 }
print(product) // 12
```

> `$0`, `$1`, `$2`, … là **tham số ngầm định** của closure.

---

## 🧩 **5. Closures và Higher-order functions**

Swift hỗ trợ các hàm xử lý mảng rất “functional”:

### `map` – Biến đổi từng phần tử

```swift
let numbers = [1, 2, 3, 4]
let squared = numbers.map { $0 * $0 }
print(squared) // [1, 4, 9, 16]
```

### `filter` – Lọc phần tử

```swift
let evens = numbers.filter { $0 % 2 == 0 }
print(evens) // [2, 4]
```

### `reduce` – Gộp thành một giá trị

```swift
let total = numbers.reduce(0) { $0 + $1 }
print(total) // 10
```

---

## 🧮 **6. Closures “nhớ” giá trị (Capture Values)**

Closures **có thể lưu lại giá trị từ phạm vi bên ngoài** —
đây là “bí quyết” giúp SwiftUI và UIKit giữ trạng thái.

### Ví dụ:

```swift
func makeCounter() -> () -> Int {
    var count = 0
    return {
        count += 1
        return count
    }
}

let counter = makeCounter()
print(counter()) // 1
print(counter()) // 2
print(counter()) // 3
```

💡 Biến `count` vẫn tồn tại **bên trong closure** mặc dù `makeCounter()` đã chạy xong —
closure “ghi nhớ” giá trị đó ⇒ gọi là **capture**.

---

## 📦 **7. Phạm vi biến (Variable Scope)**

Phạm vi quyết định **nơi biến được truy cập** và **thời điểm biến bị hủy**.

| Loại phạm vi       | Mô tả                           | Ví dụ                 |
| ------------------ | ------------------------------- | --------------------- |
| **Global Scope**   | Biến khai báo ngoài mọi hàm     | `let PI = 3.14`       |
| **Function Scope** | Biến khai báo trong hàm         | chỉ dùng trong hàm đó |
| **Block Scope**    | Biến trong `{}` của `if`, `for` | hết block là mất      |

### Ví dụ:

```swift
var globalVar = 10

func demoScope() {
    let localVar = 5
    print(globalVar + localVar)
}
// print(localVar) ❌ lỗi: không nhìn thấy biến trong hàm
```

---

## 💡 **8. Từ khóa `inout` – truyền tham chiếu**

Mặc định Swift truyền **theo giá trị (by value)**.
Nếu muốn hàm thay đổi trực tiếp biến gốc → dùng `inout`.

```swift
func addOne(to number: inout Int) {
    number += 1
}

var myNum = 10
addOne(to: &myNum)
print(myNum) // 11
```

---

## ⚙️ **9. Escaping vs Non-escaping Closures (nâng cao)**

> Khi closure được **gọi sau khi hàm kết thúc**, ta gọi là **escaping closure** (thường dùng trong async code, callback, API call).

```swift
func fetchData(completion: @escaping () -> Void) {
    DispatchQueue.global().async {
        print("Đang tải dữ liệu...")
        completion()
    }
}
```

* `@escaping` nghĩa là closure **thoát khỏi** phạm vi của hàm ban đầu.
* `@escaping` sẽ học kỹ hơn trong module **Concurrency (async/await)**.

---

## 🧪 **10. Thực hành trong Playground**

### 🔹 Ví dụ 1

```swift
let numbers = [1, 3, 5, 7, 9]
let doubled = numbers.map { $0 * 2 }
print(doubled)
```

### 🔹 Ví dụ 2

```swift
func performTwice(action: () -> Void) {
    action()
    action()
}

performTwice {
    print("Hello Swift")
}
```

### 🔹 Ví dụ 3

```swift
let counter = makeCounter()
for _ in 1...5 {
    print("Đếm: \(counter())")
}
```

---

## 🧩 **11. Bài tập trên lớp**

1. Viết hàm `applyTwice(_:to:)` nhận một closure và một giá trị, rồi áp dụng closure đó hai lần.
2. Tạo closure `isEven` kiểm tra số chẵn, sau đó dùng `filter` để lọc mảng chẵn.
3. Viết hàm `repeatTask(times:task:)` nhận số lần và closure để thực hiện nhiều lần.

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #5)** 🎒

| Mức độ        | Đề bài                                                                         | Gợi ý                                   |
| ------------- | ------------------------------------------------------------------------------ | --------------------------------------- |
| 🟢 Cơ bản     | Tạo closure nhận tên người và in “Chào \{tên\}!”                                 | Dạng `{ (name: String) in ... }`        |
| 🟡 Trung bình | Viết hàm nhận danh sách điểm và dùng `filter` để lấy điểm ≥ 8                  | Dùng `filter`                           |
| 🔵 Nâng cao   | Viết hàm `makeMultiplier(by:)` trả về một closure nhân giá trị đầu vào với `n` | Dùng capture value                      |
| 🟣 Thử thách  | Viết `makeFibonacci()` trả về closure sinh số Fibonacci kế tiếp mỗi lần gọi    | Dùng 2 biến `a`, `b` và `capture` chúng |

**Gợi ý nâng cao:**

```swift
func makeMultiplier(by n: Int) -> (Int) -> Int {
    return { x in x * n }
}
let triple = makeMultiplier(by: 3)
print(triple(4)) // 12
```

**Thử thách:**

```swift
func makeFibonacci() -> () -> Int {
    var a = 0, b = 1
    return {
        let next = a
        (a, b) = (b, a + b)
        return next
    }
}

let fib = makeFibonacci()
for _ in 1...7 { print(fib()) } // 0 1 1 2 3 5 8
```

---

## 📚 **Tổng kết kiến thức**

| Chủ đề           | Ý nghĩa                     | Ví dụ                         |
| ---------------- | --------------------------- | ----------------------------- |
| Closure          | Hàm vô danh, có thể lưu trữ | `{ x in x * 2 }`              |
| Trailing closure | Closure cuối cùng trong hàm | `func run { ... }`            |
| Capture          | Lưu giá trị từ bên ngoài    | `makeCounter()`               |
| Scope            | Phạm vi biến hoạt động      | Trong `{}`                    |
| inout            | Truyền tham chiếu           | `func change(_ n: inout Int)` |

---

## 🧭 **Kết thúc bài học**

✅ Hôm nay, các bạn đã:

* Hiểu closure – một trong những khái niệm **nâng cấp trí tuệ** của Swift.
* Biết cách closure “nhớ” dữ liệu, và điều này cực kỳ quan trọng trong UIKit, SwiftUI, và async code.
* Thành thạo tư duy **hàm bậc cao** – foundation cho Swift hiện đại.

---

🎓 **Bài 11 sắp tới:**
Chúng ta sẽ học **Cấu trúc dữ liệu (Arrays – Dictionaries – Sets)**,
bắt đầu thao tác trên **bộ sưu tập dữ liệu thực tế**, để viết được mini app quản lý danh sách học sinh, tính điểm trung bình, v.v.

---

Thầy hỏi cả lớp một câu vui nhé:
Ai muốn thầy làm **“project mini – Bộ đếm thông minh bằng closure (Counter App)”** để các em vừa học vừa thực hành UIKit sau khi xong bài 11 không?
