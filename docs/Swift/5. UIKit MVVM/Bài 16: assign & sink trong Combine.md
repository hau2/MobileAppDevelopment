Ok thầy hiểu.
Bắt đầu **TUẦN 4 – Bài 16** theo đúng roadmap của em.
Thầy sẽ dạy cực kỳ **chậm – chi tiết – code đầy đủ – giải thích rõ tại sao dùng**, để em nắm Combine trong UIKit một cách *CHẮC NỊCH* (vì rất nhiều dev học Combine nhưng chỉ học theo kiểu “SwiftUI”, còn UIKit thì bỏ trống → sai kiến trúc, sai MVVM).

---

# 🌟 **BÀI 16 — `assign` & `sink` trong Combine**

*(Binding dữ liệu ViewModel → UI trong UIKit)*

## Mục tiêu bài học:

* Hiểu **khác nhau giữa `.assign` và `.sink`**
* Khi nào dùng `.assign`
* Khi nào vẫn phải dùng `.sink`
* Ví dụ đầy đủ:

  * ViewModel xuất `@Published`
  * ViewController subscribe bằng `.assign`
  * Code chạy được, rất gọn
* Giải thích kiến trúc “MVVM + Combine + UIKit”

---

# 🧠 1. Tại sao phải học `assign`?

Vì trong MVVM:

* VM phát dữ liệu (publisher)
* VC cập nhật UI

Trước giờ ta dùng:

```swift
viewModel.$text
    .sink { [weak self] v in self?.label.text = v }
```

Nhưng Combine cung cấp cách ngắn hơn:

```swift
viewModel.$text
    .assign(to: \.text, on: label)
```

🔥 **Cực kỳ ngắn gọn**

---

# 🧠 2. `assign(to:on:)` là gì?

Nó nhận 2 thứ:

* **keyPath**: thuộc tính UI muốn thay đổi
* **object**: nơi chứa thuộc tính đó (label, button, viewModel khác…)

### Ví dụ:

```swift
publisher.assign(to: \.text, on: label)
```

Ý nghĩa:

* Mỗi lần publisher phát giá trị → gán thẳng vào `label.text`

---

# 🧠 3. `assign(to:)` là gì?

Dùng để **bind publisher → property @Published trong ViewModel khác**.

Ví dụ:

```swift
publisher
    .assign(to: &viewModel.$text)
```

---

# 💥 4. Sự khác nhau: `sink` vs `assign`

| Khía cạnh          | `.assign`                               | `.sink`                   |
| ------------------ | --------------------------------------- | ------------------------- |
| Công dụng          | Assign value trực tiếp vào property     | Xử lý logic tùy ý         |
| Ngắn gọn           | ✔ Rất gọn                               | ❌ hơi dài                 |
| Flexible           | ❌ không linh hoạt                       | ✔ dùng cho mọi loại logic |
| UI update          | ✔ tốt                                   | ✔ tốt                     |
| Validate/transform | ❌ không làm được                        | ✔ làm được                |
| Khi dùng           | Khi chỉ muốn gán value vào một property | Khi có nhiều xử lý logic  |

👉 Quy tắc nhớ:

> * **Nếu chỉ update UI → dùng assign**
> * **Nếu cần logic → dùng sink**

---

# 🧩 5. Ví dụ hoàn chỉnh — MVVM + Combine + assign

## 1️⃣ ViewModel

```swift
import Combine

class CounterViewModel {

    @Published var count = 0

    func increase() {
        count += 1
    }

    func decrease() {
        count -= 1
    }
}
```

---

## 2️⃣ ViewController — binding bằng `.assign`

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    private let viewModel = CounterViewModel()
    private var cancellables = Set<AnyCancellable>()

    private let label = UILabel()
    private let plusButton = UIButton(type: .system)
    private let minusButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()

        setupUI()
        bindViewModel()
        addActions()
    }

    private func bindViewModel() {

        // 🔥 Binding count → label.text
        viewModel.$count
            .map { "\($0)" }              // assign yêu cầu đúng type
            .assign(to: \.text, on: label)
            .store(in: &cancellables)
    }

    private func addActions() {
        plusButton.addTarget(self, action: #selector(onPlus), for: .touchUpInside)
        minusButton.addTarget(self, action: #selector(onMinus), for: .touchUpInside)
    }

    @objc private func onPlus() {
        viewModel.increase()
    }

    @objc private func onMinus() {
        viewModel.decrease()
    }

    private func setupUI() {
        label.font = .systemFont(ofSize: 45, weight: .bold)
        plusButton.setTitle("+", for: .normal)
        minusButton.setTitle("-", for: .normal)

        let stack = UIStackView(arrangedSubviews: [label, plusButton, minusButton])
        stack.axis = .vertical
        stack.spacing = 20
        stack.alignment = .center

        view.addSubview(stack)
        stack.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
}
```

---

# 🔥 6. Kết quả Binding bằng `assign` (rất sạch & gọn)

* Không cần closure
* Không cần delegate
* Không cần xử lý `weak self`
* Không cần viết nhiều dòng

Dòng cực gọn:

```swift
viewModel.$count
    .map(String.init)
    .assign(to: \.text, on: label)
```

---

# 🧠 7. Vậy khi nào *KHÔNG dùng assign*?

**1) Nếu có logic trong UI update**

Ví dụ:

```swift
sink {
    self.view.backgroundColor = $0 > 5 ? .red : .blue
}
```

Assign không làm được.

---

**2) Nếu cần validate hoặc transform phức tạp**

Ví dụ:

```swift
map { $0 * 2 }
filter { $0 < 100 }
removeDuplicates()
sink { ... }
```

Assign không handle được các case này.

---

**3) Nếu update nhiều property cùng lúc**

Sink cho phép:

```swift
sink { [weak self] value in
    self?.label.text = "\(value)"
    self?.button.isEnabled = value > 5
    self?.view.alpha = Double(value) / 10
}
```

Assign không làm được.

---

# 🎯 Tóm tắt Bài 16

| Kết luận                  | Ý nghĩa                                       |
| ------------------------- | --------------------------------------------- |
| `assign`                  | khi chỉ cần bind value → property             |
| `sink`                    | khi cần xử lý logic trước khi update UI       |
| `assign` + Combine + MVVM | cực phù hợp với binding trong UIKit           |
| `sink`                    | vẫn là operator quan trọng nhất trong Combine |

---

# 🎁 Bài tập cuối bài 16

### **Bài 1**

Tạo ViewModel:

```
@Published var title = "Hello"
```

→ VC dùng `.assign(to:)` để bind title → `navigationItem.title`.

### **Bài 2**

Tạo:

```
@Published var isDarkMode = false
```

→ Khi giá trị thay đổi:

* nếu true → background đen
* nếu false → background trắng

**Lưu ý:** bài này không dùng assign → chỉ dùng `.sink` (vì có logic)

---

# ✔ Ready cho Bài 17?

**Bài 17 — Combine trong UIKit (cực quan trọng!)**
Nội dung bao gồm:

* Làm thế nào để bắt sự kiện từ textField bằng Combine?
* Tạo `textPublisher` cho UITextField
* Làm SearchBar real-time
* Tạo extension Combine cho UIKit
* Cách tránh memory leak với UIControl + Combine

Chỉ cần nói:
👉 **"Qua bài 17 đi thầy"**
