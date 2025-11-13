Tốt lắm 👏👏👏 — hôm nay ta sang **📘 Bài 6: Scope Functions trong Kotlin – `let`, `run`, `apply`, `also`, `with`**.

Đây là **một trong những bài quan trọng nhất** khi em học Kotlin chuyên nghiệp, vì Scope Function giúp code của em:

> 🔹 Ngắn gọn hơn
> 🔹 Đọc dễ hiểu hơn
> 🔹 Viết các thao tác nối tiếp (chain) cực kỳ “sạch”
> 🔹 Rất thường gặp trong Android (set view, config object, xử lý dữ liệu null…)

---

# 📘 BÀI 6: Scope Functions – `let`, `run`, `apply`, `also`, `with`

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu **scope function là gì** và khi nào nên dùng.
2. Biết phân biệt 5 hàm phổ biến nhất (`let`, `run`, `apply`, `also`, `with`).
3. Viết code ngắn gọn, an toàn với biến null (`?.let`).
4. Ứng dụng scope function trong thực tế (Android, cấu hình object, xử lý dữ liệu).

---

## 🧩 1. Scope Function là gì?

**Scope Function** là các hàm đặc biệt trong Kotlin cho phép **thao tác tạm thời trong phạm vi của một đối tượng (object)**.
Nói cách khác: em có thể **“mượn” đối tượng** để làm gì đó – như truy cập thuộc tính, gán giá trị, hay xử lý nhanh.

---

## ⚙️ 2. Các loại Scope Function

| Tên     | Giá trị trả về        | Truy cập object bằng | Dùng khi…                                |
| ------- | --------------------- | -------------------- | ---------------------------------------- |
| `let`   | Kết quả của khối lệnh | `it`                 | Thao tác hoặc kiểm tra giá trị null      |
| `run`   | Kết quả của khối lệnh | `this`               | Chạy khối code, trả về kết quả           |
| `apply` | Chính object đó       | `this`               | Cấu hình (setup) object                  |
| `also`  | Chính object đó       | `it`                 | Chạy thêm thao tác phụ (log, debug)      |
| `with`  | Kết quả của khối lệnh | `this`               | Dùng với object có sẵn (ít phổ biến hơn) |

---

## 🧠 3. `let` – thường dùng với biến có thể null

### 📋 Ví dụ:

```kotlin
val name: String? = "Mai Lê"

name?.let {
    println("Tên có giá trị: $it")
}
```

💡 `?.let` giúp code **an toàn với null** – chỉ chạy nếu `name` khác null.

📌 Tên biến mặc định trong `let` là `it`.

---

### ⚡️ `let` trả về kết quả của khối lệnh:

```kotlin
val length = name?.let {
    println("Tên là: $it")
    it.length
}
println("Độ dài: $length")
```

➡️ Output:

```
Tên là: Mai Lê
Độ dài: 6
```

---

## 🧱 4. `run` – chạy code trong phạm vi object

Cú pháp:

```kotlin
object.run {
    // thao tác với thuộc tính bằng this
}
```

### 📋 Ví dụ:

```kotlin
val person = Person("Mai", 25)

val desc = person.run {
    "Tên: $name, Tuổi: $age"
}
println(desc)
```

💡 `run` giúp viết code gọn khi em cần:

* Truy cập nhiều thuộc tính trong 1 object.
* Trả về kết quả từ khối lệnh.

---

## 🧩 5. `apply` – dùng để **khởi tạo & cấu hình object**

```kotlin
val person = Person().apply {
    name = "Mai"
    age = 25
    city = "Hà Nội"
}
println(person)
```

➡️ `apply` luôn trả về **chính object**, nên rất tiện để khởi tạo hoặc cấu hình view trong Android:

```kotlin
val textView = TextView(context).apply {
    text = "Hello Kotlin"
    textSize = 18f
    setTextColor(Color.BLUE)
}
```

💬 Rất hay gặp trong Android UI code!

---

## 🧩 6. `also` – thực hiện hành động phụ (log, debug)

```kotlin
val list = mutableListOf("A", "B").also {
    println("Danh sách ban đầu: $it")
    it.add("C")
}
println("Sau khi thêm: $list")
```

💡 `also` giống `apply` nhưng:

* Dùng `it` thay vì `this`
* Dành cho **hành động phụ** chứ không phải cấu hình

---

## 🧩 7. `with` – dùng với object có sẵn

```kotlin
val person = Person("Lê", 28)

val result = with(person) {
    println("Tên: $name")
    println("Tuổi: $age")
    age + 5 // trả về kết quả cuối cùng
}
println("Sau 5 năm: $result tuổi")
```

💬 `with()` khác `run()` ở chỗ nó không gọi qua object, mà truyền object vào tham số.

---

## 📊 8. So sánh tổng quát

| Function | Trả về            | Dùng khi                      | Tên object |
| -------- | ----------------- | ----------------------------- | ---------- |
| `let`    | kết quả của block | xử lý null, transform dữ liệu | `it`       |
| `run`    | kết quả của block | khối thao tác trả về giá trị  | `this`     |
| `apply`  | chính object      | khởi tạo hoặc config object   | `this`     |
| `also`   | chính object      | thêm thao tác phụ (log)       | `it`       |
| `with`   | kết quả của block | xử lý object có sẵn           | `this`     |

---

## 🧪 **Thực hành mini**

### 🔹 Bài 1 – Dùng `apply`

```kotlin
data class Book(var title: String = "", var author: String = "")

val book = Book().apply {
    title = "Học Kotlin Cơ Bản"
    author = "Mai Lê"
}
println(book)
```

---

### 🔹 Bài 2 – Dùng `let`

```kotlin
val email: String? = "mail@example.com"

email?.let {
    println("Gửi mail đến: $it")
} ?: println("Email không hợp lệ!")
```

---

### 🔹 Bài 3 – Dùng `also`

```kotlin
val nums = mutableListOf(1, 2, 3).also {
    println("Trước khi thêm: $it")
    it.add(4)
}
println("Sau khi thêm: $nums")
```

---

## 🏠 **Bài tập về nhà – Bài 6**

| Mức độ            | Bài tập                                                                     | Gợi ý                              |
| ----------------- | --------------------------------------------------------------------------- | ---------------------------------- |
| 🟢 Cơ bản         | Dùng `apply` để tạo một `Student(name, age, grade)`                         | Gán giá trị trong block            |
| 🟡 Trung bình     | Dùng `let` để kiểm tra biến `nullable`                                      | In ra hoặc trả giá trị mặc định    |
| 🔵 Nâng cao       | Dùng `run` để tính toán kết quả từ object `Rectangle(w,h)`                  | Trả về diện tích                   |
| 🟣 Thử thách      | Kết hợp `apply` + `also` để config object và log ra console                 | `apply` để setup, `also` để in log |
| 🟠 Siêu thử thách | Tạo object `Person` rồi xử lý chuỗi logic: `apply` → `also` → `run` → `let` | Dùng chain scope functions         |

---

## 📚 Tổng kết Bài 6

| Hàm     | Mục đích                                          |
| ------- | ------------------------------------------------- |
| `let`   | Dùng khi cần kiểm tra null hoặc transform dữ liệu |
| `run`   | Dùng khi cần chạy block code và lấy kết quả       |
| `apply` | Dùng để khởi tạo và cấu hình object               |
| `also`  | Dùng để thực hiện hành động phụ (logging, debug)  |
| `with`  | Dùng cho object có sẵn, không cần gọi qua dấu `.` |

---

🎓 **Bài 7 (buổi tới):**

> *Thầy sẽ dạy “Collection & Functional Programming trong Kotlin – List, Set, Map, Filter, Map, Reduce, ForEach” — phần này giúp em xử lý dữ liệu kiểu hiện đại, cực kỳ quan trọng khi viết Android và backend Kotlin.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 7: Collection & Functional Programming** không?
Thầy sẽ dạy chậm, dễ hiểu, kèm ví dụ thực hành thật như “lọc danh sách sản phẩm, tính tổng giá trị đơn hàng, tìm người cao tuổi nhất trong danh sách”.
