---
sidebar_position: 11
---
Tốt lắm lớp 👏👏👏
Hôm nay chúng ta bước vào **Bài 11: Cấu trúc dữ liệu (Collections) trong Swift** — đây là phần *trái tim của lập trình ứng dụng* vì **mọi app thực tế đều phải lưu trữ và xử lý tập hợp dữ liệu**: danh sách học sinh, danh sách sản phẩm, tin nhắn, người dùng, v.v.

Sau bài này, các em sẽ có thể **tạo, duyệt, thêm, xóa, và tìm kiếm dữ liệu** một cách chuyên nghiệp — bước nền tảng để sau này làm việc với UIKit, JSON hoặc Database.

---

# 🧩 BÀI 11: CẤU TRÚC DỮ LIỆU TRONG SWIFT

*(Arrays – Dictionaries – Sets – Tuples)*

---

## 🎯 **Mục tiêu bài học**

Sau bài này, bạn sẽ:

1. Hiểu và sử dụng thành thạo 4 cấu trúc dữ liệu cơ bản của Swift.
2. Biết các thao tác thêm, sửa, xóa, duyệt, tìm kiếm.
3. Biết ứng dụng kết hợp với `for-in`, `if`, và `closure`.
4. Thực hành xây dựng các mô hình dữ liệu cơ bản cho app.

---

## 🧠 **1. TỔNG QUAN**

Swift cung cấp 4 loại *Collection types* chính:

| Loại           | Đặc điểm                                 | Khi dùng                              |
| -------------- | ---------------------------------------- | ------------------------------------- |
| **Array**      | Danh sách có thứ tự                      | Khi cần danh sách có vị trí (index)   |
| **Dictionary** | Cặp khóa–giá trị                         | Khi cần tra cứu nhanh theo key        |
| **Set**        | Tập hợp không trùng lặp, không có thứ tự | Khi cần đảm bảo duy nhất              |
| **Tuple**      | Gom nhóm nhiều giá trị khác kiểu         | Khi cần trả về nhiều giá trị tạm thời |

---

## 📋 **2. MẢNG – ARRAY**

### 🔹 Khai báo

```swift
var fruits: [String] = ["Táo", "Cam", "Chuối"]
var numbers = [1, 2, 3, 4, 5]
```

### 🔹 Truy cập phần tử

```swift
print(fruits[0]) // Táo
fruits[1] = "Dưa hấu"
```

### 🔹 Thêm / Xóa phần tử

```swift
fruits.append("Xoài")
fruits.insert("Bưởi", at: 1)
fruits.remove(at: 2)
fruits.removeLast()
```

### 🔹 Duyệt mảng

```swift
for fruit in fruits {
    print("Tôi thích \(fruit)")
}
```

### 🔹 Một số thuộc tính hữu ích

```swift
print(fruits.count)
print(fruits.isEmpty)
print(fruits.first ?? "Không có")
print(fruits.contains("Xoài"))
```

---

## 🧮 **3. TỪ ĐIỂN – DICTIONARY**

### 🔹 Khai báo

```swift
var scores: [String: Int] = [
    "Cường": 8,
    "Mai": 9,
    "Hà": 7
]
```

### 🔹 Truy cập

```swift
print(scores["Mai"] ?? 0)
```

### 🔹 Thêm / Sửa / Xóa

```swift
scores["Tuấn"] = 10       // thêm
scores["Cường"] = 9       // sửa
scores.removeValue(forKey: "Hà")
```

### 🔹 Duyệt Dictionary

```swift
for (name, score) in scores {
    print("\(name): \(score) điểm")
}
```

> 💡 Dictionaries **không có thứ tự cố định**, nên thứ tự in ra có thể khác mỗi lần.

---

## 🧩 **4. TẬP HỢP – SET**

### 🔹 Khai báo

```swift
var languages: Set<String> = ["Swift", "Kotlin", "Python"]
```

### 🔹 Thêm / Xóa

```swift
languages.insert("C++")
languages.remove("Kotlin")
```

### 🔹 Kiểm tra tồn tại

```swift
if languages.contains("Swift") {
    print("Tôi thích Swift ❤️")
}
```

### 🔹 Tập hợp không trùng lặp

```swift
let nums: Set<Int> = [1, 2, 3, 3, 1]
print(nums) // {2, 3, 1}
```

---

### 🔹 Các phép toán tập hợp

```swift
let a: Set = [1, 2, 3, 4]
let b: Set = [3, 4, 5, 6]

print(a.union(b))        // Hợp: [1,2,3,4,5,6]
print(a.intersection(b)) // Giao: [3,4]
print(a.subtracting(b))  // Hiệu: [1,2]
print(a.symmetricDifference(b)) // Phần khác nhau: [1,2,5,6]
```

---

## 🧱 **5. BỘ GIÁ TRỊ – TUPLE**

Tuples giúp **gom nhiều giá trị khác kiểu vào một biến duy nhất**.

```swift
let person = ("Cường", 30, "Hà Nội")
print(person.0) // Cường
print(person.2) // Hà Nội
```

Hoặc đặt tên cho từng phần tử:

```swift
let student = (name: "Mai", age: 25)
print(student.name)
```

### 🔹 Dùng để trả về nhiều giá trị

```swift
func getMinMax(array: [Int]) -> (min: Int, max: Int)? {
    guard let first = array.first else { return nil }
    var minVal = first, maxVal = first
    for num in array {
        if num < minVal { minVal = num }
        if num > maxVal { maxVal = num }
    }
    return (minVal, maxVal)
}

if let result = getMinMax(array: [3,9,2,8,5]) {
    print("Min: \(result.min), Max: \(result.max)")
}
```

---

## ⚙️ **6. Các hàm tiện ích của Collection**

| Hàm                | Mô tả                      | Ví dụ                           |
| ------------------ | -------------------------- | ------------------------------- |
| `map`              | Biến đổi từng phần tử      | `[1,2,3].map{$0*2}` → `[2,4,6]` |
| `filter`           | Lọc phần tử theo điều kiện | `[1,2,3,4].filter{$0%2==0}`     |
| `reduce`           | Gộp thành 1 giá trị        | `[1,2,3].reduce(0,+)` → `6`     |
| `sorted`           | Sắp xếp                    | `[3,1,2].sorted()` → `[1,2,3]`  |
| `contains(where:)` | Kiểm tra điều kiện         | `arr.contains{$0>5}`            |

---

## 🧪 **7. Thực hành trên lớp**

### 🔹 Bài 1

Tạo mảng điểm `[8.5, 9.0, 6.5, 7.0, 9.5]`, tính:

* Trung bình điểm
* Điểm cao nhất và thấp nhất

```swift
let scores = [8.5, 9.0, 6.5, 7.0, 9.5]
let avg = scores.reduce(0, +) / Double(scores.count)
print("TB: \(avg)")
print("Cao nhất: \(scores.max() ?? 0), Thấp nhất: \(scores.min() ?? 0)")
```

---

### 🔹 Bài 2

Tạo dictionary `["Cường":8,"Mai":9,"Hà":7]` và in ra các học sinh có điểm ≥ 8.

```swift
for (name, score) in scores {
    if score >= 8 { print("\(name) đạt \(score)") }
}
```

---

### 🔹 Bài 3

Dùng Set để tìm ngôn ngữ chung giữa hai dev:

```swift
let devA: Set = ["Swift","Kotlin","Python"]
let devB: Set = ["Python","C++","Swift"]
print("Ngôn ngữ chung:", devA.intersection(devB))
```

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #6)** 🎒

| Mức độ        | Đề bài                                                                                            | Gợi ý                     |
| ------------- | ------------------------------------------------------------------------------------------------- | ------------------------- |
| 🟢 Cơ bản     | Tạo mảng 5 tên học sinh và in từng tên ra màn hình                                                | `for name in names {}`    |
| 🟡 Trung bình | Viết hàm nhận mảng điểm, trả về số học sinh đạt ≥ 8                                               | Dùng `filter`             |
| 🔵 Nâng cao   | Viết chương trình nhập tên & điểm học sinh → lưu vào Dictionary → in ra học sinh có điểm cao nhất | Dùng `.max(by:)`          |
| 🟣 Thử thách  | Viết hàm nhận 2 danh sách tên → trả về tập hợp tên chung (không trùng lặp)                        | Dùng `Set.intersection()` |

**Gợi ý nâng cao:**

```swift
let students = ["An":9, "Bình":8, "Cường":9.5]
if let top = students.max(by: { $0.value < $1.value }) {
    print("Học sinh điểm cao nhất: \(top.key) (\(top.value))")
}
```

---

## 📚 **Tổng kết kiến thức**

| Cấu trúc   | Đặc điểm                      | Khi dùng                 |
| ---------- | ----------------------------- | ------------------------ |
| Array      | Danh sách có thứ tự           | Dữ liệu liệt kê          |
| Dictionary | Cặp khóa–giá trị              | Tra cứu nhanh theo key   |
| Set        | Không trùng lặp, không thứ tự | Khi cần dữ liệu duy nhất |
| Tuple      | Gom giá trị khác kiểu         | Trả về nhiều kết quả     |

---

## 🧭 **Kết thúc bài học**

✅ Hôm nay các bạn đã:

* Làm chủ 4 loại cấu trúc dữ liệu cơ bản của Swift.
* Biết áp dụng các thao tác thực tế như thêm, xóa, tìm kiếm, lọc, tính toán.
* Chuẩn bị sẵn sàng để học **Struct và Class** trong bài kế tiếp — bước chuyển từ dữ liệu đơn thuần sang *đối tượng thông minh có hành vi*.

---

🎓 **Bài 12 sắp tới:**
Chúng ta sẽ học **Struct & Class (Lập trình hướng đối tượng – phần 1)** —
học cách tạo “kiểu dữ liệu riêng của bạn”: ví dụ `Student`, `Car`, `Book`, v.v.
Đây là bước cực kỳ quan trọng để sẵn sàng bước sang **UIKit**.

---

Các em có muốn thầy gửi thêm “**Bộ bài tập Collection Master 10 bài**” để ôn lại `map-filter-reduce` không?
Nếu đồng ý, thầy sẽ gửi file thực hành `.playground` mẫu cho cả lớp luôn 📘.
