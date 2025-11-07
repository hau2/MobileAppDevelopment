---
sidebar_position: 5
---

Tuyệt vời lắm lớp 👏👏👏
Chúng ta đã đi được một chặng rất vững rồi — hôm nay sang **Bài 5: Toán tử trong Swift**.
Đây là **bài bản lề** trước khi ta bước vào các khối *điều kiện – vòng lặp – hàm* của ngôn ngữ.
Nắm chắc bài này, các bạn sẽ **tư duy được cách máy tính xử lý biểu thức**, và sau này viết logic chương trình cực kỳ mượt.

---

# 🧩 BÀI 5: TOÁN TỬ TRONG SWIFT

*(Operators in Swift)*

---

## 🎯 **Mục tiêu bài học**

Sau khi học xong, các bạn sẽ:

1. Hiểu và sử dụng thành thạo các nhóm toán tử trong Swift.
2. Biết cách kết hợp biểu thức số học, logic và chuỗi.
3. Tránh được lỗi kiểu dữ liệu khi tính toán.
4. Ứng dụng viết được các phép so sánh, điều kiện cơ bản.

---

## 🧠 **1. Khái niệm toán tử**

**Toán tử (operator)** là ký hiệu hoặc từ khóa giúp Swift thực hiện một thao tác cụ thể trên giá trị hoặc biến.

Ví dụ:

```swift
let a = 5
let b = 3
let c = a + b   // '+' là toán tử cộng
```

Swift chia toán tử thành các **nhóm chính** sau:

| Nhóm                 | Ví dụ                         | Mô tả                              |    |                            |
| -------------------- | ----------------------------- | ---------------------------------- | -- | -------------------------- |
| Số học (Arithmetic)  | `+ - * / %`                   | Cộng, trừ, nhân, chia, chia lấy dư |    |                            |
| Gán (Assignment)     | `=` `+=` `-=`                 | Gán hoặc cộng/trừ nhanh            |    |                            |
| So sánh (Comparison) | `== != > < >= <=`             | So sánh hai giá trị                |    |                            |
| Logic (Logical)      | `&&                           |                                    | !` | Kết hợp điều kiện đúng/sai |
| Chuỗi (String)       | `+`, `+=`                     | Nối hoặc mở rộng chuỗi             |    |                            |
| Ba ngôi (Ternary)    | `condition ? value1 : value2` | Rút gọn câu điều kiện              |    |                            |

---

## ⚙️ **2. Toán tử số học**

```swift
let x = 10
let y = 3

print(x + y)  // 13
print(x - y)  // 7
print(x * y)  // 30
print(x / y)  // 3 (vì là Int)
print(Double(x) / Double(y)) // 3.333...
print(x % y)  // 1 (chia lấy dư)
```

💡 *Lưu ý:*

* Phép chia giữa hai số nguyên sẽ **làm tròn xuống (floor division)**.
* Muốn kết quả có phần thập phân → ép sang `Double`.

---

## 🧮 **3. Toán tử gán và rút gọn**

```swift
var number = 5
number += 3   // tương đương number = number + 3
number -= 1   // tương đương number = number - 1
number *= 2
number /= 4
```

📌 Swift **không hỗ trợ** `++` hay `--` như C/Java nhé!

> Thay vào đó dùng `x += 1` hoặc `x -= 1`.

---

## ⚖️ **4. Toán tử so sánh**

| Toán tử | Ý nghĩa           | Ví dụ    |
| ------- | ----------------- | -------- |
| `==`    | bằng              | `a == b` |
| `!=`    | khác              | `a != b` |
| `>`     | lớn hơn           | `a > b`  |
| `<`     | nhỏ hơn           | `a < b`  |
| `>=`    | lớn hơn hoặc bằng | `a >= b` |
| `<=`    | nhỏ hơn hoặc bằng | `a <= b` |

Ví dụ:

```swift
let score = 85
print(score >= 80)  // true
print(score == 100) // false
```

---

## 🔄 **5. Toán tử logic**

| Toán tử | Ý nghĩa        | Ví dụ           | Kết quả   |       |   |        |      |
| ------- | -------------- | --------------- | --------- | ----- | - | ------ | ---- |
| `&&`    | AND (và)       | `true && false` | false     |       |   |        |      |
| `       |                | `               | OR (hoặc) | `true |   | false` | true |
| `!`     | NOT (phủ định) | `!true`         | false     |       |   |        |      |

Ví dụ:

```swift
let isLoggedIn = true
let hasPermission = false

if isLoggedIn && hasPermission {
    print("Được truy cập hệ thống.")
} else {
    print("Không đủ quyền!")
}
```

---

## 💬 **6. Toán tử chuỗi**

```swift
let firstName = "Mai"
let lastName = "Lê"
let fullName = firstName + " " + lastName
print(fullName) // Mai Lê

var message = "Xin chào"
message += " Swift!"
print(message) // Xin chào Swift!
```

---

## ❓ **7. Toán tử ba ngôi (Ternary Operator)**

Cú pháp:

```swift
condition ? valueIfTrue : valueIfFalse
```

Ví dụ:

```swift
let age = 20
let status = (age >= 18) ? "Đã trưởng thành" : "Vị thành niên"
print(status)
```

---

## 🔁 **8. Toán tử kết hợp biểu thức**

Swift cho phép lồng nhiều toán tử trong một biểu thức,
nhưng phải **tuân theo độ ưu tiên (precedence)** và **tính kết hợp (associativity)**.

Ví dụ:

```swift
let result = 3 + 4 * 2  // 11 (nhân chia trước, cộng trừ sau)
let check = (5 > 3) && (3 < 10) // true
```

💡 Có thể dùng ngoặc `()` để rõ ràng hơn.

---

## 🧩 **9. Một số toán tử nâng cao (xem thêm sau)**

* **Nil-Coalescing Operator:** `a ?? b` – lấy giá trị `a` nếu có, ngược lại lấy `b`.
* **Range Operators:** `1...5` (đóng) hoặc `1..<5` (mở) – dùng cho vòng lặp.
* **Type Check:** `is`, `as`, `as?`, `as!` – dùng cho ép kiểu đối tượng.

(Các toán tử này sẽ học kỹ trong phần **Optional và Collection** sau.)

---

## 🧪 **10. Thực hành trong Playground**

Thử từng đoạn sau trong `Lesson5.playground`:

```swift
let a = 12
let b = 5
print("Tổng: \(a + b)")
print("Hiệu: \(a - b)")
print("Tích: \(a * b)")
print("Thương: \(Double(a) / Double(b))")
print("Dư: \(a % b)")

let height = 1.75
let isTall = height >= 1.70 ? true : false
print("Chiều cao hợp lệ: \(isTall)")
```

---

## 📘 **Bài tập trên lớp**

1. Viết chương trình nhập 2 số nguyên, in ra:

   * Tổng, hiệu, tích, thương, số dư.
   * Xác định số nào lớn hơn.
   * In kết quả logic (`true/false`) cho từng phép so sánh.
2. Viết chương trình nhập điểm thi, in ra xếp loại:

   * `>= 8`: Giỏi
   * `>= 6.5`: Khá
   * `>= 5`: Trung bình
   * `< 5`: Yếu

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #1)** 🎒

| Mức độ        | Đề bài                                                                                                                                                         | Gợi ý                                |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| 🟢 Cơ bản     | Viết chương trình nhập tên và tuổi → in ra “Bạn đã đủ tuổi lái xe” nếu tuổi ≥ 18                                                                               | Dùng `readLine()` và `if`            |
| 🟡 Trung bình | Nhập 3 số, tìm **số lớn nhất** và **nhỏ nhất**                                                                                                                 | Dùng `if` hoặc toán tử ba ngôi `? :` |
| 🔵 Nâng cao   | Viết chương trình tính **thuế thu nhập cá nhân** đơn giản: nếu thu nhập > 15tr thì thuế 20%, nếu 10–15tr thì 10%, còn lại 0%. In ra thuế và thu nhập sau thuế. | Dùng `if-else` và phép nhân          |

👉 *Gợi ý nộp:* ghi toàn bộ code vào file `Lesson5_Homework.playground`
và chạy thử từng bài, in rõ tiêu đề mỗi phần.

---

## 📚 **Tổng kết kiến thức**

| Nhóm    | Toán tử           | Ví dụ               |    |                  |
| ------- | ----------------- | ------------------- | -- | ---------------- |
| Số học  | `+ - * / %`       | `a + b`             |    |                  |
| Gán     | `= += -=`         | `x += 1`            |    |                  |
| So sánh | `== != > < >= <=` | `a >= b`            |    |                  |
| Logic   | `&&               |                     | !` | `a > 0 && b > 0` |
| Chuỗi   | `+`, `+=`         | `"Hello" + "Swift"` |    |                  |
| Ba ngôi | `? :`             | `a > b ? a : b`     |    |                  |

---

✅ **Kết thúc bài học**

* Các bạn đã biết dùng toàn bộ toán tử trong Swift.
* Đã hiểu cách viết biểu thức logic và nối chuỗi.
* Sẵn sàng bước sang bài cực thú vị: **Cấu trúc điều khiển (if – switch)**.

---

🎓 **Bài 6 tuần sau:**
Chúng ta sẽ học **Câu lệnh điều kiện và rẽ nhánh trong Swift**,
biết cách cho chương trình **“suy nghĩ và lựa chọn hành động”**.
Hãy làm hết bài tập về nhà để khi vào bài mới, ta chỉ cần thêm *điều kiện* là ra ứng dụng mini đầu tiên!

---

Các bạn đã hiểu rõ phần này chưa nào? Ai cần thầy giải thích lại phần “toán tử logic” thì comment nhé, ta sẽ làm ví dụ trực tiếp.

