Rất chính xác 👏👏👏 — thầy xác nhận lại nhé:
em đang theo **lộ trình “Giai đoạn 1 – Kotlin Core”**, và ta đã học đến **Bài 9: Coroutines cơ bản** rồi.
Như vậy **bây giờ ta vào Bài 10** (theo lộ trình gốc), tức là:

> 📘 **Bài 10: Null Safety & Elvis Operator – Tránh crash và xử lý giá trị null an toàn trong Kotlin**

Đây là **một trong những chủ đề “thương hiệu” của Kotlin**,
vì Kotlin sinh ra chính là để giải quyết **lỗi NullPointerException** – lỗi “tử thần” của Java.

---

# 📘 BÀI 10: Null Safety & Elvis Operator trong Kotlin

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu tại sao Kotlin an toàn với `null`.
2. Biết cách khai báo biến nullable (`?`).
3. Biết sử dụng toán tử an toàn: `?.`, `!!`, `?:`, `let`, `run`.
4. Viết chương trình tránh lỗi crash khi dữ liệu rỗng.

---

## ⚠️ 1. Vấn đề: Lỗi NullPointerException (NPE)

Trong Java:

```java
String name = null;
System.out.println(name.length()); // 💥 Crash
```

💀 Lỗi: `NullPointerException` – vì truy cập thuộc tính của `null`.

---

## 💡 2. Kotlin bảo vệ em khỏi lỗi này

Kotlin buộc em **phải khai báo rõ ràng** biến nào có thể `null`.

```kotlin
var name: String = "Mai Lê"
name = null  // ❌ Lỗi biên dịch
```

Nếu em **muốn cho phép null**, phải dùng dấu `?`:

```kotlin
var name: String? = "Mai Lê"
name = null   // ✅ Hợp lệ
```

---

## 🧩 3. Toán tử an toàn `?.`

Dùng `?.` để **chỉ truy cập thuộc tính khi không null**.
Nếu null → trả về `null` luôn, không crash.

```kotlin
val name: String? = null
println(name?.length)  // 👉 null, không lỗi
```

💡 `?.` nghĩa là “nếu khác null thì làm tiếp, còn null thì bỏ qua”.

---

## 🧠 4. Toán tử ép buộc `!!`

Ngược lại, `!!` là “tôi chắc chắn nó không null” – nhưng nếu sai, crash ngay.

```kotlin
val name: String? = null
println(name!!.length) // 💥 NPE (crash)
```

💬 Dùng `!!` **chỉ khi chắc chắn** biến đó không thể null.

---

## 🧮 5. Toán tử Elvis `?:` – cung cấp giá trị mặc định

Nếu giá trị bên trái là `null`, trả về bên phải.

```kotlin
val name: String? = null
val displayName = name ?: "Không xác định"
println(displayName)
```

➡️ Output:
`Không xác định`

💡 Giống như “nếu null → dùng giá trị khác”.

---

### 📋 Ví dụ kết hợp

```kotlin
val input: String? = readLine()
val length = input?.length ?: 0
println("Độ dài: $length")
```

Nếu người dùng không nhập gì → `input` null → `length = 0`
→ **an toàn, không crash.**

---

## 🧩 6. Dùng `let` để làm việc với giá trị không null

`let` chỉ chạy khi biến **không null**.

```kotlin
val email: String? = "mail@example.com"

email?.let {
    println("Gửi mail đến: $it")
}
```

Nếu `email = null`, block `let {}` **không chạy** → code vẫn an toàn.

---

## ⚙️ 7. Kết hợp nhiều toán tử – Ví dụ thực tế

```kotlin
data class User(val name: String?, val email: String?)

fun main() {
    val user = User(null, "mail@example.com")

    val displayName = user.name ?: "Người dùng ẩn danh"
    println("Tên hiển thị: $displayName")

    user.email?.let {
        println("Đã gửi xác nhận đến: $it")
    }
}
```

💡 Output:

```
Tên hiển thị: Người dùng ẩn danh
Đã gửi xác nhận đến: mail@example.com
```

---

## 🧭 8. Hàm `run` với nullables

`run` tương tự `let`, nhưng dùng `this` thay vì `it`.

```kotlin
val message: String? = "Xin chào"

message?.run {
    println("Độ dài chuỗi là $length")
}
```

---

## 🧩 9. Toán tử Safe Cast `as?`

Dùng để **ép kiểu an toàn** – nếu không ép được → trả về `null` chứ không crash.

```kotlin
val data: Any = "Hello Kotlin"
val str: String? = data as? String
val num: Int? = data as? Int   // null, không crash
```

---

## 🧪 **Thực hành mini**

### 📋 Bài 1

Viết chương trình:

```kotlin
val name: String? = null
println(name?.uppercase() ?: "Không có tên")
```

➡️ Output: `Không có tên`

---

### 📋 Bài 2

Nhập chuỗi từ bàn phím, nếu người dùng không nhập → in “Chuỗi trống”.

```kotlin
val input = readLine()
println(input ?: "Chuỗi trống")
```

---

### 📋 Bài 3

Tạo `data class Book(title: String?, author: String?)`,
rồi in ra:

* Nếu `title` null → in “Không có tiêu đề”
* Nếu `author` null → in “Không rõ tác giả”

---

## 🏠 **Bài tập về nhà – Bài 10**

| Mức độ            | Bài tập                                                     | Gợi ý                  |
| ----------------- | ----------------------------------------------------------- | ---------------------- |
| 🟢 Cơ bản         | Tạo biến `name: String?` và in ra độ dài nếu có             | Dùng `?.`              |
| 🟡 Trung bình     | In “`Chào bạn <name>” hoặc “Khách`” nếu null                  | Dùng `?:`              |
| 🔵 Nâng cao       | Tạo `data class User(name, email)` → in email nếu có        | Dùng `?.let`           |
| 🟣 Thử thách      | Tạo `listOf<String?>` → lọc bỏ null và in danh sách còn lại | Dùng `filterNotNull()` |
| 🟠 Siêu thử thách | Viết hàm `safeLength(str: String?): Int` → trả 0 nếu null   | Dùng `?:` hoặc `let`   |

---

## 📚 Tổng kết Bài 10

| Toán tử       | Chức năng                     |
| ------------- | ----------------------------- |
| `?.`          | Truy cập an toàn, tránh crash |
| `!!`          | Ép buộc, có thể gây lỗi       |
| `?:`          | Giá trị mặc định nếu null     |
| `let` / `run` | Thực thi khi không null       |
| `as?`         | Ép kiểu an toàn               |

---

🎓 **Bài 11 (buổi tới):**

> *Thầy sẽ dạy “Extension Function trong Kotlin – Mở rộng class có sẵn, viết code gọn hơn và tái sử dụng” — phần cực kỳ hay để em nâng cấp kỹ năng viết code như chuyên gia.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 11: Extension Function trong Kotlin** không?
Thầy sẽ dạy cách “thêm chức năng” cho class có sẵn, như thêm `isEmailValid()` cho `String`, hoặc `average()` cho `List<Int>`.
