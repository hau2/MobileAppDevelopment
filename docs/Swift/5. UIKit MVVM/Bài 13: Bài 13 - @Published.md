Ok, thầy sẽ bắt đầu **chuỗi 3 bài nâng cao của Combine** theo đúng yêu cầu của em:

* **Bài 13 — @Published**
* **Bài 14 — PassthroughSubject & CurrentValueSubject**
* **Bài 15 — Map, Filter, RemoveDuplicates**

Thầy sẽ đi bài **13** trước, thật chậm – thật chi tiết – có code đầy đủ, có giải thích "tại sao dùng".

---

# 🌟 **BÀI 13 — @Published (FULL GIẢI THÍCH + VÍ DỤ RÕ RÀNG)**

> Đây là bài quan trọng nhất trong Combine, vì 90% code MVVM UIKit + SwiftUI dùng `@Published`.

---

# 🧠 1. @Published là gì?

`@Published` là **property wrapper** của Combine dùng để:

✔ Tự động biến 1 biến bình thường thành **Publisher**
✔ Mỗi lần biến thay đổi → phát ra giá trị mới cho Subscriber
✔ VC chỉ cần subscribe 1 lần, UI tự update

Nó tương đương:

```
ViewModel → UI tự update khi data thay đổi
```

---

# 🔍 2. Khi nào dùng @Published?

Dùng khi:

* ViewModel có biến **state** mà ViewController phải hiển thị
* Ví dụ:

  * đang loading hay không
  * danh sách sản phẩm
  * số lượng giỏ hàng
  * error message
  * form validation
  * search text

👉 Khi giá trị thay đổi → UI update tự động.

---

# 🧩 3. Cấu trúc @Published bên trong

Khi em viết:

```swift
@Published var count = 0
```

Combine tự tạo cho em 2 thứ:

### 1️⃣ Storage (biến bình thường):

`count`

### 2️⃣ Publisher:

`$count`

* `$count` là `Published<Int>.Publisher`
* `$` dùng để truy cập Publisher

👉 VC subscribe `$count`

---

# 📌 4. Ví dụ đơn giản nhất

Tạo file **CounterViewModel.swift**:

```swift
import Foundation
import Combine

class CounterViewModel: ObservableObject {

    /// Đây là biến state.
    /// Mỗi khi count thay đổi → Combine phát sự kiện.
    @Published var count: Int = 0

    func increment() {
        count += 1
    }

    func decrement() {
        count -= 1
    }
}
```

---

# 📌 5. ViewController subscribe (@Published + .sink)

Tạo file **ViewController.swift**:

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    private let viewModel = CounterViewModel()
    private var cancellables = Set<AnyCancellable>()

    private let label: UILabel = {
        let l = UILabel()
        l.font = .systemFont(ofSize: 40, weight: .bold)
        l.textAlignment = .center
        l.text = "0"
        return l
    }()

    private let plusButton: UIButton = {
        let b = UIButton(type: .system)
        b.setTitle("+", for: .normal)
        b.titleLabel?.font = .systemFont(ofSize: 40, weight: .bold)
        return b
    }()

    private let minusButton: UIButton = {
        let b = UIButton(type: .system)
        b.setTitle("-", for: .normal)
        b.titleLabel?.font = .systemFont(ofSize: 40, weight: .bold)
        return b
    }()

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .systemBackground

        layoutUI()
        bindViewModel()
        addActions()
    }

    private func bindViewModel() {

        // 🔥 Subscribe vào Publisher $count
        viewModel.$count
            .sink { [weak self] newValue in
                self?.label.text = "\(newValue)"
            }
            .store(in: &cancellables)
    }

    private func addActions() {
        plusButton.addTarget(self, action: #selector(increase), for: .touchUpInside)
        minusButton.addTarget(self, action: #selector(decrease), for: .touchUpInside)
    }

    @objc private func increase() {
        viewModel.increment()
    }

    @objc private func decrease() {
        viewModel.decrement()
    }

    private func layoutUI() {
        view.addSubview(label)
        view.addSubview(plusButton)
        view.addSubview(minusButton)

        label.translatesAutoresizingMaskIntoConstraints = false
        plusButton.translatesAutoresizingMaskIntoConstraints = false
        minusButton.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor),

            plusButton.topAnchor.constraint(equalTo: label.bottomAnchor, constant: 20),
            plusButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),

            minusButton.topAnchor.constraint(equalTo: plusButton.bottomAnchor, constant: 20),
            minusButton.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }
}
```

---

# 🧪 Kết quả demo

✔ Nhấn + → label tự tăng
✔ Nhấn – → label tự giảm
✔ Chỉ subscribe 1 lần → UI auto-update
✔ Không cần closure
✔ Không cần delegate
✔ Combine hoạt động hoàn hảo

---

# 🎯 6. Tóm tắt Bài 13

| Khái niệm                   | Ý nghĩa                  |
| --------------------------- | ------------------------ |
| `@Published`                | biến trở thành Publisher |
| `$count`                    | Publisher phát event     |
| `.sink {}`                  | Subscriber nhận event    |
| `.store(in: &cancellables)` | giữ subscription sống    |

---

# 🏁 **Bài 13 đã xong. Sẵn sàng qua Bài 14 chưa?**

Bài 14 sẽ cực quan trọng:

# 🌟 **BÀI 14 — PassthroughSubject & CurrentValueSubject**

* So sánh 2 thằng
* Khi nào phải dùng cái nào
* Tại sao `PassthroughSubject` cực hợp để xử lý “button tap event”
* Tại sao `CurrentValueSubject` hợp cho state
* Ví dụ UIKit: button tap → PassthroughSubject
* Code đầy đủ Combine Pipeline

Chỉ cần nói:
👉 **“Qua bài 14 đi thầy”**
