Ok, mình qua **Bài 2** nhé!
Bài này cực quan trọng vì bạn sẽ dùng **AutoLayout bằng anchor** suốt cả lộ trình UIKit.

Thầy sẽ dạy thật **chậm – dễ hiểu – có code chạy được – có hình dung trực quan**.

---

# 🎯 **BÀI 2 — AUTO LAYOUT BẰNG ANCHOR (UIKit code thuần)**

Sau bài này, bạn sẽ:

✔ Biết dùng `translatesAutoresizingMaskIntoConstraints = false`
✔ Biết gắn constraint bằng Anchor
✔ Biết 5 nhóm constraint quan trọng nhất
✔ Tự căn giữa 1 view
✔ Tự layout 2–3 view theo chiều dọc
✔ Copy/paste chạy được trên project Bài 1

---

# 🧱 **1. Vì sao phải dùng AutoLayout bằng code?**

UIKit khi code thuần thì **không thể đặt frame cứng** (vì nhiều kích thước màn hình):

❌ Không dùng cách này:

```swift
label.frame = CGRect(x: 0, y: 0, width: 200, height: 50)
```

✔ Thay vào đó dùng AutoLayout:

```swift
label.centerXAnchor.constraint(equalTo: view.centerXAnchor)
```

---

# 🧱 **2. Quy tắc BẮT BUỘC trước khi dùng AutoLayout**

Mọi view muốn dùng Anchor phải tắt AutoResizingMask:

```swift
label.translatesAutoresizingMaskIntoConstraints = false
```

Nếu quên dòng này → UI sai, lệch, hoặc crash.

---

# 🧱 **3. 5 nhóm Anchor bạn phải thuộc lòng**

### (1) **Vị trí**

* `topAnchor`
* `bottomAnchor`
* `leadingAnchor`
* `trailingAnchor`

Ví dụ:

```swift
label.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16)
```

---

### (2) **Căn giữa (center)**

* `centerXAnchor`
* `centerYAnchor`

Ví dụ:

```swift
label.centerXAnchor.constraint(equalTo: view.centerXAnchor)
```

---

### (3) **Kích thước**

* `widthAnchor`
* `heightAnchor`

Ví dụ:

```swift
button.heightAnchor.constraint(equalToConstant: 44)
```

---

### (4) **Tạo khoảng cách giữa các view**

```swift
label2.topAnchor.constraint(equalTo: label1.bottomAnchor, constant: 20)
```

---

### (5) **Safe Area Layout Guide**

Dùng cho top/bottom để tránh tai thỏ (notch):

```swift
view.safeAreaLayoutGuide.topAnchor
```

---

# 🟦 **4. Code hoàn chỉnh Bài 2 (copy/paste chạy được)**

👉 Bạn thay toàn bộ file `ViewController.swift` bằng bản dưới.
👉 Đây là demo **2 label + 1 button**, layout bằng Anchor.

---

## 📌 **ViewController.swift**

```swift
import UIKit

class ViewController: UIViewController {

    private let titleLabel: UILabel = {
        let label = UILabel()
        label.text = "AutoLayout bằng Anchor"
        label.font = .systemFont(ofSize: 26, weight: .bold)
        label.textAlignment = .center
        return label
    }()

    private let subtitleLabel: UILabel = {
        let label = UILabel()
        label.text = "Bài 2 - Căn giữa và đặt layout theo chiều dọc"
        label.font = .systemFont(ofSize: 16)
        label.textAlignment = .center
        label.textColor = .secondaryLabel
        return label
    }()

    private let actionButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("Nhấn em nhẹ nhàng", for: .normal)
        button.titleLabel?.font = .systemFont(ofSize: 18, weight: .medium)
        button.backgroundColor = .systemBlue
        button.tintColor = .white
        button.layer.cornerRadius = 10
        return button
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        // Thêm views
        view.addSubview(titleLabel)
        view.addSubview(subtitleLabel)
        view.addSubview(actionButton)

        setupLayout()

        actionButton.addTarget(self, action: #selector(handleTap), for: .touchUpInside)
    }

    private func setupLayout() {
        // Tắt autoresizing cho tất cả view
        [titleLabel, subtitleLabel, actionButton].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
        }

        NSLayoutConstraint.activate([
            // ===== 1. TitleLabel =====
            titleLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 100),
            titleLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            titleLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),

            // ===== 2. SubtitleLabel =====
            subtitleLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 16),
            subtitleLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),
            subtitleLabel.trailingAnchor.constraint(equalTo: titleLabel.trailingAnchor),

            // ===== 3. Button =====
            actionButton.topAnchor.constraint(equalTo: subtitleLabel.bottomAnchor, constant: 40),
            actionButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            actionButton.widthAnchor.constraint(equalToConstant: 220),
            actionButton.heightAnchor.constraint(equalToConstant: 50)
        ])
    }

    @objc private func handleTap() {
        print("Bạn đã nhấn nút!")
    }
}
```

---

# 🧪 **Kết quả:**

Khi chạy app:

✔ Dòng `"AutoLayout bằng Anchor"` nằm cách top 100 px
✔ Dòng `"Bài 2..."` nằm dưới đó 16 px
✔ Button nằm dưới subtitle 40 px
✔ Tất cả đều căn giữa đúng

---

# 🎁 Bài tập cuối bài (rất quan trọng)

**Bài tập 1:**
Thêm 1 `UIImageView` nằm dưới button, cách 30px.

**Bài tập 2:**
Căn giữa toàn bộ UI theo chiều ngang bằng anchor.

**Bài tập 3:**
Thay anchor sau đây:

```swift
button.centerXAnchor.constraint(equalTo: view.centerXAnchor)
```

bằng cách căn trái/phải cách 50px.

**Bài tập 4:**
Tạo thêm 1 view hình vuông (100×100), đặt giữa màn hình.

Nếu làm xong gửi thầy review 🧑‍🏫

---

# 👉 Sẵn sàng qua **Bài 3 – UIStackView** chưa?

Bạn chỉ cần nói:
**“Qua bài 3 đi thầy”**
