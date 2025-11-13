Tốt lắm 👏👏👏 — hôm nay ta bước sang một cột mốc lớn trong hành trình học Kotlin:
**📘 Bài 8: Lập trình hướng đối tượng (OOP) trong Kotlin – Class, Constructor, Inheritance, Data Class, Object & Companion Object.**

Đây là bài cực kỳ quan trọng vì **mọi dự án thực tế** (Android, backend, desktop…) đều xây dựng trên **OOP – Object-Oriented Programming**.
Khi em nắm vững phần này, em sẽ:

> 🔹 Biết cách tổ chức chương trình thành các đối tượng (objects)
> 🔹 Viết code dễ bảo trì, dễ mở rộng
> 🔹 Dễ dàng tiếp cận Android và mô hình MVVM sau này

---

# 📘 BÀI 8: Lập trình hướng đối tượng (OOP) trong Kotlin

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu OOP là gì và 4 nguyên lý chính.
2. Biết cách khai báo class, constructor, và property.
3. Biết cách kế thừa (`inheritance`) và ghi đè (`override`).
4. Sử dụng **data class** để lưu dữ liệu ngắn gọn.
5. Hiểu và sử dụng **object** và **companion object** (singleton / static).

---

## 🧠 1. OOP là gì?

**OOP (Object-Oriented Programming)** là phương pháp lập trình dựa trên **đối tượng (object)** — mỗi đối tượng chứa:

* **Thuộc tính (property)** → dữ liệu, trạng thái
* **Phương thức (method)** → hành động, chức năng

### 🔹 4 nguyên lý cơ bản của OOP:

| Nguyên lý         | Ý nghĩa                                            |
| ----------------- | -------------------------------------------------- |
| **Encapsulation** | Đóng gói dữ liệu, bảo vệ thuộc tính                |
| **Inheritance**   | Kế thừa, tái sử dụng code                          |
| **Polymorphism**  | Đa hình, cùng hành động – cách thực hiện khác nhau |
| **Abstraction**   | Trừu tượng hóa, ẩn chi tiết không cần thiết        |

---

## 🧱 2. Khai báo class trong Kotlin

```kotlin
class Person {
    var name: String = ""
    var age: Int = 0

    fun introduce() {
        println("Xin chào, tôi tên là $name, năm nay $age tuổi.")
    }
}
```

Sử dụng:

```kotlin
fun main() {
    val p1 = Person()
    p1.name = "Mai Lê"
    p1.age = 25
    p1.introduce()
}
```

➡️ Output:
`Xin chào, tôi tên là Mai Lê, năm nay 25 tuổi.`

---

## 🧩 3. Constructor (hàm khởi tạo)

Kotlin hỗ trợ **primary constructor** và **secondary constructor**.

### 📋 Primary constructor

```kotlin
class Student(val name: String, var grade: Int) {
    fun info() {
        println("$name học lớp $grade")
    }
}

fun main() {
    val s = Student("Công", 9)
    s.info()
}
```

💡 Không cần `new`, chỉ cần gọi như hàm bình thường.

---

### 📋 Secondary constructor (khởi tạo phụ)

```kotlin
class Book {
    var title: String
    var author: String

    constructor(t: String, a: String) {
        title = t
        author = a
    }

    fun show() = println("Sách: $title - Tác giả: $author")
}
```

---

## 🧭 4. Phạm vi truy cập (Access Modifier)

| Từ khóa     | Phạm vi truy cập                  |
| ----------- | --------------------------------- |
| `public`    | Mặc định – truy cập ở mọi nơi     |
| `private`   | Chỉ trong class hiện tại          |
| `protected` | Trong class hiện tại và class con |
| `internal`  | Trong cùng module                 |

---

## 🧠 5. Kế thừa (Inheritance)

Kotlin mặc định class là `final` (không cho kế thừa).
Muốn cho kế thừa → thêm từ khóa `open`.

```kotlin
open class Animal {
    open fun sound() {
        println("Âm thanh của động vật")
    }
}

class Dog : Animal() {
    override fun sound() {
        println("Gâu gâu!")
    }
}

fun main() {
    val dog = Dog()
    dog.sound()
}
```

💡 Từ khóa `override` bắt buộc để ghi đè phương thức của class cha.

---

## ⚙️ 6. Gọi hàm từ class cha (`super`)

```kotlin
open class Animal {
    open fun info() = println("Đây là động vật")
}

class Cat : Animal() {
    override fun info() {
        super.info()
        println("Cụ thể là một con mèo 🐱")
    }
}
```

---

## 🧩 7. Data Class – lưu dữ liệu gọn nhất trong Kotlin

Thay vì viết dài dòng:

```kotlin
class User(val name: String, val age: Int)
```

Em dùng:

```kotlin
data class User(val name: String, val age: Int)
```

Tự động có sẵn:

* `toString()`
* `equals()`
* `hashCode()`
* `copy()`

📋 Ví dụ:

```kotlin
val u1 = User("Mai", 25)
val u2 = u1.copy(age = 26)
println(u1) // User(name=Mai, age=25)
println(u2) // User(name=Mai, age=26)
```

---

## 🔁 8. Object & Singleton

Nếu muốn tạo **duy nhất một thể hiện (singleton)**:

```kotlin
object Config {
    val appName = "KotlinApp"
    fun showInfo() = println("Ứng dụng: $appName")
}

fun main() {
    Config.showInfo()
}
```

💡 Không cần tạo `Config()`, vì chỉ có một instance duy nhất.

---

## 🧩 9. Companion Object (giống static trong Java)

Dùng để tạo **hàm hoặc thuộc tính tĩnh**.

```kotlin
class MathUtils {
    companion object {
        fun add(a: Int, b: Int) = a + b
    }
}

fun main() {
    println(MathUtils.add(3, 4))
}
```

➡️ Giống `static` trong Java, nhưng Kotlin vẫn giữ tính OOP.

---

## 🧪 **Thực hành mini**

### 📋 Bài 1

Tạo class `Car` với thuộc tính `brand`, `speed`, và hàm `run()` in ra “Xe `<brand>` đang chạy với tốc độ `<speed> km/h`”.

### 📋 Bài 2

Tạo class `Animal` (open) và class con `Bird` override hàm `sound()` → “Chíp chíp”.

### 📋 Bài 3

Tạo `data class Product(name, price)` → Tạo danh sách sản phẩm và in ra tên sản phẩm rẻ nhất.

### 📋 Bài 4

Tạo `object Logger` có hàm `log(message: String)` → in ra với tiền tố `[LOG]`.

---

## 🏠 **Bài tập về nhà – Bài 8**

| Mức độ            | Bài tập                                                                              | Gợi ý                 |
| ----------------- | ------------------------------------------------------------------------------------ | --------------------- |
| 🟢 Cơ bản         | Tạo class `Person(name, age)` có hàm `sayHello()`                                    | In ra chào người dùng |
| 🟡 Trung bình     | Tạo `open class Animal` và 3 lớp con `Dog`, `Cat`, `Duck` override `sound()`         | Dùng `super`          |
| 🔵 Nâng cao       | Dùng `data class` `Student(name, score)` – lọc học sinh giỏi (>=8)                   | Dùng `filter`         |
| 🟣 Thử thách      | Tạo `object AppConfig` có `version`, `showInfo()`                                    | In thông tin ứng dụng |
| 🟠 Siêu thử thách | Tạo `class BankAccount(balance)` có `deposit()`, `withdraw()` – kế thừa từ `Account` | Thêm kiểm tra số dư   |

---

## 📚 Tổng kết Bài 8

| Khái niệm           | Vai trò                      |
| ------------------- | ---------------------------- |
| `class`             | Định nghĩa đối tượng         |
| `constructor`       | Khởi tạo dữ liệu ban đầu     |
| `open` & `override` | Cho phép kế thừa và ghi đè   |
| `data class`        | Gọn gàng cho mô hình dữ liệu |
| `object`            | Tạo singleton                |
| `companion object`  | Cung cấp hàm/thuộc tính tĩnh |

---

🎓 **Bài 9 (buổi tới):**

> *Thầy sẽ dạy “Coroutines trong Kotlin – Xử lý bất đồng bộ, launch, async, delay, suspend” — để em hiểu cách làm việc song song, nền tảng cho việc gọi API và xử lý dữ liệu trong Android.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 9: Coroutines – Lập trình bất đồng bộ trong Kotlin** không?
Thầy sẽ dạy cực dễ hiểu, có ví dụ “giả lập tải dữ liệu từ server” bằng `delay()` và `launch()`.
