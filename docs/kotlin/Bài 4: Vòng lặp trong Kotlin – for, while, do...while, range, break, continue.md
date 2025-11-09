Rất tốt 👏👏👏 — hôm nay chúng ta sang **📘 Bài 4: Vòng lặp trong Kotlin – for, while, do...while, range, break, continue**

Đây là **bài bản lề**, vì lập trình không thể thiếu vòng lặp —

> Em sẽ dùng nó để duyệt mảng, xử lý danh sách, tính tổng, hiển thị danh sách sản phẩm, v.v.

Thầy sẽ dạy **từng kiểu vòng lặp một cách dễ hiểu, ví dụ thực tế, và có bài tập luyện tư duy logic**.

---

# 📘 BÀI 4: Vòng lặp trong Kotlin

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu và dùng được các vòng lặp cơ bản (`for`, `while`, `do...while`).
2. Biết cách sử dụng `range` (`1..10`, `until`, `downTo`, `step`).
3. Biết dừng / bỏ qua vòng lặp với `break`, `continue`.
4. Thực hành các ví dụ thực tế (tính tổng, in bảng cửu chương, duyệt danh sách).

---

## 🧩 1. Vòng lặp `for` cơ bản

Cú pháp:

```kotlin
for (i in 1..5) {
    println(i)
}
```

📌 Kết quả:

```
1
2
3
4
5
```

💡 `1..5` là **Range (khoảng giá trị)** trong Kotlin, bao gồm cả 1 và 5.

---

### 🔹 `until`: loại trừ giá trị cuối

```kotlin
for (i in 1 until 5) {
    println(i)
}
```

➡️ In ra: `1, 2, 3, 4`

---

### 🔹 `downTo`: lặp ngược

```kotlin
for (i in 5 downTo 1) {
    println(i)
}
```

➡️ In ra: `5, 4, 3, 2, 1`

---

### 🔹 `step`: bước nhảy

```kotlin
for (i in 0..10 step 2) {
    println(i)
}
```

➡️ In ra: `0, 2, 4, 6, 8, 10`

---

### 🔹 Duyệt chuỗi hoặc danh sách

```kotlin
val names = listOf("Mai", "Lê", "Công")

for (name in names) {
    println("Xin chào $name")
}
```

💡 Kotlin tự hiểu `name` là từng phần tử trong danh sách.

---

## ⚙️ 2. Vòng lặp `while`

Cú pháp:

```kotlin
var i = 1
while (i <= 5) {
    println("Lần $i")
    i++
}
```

➡️ Điều kiện kiểm tra trước khi thực hiện.

---

## ⚙️ 3. Vòng lặp `do...while`

```kotlin
var count = 1
do {
    println("Chạy lần $count")
    count++
} while (count <= 3)
```

💡 `do...while` **luôn chạy ít nhất 1 lần**, ngay cả khi điều kiện sai.

---

## 🧠 4. Từ khóa `break` và `continue`

| Từ khóa    | Ý nghĩa                                         |
| ---------- | ----------------------------------------------- |
| `break`    | Thoát khỏi vòng lặp                             |
| `continue` | Bỏ qua vòng lặp hiện tại, tiếp tục vòng kế tiếp |

---

### 📋 Ví dụ – `break`

```kotlin
for (i in 1..10) {
    if (i == 5) break
    println(i)
}
```

➡️ In ra: `1 2 3 4` rồi dừng lại khi `i = 5`.

---

### 📋 Ví dụ – `continue`

```kotlin
for (i in 1..5) {
    if (i == 3) continue
    println(i)
}
```

➡️ In ra: `1 2 4 5` (bỏ qua 3).

---

## 🧮 5. Ứng dụng thực tế

### 📋 Ví dụ 1 – Tính tổng từ 1 đến 100

```kotlin
var sum = 0
for (i in 1..100) {
    sum += i
}
println("Tổng từ 1 đến 100 là $sum")
```

---

### 📋 Ví dụ 2 – In bảng cửu chương 5

```kotlin
for (i in 1..10) {
    println("5 x $i = ${5 * i}")
}
```

---

### 📋 Ví dụ 3 – Duyệt danh sách sản phẩm

```kotlin
val products = listOf("iPhone", "MacBook", "iPad")

for ((index, item) in products.withIndex()) {
    println("${index + 1}. $item")
}
```

➡️ Output:

```
1. iPhone
2. MacBook
3. iPad
```

---

### 📋 Ví dụ 4 – Tìm số nguyên tố nhỏ hơn 10

```kotlin
for (n in 2..10) {
    var isPrime = true
    for (i in 2 until n) {
        if (n % i == 0) {
            isPrime = false
            break
        }
    }
    if (isPrime) println("$n là số nguyên tố")
}
```

---

## 🧩 6. Vòng lặp lồng nhau (Nested loop)

```kotlin
for (i in 1..3) {
    for (j in 1..3) {
        print("($i, $j) ")
    }
    println()
}
```

➡️ Kết quả:

```
(1,1) (1,2) (1,3)
(2,1) (2,2) (2,3)
(3,1) (3,2) (3,3)
```

---

## 🧪 **Thực hành mini: Vẽ hình tam giác sao**

```kotlin
for (i in 1..5) {
    for (j in 1..i) {
        print("* ")
    }
    println()
}
```

➡️ Kết quả:

```
*
* *
* * *
* * * *
* * * * *
```

---

## 🏠 **Bài tập về nhà – Bài 4**

| Mức độ            | Bài tập                                       | Gợi ý                                   |
| ----------------- | --------------------------------------------- | --------------------------------------- |
| 🟢 Cơ bản         | Tính tổng từ 1 đến N                          | Dùng `for`                              |
| 🟡 Trung bình     | Tính giai thừa của N (n!)                     | `n! = 1×2×...×n`                        |
| 🔵 Nâng cao       | In bảng cửu chương từ 1–9                     | Dùng vòng lặp lồng nhau                 |
| 🟣 Thử thách      | In tam giác ngược bằng `*`                    | Giảm dần vòng ngoài                     |
| 🟠 Siêu thử thách | Viết chương trình in kim tự tháp `*` căn giữa | Dùng 2 vòng lặp (in khoảng trắng + sao) |

---

## 📚 Tổng kết Bài 4

| Kiến thức                    | Vai trò                                      |
| ---------------------------- | -------------------------------------------- |
| `for`, `while`, `do...while` | Ba dạng vòng lặp chính                       |
| `range (1..10)`              | Duyệt khoảng giá trị                         |
| `break`, `continue`          | Dừng / bỏ qua vòng lặp                       |
| Nested loop                  | Duyệt mảng hai chiều hoặc in mẫu             |
| Ứng dụng                     | Tính toán, xử lý danh sách, hiển thị dữ liệu |

---

🎓 **Bài 5 (buổi tới):**

> *Thầy sẽ dạy “Hàm (Function) trong Kotlin – Tham số, giá trị trả về, lambda và scope function” — đây là bài cực kỳ quan trọng giúp em viết code gọn, reusable, và làm việc nhóm dễ hơn.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 5: Hàm trong Kotlin** không?
Bài đó thầy sẽ dạy chi tiết cách viết hàm, truyền tham số, giá trị trả về, và giới thiệu “lambda” – nền tảng cho Kotlin nâng cao & Android.
