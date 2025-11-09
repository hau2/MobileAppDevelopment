---
sidebar_position: 1
---

Rất tốt 👏 — ta bắt đầu **Bài 1: Giới thiệu về Swift và môi trường Xcode**, đây là nền tảng để bạn hiểu triết lý ngôn ngữ Swift, công cụ lập trình chính, và cấu trúc dự án cơ bản.
Mục tiêu của bài này là **giúp bạn thiết lập môi trường, hiểu triết lý của Swift, và viết dòng code đầu tiên**.

---

# 🧩 BÀI 1: GIỚI THIỆU VỀ SWIFT VÀ MÔI TRƯỜNG XCODE

---

## 🎯 **Mục tiêu bài học**

Sau khi học xong, bạn sẽ:

1. Hiểu Swift là gì và vai trò của nó trong hệ sinh thái Apple.
2. Cài đặt và làm quen với Xcode, Swift Playgrounds.
3. Hiểu cấu trúc một file Swift cơ bản.
4. Viết và chạy chương trình đầu tiên: “Hello, Swift!”.
5. Hiểu về quá trình biên dịch và chạy mã Swift.

---

## 🧠 **1. Tổng quan về Swift**

### 🔹 Swift là gì?

Swift là **ngôn ngữ lập trình do Apple phát triển**, ra mắt năm 2014, nhằm thay thế Objective-C.
Nó được thiết kế với 3 tiêu chí chính:

* **An toàn (Safe)**: giảm thiểu lỗi runtime, tránh null pointer.
* **Hiện đại (Modern)**: cú pháp ngắn gọn, dễ đọc, gần với Python và Kotlin.
* **Hiệu năng cao (Fast)**: biên dịch xuống mã máy tối ưu tương đương C/C++.

### 🔹 Swift được dùng ở đâu?

* iOS, iPadOS (ứng dụng di động)
* macOS (ứng dụng desktop)
* watchOS, tvOS
* Server-side (thông qua SwiftNIO, Vapor)
* Machine Learning (Swift for TensorFlow — bản thử nghiệm)

---

## ⚙️ **2. Cài đặt và làm quen với Xcode**

### 🔸 Cài đặt Xcode

* Mở **App Store** → Tìm “Xcode” → Nhấn **Get** → Cài đặt.
* Dung lượng khoảng **15–20GB**.
* Khi mở lần đầu, Xcode sẽ cài thêm “Command Line Tools” tự động.

### 🔸 Các thành phần chính trong Xcode

| Thành phần                | Mô tả                             |
| ------------------------- | --------------------------------- |
| **Navigator Area (trái)** | Danh sách file, class, asset      |
| **Editor Area (giữa)**    | Khu vực viết code                 |
| **Utilities Area (phải)** | Thuộc tính, thông tin chi tiết    |
| **Toolbar (trên cùng)**   | Nút Run ▶️, chọn thiết bị giả lập |
| **Console (dưới)**        | Hiển thị log, kết quả in ra       |

---

## 💡 **3. Làm việc với Playground**

**Playground** là môi trường thử nghiệm Swift nhanh (giống REPL).
Thích hợp cho người mới vì:

* Chạy từng dòng code mà không cần build toàn bộ dự án.
* Thấy ngay kết quả bên phải (interactive).

**Tạo Playground mới:**

1. Mở Xcode → File → New → Playground.
2. Chọn **iOS > Blank** → Next → Lưu tên “Lesson1.playground”.
3. Bạn sẽ thấy đoạn code mẫu:

   ```swift
   import UIKit

   var greeting = "Hello, playground"
   print(greeting)
   ```

**Chạy thử:**
Nhấn ▶️ để chạy — kết quả hiện trong console:

```
Hello, playground
```

---

## 🧩 **4. Cấu trúc một file Swift**

Một file `.swift` gồm:

1. **Import thư viện** (ví dụ: `import Foundation`, `import UIKit`)
2. **Khai báo biến, hàm, lớp, struct, enum**
3. **Phần logic chính (chạy chương trình)**

Ví dụ:

```swift
import Foundation

let name = "Cường"
let year = 2025

func greeting(_ person: String) -> String {
    return "Xin chào, \(person)! Năm nay là \(year)."
}

print(greeting(name))
```

👉 Kết quả: `Xin chào, Cường! Năm nay là 2025.`

---

## 🧮 **5. Quá trình biên dịch và chạy Swift**

Khi bạn nhấn “Run” trong Xcode:

1. Swift Compiler (LLVM backend) biên dịch mã `.swift` thành mã máy (binary).
2. Liên kết với các thư viện (`UIKit`, `Foundation`).
3. Tạo file thực thi `.app` hoặc `.o` (object file).
4. Trình giả lập iPhone (hoặc máy Mac thật) chạy ứng dụng.

> 🔸 Swift sử dụng mô hình **Ahead-of-Time Compilation (AOT)** – tức là code được biên dịch trước khi chạy, giúp tăng hiệu năng.

---

## 🧰 **6. Các lệnh Swift cơ bản trong Terminal (CLI)**

Bạn có thể chạy Swift mà **không cần Xcode**, chỉ cần Terminal:

```bash
$ swift
Welcome to Swift version 5.10
1> print("Xin chào Swift CLI!")
Xin chào Swift CLI!
```

Hoặc tạo file `.swift`:

```swift
// hello.swift
print("Hello, Swift!")
```

Chạy:

```bash
swift hello.swift
```

---

## 🧪 **7. Bài tập thực hành**

| Mức độ        | Yêu cầu                                                                 | Gợi ý                              |
| ------------- | ----------------------------------------------------------------------- | ---------------------------------- |
| 🟢 Cơ bản     | In ra câu “Xin chào, tôi đang học Swift!”                               | Sử dụng `print()`                  |
| 🟡 Trung bình | Viết chương trình chào người dùng theo tên nhập vào                     | Dùng `readLine()`                  |
| 🔵 Nâng cao   | Viết chương trình chào người dùng theo giờ trong ngày (sáng, trưa, tối) | Dùng `Date`, `Calendar`, `if-else` |

**Ví dụ gợi ý:**

```swift
print("Nhập tên của bạn:")
if let name = readLine() {
    let hour = Calendar.current.component(.hour, from: Date())
    if hour < 12 {
        print("Chào buổi sáng, \(name)!")
    } else if hour < 18 {
        print("Chào buổi chiều, \(name)!")
    } else {
        print("Chào buổi tối, \(name)!")
    }
}
```

---

## 📚 **Tài liệu tham khảo**

* [Swift.org – The Swift Programming Language, Chapter 1](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/guidedtour/)
* Apple Developer: *Your First Swift Program*
* Video gợi ý: “Swift for Absolute Beginners” – Sean Allen / CodeWithChris

---

## 🧭 **Kết thúc bài học**

✅ Bạn đã:

* Cài đặt và chạy được Xcode/Playground
* Hiểu cấu trúc file Swift
* Viết được chương trình Swift đầu tiên

---

Bạn muốn tôi tiếp tục sang **Bài 2 – Làm quen với Playground & Swift REPL (thực hành nâng cao)** luôn không?
Bài 2 sẽ tập trung vào cách sử dụng Playground như công cụ học tương tác, thử nghiệm biến, biểu thức, hàm và debug.
