Tuyệt vời 👏👏👏 — hôm nay chúng ta sang **📘 Bài 3: Cấu trúc điều kiện trong Kotlin – if, when và biểu thức logic nâng cao**

Đây là bài cực kỳ quan trọng, vì điều kiện chính là “trái tim” giúp chương trình biết **ra quyết định**,
ví dụ: hiển thị thông báo, tính thuế, phân loại học sinh, kiểm tra dữ liệu đầu vào...

Thầy sẽ hướng dẫn **rất chậm – dễ hiểu – kèm ví dụ rõ ràng và bài tập thực tế.**

---

# 📘 BÀI 3: Cấu trúc điều kiện trong Kotlin – `if`, `when` và biểu thức logic

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ biết:

1. Sử dụng `if`, `else if`, `else` để ra quyết định.
2. Viết điều kiện gọn gàng hơn bằng **biểu thức Kotlin**.
3. Dùng `when` (thay cho `switch-case` của Java).
4. Viết điều kiện lồng nhau và logic phức tạp.

---

## 🧱 1. Cấu trúc `if` cơ bản

Cú pháp:

```kotlin
if (điều_kiện) {
    // Khối lệnh nếu điều kiện đúng
}
```

### 📋 Ví dụ

```kotlin
val age = 18
if (age >= 18) {
    println("Bạn đã đủ tuổi trưởng thành.")
}
```

➡️ Output:

```
Bạn đã đủ tuổi trưởng thành.
```

---

## 🧩 2. `if ... else`

```kotlin
val score = 60
if (score >= 50) {
    println("Bạn đã qua môn.")
} else {
    println("Bạn chưa đạt yêu cầu.")
}
```

➡️ Output: `Bạn đã qua môn.`

---

## 🔸 3. `if ... else if ... else` (nhiều điều kiện)

```kotlin
val grade = 85

if (grade >= 90) {
    println("Xuất sắc")
} else if (grade >= 75) {
    println("Giỏi")
} else if (grade >= 60) {
    println("Khá")
} else {
    println("Trung bình")
}
```

💡 Kotlin cho phép **viết gọn**:

```kotlin
val result = if (grade >= 90) "Xuất sắc"
             else if (grade >= 75) "Giỏi"
             else if (grade >= 60) "Khá"
             else "Trung bình"
println(result)
```

➡️ `if` trong Kotlin **là biểu thức** (có giá trị trả về) – khác với Java!

---

## 🧠 4. Cấu trúc `when` – Thay thế `switch-case`

Cú pháp:

```kotlin
when (giá_trị) {
    trường_hợp1 -> { ... }
    trường_hợp2 -> { ... }
    else -> { ... }
}
```

---

### 📋 Ví dụ 1 – Kiểm tra thứ trong tuần

```kotlin
val day = 3
when (day) {
    1 -> println("Thứ Hai")
    2 -> println("Thứ Ba")
    3 -> println("Thứ Tư")
    4 -> println("Thứ Năm")
    5 -> println("Thứ Sáu")
    6, 7 -> println("Cuối tuần")
    else -> println("Không hợp lệ")
}
```

💬 Em có thể gom nhiều giá trị cùng một nhánh (6,7 → cuối tuần).

---

### 📋 Ví dụ 2 – `when` không cần biểu thức

```kotlin
val number = 10
when {
    number < 0 -> println("Số âm")
    number == 0 -> println("Bằng 0")
    number > 0 -> println("Số dương")
}
```

💡 `when` không bắt buộc có giá trị so sánh – dùng để kiểm tra điều kiện phức tạp.

---

## ⚙️ 5. Kết hợp điều kiện logic

Em có thể dùng `&&`, `||`, `!` như trong bài trước:

```kotlin
val age = 20
val hasTicket = true

if (age >= 18 && hasTicket) {
    println("Được vào rạp phim.")
} else {
    println("Không đủ điều kiện.")
}
```

➡️ Output: `Được vào rạp phim.`

---

## 🧩 6. Biểu thức điều kiện ngắn gọn (Ternary Expression style)

Java có `a > b ? a : b`,
nhưng Kotlin không có toán tử `?:` kiểu đó — mà dùng `if` như biểu thức:

```kotlin
val a = 10
val b = 20
val max = if (a > b) a else b
println("Số lớn hơn là $max")
```

---

## 🧠 7. Khi nào nên dùng `if` và `when`?

| Trường hợp                  | Dùng cấu trúc                  |
| --------------------------- | ------------------------------ |
| So sánh nhỏ hơn, lớn hơn    | `if`                           |
| So sánh bằng nhau (rời rạc) | `when`                         |
| Nhiều điều kiện phức tạp    | `when { ... }`                 |
| Xử lý giá trị có thể null   | `when (value)` + `else -> ...` |

---

## 🧪 **Thực hành mini 1: Kiểm tra số chẵn/lẻ**

```kotlin
fun main() {
    val number = 13
    if (number % 2 == 0)
        println("$number là số chẵn")
    else
        println("$number là số lẻ")
}
```

---

## 🧪 **Thực hành mini 2: Phân loại học lực**

```kotlin
fun main() {
    val diem = 7.5
    val hocLuc = when {
        diem >= 8.5 -> "Giỏi"
        diem >= 6.5 -> "Khá"
        diem >= 5 -> "Trung bình"
        else -> "Yếu"
    }
    println("Học lực của bạn là: $hocLuc")
}
```

---

## 🧪 **Thực hành mini 3: Kiểm tra năm nhuận**

💡 Năm nhuận: chia hết cho 400 **hoặc** chia hết cho 4 nhưng không chia hết cho 100.

```kotlin
fun main() {
    val year = 2024
    if (year % 400 == 0 || (year % 4 == 0 && year % 100 != 0)) {
        println("$year là năm nhuận.")
    } else {
        println("$year không phải năm nhuận.")
    }
}
```

---

## 🏠 **Bài tập về nhà – Bài 3**

| Mức độ        | Bài tập                                                            | Gợi ý                                    |
| ------------- | ------------------------------------------------------------------ | ---------------------------------------- |
| 🟢 Cơ bản     | Kiểm tra tuổi người dùng và in “Trẻ em”, “Thanh niên”, “Người lớn” | Dùng `if...else if...else`               |
| 🟡 Trung bình | Nhập điểm → in xếp loại học lực                                    | Dùng `when`                              |
| 🔵 Nâng cao   | Nhập tháng → in số ngày trong tháng                                | Dùng `when` và xét tháng 2 là 28 hoặc 29 |
| 🟣 Thử thách  | Tạo menu lựa chọn (1.Thêm, 2.Sửa, 3.Xoá, 4.Thoát)                  | Dùng vòng lặp + `when`                   |

---

## 📚 Tổng kết Bài 3

| Kiến thức                    | Vai trò                              |         |                   |
| ---------------------------- | ------------------------------------ | ------- | ----------------- |
| `if`, `else if`, `else`      | Cấu trúc điều kiện cơ bản            |         |                   |
| `if` là biểu thức            | Có thể gán giá trị trực tiếp         |         |                   |
| `when`                       | Thay thế `switch`, dễ đọc, linh hoạt |         |                   |
| Toán tử logic (`&&`, `       |                                      | `, `!`) | Kết hợp điều kiện |
| Biểu thức điều kiện ngắn gọn | Viết code tinh gọn, dễ hiểu          |         |                   |

---

🎓 **Bài 4 (buổi tới):**

> *Thầy sẽ dạy “Vòng lặp trong Kotlin – for, while, do...while, range, break, continue” — để em làm được các chương trình tính toán, lặp danh sách, và xử lý dữ liệu hiệu quả.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 4: Vòng lặp trong Kotlin** không?
Bài này cực kỳ quan trọng vì giúp em “làm việc với dữ liệu lặp” – nền tảng cho việc hiển thị danh sách, adapter trong Android sau này.
