Tuyệt vời 👏👏👏
Hôm nay chúng ta chính thức bước vào **Bài 13: Protocol, Extension & OOP Nâng Cao trong Swift** — đây là phần **đỉnh cao của Swift Core**, và là nền tảng để các em có thể tự tin bước sang **UIKit, SwiftUI, hoặc Combine sau này**.

Nếu bài trước (Struct & Class) dạy ta cách **tạo kiểu dữ liệu có thuộc tính và hành vi**, thì bài hôm nay sẽ dạy ta **chuẩn hoá hành vi – mở rộng khả năng – và tái sử dụng logic** theo đúng triết lý lập trình hướng đối tượng hiện đại.

---

# 🧩 BÀI 13: PROTOCOL, EXTENSION & OOP NÂNG CAO

---

## 🎯 **Mục tiêu bài học**

Sau bài này, các em sẽ:

1. Hiểu **Protocol** là gì và cách dùng để chuẩn hoá hành vi.
2. Biết dùng **Extension** để mở rộng chức năng mà không cần sửa code gốc.
3. Hiểu sâu hơn **OOP nâng cao**: kế thừa, đa hình, override, static/dynamic dispatch.
4. Áp dụng kiến thức vào **bài tập tổng hợp mô phỏng hệ thống quản lý người dùng – giáo viên – học sinh**.

---

## 🧠 **1. Protocol là gì**

> **Protocol** là “bản cam kết” – nó định nghĩa *các hành vi (hàm, thuộc tính)* mà mọi kiểu dữ liệu *phải thực hiện* nếu tuân thủ (conform) theo nó.

Giống như “giao ước”:
👉 *Nếu bạn là `Printable`, bạn PHẢI có hàm `printInfo()`.*

### 🔹 Ví dụ cơ bản

```swift
protocol Printable {
    func printInfo()
}

struct Student: Printable {
    var name: String
    var age: Int
    
    func printInfo() {
        print("Học sinh: \(name), \(age) tuổi.")
    }
}

let s = Student(name: "Mai", age: 15)
s.printInfo()
```

---

## ⚙️ **2. Protocol với Thuộc tính**

Protocol có thể yêu cầu:

* Thuộc tính đọc (`get`)
* Hoặc đọc + ghi (`get set`)

```swift
protocol Identifiable {
    var id: String { get set }
}

struct User: Identifiable {
    var id: String
}
```

---

## 🧩 **3. Protocol với Hàm có tham số & trả về**

```swift
protocol Calculable {
    func sum(a: Int, b: Int) -> Int
}

struct Math: Calculable {
    func sum(a: Int, b: Int) -> Int { a + b }
}

let m = Math()
print(m.sum(a: 3, b: 4))
```

---

## 🧱 **4. Protocol Kế Thừa & Đa Protocol**

Protocol có thể **kế thừa nhau** hoặc **một kiểu conform nhiều protocol**.

```swift
protocol A { func doA() }
protocol B { func doB() }
protocol AB: A, B {}

struct Worker: AB {
    func doA() { print("Làm A") }
    func doB() { print("Làm B") }
}
```

---

## ⚡ **5. Protocol + Class (Delegation pattern)**

Khi gắn protocol với class, ta có thể mô phỏng cơ chế **ủy quyền hành vi** (delegate).

```swift
protocol DownloadDelegate {
    func downloadDidFinish()
}

class Downloader {
    var delegate: DownloadDelegate?
    
    func start() {
        print("Đang tải...")
        delegate?.downloadDidFinish()
    }
}

class Manager: DownloadDelegate {
    func downloadDidFinish() {
        print("Tải xong, xử lý dữ liệu.")
    }
}

let manager = Manager()
let downloader = Downloader()
downloader.delegate = manager
downloader.start()
```

> Đây là **mô hình delegate** cực kỳ phổ biến trong UIKit (ví dụ: `UITableViewDelegate`).

---

## ✨ **6. Extension – Mở rộng kiểu dữ liệu**

> Extension cho phép ta “mở rộng” thêm thuộc tính, hàm, computed property cho **kiểu có sẵn** mà **không cần chỉnh sửa code gốc**.

### 🔹 Ví dụ:

```swift
extension String {
    func reversedString() -> String {
        return String(self.reversed())
    }
}

print("Swift".reversedString()) // "tfiwS"
```

---

### 🔹 Extension cũng có thể thêm initializer hoặc protocol conformance:

```swift
struct Rectangle {
    var width: Double
    var height: Double
}

extension Rectangle {
    var area: Double { width * height }
}

let rect = Rectangle(width: 4, height: 5)
print(rect.area)
```

---

## 🧮 **7. Protocol + Extension = Sức mạnh cực lớn**

Ta có thể **định nghĩa hành vi mặc định** cho protocol qua extension.

```swift
protocol Greetable {
    func greet()
}

extension Greetable {
    func greet() {
        print("Xin chào bạn!")
    }
}

struct Student: Greetable {}
let s = Student()
s.greet() // dùng hành vi mặc định
```

> 👉 Đây là cách Apple định nghĩa sẵn nhiều chức năng của SwiftUI & UIKit mà không cần lặp code.

---

## 🧠 **8. Tính Đa hình (Polymorphism)**

> Polymorphism = Nhiều hình thái.
> Một phương thức có thể thực hiện **khác nhau tùy kiểu đối tượng**.

```swift
class Animal {
    func sound() { print("...") }
}

class Dog: Animal {
    override func sound() { print("Gâu gâu") }
}

class Cat: Animal {
    override func sound() { print("Meo meo") }
}

let pets: [Animal] = [Dog(), Cat()]
for pet in pets { pet.sound() }
```

---

## 🧮 **9. Static vs Dynamic Dispatch**

* Struct (Value Type) → **Static Dispatch** → nhanh, an toàn, compile-time.
* Class (Reference Type) → **Dynamic Dispatch** → linh hoạt, runtime (qua `vtable`).

💡 Swift kết hợp cả hai để cân bằng **hiệu năng + linh hoạt**.

---

## 🧪 **10. Thực hành tổng hợp**

### 🔹 Ví dụ: Mô hình giáo viên – học sinh – trường học

```swift
protocol Person {
    var name: String { get }
    func introduce()
}

extension Person {
    func introduce() {
        print("Xin chào, tôi là \(name).")
    }
}

class Teacher: Person {
    var name: String
    var subject: String
    
    init(name: String, subject: String) {
        self.name = name
        self.subject = subject
    }
    
    func teach() {
        print("\(name) đang dạy môn \(subject).")
    }
}

struct Student: Person {
    var name: String
    var grade: Double
}

let t = Teacher(name: "Hoàng", subject: "Toán")
let s = Student(name: "Mai", grade: 8.7)

t.introduce()
t.teach()
s.introduce()
```

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #8)** 🎒

| Mức độ        | Đề bài                                                                                       | Gợi ý                     |
| ------------- | -------------------------------------------------------------------------------------------- | ------------------------- |
| 🟢 Cơ bản     | Tạo protocol `Describable` với hàm `describe()` và cho `Car`, `Book` thực hiện               | In thông tin mô tả        |
| 🟡 Trung bình | Tạo extension cho `Int` thêm hàm `isEven()` và `square()`                                    | Trả về Bool & bình phương |
| 🔵 Nâng cao   | Tạo protocol `Shape` có `area`, `perimeter`; tạo `Circle`, `Rectangle` conform và in kết quả | Dùng `Double.pi`          |
| 🟣 Thử thách  | Tạo protocol `Playable` + extension mặc định `play()`; tạo `Song` override lại `play()`      | Dùng kế thừa và override  |

**Gợi ý nâng cao:**

```swift
protocol Shape {
    var area: Double { get }
    var perimeter: Double { get }
}

struct Circle: Shape {
    var r: Double
    var area: Double { Double.pi * r * r }
    var perimeter: Double { 2 * Double.pi * r }
}
```

---

## 📚 **Tổng kết kiến thức**

| Chủ đề         | Chức năng         | Ví dụ                 |
| -------------- | ----------------- | --------------------- |
| Protocol       | Giao ước hành vi  | `protocol Printable`  |
| Extension      | Mở rộng chức năng | `extension String {}` |
| Polymorphism   | Đa hình, override | `Dog: Animal`         |
| Delegate       | Ủy quyền hành vi  | `DownloadDelegate`    |
| Access Control | Giới hạn truy cập | `private var balance` |

---

## 🧭 **Kết thúc bài học**

✅ Các em đã học:

* Protocol – cam kết hành vi.
* Extension – mở rộng và tái sử dụng logic.
* OOP nâng cao – đa hình, kế thừa, delegate.

---

🎓 **Bài 14 tới đây**
Chúng ta sẽ học **Error Handling & Optional trong Swift** —
học cách “phòng lỗi” cho chương trình, tránh crash, và xử lý lỗi một cách **chuyên nghiệp – chuẩn Apple Developer**.

---

Ai trong lớp muốn thầy gửi **“Bộ project nhỏ – Quản lý học sinh với Protocol & Extension”** để luyện thực tế thì giơ tay nhé 🙋 — thầy sẽ gửi kèm file `.playground` để làm theo từng bước.
