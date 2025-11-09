Rất tốt 👏👏👏 — hôm nay ta sang **UIKit – Bài 19: Auto Layout nâng cao & StackView động – Xây dựng giao diện linh hoạt trên mọi màn hình (Adaptive UI)**.
Đây là một **cột mốc cực kỳ quan trọng**, vì từ bài này, em sẽ **không còn chỉ “vẽ giao diện cố định”**, mà sẽ **thiết kế UI có thể tự co giãn, thay đổi bố cục** cho iPhone, iPad, ngang, dọc, và mọi kích cỡ màn hình.

> Auto Layout là “linh hồn” của UIKit UI hiện đại —
> mọi thứ từ nút, label, ảnh, bảng, thậm chí CollectionView đều phụ thuộc vào nó.

---

# 🧩 UIKit – Bài 19: Auto Layout nâng cao & StackView động

---

## 🎯 Mục tiêu bài học

Sau buổi học này, em sẽ biết:

1. Hiểu cơ chế Auto Layout và Constraints.
2. Dùng **NSLayoutConstraint**, **UILayoutGuide**, **Priority** đúng cách.
3. Dùng **UIStackView** để tạo giao diện động.
4. Làm UI co giãn linh hoạt trên mọi thiết bị.
5. Kết hợp Auto Layout với Animation và Rotation (xoay ngang/dọc).

---

## 🧠 1. Ôn lại Auto Layout cơ bản

Auto Layout giúp ta mô tả **quan hệ giữa các view** thay vì kích thước tuyệt đối.

Ví dụ:

> “Label này cách cạnh trên 40pt, căn giữa ngang, rộng bằng superview trừ 32pt.”

Không cần tính toán thủ công — iOS sẽ **tự động sắp xếp lại** khi xoay màn hình, đổi thiết bị hoặc font.

---

## ⚙️ 2. Tạo giao diện bằng Auto Layout (không storyboard)

**AutoLayoutDemoViewController.swift**

```swift
import UIKit

final class AutoLayoutDemoViewController: UIViewController {
    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let bioLabel = UILabel()
    private let actionButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Auto Layout Demo"
        view.backgroundColor = .systemBackground
        setupUI()
        setupConstraints()
    }

    private func setupUI() {
        avatarImageView.image = UIImage(systemName: "person.circle.fill")
        avatarImageView.tintColor = .systemBlue
        avatarImageView.translatesAutoresizingMaskIntoConstraints = false
        avatarImageView.contentMode = .scaleAspectFit

        nameLabel.text = "Mai Lê"
        nameLabel.font = .preferredFont(forTextStyle: .title2)
        nameLabel.textAlignment = .center
        nameLabel.translatesAutoresizingMaskIntoConstraints = false

        bioLabel.text = "Nhà phát triển iOS & nhà quản lý giáo dục"
        bioLabel.textAlignment = .center
        bioLabel.numberOfLines = 0
        bioLabel.textColor = .secondaryLabel
        bioLabel.translatesAutoresizingMaskIntoConstraints = false

        actionButton.setTitle("Xem chi tiết", for: .normal)
        actionButton.titleLabel?.font = .boldSystemFont(ofSize: 17)
        actionButton.translatesAutoresizingMaskIntoConstraints = false
        actionButton.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)

        [avatarImageView, nameLabel, bioLabel, actionButton].forEach {
            view.addSubview($0)
        }
    }

    private func setupConstraints() {
        NSLayoutConstraint.activate([
            avatarImageView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            avatarImageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            avatarImageView.widthAnchor.constraint(equalToConstant: 120),
            avatarImageView.heightAnchor.constraint(equalToConstant: 120),

            nameLabel.topAnchor.constraint(equalTo: avatarImageView.bottomAnchor, constant: 16),
            nameLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            nameLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),

            bioLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 8),
            bioLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            bioLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),

            actionButton.topAnchor.constraint(equalTo: bioLabel.bottomAnchor, constant: 24),
            actionButton.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }

    @objc private func buttonTapped() {
        UIView.animate(withDuration: 0.3, animations: {
            self.avatarImageView.transform = CGAffineTransform(scaleX: 1.2, y: 1.2)
        }) { _ in
            UIView.animate(withDuration: 0.3) {
                self.avatarImageView.transform = .identity
            }
        }
    }
}
```

---

## 🧩 3. Dùng UIStackView – “Auto Layout biết đi”

> `UIStackView` giúp nhóm các view và tự động sắp xếp chúng theo chiều **ngang hoặc dọc**
> mà không cần tạo từng constraint thủ công.

```swift
private func setupWithStackView() {
    let stack = UIStackView(arrangedSubviews: [avatarImageView, nameLabel, bioLabel, actionButton])
    stack.axis = .vertical
    stack.spacing = 16
    stack.alignment = .center
    stack.translatesAutoresizingMaskIntoConstraints = false
    view.addSubview(stack)

    NSLayoutConstraint.activate([
        stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
        stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
        stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
    ])

    avatarImageView.widthAnchor.constraint(equalToConstant: 120).isActive = true
    avatarImageView.heightAnchor.constraint(equalToConstant: 120).isActive = true
}
```

💡 Khi em thêm hoặc ẩn một subview:

```swift
bioLabel.isHidden = true
UIView.animate(withDuration: 0.3) {
    stack.layoutIfNeeded() // tự động co lại mượt mà
}
```

---

## ⚙️ 4. Dùng Priority & Compression Resistance

Auto Layout cho phép gán **độ ưu tiên** khi hai constraint mâu thuẫn.
Ví dụ: “Label có thể co lại nhưng không nhỏ hơn 80pt”.

```swift
nameLabel.setContentHuggingPriority(.defaultHigh, for: .vertical)
bioLabel.setContentCompressionResistancePriority(.defaultLow, for: .vertical)
```

| Loại        | Ý nghĩa                    |
| ----------- | -------------------------- |
| Hugging     | “Không muốn giãn ra”       |
| Compression | “Không muốn bị ép nhỏ lại” |

---

## 💡 5. UILayoutGuide – khoảng trống “vô hình”

Dùng khi em muốn **tạo khoảng trống bố cục hợp lý** mà không cần view thật:

```swift
let guide = UILayoutGuide()
view.addLayoutGuide(guide)

NSLayoutConstraint.activate([
    guide.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
    guide.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
    nameLabel.leadingAnchor.constraint(equalTo: guide.leadingAnchor),
    nameLabel.trailingAnchor.constraint(equalTo: guide.trailingAnchor)
])
```

---

## 📱 6. Adaptive UI – tự điều chỉnh khi xoay

Dùng trait collection để nhận biết orientation và thay đổi layout:

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    if traitCollection.verticalSizeClass == .compact {
        print("Đang ở ngang")
    } else {
        print("Đang ở dọc")
    }
}
```

💡 Khi ở ngang: có thể đổi `stack.axis = .horizontal`.

---

## 🧪 7. Kiểm thử thực tế

1️⃣ Chạy app → bố cục căn giữa đẹp.
2️⃣ Xoay ngang → các phần tử vẫn giữ khoảng cách chuẩn.
3️⃣ Ẩn phần mô tả (`bioLabel`) → stack tự co lại mượt mà.
4️⃣ Tăng font chữ trong Accessibility → layout tự giãn đúng.
5️⃣ Giao diện hoạt động tốt trên mọi iPhone, iPad. 🎉

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #19)** 🎒

| Mức độ        | Bài tập                                    | Gợi ý                                  |
| ------------- | ------------------------------------------ | -------------------------------------- |
| 🟢 Cơ bản     | Tạo bố cục với Auto Layout bằng code       | Dùng `NSLayoutConstraint.activate([])` |
| 🟡 Trung bình | Dùng StackView để sắp xếp ảnh + text + nút | `UIStackView(axis:.vertical)`          |
| 🔵 Nâng cao   | Thay đổi layout khi xoay ngang             | `traitCollectionDidChange`             |
| 🟣 Thử thách  | Tạo UI co giãn với Priority khác nhau      | `setContentHuggingPriority()`          |

---

## 📚 Tổng kết

| Thành phần         | Vai trò                                 |
| ------------------ | --------------------------------------- |
| Auto Layout        | Cơ chế định vị linh hoạt cho UI         |
| NSLayoutConstraint | Quy tắc ràng buộc giữa các view         |
| StackView          | Quản lý bố cục động, giảm code          |
| UILayoutGuide      | Khoảng cách “ảo” để bố trí hợp lý       |
| Priority           | Quyết định mức ưu tiên khi co giãn      |
| Adaptive Layout    | Tự đổi bố cục theo thiết bị, hướng xoay |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Tạo bố cục tự động bằng Auto Layout thuần code.
* Dùng StackView để giảm code và tăng linh hoạt.
* Dùng Priority để giải quyết xung đột kích thước.
* Làm giao diện thích ứng mọi màn hình.

---

🎓 **UIKit – Bài 20 (buổi tới):**

> *ScrollView, Paging & Nested Layout – Cuộn, zoom, và hiển thị nhiều màn trong 1 View.*

Thầy sẽ hướng dẫn:

* Làm **scroll tự động & paging** như onboarding app.
* Dùng `UIScrollView` với `UIStackView`.
* Zoom ảnh với gesture.
* Xử lý layout trong ScrollView bằng constraint đúng chuẩn.

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 20: ScrollView & Paging nâng cao (cuộn, zoom, trang lật)** không?
