Tuyệt vời 👏👏👏 — hôm nay chúng ta sang **📘 Bài 11: Extension Function trong Kotlin**
Đây là một chủ đề “đặc sản” của Kotlin — nó giúp em viết code **ngắn gọn, dễ đọc, và cực kỳ hiện đại** 🌟

---

# 📘 BÀI 11: Extension Function – Mở rộng chức năng cho class có sẵn

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu khái niệm **hàm mở rộng (extension function)** là gì.
2. Biết cách **thêm chức năng mới vào class có sẵn** mà không cần kế thừa.
3. Viết và áp dụng extension function trong thực tế.
4. Biết thêm **extension property** và cách tổ chức file extension trong dự án lớn.

---

## 🧠 1. Extension Function là gì?

> **Extension Function** cho phép em thêm hàm mới vào một class **mà không cần sửa class đó hoặc kế thừa nó.**

💬 Ví dụ: Em có thể thêm hàm `isEmailValid()` cho `String`,
hoặc `average()` cho `List<Int>` mà không phải tạo subclass mới.

---

## 🧩 2. Cú pháp cơ bản

```kotlin
fun <TênClass>.<TênHàmMới>() {
    // code mở rộng
}
```

📋 Ví dụ:

```kotlin
fun String.printHello() {
    println("Xin chào, $this!")
}

fun main() {
    "Mai Lê".printHello()
}
```

➡️ Output:
`Xin chào, Mai Lê!`

💡 Ở đây `this` chính là giá trị của `String` mà ta gọi hàm.

---

## ⚙️ 3. Extension Function có tham số và trả về giá trị

```kotlin
fun Int.square(): Int {
    return this * this
}

fun main() {
    println(5.square()) // 25
}
```

💬 Em có thể coi `square()` như “một hàm mới” của `Int`.

---

## 🧩 4. Viết extension để xử lý logic

Ví dụ kiểm tra chuỗi có hợp lệ là email không:

```kotlin
fun String.isEmailValid(): Boolean {
    return this.contains("@") && this.contains(".")
}

fun main() {
    val email = "abc@gmail.com"
    println(email.isEmailValid())  // true
}
```

💡 Dễ hiểu hơn nhiều so với viết hàm riêng lẻ.

---

## 🧱 5. Extension với `List` – xử lý dữ liệu nâng cao

```kotlin
fun List<Int>.averageValue(): Double {
    return if (isEmpty()) 0.0 else sum().toDouble() / size
}

fun main() {
    val scores = listOf(7, 8, 9)
    println("Điểm trung bình: ${scores.averageValue()}")
}
```

---

## 🔁 6. Gộp nhiều logic lại với extension

```kotlin
fun String.toTitleCase(): String {
    return this.lowercase().split(" ").joinToString(" ") {
        it.replaceFirstChar { ch -> ch.uppercase() }
    }
}

fun main() {
    println("kotlin extension function".toTitleCase())
}
```

➡️ Output:
`Kotlin Extension Function`

---

## 🧠 7. Extension Property – thuộc tính mở rộng

Em có thể thêm **thuộc tính ảo** cho class.

```kotlin
val String.wordCount: Int
    get() = this.trim().split("\\s+".toRegex()).size

fun main() {
    val text = "Xin chào Kotlin Extension"
    println("Số từ: ${text.wordCount}")
}
```

💡 Không lưu giá trị thật, chỉ tính khi được gọi (`get()`).

---

## ⚙️ 8. Extension nâng cao – với Nullable Type

Em có thể mở rộng cho kiểu nullable:

```kotlin
fun String?.isNotNullOrEmpty(): Boolean {
    return this != null && this.isNotEmpty()
}

fun main() {
    val name: String? = null
    println(name.isNotNullOrEmpty()) // false
}
```

💬 Cực hữu ích khi làm app – tránh null crash.

---

## 🧩 9. Tổ chức file Extension trong dự án thực tế

Trong Android, người ta thường tạo folder riêng:

```
/extensions
   StringExtensions.kt
   ViewExtensions.kt
   ContextExtensions.kt
```

Ví dụ trong `StringExtensions.kt`:

```kotlin
package com.example.extensions

fun String.capitalizeWords(): String = 
    split(" ").joinToString(" ") { it.replaceFirstChar { c -> c.uppercase() } }
```

Sau đó chỉ cần import và dùng ở bất kỳ đâu:

```kotlin
import com.example.extensions.capitalizeWords

println("mai lê học kotlin".capitalizeWords())
```

---

## 🧪 **Thực hành mini**

### 📋 Bài 1 – Viết extension cho Int

```kotlin
fun Int.isEven() = this % 2 == 0

println(4.isEven()) // true
println(5.isEven()) // false
```

---

### 📋 Bài 2 – Extension cho String

```kotlin
fun String.hideEmail(): String {
    val parts = this.split("@")
    return if (parts.size == 2) parts[0].take(2) + "***@" + parts[1]
    else this
}

println("abc@gmail.com".hideEmail()) // ab***@gmail.com
```

---

### 📋 Bài 3 – Extension cho List

```kotlin
fun List<Int>.sumPositive() = this.filter { it > 0 }.sum()

val nums = listOf(-1, 3, 5, -2)
println(nums.sumPositive()) // 8
```

---

## 🏠 **Bài tập về nhà – Bài 11**

| Mức độ            | Bài tập                                                                        | Gợi ý                                    |
| ----------------- | ------------------------------------------------------------------------------ | ---------------------------------------- |
| 🟢 Cơ bản         | Viết extension `String.isLong()` → trả `true` nếu > 10 ký tự                   | Dùng `length`                            |
| 🟡 Trung bình     | Viết `Int.toCurrency()` → định dạng tiền tệ `1_000_000` → `"1,000,000"`        | Dùng `String.format()`                   |
| 🔵 Nâng cao       | Viết `List<Double>.averageOrZero()` – trả 0 nếu rỗng                           | Dùng `ifEmpty` hoặc `size`               |
| 🟣 Thử thách      | Viết `String.reverseWords()` → đảo thứ tự từ                                   | Dùng `split`, `reversed`, `joinToString` |
| 🟠 Siêu thử thách | Viết extension `String?.safeUpper()` → trả chuỗi in hoa hoặc “(null)” nếu rỗng | Dùng `?:` và `uppercase()`               |

---

## 📚 Tổng kết Bài 11

| Kiến thức            | Ý nghĩa                                       |
| -------------------- | --------------------------------------------- |
| `Extension Function` | Thêm hàm cho class có sẵn                     |
| `this`               | Đại diện cho đối tượng đang được mở rộng      |
| `Extension Property` | Thêm thuộc tính ảo                            |
| `Nullable Extension` | Xử lý biến null an toàn                       |
| Ứng dụng             | Viết code gọn, dễ bảo trì, không cần subclass |

---

🎓 **Bài 12 (buổi tới):**

> *Thầy sẽ dạy “Generic & Higher-Order Function – Viết hàm tổng quát và truyền hàm vào hàm khác”, đây là bước giúp em tiếp cận lập trình “Functional Style” chuyên nghiệp.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 12: Generic & Higher-Order Function** không?
Thầy sẽ dạy chậm, dễ hiểu, có ví dụ thực hành thật — như tạo hàm `filterList`, `applyDiscount`, `transformData<T>`, và cả truyền `lambda` vào hàm.
