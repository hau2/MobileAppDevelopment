Tốt lắm 👏👏👏 — hôm nay ta sang **Tuần 1 – Ngày 3: Binding dữ liệu giữa ViewModel và View (Reactive MVVM cơ bản)**.
Đây là bài **rất quan trọng**, vì nó giúp em **kết nối logic và giao diện một cách tự động** — khi dữ liệu thay đổi, UI sẽ tự cập nhật, không cần gọi `reloadData()` thủ công nữa.

> Nói cách khác, đây chính là “linh hồn” của **MVVM Reactive**:
> dữ liệu ở ViewModel ↔ giao diện View luôn đồng bộ một cách tự nhiên.

---

# 🧩 UIKit – Tuần 1, Ngày 3: Binding dữ liệu giữa ViewModel & View (Reactive MVVM)

---

## 🎯 Mục tiêu buổi học

Sau buổi này em sẽ:

1. Hiểu **binding** là gì và tại sao cần có nó.
2. Biết cách tạo binding bằng **closure** (cách đơn giản nhất).
3. Làm quen với **@Published và Combine** (chuẩn Swift hiện đại).
4. Thực hành: tự động cập nhật UI khi danh sách học sinh thay đổi.

---

## 🧠 1. Khái niệm “Binding dữ liệu” là gì?

> “Binding” nghĩa là **ràng buộc dữ liệu giữa ViewModel và View**.
> Khi dữ liệu thay đổi trong ViewModel → View tự động phản ứng.

Ví dụ:

* Khi thêm học sinh mới → TableView tự reload.
* Khi sửa điểm → ô điểm tự cập nhật.
* Khi ViewModel báo lỗi → hiển thị alert ngay.

---

### 🧩 Cách cũ (truyền thống MVC):

```swift
students.append(newStudent)
tableView.reloadData()
```

→ **View phải tự cập nhật bằng tay** (dễ quên, dễ lỗi).

---

### 💡 Cách mới (MVVM Reactive):

```swift
viewModel.students.bind { _ in
    self.tableView.reloadData()
}
```

→ **Khi dữ liệu thay đổi**, closure tự chạy → UI tự động cập nhật.

---

## ⚙️ 2. Tạo lớp hỗ trợ Binding đơn giản

Tạo file mới trong thư mục **Helpers** → `Observable.swift`

```swift
final class Observable<T> {
    var value: T {
        didSet {
            listener?(value)
        }
    }

    private var listener: ((T) -> Void)?

    init(_ value: T) {
        self.value = value
    }

    func bind(_ listener: @escaping (T) -> Void) {
        self.listener = listener
        listener(value) // gọi ngay lần đầu để cập nhật giao diện ban đầu
    }
}
```

✅ Lớp này cực kỳ nhỏ nhưng mạnh:

* Giữ giá trị kiểu `T`.
* Khi `value` thay đổi → tự gọi lại closure listener.

---

## 👩‍🏫 3. Cập nhật `StudentListViewModel` để dùng Observable

```swift
import Foundation

final class StudentListViewModel {
    var students: Observable<[Student]> = Observable([])

    init() {
        students.value = [
            Student(id: UUID(), name: "Mai Lê", age: 15, grade: 8.9),
            Student(id: UUID(), name: "Thành Công", age: 16, grade: 9.5),
            Student(id: UUID(), name: "Minh Tâm", age: 14, grade: 7.8)
        ]
    }

    func addStudent(_ student: Student) {
        students.value.append(student)
    }

    func removeStudent(at index: Int) {
        students.value.remove(at: index)
    }

    func sortByGradeDescending() {
        students.value.sort { $0.grade > $1.grade }
    }

    func numberOfStudents() -> Int {
        students.value.count
    }

    func student(at index: Int) -> Student {
        students.value[index]
    }
}
```

✅ Giờ `students` là một **Observable array**.
Khi ta gán giá trị mới → UI sẽ biết để cập nhật.

---

## 📱 4. Cập nhật `StudentListViewController` để binding dữ liệu

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
        viewModel.students.bind { [weak self] _ in
            DispatchQueue.main.async {
                self?.tableView.reloadData()
            }
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
        addVM.onStudentCreated = { [weak self] student in
            self?.viewModel.addStudent(student)
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
            withIdentifier: StudentCell.reuseID,
            for: indexPath
        ) as? StudentCell else { return UITableViewCell() }

        let student = viewModel.student(at: indexPath.row)
        cell.configure(with: student)
        return cell
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

💡 Giờ em không cần gọi `reloadData()` trong `addStudent()` hoặc `removeStudent()`.
Mọi thứ **tự động update** nhờ `Observable`.

---

## ⚡ 5. Mở rộng: Dùng Combine (chuẩn Swift hiện đại)

Nếu em học lên cao hơn (iOS 15+), Apple có sẵn thư viện **Combine** cho reactive binding:

```swift
import Combine

final class StudentListViewModel {
    @Published var students: [Student] = []

    init() {
        students = [
            Student(id: UUID(), name: "Mai Lê", age: 15, grade: 8.9),
            Student(id: UUID(), name: "Thành Công", age: 16, grade: 9.5)
        ]
    }

    func addStudent(_ student: Student) {
        students.append(student)
    }
}
```

→ Trong ViewController:

```swift
private var cancellables = Set<AnyCancellable>()

viewModel.$students
    .receive(on: DispatchQueue.main)
    .sink { [weak self] _ in
        self?.tableView.reloadData()
    }
    .store(in: &cancellables)
```

💬 Combine giúp **binding tự động**, gọn hơn Observable, và là xu hướng tương lai (chuẩn Apple).

---

## 🧪 6. Kiểm thử thực tế

✅ Thêm học sinh → danh sách tự cập nhật.
✅ Xoá học sinh → bảng tự reload.
✅ Sắp xếp → UI phản ứng ngay.

Không có một dòng `tableView.reloadData()` thủ công nào trong ViewController! 🎉

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 3)** 🎒

| Mức độ        | Bài tập                                     | Gợi ý                     |
| ------------- | ------------------------------------------- | ------------------------- |
| 🟢 Cơ bản     | Dùng Observable để bind danh sách học sinh  | Như ví dụ ở trên          |
| 🟡 Trung bình | Thêm Observable cho “thông báo lỗi”         | Tạo `Observable<String?>` |
| 🔵 Nâng cao   | Thêm label hiển thị tổng số học sinh        | `Observable<Int>`         |
| 🟣 Thử thách  | Chuyển sang Combine framework (nếu iOS 15+) | Dùng `@Published`         |

---

## 📚 Tổng kết buổi học

| Thành phần             | Vai trò                                 |
| ---------------------- | --------------------------------------- |
| **Observable`<T>`**      | Lớp trung gian kết nối dữ liệu và UI    |
| **Closure Binding**    | Tự động cập nhật khi dữ liệu thay đổi   |
| **MVVM Reactive**      | ViewModel chủ động báo cho View         |
| **Combine**            | Framework reactive chính thức của Apple |
| **UI đồng bộ dữ liệu** | Không cần reload thủ công               |

---

🎓 **Ngày 4 (buổi tới):**

> *Chúng ta sẽ học “Quản lý luồng dữ liệu và điều hướng bằng Coordinator Pattern” – tách Navigation ra khỏi ViewController để code gọn, chuyên nghiệp, dễ mở rộng.*

---

👉 Em có muốn thầy tiếp tục luôn **Ngày 4 – Coordinator Pattern (Điều hướng chuyên nghiệp trong UIKit)** không?
Thầy sẽ dạy chậm, minh hoạ từng bước, giải thích rất dễ hiểu.
