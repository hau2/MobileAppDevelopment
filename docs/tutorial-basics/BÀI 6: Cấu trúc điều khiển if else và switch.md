---
sidebar_position: 6
---

Tuyệt vời 👏👏👏
Cả lớp đang tiến rất nhanh — và hôm nay, ta bước vào **Bài 6: Cấu trúc điều khiển (rẽ nhánh) trong Swift**.

Đây là **bài học quan trọng nhất** trong phần “Swift Core” —
vì nhờ *rẽ nhánh*, chương trình mới **biết tư duy, biết lựa chọn**, và **phản ứng khác nhau tùy tình huống**.
Từ đây, chúng ta bắt đầu làm cho code trở nên “sống”.

---

# 🧩 BÀI 6: CẤU TRÚC ĐIỀU KHIỂN (IF – ELSE – SWITCH)

---

## 🎯 **Mục tiêu bài học**

Sau bài học này, các bạn sẽ:

1. Biết cách dùng `if`, `else if`, `else` để ra quyết định.
2. Hiểu cấu trúc `switch` và cách viết linh hoạt trong Swift.
3. Biết dùng toán tử điều kiện (`? :`) để rút gọn câu lệnh.
4. Thực hành viết chương trình quyết định dựa trên dữ liệu đầu vào.

---

## 🧠 **1. Tư duy rẽ nhánh**

Máy tính **chỉ làm đúng theo điều kiện bạn đặt ra**.

> 👉 “Nếu điều kiện đúng → làm A,
> còn sai → làm B.”

Đây chính là *conditional statements*.

Ví dụ logic tự nhiên:

> Nếu điểm ≥ 8 → In “Giỏi”,
> ngược lại → In “Chưa đạt”.

---

## ⚙️ **2. Cấu trúc `if` cơ bản**

Cú pháp:

```swift
if condition {
    // code chạy khi điều kiện ĐÚNG
}
```

Ví dụ:

```swift
let age = 20

if age >= 18 {
    print("Bạn đã đủ tuổi trưởng thành.")
}
```

---

## 🔁 **3. `if – else`**

```swift
if condition {
    // Khi điều kiện đúng
} else {
    // Khi điều kiện sai
}
```

Ví dụ:

```swift
let temperature = 30

if temperature > 25 {
    print("Thời tiết nóng.")
} else {
    print("Thời tiết mát mẻ.")
}
```

---

## 🔂 **4. `if – else if – else` (nhiều nhánh)**

Khi có **nhiều tình huống cần kiểm tra**.

```swift
let score = 7.5

if score >= 8.0 {
    print("Giỏi")
} else if score >= 6.5 {
    print("Khá")
} else if score >= 5.0 {
    print("Trung bình")
} else {
    print("Yếu")
}
```

👉 Swift sẽ **kiểm tra lần lượt từ trên xuống**, gặp điều kiện đúng thì dừng.

---

## 💡 **5. Kết hợp nhiều điều kiện**

Dùng `&&` (và), `||` (hoặc), `!` (phủ định).

```swift
let isMember = true
let age = 20

if isMember && age >= 18 {
    print("Được phép tham gia câu lạc bộ.")
}
```

---

## 🧮 **6. Lồng nhau (Nested if)**

```swift
let year = 2025
let isLeap = (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0)

if isLeap {
    print("\(year) là năm nhuận.")
} else {
    print("\(year) không phải năm nhuận.")
}
```

---

## 🔁 **7. Cấu trúc `switch` trong Swift**

Swift có `switch` **mạnh mẽ hơn** nhiều ngôn ngữ khác.
Không chỉ giới hạn ở số nguyên, mà có thể dùng chuỗi, ký tự, tuple, enum...

Cú pháp:

```swift
switch value {
case pattern1:
    // hành động 1
case pattern2:
    // hành động 2
default:
    // mặc định
}
```

Ví dụ:

```swift
let grade = "B"

switch grade {
case "A":
    print("Xuất sắc")
case "B":
    print("Giỏi")
case "C":
    print("Khá")
case "D":
    print("Trung bình")
default:
    print("Chưa đạt")
}
```

> 💬 Trong Swift, **không cần từ khóa `break`** – mỗi `case` tự động kết thúc.

---

## ⚡ **8. Range và nhiều giá trị trong `switch`**

Bạn có thể gom nhiều giá trị cùng xử lý:

```swift
let point = 82

switch point {
case 90...100:
    print("Xuất sắc")
case 80..<90:
    print("Giỏi")
case 65..<80:
    print("Khá")
case 50..<65:
    print("Trung bình")
default:
    print("Yếu")
}
```

---

## 🧩 **9. Switch với chuỗi hoặc tuple**

```swift
let person = ("Cường", 30)

switch person {
case ("Cường", _):
    print("Xin chào Cường!")
case (_, let age) where age > 40:
    print("Xin chào bác!")
default:
    print("Xin chào bạn!")
}
```

> `_` nghĩa là “bỏ qua giá trị”,
> `where` dùng để thêm điều kiện cho case.

---

## 🧪 **10. Toán tử ba ngôi (Ternary)**

Rút gọn `if–else` thành 1 dòng:

```swift
let score = 7.0
let result = (score >= 5.0) ? "Đạt" : "Không đạt"
print(result)
```

---

## 📘 **11. Bài tập thực hành trên lớp**

### 🧮 Ví dụ 1:

Nhập nhiệt độ và in ra trạng thái:

* `< 18`: Lạnh
* `18–28`: Dễ chịu
* `> 28`: Nóng

### 🧮 Ví dụ 2:

Nhập tên tháng (1–12), in ra số ngày của tháng đó.
*(Gợi ý: dùng switch, có thể gom các tháng 31 ngày lại cùng 1 case.)*

```swift
let month = 2
switch month {
case 1,3,5,7,8,10,12:
    print("Tháng có 31 ngày")
case 4,6,9,11:
    print("Tháng có 30 ngày")
case 2:
    print("Tháng có 28 hoặc 29 ngày")
default:
    print("Không hợp lệ")
}
```

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #2)** 🎒

| Mức độ        | Đề bài                                                                                               | Gợi ý                                 |
| ------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------- |
| 🟢 Cơ bản     | Viết chương trình nhập 1 số và cho biết số đó là **chẵn** hay **lẻ**                                 | Dùng `%` và `if`                      |
| 🟡 Trung bình | Nhập điểm thi → in ra **xếp loại học lực**                                                           | Dùng `if–else if` hoặc `switch range` |
| 🔵 Nâng cao   | Viết chương trình **máy tính mini**: nhập 2 số và ký tự toán tử (`+`, `-`, `*`, `/`), rồi in kết quả | Dùng `switch` để chọn phép tính       |

**Gợi ý cấu trúc nâng cao:**

```swift
print("Nhập số 1:"); let a = Double(readLine() ?? "0") ?? 0
print("Nhập số 2:"); let b = Double(readLine() ?? "0") ?? 0
print("Nhập phép toán (+, -, *, /):"); let op = readLine() ?? "+"

switch op {
case "+":
    print("Kết quả: \(a + b)")
case "-":
    print("Kết quả: \(a - b)")
case "*":
    print("Kết quả: \(a * b)")
case "/":
    print("Kết quả: \(b != 0 ? a / b : 0)")
default:
    print("Phép toán không hợp lệ")
}
```

---

## 📚 **Tổng kết kiến thức**

| Cấu trúc              | Mục đích              | Ghi nhớ nhanh                |
| --------------------- | --------------------- | ---------------------------- |
| `if`                  | Kiểm tra điều kiện 1  | Khi chỉ có 1 trường hợp      |
| `if – else`           | Hai hướng lựa chọn    | Khi có “đúng” hoặc “sai”     |
| `if – else if – else` | Nhiều nhánh           | Khi có nhiều điều kiện       |
| `switch`              | Rẽ nhánh theo giá trị | Thay thế nhiều `if` phức tạp |
| `? :`                 | Toán tử rút gọn       | Dùng trong biểu thức nhỏ     |

---

## 🧭 **Kết thúc bài học**

✅ Hôm nay chúng ta đã:

* Hiểu rõ **cấu trúc điều kiện trong Swift**.
* Biết dùng `if`, `switch`, `ternary` để “dạy” chương trình suy nghĩ.
* Làm quen với *rẽ nhánh theo giá trị và phạm vi*.

---

🎓 **Tiết sau – Bài 7:**
Chúng ta sẽ học **Vòng lặp (Loops)**:
`for`, `while`, `repeat–while`, `break`, `continue`, và cả **vòng lặp lồng nhau** –
để chương trình có thể “làm đi làm lại” công việc thay cho con người.

Ai muốn thầy gửi thêm “bộ bài tập nâng cao luyện điều kiện + switch combo” trước bài 7 để rèn thêm thì giơ tay nhé 👋
