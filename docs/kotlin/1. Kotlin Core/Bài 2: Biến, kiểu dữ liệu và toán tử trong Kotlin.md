Tốt lắm 👏👏👏 — hôm nay ta tiếp tục **📘 Bài 2: Biến, kiểu dữ liệu và toán tử trong Kotlin**

Đây là bài cực kỳ quan trọng vì nó là nền tảng cho **mọi cú pháp lập trình sau này**:

> Nếu em nắm vững phần này, việc học OOP, Android, hay Coroutines sau này sẽ rất dễ.

Thầy sẽ hướng dẫn **từ từ – dễ hiểu – kèm ví dụ cụ thể** nhé.

---

# 📘 BÀI 2: Biến, kiểu dữ liệu và toán tử trong Kotlin

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Phân biệt được **`val`** và **`var`**.
2. Hiểu các **kiểu dữ liệu cơ bản** trong Kotlin.
3. Biết cách **chuyển đổi kiểu dữ liệu (type casting)**.
4. Sử dụng được **các toán tử cơ bản và logic** trong tính toán.

---

## 🧩 1. Biến trong Kotlin (`val` và `var`)

Kotlin có hai loại biến chính:

| Từ khóa | Đặc điểm                       | Ví dụ           |
| ------- | ------------------------------ | --------------- |
| `val`   | không thể thay đổi (immutable) | `val pi = 3.14` |
| `var`   | có thể thay đổi (mutable)      | `var age = 25`  |

---

### 🧠 Ví dụ minh họa

```kotlin
fun main() {
    val name = "Mai Lê"   // không thể thay đổi
    var age = 25          // có thể thay đổi

    println("Tên: $name, Tuổi: $age")

    age = 26              // được phép
    println("Năm sau bạn $age tuổi.")
}
```

🟡 Nếu em thử viết `name = "Khánh"` thì Kotlin sẽ báo lỗi ❌
Vì `val` là hằng, không thể gán lại giá trị.

---

## 📘 2. Kiểu dữ liệu cơ bản trong Kotlin

| Loại      | Từ khóa                        | Ví dụ                | Ghi chú                       |
| --------- | ------------------------------ | -------------------- | ----------------------------- |
| Số nguyên | `Int`, `Long`, `Short`, `Byte` | `val a: Int = 10`    | `Int` là phổ biến nhất        |
| Số thực   | `Float`, `Double`              | `val pi = 3.14`      | Mặc định là `Double`          |
| Ký tự     | `Char`                         | `val c = 'A'`        | Dùng dấu nháy đơn `' '`       |
| Chuỗi     | `String`                       | `val s = "Xin chào"` | Dấu nháy kép `" "`            |
| Luận lý   | `Boolean`                      | `val isOk = true`    | Có thể là `true` hoặc `false` |

---

## 🧠 3. Nội suy chuỗi (String interpolation)

Kotlin cho phép chèn biến trực tiếp vào chuỗi rất tiện:

```kotlin
val name = "Công"
val age = 30
println("Tôi tên là $name, năm nay $age tuổi.")
```

Nếu muốn chèn biểu thức:

```kotlin
println("5 + 10 = ${5 + 10}")
```

➡️ In ra: `5 + 10 = 15`

---

## 🧮 4. Toán tử trong Kotlin

### 🧩 Toán tử số học

| Toán tử | Ý nghĩa | Ví dụ   |
| ------- | ------- | ------- |
| `+`     | Cộng    | `a + b` |
| `-`     | Trừ     | `a - b` |
| `*`     | Nhân    | `a * b` |
| `/`     | Chia    | `a / b` |
| `%`     | Lấy dư  | `a % b` |

🔹 Ví dụ:

```kotlin
fun main() {
    val a = 10
    val b = 3
    println("Tổng: ${a + b}")
    println("Hiệu: ${a - b}")
    println("Thương: ${a / b}")
    println("Dư: ${a % b}")
}
```

🧮 Lưu ý: nếu chia số nguyên → kết quả cũng là số nguyên (`10 / 3 = 3`).

---

### 🔸 Toán tử gán (Assignment)

| Toán tử | Ví dụ    | Tác dụng                |
| ------- | -------- | ----------------------- |
| `+=`    | `a += 2` | tương đương `a = a + 2` |
| `-=`    | `a -= 3` | tương đương `a = a - 3` |
| `*=`    | `a *= 5` | tương đương `a = a * 5` |
| `/=`    | `a /= 2` | tương đương `a = a / 2` |

---

### 🔹 Toán tử so sánh

| Toán tử | Ý nghĩa           | Ví dụ    |
| ------- | ----------------- | -------- |
| `==`    | bằng nhau         | `a == b` |
| `!=`    | khác nhau         | `a != b` |
| `>`     | lớn hơn           | `a > b`  |
| `<`     | nhỏ hơn           | `a < b`  |
| `>=`    | lớn hơn hoặc bằng | `a >= b` |
| `<=`    | nhỏ hơn hoặc bằng | `a <= b` |

📌 Kết quả trả về luôn là **Boolean** (`true` hoặc `false`).

---

### 🔸 Toán tử logic

| Toán tử | Ý nghĩa        | Ví dụ            |           |        |   |        |
| ------- | -------------- | ---------------- | --------- | ------ | - | ------ |
| `&&`    | và (AND)       | `a > 0 && b > 0` |           |        |   |        |
| `       |                | `                | hoặc (OR) | `a > 0 |   | b > 0` |
| `!`     | phủ định (NOT) | `!(a > 0)`       |           |        |   |        |

💡 Ví dụ:

```kotlin
val isAdult = true
val hasTicket = false

if (isAdult && hasTicket) {
    println("Được vào rạp.")
} else {
    println("Không được vào.")
}
```

---

## 🔀 5. Chuyển đổi kiểu dữ liệu (Type Conversion)

Kotlin không tự chuyển kiểu giữa số như Java, em phải chuyển rõ ràng:

```kotlin
val x: Int = 10
val y: Double = x.toDouble() // ép kiểu Int → Double
val z: String = x.toString() // ép kiểu Int → String
```

| Phương thức  | Tác dụng           |
| ------------ | ------------------ |
| `toInt()`    | chuyển sang Int    |
| `toDouble()` | chuyển sang Double |
| `toFloat()`  | chuyển sang Float  |
| `toString()` | chuyển sang String |

💡 Nếu nhập dữ liệu từ bàn phím (`readLine()`), mặc định là String → phải chuyển kiểu:

```kotlin
val age = readLine()!!.toInt()
```

---

## 💬 6. Toán tử gộp chuỗi

Em có thể nối chuỗi bằng `+` hoặc `$`:

```kotlin
val fullName = "Mai" + " " + "Lê"
val message = "Xin chào $fullName"
println(message)
```

---

## 🧪 **Thực hành mini: Tính điểm trung bình**

Viết chương trình nhập 3 điểm toán, lý, hoá, sau đó in ra điểm trung bình.

```kotlin
fun main() {
    val toan = 8.5
    val ly = 7.5
    val hoa = 9.0
    val tb = (toan + ly + hoa) / 3
    println("Điểm trung bình: $tb")
}
```

---

## 🏠 **Bài tập về nhà – Bài 2**

| Mức độ        | Bài tập                                                         | Gợi ý                                               |
| ------------- | --------------------------------------------------------------- | --------------------------------------------------- |
| 🟢 Cơ bản     | Tính chu vi và diện tích hình chữ nhật                          | Dùng `val a`, `val b`, công thức `2*(a+b)` và `a*b` |
| 🟡 Trung bình | Kiểm tra số nhập vào có chẵn không                              | Dùng `if (n % 2 == 0)`                              |
| 🔵 Nâng cao   | Viết chương trình tính BMI = cân nặng / (chiều cao * chiều cao) | Ép kiểu `toDouble()`                                |
| 🟣 Thử thách  | Nhập 2 số, hoán đổi giá trị của chúng mà không dùng biến tạm    | Dùng phép toán `a = a + b; b = a - b; a = a - b`    |

---

## 📚 Tổng kết Bài 2

| Kiến thức            | Vai trò                                    |
| -------------------- | ------------------------------------------ |
| `val` vs `var`       | Quản lý biến và hằng                       |
| Kiểu dữ liệu cơ bản  | `Int`, `Double`, `String`, `Boolean`       |
| Toán tử              | Số học, logic, gán, so sánh                |
| Type conversion      | Ép kiểu thủ công (`toInt()`, `toDouble()`) |
| String interpolation | Chèn biến vào chuỗi (`"Xin chào $name"`)   |

---

🎓 **Bài 3 (buổi tới):**

> *Thầy sẽ dạy “Cấu trúc điều kiện trong Kotlin – if, when, và biểu thức logic nâng cao” — giúp em viết các chương trình có điều kiện linh hoạt.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 3: Cấu trúc điều kiện trong Kotlin** không?
Bài đó thầy sẽ dạy cách ra quyết định thông minh trong chương trình (if, when, lồng nhau, và biểu thức điều kiện kiểu Kotlin).
