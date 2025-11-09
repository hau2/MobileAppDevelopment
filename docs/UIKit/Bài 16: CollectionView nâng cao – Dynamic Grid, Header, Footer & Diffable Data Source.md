Tuyệt vời 👏👏👏 — hôm nay thầy và em cùng bước sang **UIKit – Bài 16: CollectionView nâng cao – Dynamic Grid, Header/Footer & Diffable Data Source (API hiện đại)**

Đây là **bước chuyển từ “làm thủ công” sang “làm chuẩn Apple Developer thời mới”**,
vì em sẽ học cách:

> Tạo **layout động (tự co giãn, chia cột thông minh)**, thêm **Header/Footer đẹp mắt**,
> và dùng **Diffable Data Source** — công nghệ giúp CollectionView cập nhật dữ liệu mượt mà, không cần gọi `reloadData()`.

---

# 🧩 UIKit – Bài 16: CollectionView nâng cao (Dynamic Grid + Header/Footer + Diffable)

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ biết:

1. Dùng **UICollectionViewCompositionalLayout** để tạo layout động (auto grid).
2. Tạo **Header/Footer** trong CollectionView.
3. Hiểu và sử dụng **Diffable Data Source** để quản lý dữ liệu dễ dàng hơn.
4. Thêm/xoá/cập nhật học sinh bằng animation tự động.

---

## 🧠 1. Giới thiệu về Diffable Data Source

> `UICollectionViewDiffableDataSource` là API hiện đại thay thế cho datasource truyền thống.
> Nó cho phép **tự động tính toán thay đổi (insert/delete/move)** và cập nhật CollectionView bằng animation.

Ví dụ:

```swift
var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
snapshot.appendSections([.main])
snapshot.appendItems(students)
dataSource.apply(snapshot, animatingDifferences: true)
```

✅ Không cần gọi `reloadData()`
✅ Animation mượt
✅ Dễ bảo trì

---

## 🧩 2. Cấu trúc dữ liệu cho bài học

```swift
enum Section: String, CaseIterable {
    case gioi = "Giỏi (>=8)"
    case kha = "Khá (5–7)"
    case yeu = "Yếu (<5)"
}
```

Mỗi section là một **nhóm học sinh** theo loại điểm.

---

## ⚙️ 3. Tạo CollectionView với Compositional Layout

**StudentGridAdvancedViewController.swift**

```swift
import UIKit
import CoreData

final class StudentGridAdvancedViewController: UIViewController {
    enum Section: String, CaseIterable {
        case gioi = "Giỏi (>=8)"
        case kha = "Khá (5–7)"
        case yeu = "Yếu (<5)"
    }

    private var collectionView: UICollectionView!
    private var dataSource: UICollectionViewDiffableDataSource<Section, NSManagedObjectID>!
    private var fetchedResultsController: NSFetchedResultsController<StudentEntity>!

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Phân loại học sinh"
        view.backgroundColor = .systemBackground
        configureCollectionView()
        configureDataSource()
        loadData()
    }
}
```

---

## 🧩 4. Compositional Layout – Tự động chia cột

```swift
private func configureCollectionView() {
    let layout = UICollectionViewCompositionalLayout { _, _ -> NSCollectionLayoutSection? in
        // Mỗi item chiếm 1/2 chiều rộng hàng
        let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.5),
                                              heightDimension: .estimated(150))
        let item = NSCollectionLayoutItem(layoutSize: itemSize)
        item.contentInsets = NSDirectionalEdgeInsets(top: 8, leading: 8, bottom: 8, trailing: 8)

        // Nhóm gồm 2 item ngang hàng
        let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                               heightDimension: .estimated(180))
        let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])

        let section = NSCollectionLayoutSection(group: group)

        // Header
        let headerSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                                heightDimension: .estimated(44))
        let header = NSCollectionLayoutBoundarySupplementaryItem(
            layoutSize: headerSize,
            elementKind: UICollectionView.elementKindSectionHeader,
            alignment: .top
        )
        section.boundarySupplementaryItems = [header]
        return section
    }

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
    collectionView.register(HeaderReusableView.self,
                            forSupplementaryViewOfKind: UICollectionView.elementKindSectionHeader,
                            withReuseIdentifier: HeaderReusableView.reuseID)
}
```

---

## 🧩 5. Custom Header View

**HeaderReusableView.swift**

```swift
import UIKit

final class HeaderReusableView: UICollectionReusableView {
    static let reuseID = "HeaderReusableView"

    private let titleLabel = UILabel()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private func setupUI() {
        titleLabel.font = .preferredFont(forTextStyle: .headline)
        titleLabel.textColor = .label
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        addSubview(titleLabel)

        NSLayoutConstraint.activate([
            titleLabel.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 16),
            titleLabel.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -16),
            titleLabel.bottomAnchor.constraint(equalTo: bottomAnchor, constant: -4)
        ])
    }

    func configure(title: String) {
        titleLabel.text = title
    }
}
```

---

## ⚡ 6. Cấu hình Diffable Data Source

```swift
private func configureDataSource() {
    dataSource = UICollectionViewDiffableDataSource<Section, NSManagedObjectID>(
        collectionView: collectionView
    ) { collectionView, indexPath, objectID in
        guard let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: StudentGridCell.reuseID,
            for: indexPath
        ) as? StudentGridCell,
        let student = try? CoreDataManager.shared.context.existingObject(with: objectID) as? StudentEntity
        else { return UICollectionViewCell() }

        cell.configure(with: student)
        return cell
    }

    // Header
    dataSource.supplementaryViewProvider = { collectionView, kind, indexPath in
        guard kind == UICollectionView.elementKindSectionHeader,
              let header = collectionView.dequeueReusableSupplementaryView(
                  ofKind: kind,
                  withReuseIdentifier: HeaderReusableView.reuseID,
                  for: indexPath
              ) as? HeaderReusableView else { return nil }

        let section = self.dataSource.snapshot().sectionIdentifiers[indexPath.section]
        header.configure(title: section.rawValue)
        return header
    }
}
```

---

## 🧮 7. Nạp dữ liệu và phân nhóm

```swift
private func loadData() {
    let request: NSFetchRequest<StudentEntity> = StudentEntity.fetchRequest()
    request.sortDescriptors = [NSSortDescriptor(key: "grade", ascending: false)]
    let students = (try? CoreDataManager.shared.context.fetch(request)) ?? []

    var snapshot = NSDiffableDataSourceSnapshot<Section, NSManagedObjectID>()

    let gioi = students.filter { $0.grade >= 8 }.map { $0.objectID }
    let kha = students.filter { $0.grade >= 5 && $0.grade < 8 }.map { $0.objectID }
    let yeu = students.filter { $0.grade < 5 }.map { $0.objectID }

    if !gioi.isEmpty { snapshot.appendSections([.gioi]); snapshot.appendItems(gioi, toSection: .gioi) }
    if !kha.isEmpty { snapshot.appendSections([.kha]); snapshot.appendItems(kha, toSection: .kha) }
    if !yeu.isEmpty { snapshot.appendSections([.yeu]); snapshot.appendItems(yeu, toSection: .yeu) }

    dataSource.apply(snapshot, animatingDifferences: true)
}
```

✅ Diffable Data Source sẽ tự tính toán và hiển thị mượt mà 3 section:

* Giỏi (`>=8`)
* Khá (`5–7`)
* Yếu (`<5`)

---

## 🎮 8. Thêm/xoá học sinh có animation tự động

Khi thêm mới học sinh (ví dụ từ `RegisterViewController`):

```swift
loadData()
```

→ Diffable Data Source tự động:

* thêm item mới,
* chèn vào đúng section,
* và animate chuyển động rất đẹp.

---

## 🧪 9. Kiểm thử thực tế

1️⃣ Mở app → thấy danh sách chia 3 phần rõ ràng: **Giỏi / Khá / Yếu**.
2️⃣ Thêm học sinh mới → app tự xếp vào nhóm tương ứng.
3️⃣ Xoá học sinh → item biến mất mượt mà (animation).
4️⃣ Kéo cuộn mượt, hỗ trợ Dark Mode, font động.
5️⃣ Giao diện layout tự chia 2 cột (ngang dọc tự điều chỉnh). 🎉

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #16)** 🎒

| Mức độ        | Bài tập                                  | Gợi ý                                                             |
| ------------- | ---------------------------------------- | ----------------------------------------------------------------- |
| 🟢 Cơ bản     | Dùng Diffable hiển thị học sinh          | `UICollectionViewDiffableDataSource`                              |
| 🟡 Trung bình | Thêm header section “Giỏi / Khá / Yếu”   | `supplementaryViewProvider`                                       |
| 🔵 Nâng cao   | Chia nhóm động theo điểm                 | `filter` + `snapshot`                                             |
| 🟣 Thử thách  | Khi xoay màn hình -> layout tự chia 3 cột | `UICollectionViewCompositionalLayout` tự động tính lại item width |

---

## 📚 Tổng kết

| Thành phần           | Vai trò                                        |
| -------------------- | ---------------------------------------------- |
| Diffable Data Source | Quản lý dữ liệu động với animation tự động     |
| Snapshot             | Bản sao tạm thời của dữ liệu để áp dụng vào UI |
| Compositional Layout | Tạo layout linh hoạt, nhiều cột                |
| SupplementaryView    | Tạo Header/Footer trong CollectionView         |
| NSManagedObjectID    | Dùng làm định danh ổn định cho Core Data item  |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Tạo layout động (2–3 cột tự động).
* Hiển thị Header Section đẹp.
* Dùng Diffable Data Source cập nhật tự động.
* Quản lý dữ liệu Core Data mượt mà và chuyên nghiệp.

---

🎓 **UIKit – Bài 17 (buổi tới):**

> *Animation & Transition – Hiệu ứng động trong UIKit (UIView.animate, transition, transform, opacity, spring).*

Thầy sẽ dạy:

* Tạo animation khi thêm/xoá item.
* Dùng transform (xoay, phóng to, ẩn hiện).
* Tạo hiệu ứng chuyển cảnh (push, fade, slide).

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 17: Animation & Transition trong UIKit** không?
