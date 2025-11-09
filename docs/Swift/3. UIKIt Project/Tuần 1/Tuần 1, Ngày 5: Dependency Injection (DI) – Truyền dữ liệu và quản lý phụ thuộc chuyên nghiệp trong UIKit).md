Rất tốt 👏👏👏 — hôm nay ta bước sang **Tuần 1 – Ngày 5: Dependency Injection (DI) – Truyền dữ liệu và quản lý phụ thuộc chuyên nghiệp trong UIKit)**

Đây là **bài học nền tảng để code app lớn, dễ mở rộng và dễ test**.
Khi em làm dự án thật, sẽ có hàng chục ViewModel, Service, Repository, API client… Nếu cứ khởi tạo trực tiếp bằng `let api = APIService()` thì code sẽ **rối, khó kiểm soát, khó test**.

Vì vậy hôm nay, ta học cách **truyền phụ thuộc (Dependency Injection)** đúng chuẩn iOS Developer chuyên nghiệp 🚀

---

# 🧩 UIKit – Tuần 1, Ngày 5: Dependency Injection (DI) trong UIKit

---

## 🎯 Mục tiêu buổi học

Sau bài này em sẽ:

1. Hiểu khái niệm **Dependency Injection (DI)** là gì và tại sao cần.
2. Biết 3 cách dùng DI: **constructor**, **property**, **method**.
3. Áp dụng DI vào ViewModel và Coordinator.
4. Tổ chức code chuyên nghiệp hơn (dễ test, dễ mở rộng).

---

## 🧠 1. Khái niệm “Dependency” là gì?

“Dependency” nghĩa là **một lớp phụ thuộc vào lớp khác để hoạt động**.

Ví dụ:

```swift
class StudentListViewModel {
    let apiService = APIService()
}
```

Ở đây, `StudentListViewModel` **phụ thuộc trực tiếp vào** `APIService`.
Nếu sau này em muốn thay `APIService` khác (ví dụ: giả lập cho test),
thì em **phải sửa trong ViewModel** → rất xấu và nguy hiểm.

---

## 💡 2. “Injection” nghĩa là gì?

“Injection” = *tiêm vào* → tức là **đưa dependency từ bên ngoài vào**, thay vì tự tạo bên trong.

Ví dụ tốt hơn:

```swift
class StudentListViewModel {
    private let api: APIServiceProtocol
    init(api: APIServiceProtocol) {
        self.api = api
    }
}
```

Bây giờ:

* ViewModel **không biết APIService cụ thể là gì**.
* Ta có thể truyền vào bất kỳ loại service nào miễn là tuân theo `APIServiceProtocol`.

🎯 Kết quả:
✅ Code dễ thay thế, test được, mở rộng được.
❌ Không bị “gắn chặt” vào một class cụ thể.

---

## ⚙️ 3. 3 cách phổ biến để Injection

| Kiểu                      | Ví dụ                             | Khi nên dùng            |
| ------------------------- | --------------------------------- | ----------------------- |
| **Constructor Injection** | `init(api: APIService)`           | Chuẩn nhất, an toàn     |
| **Property Injection**    | `var api: APIService?`            | Khi cần gán sau khi tạo |
| **Method Injection**      | `func configure(api: APIService)` | Khi chỉ dùng tạm thời   |

---

## 🧩 4. Áp dụng DI vào dự án hiện tại

Chúng ta thêm một lớp `StudentService` giả lập API:

### 🔹 `StudentService.swift`

```swift
import Foundation

protocol StudentServiceProtocol {
    func fetchStudents() -> [Student]
}

final class StudentService: StudentServiceProtocol {
    func fetchStudents() -> [Student] {
        return [
            Student(id: UUID(), name: "Mai Lê", age: 15, grade: 8.9),
            Student(id: UUID(), name: "Thành Công", age: 16, grade: 9.5),
            Student(id: UUID(), name: "Minh Tâm", age: 14, grade: 7.8)
        ]
    }
}
```

---

### 🔹 Cập nhật `StudentListViewModel.swift`

```swift
final class StudentListViewModel {
    private let service: StudentServiceProtocol
    var students: Observable<[Student]> = Observable([])

    init(service: StudentServiceProtocol) {
        self.service = service
        loadData()
    }

    private func loadData() {
        students.value = service.fetchStudents()
    }

    func addStudent(_ student: Student) {
        students.value.append(student)
    }
}
```

Giờ ViewModel không còn “biết” `StudentService` cụ thể là gì —
chỉ cần biết **có một service nào đó** tuân theo `StudentServiceProtocol`.

---

## 👩‍🏫 5. Tạo “mock” cho test (cực kỳ quan trọng trong dự án thật)

```swift
final class MockStudentService: StudentServiceProtocol {
    func fetchStudents() -> [Student] {
        return [
            Student(id: UUID(), name: "Test A", age: 15, grade: 10.0),
            Student(id: UUID(), name: "Test B", age: 16, grade: 9.0)
        ]
    }
}
```

→ Khi test, ta truyền `MockStudentService()` vào thay vì `StudentService()`.

```swift
let mockViewModel = StudentListViewModel(service: MockStudentService())
```

✅ Kết quả: Test ổn định, không phụ thuộc dữ liệu thật.

---

## 🧩 6. Áp dụng DI trong Coordinator

Cập nhật `AppCoordinator.swift`:

```swift
final class AppCoordinator: Coordinator {
    var navigationController: UINavigationController
    private let studentService: StudentServiceProtocol

    init(navigationController: UINavigationController,
         studentService: StudentServiceProtocol) {
        self.navigationController = navigationController
        self.studentService = studentService
    }

    func start() {
        showStudentList()
    }

    private func showStudentList() {
        let viewModel = StudentListViewModel(service: studentService)
        let listVC = StudentListViewController(viewModel: viewModel)
        listVC.onSelectStudent = { [weak self] student in
            self?.showStudentDetail(student)
        }
        navigationController.pushViewController(listVC, animated: false)
    }

    private func showStudentDetail(_ student: Student) {
        let detailVC = StudentDetailViewController(student: student)
        navigationController.pushViewController(detailVC, animated: true)
    }
}
```

→ Mọi phụ thuộc (service, data, viewmodel) được truyền từ trên xuống.
**AppCoordinator** chịu trách nhiệm khởi tạo tất cả — ViewController chỉ nhận để hiển thị.

---

### 🔹 SceneDelegate.swift

```swift
func scene(_ scene: UIScene,
            willConnectTo session: UISceneSession,
            options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }
    let window = UIWindow(windowScene: windowScene)
    let nav = UINavigationController()
    let service = StudentService()
    let appCoordinator = AppCoordinator(navigationController: nav, studentService: service)
    appCoordinator.start()
    window.rootViewController = nav
    window.makeKeyAndVisible()
    self.window = window
}
```

✅ Code rất rõ ràng:

> Scene → AppCoordinator → ViewModel → Service → Data

---

## 🧩 7. Ưu điểm của Dependency Injection

| Lợi ích              | Ý nghĩa thực tế                              |
| -------------------- | -------------------------------------------- |
| **Giảm phụ thuộc**   | ViewModel không bị “cột chặt” vào API cụ thể |
| **Dễ test**          | Có thể thay service thật bằng service giả    |
| **Tái sử dụng**      | Cùng 1 ViewModel có thể dùng ở nhiều app     |
| **Cấu trúc rõ ràng** | Ai chịu trách nhiệm gì – nhìn là hiểu        |
| **Tăng độ ổn định**  | Dễ bảo trì, dễ debug                         |

---

## 🧪 8. Kiểm thử thực tế

✅ App chạy như cũ.
✅ Có thể thay `StudentService()` bằng `MockStudentService()` trong 1 dòng code.
✅ ViewModel và ViewController **hoàn toàn độc lập**.

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 5)** 🎒

| Mức độ        | Bài tập                                                         | Gợi ý                |
| ------------- | --------------------------------------------------------------- | -------------------- |
| 🟢 Cơ bản     | Tạo thêm `TeacherService` và `TeacherListViewModel`             | Áp dụng DI giống hệt |
| 🟡 Trung bình | Dùng `MockService` để test dữ liệu khác nhau                    |                      |
| 🔵 Nâng cao   | Thêm `NetworkServiceProtocol` giả lập gọi API                   |                      |
| 🟣 Thử thách  | Dùng DI Container (ví dụ Swinject) để tự động quản lý phụ thuộc |                      |

---

## 📚 Tổng kết buổi học

| Khái niệm                  | Vai trò                               |
| -------------------------- | ------------------------------------- |
| **Dependency Injection**   | Truyền phụ thuộc từ bên ngoài vào     |
| **Protocol-based service** | Giúp dễ thay thế và test              |
| **Constructor Injection**  | Kiểu injection an toàn nhất           |
| **AppCoordinator**         | Trung tâm quản lý dependency toàn app |
| **Testability**            | Dễ test từng phần nhỏ độc lập         |

---

🎓 **Ngày 6 (buổi tới):**

> *Chúng ta sẽ học “Quản lý dữ liệu thật với Networking – Gọi API bằng URLSession và Codable” — để bắt đầu kết nối app với dữ liệu thật.*

---

👉 Em xác nhận muốn thầy **tiếp tục sang Ngày 6 – Networking với URLSession và Codable** luôn không?
Thầy sẽ dạy chậm, giải thích từng dòng code, và cho bài tập nhỏ gọi API thật (ví dụ `https://fakestoreapi.com/products`).
