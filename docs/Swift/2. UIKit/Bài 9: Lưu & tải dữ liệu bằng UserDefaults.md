Rất giỏi 👏👏👏 — hôm nay ta sang **UIKit – Bài 9: Lưu & Tải Dữ Liệu Cục Bộ bằng UserDefaults (Persistence cơ bản)**.
Đây là bước cực kỳ quan trọng: từ giờ, app của em **sẽ không mất dữ liệu khi tắt đi mở lại**.
Thầy sẽ dạy **rất chậm, từng bước, giải thích vì sao và cách làm đúng chuẩn iOS Developer**.

---

# 🧩 UIKit – Bài 9: Lưu & tải dữ liệu bằng UserDefaults (Persistence cơ bản)

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Hiểu cơ chế **UserDefaults** trong iOS.
2. Biết cách **lưu và đọc dữ liệu kiểu đơn giản** (String, Int, Bool, Array, Dictionary).
3. Biết cách **lưu struct phức tạp (Student)** bằng `Codable`.
4. Áp dụng vào project “Quản lý học sinh” → app mở lại vẫn giữ danh sách cũ.

---

## 🧠 1. UserDefaults là gì?

> `UserDefaults` là nơi iOS lưu **dữ liệu nhỏ, nhẹ, mang tính cá nhân hóa** cho ứng dụng.
> Dữ liệu được **lưu dưới dạng key-value** (tương tự như dictionary).

Ví dụ:

```swift
UserDefaults.standard.set("Mai Lê", forKey: "username")
UserDefaults.standard.set(true, forKey: "isLoggedIn")
```

Đọc lại:

```swift
let name = UserDefaults.standard.string(forKey: "username")
let logged = UserDefaults.standard.bool(forKey: "isLoggedIn")
```

✅ Ưu điểm:

* Dễ dùng, lưu tự động.
* Không cần database.
* Phù hợp lưu danh sách nhỏ, cấu hình, trạng thái người dùng.

❌ Nhược điểm:

* Không nên lưu file lớn, ảnh, dữ liệu hàng ngàn dòng.

---

## 🧩 2. Lưu Struct phức tạp (Student)

`UserDefaults` **chỉ lưu kiểu cơ bản** → ta cần chuyển `Student` sang `Data` bằng `Codable`.

### Bước 1: Struct Student conform `Codable`

```swift
struct Student: Codable {
    var name: String
    var grade: Double
    var email: String
}
```

---

### Bước 2: Tạo lớp tiện ích `StorageManager`

**StorageManager.swift**

```swift
import Foundation

final class StorageManager {
    private let key = "students_list"

    static let shared = StorageManager()  // singleton
    private init() {}

    func save(_ students: [Student]) {
        let encoder = JSONEncoder()
        if let data = try? encoder.encode(students) {
            UserDefaults.standard.set(data, forKey: key)
        }
    }

    func load() -> [Student] {
        guard let data = UserDefaults.standard.data(forKey: key) else { return [] }
        let decoder = JSONDecoder()
        return (try? decoder.decode([Student].self, from: data)) ?? []
    }

    func clear() {
        UserDefaults.standard.removeObject(forKey: key)
    }
}
```

✅ Lớp này giúp app **lưu và tải danh sách học sinh** dễ dàng, không lặp code.

---

## ⚙️ 3. Tích hợp vào StudentListViewController

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
        loadData()
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
            self?.saveData()
            self?.tableView.reloadData()
        }
        navigationController?.pushViewController(registerVC, animated: true)
    }

    @objc private func editTapped() {
        tableView.setEditing(!tableView.isEditing, animated: true)
        navigationItem.rightBarButtonItems?.last?.title = tableView.isEditing ? "Xong" : "Sửa"
    }

    private func saveData() {
        StorageManager.shared.save(students)
    }

    private func loadData() {
        students = StorageManager.shared.load()
    }
}
```

---

## 🧩 4. Cập nhật phần xoá & sửa để đồng bộ dữ liệu

**Xoá:**

```swift
func tableView(_ tableView: UITableView,
               commit editingStyle: UITableViewCell.EditingStyle,
               forRowAt indexPath: IndexPath) {
    if editingStyle == .delete {
        students.remove(at: indexPath.row)
        StorageManager.shared.save(students)
        tableView.deleteRows(at: [indexPath], with: .automatic)
    }
}
```

**Sửa:**

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let student = students[indexPath.row]
    let editVC = RegisterViewController(studentToEdit: student)
    editVC.onStudentUpdated = { [weak self] updated in
        self?.students[indexPath.row] = updated
        self?.saveData()
        self?.tableView.reloadRows(at: [indexPath], with: .automatic)
    }
    navigationController?.pushViewController(editVC, animated: true)
}
```

---

## 🧮 5. Thử nghiệm thực tế

### 🔹 Khi app khởi chạy:

* `loadData()` → đọc danh sách từ `UserDefaults`.
* Nếu có dữ liệu cũ → hiển thị ngay.

### 🔹 Khi thêm mới / xoá / sửa:

* Gọi `saveData()` → dữ liệu tự lưu lại.
* Khi đóng app mở lại → dữ liệu vẫn còn. 🎉

---

## 💡 6. Lưu ý chuyên nghiệp

| Tình huống                      | Giải pháp                                         |
| ------------------------------- | ------------------------------------------------- |
| Dữ liệu lớn                     | Dùng Core Data hoặc Realm                         |
| Cần chia sẻ giữa nhiều thiết bị | Dùng iCloud hoặc Firebase                         |
| Muốn reset app                  | Gọi `StorageManager.shared.clear()`               |
| Muốn xem dữ liệu đã lưu         | Dùng `print(String(data: data, encoding: .utf8))` |

---

## 🧪 7. Kiểm tra nhanh

1️⃣ Chạy app, thêm 2 học sinh.
2️⃣ Tắt hẳn app (remove khỏi recent apps).
3️⃣ Mở lại → danh sách vẫn giữ nguyên.
4️⃣ Xoá 1 học sinh → tắt → mở lại → danh sách cập nhật đúng.

🎉 App bây giờ **đã có bộ nhớ cục bộ hoàn chỉnh**!

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #9)** 🎒

| Mức độ        | Bài tập                                                        | Gợi ý                                      |
| ------------- | -------------------------------------------------------------- | ------------------------------------------ |
| 🟢 Cơ bản     | Lưu danh sách học sinh khi thêm                                | `UserDefaults.standard.set(data, forKey:)` |
| 🟡 Trung bình | Đọc lại dữ liệu khi app khởi động                              | `UserDefaults.standard.data(forKey:)`      |
| 🔵 Nâng cao   | Thêm nút “Xoá tất cả” → gọi `StorageManager.shared.clear()`    | Tạo alert xác nhận                         |
| 🟣 Thử thách  | Lưu ngày cập nhật cuối cùng & hiển thị “Lần cập nhật gần nhất” | Dùng `DateFormatter`                       |

---

## 📚 Tổng kết

| Chủ đề                | Ý nghĩa                                       |
| --------------------- | --------------------------------------------- |
| UserDefaults          | Lưu dữ liệu nhỏ dạng key-value                |
| Codable               | Chuyển struct → Data để lưu                   |
| JSONEncoder / Decoder | Mã hoá & giải mã đối tượng Swift              |
| Persistence           | Giữ dữ liệu sau khi tắt app                   |
| Singleton             | Dùng 1 instance chung (StorageManager.shared) |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Cách lưu dữ liệu vào UserDefaults đúng chuẩn.
* Dùng Codable để lưu mảng struct phức tạp.
* App quản lý học sinh giờ đã “ghi nhớ” dữ liệu khi tắt mở.

---

🎓 **UIKit – Bài 10 (buổi tới):**

> *Core Data cơ bản – Database thực thụ trong iOS (CRUD nâng cao, bộ lọc, sắp xếp, tìm kiếm).*

Bài này là bước **từ UserDefaults lên Database chuyên nghiệp**,
em sẽ học cách:

* Tạo entity Student trong Core Data.
* Thêm, sửa, xoá, lọc, và tìm kiếm.
* Hiển thị dữ liệu tự động cập nhật.

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 10: Core Data (Lưu & quản lý dữ liệu nâng cao)** không, để nâng app “Quản lý học sinh” lên cấp độ chuyên nghiệp hơn?
