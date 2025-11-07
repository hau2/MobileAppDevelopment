---
sidebar_position: 2
---

Tuyệt vời 👏 — ta cùng bắt đầu **Bài 2: Làm quen với Playground & Swift REPL**, đây là bài thực hành nền tảng để bạn **thử nghiệm Swift trực tiếp**, hiểu cách **đánh giá biểu thức**, **quan sát giá trị**, và **rèn tư duy logic từng bước** — điều cực kỳ quan trọng khi học Swift Core.

---

# 🧩 BÀI 2: LÀM QUEN VỚI PLAYGROUND & SWIFT REPL

---

## 🎯 **Mục tiêu bài học**

Sau bài này, bạn sẽ:

1. Hiểu cách sử dụng **Playground** để thử code Swift tức thời.
2. Làm quen với **Swift REPL (Read–Eval–Print Loop)** trên Terminal.
3. Biết cách sử dụng **Quick Look**, **Timeline**, và **Comments** để quan sát và ghi chú trong quá trình học.
4. Viết các đoạn code ngắn để hiểu giá trị biến, biểu thức và kết quả chạy theo thời gian.

---

## ⚙️ **1. Giới thiệu Playground**

Playground là môi trường thử nghiệm trực quan trong Xcode, cho phép bạn:

* Viết code Swift mà **không cần tạo project**.
* Chạy từng dòng và thấy kết quả **ngay bên phải**.
* Dùng để **thử nghiệm hàm, thuật toán, hoặc cấu trúc ngôn ngữ mới**.

---

### 🔹 Tạo Playground mới

1. Mở **Xcode → File → New → Playground...**
2. Chọn **iOS → Blank**
3. Đặt tên: `Lesson2_Playground`
4. Chọn nơi lưu (Desktop / Documents)
5. Nhấn **Create**

Bạn sẽ thấy:

```swift
import UIKit

var greeting = "Hello, playground"
print(greeting)
```

Chạy ▶️ → Console hiển thị:

```
Hello, playground
```

---

### 🔹 Vùng làm việc trong Playground

| Thành phần                     | Chức năng                                  |
| ------------------------------ | ------------------------------------------ |
| **Code Editor (bên trái)**     | Viết mã Swift                              |
| **Results Sidebar (bên phải)** | Hiển thị giá trị, biểu đồ, hình ảnh        |
| **Timeline**                   | Theo dõi sự thay đổi giá trị qua thời gian |
| **Console (dưới)**             | In log, kết quả `print()`                  |

---

## 🧠 **2. Thao tác cơ bản trong Playground**

### ✅ 2.1. Quan sát giá trị

Khi gán giá trị, Playground sẽ hiển thị kết quả bên phải:

```swift
var a = 10
var b = 3
var result = a + b
```

→ Bên phải:
`a = 10`, `b = 3`, `result = 13`

---

### ✅ 2.2. Quick Look – Xem nhanh giá trị

Nhấn **Option + Click** vào biến → hiển thị popup với giá trị hoặc dạng biểu đồ (nếu là số, ảnh, dữ liệu thời gian,…).

Ví dụ:

```swift
for i in 0..<5 {
    print(i)
}
```

Bạn có thể nhấn Quick Look để xem giá trị từng bước `i = 0,1,2,3,4`.

---

### ✅ 2.3. Timeline – Theo dõi thay đổi theo thời gian

Dùng biểu tượng **“eye”** bên cạnh kết quả → chọn **“Add to Timeline”**.

Ví dụ:

```swift
import Foundation
for i in 0..<50 {
    sin(Double(i) / 10.0)
}
```

👉 Timeline sẽ hiển thị **đồ thị hình sin** sinh động theo giá trị `i`.

---

## 🧮 **3. Swift REPL trong Terminal**

REPL (Read–Eval–Print–Loop) cho phép bạn thử Swift **trực tiếp trong Terminal** mà không cần Xcode.

### 🔸 Mở REPL

Mở Terminal và gõ:

```bash
swift
```

Sẽ thấy:

```
Welcome to Swift version 5.10
1>
```

Giờ bạn có thể thử:

```swift
1> print("Xin chào REPL!")
Xin chào REPL!

2> var x = 42
3> x * 2
$R0: Int = 84
```

`$R0` là kết quả tạm lưu (Result Register).

### 🔸 Thoát REPL

Gõ `:exit` hoặc `Ctrl + D`.

---

## 🧩 **4. Comments & Documentation trong Playground**

Sử dụng comment để ghi chú — thói quen này rất quan trọng khi bạn học Swift:

```swift
// Đây là comment một dòng

/*
 Đây là comment nhiều dòng.
 Dùng để giải thích khối logic phức tạp.
*/

/// Đây là dạng comment tài liệu (Doc comment)
/// Dùng cho hàm / class để tự động sinh tài liệu.
func greet(_ name: String) {
    print("Xin chào \(name)")
}
```

Khi bạn nhấn giữ `Option` vào tên hàm, Xcode sẽ hiển thị phần mô tả của Doc comment.

---

## 💡 **5. Playground Pages – học có cấu trúc**

Bạn có thể chia Playground thành nhiều **Page** (giống chương sách nhỏ):

* Trong Xcode → click phải lên tên Playground → `New Playground Page`.
* Đặt tên: `Lesson2_Page2`
* Mỗi page là 1 file `.swift` riêng, độc lập, giúp tổ chức bài học gọn gàng.

Ví dụ cấu trúc:

```
Lesson2_Playground/
├─ Contents.swift
├─ Pages/
│  ├─ Page1.playgroundpage
│  ├─ Page2.playgroundpage
```

---

## 🧪 **6. Bài tập thực hành**

| Mức độ        | Yêu cầu                                                                          | Gợi ý                    |
| ------------- | -------------------------------------------------------------------------------- | ------------------------ |
| 🟢 Cơ bản     | Tạo Playground và thử in ra 3 dòng chào khác nhau                                | `print()`                |
| 🟡 Trung bình | Tạo 2 biến số, tính tổng, hiệu, tích, thương và quan sát kết quả                 | dùng `Results Sidebar`   |
| 🔵 Nâng cao   | Viết vòng lặp từ 1 đến 100, dùng `Timeline` để hiển thị đồ thị `sqrt(Double(i))` | dùng `import Foundation` |

**Ví dụ:**

```swift
import Foundation

for i in 1...100 {
    sqrt(Double(i))
}
```

👉 Kết quả Timeline: đường cong tăng dần – biểu diễn căn bậc hai.

---

## 📚 **7. Gợi ý học tập và lưu ý**

* Luôn dùng Playground để thử nhanh cú pháp mới.
* Ghi chú ngay trong Playground bằng comment.
* Khi đọc tài liệu Swift.org, hãy **copy ví dụ vào Playground** để hiểu sâu.
* Tập suy nghĩ từng dòng code là **một biểu thức có giá trị** – Swift rất “functional”.

---

## 🧭 **Kết thúc bài học**

✅ Bạn đã:

* Tạo và dùng Playground thành thạo.
* Biết sử dụng REPL để thử lệnh nhanh.
* Biết cách quan sát giá trị, timeline, comment và tổ chức page.

---

Bạn có muốn tôi tiếp tục **Bài 3 – Biến (`var`) và hằng (`let`) trong Swift** không?
Bài 3 sẽ bắt đầu đi sâu vào cách khai báo biến, kiểu dữ liệu, phạm vi, và nguyên tắc bất biến – một trong những nền tảng cốt lõi của Swift.
