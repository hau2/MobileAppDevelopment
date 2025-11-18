Ok, tới **Bài 15 — Map, Filter, RemoveDuplicates (Combine Operators)** 👨‍🏫
Đây là **bài xương sống** giúp em làm được:

* Form Validation (validate username, password)
* Search bar realtime
* Pagination logic
* Combine event từ nhiều publisher
* Logic reactive như RxSwift

Thầy sẽ dạy **cực kỳ chậm – chi tiết – có ví dụ MVVM UIKit – copy/paste chạy được**.

---

# 🌟 **BÀI 15 — MAP, FILTER, REMOVEDUPLICATES**

Nội dung gồm:

1. Operator là gì trong Combine?
2. `map` – xử lý dữ liệu
3. `filter` – loại bỏ dữ liệu không hợp lệ
4. `removeDuplicates` – tránh xử lý lặp lại
5. Tại sao cần dùng 3 operator này?
6. Ví dụ cơ bản (Playground style)
7. Ví dụ áp dụng MVVM (validate form login)
8. Pipeline hoàn chỉnh

---

# 🧠 1. Operator trong Combine là gì?

Operator = **phép biến đổi dòng dữ liệu (stream)**.

Ví dụ stream từ ViewModel:

```
" a"  →  "A"     →   skip     →   validated
   map    filter       removeDuplicates
```

👉 Operator nối vào nhau tạo thành **pipeline** giống RxSwift.

---

# 🟦 2. `map` – chuyển đổi dữ liệu

### Tại sao dùng?

* Khi em muốn **biến đổi** giá trị:

  * Int → String
  * String → Bool
  * Model → ViewModel
  * Data API → domain model
  * “abc” → “ABC”

### Cú pháp:

```swift
publisher
    .map { value in
        return value * 2
    }
```

---

# 🟧 3. `filter` – chỉ cho phép giá trị hợp lệ đi qua

### Tại sao dùng?

* Validate input
* Chỉ cho phép event thỏa điều kiện:

  * số > 0
  * chuỗi không rỗng
  * password đủ độ dài

### Cú pháp:

```swift
publisher
    .filter { value in
        value > 0
    }
```

---

# 🟩 4. `removeDuplicates` – bỏ event giống nhau

### Tại sao dùng?

* Tránh spam UI
* Tránh reload TableView nhiều lần
* Tránh gọi API liên tục
* Search text không gọi khi người dùng gõ cùng một từ

### Cú pháp:

```swift
publisher
    .removeDuplicates()
```

---

# 🧪 5. Ví dụ cơ bản (Playground style)

## Ví dụ 1 — map

```swift
let subject = PassthroughSubject<Int, Never>()
let cancellable = subject
    .map { $0 * 2 }
    .sink { print("Value:", $0) }

subject.send(10)
// Output: Value: 20
```

---

## Ví dụ 2 — filter

```swift
subject
    .filter { $0 > 5 }
    .sink { print("OK:", $0) }

subject.send(3)  // không in
subject.send(10) // in: OK: 10
```

---

## Ví dụ 3 — removeDuplicates

```swift
subject
    .removeDuplicates()
    .sink { print("Value:", $0) }

subject.send(1) // in
subject.send(1) // không in
subject.send(2) // in
subject.send(2) // không in
```

---

# ⭐ 6. Áp dụng thực tế — **Form Login với Combine Operators**

**Mục tiêu:**

* Khi user nhập username + password
* ViewModel xử lý pipeline:

  * loại dấu cách
  * uppercase
  * kiểm tra length
  * removeDuplicates
* Bật nút Login nếu hợp lệ

---

## 🧱 6.1 ViewModel — dùng Combine Operators

**LoginViewModel.swift**

```swift
import Combine
import Foundation

class LoginViewModel {

    // Input
    let username = CurrentValueSubject<String, Never>("")
    let password = CurrentValueSubject<String, Never>("")

    // Output
    @Published var isLoginEnabled: Bool = false

    private var cancellables = Set<AnyCancellable>()

    init() {
        setupPipeline()
    }

    private func setupPipeline() {

        // Combine 2 input
        Publishers.CombineLatest(username, password)
            // 1. map: chuyển thành boolean điều kiện login
            .map { username, password in
                return username.count >= 3 && password.count >= 6
            }
            // 2. remove duplicates: tránh update button liên tục
            .removeDuplicates()
            // 3. output
            .sink { [weak self] isValid in
                self?.isLoginEnabled = isValid
            }
            .store(in: &cancellables)
    }
}
```

### Thầy giải thích:

1. `CombineLatest(username, password)`
   → viewModel lắng nghe CẢ HAI input

2. `map`
   → chuyển input thành điều kiện boolean

3. `removeDuplicates()`
   → tránh gọi update UI không cần thiết

4. `.sink`
   → subscriber nhận state hợp lệ và gán vào `isLoginEnabled`

---

## 🧱 6.2 ViewController — subscribe để bật nút Login

**ViewController.swift**

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    private let viewModel = LoginViewModel()
    private var cancellables = Set<AnyCancellable>()

    private let usernameField = UITextField()
    private let passwordField = UITextField()
    private let loginButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()

        setupUI()
        bindViewModel()
    }

    private func bindViewModel() {

        // Input pipeline
        usernameField.addTarget(self, action: #selector(usernameChanged), for: .editingChanged)
        passwordField.addTarget(self, action: #selector(passwordChanged), for: .editingChanged)

        // Output pipeline
        viewModel.$isLoginEnabled
            .sink { [weak self] enabled in
                self?.loginButton.isEnabled = enabled
                self?.loginButton.alpha = enabled ? 1 : 0.5
            }
            .store(in: &cancellables)
    }

    @objc private func usernameChanged() {
        viewModel.username.send(usernameField.text ?? "")
    }

    @objc private func passwordChanged() {
        viewModel.password.send(passwordField.text ?? "")
    }

    private func setupUI() {
        view.backgroundColor = .systemBackground

        usernameField.placeholder = "Username ≥ 3 ký tự"
        passwordField.placeholder = "Password ≥ 6 ký tự"
        passwordField.isSecureTextEntry = true

        loginButton.setTitle("Login", for: .normal)
        loginButton.isEnabled = false
        loginButton.alpha = 0.5

        let stack = UIStackView(arrangedSubviews: [usernameField, passwordField, loginButton])
        stack.axis = .vertical
        stack.spacing = 20

        view.addSubview(stack)
        stack.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            stack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            stack.widthAnchor.constraint(equalToConstant: 260)
        ])
    }
}
```

---

# 🧪 KẾT QUẢ DEMO

✔ Nhập username < 3 ký tự → Login **off**
✔ Nhập password < 6 ký tự → Login **off**
✔ Cả 2 hợp lệ → Login **ON**, nút sáng lên
✔ Không bắn lại event nếu input không đổi (`removeDuplicates`)
✔ Không cần closure hay delegate → pure Combine

---

# 🚀 TÓM TẮT BÀI 15

| Operator             | Chức năng                 | Khi dùng                      |
| -------------------- | ------------------------- | ----------------------------- |
| **map**              | Chuyển giá trị A → B      | validate form, chuyển model   |
| **filter**           | Loại bỏ giá trị không hợp | chặn error, validate text     |
| **removeDuplicates** | Bỏ giá trị lặp            | tối ưu UI, tránh reload 2 lần |

---

# 🎁 Bài tập cuối bài

### **Bài 1**

Thêm operator:

```
.trim()
```

→ loại khoảng trắng đầu cuối username.

### **Bài 2**

Tạo form email:
email phải chứa `"@"` → dùng filter.

### **Bài 3 (nâng cao)**

Dùng pipeline:

```
username
    .debounce(0.3, scheduler: RunLoop.main)
    .removeDuplicates()
    .sink(…)
```

→ làm search bar realtime (gần giống Instagram search).

---

# 👉 Sẵn sàng qua **Bài 16 — Debounce, CombineLatest, Zip & Merge** chưa thầy?

Đây là bài cực mạnh để làm search bar, pagination, debounce API.

Chỉ cần nói:

**“Qua bài 16 đi thầy”**
