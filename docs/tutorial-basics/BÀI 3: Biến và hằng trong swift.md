---
sidebar_position: 3
---

Rất tốt 👏 — chào mừng các bạn đến với **Bài 3: Biến (`var`) và Hằng (`let`) trong Swift**.
Đây là **bài học nền tảng cực kỳ quan trọng**, vì trong mọi chương trình Swift, việc **lưu trữ, quản lý và bảo vệ dữ liệu** đều bắt đầu từ khái niệm **biến và hằng**.
Hôm nay, chúng ta sẽ cùng nhau tìm hiểu cách Swift xử lý giá trị — tại sao ngôn ngữ này lại “an toàn” hơn C, Java hay Python ở điểm này.

---

# 🧩 BÀI 3: BIẾN (`var`) VÀ HẰNG (`let`) TRONG SWIFT

---

## 🎯 **Mục tiêu bài học**

Sau khi học xong bài này, bạn sẽ:

1. Hiểu khái niệm **biến (variable)** và **hằng (constant)**.
2. Biết cách khai báo, gán giá trị và in ra dữ liệu.
3. Phân biệt được kiểu dữ liệu **ngầm định** và **tường minh**.
4. Hiểu về **phạm vi biến (scope)** và **quy tắc đặt tên chuẩn**.
5. Thực hành viết chương trình có biến – hằng linh hoạt và an toàn.

---

## 🧠 **1. Khái niệm cơ bản**

Trong Swift, mọi giá trị trong chương trình đều được lưu vào **biến hoặc hằng**.

* **Biến (`var`)** là giá trị có thể **thay đổi** sau khi gán.
* **Hằng (`let`)** là giá trị **không thể thay đổi** (immutable).

> 💬 *Tư duy Swift: hãy dùng `let` mặc định, chỉ dùng `var` khi thật cần thiết.*

---

## 🔹 **Ví dụ đơn giản**

```swift
var name = "Cường"
let birthYear = 1990

print("Xin chào \(name)! Năm sinh của bạn là \(birthYear).")

name = "Thành Công" // ✅ được phép
// birthYear = 2000  ❌ lỗi - không thể thay đổi hằng số
```

Khi bạn thử thay đổi `birthYear`, Swift sẽ báo lỗi:

> Cannot assign to value: 'birthYear' is a 'let' constant

---

## ⚙️ **2. Kiểu dữ liệu và kiểu suy luận (Type Inference)**

Swift có **Type Inference** — tức là tự động hiểu kiểu của biến dựa trên giá trị gán.

```swift
var age = 35      // Swift hiểu là Int
var pi = 3.14     // Swift hiểu là Double
var city = "Hà Nội" // Swift hiểu là String
var isOnline = true  // Swift hiểu là Bool
```

Nhưng bạn vẫn có thể **chỉ định tường minh kiểu dữ liệu** nếu cần:

```swift
var score: Int = 95
let studentName: String = "Mai Lê"
```

---

## 🧮 **3. Gán lại và cập nhật giá trị**

Biến (`var`) có thể thay đổi giá trị bất cứ lúc nào:

```swift
var temperature = 27
print("Nhiệt độ hiện tại: \(temperature)°C")

temperature = 30
print("Nhiệt độ mới: \(temperature)°C")
```

Tuy nhiên, **hằng (`let`) không thể thay đổi**:

```swift
let maxScore = 100
// maxScore = 90 ❌ lỗi biên dịch
```

---

## 📐 **4. Tên biến và quy tắc đặt tên**

Swift cho phép tên biến dùng ký tự Unicode (kể cả tiếng Việt, emoji!), nhưng khuyến nghị theo **chuẩn camelCase**.

### ✅ Hợp lệ:

```swift
var firstName = "Cường"
var soHocSinh = 30
var 🔥Level = 10
```

### ❌ Không hợp lệ:

```swift
var 1name = "Sai"   // không bắt đầu bằng số
var let = "Sai"     // trùng từ khóa
```

### 💡 Quy tắc tốt:

* Dùng **danh từ** để mô tả dữ liệu: `age`, `price`, `username`.
* Dùng tiếng Anh chuẩn, nhất quán trong dự án.
* Hằng thường viết HOA nếu là giá trị toàn cục:

  ```swift
  let MAX_USERS = 1000
  ```

---

## 🧩 **5. Phạm vi biến (Variable Scope)**

Phạm vi quyết định **biến sống được bao lâu** và **có thể được truy cập ở đâu**.

Ví dụ:

```swift
func demoScope() {
    var x = 10
    if true {
        var y = 5
        print(x + y) // ✅ x và y đều hợp lệ
    }
    // print(y) ❌ lỗi: y không tồn tại ngoài block if
}
```

Swift sử dụng **phạm vi khối (block scope)** — biến chỉ có hiệu lực bên trong `{ }` nơi nó được khai báo.

---

## 🧠 **6. Gán nhiều biến cùng lúc**

Bạn có thể gán nhiều biến trong một dòng:

```swift
var x = 1, y = 2, z = 3
let name1 = "A", name2 = "B"
```

Hoặc dùng **tuple**:

```swift
let (a, b, c) = (10, 20, 30)
print(a + b + c) // 60
```

---

## 🧰 **7. Biến chưa có giá trị (Optional giai đoạn sau)**

Nếu bạn khai báo biến mà **chưa gán giá trị ngay**, Swift yêu cầu bạn **khai báo kiểu tường minh**:

```swift
var address: String
address = "Hồ Chí Minh"
print(address)
```

Nếu bạn không gán, và không khai báo kiểu, Swift sẽ báo lỗi:

> Variable 'address' used before being initialized

(Phần này sẽ mở rộng trong bài **Optional & Safety** sau.)

---

## 💬 **8. Tư duy “Immutable by Default” của Swift**

Swift khuyến khích bạn **giữ dữ liệu càng bất biến càng tốt**, vì:

* Giúp code an toàn hơn, ít lỗi hơn.
* Dễ debug và dễ đọc.
* Hỗ trợ tốt cho lập trình đa luồng (concurrency).

Ví dụ:

```swift
let baseURL = "https://api.example.com"
// Không ai có thể vô tình đổi đường dẫn gốc của hệ thống!
```

---

## 🧪 **9. Bài tập thực hành**

| Mức độ        | Yêu cầu                                                                                     | Gợi ý                                    |
| ------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------- |
| 🟢 Cơ bản     | Tạo 3 biến: `name`, `age`, `city` và in ra câu giới thiệu                                   | Dùng `print()` và `String interpolation` |
| 🟡 Trung bình | Dùng `let` để khai báo hằng số `PI`, tính chu vi và diện tích hình tròn có bán kính `r = 5` | `let pi = 3.14159`                       |
| 🔵 Nâng cao   | Viết chương trình hỏi tên người dùng, năm sinh → tính tuổi hiện tại (năm 2025) và in ra     | Dùng `readLine()` và phép trừ            |

Ví dụ gợi ý:

```swift
print("Nhập tên:")
let name = readLine() ?? "Ẩn danh"
print("Nhập năm sinh:")
let year = Int(readLine() ?? "0") ?? 0
let age = 2025 - year
print("Xin chào \(name)! Năm nay bạn \(age) tuổi.")
```

---

## 📚 **10. Tổng kết kiến thức**

| Khái niệm      | Mô tả                    | Ví dụ                |
| -------------- | ------------------------ | -------------------- |
| `var`          | Biến có thể thay đổi     | `var score = 90`     |
| `let`          | Hằng không thể thay đổi  | `let pi = 3.14`      |
| Type Inference | Swift tự hiểu kiểu       | `var name = "Mai"`   |
| Explicit Type  | Gán kiểu tường minh      | `var count: Int = 5` |
| Scope          | Phạm vi tồn tại của biến | Bên trong `{}`       |

---

## 🧭 **Kết thúc bài học**

✅ Bạn đã hiểu:

* Phân biệt được **biến** và **hằng**.
* Nắm được **quy tắc đặt tên, phạm vi, và kiểu dữ liệu**.
* Viết được các chương trình cơ bản có biến động.

---

👉 **Tiết sau (Bài 4)**, chúng ta sẽ bước vào **Kiểu dữ liệu cơ bản trong Swift** —
bao gồm: `Int`, `Double`, `String`, `Bool`, và cách Swift xử lý **ép kiểu, toán tử và định dạng dữ liệu**.

Bạn đã sẵn sàng chưa?

