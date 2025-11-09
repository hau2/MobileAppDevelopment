Tuyệt vời 👏👏👏 — hôm nay ta bước sang **UIKit – Bài 12: Custom UITableViewCell chuyên nghiệp**
Đây là bài cực kỳ quan trọng để giúp app của em “nâng cấp lên cấp độ App Store”:

> Danh sách học sinh sẽ hiển thị **đẹp, gọn, có ảnh đại diện, font tự động, hỗ trợ Dark Mode và Auto Layout hoàn chỉnh.**

Thầy sẽ hướng dẫn **bài bản từ nền tảng đến chuyên sâu**, kèm ví dụ hoàn chỉnh có thể chạy ngay.

---

# 🧩 UIKit – Bài 12: Custom UITableViewCell chuyên nghiệp

---

## 🎯 Mục tiêu bài học

Sau buổi này, em sẽ biết:

1. Tạo custom cell bằng code (không storyboard).
2. Dùng Auto Layout trong cell cho bố cục linh hoạt.
3. Hiển thị ảnh, tên, điểm, email gọn đẹp.
4. Hỗ trợ **Dark Mode, Dynamic Type, và reuse hiệu quả.**
5. Hiểu cơ chế **prepareForReuse()** để tối ưu hiệu năng.

---

## 🧠 1. Ôn lại UITableView cơ bản

Mỗi hàng (`UITableViewCell`) là 1 “component” hiển thị dữ liệu.
Trước đây ta dùng cell mặc định (`UITableViewCell`), giờ ta sẽ **tạo class riêng kế thừa** để tự bố trí nội dung.

---

## 🧩 2. Tạo Custom Cell Class

**StudentCell.swift**

```swift
import UIKit

final class StudentCell: UITableViewCell {
    static let reuseID = "StudentCell"

    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let gradeLabel = UILabel()
    private let emailLabel = UILabel()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private func setupUI() {
        // Avatar
        avatarImageView.translatesAutoresizingMaskIntoConstraints = false
        avatarImageView.layer.cornerRadius = 25
        avatarImageView.clipsToBounds = true
        avatarImageView.contentMode = .scaleAspectFill
        avatarImageView.backgroundColor = .systemGray5

        // Name
        nameLabel.font = .preferredFont(forTextStyle: .headline)
        nameLabel.adjustsFontForContentSizeCategory = true

        // Grade
        gradeLabel.font = .preferredFont(forTextStyle: .subheadline)
        gradeLabel.textColor = .secondaryLabel
        gradeLabel.adjustsFontForContentSizeCategory = true

        // Email
        emailLabel.font = .preferredFont(forTextStyle: .footnote)
        emailLabel.textColor = .tertiaryLabel
        emailLabel.numberOfLines = 1
        emailLabel.adjustsFontForContentSizeCategory = true

        // Stack texts
        let textStack = UIStackView(arrangedSubviews: [nameLabel, gradeLabel, emailLabel])
        textStack.axis = .vertical
        textStack.spacing = 4
        textStack.translatesAutoresizingMaskIntoConstraints = false

        contentView.addSubview(avatarImageView)
        contentView.addSubview(textStack)

        NSLayoutConstraint.activate([
            avatarImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            avatarImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            avatarImageView.widthAnchor.constraint(equalToConstant: 50),
            avatarImageView.heightAnchor.constraint(equalToConstant: 50),

            textStack.leadingAnchor.constraint(equalTo: avatarImageView.trailingAnchor, constant: 12),
            textStack.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            textStack.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 10),
            textStack.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -10)
        ])
    }

    func configure(with student: StudentEntity) {
        nameLabel.text = student.name
        gradeLabel.text = String(format: "Điểm trung bình: %.1f", student.grade)
        emailLabel.text = student.email

        // Ảnh đại diện tạm thời
        let symbolName = student.grade >= 8 ? "star.circle.fill"
                        : student.grade >= 5 ? "person.circle"
                        : "exclamationmark.circle"
        avatarImageView.image = UIImage(systemName: symbolName)
        avatarImageView.tintColor = student.grade >= 8 ? .systemYellow :
                                    student.grade >= 5 ? .systemBlue :
                                    .systemRed
    }

    override func prepareForReuse() {
        super.prepareForReuse()
        avatarImageView.image = nil
        nameLabel.text = nil
        gradeLabel.text = nil
        emailLabel.text = nil
    }
}
```

---

## ⚙️ 3. Sử dụng Custom Cell trong TableView

**StudentListViewController.swift**

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    title = "Danh sách học sinh"
    view.backgroundColor = .systemBackground
    setupTable()
    setupFetchedResultsController()
}

private func setupTable() {
    tableView.translatesAutoresizingMaskIntoConstraints = false
    view.addSubview(tableView)

    NSLayoutConstraint.activate([
        tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
        tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
        tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
    ])

    tableView.register(StudentCell.self, forCellReuseIdentifier: StudentCell.reuseID)
    tableView.dataSource = self
    tableView.delegate = self
    tableView.rowHeight = 80
}
```

---

## 🧩 4. Nguồn dữ liệu (Data Source)

```swift
extension StudentListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        fetchedResultsController.fetchedObjects?.count ?? 0
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: StudentCell.reuseID,
            for: indexPath
        ) as? StudentCell else { return UITableViewCell() }

        let student = fetchedResultsController.object(at: indexPath)
        cell.configure(with: student)
        return cell
    }
}
```

---

## 💡 5. Giao diện & hiệu ứng chuyên nghiệp

### 🔹 Tùy chỉnh màu khi chọn:

```swift
cell.selectionStyle = .none
```

### 🔹 Highlight khi chạm:

```swift
override func setHighlighted(_ highlighted: Bool, animated: Bool) {
    super.setHighlighted(highlighted, animated: animated)
    contentView.backgroundColor = highlighted ? UIColor.systemGray5 : .clear
}
```

### 🔹 Auto Layout thích ứng:

* Cell tự giãn khi font chữ lớn hơn (Dynamic Type).
* Dùng `adjustsFontForContentSizeCategory = true`.
* Không fix cứng chiều cao ảnh, chỉ nên giới hạn tối thiểu (50–60px).

---

## ⚡ 6. Dark Mode & Dynamic Type

UIKit tự hỗ trợ nếu ta dùng:

* `systemBackground`, `label`, `secondaryLabel`
* `preferredFont(forTextStyle:)`

💡 Nghĩa là cell sẽ **tự đổi màu và font phù hợp** khi người dùng chuyển chế độ tối hoặc phóng to chữ.

---

## 🧱 7. Cấu trúc UI chuyên nghiệp

**Bố cục của 1 cell:**

```
┌─────────────────────────────────────────────┐
│ 🟡 avatar |  Mai Lê              (headline) │
│           |  Điểm: 8.5           (subheadline) │
│           |  mai@gmail.com       (footnote) │
└─────────────────────────────────────────────┘
```

Toàn bộ text nằm trong `UIStackView`, dễ mở rộng thêm label khác sau này (ví dụ “Lớp”, “Ngày cập nhật”, ...).

---

## 🧩 8. Tối ưu hiệu năng Cell (rất quan trọng)

| Kỹ thuật                   | Giải thích                          |
| -------------------------- | ----------------------------------- |
| `reuseIdentifier`          | tái sử dụng cell thay vì tạo mới    |
| `prepareForReuse()`        | reset lại nội dung cũ               |
| `clipsToBounds`            | giới hạn vẽ trong avatar            |
| `system symbol`            | icon nhẹ, không cần ảnh thật        |
| `backgroundColor = .clear` | để system handle màu nền theo theme |

---

## 🧪 9. Thực hành kiểm thử

1️⃣ Mở app -> danh sách học sinh hiển thị có ảnh biểu tượng.
2️⃣ Chuyển sang Dark Mode -> giao diện đổi màu tự động.
3️⃣ Vào Settings -> Accessibility -> tăng cỡ chữ -> font tự giãn.
4️⃣ Thêm, sửa, xoá học sinh -> cell cập nhật mượt mà.
5️⃣ Điểm lớn hơn hoặc bằng 8 => icon vàng, 5 tới 7 => xanh, dưới 5 => đỏ. 🎉

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #12)** 🎒

| Mức độ        | Bài tập                                             | Gợi ý                                        |
| ------------- | --------------------------------------------------- | -------------------------------------------- |
| 🟢 Cơ bản     | Tạo cell có avatar + tên + điểm                     | Dùng `UIImageView`, `UILabel`, Auto Layout   |
| 🟡 Trung bình | Thêm email + dùng Dynamic Type                      | `preferredFont(forTextStyle:)`               |
| 🔵 Nâng cao   | Đổi màu icon theo điểm                              | `UIImage(systemName:)` + `.tintColor`        |
| 🟣 Thử thách  | Thêm “Ngày cập nhật gần nhất” & định dạng thời gian | `DateFormatter` + thêm attribute `updatedAt` |

---

## 📚 Tổng kết

| Thành phần      | Vai trò                           |
| --------------- | --------------------------------- |
| Custom Cell     | Hiển thị nội dung linh hoạt       |
| Auto Layout     | Bố cục đẹp, dễ mở rộng            |
| Dynamic Type    | Phù hợp mọi cỡ chữ                |
| Dark Mode       | Tự đổi màu theo hệ thống          |
| prepareForReuse | Dọn nội dung cell tái sử dụng     |
| SF Symbols      | Icon vector nhẹ, đẹp, đồng bộ iOS |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Tạo custom cell bằng code.
* Dùng Auto Layout cho bố cục đẹp.
* Tối ưu cell theo chuẩn Apple.
* Hỗ trợ Dark Mode và Dynamic Type.

---

🎓 **UIKit – Bài 13 (buổi tới):**

> *Navigation nâng cao: Push, Modal, Sheet Presentation, và truyền dữ liệu chi tiết học sinh.*

Em sẽ học:

* Khi chọn 1 học sinh -> mở màn chi tiết bằng push hoặc bottom sheet.
* Hiển thị ảnh lớn, email, điểm, và thêm nút “Sửa” ngay trong sheet.
* Phân biệt các kiểu chuyển màn hình: **push, modal, pageSheet, formSheet.**

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 13: Navigation nâng cao & màn chi tiết học sinh (Detail Screen)** không?
