Ok, qua **Bài 8 – Closure Pattern trong CustomView** 👨‍🏫
Và bắt đầu từ bài này, thầy sẽ **thêm phần: “Tại sao phải dùng nó?”** cho mỗi kỹ thuật.
→ Để em hiểu **bản chất** chứ không chỉ copy code.

---

# 🎯 **MỤC TIÊU BÀI 8**

Sau bài này em sẽ hiểu:

✔ Closure pattern trong CustomView là gì
✔ Tại sao phải dùng closure thay vì delegate
✔ Cách tạo một CustomView có callback
✔ Cách truyền sự kiện từ View → ViewController
✔ Cách tránh memory leak (weak self)
✔ Code đầy đủ copy/paste được

---

# 🧠 **1. Tại sao phải dùng Closure Pattern trong CustomView?**

Giả sử em tạo 1 view popup/bottom-sheet như:

```
+--------------------------+
|    Bạn có muốn xoá?     |
|   [Cancel]   [Delete]   |
+--------------------------+
```

**Câu hỏi:**
*Làm sao ViewController biết người dùng bấm Delete?*

Có 2 cách:

### **Cách 1 – Delegate (rườm rà, nhiều file, nhiều code)**

* Tạo protocol
* Tạo delegate property
* Implement ở VC
* Gán delegate

### **Cách 2 – Closure Pattern (ngắn gọn, hiện đại, dễ hiểu)**

* Trong view, em tạo closure:

  ```swift
  var onDelete: (() -> Void)?
  ```
* Khi bấm nút:

  ```swift
  onDelete?()
  ```
* Tại ViewController nhận callback:

  ```swift
  popup.onDelete = { print("Đã bấm xoá") }
  ```

**Lý do closure tốt hơn delegate:**

| Delegate                   | Closure                |
| -------------------------- | ---------------------- |
| dài dòng                   | ngắn                   |
| cần tạo protocol           | không                  |
| phải implement ở nhiều nơi | chỉ gán closure 1 dòng |
| khó đọc                    | rất dễ đọc             |
| phổ biến ở iOS cũ          | xu hướng iOS modern    |

👉 **Các team iOS 2022–2025 gần như bỏ delegate cho các view nhỏ**, dùng **closure pattern 90%**.

---

# 🧱 **2. Xây dựng CustomView có callback (thầy dạy chậm)**

Chúng ta sẽ tạo 1 CustomView gồm:

* Title label
* Button "Tap me"
* Callback `onTap`

📌 Tập trung vào 3 điểm:

* UI trong CustomView
* Closure property
* Gọi closure khi bấm nút

---

# 📌 **3. File CustomView.swift – CODE HOÀN CHỈNH + COMMENT**

Tạo file mới: **CustomActionView.swift**

```swift
import UIKit

/// CustomView này dùng để demo Closure Pattern.
/// Nó có 1 cái title và 1 cái button.
/// Khi người dùng bấm button → view gọi closure onTap()
class CustomActionView: UIView {

    // MARK: - Callback (Closure)

    /// Closure này ViewController sẽ gán hành vi vào.
    /// Mỗi khi user bấm nút → ta gọi onTap?()
    ///
    /// Vì sao dùng closure?
    /// - Giúp CustomView không cần biết ViewController là ai
    /// - Không cần delegate/protocol rườm rà
    /// - ViewController muốn làm gì khi nút bấm thì gán vào closure này
    var onTap: (() -> Void)?

    // MARK: - UI Components

    private let titleLabel: UILabel = {
        let label = UILabel()
        label.text = "Đây là CustomView"
        label.font = .systemFont(ofSize: 20, weight: .bold)
        label.textAlignment = .center
        return label
    }()

    private let actionButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("Nhấn vào đây", for: .normal)
        button.titleLabel?.font = .systemFont(ofSize: 18, weight: .medium)
        button.backgroundColor = .systemBlue
        button.tintColor = .white
        button.layer.cornerRadius = 10
        return button
    }()

    // MARK: - Init

    override init(frame: CGRect) {
        super.init(frame: frame)

        backgroundColor = .secondarySystemBackground
        layer.cornerRadius = 12

        addSubview(titleLabel)
        addSubview(actionButton)

        setupConstraints()

        // gán action cho nút
        actionButton.addTarget(self, action: #selector(handleTap), for: .touchUpInside)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - AutoLayout

    private func setupConstraints() {
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        actionButton.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            titleLabel.topAnchor.constraint(equalTo: topAnchor, constant: 20),
            titleLabel.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            titleLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),

            actionButton.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 20),
            actionButton.centerXAnchor.constraint(equalTo: centerXAnchor),
            actionButton.widthAnchor.constraint(equalToConstant: 160),
            actionButton.heightAnchor.constraint(equalToConstant: 44),

            actionButton.bottomAnchor.constraint(equalTo: bottomAnchor, constant: -20)
        ])
    }

    // MARK: - Action

    /// Hàm này chạy khi người dùng bấm button
    @objc private func handleTap() {
        print("Button trong CustomView được bấm")

        /// GỌI CLOSURE
        onTap?()
    }
}
```

---

# 🧱 **4. Dùng CustomView trong ViewController**

Mở **ViewController.swift**, thay toàn bộ bằng đoạn sau:

```swift
import UIKit

class ViewController: UIViewController {

    /// Tạo một instance của CustomActionView
    private let actionView = CustomActionView()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        title = "Bài 8 - Closure trong CustomView"

        view.addSubview(actionView)
        setupConstraints()

        // MARK: - GÁN CLOSURE
        // Chỗ này rất quan trọng.
        // ViewController quyết định hành vi khi CustomView được bấm.
        actionView.onTap = { [weak self] in
            guard let self = self else { return }
            self.showMessage()
        }
    }

    private func setupConstraints() {
        actionView.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            actionView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            actionView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            actionView.widthAnchor.constraint(equalToConstant: 260)
        ])
    }

    private func showMessage() {
        let alert = UIAlertController(
            title: "CustomView",
            message: "Bạn đã nhấn nút trong CustomView!",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

---

# 🧪 **Kết quả khi chạy**

✔ CustomView nằm giữa màn hình
✔ Gồm title + button
✔ Khi nhấn button → VC nhận được callback
✔ Hiện popup “Bạn đã nhấn nút…”
✔ Closure hoạt động hoàn hảo, không leak memory

---

# 🎯 **6. Vì sao Closure Pattern trong CustomView lại quan trọng?**

Trong dự án thực tế:

* Popup confirm
* Bottom Sheet
* Custom Header View
* Custom Cell (MVVM UI interaction)
* Modal view
* Dialog
* Date picker custom
* Filter panel
* Search bar
* Custom input view

→ **Tất cả đều dùng closure để truyền event ngược lại VC**.

### Nếu *không* dùng closure?

👉 Sẽ phải dùng delegate → code dài, phải tạo protocol, bất tiện.

### Closure giúp:

* Rõ ràng
* Gọn
* Dễ đọc
* Dễ test
* Dễ reuse View ở nhiều nơi

**Đây là tiêu chuẩn trong kiến trúc hiện đại: MVVM + UIKit Code.**

---

# 🎁 Bài tập củng cố

### **Bài 1:**

Thêm 1 nút “Huỷ” vào CustomActionView.
→ tạo closure `onCancel`

### **Bài 2:**

Thay vì 1 closure, hãy tạo struct Event:

```swift
struct Event {
   var onTap: (() -> Void)?
   var onCancel: (() -> Void)?
}
```

### **Bài 3:**

Hiển thị số lần bấm button trên label trong CustomView.

Nếu em làm xong, gửi code thầy review.

---

# 👉 Sẵn sàng học tiếp bài 9 (Closure trong UITableViewCell) không?

Chỉ cần nói:

**“Qua bài 9 đi thầy”**
