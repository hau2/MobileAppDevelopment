Tốt lắm 👏👏👏 — chào mừng em chính thức bước vào **Tuần 1 – Ngày 1: Ôn Swift + UIKit Core và dựng mini app “Danh sách học sinh” (theo mô hình MVC)**.
Đây là bài mở đầu của **chương trình thực chiến UIKit**, và cũng là nền tảng cho các dự án sau (MVVM, API, CoreData…).

> Hôm nay ta sẽ làm một **ứng dụng hoàn chỉnh từ đầu đến cuối**, nhưng vẫn nhỏ gọn:
> hiển thị danh sách học sinh → chạm để xem chi tiết → thêm học sinh mới.

---

# 🧩 UIKit – Tuần 1, Ngày 1: Mini Project “Danh sách học sinh” (MVC)

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ nắm chắc:

1. Cấu trúc thư mục dự án chuẩn UIKit (Model – View – Controller).
2. Cách tạo TableView bằng code (không storyboard).
3. Cách truyền dữ liệu giữa các màn hình.
4. Cách thêm đối tượng mới bằng delegate / closure.

---

## 🧱 1. Khởi tạo dự án

1️⃣ Mở Xcode → chọn **“App” → UIKit App (Storyboard unchecked)**
2️⃣ Đặt tên:

```
StudentListApp
```

3️⃣ Interface: **Storyboard = None**
4️⃣ Language: **Swift**
5️⃣ Target: iPhone

---

## 📂 2. Tạo cấu trúc thư mục dự án chuẩn MVC

Cấu trúc trong Xcode:

```
StudentListApp
├── Model
│   └── Student.swift
├── View
│   └── StudentCell.swift
├── Controller
│   ├── StudentListViewController.swift
│   ├── StudentDetailViewController.swift
│   └── AddStudentViewController.swift
└── AppDelegate.swift / SceneDelegate.swift
```

---

## 👨‍🎓 3. Model: `Student.swift`

```swift
import Foundation

struct Student {
    var id: UUID
    var name: String
    var age: Int
    var grade: Double
}
```

💡 Đây là **một struct đơn giản** đại diện cho mỗi học sinh.

---

## 📋 4. View: `StudentCell.swift`

```swift
import UIKit

final class StudentCell: UITableViewCell {
    static let reuseID = "StudentCell"

    private let nameLabel = UILabel()
    private let gradeLabel = UILabel()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }

    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private func setupUI() {
        nameLabel.font = .systemFont(ofSize: 17, weight: .medium)
        gradeLabel.textColor = .secondaryLabel
        gradeLabel.font = .systemFont(ofSize: 15)
        nameLabel.translatesAutoresizingMaskIntoConstraints = false
        gradeLabel.translatesAutoresizingMaskIntoConstraints = false

        contentView.addSubview(nameLabel)
        contentView.addSubview(gradeLabel)

        NSLayoutConstraint.activate([
            nameLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            nameLabel.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),

            gradeLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            gradeLabel.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }

    func configure(with student: Student) {
        nameLabel.text = student.name
        gradeLabel.text = "Điểm: \(student.grade)"
    }
}
```

---

## 📱 5. Controller: `StudentListViewController.swift`

```swift
import UIKit

final class StudentListViewController: UIViewController {
    private var tableView: UITableView!
    private var students: [Student] = [
        Student(id: UUID(), name: "Mai Lê", age: 15, grade: 8.9),
        Student(id: UUID(), name: "Thành Công", age: 16, grade: 9.5),
        Student(id: UUID(), name: "Minh Tâm", age: 14, grade: 7.8)
    ]

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Danh sách học sinh"
        view.backgroundColor = .systemBackground
        setupTableView()
        setupAddButton()
    }

    private func setupTableView() {
        tableView = UITableView(frame: .zero, style: .plain)
        tableView.translatesAutoresizingMaskIntoConstraints = false
        tableView.register(StudentCell.self, forCellReuseIdentifier: StudentCell.reuseID)
        tableView.dataSource = self
        tableView.delegate = self
        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    private func setupAddButton() {
        navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .add,
                                                            target: self,
                                                            action: #selector(addStudent))
    }

    @objc private func addStudent() {
        let addVC = AddStudentViewController()
        addVC.onStudentAdded = { [weak self] newStudent in
            self?.students.append(newStudent)
            self?.tableView.reloadData()
        }
        navigationController?.pushViewController(addVC, animated: true)
    }
}

extension StudentListViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        students.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: StudentCell.reuseID, for: indexPath
        ) as? StudentCell else { return UITableViewCell() }

        cell.configure(with: students[indexPath.row])
        return cell
    }

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        let student = students[indexPath.row]
        let detailVC = StudentDetailViewController(student: student)
        navigationController?.pushViewController(detailVC, animated: true)
    }
}
```

---

## 📘 6. Màn hình thêm học sinh: `AddStudentViewController.swift`

```swift
import UIKit

final class AddStudentViewController: UIViewController {
    var onStudentAdded: ((Student) -> Void)?

    private let nameField = UITextField()
    private let ageField = UITextField()
    private let gradeField = UITextField()
    private let saveButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Thêm học sinh"
        view.backgroundColor = .systemBackground
        setupUI()
    }

    private func setupUI() {
        [nameField, ageField, gradeField].forEach {
            $0.borderStyle = .roundedRect
            $0.translatesAutoresizingMaskIntoConstraints = false
        }

        nameField.placeholder = "Tên học sinh"
        ageField.placeholder = "Tuổi"
        ageField.keyboardType = .numberPad
        gradeField.placeholder = "Điểm trung bình"
        gradeField.keyboardType = .decimalPad

        saveButton.setTitle("Lưu", for: .normal)
        saveButton.translatesAutoresizingMaskIntoConstraints = false
        saveButton.addTarget(self, action: #selector(saveStudent), for: .touchUpInside)

        let stack = UIStackView(arrangedSubviews: [nameField, ageField, gradeField, saveButton])
        stack.axis = .vertical
        stack.spacing = 16
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)

        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20)
        ])
    }

    @objc private func saveStudent() {
        guard let name = nameField.text, !name.isEmpty,
              let age = Int(ageField.text ?? ""),
              let grade = Double(gradeField.text ?? "") else { return }

        let newStudent = Student(id: UUID(), name: name, age: age, grade: grade)
        onStudentAdded?(newStudent)
        navigationController?.popViewController(animated: true)
    }
}
```

---

## 📖 7. Màn hình chi tiết học sinh: `StudentDetailViewController.swift`

```swift
import UIKit

final class StudentDetailViewController: UIViewController {
    private let student: Student

    init(student: Student) {
        self.student = student
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Chi tiết học sinh"
        view.backgroundColor = .systemBackground
        setupUI()
    }

    private func setupUI() {
        let nameLabel = UILabel()
        let ageLabel = UILabel()
        let gradeLabel = UILabel()

        [nameLabel, ageLabel, gradeLabel].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            $0.font = .preferredFont(forTextStyle: .title3)
            view.addSubview($0)
        }

        nameLabel.text = "👩‍🎓 Tên: \(student.name)"
        ageLabel.text = "🎂 Tuổi: \(student.age)"
        gradeLabel.text = "⭐️ Điểm trung bình: \(student.grade)"

        NSLayoutConstraint.activate([
            nameLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),
            nameLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            ageLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 20),
            ageLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            gradeLabel.topAnchor.constraint(equalTo: ageLabel.bottomAnchor, constant: 20),
            gradeLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }
}
```

---

## ⚙️ 8. Cấu hình entry point (SceneDelegate.swift)

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession,
            options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }
    let window = UIWindow(windowScene: windowScene)
    let nav = UINavigationController(rootViewController: StudentListViewController())
    window.rootViewController = nav
    window.makeKeyAndVisible()
    self.window = window
}
```

---

## 🧪 9. Kết quả kiểm thử

✅ Mở app → thấy danh sách học sinh.
✅ Bấm “+” → thêm học sinh mới → trở lại, danh sách cập nhật.
✅ Bấm vào học sinh → xem chi tiết.
✅ Xoay màn hình, font lớn/nhỏ → UI vẫn đẹp.

🎉 Em vừa hoàn thiện **app CRUD đầu tiên theo MVC – thuần UIKit, không storyboard.**

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 1)** 🎒

| Mức độ        | Bài tập                                      | Gợi ý                      |
| ------------- | -------------------------------------------- | -------------------------- |
| 🟢 Cơ bản     | Thêm giới tính (nam/nữ) vào model & hiển thị | Thêm `gender: String`      |
| 🟡 Trung bình | Thêm chức năng xoá học sinh trong list       | `tableView.editing = true` |
| 🔵 Nâng cao   | Sắp xếp danh sách theo điểm giảm dần         | `.sort(by:)`               |
| 🟣 Thử thách  | Lưu danh sách học sinh bằng `UserDefaults`   | Encode JSON lưu vào key    |

---

## 📚 Tổng kết buổi 1

| Thành phần | Vai trò                           |
| ---------- | --------------------------------- |
| Model      | Dữ liệu gốc (`Student`)           |
| View       | Hiển thị dữ liệu (`StudentCell`)  |
| Controller | Điều phối logic & UI              |
| Navigation | Điều hướng giữa màn hình          |
| Closure    | Truyền dữ liệu ngược (Add → List) |

---

✅ Em đã hoàn tất **dự án đầu tiên theo MVC**, là **nền tảng của MVVM, API và CoreData sau này**.
💪 Em đã chạm đến cấp độ “Junior Developer UIKit-ready”.

---

🎓 **Ngày 2 (tiếp theo):**

> *Tổ chức cấu trúc dự án chuẩn chuyên nghiệp + Áp dụng MVVM vào mini app.*

---

👉 Em có muốn thầy dạy luôn **Ngày 2: Cấu trúc dự án chuyên nghiệp & bước đầu MVVM** không?
