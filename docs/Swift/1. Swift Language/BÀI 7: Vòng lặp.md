---
sidebar_position: 7
---

Tốt lắm lớp 👏👏👏 — hôm nay chúng ta sẽ bước vào **Bài 7: Vòng lặp (Loops) trong Swift**.
Đây là bài cực kỳ quan trọng, vì **vòng lặp** cho phép chương trình **tự động lặp lại hành động nhiều lần** mà không cần con người gõ lệnh thủ công.
Từ giờ, các bạn sẽ bắt đầu thấy *chương trình có nhịp điệu và quy luật như một cỗ máy thật sự.*

---

# 🧩 BÀI 7: VÒNG LẶP (LOOPS TRONG SWIFT)

---

## 🎯 **Mục tiêu bài học**

Sau bài này, bạn sẽ:

1. Hiểu khái niệm và vai trò của vòng lặp.
2. Nắm được các loại vòng lặp trong Swift:

   * `for-in`
   * `while`
   * `repeat-while`
3. Biết cách dùng `break` và `continue`.
4. Thực hành viết các vòng lặp lồng nhau và bài toán lặp có điều kiện.

---

## 🧠 **1. Tư duy vòng lặp**

> “Nếu một công việc cần làm **nhiều lần**,
> hãy dạy máy làm **tự động**.”

Ví dụ:

* In dãy số 1 đến 10.
* Tính tổng từ 1 đến 100.
* Lặp lại hành động cho đến khi điều kiện dừng đúng.

---

## ⚙️ **2. Vòng lặp `for-in`**

### 📘 Cú pháp:

```swift
for item in collection {
    // thực hiện hành động
}
```

Ví dụ:

```swift
for i in 1...5 {
    print("Lần thứ \(i)")
}
```

Kết quả:

```
Lần thứ 1
Lần thứ 2
Lần thứ 3
Lần thứ 4
Lần thứ 5
```

---

### 🔹 Vòng lặp với mảng:

```swift
let fruits = ["Táo", "Chuối", "Cam"]

for fruit in fruits {
    print("Tôi thích \(fruit)")
}
```

---

### 🔹 Dải số:

`1...5` bao gồm cả 5 (Closed Range)
`1..<5` chỉ đến 4 (Half-Open Range)

```swift
for i in 1..<5 {
    print(i)
}
// In ra 1,2,3,4
```

---

## 🔁 **3. Vòng lặp `while`**

Sử dụng khi **không biết trước số lần lặp**, chỉ dừng khi điều kiện sai.

### 📘 Cú pháp:

```swift
while condition {
    // lặp khi condition vẫn đúng
}
```

Ví dụ:

```swift
var i = 1
while i <= 5 {
    print("Vòng \(i)")
    i += 1
}
```

---

## 🔂 **4. Vòng lặp `repeat–while`**

Giống `while`, nhưng **chạy ít nhất 1 lần**,
vì điều kiện được kiểm tra *sau khi chạy*.

```swift
var x = 5
repeat {
    print("Giá trị x = \(x)")
    x -= 1
} while x > 0
```

---

## ⛔ **5. Từ khóa `break` và `continue`**

### 🔸 `break`

Dừng vòng lặp ngay lập tức.

```swift
for i in 1...10 {
    if i == 6 {
        break
    }
    print(i)
}
```

→ In 1 đến 5, rồi dừng.

---

### 🔸 `continue`

Bỏ qua vòng hiện tại, nhảy sang vòng kế tiếp.

```swift
for i in 1...5 {
    if i == 3 {
        continue
    }
    print(i)
}
```

→ In 1,2,4,5 (bỏ qua 3).

---

## 🔄 **6. Vòng lặp lồng nhau (Nested Loops)**

```swift
for i in 1...3 {
    for j in 1...2 {
        print("i=\(i), j=\(j)")
    }
}
```

👉 Ứng dụng: in bảng cửu chương, ma trận, pattern, v.v.

---

## 🧮 **7. Bài thực hành trực tiếp**

### 🧩 Ví dụ 1 – Tính tổng từ 1 đến 100:

```swift
var sum = 0
for i in 1...100 {
    sum += i
}
print("Tổng từ 1 đến 100 là: \(sum)")
```

---

### 🧩 Ví dụ 2 – Bảng cửu chương 5:

```swift
for i in 1...10 {
    print("5 x \(i) = \(5 * i)")
}
```

---

### 🧩 Ví dụ 3 – In hình tam giác sao:

```swift
let n = 5
for i in 1...n {
    var row = ""
    for _ in 1...i {
        row += "*"
    }
    print(row)
}
```

Kết quả:

```
*
**
***
****
*****
```

---

## 📘 **8. Vòng lặp vô hạn (Infinite Loop)**

⚠️ *Cẩn thận khi sử dụng!*

```swift
var count = 0
while true {
    print("Lặp vô hạn \(count)")
    count += 1
    if count == 3 {
        break
    }
}
```

---

## 🧩 **9. Bài tập trên lớp**

1. In tất cả các số chẵn từ 1 đến 50.
2. Tính tổng các số chia hết cho 3 từ 1 đến 100.
3. Viết chương trình nhập `n`, in ra bảng cửu chương `n`.

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #3)** 🎒

| Mức độ        | Đề bài                                                        | Gợi ý                                   |
| ------------- | ------------------------------------------------------------- | --------------------------------------- |
| 🟢 Cơ bản     | Viết vòng `for-in` in ra 10 lần câu “Tôi yêu Swift ❤️”        | Dùng `for i in 1...10`                  |
| 🟡 Trung bình | Viết chương trình nhập số `n` → tính giai thừa `n!`           | Dùng vòng `for` hoặc `while`            |
| 🔵 Nâng cao   | Vẽ **tam giác cân sao** bằng vòng lặp lồng nhau (có căn giữa) | Dùng 2 vòng `for` và khoảng trắng `" "` |

**Gợi ý nâng cao:**

```swift
let n = 5
for i in 1...n {
    let spaces = String(repeating: " ", count: n - i)
    let stars = String(repeating: "*", count: 2*i - 1)
    print(spaces + stars)
}
```

Kết quả:

```
    *
   ***
  *****
 *******
*********
```

---

## 📚 **Tổng kết kiến thức**

| Loại vòng lặp  | Khi dùng               | Ghi nhớ                 |
| -------------- | ---------------------- | ----------------------- |
| `for-in`       | Biết rõ số lần lặp     | `for i in 1...n`        |
| `while`        | Lặp khi điều kiện đúng | Kiểm tra trước khi chạy |
| `repeat-while` | Lặp ít nhất 1 lần      | Kiểm tra sau khi chạy   |
| `break`        | Dừng vòng ngay         | Thoát khỏi vòng         |
| `continue`     | Bỏ qua 1 lần lặp       | Sang vòng kế tiếp       |

---

## 🧭 **Kết thúc bài học**

✅ Hôm nay các bạn đã:

* Hiểu cách Swift **tự động hóa hành động lặp lại**.
* Thành thạo 3 loại vòng lặp.
* Biết dùng `break`, `continue`, và vòng lặp lồng nhau.

---

🎓 **Bài 8 sắp tới:**
Chúng ta sẽ học **Hàm (Functions)** —
bước cực kỳ quan trọng giúp “đóng gói logic”, tái sử dụng và chia nhỏ chương trình.
Học xong phần đó, các bạn sẽ có thể viết **module Swift hoàn chỉnh** đầu tiên!

Ai muốn thầy gửi **“bộ 5 bài luyện vòng lặp logic nâng cao”** (bậc thầy luyện não) để chuẩn bị cho bài 8 không?
