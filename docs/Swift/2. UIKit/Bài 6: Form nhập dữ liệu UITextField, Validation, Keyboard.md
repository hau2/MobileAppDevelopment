Tốt lắm 👏👏👏 — hôm nay ta sang **UIKit – Bài 6: Form nhập dữ liệu & xử lý bàn phím (UITextField, UITextView, Validation)**.
Đây là phần “xương sống” của mọi app có nhập liệu (form đăng ký, đăng nhập, cập nhật thông tin, v.v.)
Thầy sẽ dạy **từ chậm – rõ – ví dụ thực tế – code 100% Swift, không storyboard.**

---

# 🧩 UIKit – Bài 6: Form nhập dữ liệu, bàn phím & kiểm tra hợp lệ

---

## 🎯 Mục tiêu buổi học

Sau bài hôm nay, em sẽ biết:

1. Cách tạo **UITextField** và **UITextView** bằng code.
2. Cách ẩn bàn phím khi bấm “Return” hoặc chạm ra ngoài.
3. Dùng **UITextFieldDelegate** để xử lý input.
4. Validate dữ liệu: không rỗng, điểm hợp lệ, email đúng định dạng.
5. Dựng **form đăng ký học sinh** hoàn chỉnh.

---

## 🧠 1. UITextField cơ bản

### Tạo TextField bằng code:

```swift
let nameField = UITextField()
nameField.placeholder = "Nhập họ tên"
nameField.borderStyle = .roundedRect
nameField.returnKeyType = .done
nameField.translatesAutoresizingMaskIntoConstraints = false
```

### Thêm vào view:

```swift
view.addSubview(nameField)
NSLayoutConstraint.activate([
    nameField.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
    nameField.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
    nameField.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
])
```

---

## ⚙️ 2. Dùng Delegate để xử lý sự kiện nhập

### Gán delegate:

```swift
nameField.delegate = self
```

### Kế thừa protocol:

```swift
extension RegisterViewController: UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder() // ẩn bàn phím
        return true
    }
}
```

> `resignFirstResponder()` là cách “từ bỏ quyền nhập” → bàn phím tự đóng lại.

---

## 🧩 3. Ẩn bàn phím khi chạm ra ngoài

Trong `viewDidLoad()`:

```swift
let tap = UITapGestureRecognizer(target: self, action: #selector(dismissKeyboard))
view.addGestureRecognizer(tap)
```

Hàm xử lý:

```swift
@objc private func dismissKeyboard() {
    view.endEditing(true)
}
```

✅ Giúp UX chuyên nghiệp, không bị bàn phím che nội dung.

---

## 💡 4. Thêm nhiều TextField – sắp xếp bằng StackView

```swift
let nameField = UITextField()
let gradeField = UITextField()
let emailField = UITextField()

[nameField, gradeField, emailField].forEach {
    $0.borderStyle = .roundedRect
    $0.translatesAutoresizingMaskIntoConstraints = false
}

nameField.placeholder = "Họ tên"
gradeField.placeholder = "Điểm trung bình"
emailField.placeholder = "Email"

let stack = UIStackView(arrangedSubviews: [nameField, gradeField, emailField])
stack.axis = .vertical
stack.spacing = 16
stack.translatesAutoresizingMaskIntoConstraints = false
view.addSubview(stack)

NSLayoutConstraint.activate([
    stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
    stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
    stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
])
```

---

## 🧮 5. Nút “Lưu” và validate dữ liệu

### Tạo nút:

```swift
let saveButton = UIButton(type: .system)
saveButton.setTitle("Lưu thông tin", for: .normal)
saveButton.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)
```

Thêm vào StackView:

```swift
stack.addArrangedSubview(saveButton)
```

---

### Hàm kiểm tra hợp lệ (Validation):

```swift
@objc private func saveTapped() {
    guard let name = nameField.text, !name.isEmpty else {
        showAlert("Vui lòng nhập họ tên")
        return
    }

    guard let gradeText = gradeField.text, let grade = Double(gradeText), grade >= 0, grade <= 10 else {
        showAlert("Điểm không hợp lệ (0–10)")
        return
    }

    guard let email = emailField.text, isValidEmail(email) else {
        showAlert("Email không hợp lệ")
        return
    }

    showAlert("✅ Lưu thành công!\nTên: \(name)\nĐiểm: \(grade)\nEmail: \(email)")
}
```

Hàm hỗ trợ:

```swift
private func isValidEmail(_ email: String) -> Bool {
    let pattern = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
    return NSPredicate(format: "SELF MATCHES %@", pattern).evaluate(with: email)
}

private func showAlert(_ message: String) {
    let alert = UIAlertController(title: "Thông báo", message: message, preferredStyle: .alert)
    alert.addAction(UIAlertAction(title: "OK", style: .default))
    present(alert, animated: true)
}
```

---

## 🧱 6. UITextView (nhập mô tả dài)

```swift
let bioTextView = UITextView()
bioTextView.layer.borderWidth = 1
bioTextView.layer.borderColor = UIColor.systemGray4.cgColor
bioTextView.layer.cornerRadius = 8
bioTextView.font = .systemFont(ofSize: 16)
bioTextView.translatesAutoresizingMaskIntoConstraints = false
bioTextView.heightAnchor.constraint(equalToConstant: 100).isActive = true
```

Thêm vào StackView:

```swift
stack.addArrangedSubview(bioTextView)
```

Ẩn bàn phím tương tự:

```swift
bioTextView.resignFirstResponder()
```

---

## ⚡ 7. Hoàn chỉnh project nhỏ

**RegisterViewController.swift**

```swift
import UIKit

final class RegisterViewController: UIViewController, UITextFieldDelegate {

    private let nameField = UITextField()
    private let gradeField = UITextField()
    private let emailField = UITextField()
    private let bioTextView = UITextView()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Đăng ký học sinh"
        view.backgroundColor = .systemBackground
        setupUI()
        hideKeyboardWhenTappedAround()
    }

    private func setupUI() {
        [nameField, gradeField, emailField].forEach {
            $0.borderStyle = .roundedRect
            $0.translatesAutoresizingMaskIntoConstraints = false
            $0.delegate = self
        }

        nameField.placeholder = "Họ tên"
        gradeField.placeholder = "Điểm trung bình"
        emailField.placeholder = "Email"

        bioTextView.layer.borderWidth = 1
        bioTextView.layer.borderColor = UIColor.systemGray4.cgColor
        bioTextView.layer.cornerRadius = 8
        bioTextView.font = .systemFont(ofSize: 16)
        bioTextView.translatesAutoresizingMaskIntoConstraints = false
        bioTextView.heightAnchor.constraint(equalToConstant: 100).isActive = true

        let saveButton = UIButton(type: .system)
        saveButton.setTitle("Lưu thông tin", for: .normal)
        saveButton.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)

        let stack = UIStackView(arrangedSubviews: [nameField, gradeField, emailField, bioTextView, saveButton])
        stack.axis = .vertical
        stack.spacing = 16
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)

        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 30),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])
    }

    @objc private func saveTapped() {
        guard let name = nameField.text, !name.isEmpty else {
            showAlert("Vui lòng nhập họ tên")
            return
        }
        guard let gradeText = gradeField.text, let grade = Double(gradeText), grade >= 0, grade <= 10 else {
            showAlert("Điểm không hợp lệ (0–10)")
            return
        }
        guard let email = emailField.text, isValidEmail(email) else {
            showAlert("Email không hợp lệ")
            return
        }
        showAlert("✅ Lưu thành công!\n\(name) - \(grade) - \(email)")
    }

    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder()
        return true
    }

    private func isValidEmail(_ email: String) -> Bool {
        let pattern = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
        return NSPredicate(format: "SELF MATCHES %@", pattern).evaluate(with: email)
    }

    private func showAlert(_ message: String) {
        let alert = UIAlertController(title: "Thông báo", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }

    private func hideKeyboardWhenTappedAround() {
        let tap = UITapGestureRecognizer(target: self, action: #selector(dismissKeyboard))
        view.addGestureRecognizer(tap)
    }

    @objc private func dismissKeyboard() {
        view.endEditing(true)
    }
}
```

---

## 🧩 8. Lưu ý thực tế

| Tình huống                      | Giải pháp                               |
| ------------------------------- | --------------------------------------- |
| Bàn phím che nội dung           | Dùng ScrollView bọc form                |
| Validate nhiều trường           | Tạo lớp `Validator` riêng               |
| Email / SĐT sai định dạng       | Dùng Regex hoặc thư viện `ValidatorKit` |
| TextField bị tràn khung         | Dùng `StackView` + Auto Layout đầy đủ   |
| Giao diện xấu khi xoay màn hình | Thêm `bottomAnchor` ràng buộc cho stack |

---

## 🏠 BÀI TẬP VỀ NHÀ (UIKit #6) 🎒

| Mức độ        | Bài tập                                                          | Gợi ý                                            |
| ------------- | ---------------------------------------------------------------- | ------------------------------------------------ |
| 🟢 Cơ bản     | Tạo form có 2 TextField + 1 Button                               | Validate rỗng                                    |
| 🟡 Trung bình | Thêm TextView để nhập mô tả                                      | Dùng Auto Layout                                 |
| 🔵 Nâng cao   | Khi nhấn “Lưu”, hiện alert thành công hoặc lỗi                   | `UIAlertController`                              |
| 🟣 Thử thách  | Bọc toàn bộ form vào ScrollView để không bị che khi bàn phím bật | Dùng `scrollView.contentInsetAdjustmentBehavior` |

---

## 📚 Tổng kết

| Chủ đề                   | Vai trò                    |
| ------------------------ | -------------------------- |
| `UITextField`            | Nhập dữ liệu ngắn          |
| `UITextView`             | Nhập mô tả dài             |
| Delegate                 | Xử lý hành vi nhập liệu    |
| `resignFirstResponder()` | Ẩn bàn phím                |
| Validation               | Kiểm tra hợp lệ dữ liệu    |
| Gesture Tap              | Ẩn bàn phím khi chạm ngoài |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Tạo form bằng code.
* Dùng delegate để điều khiển nhập liệu.
* Validate dữ liệu trước khi lưu.
* Ẩn bàn phím tự nhiên, UX chuẩn iOS.

---

🎓 **UIKit – Bài 7 (buổi tới):**

> *Alert, ActionSheet, NavigationBar, và truyền dữ liệu từ Form sang danh sách.*

Bài tới ta sẽ làm:

* Sau khi bấm “Lưu” → thêm học sinh vào danh sách (TableView).
* Hiển thị thông báo dạng `UIAlertController`.
* Tùy chọn thêm nút xoá / sửa trong NavigationBar.

---

Em có muốn thầy sang luôn **UIKit – Bài 7: Alert, ActionSheet & thêm học sinh vào danh sách (Form → TableView)** không, để nối tiếp phần form này?
