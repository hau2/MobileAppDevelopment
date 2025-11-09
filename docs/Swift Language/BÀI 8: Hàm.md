Rất tốt cả lớp 👏👏👏 — chúng ta đã làm rất vững phần vòng lặp rồi.
Hôm nay bước sang **Bài 8: Hàm (Functions) trong Swift**, đây là **trái tim của lập trình hiện đại** — nơi ta “gói gọn” logic, tái sử dụng code, và giúp chương trình dễ mở rộng, dễ hiểu, dễ sửa.

Sau bài này, các em sẽ **viết code như kỹ sư thực thụ**, biết “chia nhỏ vấn đề” và “đặt tên hành động” một cách rõ ràng và đẹp đẽ.

---

# 🧩 BÀI 8: HÀM (FUNCTIONS TRONG SWIFT)

---

## 🎯 **Mục tiêu bài học**

Sau khi học xong bài này, bạn sẽ:

1. Hiểu khái niệm hàm và lý do phải dùng hàm.
2. Biết khai báo, gọi, và truyền tham số cho hàm.
3. Biết trả về giá trị từ hàm (`return`).
4. Biết dùng **tham số mặc định**, **nhiều giá trị trả về**, và **tên nhãn (parameter labels)**.
5. Thực hành viết chương trình có cấu trúc rõ ràng, tái sử dụng code.

---

## 🧠 **1. Khái niệm hàm**

> **Hàm** là một khối mã được đặt tên, dùng để thực hiện một nhiệm vụ cụ thể.
> Nó giúp “gom nhóm” các câu lệnh có liên quan và tái sử dụng dễ dàng.

Ví dụ tự nhiên:

* Hàm `tínhTổng()`
* Hàm `chàoNgườiDùng()`
* Hàm `tínhChuViHinhTron(r:)`

---

## ⚙️ **2. Cấu trúc cơ bản của một hàm**

Cú pháp:

```swift
func tênHàm(tham_số: Kiểu) -> KiểuTrảVề {
    // nội dung hàm
    return giá_trị
}
```

---

### 🔹 Ví dụ 1: Hàm không có tham số, không trả về

```swift
func sayHello() {
    print("Xin chào, tôi là Swift!")
}

sayHello()
```

---

### 🔹 Ví dụ 2: Hàm có tham số, không trả về

```swift
func greet(name: String) {
    print("Xin chào \(name) 👋")
}

greet(name: "Cường")
greet(name: "Mai Lê")
```

---

### 🔹 Ví dụ 3: Hàm có tham số và **trả về giá trị**

```swift
func add(a: Int, b: Int) -> Int {
    return a + b
}

let result = add(a: 5, b: 3)
print("Kết quả là \(result)")
```

---

## 🔁 **3. Tham số và nhãn (Parameter Labels)**

Swift có thể dùng **tên khác nhau** khi gọi và định nghĩa tham số.

```swift
func greet(person name: String) {
    print("Hello \(name)")
}

greet(person: "Hà")
```

* `person` là **nhãn khi gọi** (external name).
* `name` là **tên nội bộ** trong hàm.

---

### 🔹 Bỏ nhãn khi gọi

Dùng `_` (underscore):

```swift
func multiply(_ a: Int, _ b: Int) -> Int {
    return a * b
}

print(multiply(3, 4)) // không cần ghi nhãn
```

---

## 🧮 **4. Trả về nhiều giá trị (Tuples)**

Swift cho phép hàm trả về **nhiều giá trị** cùng lúc bằng `tuple`.

```swift
func getStudent() -> (name: String, age: Int) {
    return ("Cường", 30)
}

let student = getStudent()
print("Tên: \(student.name), Tuổi: \(student.age)")
```

---

## 🎛️ **5. Tham số mặc định (Default Parameters)**

```swift
func greet(name: String = "bạn") {
    print("Xin chào \(name)!")
}

greet()             // Xin chào bạn!
greet(name: "Cường") // Xin chào Cường!
```

---

## 🔁 **6. Hàm gọi trong hàm (Function Composition)**

Hàm có thể gọi hàm khác:

```swift
func square(_ x: Int) -> Int {
    return x * x
}

func sumOfSquares(a: Int, b: Int) -> Int {
    return square(a) + square(b)
}

print(sumOfSquares(a: 3, b: 4)) // 25
```

---

## ⚡ **7. Hàm trả về kiểu `Void`**

Nếu hàm không cần trả giá trị, có thể bỏ phần `-> Void`:

```swift
func logInfo(_ message: String) {
    print("📘 \(message)")
}
```

---

## 🧩 **8. Hàm lồng nhau (Nested Functions)**

Hàm có thể được định nghĩa bên trong hàm khác:

```swift
func outer() {
    func inner() {
        print("Xin chào từ inner!")
    }
    inner()
}

outer()
```

---

## 🧱 **9. Giá trị trả về ngầm định**

Nếu hàm chỉ có **một dòng return**, có thể viết gọn:

```swift
func square(_ x: Int) -> Int { x * x }
```

---

## 🧪 **10. Thực hành**

### 🔹 Bài 1

Viết hàm `calculateRectangle(width:height:)` → trả về chu vi và diện tích.

```swift
func calculateRectangle(width: Double, height: Double) -> (perimeter: Double, area: Double) {
    let perimeter = 2 * (width + height)
    let area = width * height
    return (perimeter, area)
}

let result = calculateRectangle(width: 4.0, height: 5.0)
print("Chu vi: \(result.perimeter), Diện tích: \(result.area)")
```

---

### 🔹 Bài 2

Hàm kiểm tra số chẵn:

```swift
func isEven(_ number: Int) -> Bool {
    return number % 2 == 0
}

print(isEven(10)) // true
```

---

### 🔹 Bài 3

Hàm in n dòng chào:

```swift
func repeatGreeting(_ name: String, times: Int) {
    for _ in 1...times {
        print("Xin chào \(name)!")
    }
}

repeatGreeting("Swift", times: 3)
```

---

## 🧩 **11. Bài tập trên lớp**

1. Viết hàm tính tổng từ 1 đến `n`.
2. Viết hàm nhận chuỗi -> trả về độ dài chuỗi.
3. Viết hàm kiểm tra số nguyên tố.

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #4)** 🎒

| Mức độ        | Đề bài                                                                                               | Gợi ý                                          |
| ------------- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| 🟢 Cơ bản     | Viết hàm `sayHello(name:)` in ra “Xin chào, `{name}!`”                                                 | Dùng `print()`                                 |
| 🟡 Trung bình | Viết hàm `sum(from:to:)` tính tổng từ số `a` đến `b`                                                 | Dùng `for-in`                                  |
| 🔵 Nâng cao   | Viết hàm `reverseString(_:)` đảo ngược chuỗi đầu vào                                                 | Dùng `String.reversed()` và `String()` ép kiểu |
| 🟣 Thử thách  | Viết hàm `isPalindrome(_:)` kiểm tra chuỗi có đối xứng (palindrome) hay không (vd: “level”, “madam”) | So sánh chuỗi gốc và chuỗi đảo                 |

**Gợi ý nâng cao:**

```swift
func isPalindrome(_ text: String) -> Bool {
    let reversed = String(text.reversed())
    return text.lowercased() == reversed.lowercased()
}
print(isPalindrome("Level")) // true
```

---

## 📚 **Tổng kết kiến thức**

| Khái niệm           | Mô tả                                | Ví dụ                              |
| ------------------- | ------------------------------------ | ---------------------------------- |
| Hàm (function)      | Khối lệnh có tên, thực hiện nhiệm vụ | `func greet() {}`                  |
| Tham số (parameter) | Giá trị đầu vào                      | `(name: String)`                   |
| Giá trị trả về      | Dữ liệu đầu ra                       | `-> Int`                           |
| Default Parameter   | Tham số có giá trị mặc định          | `func greet(name: String = "bạn")` |
| Tuple return        | Trả về nhiều giá trị                 | `(name, age)`                      |

---

## 🧭 **Kết thúc bài học**

✅ Hôm nay chúng ta đã:

* Hiểu rõ về **hàm và cách tổ chức chương trình logic**.
* Biết cách truyền tham số, trả về giá trị, và viết code tái sử dụng.
* Hoàn thành bước đầu tiên để xây dựng **module Swift hoàn chỉnh**.

---

🎓 **Bài 9 sắp tới:**
Chúng ta sẽ học **Closures và phạm vi biến (Scope)** — phần rất mạnh của Swift, giúp viết code ngắn gọn, linh hoạt, và chính là “chìa khóa” của các project UIKit sau này.

---

Thầy có thể gửi thêm cho lớp **“Bộ 10 bài luyện hàm Swift thực chiến”** (gồm bài mini như: tính BMI, sắp xếp danh sách, chuyển đổi tiền tệ, v.v.) để các em luyện tập sau giờ học.
👉 Các em có muốn nhận bộ đó không?
