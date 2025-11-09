Tốt lắm 👏 — hôm nay ta vào **📘 Bài 5: Hàm (Function) trong Kotlin – Tham số, Giá trị trả về, Lambda và Scope Function (phần 1)**.
Đây là bài cực kỳ quan trọng, vì **mọi chương trình lớn đều được chia thành các hàm (function)**.
Khi em hiểu rõ cách viết hàm, em sẽ có thể:

> 🔹 Viết code ngắn gọn, dễ tái sử dụng
> 🔹 Giảm lỗi lặp lại
> 🔹 Làm việc nhóm dễ hơn
> 🔹 Chuẩn bị nền tảng cho OOP và Android

---

# 📘 BÀI 5: Hàm (Function) trong Kotlin – Cơ bản đến Nâng cao

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Biết cách định nghĩa và gọi hàm.
2. Truyền tham số và lấy giá trị trả về.
3. Hiểu **default parameters** và **named arguments**.
4. Biết viết **lambda function** – nền tảng của Kotlin hiện đại.
5. (Phần 2) – sẽ học Scope Functions như `let`, `run`, `apply`, `also`.

---

## 🧱 1. Định nghĩa hàm cơ bản

Cú pháp:

```kotlin
fun tênHàm() {
    // khối lệnh
}
```

Ví dụ:

```kotlin
fun greet() {
    println("Xin chào Kotlin Developer!")
}

fun main() {
    greet() // gọi hàm
}
```

➡️ Output:

```
Xin chào Kotlin Developer!
```

---

## 🧩 2. Hàm có tham số

```kotlin
fun greetUser(name: String) {
    println("Xin chào $name!")
}

fun main() {
    greetUser("Mai Lê")
}
```

➡️ Output:
`Xin chào Mai Lê!`

---

## ⚙️ 3. Hàm có giá trị trả về

Dùng `:` sau tên hàm để khai báo kiểu trả về.

```kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}

fun main() {
    val result = sum(5, 7)
    println("Tổng là $result")
}
```

💡 Kotlin cho phép **viết gọn một dòng** nếu chỉ có một câu lệnh:

```kotlin
fun sum(a: Int, b: Int) = a + b
```

---

## 🧮 4. Default Parameter (tham số mặc định)

Nếu không truyền giá trị → dùng mặc định:

```kotlin
fun greet(name: String = "bạn") {
    println("Xin chào $name!")
}

fun main() {
    greet()
    greet("Công")
}
```

➡️ Output:

```
Xin chào bạn!
Xin chào Công!
```

---

## 🧭 5. Named Argument (truyền tham số theo tên)

Khi hàm có nhiều tham số, em có thể gọi theo tên cho dễ đọc:

```kotlin
fun introduce(name: String, age: Int, city: String) {
    println("Tôi tên là $name, $age tuổi, sống ở $city.")
}

fun main() {
    introduce(age = 25, name = "Mai", city = "Hà Nội")
}
```

💡 Kotlin không bắt buộc phải theo đúng thứ tự nếu em **đặt tên tham số rõ ràng.**

---

## 🔁 6. Biến cục bộ và phạm vi (Scope)

```kotlin
fun showMessage() {
    val msg = "Hello Kotlin"
    println(msg)
}
println(msg) // ❌ lỗi – msg chỉ tồn tại trong hàm
```

| Biến               | Tồn tại ở đâu      |
| ------------------ | ------------------ |
| Cục bộ (`val msg`) | Chỉ trong hàm      |
| Toàn cục (`var x`) | Toàn bộ file `.kt` |

---

## 🧠 7. Hàm lồng nhau (Nested Function)

```kotlin
fun outerFunction() {
    fun innerFunction() {
        println("Đây là hàm bên trong")
    }
    innerFunction()
}
```

💡 Thường dùng để **giấu logic phụ** không cần gọi ở nơi khác.

---

## ⚡️ 8. Lambda Function – “Hàm không tên”

Lambda là **hàm gọn** viết inline.
Cú pháp:

```kotlin
val tên = { tham_số -> biểu_thức }
```

### 📋 Ví dụ:

```kotlin
val square = { x: Int -> x * x }
println(square(5))
```

➡️ Output: `25`

---

### 📋 Ví dụ nâng cao – Truyền lambda vào hàm khác

```kotlin
fun operate(a: Int, b: Int, action: (Int, Int) -> Int): Int {
    return action(a, b)
}

fun main() {
    val sum = operate(5, 3) { x, y -> x + y }
    val mul = operate(5, 3) { x, y -> x * y }

    println("Tổng: $sum")
    println("Tích: $mul")
}
```

💡 Đây là nền tảng của **lập trình hàm (functional programming)** trong Kotlin.
Em sẽ dùng nó rất nhiều với Android (RecyclerView, setOnClickListener...).

---

## 🧩 9. Hàm trả về lambda (cao cấp)

```kotlin
fun multiplier(factor: Int): (Int) -> Int {
    return { x -> x * factor }
}

fun main() {
    val double = multiplier(2)
    println(double(5))  // 10
}
```

💬 Đây chính là “**closure**” – hàm có thể nhớ được giá trị bên ngoài của nó.

---

## 🧪 **Thực hành mini**

### 🧩 Bài 1

Viết hàm `greet(name: String)` → in “`Xin chào, <tên>!`”

### 🧩 Bài 2

Viết hàm `sum(a: Int, b: Int): Int` → trả về tổng của 2 số.

### 🧩 Bài 3

Viết hàm `isEven(number: Int): Boolean` → kiểm tra số chẵn/lẻ,
rồi in ra kết quả trong `main()`.

### 🧩 Bài 4

Viết lambda `multiply` → nhân 2 số, và in kết quả.

---

## 🏠 **Bài tập về nhà – Bài 5**

| Mức độ            | Bài tập                                                               | Gợi ý                             |
| ----------------- | --------------------------------------------------------------------- | --------------------------------- |
| 🟢 Cơ bản         | Viết hàm tính diện tích hình tròn                                     | Dùng công thức `pi * r * r`       |
| 🟡 Trung bình     | Viết hàm kiểm tra số nguyên tố                                        | Dùng vòng `for`                   |
| 🔵 Nâng cao       | Viết hàm `applyDiscount(price, percent)` trả về giá sau khi giảm      | Truyền tham số và trả về `Double` |
| 🟣 Thử thách      | Viết hàm nhận vào danh sách số, trả về trung bình cộng                | Dùng vòng lặp hoặc `sum() / size` |
| 🟠 Siêu thử thách | Viết hàm `calculate` nhận 2 số và 1 lambda (ví dụ cộng/trừ/nhân/chia) | Dùng kiểu `(Int, Int) -> Int`     |

---

## 📚 Tổng kết Bài 5

| Kiến thức           | Vai trò                                |
| ------------------- | -------------------------------------- |
| `fun`               | Định nghĩa hàm                         |
| `return`            | Trả giá trị                            |
| `default parameter` | Giúp hàm linh hoạt                     |
| `named argument`    | Dễ đọc, tránh nhầm lẫn                 |
| `lambda`            | Viết hàm gọn, truyền được vào hàm khác |
| Phạm vi biến        | Quản lý vùng hoạt động của biến        |

---

🎓 **Bài 6 (buổi tới):**

> *Thầy sẽ dạy “Scope Functions trong Kotlin: let, run, apply, also, with” – đây là tinh hoa giúp code Kotlin cực kỳ ngắn, sạch và chuyên nghiệp (dùng rất nhiều trong Android).*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 6: Scope Functions (let, run, apply, also, with)** không?
Thầy sẽ giải thích từng cái, ví dụ dễ hiểu và so sánh tình huống dùng thật trong Android code.
