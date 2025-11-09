Tuyệt vời 👏👏👏 — hôm nay thầy và em bước sang **UIKit – Bài 10: Core Data cơ bản – Cơ sở dữ liệu thật trong iOS (CRUD nâng cao)**
Đây là **bước nhảy vọt** từ việc lưu tạm bằng `UserDefaults` sang **database chuyên nghiệp**, được Apple tích hợp sẵn — có thể lưu **hàng ngàn bản ghi, lọc, sắp xếp, tìm kiếm, cập nhật tự động UI**.
Thầy sẽ dạy chậm, rõ, minh hoạ từng phần — không cần storyboard.

---

# 🧩 UIKit – Bài 10: Core Data cơ bản (Từ lý thuyết → thực hành)

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Hiểu Core Data là gì và tại sao dùng nó.
2. Tạo **Model (Entity)** trong Core Data.
3. Dùng **NSManagedObjectContext** để thêm, xoá, sửa, đọc dữ liệu.
4. Kết nối **UITableView** hiển thị dữ liệu trực tiếp từ Core Data.
5. Sử dụng **NSFetchedResultsController** để tự động cập nhật danh sách khi có thay đổi.

---

## 🧠 1. Core Data là gì?

> Core Data là **framework quản lý dữ liệu dạng đối tượng**, do Apple cung cấp.
> Nó **không chỉ là database**, mà là **ORM (Object Relational Mapping)** giúp em thao tác dữ liệu bằng Swift Object.

| Thành phần                           | Vai trò                         |
| ------------------------------------ | ------------------------------- |
| **Entity**                           | Giống bảng trong database       |
| **Attribute**                        | Giống cột (name, grade, email)  |
| **ManagedObject**                    | Một dòng dữ liệu (student)      |
| **Context (NSManagedObjectContext)** | Nơi thao tác thêm/sửa/xoá       |
| **PersistentContainer**              | Quản lý toàn bộ Core Data stack |

---

## ⚙️ 2. Tạo Core Data Model

1️⃣ Trong Xcode → New File → *Data Model* → đặt tên `StudentModel.xcdatamodeld`.
2️⃣ Thêm **Entity**: `StudentEntity`.
3️⃣ Thêm các **Attribute**:

| Tên   | Kiểu   |
| ----- | ------ |
| name  | String |
| grade | Double |
| email | String |

4️⃣ Đánh dấu entity là **Codegen: Class Definition**
→ Xcode tự sinh ra lớp `StudentEntity+CoreDataProperties.swift`.

---

## 🧩 3. Tạo Core Data Stack

Thêm file **CoreDataManager.swift** để quản lý stack:

```swift
import CoreData

final class CoreDataManager {
    static let shared = CoreDataManager()
    private init() {}

    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "StudentModel")
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("❌ Lỗi khởi tạo Core Data: \(error)")
            }
        }
        return container
    }()

    var context: NSManagedObjectContext {
        return persistentContainer.viewContext
    }

    func saveContext() {
        let context = persistentContainer.viewContext
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                print("❌ Lỗi lưu context: \(error.localizedDescription)")
            }
        }
    }
}
```

---

## 🧱 4. CRUD cơ bản trong Core Data

### 🟢 Thêm học sinh:

```swift
func addStudent(name: String, grade: Double, email: String) {
    let student = StudentEntity(context: CoreDataManager.shared.context)
    student.name = name
    student.grade = grade
    student.email = email
    CoreDataManager.shared.saveContext()
}
```

---

### 🟡 Đọc tất cả học sinh:

```swift
func fetchStudents() -> [StudentEntity] {
    let request: NSFetchRequest<StudentEntity> = StudentEntity.fetchRequest()
    request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
    return (try? CoreDataManager.shared.context.fetch(request)) ?? []
}
```

---

### 🔵 Sửa học sinh:

```swift
func updateStudent(_ student: StudentEntity, name: String, grade: Double, email: String) {
    student.name = name
    student.grade = grade
    student.email = email
    CoreDataManager.shared.saveContext()
}
```

---

### 🔴 Xoá học sinh:

```swift
func deleteStudent(_ student: StudentEntity) {
    CoreDataManager.shared.context.delete(student)
    CoreDataManager.shared.saveContext()
}
```

---

## 🧩 5. Áp dụng vào giao diện

**StudentListViewController.swift**

```swift
import UIKit
import CoreData

final class StudentListViewController: UIViewController {
    private var tableView = UITableView(frame: .zero, style: .insetGrouped)
    private var students: [StudentEntity] = []

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Danh sách học sinh"
        view.backgroundColor = .systemBackground
        setupTable()
        setupNavigation()
        loadStudents()
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

        tableView.dataSource = self
        tableView.delegate = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
    }

    private func setupNavigation() {
        navigationItem.rightBarButtonItems = [
            UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(addTapped)),
            UIBarButtonItem(title: "Sửa", style: .plain, target: self, action: #selector(editTapped))
        ]
    }

    @objc private func addTapped() {
        let vc = RegisterViewController()
        vc.onStudentSaved = { [weak self] in
            self?.loadStudents()
        }
        navigationController?.pushViewController(vc, animated: true)
    }

    @objc private func editTapped() {
        tableView.setEditing(!tableView.isEditing, animated: true)
        navigationItem.rightBarButtonItems?.last?.title = tableView.isEditing ? "Xong" : "Sửa"
    }

    private func loadStudents() {
        students = fetchStudents()
        tableView.reloadData()
    }

    private func fetchStudents() -> [StudentEntity] {
        let request: NSFetchRequest<StudentEntity> = StudentEntity.fetchRequest()
        let sort = NSSortDescriptor(key: "name", ascending: true)
        request.sortDescriptors = [sort]
        return (try? CoreDataManager.shared.context.fetch(request)) ?? []
    }
}

extension StudentListViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        students.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        let s = students[indexPath.row]
        cell.textLabel?.text = "\(s.name ?? "") - \(s.grade)"
        cell.accessoryType = .disclosureIndicator
        return cell
    }

    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle,
                   forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            let s = students[indexPath.row]
            deleteStudent(s)
            students.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .automatic)
        }
    }

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let s = students[indexPath.row]
        let vc = RegisterViewController(student: s)
        vc.onStudentSaved = { [weak self] in
            self?.loadStudents()
        }
        navigationController?.pushViewController(vc, animated: true)
        tableView.deselectRow(at: indexPath, animated: true)
    }
}
```

---

## ⚡ 6. Form nhập học sinh (sử dụng Core Data)

**RegisterViewController.swift**

```swift
import UIKit
import CoreData

final class RegisterViewController: UIViewController {
    var onStudentSaved: (() -> Void)?
    private var editingStudent: StudentEntity?

    private let nameField = UITextField()
    private let gradeField = UITextField()
    private let emailField = UITextField()
    private let saveButton = UIButton(type: .system)

    init(student: StudentEntity? = nil) {
        self.editingStudent = student
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        title = editingStudent == nil ? "Thêm học sinh" : "Sửa học sinh"
        setupUI()
        fillDataIfEditing()
        hideKeyboardWhenTappedAround()
    }

    private func setupUI() {
        [nameField, gradeField, emailField].forEach {
            $0.borderStyle = .roundedRect
            $0.translatesAutoresizingMaskIntoConstraints = false
        }
        nameField.placeholder = "Họ tên"
        gradeField.placeholder = "Điểm trung bình (0–10)"
        gradeField.keyboardType = .decimalPad
        emailField.placeholder = "Email"

        saveButton.setTitle("Lưu", for: .normal)
        saveButton.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)

        let stack = UIStackView(arrangedSubviews: [nameField, gradeField, emailField, saveButton])
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

    private func fillDataIfEditing() {
        guard let s = editingStudent else { return }
        nameField.text = s.name
        gradeField.text = "\(s.grade)"
        emailField.text = s.email
    }

    @objc private func saveTapped() {
        guard let name = nameField.text, !name.isEmpty,
              let gradeText = gradeField.text, let grade = Double(gradeText),
              let email = emailField.text, !email.isEmpty else {
            showAlert("Vui lòng nhập đủ thông tin!")
            return
        }

        if let student = editingStudent {
            updateStudent(student, name: name, grade: grade, email: email)
        } else {
            addStudent(name: name, grade: grade, email: email)
        }
        onStudentSaved?()
        navigationController?.popViewController(animated: true)
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

    @objc private func dismissKeyboard() { view.endEditing(true) }
}
```

---

## 🧪 7. Thực hành

1️⃣ Mở app → thêm học sinh → Core Data tự lưu.
2️⃣ Tắt app → mở lại → danh sách vẫn còn.
3️⃣ Sửa học sinh → thông tin cập nhật ngay.
4️⃣ Xoá học sinh → hàng biến mất, database cập nhật.
🎉 App giờ dùng **cơ sở dữ liệu thật** như app thương mại chuyên nghiệp!

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #10)** 🎒

| Mức độ        | Bài tập                                                    | Gợi ý                                                |
| ------------- | ---------------------------------------------------------- | ---------------------------------------------------- |
| 🟢 Cơ bản     | Tạo model StudentEntity, thêm/sửa/xoá học sinh             | CoreDataManager.shared.context                       |
| 🟡 Trung bình | Sắp xếp danh sách theo tên                                 | `NSSortDescriptor`                                   |
| 🔵 Nâng cao   | Thêm tìm kiếm theo tên học sinh                            | `NSPredicate(format: "name CONTAINS[cd] %@", query)` |
| 🟣 Thử thách  | Thêm cột “ngày cập nhật” & sắp xếp theo thời gian mới nhất | Thêm attribute `updatedAt: Date`                     |

---

## 📚 Tổng kết

| Thành phần          | Ý nghĩa                              |
| ------------------- | ------------------------------------ |
| Core Data           | Framework lưu dữ liệu dạng đối tượng |
| Entity              | Giống bảng trong DB                  |
| ManagedObject       | Dòng dữ liệu                         |
| Context             | Quản lý thao tác CRUD                |
| PersistentContainer | Chứa toàn bộ Stack                   |
| NSFetchRequest      | Truy vấn dữ liệu                     |
| NSPredicate         | Điều kiện lọc                        |
| NSSortDescriptor    | Sắp xếp kết quả                      |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Tạo, lưu, đọc, sửa, xoá dữ liệu bằng Core Data.
* Hiển thị dữ liệu trên TableView.
* Giữ dữ liệu ổn định sau mỗi lần mở app.
* Hiểu luồng CRUD chuẩn trong iOS.

---

🎓 **UIKit – Bài 11 (buổi tới):**

> *NSFetchedResultsController – cập nhật tự động UI khi Core Data thay đổi + SearchBar + Filter.*

Thầy sẽ dạy:

* Dùng `NSFetchedResultsController` để UI tự cập nhật khi CRUD.
* Thêm `UISearchBar` tìm kiếm học sinh.
* Thêm `UISegmentedControl` để lọc theo điểm (`>=8`, `5–7`, `<5`).

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 11: FetchedResultsController + Search + Filter (Core Data nâng cao)** không?
