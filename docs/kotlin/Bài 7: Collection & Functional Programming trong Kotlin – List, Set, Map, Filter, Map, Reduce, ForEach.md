Tốt lắm 👏👏👏 — hôm nay chúng ta bước sang **📘 Bài 7: Collection & Functional Programming trong Kotlin – List, Set, Map, Filter, Map, Reduce, ForEach**

Đây là **bài cực kỳ quan trọng và thú vị**, vì nó dạy em cách làm việc với **tập hợp dữ liệu** — thứ mà em sẽ dùng **liên tục trong thực tế**: danh sách người dùng, sản phẩm, học sinh, API response, v.v.

---

# 📘 BÀI 7: Collection & Functional Programming trong Kotlin

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu ba loại collection chính: **List, Set, Map**
2. Phân biệt `mutable` và `immutable` collections
3. Biết các hàm xử lý dữ liệu kiểu “functional” như `forEach`, `filter`, `map`, `reduce`, `find`
4. Viết được chương trình xử lý danh sách dữ liệu thực tế (ví dụ danh sách sản phẩm, học sinh, nhân viên…)

---

## 🧩 1. Tổng quan về Collection trong Kotlin

Kotlin có 3 nhóm chính:

| Loại     | Đặc điểm                                          | Ví dụ                                 |
| -------- | ------------------------------------------------- | ------------------------------------- |
| **List** | Danh sách có thứ tự, cho phép trùng lặp           | `listOf(1, 2, 3)`                     |
| **Set**  | Tập hợp **không trùng lặp**, không đảm bảo thứ tự | `setOf(1, 2, 2)` → chỉ còn `1, 2`     |
| **Map**  | Dạng key–value (giống dictionary)                 | `mapOf("name" to "Mai", "age" to 25)` |

---

## 🧱 2. List – danh sách có thứ tự

### 📋 Tạo danh sách không thay đổi (immutable)

```kotlin
val fruits = listOf("Táo", "Chuối", "Nho")
println(fruits[0]) // Táo
```

### 📋 Tạo danh sách thay đổi được (mutable)

```kotlin
val numbers = mutableListOf(1, 2, 3)
numbers.add(4)
numbers.remove(2)
println(numbers)
```

➡️ Kết quả: `[1, 3, 4]`

---

### ⚙️ Duyệt List

```kotlin
for (fruit in fruits) {
    println(fruit)
}
```

Hoặc dùng **functional style**:

```kotlin
fruits.forEach { println(it) }
```

---

## 🧠 3. Set – tập hợp không trùng lặp

```kotlin
val items = mutableSetOf("A", "B", "A", "C")
println(items)  // [A, B, C]
items.add("D")
items.remove("A")
println(items)  // [B, C, D]
```

💡 Set tự động loại bỏ các giá trị trùng.

---

## 🗺️ 4. Map – lưu dữ liệu theo cặp key-value

```kotlin
val person = mutableMapOf(
    "name" to "Mai",
    "age" to 25,
    "city" to "Hà Nội"
)
println(person["name"])  // Mai
person["age"] = 26
println(person)
```

💬 Em có thể duyệt map:

```kotlin
for ((key, value) in person) {
    println("$key: $value")
}
```

---

## ⚡️ 5. Functional Style – cách làm việc hiện đại với Collection

Đây là “tinh hoa” của Kotlin.
Thay vì dùng `for`, em có thể **biến đổi, lọc, và tính toán** dữ liệu rất gọn.

---

### 📋 `forEach`

```kotlin
val list = listOf(1, 2, 3)
list.forEach { println("Giá trị: $it") }
```

---

### 📋 `filter` – lọc dữ liệu theo điều kiện

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6)
val even = numbers.filter { it % 2 == 0 }
println(even) // [2, 4, 6]
```

---

### 📋 `map` – biến đổi từng phần tử

```kotlin
val names = listOf("Mai", "Công", "Lê")
val upper = names.map { it.uppercase() }
println(upper) // [MAI, CÔNG, LÊ]
```

---

### 📋 `find` – tìm phần tử đầu tiên thỏa điều kiện

```kotlin
val found = numbers.find { it > 3 }
println(found) // 4
```

---

### 📋 `any` / `all`

```kotlin
println(numbers.any { it > 5 }) // true
println(numbers.all { it < 10 }) // true
```

---

### 📋 `reduce` – gộp dữ liệu thành 1 giá trị

```kotlin
val sum = numbers.reduce { acc, num -> acc + num }
println(sum) // 21
```

💡 `reduce` lặp qua từng phần tử và cộng dồn kết quả (giống fold).

---

### 📋 `sorted` / `sortedDescending`

```kotlin
val scores = listOf(90, 60, 80, 70)
println(scores.sorted())           // [60, 70, 80, 90]
println(scores.sortedDescending()) // [90, 80, 70, 60]
```

---

### 📋 `distinct`

```kotlin
val repeated = listOf(1, 2, 2, 3, 3, 4)
println(repeated.distinct()) // [1, 2, 3, 4]
```

---

## 💻 6. Ứng dụng thực tế – Danh sách sản phẩm

```kotlin
data class Product(val name: String, val price: Double)

val products = listOf(
    Product("iPhone", 999.0),
    Product("MacBook", 1599.0),
    Product("iPad", 699.0),
    Product("AirPods", 199.0)
)

// 1️⃣ In tất cả sản phẩm
products.forEach { println(it.name) }

// 2️⃣ Lọc sản phẩm giá > 700
val expensive = products.filter { it.price > 700 }
println(expensive)

// 3️⃣ Lấy danh sách tên sản phẩm
val names = products.map { it.name }
println(names)

// 4️⃣ Tính tổng giá trị
val total = products.map { it.price }.reduce { acc, p -> acc + p }
println("Tổng giá trị: $$total")

// 5️⃣ Tìm sản phẩm rẻ nhất
val min = products.minByOrNull { it.price }
println("Rẻ nhất: ${min?.name}")
```

---

## 🧩 7. Collection lồng nhau & xử lý nâng cao

```kotlin
val categories = listOf(
    listOf("Táo", "Cam", "Chuối"),
    listOf("Cà rốt", "Khoai tây")
)

val allItems = categories.flatten()
println(allItems) // [Táo, Cam, Chuối, Cà rốt, Khoai tây]
```

💡 `flatten()` giúp “gộp phẳng” danh sách nhiều tầng thành một.

---

## 🏠 **Bài tập về nhà – Bài 7**

| Mức độ            | Bài tập                                                                             | Gợi ý                                |
| ----------------- | ----------------------------------------------------------------------------------- | ------------------------------------ |
| 🟢 Cơ bản         | Tạo danh sách số từ 1–10, in ra các số chẵn                                         | Dùng `filter`                        |
| 🟡 Trung bình     | Tạo danh sách sản phẩm, lọc sản phẩm giá > 500                                      | Dùng `filter` + `forEach`            |
| 🔵 Nâng cao       | Tính tổng giá trị đơn hàng trong list sản phẩm                                      | Dùng `map` + `reduce`                |
| 🟣 Thử thách      | Tìm sản phẩm có giá cao nhất & thấp nhất                                            | Dùng `maxByOrNull`, `minByOrNull`    |
| 🟠 Siêu thử thách | Tạo danh sách học sinh (name, score), lọc học sinh giỏi, sắp xếp theo điểm giảm dần | Dùng `filter` + `sortedByDescending` |

---

## 📚 Tổng kết Bài 7

| Kiến thức                       | Vai trò                                  |
| ------------------------------- | ---------------------------------------- |
| `List`, `Set`, `Map`            | 3 loại collection chính                  |
| `mutableListOf()`               | Danh sách thay đổi được                  |
| `filter`, `map`, `reduce`       | Xử lý dữ liệu theo phong cách functional |
| `forEach`, `find`, `any`, `all` | Lặp và kiểm tra nhanh                    |
| Ứng dụng thực tế                | Duyệt danh sách, tính tổng, lọc dữ liệu  |

---

🎓 **Bài 8 (buổi tới):**

> *Thầy sẽ dạy “Lập trình hướng đối tượng (OOP) trong Kotlin – Class, Constructor, Inheritance, Data Class, Object & Companion Object” — nền tảng để em xây dựng app thực tế với mô hình chuẩn như Android.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 8: Lập trình hướng đối tượng trong Kotlin** không?
Thầy sẽ dạy kỹ cách tạo class, hàm khởi tạo, kế thừa, và dùng `data class` – rất quan trọng để em làm việc với dữ liệu thực trong ứng dụng Android.
