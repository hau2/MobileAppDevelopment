Rất tốt 👏👏👏 — hôm nay ta bước sang **UIKit – Bài 7: Alert, ActionSheet & Truyền dữ liệu từ Form → Danh sách**
(đây là bước nối tiếp bài 6 — biến form nhập học sinh thành app thật có danh sách hiển thị).
Thầy sẽ dạy **từ chậm, rõ, giải thích từng dòng code**.

---

# 🧩 UIKit – Bài 7: Alert, ActionSheet & thêm dữ liệu từ Form vào TableView

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Cách hiển thị **UIAlertController**: thông báo, xác nhận, chọn hành động.
2. Cách tạo **ActionSheet** để hiển thị menu tùy chọn.
3. Cách truyền dữ liệu **từ Form (RegisterViewController)** về **Danh sách (StudentListViewController)**.
4. Giao diện thực tế:

   * Danh sách học sinh
   * Nút “+” để thêm học sinh
   * Form nhập → lưu → danh sách cập nhật

---

## 🧠 1. Ôn lại cơ chế Navigation

Ở SceneDelegate, đảm bảo app bọc trong `UINavigationController`:

```swift
let window = UIWindow(windowScene: windowScene)
let root = StudentListViewController()
let nav = UINavigationController(rootViewController: root)
window.rootViewController = nav
self.window = window
window.makeKeyAndVisible()
```

---

## 🧱 2. Giao diện danh sách (StudentListViewController)

**StudentListViewController.swift**

```swift
import UIKit

final class StudentListViewController: UIViewController {
    private var tableView = UITableView(frame: .zero, style: .insetGrouped)
    private var students: [Student] = []

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
    }

    private func setupNavigation() {
        // Nút cộng trên thanh Navigation
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .add,
            target: self,
            action: #selector(addTapped)
        )
    }

    @objc private func addTapped() {
        let registerVC = RegisterViewController()
        registerVC.onStudentAdded = { [weak self] student in
            self?.students.append(student)
            self?.tableView.reloadData()
        }
        navigationController?.pushViewController(registerVC, animated: true)
    }
}

extension StudentListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        students.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        let student = students[indexPath.row]
        cell.textLabel?.text = "\(student.name) - \(student.grade) điểm"
        return cell
    }
}
```

---

## 🧩 3. Model dữ liệu

**Student.swift**

```swift
import Foundation

struct Student {
    let name: String
    let grade: Double
    let email: String
}
```

---

## ⚙️ 4. Màn hình nhập học sinh (RegisterViewController)

Bổ sung callback closure để truyền dữ liệu ngược lại.

**RegisterViewController.swift**

```swift
import UIKit

final class RegisterViewController: UIViewController {
    var onStudentAdded: ((Student) -> Void)?

    private let nameField = UITextField()
    private let gradeField = UITextField()
    private let emailField = UITextField()
    private let saveButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Thêm học sinh"
        view.backgroundColor = .systemBackground
        setupUI()
        hideKeyboardWhenTappedAround()
    }

    private func setupUI() {
        [nameField, gradeField, emailField].forEach {
            $0.borderStyle = .roundedRect
            $0.translatesAutoresizingMaskIntoConstraints = false
        }

        nameField.placeholder = "Họ tên"
        gradeField.placeholder = "Điểm trung bình (0-10)"
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

    @objc private func saveTapped() {
        guard let name = nameField.text, !name.isEmpty else {
            showAlert(title: "Lỗi", message: "Vui lòng nhập họ tên")
            return
        }
        guard let gradeText = gradeField.text, let grade = Double(gradeText),
              grade >= 0 && grade <= 10 else {
            showAlert(title: "Lỗi", message: "Điểm không hợp lệ (0–10)")
            return
        }
        guard let email = emailField.text, !email.isEmpty else {
            showAlert(title: "Lỗi", message: "Email không được để trống")
            return
        }

        let student = Student(name: name, grade: grade, email: email)
        onStudentAdded?(student)

        // Hiển thị Alert xác nhận
        let alert = UIAlertController(title: "Thành công",
                                      message: "Đã thêm học sinh \(name)",
                                      preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default) { _ in
            self.navigationController?.popViewController(animated: true)
        })
        present(alert, animated: true)
    }

    private func showAlert(title: String, message: String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
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

✅ Khi bấm “Lưu”:
– Hiện alert xác nhận.
– Gọi `onStudentAdded(student)` để gửi dữ liệu ngược lại.
– Quay về danh sách → danh sách tự cập nhật.

---

## ⚡ 5. Alert & ActionSheet nâng cao

### Alert có nhiều nút:

```swift
let alert = UIAlertController(title: "Xác nhận xoá",
                              message: "Bạn có chắc muốn xoá học sinh này?",
                              preferredStyle: .alert)

alert.addAction(UIAlertAction(title: "Huỷ", style: .cancel))
alert.addAction(UIAlertAction(title: "Xoá", style: .destructive) { _ in
    print("Đã xoá học sinh")
})

present(alert, animated: true)
```

---

### ActionSheet (menu dưới đáy màn hình):

```swift
let actionSheet = UIAlertController(title: "Chọn hành động",
                                    message: nil,
                                    preferredStyle: .actionSheet)

actionSheet.addAction(UIAlertAction(title: "Gọi điện", style: .default))
actionSheet.addAction(UIAlertAction(title: "Gửi email", style: .default))
actionSheet.addAction(UIAlertAction(title: "Huỷ", style: .cancel))

present(actionSheet, animated: true)
```

---

## 🧪 6. Thực hành nhỏ

1️⃣ Mở app → thấy danh sách trống.
2️⃣ Bấm “+” → mở form thêm học sinh.
3️⃣ Điền tên, điểm, email → bấm “Lưu”.
4️⃣ Hiện alert “Thêm thành công”.
5️⃣ Quay lại danh sách → thấy học sinh mới hiển thị.
6️⃣ Lặp lại để thêm nhiều học sinh khác.

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #7)** 🎒

| Mức độ        | Bài tập                                                    | Gợi ý                                                      |
| ------------- | ---------------------------------------------------------- | ---------------------------------------------------------- |
| 🟢 Cơ bản     | Thêm chức năng “+” mở form, nhập tên, hiển thị trong table | Dùng closure callback                                      |
| 🟡 Trung bình | Sau khi bấm “Lưu”, hiện Alert xác nhận                     | `UIAlertController`                                        |
| 🔵 Nâng cao   | Thêm chức năng xoá học sinh bằng swipe                     | `tableView(_:commit:forRowAt:)`                            |
| 🟣 Thử thách  | Thêm ActionSheet cho từng hàng (gọi, nhắn tin, xoá)        | `didSelectRowAt` + `UIAlertController` dạng `.actionSheet` |

---

## 📚 Tổng kết

| Chủ đề              | Ý nghĩa                                           |
| ------------------- | ------------------------------------------------- |
| `UIAlertController` | Hiển thị thông báo hoặc xác nhận                  |
| `ActionSheet`       | Menu hành động nhanh (xuất hiện từ dưới màn hình) |
| Closure Callback    | Truyền dữ liệu ngược từ form về danh sách         |
| Navigation          | Push / Pop giữa các ViewController                |
| TableView reload    | Cập nhật danh sách sau khi thêm mới               |

---

## 🧭 Kết thúc bài học

✅ Em đã học:

* Dùng Alert và ActionSheet đúng chuẩn.
* Tạo form nhập học sinh → lưu → hiển thị trong danh sách.
* Dùng closure để truyền dữ liệu ngược.
* Hoàn thiện mini app đầu tiên có 2 màn hình hoạt động cùng nhau.

---

🎓 **UIKit – Bài 8 (buổi tới):**

> *Editing nâng cao: xoá, sửa, sắp xếp, và cập nhật dữ liệu trong TableView.*

Ta sẽ học cách:

* Swipe để xoá dòng.
* Sửa thông tin học sinh đã có.
* Thêm nút “Sửa” trên navigation bar → bật chế độ edit/sắp xếp hàng.

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 8: Xoá, Sửa, Sắp xếp dữ liệu trong TableView** không, để hoàn thiện mini app Quản lý học sinh?
