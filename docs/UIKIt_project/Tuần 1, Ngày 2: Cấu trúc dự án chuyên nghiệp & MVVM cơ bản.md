Rất tốt 👏👏👏 — hôm nay chúng ta bước sang **Tuần 1 – Ngày 2: Tổ chức cấu trúc dự án chuyên nghiệp & bước đầu với mô hình MVVM**.

Đây là **bước chuyển cực kỳ quan trọng**:
Từ chỗ em viết code “chạy được” sang code “dễ mở rộng, dễ bảo trì, làm việc nhóm chuyên nghiệp”.
Từ hôm nay, ta sẽ **tổ chức lại cấu trúc project**, tách logic ra khỏi giao diện — chuẩn như trong dự án thật của các công ty iOS developer chuyên nghiệp.

---

# 🧩 UIKit – Tuần 1, Ngày 2: Cấu trúc dự án chuyên nghiệp & MVVM cơ bản

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Hiểu rõ sự khác biệt giữa **MVC** và **MVVM**.
2. Biết cách tách logic khỏi ViewController (sạch code hơn).
3. Tổ chức thư mục & tệp đúng chuẩn dự án công ty.
4. Áp dụng MVVM vào mini app “Danh sách học sinh”.

---

## 🧠 1. Vì sao phải chuyển từ MVC → MVVM?

| Mô hình  | Đặc điểm                      | Hạn chế                            |
| -------- | ----------------------------- | ---------------------------------- |
| **MVC**  | Controller điều khiển mọi thứ | Code dễ “phình to” (Massive VC)    |
| **MVVM** | Tách logic ra `ViewModel`     | Dễ test, dễ mở rộng, code gọn gàng |

📘 Trong MVVM:

* **Model**: dữ liệu (giống trước).
* **ViewModel**: xử lý logic, cung cấp dữ liệu cho View.
* **ViewController (View)**: chỉ hiển thị giao diện, không làm logic.

> Nói ngắn gọn:
> **ViewModel = “bộ não”**,
> **ViewController = “khuôn mặt”**.

---

## 🧱 2. Cấu trúc thư mục chuẩn MVVM

Cập nhật lại project `StudentListApp` như sau:

```
StudentListApp
├── App
│   └── AppDelegate.swift
│   └── SceneDelegate.swift
├── Models
│   └── Student.swift
├── ViewModels
│   └── StudentListViewModel.swift
│   └── AddStudentViewModel.swift
├── Views
│   └── StudentCell.swift
├── Screens
│   ├── StudentList
│   │   └── StudentListViewController.swift
│   ├── AddStudent
│   │   └── AddStudentViewController.swift
│   └── StudentDetail
│       └── StudentDetailViewController.swift
└── Resources
    └── Assets.xcassets
```

💡 **Ưu điểm:**

* Từng màn hình có “cặp đôi” View + ViewModel riêng.
* Code dễ tìm, dễ mở rộng.
* Chuẩn kiến trúc iOS hiện đại (Apple và công ty đều dùng kiểu này).

---

## 🧩 3. Tạo `StudentListViewModel.swift`

```swift
import Foundation

final class StudentListViewModel {
    private(set) var students: [Student] = [
        Student(id: UUID(), name: "Mai Lê", age: 15, grade: 8.9),
        Student(id: UUID(), name: "Thành Công", age: 16, grade: 9.5),
        Student(id: UUID(), name: "Minh Tâm", age: 14, grade: 7.8)
    ]

    var onDataChanged: (() -> Void)?

    func addStudent(_ student: Student) {
        students.append(student)
        onDataChanged?()
    }

    func student(at index: Int) -> Student {
        students[index]
    }

    func numberOfStudents() -> Int {
        students.count
    }

    func removeStudent(at index: Int) {
        students.remove(at: index)
        onDataChanged?()
    }

    func sortByGradeDescending() {
        students.sort { $0.grade > $1.grade }
        onDataChanged?()
    }
}
```

💡 Giờ đây toàn bộ logic (thêm, xoá, sắp xếp, lấy dữ liệu) **được tách khỏi ViewController**.

---

## 🧩 4. Cập nhật `StudentListViewController.swift`

```swift
import UIKit

final class StudentListViewController: UIViewController {
    private var tableView: UITableView!
    private let viewModel = StudentListViewModel()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Danh sách học sinh"
        view.backgroundColor = .systemBackground
        setupTableView()
        setupButtons()
        bindViewModel()
    }

    private func bindViewModel() {
        viewModel.onDataChanged = { [weak self] in
            self?.tableView.reloadData()
        }
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

    private func setupButtons() {
        navigationItem.rightBarButtonItems = [
            UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(addStudent)),
            UIBarButtonItem(title: "Sắp xếp", style: .plain, target: self, action: #selector(sortStudents))
        ]
    }

    @objc private func addStudent() {
        let addVM = AddStudentViewModel()
        let addVC = AddStudentViewController(viewModel: addVM)
        addVM.onStudentCreated = { [weak self] newStudent in
            self?.viewModel.addStudent(newStudent)
        }
        navigationController?.pushViewController(addVC, animated: true)
    }

    @objc private func sortStudents() {
        viewModel.sortByGradeDescending()
    }
}

extension StudentListViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        viewModel.numberOfStudents()
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: StudentCell.reuseID, for: indexPath
        ) as? StudentCell else { return UITableViewCell() }

        let student = viewModel.student(at: indexPath.row)
        cell.configure(with: student)
        return cell
    }

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        let student = viewModel.student(at: indexPath.row)
        let detailVC = StudentDetailViewController(student: student)
        navigationController?.pushViewController(detailVC, animated: true)
    }

    func tableView(_ tableView: UITableView,
                   trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath)
    -> UISwipeActionsConfiguration? {
        let delete = UIContextualAction(style: .destructive, title: "Xóa") { [weak self] _, _, _ in
            self?.viewModel.removeStudent(at: indexPath.row)
        }
        return UISwipeActionsConfiguration(actions: [delete])
    }
}
```

💡 Giờ đây `ViewController` chỉ hiển thị giao diện và nhận tương tác.
Logic (thêm/xoá/sắp xếp) đều do `ViewModel` xử lý.

---

## 👩‍🏫 5. Tạo `AddStudentViewModel.swift`

```swift
import Foundation

final class AddStudentViewModel {
    var name: String = ""
    var age: Int = 0
    var grade: Double = 0.0
    var onStudentCreated: ((Student) -> Void)?

    func createStudent() {
        let student = Student(id: UUID(), name: name, age: age, grade: grade)
        onStudentCreated?(student)
    }
}
```

---

## 🧩 6. Cập nhật `AddStudentViewController.swift`

```swift
import UIKit

final class AddStudentViewController: UIViewController {
    private let viewModel: AddStudentViewModel
    private let nameField = UITextField()
    private let ageField = UITextField()
    private let gradeField = UITextField()
    private let saveButton = UIButton(type: .system)

    init(viewModel: AddStudentViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

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
        gradeField.placeholder = "Điểm"
        ageField.keyboardType = .numberPad
        gradeField.keyboardType = .decimalPad

        saveButton.setTitle("Lưu", for: .normal)
        saveButton.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)
        saveButton.translatesAutoresizingMaskIntoConstraints = false

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

    @objc private func saveTapped() {
        guard let name = nameField.text, !name.isEmpty,
              let age = Int(ageField.text ?? ""),
              let grade = Double(gradeField.text ?? "") else { return }

        viewModel.name = name
        viewModel.age = age
        viewModel.grade = grade
        viewModel.createStudent()
        navigationController?.popViewController(animated: true)
    }
}
```

---

## 🧪 7. Chạy thử và quan sát

✅ App hoạt động như cũ:

* Thêm học sinh → danh sách tự cập nhật.
* Xoá học sinh → cập nhật mượt.
* Sắp xếp theo điểm.

⚙️ Nhưng giờ cấu trúc **rõ ràng, tách biệt logic & giao diện**, đúng chuẩn doanh nghiệp.

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 2)** 🎒

| Mức độ        | Bài tập                                   | Gợi ý                                  |
| ------------- | ----------------------------------------- | -------------------------------------- |
| 🟢 Cơ bản     | Thêm giới tính & cập nhật ViewModel       | thêm field `gender`                    |
| 🟡 Trung bình | Thêm nút “Sắp xếp theo tên”               | thêm `sortByName()` trong ViewModel    |
| 🔵 Nâng cao   | Thêm tính năng “Tìm kiếm học sinh”        | tạo `filteredStudents` trong ViewModel |
| 🟣 Thử thách  | Lưu toàn bộ danh sách bằng `UserDefaults` | serialize `students` thành JSON        |

---

## 📚 Tổng kết buổi học

| Thành phần          | Vai trò                                  |
| ------------------- | ---------------------------------------- |
| **ViewModel**       | Xử lý logic, cung cấp dữ liệu cho View   |
| **ViewController**  | Hiển thị giao diện & phản hồi người dùng |
| **Closure Binding** | Liên kết data thay đổi → UI cập nhật     |
| **MVVM**            | Mô hình phổ biến trong iOS hiện đại      |
| **Clean Code**      | Dễ đọc, dễ mở rộng, dễ làm việc nhóm     |

---

🎓 **Ngày 3 (buổi tới):**

> *Giới thiệu Binding nâng cao – Kết nối dữ liệu giữa ViewModel & View bằng Closure, Combine và cập nhật tự động (MVVM Reactive).*

---

👉 Em có muốn thầy tiếp tục luôn **Ngày 3 – Binding dữ liệu giữa ViewModel & View (Reactive MVVM)** không?
