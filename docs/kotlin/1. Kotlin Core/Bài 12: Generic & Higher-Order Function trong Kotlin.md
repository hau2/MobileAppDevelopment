Tuyệt vời 👏👏👏 — hôm nay chúng ta bước sang **📘 Bài 12: Generic & Higher-Order Function trong Kotlin**
Đây là bài cực kỳ quan trọng, giúp em **viết code ngắn gọn, tái sử dụng cao, và mang phong cách “Functional Programming” hiện đại** — điều mà Kotlin làm rất xuất sắc 💪

---

# 📘 BÀI 12: Generic & Higher-Order Function – Hàm tổng quát và hàm bậc cao

---

## 🎯 Mục tiêu bài học

Sau bài học này, em sẽ:

1. Hiểu **Generic** là gì, tại sao dùng.
2. Biết viết hàm & class **tổng quát cho nhiều kiểu dữ liệu**.
3. Hiểu **Higher-Order Function** – hàm có thể nhận hàm khác làm tham số.
4. Ứng dụng lambda và inline function trong lập trình thực tế.

---

## 🧠 1. Generic là gì?

> **Generic** cho phép em viết code “một lần” nhưng dùng cho **nhiều kiểu dữ liệu khác nhau**
> (giống như “mẫu khuôn” cho kiểu dữ liệu).

---

### 📋 Ví dụ không có Generic

```kotlin
fun printIntList(list: List<Int>) {
    list.forEach { println(it) }
}

fun printStringList(list: List<String>) {
    list.forEach { println(it) }
}
```

💬 Hai hàm trên giống hệt nhau, chỉ khác kiểu dữ liệu.

---

### 📋 Viết lại bằng Generic

```kotlin
fun <T> printList(list: List<T>) {
    list.forEach { println(it) }
}

fun main() {
    printList(listOf(1, 2, 3))
    printList(listOf("A", "B", "C"))
}
```

➡️ `T` là **type parameter** – đại diện cho “kiểu dữ liệu tùy ý”.

---

## ⚙️ 2. Generic trong Class

```kotlin
class Box<T>(val item: T) {
    fun show() = println("Hộp chứa: $item")
}

fun main() {
    val intBox = Box(123)
    val strBox = Box("Kotlin")
    intBox.show()
    strBox.show()
}
```

➡️ Output:

```
Hộp chứa: 123
Hộp chứa: Kotlin
```

💡 Giống như Java’s `Box<T>` nhưng ngắn gọn và an toàn hơn.

---

## 💡 3. Ràng buộc kiểu (`where`, `:`)

Nếu muốn giới hạn kiểu Generic – ví dụ chỉ chấp nhận số:

```kotlin
fun <T : Number> sum(a: T, b: T): Double {
    return a.toDouble() + b.toDouble()
}

println(sum(3, 4))     // 7.0
println(sum(2.5, 4.5)) // 7.0
```

💬 `T : Number` nghĩa là T phải là hoặc kế thừa `Number`.

---

## 🧩 4. Hàm bậc cao (Higher-Order Function)

> Là hàm **nhận hàm khác làm tham số**, hoặc **trả về hàm khác**.

💡 Kotlin coi hàm là “công dân hạng nhất” → có thể truyền hàm như biến!

---

### 📋 Ví dụ cơ bản

```kotlin
fun operate(a: Int, b: Int, action: (Int, Int) -> Int): Int {
    return action(a, b)
}

fun main() {
    val sum = operate(3, 4) { x, y -> x + y }
    val mul = operate(3, 4) { x, y -> x * y }

    println(sum) // 7
    println(mul) // 12
}
```

💬 Hàm `operate` nhận tham số thứ 3 là **một hàm lambda**.

---

## ⚡️ 5. Cú pháp Lambda

Lambda là cách viết hàm ngắn gọn.

```kotlin
val greet: (String) -> Unit = { name ->
    println("Xin chào $name")
}

greet("Mai Lê")
```

➡️ Output: `Xin chào Mai Lê`

---

## 🧠 6. Truyền Lambda vào hàm

```kotlin
fun repeatAction(times: Int, action: () -> Unit) {
    repeat(times) { action() }
}

fun main() {
    repeatAction(3) { println("Học Kotlin vui quá!") }
}
```

➡️ Output:

```
Học Kotlin vui quá!
Học Kotlin vui quá!
Học Kotlin vui quá!
```

---

## 🧩 7. Higher-Order Function trả về hàm khác

```kotlin
fun multiplier(factor: Int): (Int) -> Int {
    return { num -> num * factor }
}

fun main() {
    val double = multiplier(2)
    println(double(5)) // 10
}
```

💡 `multiplier(2)` trả về một hàm nhân đôi giá trị.

---

## 🧱 8. Higher-Order Function kết hợp Generic

```kotlin
fun <T> filterList(list: List<T>, condition: (T) -> Boolean): List<T> {
    val result = mutableListOf<T>()
    for (item in list) {
        if (condition(item)) result.add(item)
    }
    return result
}

fun main() {
    val nums = listOf(1, 2, 3, 4, 5)
    val even = filterList(nums) { it % 2 == 0 }
    println(even) // [2, 4]
}
```

---

## ⚙️ 9. Inline Function (tối ưu hiệu năng)

Khi gọi nhiều lambda, Kotlin có thể sinh ra overhead.
Dùng `inline` để giảm chi phí gọi hàm:

```kotlin
inline fun measure(block: () -> Unit) {
    val start = System.currentTimeMillis()
    block()
    val end = System.currentTimeMillis()
    println("Thời gian chạy: ${end - start} ms")
}

fun main() {
    measure {
        repeat(1_000_000) { }
    }
}
```

---

## 🧪 **Thực hành mini**

### 📋 Bài 1 – Viết hàm tổng quát

```kotlin
fun <T> printElement(element: T) {
    println("Giá trị: $element")
}

printElement("Kotlin")
printElement(123)
```

---

### 📋 Bài 2 – Viết hàm tính toán linh hoạt

```kotlin
fun operate(x: Int, y: Int, op: (Int, Int) -> Int): Int = op(x, y)

println(operate(5, 3) { a, b -> a + b })
println(operate(5, 3) { a, b -> a * b })
```

---

### 📋 Bài 3 – Kết hợp Generic & Lambda

```kotlin
fun <T> transformList(list: List<T>, action: (T) -> T): List<T> {
    return list.map { action(it) }
}

val names = listOf("mai", "lê", "công")
println(transformList(names) { it.uppercase() })
```

---

## 🏠 **Bài tập về nhà – Bài 12**

| Mức độ            | Bài tập                                                                            | Gợi ý                             |
| ----------------- | ---------------------------------------------------------------------------------- | --------------------------------- |
| 🟢 Cơ bản         | Viết `fun <T> echo(value: T)` → in ra giá trị                                      | Generic đơn giản                  |
| 🟡 Trung bình     | Viết `fun calc(a, b, op)` – truyền lambda cộng/trừ/nhân/chia                       | Lambda                            |
| 🔵 Nâng cao       | Viết `fun <T> filter(list, condition)` → lọc dữ liệu                               | Dùng `mutableListOf`              |
| 🟣 Thử thách      | Viết `fun <T> measureResult(block: () -> T)` – đo thời gian chạy và trả về kết quả | Dùng `System.currentTimeMillis()` |
| 🟠 Siêu thử thách | Viết hàm `makeLogger(prefix: String): (String) -> Unit` → tạo hàm log tự động      | Trả về lambda                     |

---

## 📚 Tổng kết Bài 12

| Khái niệm               | Mô tả                                                    |
| ----------------------- | -------------------------------------------------------- |
| `Generic <T>`           | Viết code cho mọi kiểu dữ liệu                           |
| `Higher-Order Function` | Hàm nhận/trả hàm khác                                    |
| `Lambda`                | Cách viết hàm ngắn gọn                                   |
| `inline`                | Tối ưu hiệu năng khi dùng lambda                         |
| Ứng dụng                | Filter list, xử lý dữ liệu, viết DSL, mô hình functional |

---

🎓 **Bài 13 (buổi tới):**

> *Thầy sẽ dạy “Coroutines nâng cao – launch, async, withContext, suspend function, scope” — giúp em hiểu sâu hơn về bất đồng bộ và chuẩn bị cho phần làm app thực tế.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 13: Coroutines nâng cao** không?
Thầy sẽ dạy chậm – dễ hiểu – có ví dụ “giả lập tải dữ liệu API song song và cập nhật UI”.
