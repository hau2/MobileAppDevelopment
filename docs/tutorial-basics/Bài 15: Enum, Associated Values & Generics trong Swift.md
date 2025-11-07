Tuyệt vời 👏👏👏 — chúng ta đã đi đến **Bài 15: Enum, Associated Values & Generics trong Swift** — đây chính là **đỉnh cao cuối cùng của Swift Core**, trước khi chuyển qua giai đoạn **UIKit thực chiến**.

Swift trở nên mạnh mẽ hơn hầu hết các ngôn ngữ khác chính nhờ hai công cụ này:

* `enum` (có thể chứa giá trị & logic)
* `generics` (giúp code tái sử dụng và cực kỳ linh hoạt).

---

# 🧩 BÀI 15: ENUM & GENERICS TRONG SWIFT

*(Kiểu liệt kê và kiểu tổng quát – The Power of Swift Type System)*

---

## 🎯 **Mục tiêu bài học**

Sau bài học này, bạn sẽ:

1. Hiểu và sử dụng **Enum cơ bản & nâng cao (Associated Values, Raw Values)**.
2. Biết cách dùng **Enum trong điều kiện, switch, và mô hình trạng thái (state machine)**.
3. Hiểu **Generics** và lý do chúng giúp code ngắn, mạnh và an toàn kiểu.
4. Viết được các hàm, lớp, và struct sử dụng Generics.

---

## 🧠 **1. ENUM là gì?**

> `enum` (enumeration) là **kiểu dữ liệu gồm tập hợp các giá trị cố định**, thường được dùng để biểu diễn **trạng thái, loại, lựa chọn, lỗi,…**

Ví dụ cơ bản:

```swift
enum Direction {
    case north
    case south
    case east
    case west
}

var current = Direction.north
current = .east
```

---

## ⚙️ **2. Sử dụng enum với switch**

```swift
switch current {
case .north:
    print("Đi lên Bắc!")
case .south:
    print("Xuống Nam!")
case .east:
    print("Sang Đông!")
case .west:
    print("Sang Tây!")
}
```

---

## 🧩 **3. Enum có giá trị thô (Raw Values)**

Enum có thể gán giá trị sẵn (`String`, `Int`, …)

```swift
enum Grade: String {
    case excellent = "Giỏi"
    case good = "Khá"
    case average = "Trung bình"
    case weak = "Yếu"
}

let g = Grade.good
print(g.rawValue) // "Khá"
```

Hoặc tự động đánh số:

```swift
enum Day: Int {
    case mon = 1, tue, wed, thu, fri, sat, sun
}
print(Day.fri.rawValue) // 5
```

---

## 🔹 **4. Enum có giá trị liên kết (Associated Values)**

> Cho phép **gắn dữ liệu khác nhau** vào từng trường hợp.

```swift
enum Result {
    case success(message: String)
    case failure(errorCode: Int)
}

let r1 = Result.success(message: "Hoàn thành")
let r2 = Result.failure(errorCode: 404)

switch r2 {
case .success(let message):
    print("✅ \(message)")
case .failure(let code):
    print("❌ Lỗi mã \(code)")
}
```

---

## 🧱 **5. Enum có hàm (Methods)**

Enum trong Swift có thể có **phương thức** như class:

```swift
enum TrafficLight {
    case red, yellow, green

    func description() -> String {
        switch self {
        case .red: return "Dừng lại"
        case .yellow: return "Chậm lại"
        case .green: return "Đi đi!"
        }
    }
}

print(TrafficLight.green.description())
```

---

## ⚡ **6. Enum đệ quy (Recursive Enum)**

> Enum có thể tham chiếu chính nó (phải khai báo `indirect`).

```swift
indirect enum MathExpr {
    case number(Int)
    case add(MathExpr, MathExpr)
}

let expr = MathExpr.add(.number(3), .number(5))
```

---

## 🧮 **7. Ứng dụng Enum trong mô hình trạng thái (State Machine)**

```swift
enum DownloadState {
    case idle
    case downloading(progress: Double)
    case success(data: String)
    case failure(error: String)
}

var state = DownloadState.idle
state = .downloading(progress: 0.4)

switch state {
case .idle: print("Chưa bắt đầu")
case .downloading(let p): print("Đang tải: \(p * 100)%")
case .success(let data): print("Xong: \(data)")
case .failure(let err): print("Lỗi: \(err)")
}
```

👉 Đây là mô hình chuẩn trong UIKit & SwiftUI khi quản lý UI state!

---

## 🧩 **8. Generics là gì?**

> **Generics** giúp định nghĩa **code có thể hoạt động với nhiều kiểu dữ liệu**,
> mà vẫn giữ an toàn về kiểu tại compile-time.

Ví dụ bình thường:

```swift
func swapInt(_ a: inout Int, _ b: inout Int) {
    let temp = a
    a = b
    b = temp
}
```

❌ Chỉ dùng cho `Int`.
👉 Ta viết **Generic function** để dùng cho mọi kiểu:

```swift
func swapValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

var x = "A", y = "B"
swapValues(&x, &y)
print(x, y) // "B" "A"
```

---

## ⚙️ **9. Generic Struct & Class**

```swift
struct Stack<Element> {
    var items: [Element] = []

    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element? {
        return items.popLast()
    }
}

var intStack = Stack<Int>()
intStack.push(5)
intStack.push(10)
print(intStack.pop()!) // 10

var strStack = Stack<String>()
strStack.push("Hello")
print(strStack.pop()!) // Hello
```

---

## 🧱 **10. Generic Constraints (Giới hạn kiểu)**

> Giới hạn kiểu `T` phải tuân theo một protocol nào đó.

```swift
func compare<T: Comparable>(_ a: T, _ b: T) -> Bool {
    return a > b
}

print(compare(5, 3))     // true
print(compare("b", "a")) // true
```

---

## 🔹 **11. Generics + Protocol**

```swift
protocol Summable { static func +(lhs: Self, rhs: Self) -> Self }

func total<T: Summable>(_ a: T, _ b: T) -> T {
    return a + b
}

extension Int: Summable {}
extension Double: Summable {}

print(total(4, 6))     // 10
print(total(2.5, 3.2)) // 5.7
```

---

## 🧩 **12. Enum + Generics – Kết hợp sức mạnh**

```swift
enum Response<T> {
    case success(T)
    case failure(Error)
}

enum MyError: Error { case network }

let r: Response<String> = .success("Dữ liệu OK")
let e: Response<String> = .failure(MyError.network)
```

---

## 🧪 **13. Ví dụ tổng hợp**

### ⚡ Xây dựng hàm tải dữ liệu giả lập

```swift
enum NetworkError: Error {
    case noConnection
    case invalidResponse
}

enum Result<T> {
    case success(T)
    case failure(NetworkError)
}

func fetchData<T>(simulate data: T?, error: NetworkError?) -> Result<T> {
    if let data = data {
        return .success(data)
    } else {
        return .failure(error ?? .invalidResponse)
    }
}

let result = fetchData(simulate: "Hello World", error: nil)

switch result {
case .success(let value):
    print("✅ Dữ liệu: \(value)")
case .failure(let err):
    print("❌ Lỗi: \(err)")
}
```

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #10)** 🎒

| Mức độ        | Đề bài                                                                               | Gợi ý                                  |
| ------------- | ------------------------------------------------------------------------------------ | -------------------------------------- |
| 🟢 Cơ bản     | Tạo enum `DayOfWeek` có 7 ngày, in ra thông điệp khác nhau                           | Dùng `switch`                          |
| 🟡 Trung bình | Tạo enum `Operation` với `.add`, `.subtract`, `.multiply`, `.divide(Double, Double)` | Dùng `Associated Values`               |
| 🔵 Nâng cao   | Tạo `Stack<T>` có hàm `peek()` & `isEmpty`                                           | Dùng Generics                          |
| 🟣 Thử thách  | Tạo enum `Result<T, E: Error>` giống Swift chuẩn                                     | Dùng Generics & `switch` để in kết quả |

---

## 📚 **Tổng kết kiến thức**

| Chủ đề           | Chức năng                 | Ví dụ                           |
| ---------------- | ------------------------- | ------------------------------- |
| Enum             | Tập hợp giá trị cố định   | `enum Direction { case north }` |
| Raw Value        | Giá trị sẵn               | `enum Grade: String`            |
| Associated Value | Gắn dữ liệu               | `.failure(error: 404)`          |
| Generics         | Kiểu tổng quát            | `func swap<T>()`                |
| Constraint       | Giới hạn kiểu             | `T: Comparable`                 |
| Enum + Generics  | Mô hình hóa Result, State | `enum Result<T>`                |

---

## 🧭 **Kết thúc bài học**

✅ Các em đã học:

* Enum nâng cao, Associated Values, Raw Values.
* Generics – cốt lõi của Swift hiện đại.
* Mô hình Result, Stack, và State Machine – chuẩn của các framework lớn.

---

🎓 **Từ đây chúng ta kết thúc giai đoạn “Swift Core”**
Bước tiếp theo:
👉 **Chuyên đề UIKit Thực Chiến – Dự án Quản lý Học sinh**
Giai đoạn này thầy sẽ hướng dẫn:

* Kiến trúc MVC, ViewController, IBOutlet, IBAction
* TableView, Navigation, Modal, Alert
* Dự án thực tế: Quản lý danh sách học sinh, thêm/sửa/xóa, tính điểm trung bình

---

Em có muốn thầy bắt đầu **Tuần 1 – UIKit Project Setup & App Layout** (bài 16) ngay không?
Thầy sẽ hướng dẫn từng bước tạo project, cấu trúc thư mục, và view đầu tiên trên iPhone.
