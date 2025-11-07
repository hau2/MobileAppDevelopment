Tuyệt vời lắm lớp 👏👏👏
Hôm nay chúng ta sang **Bài 14: Optional & Error Handling trong Swift** — một trong những **bài quan trọng nhất** trong toàn bộ Swift Core.

Nếu coi Swift là một ngôi nhà an toàn, thì `Optional` chính là **hệ thống khóa cửa** và `Error Handling` là **hệ thống cảnh báo và cứu hộ**.
Hai khái niệm này giúp Swift **an toàn, ổn định, và khó crash hơn** bất kỳ ngôn ngữ nào khác (Java, C#, Kotlin, v.v.).

---

# 🧩 BÀI 14: OPTIONAL & ERROR HANDLING TRONG SWIFT

---

## 🎯 **Mục tiêu bài học**

Sau bài này, các em sẽ:

1. Hiểu bản chất của `Optional` và lý do nó tồn tại.
2. Biết cách unwrap an toàn bằng `if let`, `guard let`, `??`.
3. Biết xử lý lỗi bằng `do-catch`, `throw`, `try`.
4. Biết tạo lỗi tùy chỉnh (custom error) và tránh crash trong chương trình thật.

---

## 🧠 **1. Optional là gì?**

> `Optional` là một **kiểu dữ liệu đặc biệt** trong Swift, dùng để **biểu diễn giá trị có thể *có* hoặc *không có***.

Nói ngắn gọn:

* `Optional` = “Có thể có giá trị” hoặc “nil (không có gì cả)”.

### 🔹 Ví dụ

```swift
var name: String? = "Mai"
print(name) // Optional("Mai")

name = nil
print(name) // nil
```

---

## ⚙️ **2. Tại sao cần Optional**

Trong ngôn ngữ khác, việc dùng biến chưa có giá trị thường gây **crash (null pointer)**.

Swift giải quyết bằng cách *bắt buộc* bạn phải **unwrap** trước khi dùng.
Nhờ vậy, chương trình an toàn hơn.

---

## 🧩 **3. Kiểm tra & Unwrap Optional**

### 🔹 3.1 Kiểm tra bằng `if`

```swift
var name: String? = "Mai"
if name != nil {
    print("Tên: \(name!)") // Dấu ! để ép buộc lấy giá trị
}
```

⚠️ Dấu `!` gọi là **force unwrap** – nguy hiểm nếu biến là `nil`.

---

### 🔹 3.2 Unwrap an toàn bằng `if let`

```swift
if let realName = name {
    print("Xin chào \(realName)")
} else {
    print("Không có tên")
}
```

---

### 🔹 3.3 Unwrap bằng `guard let` (thường dùng trong hàm)

```swift
func greet(name: String?) {
    guard let realName = name else {
        print("Không có tên!")
        return
    }
    print("Xin chào \(realName)")
}
```

---

### 🔹 3.4 Cung cấp giá trị mặc định `??`

```swift
let userName = name ?? "Người dùng"
print("Xin chào \(userName)")
```

---

## ⚡ **4. Optional Chaining**

Giúp truy cập nhiều lớp optional mà không bị crash.

```swift
struct Address { var city: String }
struct User { var address: Address? }

let user = User(address: Address(city: "Hà Nội"))
print(user.address?.city) // Optional("Hà Nội")

let user2 = User(address: nil)
print(user2.address?.city) // nil, không crash
```

---

## 🔍 **5. Optional Binding nâng cao**

Có thể unwrap nhiều biến cùng lúc:

```swift
let a: Int? = 3
let b: Int? = 5

if let x = a, let y = b {
    print("Tổng: \(x + y)")
}
```

---

## 🧱 **6. Tạo Optional nhiều tầng (Nested Optionals)**

```swift
var nested: Int?? = 5
print(nested ?? 0)     // Optional(5)
print(nested!! + 1)    // 6
```

> 💡 Không nên lạm dụng Optional lồng nhau — nên unwrap sớm.

---

## ⚙️ **7. Error Handling – Xử lý lỗi**

Swift không để lỗi “nổ tung” như C/Java,
mà cho phép ta **chủ động ném và bắt lỗi.**

### Cấu trúc:

```swift
do {
    try someThrowingFunction()
} catch {
    print("Đã xảy ra lỗi: \(error)")
}
```

---

## 💥 **8. Định nghĩa lỗi (Error Type)**

Tạo enum tuân theo `Error` protocol:

```swift
enum MathError: Error {
    case divideByZero
}

func divide(_ a: Double, by b: Double) throws -> Double {
    if b == 0 {
        throw MathError.divideByZero
    }
    return a / b
}

do {
    let result = try divide(10, by: 0)
    print(result)
} catch MathError.divideByZero {
    print("Không thể chia cho 0!")
}
```

---

## ⚡ **9. Các cách xử lý lỗi khác**

| Cách   | Ý nghĩa                             | Ví dụ                           |
| ------ | ----------------------------------- | ------------------------------- |
| `try`  | Có thể ném lỗi                      | `try riskyFunc()`               |
| `try?` | Trả về Optional nếu có lỗi          | `let result = try? riskyFunc()` |
| `try!` | Ép buộc không lỗi (nếu lỗi → crash) | `let r = try! riskyFunc()` ⚠️   |

---

## 🧩 **10. Tạo nhiều loại lỗi**

```swift
enum LoginError: Error {
    case wrongPassword
    case userNotFound
}

func login(user: String, password: String) throws {
    if user != "admin" { throw LoginError.userNotFound }
    if password != "1234" { throw LoginError.wrongPassword }
    print("Đăng nhập thành công!")
}

do {
    try login(user: "admin", password: "4321")
} catch LoginError.wrongPassword {
    print("Sai mật khẩu!")
} catch {
    print("Lỗi khác: \(error)")
}
```

---

## 🧪 **11. Kết hợp Optional & Error**

```swift
func fetchName(id: Int) -> String? {
    if id == 1 { return "Mai" }
    return nil
}

do {
    if let name = fetchName(id: 1) {
        print("Người dùng: \(name)")
    } else {
        throw LoginError.userNotFound
    }
} catch {
    print("Không tìm thấy người dùng!")
}
```

---

## 🧩 **12. Ví dụ tổng hợp – Máy tính an toàn**

```swift
enum CalcError: Error {
    case divideByZero
    case invalidInput
}

func safeDivide(_ a: String, _ b: String) throws -> Double {
    guard let numA = Double(a), let numB = Double(b) else {
        throw CalcError.invalidInput
    }
    if numB == 0 { throw CalcError.divideByZero }
    return numA / numB
}

do {
    let result = try safeDivide("10", "0")
    print(result)
} catch CalcError.divideByZero {
    print("Lỗi: chia cho 0")
} catch CalcError.invalidInput {
    print("Lỗi: dữ liệu không hợp lệ")
}
```

---

## 🏠 **BÀI TẬP VỀ NHÀ (Homework #9)** 🎒

| Mức độ        | Đề bài                                                                                             | Gợi ý                       |
| ------------- | -------------------------------------------------------------------------------------------------- | --------------------------- |
| 🟢 Cơ bản     | Tạo biến `String?`, in ra nội dung nếu có, nếu không in “Không có dữ liệu”                         | Dùng `??`                   |
| 🟡 Trung bình | Viết hàm `findSquare(of:)` trả về `Int?`, nếu số âm → nil                                          | Dùng `guard let`            |
| 🔵 Nâng cao   | Viết hàm `divide(_:_:)` có thể ném lỗi `divideByZero`, xử lý bằng `do-catch`                       | Dùng `throws`               |
| 🟣 Thử thách  | Viết hàm `login(user:pass:)` như ví dụ, và xử lý 3 tình huống: thành công, sai pass, không tồn tại | Dùng `enum Error` và `try?` |

---

## 📚 **Tổng kết kiến thức**

| Chủ đề             | Ý nghĩa                           | Ví dụ                       |
| ------------------ | --------------------------------- | --------------------------- |
| Optional           | Có thể có hoặc nil                | `var name: String?`         |
| if let / guard let | Unwrap an toàn                    | `if let n = name {}`        |
| ??                 | Giá trị mặc định                  | `name ?? "User"`            |
| throw / do-catch   | Xử lý lỗi                         | `try risky()`               |
| try? / try!        | Biến lỗi thành Optional / ép buộc | `let r = try? func()`       |
| enum Error         | Định nghĩa lỗi tùy chỉnh          | `enum LoginError: Error {}` |

---

## 🧭 **Kết thúc bài học**

✅ Hôm nay các em đã:

* Hiểu rõ **Optional** và tránh crash khi dữ liệu `nil`.
* Biết **xử lý lỗi chuyên nghiệp** bằng `try`, `catch`, `throw`.
* Sẵn sàng cho lập trình iOS thực tế — vì mọi API UIKit/SwiftUI đều dùng `Optional` & `Error Handling`.

---

🎓 **Bài 15 sắp tới:**
Chúng ta sẽ học **Enums nâng cao, Generics và Associated Values** —
đây là bài “cao thủ” của Swift, giúp ta viết code linh hoạt, mạnh mẽ và cực ngắn gọn (sức mạnh của SwiftUI và Combine nằm ở đây).

---

Thầy hỏi lớp một câu:
Ai muốn sau khi kết thúc bài 15, thầy mở **“Chuyên đề Swift UIKIT thực chiến – Xây App Quản lý Học sinh”** để chuyển giai đoạn *Swift Core → UIKit Project* luôn?
(Thầy sẽ chia theo tuần, có project mẫu, storyboard và version code 1.0)
