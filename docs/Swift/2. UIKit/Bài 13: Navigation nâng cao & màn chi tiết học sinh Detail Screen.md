Tốt lắm 👏👏👏 — hôm nay thầy và em sang **UIKit – Bài 13: Navigation nâng cao & Màn chi tiết học sinh (Detail Screen)**.
Đây là **bước cực kỳ quan trọng** giúp ứng dụng của em trở nên chuyên nghiệp hơn, vì từ nay app sẽ có **màn hình chi tiết, sửa thông tin, mở bằng push hoặc sheet, truyền dữ liệu giữa ViewController đúng chuẩn Apple.**

---

# 🧩 UIKit – Bài 13: Navigation nâng cao & Detail Screen

---

## 🎯 Mục tiêu bài học

Sau buổi này, em sẽ biết:

1. Hiểu và sử dụng các kiểu **chuyển màn hình**: Push, Modal, Sheet.
2. Truyền dữ liệu giữa các ViewController (ví dụ: từ danh sách → chi tiết).
3. Hiển thị chi tiết học sinh trong màn riêng (avatar, điểm, email, ngày cập nhật).
4. Cho phép chỉnh sửa và lưu ngược về danh sách bằng Core Data.
5. Hiểu quy trình “Data Flow” trong app iOS.

---

## 🧠 1. Ôn lại Navigation cơ bản

UIKit có hai cách chính để chuyển màn hình:

| Cách                | Mô tả                           | Ví dụ                                           |
| ------------------- | ------------------------------- | ----------------------------------------------- |
| **Push Navigation** | Đưa màn hình mới vào *ngăn xếp* | `navigationController?.pushViewController(...)` |
| **Modal / Sheet**   | Mở màn hình nổi lên (trang phủ) | `present(vc, animated: true)`                   |

💡 Ta thường dùng **push** cho “mở chi tiết”,
và **sheet** cho “sửa nhanh hoặc xác nhận”.

---

## ⚙️ 2. Màn hình danh sách (StudentListViewController)

Ta đã có sẵn danh sách với custom cell.
Giờ ta thêm hàm khi chọn học sinh:

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let student = fetchedResultsController.object(at: indexPath)
    let detailVC = StudentDetailViewController(student: student)
    navigationController?.pushViewController(detailVC, animated: true)
    tableView.deselectRow(at: indexPath, animated: true)
}
```

---

## 🧩 3. Tạo màn chi tiết

**StudentDetailViewController.swift**

```swift
import UIKit
import CoreData

final class StudentDetailViewController: UIViewController {
    private var student: StudentEntity

    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let gradeLabel = UILabel()
    private let emailLabel = UILabel()
    private let updateButton = UIButton(type: .system)
    private let lastUpdatedLabel = UILabel()

    init(student: StudentEntity) {
        self.student = student
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Chi tiết học sinh"
        view.backgroundColor = .systemBackground
        setupUI()
        displayData()
    }

    private func setupUI() {
        avatarImageView.translatesAutoresizingMaskIntoConstraints = false
        avatarImageView.layer.cornerRadius = 50
        avatarImageView.clipsToBounds = true
        avatarImageView.contentMode = .scaleAspectFill
        avatarImageView.tintColor = .systemBlue

        [nameLabel, gradeLabel, emailLabel, lastUpdatedLabel].forEach {
            $0.textAlignment = .center
            $0.adjustsFontForContentSizeCategory = true
            $0.numberOfLines = 1
        }

        nameLabel.font = .preferredFont(forTextStyle: .title2)
        gradeLabel.font = .preferredFont(forTextStyle: .headline)
        gradeLabel.textColor = .secondaryLabel
        emailLabel.font = .preferredFont(forTextStyle: .subheadline)
        emailLabel.textColor = .tertiaryLabel
        lastUpdatedLabel.font = .preferredFont(forTextStyle: .footnote)
        lastUpdatedLabel.textColor = .secondaryLabel

        updateButton.setTitle("Sửa thông tin", for: .normal)
        updateButton.titleLabel?.font = .boldSystemFont(ofSize: 16)
        updateButton.addTarget(self, action: #selector(editTapped), for: .touchUpInside)

        let stack = UIStackView(arrangedSubviews: [
            avatarImageView,
            nameLabel,
            gradeLabel,
            emailLabel,
            lastUpdatedLabel,
            updateButton
        ])
        stack.axis = .vertical
        stack.spacing = 12
        stack.alignment = .center
        stack.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(stack)

        NSLayoutConstraint.activate([
            avatarImageView.widthAnchor.constraint(equalToConstant: 100),
            avatarImageView.heightAnchor.constraint(equalToConstant: 100),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])
    }

    private func displayData() {
        nameLabel.text = student.name
        gradeLabel.text = String(format: "Điểm trung bình: %.1f", student.grade)
        emailLabel.text = student.email

        let symbolName = student.grade >= 8 ? "star.circle.fill"
                        : student.grade >= 5 ? "person.circle"
                        : "exclamationmark.circle"
        avatarImageView.image = UIImage(systemName: symbolName)
        avatarImageView.tintColor = student.grade >= 8 ? .systemYellow :
                                    student.grade >= 5 ? .systemBlue :
                                    .systemRed

        if let updatedAt = student.value(forKey: "updatedAt") as? Date {
            let formatter = DateFormatter()
            formatter.dateFormat = "dd/MM/yyyy HH:mm"
            lastUpdatedLabel.text = "Cập nhật lần cuối: \(formatter.string(from: updatedAt))"
        } else {
            lastUpdatedLabel.text = "Chưa có dữ liệu cập nhật"
        }
    }

    @objc private func editTapped() {
        let editVC = RegisterViewController(student: student)
        let nav = UINavigationController(rootViewController: editVC)
        editVC.onStudentSaved = { [weak self] in
            self?.displayData()
        }
        if let sheet = nav.sheetPresentationController {
            sheet.detents = [.medium(), .large()]
            sheet.prefersGrabberVisible = true
        }
        present(nav, animated: true)
    }
}
```

---

## ⚙️ 4. Giải thích

| Thành phần                           | Vai trò                                     |
| ------------------------------------ | ------------------------------------------- |
| `pushViewController`                 | Mở màn chi tiết từ danh sách                |
| `present` + `UINavigationController` | Mở form sửa bằng sheet                      |
| `onStudentSaved` closure             | Callback để cập nhật lại màn chi tiết       |
| `sheet.detents`                      | Kiểu hiển thị `.medium`, `.large` (iOS 15+) |
| `DateFormatter`                      | Hiển thị ngày cập nhật đẹp mắt              |

---

## 💡 5. Sơ đồ luồng dữ liệu (Data Flow)

```
StudentListViewController
       │
   [Chọn hàng]
       ↓
StudentDetailViewController
       │
   [Bấm "Sửa"]
       ↓
RegisterViewController (Sheet)
       │
   [Nhập + Lưu]
       ↓
Core Data update
       ↓
StudentDetailViewController refresh
       ↓
FRC tự cập nhật danh sách
```

✅ Mọi thứ **đồng bộ, realtime, chuẩn Apple**.

---

## 🧪 6. Kiểm thử

1️⃣ Chạy app → chọn 1 học sinh.
2️⃣ Mở chi tiết → thấy avatar, email, điểm, thời gian cập nhật.
3️⃣ Bấm “Sửa thông tin” → form mở dưới dạng **bottom sheet**.
4️⃣ Sửa xong → sheet đóng, dữ liệu chi tiết cập nhật.
5️⃣ Quay lại danh sách → cell tự thay đổi (nhờ FRC). 🎉

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #13)** 🎒

| Mức độ        | Bài tập                                 | Gợi ý                                        |
| ------------- | --------------------------------------- | -------------------------------------------- |
| 🟢 Cơ bản     | Mở chi tiết bằng push                   | `navigationController?.pushViewController()` |
| 🟡 Trung bình | Thêm nút “Sửa” mở bằng sheet            | `present(nav, animated: true)`               |
| 🔵 Nâng cao   | Hiển thị ngày cập nhật cuối             | `DateFormatter`                              |
| 🟣 Thử thách  | Thêm avatar thật (chọn ảnh từ thư viện) | `UIImagePickerController`                    |

---

## 📚 Tổng kết

| Thành phần           | Vai trò                            |
| -------------------- | ---------------------------------- |
| NavigationController | Quản lý ngăn xếp màn hình          |
| Push                 | Dùng cho đi sâu vào chi tiết       |
| Modal / Sheet        | Dùng cho tác vụ phụ hoặc chỉnh sửa |
| Closure              | Truyền dữ liệu ngược lại           |
| Core Data            | Nguồn dữ liệu chính                |
| DateFormatter        | Hiển thị ngày đẹp mắt              |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Dùng push để mở màn chi tiết.
* Dùng sheet để mở form sửa.
* Truyền dữ liệu 2 chiều giữa ViewController.
* Cập nhật UI tự động nhờ Core Data + Closure.

---

🎓 **UIKit – Bài 14 (buổi tới):**

> *TableView nâng cao: Swipe hành động tùy chỉnh (Delete, Edit, Share), Context Menu, và haptic feedback.*

Bài này giúp app của em:

* Có hành động trượt (xoá, sửa, chia sẻ).
* Có menu khi nhấn giữ (context menu).
* Có phản hồi rung nhẹ (haptic) chuyên nghiệp.

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 14: TableView nâng cao (Swipe Actions + Context Menu + Haptic Feedback)** không?
