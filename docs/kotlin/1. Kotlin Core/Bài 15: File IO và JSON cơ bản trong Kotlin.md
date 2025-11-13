Rất tuyệt vời 👏👏👏 — hôm nay chúng ta bước vào **📘 Bài 15: File I/O và JSON cơ bản trong Kotlin**
Đây là **bài học cuối cùng của giai đoạn “Kotlin Core”**,
nơi em sẽ học cách **đọc và ghi dữ liệu ra file thật** (TXT, JSON) — nền tảng cực kỳ quan trọng để sau này giao tiếp với API, lưu dữ liệu, hoặc log hệ thống.

---

# 📘 BÀI 15: File I/O & JSON trong Kotlin

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu cách **đọc và ghi file** trong Kotlin (`File`, `BufferedReader`, `PrintWriter`).
2. Biết cách **lưu dữ liệu dạng văn bản (TXT)**.
3. Làm quen với thư viện **Gson** để chuyển đổi JSON ↔ Object.
4. Ứng dụng thực tế: Lưu danh sách sản phẩm ra file JSON.

---

## 🧠 1. Làm việc với file trong Kotlin

Kotlin tận dụng thư viện chuẩn của Java (`java.io.File`),
vì vậy cú pháp rất thân thiện và ngắn gọn.

---

## 📂 2. Đọc file văn bản (Text File)

Giả sử có file `data.txt` với nội dung:

```
Mai Lê
Công
Hà
```

Ta đọc file bằng:

```kotlin
import java.io.File

fun main() {
    val file = File("data.txt")
    val lines = file.readLines()

    println("📄 Danh sách:")
    lines.forEach { println(it) }
}
```

➡️ Output:

```
📄 Danh sách:
Mai Lê
Công
Hà
```

💡 `readLines()` tự động tách từng dòng thành `List<String>`.

---

### 📋 Đọc toàn bộ nội dung file

```kotlin
val content = File("data.txt").readText()
println(content)
```

---

## ✏️ 3. Ghi dữ liệu vào file (Write File)

```kotlin
import java.io.File

fun main() {
    val file = File("output.txt")
    file.writeText("Xin chào Kotlin!\nĐây là dòng thứ hai.")
}
```

💡 Nếu file chưa tồn tại, Kotlin sẽ **tự tạo mới**.
Mỗi lần `writeText()` sẽ **ghi đè nội dung cũ**.

---

### 📋 Ghi thêm vào cuối file

```kotlin
val file = File("output.txt")
file.appendText("\nThêm dòng mới!")
```

➡️ Không xóa dữ liệu cũ, chỉ nối thêm.

---

## ⚙️ 4. Đọc ghi file trong thư mục riêng

```kotlin
val dir = File("data")
if (!dir.exists()) dir.mkdir()

val file = File(dir, "log.txt")
file.appendText("App started at ${System.currentTimeMillis()}\n")
```

💡 `File(dir, "log.txt")` giúp gộp thư mục + tên file an toàn.

---

## 🧩 5. Kiểm tra file tồn tại & xóa

```kotlin
val file = File("output.txt")

if (file.exists()) {
    println("File tồn tại! Kích thước: ${file.length()} bytes")
} else {
    println("File không tồn tại.")
}

// Xóa file:
file.delete()
```

---

## 📦 6. Làm việc với JSON – Thư viện Gson

> **Gson** là thư viện của Google giúp **chuyển đổi giữa Object ↔ JSON**.

Cài dependency trong Gradle (nếu là project thật):

```gradle
implementation("com.google.code.gson:gson:2.10.1")
```

---

## 🧱 7. Chuyển Object → JSON

```kotlin
import com.google.gson.Gson

data class User(val name: String, val age: Int)

fun main() {
    val user = User("Mai Lê", 25)
    val gson = Gson()
    val json = gson.toJson(user)
    println(json)
}
```

➡️ Output:

```json
{"name":"Mai Lê","age":25}
```

---

## 🔁 8. Chuyển JSON → Object

```kotlin
import com.google.gson.Gson

data class User(val name: String, val age: Int)

fun main() {
    val json = """{"name":"Mai Lê","age":25}"""
    val gson = Gson()
    val user = gson.fromJson(json, User::class.java)
    println(user)
}
```

➡️ Output:
`User(name=Mai Lê, age=25)`

---

## 🧩 9. Danh sách JSON (List)

```kotlin
import com.google.gson.reflect.TypeToken

data class Product(val name: String, val price: Double)

fun main() {
    val products = listOf(
        Product("iPhone", 999.0),
        Product("MacBook", 1599.0)
    )

    val gson = Gson()
    val json = gson.toJson(products)
    println(json)

    // Parse lại
    val type = object : TypeToken<List<Product>>() {}.type
    val parsed = gson.fromJson<List<Product>>(json, type)

    println(parsed)
}
```

---

## ⚡️ 10. Kết hợp File + JSON

### 📋 Ví dụ: Lưu danh sách sản phẩm vào file JSON

```kotlin
import java.io.File
import com.google.gson.Gson
import com.google.gson.reflect.TypeToken

data class Product(val name: String, val price: Double)

fun main() {
    val products = listOf(
        Product("Chuột Logitech", 350.0),
        Product("Bàn phím Keychron", 1200.0)
    )

    val gson = Gson()
    val json = gson.toJson(products)

    val file = File("products.json")
    file.writeText(json)

    println("Đã lưu file products.json")

    // Đọc lại
    val jsonRead = file.readText()
    val type = object : TypeToken<List<Product>>() {}.type
    val restored = gson.fromJson<List<Product>>(jsonRead, type)

    println("📦 Dữ liệu đọc lại:")
    restored.forEach { println("- ${it.name}: ${it.price}") }
}
```

➡️ Output:

```
Đã lưu file products.json
📦 Dữ liệu đọc lại:
- Chuột Logitech: 350.0
- Bàn phím Keychron: 1200.0
```

---

## 🧪 **Thực hành mini**

### 📋 Bài 1

Tạo file `hello.txt` → ghi “Xin chào Kotlin File I/O” → đọc lại và in ra.

### 📋 Bài 2

Tạo `data class Student(name, grade)` → lưu danh sách 3 học sinh ra file `students.json` → đọc lại và in danh sách.

### 📋 Bài 3

Thêm `if (!file.exists())` → tạo file mới nếu chưa có,
sau đó `appendText()` mỗi lần chạy app.

---

## 🏠 **Bài tập về nhà – Bài 15**

| Mức độ            | Bài tập                                              | Gợi ý                        |
| ----------------- | ---------------------------------------------------- | ---------------------------- |
| 🟢 Cơ bản         | Đọc file `.txt` và in từng dòng                      | `File.readLines()`           |
| 🟡 Trung bình     | Ghi danh sách 3 dòng text vào file                   | `writeText()`                |
| 🔵 Nâng cao       | Tạo `Product` list → ghi file JSON → đọc lại         | `Gson`                       |
| 🟣 Thử thách      | Kiểm tra nếu file > 10KB → xóa                       | `File.length()`              |
| 🟠 Siêu thử thách | Ghi log app khởi động (`appendText()`) kèm thời gian | `System.currentTimeMillis()` |

---

## 📚 Tổng kết Bài 15

| Khái niệm                 | Vai trò                           |
| ------------------------- | --------------------------------- |
| `File`                    | Đọc/ghi dữ liệu cục bộ            |
| `writeText`, `appendText` | Ghi file                          |
| `readText`, `readLines`   | Đọc file                          |
| `Gson`                    | Chuyển đổi Object ↔ JSON          |
| `TypeToken`               | Đọc danh sách JSON                |
| Ứng dụng                  | Lưu cấu hình, dữ liệu, cache, log |

---

🎓 **🎯 Kết thúc Giai đoạn 1 – Kotlin Core**

> Em đã học xong toàn bộ nền tảng ngôn ngữ Kotlin:
> biến, hàm, collection, OOP, coroutine, extension, generic, file, JSON.

Từ đây, em có thể **bắt đầu Giai đoạn 2: Kotlin Android – Lập trình ứng dụng với UI, Activity, ViewModel, Retrofit, Room, v.v.**

---

👉 Em có muốn thầy **chuyển sang Giai đoạn 2: Kotlin App Programmatic (Lập trình Android không XML, UI code hoàn toàn)** không?
Nếu em đồng ý, thầy sẽ mở **Tuần 1 – Bài 1: Giới thiệu Android, cấu trúc project và Activity cơ bản (code UI bằng Kotlin thuần)**.
