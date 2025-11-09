Tốt lắm 👏👏👏 — hôm nay ta sang **Tuần 1 – Ngày 4: Làm chủ điều hướng chuyên nghiệp với *Coordinator Pattern***.

Đây là một **bước tiến rất quan trọng** trong lộ trình của em, vì nó giúp em quản lý **luồng điều hướng (Navigation flow)** trong ứng dụng lớn một cách sạch sẽ, linh hoạt, và cực kỳ “chuẩn công ty”.
Từ giờ trở đi, ViewController của em **chỉ làm UI**, còn phần “đi đâu, chuyển sang màn nào” sẽ do *Coordinator* đảm nhận.

---

# 🧭 UIKit – Tuần 1, Ngày 4: Điều hướng trong UIKit bằng Coordinator Pattern

---

## 🎯 Mục tiêu buổi học

Sau bài này em sẽ:

1. Hiểu rõ **vì sao nên dùng Coordinator** thay vì để `ViewController` tự điều hướng.
2. Biết cách **tạo BaseCoordinator** và **AppCoordinator**.
3. Biết cách **chuyển màn hình** mà không làm code bị rối.
4. Hiểu cách **truyền dữ liệu giữa các màn hình qua Coordinator**.

---

## 🧠 1. Vấn đề: ViewController bị “béo phì” (Massive View Controller)

Trong những bài trước, em có đoạn:

```swift
let detailVC = StudentDetailViewController(student: student)
navigationController?.pushViewController(detailVC, animated: true)
```

Tức là **ViewController đang kiêm luôn điều hướng**, dẫn tới:

* Mỗi lần thêm màn hình mới, phải sửa hàng loạt nơi.
* Không thể tái sử dụng luồng điều hướng.
* Dễ gây lỗi, khó test.

---

## 💡 2. Giải pháp: Coordinator Pattern

> **Coordinator** là một lớp độc lập, chịu trách nhiệm điều hướng giữa các màn hình.
> ViewController chỉ việc **báo sự kiện**, còn Coordinator quyết định **chuyển đi đâu**.

Hình minh hoạ luồng dữ liệu:

```
User tap → ViewController → thông báo cho Coordinator → Coordinator điều hướng → hiển thị ViewController mới
```

---

## ⚙️ 3. Tạo protocol cơ bản cho Coordinator

Tạo file mới: `Coordinator.swift`

```swift
import UIKit

protocol Coordinator: AnyObject {
    var navigationController: UINavigationController { get set }
    func start()
}
```

Đây là **giao diện chung** cho tất cả Coordinator.
Mỗi Coordinator sẽ “biết” cách khởi động một luồng điều hướng cụ thể.

---

## 🧩 4. Tạo AppCoordinator – điểm bắt đầu của ứng dụng

Tạo file: `AppCoordinator.swift`

```swift
import UIKit

final class AppCoordinator: Coordinator {
    var navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        showStudentList()
    }

    private func showStudentList() {
        let listVC = StudentListViewController()
        listVC.onSelectStudent = { [weak self] student in
            self?.showStudentDetail(student)
        }
        listVC.onAddStudent = { [weak self] in
            self?.showAddStudent()
        }
        navigationController.pushViewController(listVC, animated: false)
    }

    private func showStudentDetail(_ student: Student) {
        let detailVC = StudentDetailViewController(student: student)
        navigationController.pushViewController(detailVC, animated: true)
    }

    private func showAddStudent() {
        let addVC = AddStudentViewController()
        navigationController.pushViewController(addVC, animated: true)
    }
}
```

💡 `AppCoordinator` nắm luồng chính của app: danh sách → chi tiết → thêm mới.
ViewController chỉ cần “báo”, không tự `push()` nữa.

---

## 🧩 5. Kết nối AppCoordinator trong `SceneDelegate.swift`

```swift
func scene(_ scene: UIScene,
            willConnectTo session: UISceneSession,
            options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }
    let window = UIWindow(windowScene: windowScene)
    let nav = UINavigationController()
    let appCoordinator = AppCoordinator(navigationController: nav)
    appCoordinator.start()
    window.rootViewController = nav
    window.makeKeyAndVisible()
    self.window = window
}
```

---

## 👩‍🏫 6. Sửa lại `StudentListViewController.swift`

```swift
import UIKit

final class StudentListViewController: UIViewController {
    var onSelectStudent: ((Student) -> Void)?
    var onAddStudent: (() -> Void)?

    private let tableView = UITableView()
    private var students: [Student] = [
        Student(id: UUID(), name: "Mai Lê", age: 15, grade: 8.9),
        Student(id: UUID(), name: "Thành Công", age: 16, grade: 9.5)
    ]

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Danh sách học sinh"
        view.backgroundColor = .systemBackground
        setupTableView()
        navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .add,
                                                            target: self,
                                                            action: #selector(addTapped))
    }

    private func setupTableView() {
        tableView.translatesAutoresizingMaskIntoConstraints = false
        tableView.dataSource = self
        tableView.delegate = self
        tableView.register(StudentCell.self, forCellReuseIdentifier: StudentCell.reuseID)
        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    @objc private func addTapped() {
        onAddStudent?()
    }
}

extension StudentListViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        students.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(
            withIdentifier: StudentCell.reuseID, for: indexPath
        ) as! StudentCell
        cell.configure(with: students[indexPath.row])
        return cell
    }

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        onSelectStudent?(students[indexPath.row])
    }
}
```

💡 ViewController giờ **hoàn toàn không biết push hay present ai**.
Nó chỉ thông báo “người dùng đã chọn / thêm mới”, còn *Coordinator* xử lý phần còn lại.

---

## ⚡ 7. Ưu điểm cực lớn của Coordinator

| Tiêu chí      | Khi dùng Coordinator             | Khi không dùng           |
| ------------- | -------------------------------- | ------------------------ |
| Tổ chức code  | Gọn gàng, dễ tìm                 | Lẫn lộn giữa UI và logic |
| Mở rộng app   | Dễ thêm flow mới                 | Dễ phá vỡ flow cũ        |
| Tái sử dụng   | Có thể dùng lại Coordinator khác | Không thể                |
| Test          | Dễ test từng luồng riêng         | Khó test navigation      |
| Làm việc nhóm | Mỗi người 1 flow riêng           | Dễ đụng code của nhau    |

---

## 🧪 8. Kiểm thử thực tế

✅ Khi mở app → hiện danh sách học sinh.
✅ Khi chạm vào tên → Coordinator điều hướng đến màn chi tiết.
✅ Khi bấm “+” → Coordinator điều hướng đến màn thêm.
✅ ViewController không có 1 dòng `pushViewController()` nào.

🎉 Em đã **hoàn thiện một flow có điều hướng chuẩn công ty** bằng *Coordinator Pattern*.

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 4)** 🎒

| Mức độ        | Bài tập                                                         | Gợi ý                              |
| ------------- | --------------------------------------------------------------- | ---------------------------------- |
| 🟢 Cơ bản     | Thêm `EditStudentViewController` để chỉnh sửa học sinh          | Dùng callback trong Coordinator    |
| 🟡 Trung bình | Tách flow “AddStudent” ra thành `AddStudentCoordinator` riêng   | Học cách quản lý nhiều Coordinator |
| 🔵 Nâng cao   | Làm “flow con” (child coordinator)                              | Dùng mảng `childCoordinators`      |
| 🟣 Thử thách  | Viết hàm `finish()` để giải phóng Coordinator khi flow kết thúc |                                    |

---

## 📚 Tổng kết buổi học

| Thành phần           | Vai trò                                     |
| -------------------- | ------------------------------------------- |
| **Coordinator**      | Điều hướng toàn bộ flow                     |
| **ViewController**   | Hiển thị giao diện và phát tín hiệu sự kiện |
| **AppCoordinator**   | Gốc của ứng dụng                            |
| **ChildCoordinator** | Điều hướng cho từng module nhỏ              |
| **Lợi ích**          | Dễ mở rộng, sạch code, phù hợp teamwork     |

---

🎓 **Ngày 5 (buổi tới):**

> *Ta sẽ học về “Dependency Injection (DI)” – cách truyền dữ liệu, service, và ViewModel giữa các lớp mà không phụ thuộc trực tiếp, giúp dự án thực tế sạch hơn và dễ test.*

---

👉 Em xác nhận muốn thầy dạy tiếp luôn **Ngày 5 – Dependency Injection (Truyền dữ liệu và quản lý phụ thuộc chuyên nghiệp)** nhé?
Thầy sẽ hướng dẫn **rất chậm và kỹ từng bước** vì đây là một chủ đề quan trọng cho giai đoạn “dự án thật”.
