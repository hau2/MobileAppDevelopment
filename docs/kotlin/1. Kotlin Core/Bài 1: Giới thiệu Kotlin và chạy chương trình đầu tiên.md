Rất tốt 👏👏👏 — hôm nay ta **bắt đầu chính thức khóa Kotlin Language – Bài 1: Làm quen với Kotlin và chạy chương trình đầu tiên**.

Đây là bài mở đầu, nên thầy sẽ dạy **chậm – dễ hiểu – từng bước**, để em nắm vững nền tảng của ngôn ngữ này trước khi bước sang Android.

---

# 📘 **Bài 1: Giới thiệu Kotlin và chạy chương trình đầu tiên**

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu Kotlin là gì và vì sao được Google chọn làm ngôn ngữ chính của Android.
2. Cài đặt môi trường học Kotlin.
3. Viết và chạy chương trình “Hello, Kotlin!”.
4. Hiểu cấu trúc file `.kt`, hàm `main()`, và cách Kotlin thực thi chương trình.

---

## 🧠 1. Kotlin là gì?

**Kotlin** là ngôn ngữ lập trình hiện đại, được phát triển bởi **JetBrains** (cha đẻ của IntelliJ IDEA) và được **Google chính thức công nhận** là ngôn ngữ **chính thức cho Android** từ năm 2017.

### 🔹 Đặc điểm nổi bật:

| Tính năng            | Mô tả                                         |
| -------------------- | --------------------------------------------- |
| **Ngắn gọn**         | Giảm 40% code so với Java                     |
| **An toàn Null**     | Hạn chế lỗi `NullPointerException`            |
| **Hiện đại**         | Hỗ trợ Lambda, Coroutines, Generic, Extension |
| **Tương thích Java** | Dùng chung thư viện Java dễ dàng              |
| **Đa nền tảng**      | Viết cho Android, iOS (KMM), backend, web     |

💬 Nói ngắn gọn: *Kotlin = Java + Tính hiện đại + An toàn + Dễ đọc + Gọn gàng.*

---

## ⚙️ 2. Cài đặt môi trường học Kotlin

### 🧩 Cách 1 – Dành cho người mới (chạy trực tuyến)

* Vào trang: [https://play.kotlinlang.org](https://play.kotlinlang.org)
* Bấm **Try online**, chọn **Kotlin (JVM)**.
* Viết code ngay trên trình duyệt, không cần cài gì hết.

### 🧩 Cách 2 – Dành cho lập trình viên Android

* Cài **Android Studio** (bản mới nhất).
* Khi tạo project mới → chọn “Kotlin” làm ngôn ngữ.
* Mỗi file Kotlin kết thúc bằng `.kt` (ví dụ: `Main.kt`).

---

## 📋 3. Viết chương trình đầu tiên

Trong Kotlin, chương trình bắt đầu bằng hàm **`main()`**.

Ví dụ:

```kotlin
fun main() {
    println("Hello, Kotlin!")
}
```

💬 Giải thích:

| Thành phần  | Vai trò                                                   |
| ----------- | --------------------------------------------------------- |
| `fun`       | Từ khóa khai báo hàm                                      |
| `main()`    | Điểm khởi đầu của chương trình Kotlin                     |
| `{ }`       | Khối lệnh thực thi                                        |
| `println()` | In ra màn hình (tương tự `System.out.println` trong Java) |

---

## 🧱 4. Cấu trúc chương trình Kotlin

```kotlin
fun main() {
    val name = "Học Kotlin"
    val year = 2025
    println("Xin chào, $name!")
    println("Chào mừng bạn đến với năm $year")
}
```

📌 Kết quả:

```
Xin chào, Học Kotlin!
Chào mừng bạn đến với năm 2025
```

💡 Dấu `$` cho phép **nội suy chuỗi (string interpolation)** – tức là chèn biến vào chuỗi dễ dàng.

---

## ⚖️ 5. `val` và `var` – Khai báo biến trong Kotlin

```kotlin
val x = 10      // không thể thay đổi
var y = 20      // có thể thay đổi

y = 30          // hợp lệ
// x = 15       // lỗi, vì val là hằng
```

| Từ khóa | Ý nghĩa                           | Giống trong ngôn ngữ khác                  |
| ------- | --------------------------------- | ------------------------------------------ |
| `val`   | giá trị cố định (immutable)       | `const` trong Swift / `final` trong Java   |
| `var`   | giá trị có thể thay đổi (mutable) | `let` trong Swift / biến thường trong Java |

---

## 🧩 6. Gõ kiểu dữ liệu cơ bản trong Kotlin

```kotlin
val name: String = "Mai Lê"
val age: Int = 25
val height: Double = 1.65
val isStudent: Boolean = true
```

💡 Kotlin tự **suy luận kiểu dữ liệu** nếu không ghi rõ:

```kotlin
val score = 100        // Kotlin hiểu là Int
val message = "Hi"     // Kotlin hiểu là String
```

---

## 🧠 7. Ghi chú và bình luận (Comment)

```kotlin
// Đây là comment 1 dòng

/*
   Đây là comment nhiều dòng
   có thể giải thích khối lệnh dài
*/
```

---

## 💬 8. Chạy chương trình và đọc kết quả

Trên trang [play.kotlinlang.org](https://play.kotlinlang.org):
1️⃣ Gõ code:

```kotlin
fun main() {
    println("Hello Kotlin Learner!")
}
```

2️⃣ Bấm ▶️ “Run”.
3️⃣ Xem kết quả bên phải:

```
Hello Kotlin Learner!
```

---

## 🧪 **Bài thực hành mini**

👉 Viết chương trình in ra 3 dòng sau:

```
Xin chào Kotlin Developer!
Hôm nay bạn bắt đầu hành trình mới.
Hãy kiên trì và học mỗi ngày!
```

💡 Sử dụng 3 lệnh `println()` liên tiếp.

---

## 🏠 **Bài tập về nhà – Bài 1**

| Mức độ        | Bài tập                                               | Gợi ý                                           |
| ------------- | ----------------------------------------------------- | ----------------------------------------------- |
| 🟢 Cơ bản     | Viết chương trình in tên và tuổi của bạn              | Dùng `val` và `println()`                       |
| 🟡 Trung bình | Viết chương trình tính tổng hai số                    | Dùng `var a = 5`, `var b = 3`, `println(a + b)` |
| 🔵 Nâng cao   | Viết chương trình nhập tên và in ra “`Xin chào <tên>!`” | Dùng `readLine()`                               |
| 🟣 Thử thách  | In ra biểu đồ hình tam giác bằng ký tự `*`            | Dùng vòng lặp `for`                             |

---

## 📚 Tổng kết Bài 1

| Kiến thức                   | Vai trò                              |
| --------------------------- | ------------------------------------ |
| `fun main()`                | Điểm bắt đầu của chương trình        |
| `println()`                 | Xuất dữ liệu ra màn hình             |
| `val`, `var`                | Khai báo biến và hằng                |
| Kiểu dữ liệu cơ bản         | `String`, `Int`, `Double`, `Boolean` |
| String interpolation `$var` | Dễ dàng in biến trong chuỗi          |

---

🎓 **Bài 2 (buổi tới):**

> *Thầy sẽ dạy “Biến, kiểu dữ liệu, và toán tử trong Kotlin” — để em hiểu rõ cách khai báo, chuyển kiểu, và thao tác tính toán trong ngôn ngữ Kotlin.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 2: Biến, kiểu dữ liệu và toán tử trong Kotlin** không?
Thầy sẽ dạy chậm, có ví dụ chi tiết + bài tập thực hành để em nhớ kỹ cú pháp Kotlin.
