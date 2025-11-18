Ok, qua **Bài 14 — PassthroughSubject & CurrentValueSubject** 👨‍🏫
Đây là bài **rất quan trọng** trong Combine vì hai “subject” này chính là nền tảng của:

* Button tap event
* Event bus
* MVVM Input / Output
* Repository callback
* Pagination
* Real-time UI updates

Thầy sẽ dạy thật **chậm – chi tiết – có ví dụ đầy đủ – giải thích vì sao dùng cái nào**.

---

# 🌟 **BÀI 14 – PassthroughSubject & CurrentValueSubject**

## Nội dung:

1. Subject là gì
2. PassthroughSubject
3. CurrentValueSubject
4. Khi nào dùng cái nào
5. Ví dụ cụ thể (button tap) với PassthroughSubject
6. Ví dụ lưu state với CurrentValueSubject
7. Tích hợp vào MVVM

---

# 🧠 **1. Subject là gì?**

**Subject = vừa là Publisher vừa là “bộ phát sự kiện”**.

Khác với `@Published`, Subject cho phép:

* **thủ công** gửi giá trị đi → `.send(value)`
* tạo ra dòng sự kiện **chủ động**

Subject = "Event bus" trong Combine.

---

# 🧱 **2. PassthroughSubject**

> Chủ động phát sự kiện, **không lưu giá trị**, subscriber phải lắng nghe *từ trước* mới nhận được.

### Khai báo:

```swift
let tapSubject = PassthroughSubject<Void, Never>()
```

* `Void` = loại event (ở đây là sự kiện tap)
* `Never` = không phát lỗi (phù hợp cho UI event)

### Phát sự kiện:

```swift
tapSubject.send(())
```

### Đặc điểm chính:

| Thuộc tính                          | Giá trị                                   |
| ----------------------------------- | ----------------------------------------- |
| Lưu giá trị gần nhất?               | ❌ Không                                   |
| Gửi giá trị cho subscriber sau này? | ❌ Không                                   |
| Dùng khi?                           | UI event, button tap, navigation, command |

---

# 🧱 **3. CurrentValueSubject**

> Giống `BehaviorSubject` của RxSwift: **giữ giá trị hiện tại**, subscriber mới vẫn nhận được ngay.

### Khai báo:

```swift
let count = CurrentValueSubject<Int, Never>(0)
```

### Gửi giá trị mới:

```swift
count.send(1)
```

### Lấy giá trị hiện tại:

```swift
print(count.value)
```

### Đặc điểm:

| Thuộc tính                            | Giá trị                            |
| ------------------------------------- | ---------------------------------- |
| Lưu giá trị gần nhất?                 | ✔ Có                               |
| Subscriber mới nhận giá trị hiện tại? | ✔ Có                               |
| Dùng khi?                             | State, form input, dữ liệu persist |

---

# 🧩 **4. Khi nào dùng cái nào?**

## 🟦 Dùng **PassthroughSubject** khi:

* Sự kiện không có giá trị lưu trữ
* Không cần nhớ giá trị cũ
* Chỉ cần “đẩy sự kiện”
* Ví dụ:

  * Button tap
  * Refresh event
  * Navigation event
  * Submit form event
  * Pagination next-page event

**→ Giống như "EventBus" thuần**.

---

## 🟧 Dùng **CurrentValueSubject** khi:

* Cần giữ “state hiện tại”
* View hoặc subscriber mới phải nhận được state ban đầu
* State thay đổi theo thời gian
* Ví dụ:

  * count
  * selected index
  * danh sách sản phẩm (cache)
  * login state
  * app theme
  * network state

**→ Giống như "State Store"**.

---

# 🟦 **5. Ví dụ 1 — BUTTON TAP EVENT (PassthroughSubject)**

### 🎯 Mục đích:

Khi user bấm nút → ViewModel nhận event → xử lý → VC update UI.

---

## 🔹 ViewModel – dùng PassthroughSubject

```swift
import Combine

class ButtonViewModel {

    // Sự kiện tap
    let buttonTapped = PassthroughSubject<Void, Never>()

    // Output text
    @Published var message = "Ấn nút đi!"

    private var cancellables = Set<AnyCancellable>()

    init() {
        // Nhận event tap từ VC → xử lý logic
        buttonTapped
            .sink { [weak self] in
                self?.message = "Đã bấm nút lúc \(Date())"
            }
            .store(in: &cancellables)
    }
}
```

---

## 🔹 ViewController – gửi event → subscribe message

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    private let viewModel = ButtonViewModel()
    private var cancellables = Set<AnyCancellable>()

    private let label = UILabel()
    private let button = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .systemBackground
        setupUI()
        bindViewModel()

        // gửi event tap
        button.addTarget(self, action: #selector(handleTap), for: .touchUpInside)
    }

    @objc private func handleTap() {
        viewModel.buttonTapped.send(())
    }

    private func bindViewModel() {
        viewModel.$message
            .sink { [weak self] text in
                self?.label.text = text
            }
            .store(in: &cancellables)
    }

    private func setupUI() {
        label.font = .systemFont(ofSize: 20)
        label.textAlignment = .center
        button.setTitle("Nhấn tôi", for: .normal)

        view.addSubview(label)
        view.addSubview(button)

        label.translatesAutoresizingMaskIntoConstraints = false
        button.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -40),
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.topAnchor.constraint(equalTo: label.bottomAnchor, constant: 20)
        ])
    }
}
```

---

# 🧪 KẾT QUẢ:

✔ Nhấn button → viewModel nhận event → gửi message mới
✔ UI tự update
✔ Không closure
✔ Không delegate
✔ Không retain cycle

---

# 🟧 **6. Ví dụ 2 — STATE STORE (CurrentValueSubject)**

Ví dụ: đếm số lượng (giống bài 13) nhưng dùng CurrentValueSubject.

---

## 🔹 ViewModel – dùng CurrentValueSubject

```swift
import Combine

class CounterViewModel {

    // STATE (khởi tạo bằng 0)
    let count = CurrentValueSubject<Int, Never>(0)

    func increment() {
        count.send(count.value + 1)
    }

    func decrement() {
        count.send(count.value - 1)
    }
}
```

---

## 🔹 ViewController – subscribe

```swift
viewModel.count
    .sink { [weak self] value in
        self?.label.text = "\(value)"
    }
    .store(in: &cancellables)
```

---

# 🧠 7. So sánh 2 ví dụ

| Tình huống              | Giải pháp               |
| ----------------------- | ----------------------- |
| Button tap event        | **PassthroughSubject**  |
| Lưu trạng thái số       | **CurrentValueSubject** |
| Sự kiện 1 lần           | Passthrough             |
| Sự kiện cần nhớ giá trị | CurrentValue            |

---

# 🧠 8. Kết hợp trong MVVM chuẩn

MVVM hiện đại:

* Input → PassthroughSubject
* Output → @Published hoặc CurrentValueSubject

Ví dụ chuẩn:

```
struct Input {
   let refresh = PassthroughSubject<Void, Never>()
}

struct Output {
   @Published var items: [Item] = []
}
```

---

# 🎯 Tóm tắt Bài 14

## PassthroughSubject

* Không nhớ giá trị
* Không có initial value
* Dùng cho event (tap, refresh, navigation)

## CurrentValueSubject

* Có initial value
* Nhớ giá trị hiện tại
* Subscriber mới vẫn nhận được giá trị
* Dùng cho State

---

# 🎁 Bài tập cuối bài

### Bài 1

Tạo ViewModel có:

```
var didSubmit = PassthroughSubject<String, Never>()
```

→ Gửi tên người dùng khi nhấn Submit.

### Bài 2

Tạo counter bằng CurrentValueSubject như bài 13.

### Bài 3

Tạo ViewModel có 2 biến:

```
let username = CurrentValueSubject<String, Never>("")
let password = CurrentValueSubject<String, Never>("")
```

→ Combine 2 biến để bật nút Login khi đủ điều kiện.

---

Nếu xong bài 14 → mình qua:

# 🌟 **Bài 15 — Map, Filter, RemoveDuplicates (Xử lý Pipeline như RxSwift)**

Chỉ cần nói:
👉 **“Qua bài 15 đi thầy”**
