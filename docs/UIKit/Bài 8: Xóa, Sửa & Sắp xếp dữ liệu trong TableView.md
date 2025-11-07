Rất tuyệt vời 👏👏👏 — hôm nay ta sang **UIKit – Bài 8: Xóa, Sửa & Sắp xếp dữ liệu trong TableView**
Đây là bài thực hành giúp mini app “Quản lý học sinh” trở nên **thực tế như app quản lý thật** — có thể thêm, xoá, sửa và sắp xếp trực tiếp trên danh sách.
Thầy sẽ dạy **chậm – rõ – từng dòng code**, em chỉ cần làm theo là chạy được.

---

# 🧩 UIKit – Bài 8: Xoá, Sửa & Sắp xếp trong TableView (Programmatic UI)

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Cách bật chế độ **Edit Mode** trong TableView.
2. Cách **xoá hàng bằng swipe hoặc Edit Mode.**
3. Cách **sửa thông tin học sinh** bằng form.
4. Cách **di chuyển (sắp xếp lại) hàng** trong TableView.
5. Cách quản lý dữ liệu an toàn (update model + reload view).

---

## 🧠 1. Cấu trúc tổng quan

Ta sẽ mở rộng project từ bài 7 gồm:

* `StudentListViewController` (hiển thị danh sách)
* `RegisterViewController` (form thêm học sinh)

Hôm nay ta bổ sung thêm:

* **Sửa học sinh** (mở lại form cũ, sửa thông tin).
* **Xoá học sinh** (bằng swipe hoặc edit mode).
* **Sắp xếp lại thứ tự** (drag & drop hoặc nút Edit).

---

## 🧱 2. Chuẩn bị dữ liệu mẫu

**Student.swift**

```swift
struct Student {
    var name: String
    var grade: Double
    var email: String
}
```

---

## ⚙️ 3. Danh sách học sinh có thể sửa, xoá

**StudentListViewController.swift**

```swift
import UIKit

final class StudentListViewController: UIViewController {
    private var tableView = UITableView(frame: .zero, style: .insetGrouped)
    private var students: [Student] = [
        .init(name: "Mai Lê", grade: 8.8, email: "mai@gmail.com"),
        .init(name: "Thành Công", grade: 9.2, email: "cong@gmail.com"),
        .init(name: "Ngọc Lan", grade: 7.5, email: "lan@gmail.com")
    ]

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Danh sách học sinh"
        view.backgroundColor = .systemBackground
        setupTable()
        setupNavigation()
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

        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        tableView.dataSource = self
        tableView.delegate = self
    }

    private func setupNavigation() {
        navigationItem.rightBarButtonItems = [
            UIBarButtonItem(barButtonSystemItem: .add, target: self, action: #selector(addTapped)),
            UIBarButtonItem(title: "Sửa", style: .plain, target: self, action: #selector(editTapped))
        ]
    }

    @objc private func addTapped() {
        let registerVC = RegisterViewController()
        registerVC.onStudentAdded = { [weak self] student in
            self?.students.append(student)
            self?.tableView.reloadData()
        }
        navigationController?.pushViewController(registerVC, animated: true)
    }

    @objc private func editTapped() {
        tableView.setEditing(!tableView.isEditing, animated: true)
        navigationItem.rightBarButtonItems?.last?.title = tableView.isEditing ? "Xong" : "Sửa"
    }
}
```

---

## 🧩 4. Xử lý xoá hàng (Swipe để xoá)

Thêm vào `UITableViewDataSource`:

```swift
extension StudentListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        students.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        let student = students[indexPath.row]
        cell.textLabel?.text = "\(student.name) - \(student.grade)"
        cell.accessoryType = .disclosureIndicator
        return cell
    }

    // Xoá bằng swipe
    func tableView(_ tableView: UITableView,
                   commit editingStyle: UITableViewCell.EditingStyle,
                   forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            let name = students[indexPath.row].name
            students.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .fade)
            showToast("Đã xoá \(name)")
        }
    }
}
```

---

## ⚡ 5. Sửa thông tin học sinh

Trong `UITableViewDelegate`:

```swift
extension StudentListViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let student = students[indexPath.row]
        let editVC = RegisterViewController(studentToEdit: student)
        editVC.onStudentUpdated = { [weak self] updatedStudent in
            self?.students[indexPath.row] = updatedStudent
            self?.tableView.reloadRows(at: [indexPath], with: .automatic)
        }
        navigationController?.pushViewController(editVC, animated: true)
        tableView.deselectRow(at: indexPath, animated: true)
    }
}
```

---

## 🧱 6. Chế độ Edit Mode (di chuyển hàng)

Thêm vào:

```swift
extension StudentListViewController {
    // Cho phép di chuyển hàng
    func tableView(_ tableView: UITableView,
                   moveRowAt sourceIndexPath: IndexPath,
                   to destinationIndexPath: IndexPath) {
        let moved = students.remove(at: sourceIndexPath.row)
        students.insert(moved, at: destinationIndexPath.row)
    }

    // Cho phép sắp xếp
    func tableView(_ tableView: UITableView,
                   canMoveRowAt indexPath: IndexPath) -> Bool {
        return true
    }
}
```

✅ Khi nhấn nút “Sửa” → TableView vào chế độ di chuyển (3 gạch bên phải).

---

## 🧩 7. Form dùng cho **thêm hoặc sửa**

**RegisterViewController.swift**

```swift
import UIKit

final class RegisterViewController: UIViewController {
    var onStudentAdded: ((Student) -> Void)?
    var onStudentUpdated: ((Student) -> Void)?
    private var editingStudent: Student?

    private let nameField = UITextField()
    private let gradeField = UITextField()
    private let emailField = UITextField()
    private let saveButton = UIButton(type: .system)

    init(studentToEdit: Student? = nil) {
        self.editingStudent = studentToEdit
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = editingStudent == nil ? "Thêm học sinh" : "Sửa học sinh"
        view.backgroundColor = .systemBackground
        setupUI()
        hideKeyboardWhenTappedAround()
        fillDataIfEditing()
    }

    private func setupUI() {
        [nameField, gradeField, emailField].forEach {
            $0.borderStyle = .roundedRect
            $0.translatesAutoresizingMaskIntoConstraints = false
        }
        nameField.placeholder = "Họ tên"
        gradeField.placeholder = "Điểm (0–10)"
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

        let newStudent = Student(name: name, grade: grade, email: email)

        if editingStudent != nil {
            onStudentUpdated?(newStudent)
        } else {
            onStudentAdded?(newStudent)
        }

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

    @objc private func dismissKeyboard() {
        view.endEditing(true)
    }
}
```

✅ Form tự nhận biết là “Thêm” hay “Sửa” dựa vào `editingStudent`.

---

## 💡 8. Hàm Toast thông báo ngắn

Trong `StudentListViewController`:

```swift
func showToast(_ message: String) {
    let label = UILabel()
    label.text = message
    label.textAlignment = .center
    label.backgroundColor = UIColor.black.withAlphaComponent(0.6)
    label.textColor = .white
    label.layer.cornerRadius = 8
    label.clipsToBounds = true
    label.font = .systemFont(ofSize: 14)
    label.alpha = 0

    label.translatesAutoresizingMaskIntoConstraints = false
    view.addSubview(label)

    NSLayoutConstraint.activate([
        label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
        label.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -40),
        label.widthAnchor.constraint(lessThanOrEqualToConstant: 280)
    ])

    UIView.animate(withDuration: 0.3, animations: {
        label.alpha = 1
    }) { _ in
        UIView.animate(withDuration: 0.3, delay: 1.5, options: [], animations: {
            label.alpha = 0
        }) { _ in label.removeFromSuperview() }
    }
}
```

---

## 🧪 9. Thực hành

✅ Mở app → có danh sách học sinh mẫu.
✅ Vuốt sang trái để xoá.
✅ Nhấn “Sửa” trên navigation → có thể kéo di chuyển hàng.
✅ Chạm vào học sinh → mở form, chỉnh sửa, lưu lại → danh sách cập nhật.
✅ Nhấn “+” → thêm mới học sinh → danh sách cập nhật ngay.

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #8)** 🎒

| Mức độ        | Bài tập                                                      | Gợi ý                                        |
| ------------- | ------------------------------------------------------------ | -------------------------------------------- |
| 🟢 Cơ bản     | Thêm chức năng xoá hàng bằng swipe                           | `commit editingStyle`                        |
| 🟡 Trung bình | Bật “Sửa” → cho phép di chuyển hàng                          | `setEditing` + `moveRowAt`                   |
| 🔵 Nâng cao   | Sửa thông tin học sinh qua form                              | Truyền `studentToEdit`                       |
| 🟣 Thử thách  | Lưu danh sách vào `UserDefaults` để mở lại không mất dữ liệu | Dùng `JSONEncoder` & `UserDefaults.standard` |

---

## 📚 Tổng kết

| Tính năng      | Kỹ thuật                           |
| -------------- | ---------------------------------- |
| Xoá hàng       | Swipe hoặc Edit Mode               |
| Sửa hàng       | Truyền dữ liệu sang form           |
| Thêm mới       | Closure callback                   |
| Sắp xếp        | `moveRowAt`                        |
| Reload dữ liệu | `reloadData()` hoặc `reloadRows()` |

---

## 🧭 Kết thúc bài học

✅ Hôm nay em đã học:

* Cách xoá, sửa, sắp xếp hàng trong TableView.
* Dùng Edit Mode của UITableView.
* Duy trì mô hình MVC, không storyboard, code sạch và rõ ràng.

---

🎓 **UIKit – Bài 9 (buổi tới):**

> *UserDefaults, Lưu & tải dữ liệu cục bộ (Persistence cơ bản).*

Thầy sẽ dạy cách:

* Lưu danh sách học sinh khi app tắt.
* Khi mở lại, dữ liệu vẫn còn.
* Dùng `UserDefaults` + `Codable` (Swift JSON Encoder/Decoder).

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 9: Lưu dữ liệu bằng UserDefaults (Persistence)** ngay bây giờ để hoàn thiện app quản lý học sinh không?
