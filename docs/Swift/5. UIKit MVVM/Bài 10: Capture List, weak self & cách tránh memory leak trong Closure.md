Ok, chúng ta qua **Bài 10 – Capture List, weak self & cách tránh memory leak trong Closure** 👨‍🏫
Đây là bài **CỰC KỲ QUAN TRỌNG** khi dùng closure trong:

* UITableViewCell
* CustomView
* ViewModel → ViewController
* Networking (URLSession)
* Combine (sau này)

Nếu không hiểu bài này → **app sẽ bị memory leak (retain cycle)** và bị Apple reject.

Thầy sẽ dạy:

* Chậm
* Rõ
* Nhiều ví dụ chạy được
* Có sơ đồ minh họa
* Giải thích tại sao dùng nó

---

# 🎯 MỤC TIÊU BÀI 10

Sau bài này em sẽ:

✔ Hiểu retain cycle là gì
✔ Hiểu vì sao dùng `[weak self]`
✔ Biết khi nào dùng `[unowned self]`
✔ Biết closure nào cần weak
✔ Biết tránh leak khi dùng trong Cell, CustomView
✔ Có ví dụ code đầy đủ & comment chi tiết

---

# 🧠 1. **Tại sao phải học Weak Self?**

Vì **closure giữ mạnh (strong reference)** đến object chứa nó.

Nếu không cẩn thận → **retain cycle**:

```
ViewController → giữ strong → ViewModel
ViewModel → giữ strong → closure
closure → giữ strong → self (ViewController)
```

→ Không object nào được giải phóng
→ VC không bao giờ deinit
→ Memory leak

💥 Đây là lỗi CHÍNH của dev mới.

---

# 🧱 2. **Retain Cycle là gì?**

Ví dụ:

```swift
class A {
    var closure: (() -> Void)?
}

let a = A()

a.closure = {
    print(a) // closure giữ a
}
```

👉 Closure giữ **a**
👉 **a** giữ **closure**

→ **Không thằng nào chết → leak**.

---

# 🟦 3. **SOLVE: Sử dụng [weak self]**

Cú pháp:

```swift
{ [weak self] in
    self?.doSomething()
}
```

Ý nghĩa:

* `self` trong closure là **yếu (weak)**
* Không giữ self
* Không tạo vòng lặp strong reference

---

# 📌 4. Ví dụ dễ hiểu nhất – trong ViewController

```swift
func loadData(completion: @escaping () -> Void) {
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
        completion()
    }
}

override func viewDidLoad() {
    super.viewDidLoad()

    loadData { [weak self] in
        self?.showAlert()
    }
}
```

Giải thích:

* Hàm loadData chạy async → closure chạy SAU khi hàm kết thúc
* ViewController phải dùng weak self để tránh closure giữ self quá lâu

---

# 📌 5. Ví dụ cực kỳ quan trọng – trong UITableViewCell

⚠️ Tình huống **thường gây leak**:

```swift
cell.onFavoriteTapped = {
    self.doSomething(indexPath)
}
```

→ NHẦM! ❌

VC sẽ bị giữ luôn → leak.

### FIX

```swift
cell.onFavoriteTapped = { [weak self] in
    guard let self = self else { return }
    self.doSomething(indexPath)
}
```

Bắt buộc dùng `[weak self]`.

---

# 📌 6. Ví dụ trong CustomView

```swift
actionView.onTap = { [weak self] in
    self?.showAlert()
}
```

---

# 🧠 7. Khi nào CẦN dùng [weak self]?

### ✔ 100% phải dùng khi:

* closure *được giữ* bởi object có chứa self
* closure chạy async
* closure nằm trong cell
* closure nằm trong customView
* closure nằm trong ViewModel
* closure dùng trong networking
* Combine `.sink {}` → phải weak self

---

# 🧠 8. Khi nào KHÔNG cần weak self?

### ❌ Không cần weak self khi:

* closure chạy *ngay lập tức*, không lưu lại
* closure không capture self
* closure chỉ return value

Ví dụ OK:

```swift
let arr = [1,2,3]
let newArr = arr.map { $0 * 2 }  // không cần weak
```

Ví dụ OK:

```swift
func runNow(closure: () -> Void) {
    closure() // chạy ngay
}

runNow {
    print(self) // vẫn OK
}
```

---

# 🧱 9. **[weak self] vs [unowned self]**

| Loại    | Nghĩa               | Khi dùng                                   |
| ------- | ------------------- | ------------------------------------------ |
| weak    | là optional (self?) | 99% trường hợp                             |
| unowned | không optional      | khi TA CHẮC CHẮN self sống lâu hơn closure |

👉 Dev mới chỉ nên dùng: **[weak self]**

---

# 🧩 10. Thí dụ minh hoạ VÀNG (nên nhớ suốt đời)

### ❌ Sai – gây leak:

```swift
class ViewModel {
    var onDataLoaded: (() -> Void)?

    func load() {
        onDataLoaded = {
            self.doSomething()  // giữ self
        }
    }
}
```

→ self giữ onDataLoaded, onDataLoaded giữ self → leak.

### ✔ Đúng:

```swift
onDataLoaded = { [weak self] in
    self?.doSomething()
}
```

---

# 📌 11. Ví dụ FULL chạy được – ViewController + Timer (GÂY LEAK)

Copy code dưới và chạy thử:

```swift
import UIKit

class LeakVC: UIViewController {

    var timer: Timer?

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .systemBackground
        title = "Leak Demo"

        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            print("Timer tick")
        }
    }

    deinit {
        print("LeakVC bị hủy") // sẽ không bao giờ chạy!
    }
}
```

👉 Timer giữ closure → closure giữ self
→ VC không bao giờ được hủy → leak!

### FIX bằng weak self:

```swift
timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { [weak self] _ in
    guard let self else { return }
    print("Timer tick")
}
```

---

# 🎯 TÓM TẮT BÀI 10 (RẤT QUAN TRỌNG)

1. Closure giữ reference → tạo retain cycle
2. Phải dùng `[weak self]` trong 90% tình huống
3. Không dùng `[unowned self]` (nguy hiểm)
4. Luôn weak trong:

   * cell → VC
   * view → VC
   * networking → VC
   * ViewModel → VC
   * Combine → VC
   * Timer → VC
5. Điều kiện dùng weak: closure chạy async hoặc lưu lại

Nếu làm đúng → app sạch, không leak, không crash.

---

# 🎁 BÀI TẬP CỦNG CỐ

**Bài 1:**
Tạo ViewController có Timer tick mỗi 1 giây.
→ Dùng `[weak self]` để tránh memory leak.

**Bài 2:**
Trong ProductCell, thêm button "🛒 Add":
→ callback closure → VC → log ra đã chọn sản phẩm.

**Bài 3:**
Trong ViewModel, tạo closure

```swift
var onLoadingChange: ((Bool) -> Void)?
```

→ Dùng weak self trong VC khi gán.

Nếu em muốn thầy kiểm tra bài → gửi code.

---

# 👉 Qua **Bài 11 – Completion Handler nâng cao + Async Closure** chứ?

Chỉ cần nói:

**“Qua bài 11 đi thầy”**
