Ok, qua **Bài 7 – Closure Pattern (phần 1: cơ bản)** nha trò 👨‍🏫
Từ bài này trở đi, chúng ta bắt đầu bước vào **nền tảng quan trọng nhất của MVVM UIKit**: **Closure Pattern**.

Thầy sẽ dạy **cực kỳ chậm – dễ hiểu – ví dụ nhỏ → ví dụ lớn** và **code luôn có comment chi tiết**.

---

# 🎯 **MỤC TIÊU BÀI 7 – Closure cơ bản**

Sau bài này em sẽ hiểu:

✔ Closure là gì
✔ Closure có mấy dạng
✔ Closure nhận tham số như thế nào
✔ Closure trả về giá trị như thế nào
✔ Closure lưu trữ trong biến như thế nào
✔ Cách gọi closure
✔ Callback bằng closure
✔ Tạo hàm dùng closure như completion handler
✔ Code đầy đủ copy/paste chạy được

---

# 🧱 1. Closure là gì?

**Closure = một khối code có thể được lưu trữ vào biến và gọi lại sau.**

Giống như **function inline**, nhưng:

* Có thể gán vào biến
* Có thể truyền như 1 tham số
* Có thể gọi lại khi cần

Nếu em từng làm JavaScript → **closure Swift giống hệt callback JS**.

Ví dụ JS:

```js
button.onClick(() => { console.log("clicked") })
```

Swift cũng vậy:

```swift
onTap = { print("clicked") }
```

---

# 🧱 2. Cú pháp cơ bản

## Dạng đầy đủ:

```swift
{ (param1: Type, param2: Type) -> ReturnType in
    // code
}
```

## Dạng rút gọn nhất:

```swift
{ print("Hello") }
```

---

# 🧱 3. Ví dụ 1 – Closure không tham số, không trả về (simple callback)

📌 Thầy viết đoạn code **copy/paste chạy được** trong Playground hoặc ViewController:

```swift
import UIKit

class ViewController: UIViewController {

    /// Khai báo closure (callback)
    /// () -> Void nghĩa là:
    /// - không nhận tham số
    /// - không trả về gì
    var onDone: (() -> Void)?

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        // GÁN closure
        onDone = {
            print("Closure được gọi!")
        }

        // GỌI closure
        onDone?()
    }
}
```

### Kết quả log:

```
Closure được gọi!
```

Rất đơn giản đúng không? 😄

---

# 🧱 4. Ví dụ 2 – Closure có **tham số**

Closure có thể nhận tham số như hàm.

```swift
var onMessage: ((String) -> Void)?

override func viewDidLoad() {
    super.viewDidLoad()

    // gán closure
    onMessage = { text in
        print("Message:", text)
    }

    // gọi closure
    onMessage?("Xin chào closure!")
}
```

Kết quả:

```
Message: Xin chào closure!
```

---

# 🧱 5. Ví dụ 3 – Closure có **nhiều tham số**

```swift
var onSum: ((Int, Int) -> Void)?

override func viewDidLoad() {
    super.viewDidLoad()

    onSum = { a, b in
        print("Tổng =", a + b)
    }

    onSum?(10, 20) // Kết quả: Tổng = 30
}
```

---

# 🧱 6. Ví dụ 4 – Closure **trả về giá trị**

```swift
var multiply: ((Int, Int) -> Int)?

override func viewDidLoad() {
    super.viewDidLoad()

    multiply = { a, b in
        return a * b
    }

    let result = multiply?(5, 4)
    print("Kết quả =", result ?? 0)
}
```

Kết quả:

```
Kết quả = 20
```

---

# 🧱 7. Ví dụ 5 – Closure làm **completion handler** (rất quan trọng)

Closure trong hàm là nền tảng cho networking & MVVM sau này.

```swift
func downloadData(completion: @escaping (String) -> Void) {
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
        completion("Dữ liệu đã tải xong!")
    }
}

override func viewDidLoad() {
    super.viewDidLoad()

    downloadData { result in
        print(result)
    }
}
```

Giải thích:

* `completion` là closure
* `@escaping` = closure được chạy SAU KHI hàm kết thúc (giống async)

Kết quả:

```
Dữ liệu đã tải xong!
```

---

# 🧱 8. Ví dụ 6 – Closure lưu vào property (chuẩn MVVM)

Đây là nền tảng MVVM UIKit:

```swift
class ViewModel {

    var onDataLoaded: ((String) -> Void)?

    func loadData() {
        onDataLoaded?("Dữ liệu từ ViewModel")
    }
}
```

ViewController:

```swift
let vm = ViewModel()

override func viewDidLoad() {
    super.viewDidLoad()

    vm.onDataLoaded = { text in
        print("VC nhận:", text)
    }

    vm.loadData()
}
```

Kết quả:

```
VC nhận: Dữ liệu từ ViewModel
```

→ Đây chính là cách MVVM truyền data từ **ViewModel → View** bằng **Closure Pattern**.

---

# 🎁 Tổng kết Bài 7

| Khái niệm               | Em đã học |
| ----------------------- | --------- |
| Closure là gì           | ✔         |
| Gán closure cho biến    | ✔         |
| Closure không tham số   | ✔         |
| Closure có tham số      | ✔         |
| Closure trả về giá trị  | ✔         |
| Completion handler      | ✔         |
| Closure trong ViewModel | ✔         |

Em đã nắm nền tảng để qua:

👉 **Bài 8 – Closure Pattern trong CustomView (Callback UI)**
→ Đây là bài cực quan trọng để tạo bottom sheet, popup, cell event…

Chỉ cần nói:
**“Qua bài 8 đi thầy”**
