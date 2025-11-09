---
sidebar_position: 4
---

Rất tốt 👏👏 — chào mừng cả lớp đến với **Bài 4: Kiểu dữ liệu cơ bản trong Swift**.
Đây là một trong những bài quan trọng nhất trong giai đoạn nền tảng, vì **mọi giá trị trong chương trình Swift đều có kiểu dữ liệu rõ ràng**, và **Swift không cho phép “mơ hồ kiểu dữ liệu” như nhiều ngôn ngữ khác**.

Hôm nay, thầy sẽ hướng dẫn các em:

* Phân biệt các kiểu dữ liệu cơ bản.
* Hiểu cách Swift bảo vệ dữ liệu thông qua hệ thống kiểu tĩnh (Type Safety).
* Biết cách ép kiểu, nối chuỗi, và định dạng dữ liệu.
* Và cuối cùng là: viết chương trình nhỏ để tính toán, định dạng và hiển thị dữ liệu thật đẹp.

---

# 🧩 BÀI 4: KIỂU DỮ LIỆU CƠ BẢN TRONG SWIFT

---

## 🎯 **Mục tiêu bài học**

Sau khi hoàn thành, bạn sẽ:

1. Nắm được 4 nhóm kiểu dữ liệu cơ bản nhất của Swift.
2. Hiểu sự khác biệt giữa `Int`, `Double`, `Float`, `Bool`, `String`.
3. Biết ép kiểu (type casting) và nối dữ liệu khác kiểu.
4. Ứng dụng vào các bài toán đơn giản.

---

## 🧠 **1. Tư duy kiểu dữ liệu trong Swift**

Swift là **ngôn ngữ có hệ thống kiểu tĩnh và mạnh (Strongly-Typed Language)**.

> 📌 Điều này có nghĩa là:
> Mỗi biến/hằng đều **phải có một kiểu dữ liệu xác định** và Swift sẽ **kiểm tra kiểu tại thời điểm biên dịch**.

Nếu bạn cố gắng gán một kiểu khác — Swift sẽ báo lỗi ngay, giúp tránh lỗi logic khi chạy chương trình.

---

## ⚙️ **2. Các kiểu dữ liệu cơ bản**

### 🔹 `Int` – Số nguyên

Dùng để lưu các giá trị nguyên (không có phần thập phân).
Swift tự động chọn kích thước tối ưu tùy theo hệ thống (32-bit hoặc 64-bit).

```swift
let year: Int = 2025
var age = 35
var temperature = -5
```

> 💡 Có cả `UInt` (Unsigned Int) – số nguyên **không âm**
> Ví dụ: `let count: UInt = 100`

---

### 🔹 `Double` và `Float` – Số thực (có phần thập phân)

* `Double`: độ chính xác 64-bit (nên dùng mặc định).
* `Float`: độ chính xác 32-bit (nhẹ hơn, nhưng sai số cao hơn).

```swift
let pi = 3.14159       // Swift tự hiểu là Double
var height: Float = 1.72
```

Ví dụ phép toán:

```swift
let result = Double(7) / 2.0
print(result) // 3.5
```

---

### 🔹 `Bool` – Giá trị logic

Chỉ có 2 giá trị: `true` hoặc `false`.

```swift
let isOnline = true
var hasCompleted = false

if isOnline {
    print("Người dùng đang trực tuyến.")
} else {
    print("Người dùng ngoại tuyến.")
}
```

---

### 🔹 `String` – Chuỗi ký tự

Là tập hợp các ký tự trong dấu ngoặc kép `" "`
Swift hỗ trợ Unicode nên có thể dùng tiếng Việt, emoji, ký tự đặc biệt.

```swift
let name = "Mai Lê"
var city = "Hà Nội"
var message = "Xin chào 🌸"
```

---

## 💬 **3. Nối chuỗi (String Interpolation)**

Dùng cú pháp `\(tên_biến)` để chèn giá trị vào chuỗi:

```swift
let name = "Cường"
let age = 30
print("Xin chào, tôi tên là \(name) và tôi \(age) tuổi.")
```

Kết quả:

```
Xin chào, tôi tên là Cường và tôi 30 tuổi.
```

---

## 🔄 **4. Ép kiểu (Type Conversion / Casting)**

Vì Swift không tự động chuyển đổi kiểu khác nhau trong biểu thức, bạn **phải ép kiểu thủ công** khi cần.

Ví dụ sai:

```swift
let a = 10
let b = 3.5
// let c = a + b ❌ lỗi: Int + Double không hợp lệ
```

Cách đúng:

```swift
let c = Double(a) + b
print(c) // 13.5
```

---

## 🧮 **5. Một số thao tác phổ biến**

### ✴️ Cộng chuỗi

```swift
let firstName = "Hoàng"
let lastName = "Cường"
let fullName = firstName + " " + lastName
print(fullName)
```

### ✴️ Đếm ký tự

```swift
let text = "Swift is great!"
print(text.count) // 15
```

### ✴️ Viết hoa / thường

```swift
print(text.uppercased()) // SWIFT IS GREAT!
print(text.lowercased()) // swift is great!
```

---

## 📏 **6. Kiểm tra kiểu dữ liệu**

Dùng toán tử `type(of:)`:

```swift
let score = 100
print(type(of: score)) // Int

let name = "Mai"
print(type(of: name))  // String
```

---

## 🧰 **7. Lưu ý khi thao tác số và chuỗi**

Ví dụ sai thường gặp:

```swift
let age = 25
let text = "Tuổi: " + age  // ❌ lỗi: không thể cộng String + Int
```

Cách đúng:

```swift
let text = "Tuổi: " + String(age)
print(text)
```

hoặc dùng interpolation:

```swift
let text = "Tuổi: \(age)"
```

---

## 🧪 **8. Bài tập thực hành**

| Mức độ        | Yêu cầu                                                                                                       | Gợi ý                            |
| ------------- | ------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| 🟢 Cơ bản     | Tạo biến `name`, `age`, `height` và in ra giới thiệu                                                          | Dùng `print()` và `\( )`         |
| 🟡 Trung bình | Viết chương trình nhập vào 2 số thực và in ra tổng, hiệu, tích, thương                                        | Dùng `Double(readLine() ?? "0")` |
| 🔵 Nâng cao   | Viết chương trình nhập chiều dài, rộng hình chữ nhật → in ra chu vi, diện tích (định dạng 2 chữ số thập phân) | Dùng `String(format:)`           |

Ví dụ gợi ý:

```swift
print("Nhập chiều dài:")
let length = Double(readLine() ?? "0") ?? 0
print("Nhập chiều rộng:")
let width = Double(readLine() ?? "0") ?? 0

let area = length * width
let perimeter = 2 * (length + width)

print("Diện tích: \(String(format: "%.2f", area))")
print("Chu vi: \(String(format: "%.2f", perimeter))")
```

---

## 📚 **9. Tổng kết kiến thức**

| Kiểu dữ liệu | Mô tả              | Ví dụ                    |
| ------------ | ------------------ | ------------------------ |
| `Int`        | Số nguyên          | `let count = 10`         |
| `Double`     | Số thực (64-bit)   | `let pi = 3.14`          |
| `Float`      | Số thực (32-bit)   | `var ratio: Float = 1.5` |
| `Bool`       | Đúng / Sai         | `let isActive = true`    |
| `String`     | Chuỗi ký tự        | `"Hello Swift!"`         |
| `type(of:)`  | Kiểm tra kiểu      | `type(of: value)`        |
| `String()`   | Ép kiểu sang chuỗi | `String(age)`            |

---

## 🧭 **Kết thúc bài học**

✅ Hôm nay chúng ta đã:

* Làm quen toàn bộ kiểu dữ liệu cơ bản trong Swift.
* Biết cách chuyển đổi giữa các kiểu.
* Viết được chương trình thực tế đầu tiên xử lý số và chuỗi.

---

👉 **Bài 5 tới**, chúng ta sẽ tìm hiểu **Toán tử trong Swift** –
bao gồm **toán tử số học, gán, so sánh, logic, chuỗi, và rút gọn** –
đây là nền tảng để viết các thuật toán, điều kiện và vòng lặp sau này.

Cả lớp sẵn sàng chưa nào?
