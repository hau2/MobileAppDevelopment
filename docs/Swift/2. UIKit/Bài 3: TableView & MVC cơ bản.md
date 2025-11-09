Tuyệt vời lớp 👏 — hôm nay thầy dạy **chậm – kỹ – từng bước** về **UITableView & mô hình MVC cơ bản** (100% code, không storyboard). Cứ theo thứ tự dưới đây, copy từng đoạn chạy thử là được.

---

# 🧩 UIKit – Bài 3: UITableView & MVC (Programmatic UI)

## 🎯 Mục tiêu

* Hiểu **MVC** trong UIKit ở mức thực dụng.
* Tạo **danh sách (UITableView)** chỉ bằng code.
* Viết **cell tùy chỉnh** bằng Auto Layout.
* Bắt sự kiện chọn dòng → **đi màn chi tiết** (push).
* Nắm các điểm hay lỗi: đăng ký cell, reuse id, chiều cao tự động.

---

## 0) Chuẩn bị: bọc app trong `UINavigationController`

Để push sang màn chi tiết, ta cần Navigation Controller.

**SceneDelegate.swift**

```swift
func scene(_ scene: UIScene,
           willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }

    let window = UIWindow(windowScene: windowScene)
    let root = StudentListViewController()         // màn danh sách
    let nav = UINavigationController(rootViewController: root)
    window.rootViewController = nav
    self.window = window
    window.makeKeyAndVisible()
}
```

---

## 1) MVC “đủ xài” cho TableView

* **Model**: dữ liệu thuần (struct `Student`).
* **View**: `StudentCell` (kế thừa `UITableViewCell`), chỉ lo layout.
* **Controller**: `StudentListViewController` quản lý table, nguồn dữ liệu, điều hướng.

Sơ đồ:

```
Student (Model)  ←→  StudentListViewController (Controller)  ←→  StudentCell (View)
```

---

## 2) Tạo Model

**Student.swift**

```swift
import Foundation

struct Student {
    let id: UUID = .init()
    let name: String
    let grade: Double
    let city: String
}
```

---

## 3) Tạo Cell tùy chỉnh (View)

**StudentCell.swift**

```swift
import UIKit

final class StudentCell: UITableViewCell {
    static let reuseID = "StudentCell"

    private let nameLabel = UILabel()
    private let gradeLabel = UILabel()
    private let cityLabel = UILabel()
    private let container = UIStackView()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private func setupUI() {
        // Cấu hình nhãn
        nameLabel.font = .systemFont(ofSize: 17, weight: .semibold)
        gradeLabel.font = .systemFont(ofSize: 15)
        gradeLabel.textColor = .secondaryLabel
        cityLabel.font = .systemFont(ofSize: 15)
        cityLabel.textColor = .tertiaryLabel
        nameLabel.numberOfLines = 0
        cityLabel.numberOfLines = 0

        // Dùng StackView dọc
        container.axis = .vertical
        container.spacing = 4
        container.translatesAutoresizingMaskIntoConstraints = false

        container.addArrangedSubview(nameLabel)
        container.addArrangedSubview(gradeLabel)
        container.addArrangedSubview(cityLabel)

        contentView.addSubview(container)

        // Auto Layout
        NSLayoutConstraint.activate([
            container.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 12),
            container.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            container.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            container.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -12)
        ])

        // Cho phép cell tự co chiều cao theo nội dung
        nameLabel.setContentCompressionResistancePriority(.required, for: .vertical)
        cityLabel.setContentCompressionResistancePriority(.defaultLow, for: .vertical)
    }

    func configure(with student: Student) {
        nameLabel.text = student.name
        gradeLabel.text = String(format: "Điểm TB: %.2f", student.grade)
        cityLabel.text  = "Khu vực: " + student.city
        accessoryType = .disclosureIndicator
    }
}
```

**Ghi nhớ quan trọng**

* `static let reuseID` để đăng ký/nhận cell.
* Đặt constraints đủ bốn phía cho stack → **tự tính chiều cao**.
* Không set `translatesAutoresizingMaskIntoConstraints` cho label (vì đã nằm trong stack).

---

## 4) Tạo màn danh sách (Controller)

**StudentListViewController.swift**

```swift
import UIKit

final class StudentListViewController: UIViewController {
    private let tableView = UITableView(frame: .zero, style: .insetGrouped)

    // Data mẫu
    private var students: [Student] = [
        .init(name: "Mai Lê", grade: 8.7, city: "Hà Nội"),
        .init(name: "Thành Công", grade: 9.2, city: "Hải Phòng"),
        .init(name: "Hoàng Cường", grade: 7.8, city: "Đà Nẵng"),
        .init(name: "Ngọc Lan", grade: 8.0, city: "TP.HCM")
    ]

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Danh sách học sinh"
        view.backgroundColor = .systemBackground
        setupTable()
        setupRefreshControl()
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

        // Đăng ký cell custom
        tableView.register(StudentCell.self, forCellReuseIdentifier: StudentCell.reuseID)

        // Cấu hình cho tự động tính chiều cao
        tableView.estimatedRowHeight = 64
        tableView.rowHeight = UITableView.automaticDimension

        tableView.dataSource = self
        tableView.delegate   = self
    }

    private func setupRefreshControl() {
        let rc = UIRefreshControl()
        rc.addTarget(self, action: #selector(didPullToRefresh), for: .valueChanged)
        tableView.refreshControl = rc
    }

    @objc private func didPullToRefresh() {
        // Demo: xáo trộn danh sách rồi reload
        students.shuffle()
        tableView.reloadData()
        tableView.refreshControl?.endRefreshing()
    }
}

// MARK: - UITableViewDataSource
extension StudentListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        students.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: StudentCell.reuseID,
            for: indexPath
        ) as? StudentCell else { return UITableViewCell() }

        cell.configure(with: students[indexPath.row])
        return cell
    }
}

// MARK: - UITableViewDelegate
extension StudentListViewController: UITableViewDelegate {
    // Chọn dòng → push chi tiết
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let student = students[indexPath.row]
        let detail = StudentDetailViewController(student: student)
        navigationController?.pushViewController(detail, animated: true)
        tableView.deselectRow(at: indexPath, animated: true)
    }
}
```

**Điểm mấu chốt**

* **Đăng ký cell** trước khi `dequeue`.
* Dùng `automaticDimension` để cell co giãn theo nội dung.
* `didSelectRowAt` để điều hướng.

---

## 5) Màn chi tiết (push)

**StudentDetailViewController.swift**

```swift
import UIKit

final class StudentDetailViewController: UIViewController {
    private let student: Student

    init(student: Student) {
        self.student = student
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private let stack = UIStackView()
    private let nameLabel = UILabel()
    private let gradeLabel = UILabel()
    private let cityLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Chi tiết"
        view.backgroundColor = .systemBackground
        setupUI()
        fillData()
    }

    private func setupUI() {
        stack.axis = .vertical
        stack.spacing = 12
        stack.translatesAutoresizingMaskIntoConstraints = false

        nameLabel.font = .systemFont(ofSize: 22, weight: .bold)
        gradeLabel.textColor = .secondaryLabel

        stack.addArrangedSubview(nameLabel)
        stack.addArrangedSubview(gradeLabel)
        stack.addArrangedSubview(cityLabel)

        view.addSubview(stack)
        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 24),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
    }

    private func fillData() {
        nameLabel.text  = student.name
        gradeLabel.text = String(format: "Điểm trung bình: %.2f", student.grade)
        cityLabel.text  = "Khu vực: \(student.city)"
    }
}
```

---

## 6) Kiểm tra nhanh các lỗi thường gặp

* ❗ Quên `register(_:forCellReuseIdentifier:)` → crash khi dequeue.
* ❗ Thiếu constraints 4 phía → cell **không** tự căn chiều cao.
* ❗ Đặt `rowHeight` cố định → chữ dài bị cắt; nên dùng `automaticDimension`.
* ❗ Không bọc trong `UINavigationController` → **push không chạy**.

---

## 7) Nâng cấp dần (tuỳ chọn sau khi ổn)

* **Diffable Data Source** thay cho `.reloadData()` (hiệu năng + animation).
* **Swipe Actions** (xoá/sửa) với `tableView(_:trailingSwipeActionsConfigurationForRowAt:)`.
* **Search**: thêm `UISearchController` lọc dữ liệu.
* **Pagination**: load thêm khi gần cuối danh sách.

---

## 🧪 Bài tập trên lớp (làm ngay)

1. Đổi `style` table từ `.insetGrouped` → `.plain` và quan sát khác nhau.
2. Thêm 1 thuộc tính mới trong `Student` (vd: `email`) → hiện ở cell.
3. Bấm “kéo xuống để làm mới” (pull-to-refresh) sẽ **đảo ngược danh sách** thay vì shuffle.

---

## 🏠 BÀI TẬP VỀ NHÀ (UIKit #3) 🎒

| Mức độ        | Bài tập                                                                                                              | Gợi ý                                                                            |
| ------------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 🟢 Cơ bản     | Thêm icon hệ thống (SF Symbol) bên trái tên trong cell                                                               | `UIImageView` + `symbolConfiguration`                                            |
| 🟡 Trung bình | Thêm nút “+” (barButtonItem) để **thêm học sinh** mới (tên giả)                                                      | `navigationItem.rightBarButtonItem`                                              |
| 🔵 Nâng cao   | Thêm chức năng **xoá hàng** bằng swipe                                                                               | `tableView(_:commit:forRowAt:)` hoặc `trailingSwipeActionsConfigurationForRowAt` |
| 🟣 Thử thách  | Tạo **màn chỉnh sửa** điểm học sinh: chọn dòng → màn detail có nút “Sửa điểm”, nhập số mới, cập nhật và `reloadRows` | Dùng `UIAlertController` dạng `.alert` có `textField`                            |

---

## 📚 Tổng kết

* Hiểu MVC ở mức “vừa đủ để làm việc”.
* Tạo danh sách với `UITableView` hoàn toàn bằng code.
* Viết cell custom, Auto Layout chuẩn, chiều cao tự động.
* Điều hướng push sang màn chi tiết.

---

## ⏭️ Bài 4 (tiết sau)

**UICollectionView cơ bản → nâng cao** (Programmatic):

* Layout lưới, cell custom, header, spacing, compositional layout “đẹp chuẩn Apple”.

Khi em xong bài tập #3, nói thầy: mình chuyển qua **UIKit – Bài 4: CollectionView** nhé!
