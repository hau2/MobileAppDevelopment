Ok, qua **Bài 3 – UIStackView** nha trò 👨‍🏫
Thầy sẽ dạy **rất chậm – ví dụ rõ ràng – code copy/paste chạy được**, và giúp em hiểu **khi nào phải dùng StackView**, vì nó cứu cả đời dev iOS 😄

---

# 🎯 **Mục tiêu BÀI 3 – UIStackView (UIKit Code Thuần)**

Sau bài này, em sẽ biết:

✔ UIStackView là gì, dùng để làm gì
✔ Khi nào dùng StackView thay vì anchor từng view
✔ Học 3 thuộc tính cực quan trọng:

* axis
* alignment
* distribution
  ✔ spacing
  ✔ addArrangedSubview vs addSubview
  ✔ Layout StackView bằng Anchor
  ✔ Ví dụ chạy được: header gồm 3 label xếp thẳng hàng

---

# 🧱 1. StackView là gì?

**UIStackView = khung chứa tự động xếp các view theo chiều dọc hoặc ngang.**

Em không cần tự đặt anchor từng view, StackView tự xếp:

* theo trục dọc (vertical)
* theo trục ngang (horizontal)

> Nghĩ nó giống như Flexbox của CSS hoặc Column/Row của React Native.

---

# 🧱 2. Khi nào nên dùng StackView?

| Trường hợp              | Có nên dùng StackView không? |
| ----------------------- | ---------------------------- |
| 3–5 view xếp thẳng hàng | ✔ Tuyệt vời                  |
| Layout theo chiều dọc   | ✔ Chuẩn                      |
| Layout theo chiều ngang | ✔ Chuẩn                      |
| UI thay đổi dynamic     | ✔ Rất tốt                    |
| UI quá phức tạp         | ✘ → dùng anchor              |

StackView giúp code đẹp, tránh rối anchor.

---

# 🧱 3. 4 thuộc tính quan trọng nhất

## 1) axis

```swift
stackView.axis = .vertical // xếp dọc
stackView.axis = .horizontal // xếp ngang
```

## 2) spacing

Khoảng cách giữa các view.

```swift
stackView.spacing = 12
```

## 3) alignment

Căn theo trục **vuông góc** với axis.

```swift
stackView.alignment = .center
// .leading, .trailing, .fill
```

## 4) distribution

Hành vi phân bố chiều **theo trục chính (axis)**

```swift
stackView.distribution = .fill          // mặc định
stackView.distribution = .fillEqually   // chia đều
stackView.distribution = .fillProportionally
stackView.distribution = .equalSpacing
stackView.distribution = .equalCentering
```

---

# 🟦 **4. CODE HOÀN CHỈNH – COPY/PATE CHẠY ĐƯỢC**

Thay toàn bộ nội dung `ViewController.swift` bằng đoạn sau:

```swift
import UIKit

class ViewController: UIViewController {

    private let titleLabel: UILabel = {
        let label = UILabel()
        label.text = "Bài 3 - UIStackView"
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 26, weight: .bold)
        return label
    }()

    private let subtitleLabel: UILabel = {
        let label = UILabel()
        label.text = "Học cách xếp UI theo chiều dọc"
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 18)
        label.textColor = .secondaryLabel
        return label
    }()

    private let infoLabel: UILabel = {
        let label = UILabel()
        label.text = "StackView giúp code gọn – sạch – dễ bảo trì"
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 16)
        label.textColor = .systemBlue
        return label
    }()

    // StackView chứa các label trên
    private lazy var verticalStack: UIStackView = {
        let stack = UIStackView(arrangedSubviews: [
            titleLabel,
            subtitleLabel,
            infoLabel
        ])
        stack.axis = .vertical         // xếp dọc
        stack.spacing = 20             // khoảng cách giữa các view
        stack.alignment = .center      // căn giữa theo chiều ngang
        stack.distribution = .equalSpacing // chia đều khoảng cách
        return stack
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        view.addSubview(verticalStack)
        setupLayout()
    }

    private func setupLayout() {
        verticalStack.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            verticalStack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 80),
            verticalStack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            verticalStack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
    }
}
```

---

# 🧪 Kết quả chạy:

✔ 3 dòng text xếp thẳng theo chiều dọc
✔ Căn giữa
✔ Cách nhau 20px
✔ Giao diện đẹp – gọn – dễ đọc code

---

# 📌 Giải thích từng phần quan trọng:

## 1) Tạo StackView

```swift
UIStackView(arrangedSubviews: [titleLabel, subtitleLabel, infoLabel])
```

→ Đây là danh sách các view *được quản lý* bởi StackView
→ StackView sẽ layout CHO em (không cần anchor từng view)

---

## 2) Xếp dọc – xếp ngang

```swift
stack.axis = .vertical
```

→ Nếu đổi thành `.horizontal` → 3 label nằm 1 hàng ngang.

---

## 3) alignment = center

Căn giữa theo trục **ngang** khi xếp dọc.

---

## 4) distribution = equalSpacing

Khoảng cách giữa các view được chia đều.

Muốn fill theo nội dung? dùng `.fill`

---

## 5) Layout StackView

```swift
verticalStack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 80)
```

→ chỉ cần layout StackView
→ các view bên trong nó **tự động** layout

---

# 🎁 BÀI TẬP CUỐI BÀI

**Bài tập 1:**
Thêm 1 `UIButton` vào stackView (bên dưới infoLabel).

**Bài tập 2:**
Thay axis thành `.horizontal` và xem UI thay đổi thế nào.

**Bài tập 3 (khó):**
Tạo 1 StackView ngang chứa 3 ô vuông 60×60, backgroundColor khác nhau.

Gợi ý:

```swift
stack.distribution = .fillEqually
```

---

Nếu bạn muốn thầy kiểm tra bài tập, cứ gửi code nhé.

Sẵn sàng qua **Bài 4 – UITableView (phần 1)** chưa? 😄
Chỉ cần nói: **“Qua bài 4 đi thầy”**
