Tuyệt vời 👏👏👏 — hôm nay ta sang **UIKit – Bài 11: NSFetchedResultsController + SearchBar + Filter (Core Data nâng cao)**
Đây là **bước chuyên nghiệp hóa** cho ứng dụng quản lý học sinh của em:

> App sẽ **tự động cập nhật giao diện khi có thay đổi dữ liệu (add / edit / delete)**,
> và có thể **tìm kiếm + lọc thông minh** theo tên hoặc theo điểm.

Thầy sẽ hướng dẫn **rất kỹ – từng dòng code – để hiểu bản chất cơ chế quan sát Core Data**.

---

# 🧩 UIKit – Bài 11: FetchedResultsController, Search & Filter

---

## 🎯 Mục tiêu bài học

Sau buổi học, em sẽ biết:

1. Cách dùng `NSFetchedResultsController` để tự cập nhật TableView khi dữ liệu thay đổi.
2. Cách thêm `UISearchBar` để tìm kiếm trong Core Data.
3. Cách thêm `UISegmentedControl` để lọc dữ liệu (giỏi, khá, yếu).
4. Cách kết hợp tất cả với **UITableView + Core Data** để thành ứng dụng hoàn chỉnh.

---

## 🧠 1. Giới thiệu `NSFetchedResultsController`

> `NSFetchedResultsController` (viết tắt là **FRC**) là một **trình quản lý truy vấn động cho Core Data**.
> Nó theo dõi `NSFetchRequest` và **tự động thông báo cho TableView khi dữ liệu thay đổi.**

| Ưu điểm                 | Giải thích                                       |
| ----------------------- | ------------------------------------------------ |
| 🔁 Tự reload TableView  | Không cần gọi `reloadData()` thủ công            |
| ⚡ Hiệu năng cao         | Chỉ cập nhật hàng thay đổi, không reload toàn bộ |
| 🔍 Hỗ trợ sort & filter | Dễ dàng thêm tìm kiếm                            |
| 📦 Dùng nhiều section   | Có thể nhóm dữ liệu                              |

---

## ⚙️ 2. Chuẩn bị cơ bản

Vẫn dùng `StudentEntity` trong Core Data với 3 thuộc tính:

* name: String
* grade: Double
* email: String

---

## 🧱 3. Cấu trúc ViewController mới

**StudentListViewController.swift**

```swift
import UIKit
import CoreData

final class StudentListViewController: UIViewController {
    private var tableView = UITableView(frame: .zero, style: .insetGrouped)
    private var searchBar = UISearchBar()
    private var filterControl = UISegmentedControl(items: ["Tất cả", "Giỏi ≥8", "Khá 5–7", "Yếu <5"])

    private var fetchedResultsController: NSFetchedResultsController<StudentEntity>!

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Quản lý học sinh"
        view.backgroundColor = .systemBackground
        setupUI()
        setupFetchedResultsController()
    }
}
```

---

## 🧩 4. Cấu hình giao diện

```swift
private func setupUI() {
    // Search bar
    searchBar.placeholder = "Tìm theo tên..."
    searchBar.delegate = self

    // Filter control
    filterControl.selectedSegmentIndex = 0
    filterControl.addTarget(self, action: #selector(filterChanged), for: .valueChanged)

    // Table view
    tableView.translatesAutoresizingMaskIntoConstraints = false
    view.addSubview(searchBar)
    view.addSubview(filterControl)
    view.addSubview(tableView)
    tableView.dataSource = self
    tableView.delegate = self
    tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")

    searchBar.translatesAutoresizingMaskIntoConstraints = false
    filterControl.translatesAutoresizingMaskIntoConstraints = false

    NSLayoutConstraint.activate([
        searchBar.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
        searchBar.leadingAnchor.constraint(equalTo: view.leadingAnchor),
        searchBar.trailingAnchor.constraint(equalTo: view.trailingAnchor),

        filterControl.topAnchor.constraint(equalTo: searchBar.bottomAnchor),
        filterControl.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 12),
        filterControl.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -12),

        tableView.topAnchor.constraint(equalTo: filterControl.bottomAnchor, constant: 8),
        tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
        tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
    ])

    navigationItem.rightBarButtonItem = UIBarButtonItem(
        barButtonSystemItem: .add, target: self, action: #selector(addTapped)
    )
}
```

---

## ⚙️ 5. Cấu hình FetchedResultsController

```swift
private func setupFetchedResultsController(predicate: NSPredicate? = nil) {
    let request: NSFetchRequest<StudentEntity> = StudentEntity.fetchRequest()
    request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
    if let predicate = predicate {
        request.predicate = predicate
    }

    fetchedResultsController = NSFetchedResultsController(
        fetchRequest: request,
        managedObjectContext: CoreDataManager.shared.context,
        sectionNameKeyPath: nil,
        cacheName: nil
    )
    fetchedResultsController.delegate = self

    do {
        try fetchedResultsController.performFetch()
    } catch {
        print("❌ Lỗi fetch dữ liệu: \(error.localizedDescription)")
    }

    tableView.reloadData()
}
```

---

## 🧱 6. Hiển thị dữ liệu tự động

```swift
extension StudentListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        fetchedResultsController.fetchedObjects?.count ?? 0
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        let s = fetchedResultsController.object(at: indexPath)
        cell.textLabel?.text = "\(s.name ?? "") - \(s.grade)"
        return cell
    }

    // Swipe để xoá
    func tableView(_ tableView: UITableView,
                   commit editingStyle: UITableViewCell.EditingStyle,
                   forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            let student = fetchedResultsController.object(at: indexPath)
            CoreDataManager.shared.context.delete(student)
            CoreDataManager.shared.saveContext()
        }
    }
}
```

---

## ⚡ 7. Tự cập nhật UI khi CRUD

```swift
extension StudentListViewController: NSFetchedResultsControllerDelegate {
    func controllerWillChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        tableView.beginUpdates()
    }

    func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>,
                    didChange anObject: Any,
                    at indexPath: IndexPath?,
                    for type: NSFetchedResultsChangeType,
                    newIndexPath: IndexPath?) {

        switch type {
        case .insert:
            if let newIndexPath = newIndexPath {
                tableView.insertRows(at: [newIndexPath], with: .fade)
            }
        case .delete:
            if let indexPath = indexPath {
                tableView.deleteRows(at: [indexPath], with: .fade)
            }
        case .update:
            if let indexPath = indexPath {
                let cell = tableView.cellForRow(at: indexPath)
                let s = fetchedResultsController.object(at: indexPath)
                cell?.textLabel?.text = "\(s.name ?? "") - \(s.grade)"
            }
        default:
            break
        }
    }

    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        tableView.endUpdates()
    }
}
```

✅ Từ nay, khi em **thêm / xoá / sửa dữ liệu**, TableView **tự động cập nhật** — không cần reload thủ công.

---

## 🔍 8. Tìm kiếm bằng UISearchBar

```swift
extension StudentListViewController: UISearchBarDelegate {
    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        updateFetchResults()
    }

    func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
        view.endEditing(true)
    }
}
```

---

## 🧮 9. Lọc theo điểm với UISegmentedControl

```swift
@objc private func filterChanged() {
    updateFetchResults()
}

private func updateFetchResults() {
    let text = searchBar.text?.lowercased() ?? ""
    var predicates: [NSPredicate] = []

    if !text.isEmpty {
        predicates.append(NSPredicate(format: "name CONTAINS[cd] %@", text))
    }

    switch filterControl.selectedSegmentIndex {
    case 1:
        predicates.append(NSPredicate(format: "grade >= 8"))
    case 2:
        predicates.append(NSPredicate(format: "grade >= 5 AND grade < 8"))
    case 3:
        predicates.append(NSPredicate(format: "grade < 5"))
    default:
        break
    }

    let predicate = predicates.isEmpty ? nil : NSCompoundPredicate(andPredicateWithSubpredicates: predicates)
    setupFetchedResultsController(predicate: predicate)
}
```

---

## 🧩 10. Thêm học sinh mới (tương tự các bài trước)

```swift
@objc private func addTapped() {
    let vc = RegisterViewController()
    navigationController?.pushViewController(vc, animated: true)
}
```

Khi bấm “Lưu” trong form → Core Data thay đổi → `NSFetchedResultsController` tự reload danh sách.

---

## 🧪 11. Thực hành kết quả

1️⃣ Mở app → thấy danh sách học sinh Core Data.
2️⃣ Gõ vào thanh tìm kiếm → danh sách lọc ngay.
3️⃣ Chọn “Giỏi ≥8” → chỉ còn học sinh điểm cao.
4️⃣ Thêm mới → TableView cập nhật ngay lập tức.
5️⃣ Xoá → hàng biến mất tức thì, không reload. 🎉

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #11)** 🎒

| Mức độ        | Bài tập                                 | Gợi ý                                                            |
| ------------- | --------------------------------------- | ---------------------------------------------------------------- |
| 🟢 Cơ bản     | Thêm SearchBar để tìm theo tên          | `NSPredicate(format: "name CONTAINS[cd] %@", text)`              |
| 🟡 Trung bình | Lọc theo điểm bằng SegmentedControl     | Dùng nhiều predicate kết hợp                                     |
| 🔵 Nâng cao   | Dùng FRC để tự cập nhật TableView       | `controllerDidChangeContent()`                                   |
| 🟣 Thử thách  | Thêm section “Giỏi – Khá – Yếu” tự động | `sectionNameKeyPath: "category"` (tạo thêm attribute “category”) |

---

## 📚 Tổng kết

| Thành phần                 | Chức năng                                        |
| -------------------------- | ------------------------------------------------ |
| NSFetchedResultsController | Theo dõi dữ liệu Core Data & cập nhật UI tự động |
| NSPredicate                | Bộ lọc điều kiện                                 |
| NSSortDescriptor           | Sắp xếp dữ liệu                                  |
| UISearchBar                | Tìm kiếm realtime                                |
| UISegmentedControl         | Lọc nhiều điều kiện                              |
| Delegate của FRC           | Gọi lại khi CRUD xảy ra                          |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Dùng NSFetchedResultsController để quản lý dữ liệu thông minh.
* Kết hợp SearchBar và Filter.
* Tạo app iOS quản lý học sinh có CRUD + tìm kiếm + lọc + cập nhật UI tự động.

---

🎓 **UIKit – Bài 12 (buổi tới):**

> *Tùy chỉnh Cell chuyên nghiệp: Custom UITableViewCell với Avatar, Layout, Auto Layout + Dark Mode + Dynamic Type.*

Thầy sẽ hướng dẫn:

* Thiết kế cell đẹp, có ảnh đại diện (UIImageView + label).
* Dùng Auto Layout cho giao diện linh hoạt.
* Hỗ trợ Dark Mode và cỡ chữ động (Accessibility).

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 12: Custom UITableViewCell chuyên nghiệp (Avatar + Auto Layout + Dark Mode)** không?
