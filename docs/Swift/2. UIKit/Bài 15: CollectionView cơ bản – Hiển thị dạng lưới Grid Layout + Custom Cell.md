Tốt lắm 👏👏👏 — hôm nay thầy và em bước sang **UIKit – Bài 15: CollectionView cơ bản – Hiển thị dạng lưới (Grid Layout + Custom Cell)**.
Đây là **cột mốc lớn** vì CollectionView là nền tảng cho rất nhiều giao diện hiện đại:

> App ảnh, danh bạ, sản phẩm, gallery, thậm chí layout dạng “feed” như Facebook, TikTok, Instagram — đều dùng **UICollectionView**.

Thầy sẽ dạy **từng bước rõ ràng**, từ khái niệm → ví dụ cụ thể → bài tập thực hành.

---

# 🧩 UIKit – Bài 15: CollectionView cơ bản (Grid Layout + Custom Cell)

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ biết:

1. Hiểu cấu trúc `UICollectionView` và `UICollectionViewFlowLayout`.
2. Tạo `CollectionView` hoàn toàn bằng code.
3. Hiển thị danh sách học sinh theo dạng lưới.
4. Tạo custom cell có ảnh, tên, điểm.
5. Xử lý chọn item (chạm vào học sinh để xem chi tiết).

---

## 🧠 1. UICollectionView là gì?

> `UICollectionView` là một phiên bản “nâng cao” của TableView,
> cho phép hiển thị dữ liệu dạng **lưới (grid)** hoặc **layout tự do**.

| So sánh   | TableView         | CollectionView                              |
| --------- | ----------------- | ------------------------------------------- |
| Giao diện | Một cột dọc       | Nhiều cột, có thể cuộn ngang                |
| Cell      | `UITableViewCell` | `UICollectionViewCell`                      |
| Layout    | Mặc định (List)   | Tùy chỉnh bằng `UICollectionViewFlowLayout` |
| Ứng dụng  | Danh sách, form   | Ảnh, card, gallery, tag, feed               |

---

## ⚙️ 2. Tạo CollectionView cơ bản

Tạo file mới: **StudentGridViewController.swift**

```swift
import UIKit
import CoreData

final class StudentGridViewController: UIViewController {
    private var collectionView: UICollectionView!
    private var fetchedResultsController: NSFetchedResultsController<StudentEntity>!

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Lưới học sinh"
        view.backgroundColor = .systemBackground
        setupCollectionView()
        setupFetchedResultsController()
    }
}
```

---

## 🧩 3. Cấu hình Layout

`UICollectionViewFlowLayout` giúp điều chỉnh số cột, khoảng cách, kích thước cell.

```swift
private func setupCollectionView() {
    let layout = UICollectionViewFlowLayout()
    layout.scrollDirection = .vertical
    layout.minimumLineSpacing = 16
    layout.minimumInteritemSpacing = 12
    layout.sectionInset = UIEdgeInsets(top: 16, left: 16, bottom: 16, right: 16)

    collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
    collectionView.translatesAutoresizingMaskIntoConstraints = false
    collectionView.backgroundColor = .systemBackground
    view.addSubview(collectionView)

    NSLayoutConstraint.activate([
        collectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
        collectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
        collectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        collectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
    ])

    collectionView.register(StudentGridCell.self, forCellWithReuseIdentifier: StudentGridCell.reuseID)
    collectionView.dataSource = self
    collectionView.delegate = self
}
```

---

## 🧱 4. Custom Cell cho CollectionView

**StudentGridCell.swift**

```swift
import UIKit

final class StudentGridCell: UICollectionViewCell {
    static let reuseID = "StudentGridCell"

    private let avatarImageView = UIImageView()
    private let nameLabel = UILabel()
    private let gradeLabel = UILabel()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private func setupUI() {
        avatarImageView.translatesAutoresizingMaskIntoConstraints = false
        avatarImageView.layer.cornerRadius = 35
        avatarImageView.clipsToBounds = true
        avatarImageView.contentMode = .scaleAspectFill
        avatarImageView.tintColor = .systemBlue
        avatarImageView.backgroundColor = .systemGray5

        nameLabel.font = .preferredFont(forTextStyle: .headline)
        nameLabel.textAlignment = .center
        gradeLabel.font = .preferredFont(forTextStyle: .subheadline)
        gradeLabel.textColor = .secondaryLabel
        gradeLabel.textAlignment = .center

        let stack = UIStackView(arrangedSubviews: [avatarImageView, nameLabel, gradeLabel])
        stack.axis = .vertical
        stack.spacing = 6
        stack.alignment = .center
        stack.translatesAutoresizingMaskIntoConstraints = false

        contentView.addSubview(stack)
        contentView.layer.cornerRadius = 12
        contentView.backgroundColor = .secondarySystemBackground

        NSLayoutConstraint.activate([
            avatarImageView.widthAnchor.constraint(equalToConstant: 70),
            avatarImageView.heightAnchor.constraint(equalToConstant: 70),
            stack.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 10),
            stack.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 8),
            stack.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -8),
            stack.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -10)
        ])
    }

    func configure(with student: StudentEntity) {
        nameLabel.text = student.name
        gradeLabel.text = "Điểm: \(student.grade)"

        let symbol = student.grade >= 8 ? "star.circle.fill"
                    : student.grade >= 5 ? "person.circle"
                    : "exclamationmark.circle"
        avatarImageView.image = UIImage(systemName: symbol)
        avatarImageView.tintColor = student.grade >= 8 ? .systemYellow :
                                    student.grade >= 5 ? .systemBlue :
                                    .systemRed
    }

    override var isHighlighted: Bool {
        didSet {
            UIView.animate(withDuration: 0.15) {
                self.transform = self.isHighlighted ? CGAffineTransform(scaleX: 0.95, y: 0.95) : .identity
            }
        }
    }
}
```

💡 `isHighlighted` giúp tạo hiệu ứng “nhấn xuống” khi người dùng chạm vào item.

---

## ⚡ 5. Hiển thị dữ liệu

```swift
extension StudentGridViewController: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        fetchedResultsController.fetchedObjects?.count ?? 0
    }

    func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: StudentGridCell.reuseID,
            for: indexPath
        ) as? StudentGridCell else { return UICollectionViewCell() }

        let student = fetchedResultsController.object(at: indexPath)
        cell.configure(with: student)
        return cell
    }
}
```

---

## 🧩 6. Cấu hình kích thước cell

Thêm vào `UICollectionViewDelegateFlowLayout`:

```swift
extension StudentGridViewController: UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        sizeForItemAt indexPath: IndexPath) -> CGSize {
        let screenWidth = UIScreen.main.bounds.width
        let padding: CGFloat = 16 * 3 // lề + khoảng cách
        let availableWidth = screenWidth - padding
        let widthPerItem = availableWidth / 2 // 2 cột
        return CGSize(width: widthPerItem, height: 150)
    }

    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        let student = fetchedResultsController.object(at: indexPath)
        let detailVC = StudentDetailViewController(student: student)
        navigationController?.pushViewController(detailVC, animated: true)
    }
}
```

---

## 💡 7. Dữ liệu Core Data

Sử dụng lại cấu trúc FRC từ các bài trước:

```swift
private func setupFetchedResultsController() {
    let request: NSFetchRequest<StudentEntity> = StudentEntity.fetchRequest()
    request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
    fetchedResultsController = NSFetchedResultsController(
        fetchRequest: request,
        managedObjectContext: CoreDataManager.shared.context,
        sectionNameKeyPath: nil,
        cacheName: nil
    )

    do {
        try fetchedResultsController.performFetch()
    } catch {
        print("Lỗi fetch: \(error.localizedDescription)")
    }
    collectionView.reloadData()
}
```

---

## 🧪 8. Kiểm thử kết quả

1️⃣ Chạy app → vào tab hoặc màn “Lưới học sinh”.
2️⃣ Thấy danh sách hiển thị **2 cột**, có ảnh, tên, điểm.
3️⃣ Chạm vào một học sinh → mở màn chi tiết (Bài 13).
4️⃣ Vuốt lên xuống mượt, hỗ trợ Dark Mode.
5️⃣ Cell có hiệu ứng “nhấn” khi chạm 🎉

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #15)** 🎒

| Mức độ        | Bài tập                                 | Gợi ý                                                 |
| ------------- | --------------------------------------- | ----------------------------------------------------- |
| 🟢 Cơ bản     | Hiển thị danh sách học sinh dạng 2 cột  | `UICollectionViewFlowLayout`                          |
| 🟡 Trung bình | Tùy chỉnh cell có ảnh + tên + điểm      | Custom `UICollectionViewCell`                         |
| 🔵 Nâng cao   | Tạo hiệu ứng chạm và Dark Mode          | override `isHighlighted`, dùng `systemBackground`     |
| 🟣 Thử thách  | Chuyển layout sang 3 cột khi xoay ngang | `traitCollectionDidChange()` hoặc tính lại kích thước |

---

## 📚 Tổng kết

| Thành phần                 | Vai trò                                       |
| -------------------------- | --------------------------------------------- |
| UICollectionView           | Hiển thị dữ liệu dạng lưới                    |
| UICollectionViewFlowLayout | Quy định layout (số cột, spacing, kích thước) |
| UICollectionViewCell       | Thành phần hiển thị mỗi item                  |
| Auto Layout                | Giúp cell co giãn linh hoạt                   |
| didSelectItemAt            | Xử lý chọn học sinh                           |
| isHighlighted              | Tạo hiệu ứng nhấn                             |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Cấu trúc CollectionView và layout cơ bản.
* Tạo custom cell dạng card đẹp mắt.
* Hiển thị danh sách học sinh dạng lưới 2 cột.
* Mở chi tiết học sinh khi chạm.

---

🎓 **UIKit – Bài 16 (buổi tới):**

> *CollectionView nâng cao: Custom Flow Layout, Grid động, Header/Footer, và Diffable Data Source (modern API).*

Thầy sẽ hướng dẫn:

* Tạo layout linh hoạt (grid 2–3–4 cột tự điều chỉnh).
* Thêm header “Lớp Giỏi / Khá / Yếu”.
* Dùng API mới `UICollectionViewDiffableDataSource` (iOS 13+).

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 16: CollectionView nâng cao (Dynamic Grid + Header/Footer + Diffable Data Source)** không?
