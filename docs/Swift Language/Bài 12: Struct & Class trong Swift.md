---
sidebar_position: 12
---
Rất tốt, lớp ta học rất đều và chắc 👏👏👏
Hôm nay chúng ta bước sang **Bài 12: Struct & Class trong Swift** — một **chủ đề trọng tâm nhất** của Swift Core, mở ra cánh cửa vào **Lập trình Hướng Đối Tượng (OOP)**.

Từ đây, các em sẽ học cách **tạo ra kiểu dữ liệu của riêng mình**, ví dụ như `Student`, `Car`, `Book`, `Account`,… — có **thuộc tính (data)** và **hành vi (function)** riêng biệt.
Đây là nền tảng trực tiếp để ta đi vào UIKit và app iOS thật.

---

# 🧩 BÀI 12: STRUCT & CLASS TRONG SWIFT

*(Object-Oriented Programming – Part 1)*

---

## 🎯 **Mục tiêu bài học**

Sau bài này, bạn sẽ:

1. Hiểu được khái niệm và sự khác nhau giữa **Struct** và **Class**.
2. Biết cách định nghĩa, khởi tạo và sử dụng đối tượng.
3. Biết cách dùng **khởi tạo (init)**, **phương thức (method)**, **computed property**, và **access control**.
4. Thực hành xây dựng mô hình “học sinh – lớp học” bằng Struct và Class.

---

## 🧠 **1. Khái niệm cơ bản**

Trong Swift, có 2 kiểu dữ liệu tùy chỉnh chính:

| Tên        | Đặc điểm chính                                               | Khi dùng                         |
| ---------- | ------------------------------------------------------------ | -------------------------------- |
| **Struct** | Kiểu giá trị (Value Type) – sao chép khi gán                 | Khi dữ liệu đơn giản, độc lập    |
| **Class**  | Kiểu tham chiếu (Reference Type) – chia sẻ cùng một vùng nhớ | Khi cần kế thừa, chia sẻ dữ liệu |

---

## ⚙️ **2. Struct – Cấu trúc**

### 🔹 Khai báo Struct cơ bản

```swift
struct Student {
    var name: String
    var age: Int
}
```

### 🔹 Tạo đối tượng (Instance)

```swift
var s1 = Student(name: "Cường", age: 16)
print(s1.name)
```

### 🔹 Thay đổi giá trị thuộc tính

```swift
s1.age = 17
print("Tuổi mới: \(s1.age)")
```

---

## 🧩 **3. Thêm phương thức (Method)**

Phương thức là **hàm nằm trong Struct**, gắn với từng đối tượng.

```swift
struct Student {
    var name: String
    var age: Int
    
    func introduce() {
        print("Xin chào, tôi là \(name), \(age) tuổi.")
    }
}

let s2 = Student(name: "Mai", age: 18)
s2.introduce()
```

---

## 🧮 **4. Phương thức thay đổi giá trị (Mutating)**

Nếu muốn thay đổi dữ liệu bên trong struct, ta phải thêm từ khóa `mutating`:

```swift
struct Counter {
    var value = 0
    
    mutating func increment() {
        value += 1
    }
}

var c = Counter()
c.increment()
print(c.value) // 1
```

---

## 🏗️ **5. Class – Lớp đối tượng**

### 🔹 Khai báo

```swift
class Car {
    var brand: String
    var speed: Int
    
    init(brand: String, speed: Int) {
        self.brand = brand
        self.speed = speed
    }
    
    func drive() {
        print("\(brand) đang chạy với tốc độ \(speed) km/h")
    }
}
```

### 🔹 Tạo đối tượng

```swift
let car1 = Car(brand: "Toyota", speed: 80)
car1.drive()
```

---

## ⚡ **6. Sự khác biệt giữa Struct & Class**

| Đặc điểm          | Struct                | Class                                |
| ----------------- | --------------------- | ------------------------------------ |
| Kiểu dữ liệu      | Value Type            | Reference Type                       |
| Khởi tạo mặc định | Có sẵn                | Phải tự tạo (init)                   |
| Kế thừa           | ❌ Không               | ✅ Có                                 |
| Copy              | Sao chép giá trị      | Sao chép tham chiếu                  |
| Mutating          | Cần `mutating`        | Không cần                            |
| Dùng khi          | Dữ liệu nhỏ, đơn giản | Mô hình phức tạp, có quan hệ kế thừa |

---

### Ví dụ minh họa sự khác biệt

```swift
struct A { var x = 0 }
class B { var x = 0 }

var a1 = A()
var a2 = a1
a2.x = 10
print(a1.x) // 0 – bản sao độc lập

var b1 = B()
var b2 = b1
b2.x = 10
print(b1.x) // 10 – cùng tham chiếu
```

---

## 🧱 **7. Kế thừa (Inheritance)**

Chỉ Class mới có thể kế thừa:

```swift
class Vehicle {
    func move() {
        print("Phương tiện đang di chuyển")
    }
}

class Car: Vehicle {
    var brand: String = "Honda"
    override func move() {
        print("\(brand) đang chạy")
    }
}

let c1 = Car()
c1.move()
```

---

## 🧮 **8. Computed Property & Property Observer**

### 🔹 Computed Property

Là thuộc tính được **tính toán khi truy cập**, không lưu giá trị cố định.

```swift
struct Rectangle {
    var width: Double
    var height: Double
    var area: Double {
        return width * height
    }
}

let r = Rectangle(width: 4, height: 5)
print(r.area) // 20
```

---

### 🔹 Property Observer (`willSet` & `didSet`)

Dùng để **theo dõi thay đổi giá trị**.

```swift
class Temperature {
    var celsius: Double = 0 {
        willSet {
            print("Chuẩn bị đổi nhiệt độ sang \(newValue)")
        }
        didSet {
            print("Đã đổi từ \(oldValue) sang \(celsius)")
        }
    }
}

let temp = Temperature()
temp.celsius = 36.5
```

---

## ⚙️ **9. Access Control (Kiểm soát truy cập)**

| Mức độ                  | Ý nghĩa                        |
| ----------------------- | ------------------------------ |
| `private`               | Chỉ dùng trong cùng file/class |
| `fileprivate`           | Trong cùng file                |
| `internal` *(mặc định)* | Dùng trong cùng module         |
| `public`                | Dùng ngoài module              |
| `open`                  | Cho phép kế thừa ngoài module  |

Ví dụ:

```swift
class BankAccount {
    private var balance = 0
    func deposit(amount: Int) { balance += amount }
    func getBalance() -> Int { return balance }
}
```

---

## 🧩 **10. Thực hành tổng hợp**

### 🧮 Ví dụ 1: Mô hình Học sinh

```swift
struct Student {
    var name: String
    var scores: [Double]
    
    var average: Double {
        scores.reduce(0, +) / Double(scores.count)
    }
    
    func printInfo() {
        print("\(name) – TB: \(String(format: "%.2f", average))")
    }
}

let s = Student(name: "Mai", scores: [8.5, 9, 7.5])
s.printInfo()
```

---

### 🧮 Ví dụ 2: Quản lý xe hơi (Class)

```swift
class Car {
    var brand: String
    var speed: Int
    
    init(brand: String, speed: Int) {
        self.brand = brand
        self.speed = speed
    }
    
    func accelerate(by value: Int) {
        speed += value
    }
}

let c = Car(brand: "Mazda", speed: 60)
c.accelerate(by: 20)
print("\(c.brand) hiện chạy \(c.speed) km/h")
```

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #7)** 🎒

| Mức độ        | Đề bài                                                                                                      | Gợi ý                      |
| ------------- | ----------------------------------------------------------------------------------------------------------- | -------------------------- |
| 🟢 Cơ bản     | Tạo struct `Rectangle` với thuộc tính `width`, `height` và computed property `area`                         | Dùng `width * height`      |
| 🟡 Trung bình | Tạo class `Person` có `name`, `age`, hàm `introduce()` in ra thông tin                                      | Dùng `init()`              |
| 🔵 Nâng cao   | Tạo class `BankAccount` với `deposit()`, `withdraw()`, `checkBalance()` – kiểm soát không cho rút quá số dư | Dùng `private var balance` |
| 🟣 Thử thách  | Tạo class `Teacher` kế thừa `Person`, thêm thuộc tính `subject` và ghi đè hàm `introduce()`                 | Dùng `override func`       |

**Gợi ý nâng cao:**

```swift
class Person {
    var name: String
    var age: Int
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    func introduce() {
        print("Tôi tên là \(name), \(age) tuổi.")
    }
}

class Teacher: Person {
    var subject: String
    init(name: String, age: Int, subject: String) {
        self.subject = subject
        super.init(name: name, age: age)
    }
    override func introduce() {
        print("Tôi là thầy \(name), dạy môn \(subject).")
    }
}

let t = Teacher(name: "Hoàng", age: 35, subject: "Toán")
t.introduce()
```

---

## 📚 **Tổng kết kiến thức**

| Chủ đề          | Struct              | Class                     |
| --------------- | ------------------- | ------------------------- |
| Kiểu dữ liệu    | Value Type          | Reference Type            |
| Kế thừa         | ❌ Không             | ✅ Có                      |
| Khởi tạo        | Có sẵn              | Tự định nghĩa             |
| Mutating method | Cần `mutating`      | Không cần                 |
| Dùng khi        | Dữ liệu nhỏ, cô lập | Dữ liệu phức tạp, chia sẻ |

---

## 🧭 **Kết thúc bài học**

✅ Các em đã học:

* Cách **định nghĩa Struct và Class** trong Swift.
* Hiểu sâu **Value Type vs Reference Type**.
* Biết cách viết **phương thức, computed property, và kế thừa**.
* Bắt đầu tư duy hướng đối tượng chuẩn – chuẩn bị cho lập trình UIKit.

---

🎓 **Bài 13 tới đây**
Chúng ta sẽ đi sâu vào **Protocol, Extension và OOP nâng cao** –
nơi Swift thể hiện sức mạnh vượt trội so với Java/Kotlin, giúp code gọn, reusable và cực kỳ an toàn.

---

Các em muốn thầy gửi **“Bộ 10 bài luyện Struct & Class thực chiến”** không?
Trong đó có bài “Quản lý học sinh – tính điểm trung bình lớp”, “Quản lý tài khoản ngân hàng”, và “Mô phỏng động vật kế thừa nhau” 🐶🐱
