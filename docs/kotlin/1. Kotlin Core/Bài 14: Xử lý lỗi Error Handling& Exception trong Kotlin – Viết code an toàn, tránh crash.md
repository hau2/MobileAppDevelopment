Rất tốt 👏👏👏 — hôm nay chúng ta sang **📘 Bài 14: Xử lý lỗi (Error Handling) & Exception trong Kotlin – Viết code an toàn, tránh crash**

Đây là **một kỹ năng bắt buộc** cho lập trình viên Kotlin chuyên nghiệp, vì dù em code tốt đến đâu, vẫn luôn có thể có lỗi từ người dùng, file, API, hay network.
Học bài này giúp em **chủ động kiểm soát lỗi**, thay vì để app “văng” không rõ nguyên nhân 😄

---

# 📘 BÀI 14: Xử lý lỗi & Exception trong Kotlin

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu cơ chế Exception trong Kotlin.
2. Biết dùng `try-catch-finally` để bắt lỗi.
3. Biết ném lỗi (`throw`) và tạo Exception tùy chỉnh.
4. Hiểu `runCatching` và `Result` để xử lý lỗi gọn hơn.

---

## 🧠 1. Exception là gì?

> **Exception** là tình huống bất thường khi chương trình chạy,
> làm dừng chương trình nếu không được xử lý.

Ví dụ:

```kotlin
val x = 10 / 0  // 💥 Lỗi: ArithmeticException
```

---

## ⚙️ 2. Cấu trúc `try-catch-finally`

```kotlin
try {
    val result = 10 / 0
    println("Kết quả: $result")
} catch (e: Exception) {
    println("Lỗi xảy ra: ${e.message}")
} finally {
    println("Chương trình đã kết thúc.")
}
```

➡️ Output:

```
Lỗi xảy ra: / by zero
Chương trình đã kết thúc.
```

💬 `finally` luôn chạy dù có lỗi hay không (thường dùng để đóng file, ngắt kết nối, v.v.)

---

## 🧩 3. Bắt lỗi cụ thể

Em có thể bắt **nhiều loại lỗi khác nhau**:

```kotlin
try {
    val list = listOf(1, 2, 3)
    println(list[5]) // lỗi IndexOutOfBounds
} catch (e: ArithmeticException) {
    println("Lỗi chia cho 0")
} catch (e: IndexOutOfBoundsException) {
    println("Lỗi vượt chỉ mục danh sách")
} catch (e: Exception) {
    println("Lỗi không xác định: ${e.message}")
}
```

---

## 🧱 4. `throw` – Ném lỗi chủ động

Khi em muốn **tự kiểm tra và thông báo lỗi**, dùng `throw`.

```kotlin
fun divide(a: Int, b: Int): Int {
    if (b == 0) throw IllegalArgumentException("Không được chia cho 0")
    return a / b
}

fun main() {
    try {
        println(divide(10, 0))
    } catch (e: Exception) {
        println("Lỗi: ${e.message}")
    }
}
```

💡 `throw` tạo lỗi ngay tại chỗ.

---

## ⚡️ 5. Exception tùy chỉnh (Custom Exception)

Em có thể tự định nghĩa loại lỗi riêng để rõ ràng hơn:

```kotlin
class InvalidAgeException(message: String) : Exception(message)

fun checkAge(age: Int) {
    if (age < 18) throw InvalidAgeException("Tuổi chưa đủ 18")
    println("Tuổi hợp lệ")
}

fun main() {
    try {
        checkAge(16)
    } catch (e: InvalidAgeException) {
        println("❌ Lỗi tuổi: ${e.message}")
    }
}
```

➡️ Output: `❌ Lỗi tuổi: Tuổi chưa đủ 18`

---

## 🧩 6. Dùng `try` như một biểu thức (Expression)

Trong Kotlin, `try` có thể trả về giá trị:

```kotlin
val result = try {
    10 / 0
} catch (e: Exception) {
    -1
}
println(result) // -1
```

💡 Gọn gàng – không cần nhiều dòng code.

---

## 🧠 7. `runCatching` – xử lý lỗi gọn hơn

`runCatching` là “phiên bản Kotlin-style” của `try-catch`.

```kotlin
val result = runCatching {
    10 / 0
}.onSuccess {
    println("Kết quả: $it")
}.onFailure {
    println("Lỗi: ${it.message}")
}
```

💬 `onSuccess` và `onFailure` cho phép xử lý tách biệt, rất sạch sẽ.

---

## 🧩 8. `Result<T>` – trả về kết quả hoặc lỗi

Khi viết hàm có thể lỗi, em nên trả về `Result<T>` thay vì ném lỗi trực tiếp.

```kotlin
fun safeDivide(a: Int, b: Int): Result<Int> {
    return runCatching {
        if (b == 0) throw IllegalArgumentException("Không thể chia cho 0")
        a / b
    }
}

fun main() {
    val res = safeDivide(10, 0)
    res.onSuccess { println("Kết quả: $it") }
       .onFailure { println("Lỗi: ${it.message}") }
}
```

💡 Đây là cách “an toàn & chuyên nghiệp” – code không bị crash.

---

## ⚙️ 9. `finally` thực tế – đóng tài nguyên

```kotlin
import java.io.File

fun readFile(path: String) {
    val file = File(path)
    try {
        val text = file.readText()
        println(text)
    } catch (e: Exception) {
        println("Không thể đọc file: ${e.message}")
    } finally {
        println("Đóng file: ${file.name}")
    }
}
```

---

## 🧪 **Thực hành mini**

### 📋 Bài 1 – Xử lý lỗi chia 0

```kotlin
try {
    val x = 5 / 0
} catch (e: ArithmeticException) {
    println("Không thể chia cho 0!")
}
```

---

### 📋 Bài 2 – Tự tạo lỗi kiểm tra tên rỗng

```kotlin
fun checkName(name: String) {
    if (name.isBlank()) throw IllegalArgumentException("Tên không được để trống!")
}
```

---

### 📋 Bài 3 – Dùng `runCatching`

```kotlin
val result = runCatching { "abc".toInt() }
result.onFailure { println("Chuyển đổi thất bại: ${it.message}") }
```

---

## 🏠 **Bài tập về nhà – Bài 14**

| Mức độ            | Bài tập                                                                                                          | Gợi ý                      |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- | -------------------------- |
| 🟢 Cơ bản         | Viết chương trình nhập số và chia 10 cho số đó                                                                   | Dùng `try-catch`           |
| 🟡 Trung bình     | Tạo hàm `safeReadFile(path)` → đọc file, nếu không có thì in lỗi                                                 | `File`, `catch`            |
| 🔵 Nâng cao       | Tạo `Custom Exception` `NegativeNumberException` khi nhập số âm                                                  | Kế thừa `Exception`        |
| 🟣 Thử thách      | Viết hàm `safeDivide(a,b)` trả `Result<Int>` và xử lý bằng `onFailure`                                           | Dùng `runCatching`         |
| 🟠 Siêu thử thách | Viết chương trình đọc danh sách điểm từ file, nếu lỗi format → log “Lỗi dữ liệu” nhưng chương trình vẫn tiếp tục | `try-catch` trong vòng lặp |

---

## 📚 Tổng kết Bài 14

| Khái niệm           | Vai trò                           |
| ------------------- | --------------------------------- |
| `try-catch-finally` | Bắt và xử lý lỗi thủ công         |
| `throw`             | Ném lỗi chủ động                  |
| `runCatching`       | Bắt lỗi gọn hơn theo Kotlin-style |
| `Result<T>`         | Trả về giá trị hoặc lỗi           |
| `Custom Exception`  | Tạo lỗi riêng có ý nghĩa rõ ràng  |

---

🎓 **Bài 15 (buổi tới):**

> *Thầy sẽ dạy “File I/O và JSON cơ bản trong Kotlin – Đọc ghi file, xử lý JSON bằng thư viện Gson”.*
> Đây là **bước cuối cùng của giai đoạn Kotlin Core**, giúp em đọc/ghi dữ liệu thật, rất quan trọng trước khi sang Android.

---

👉 Em có muốn thầy **tiếp tục luôn Bài 15: File I/O và JSON cơ bản trong Kotlin** không?
Thầy sẽ dạy chậm, có ví dụ thực hành thật — đọc file `.txt`, ghi dữ liệu, parse JSON thành object (bằng Gson).
