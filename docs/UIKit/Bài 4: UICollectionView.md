Tốt lắm 👏👏👏 — lớp mình đã nắm chắc **UITableView**, hôm nay sang bài cực kỳ hay và trực quan:

# 🧩 UIKit – Bài 4: UICollectionView (Lưới dữ liệu, layout linh hoạt, 100% code)

---

## 🎯 Mục tiêu bài học

Sau buổi này, bạn sẽ:

1. Hiểu cách hoạt động của `UICollectionView` và **UICollectionViewFlowLayout**.
2. Biết cách tạo layout lưới (grid) bằng code.
3. Tạo cell tùy chỉnh, hiển thị ảnh + text.
4. Biết xử lý chọn item (`didSelectItemAt`).
5. Nắm cách thay đổi spacing, insets, và số cột.

---

## 🧠 1. UICollectionView là gì?

> `UICollectionView` tương tự `UITableView`, nhưng linh hoạt hơn —
> có thể hiển thị **nhiều cột, hàng ngang, hoặc layout tùy biến** (gallery, grid, photo wall,…)

Thành phần chính:

| Thành phần                   | Vai trò                                                    |
| ---------------------------- | ---------------------------------------------------------- |
| `UICollectionView`           | Hiển thị danh sách hoặc lưới                               |
| `UICollectionViewCell`       | Đơn vị hiển thị từng phần tử                               |
| `UICollectionViewFlowLayout` | Quy định cách xếp các cell (khoảng cách, số cột, hướng, …) |

---

## ⚙️ 2. Tạo Controller cơ bản

**StudentGridViewController.swift**

```swift
import UIKit

final class StudentGridViewController: UIViewController {
    private var collectionView: UICollectionView!

    // Dữ liệu mẫu
    private let students = [
        "Mai Lê", "Thành Công", "Hoàng Cường", "Ngọc Lan",
        "Bảo Anh", "Hải Nam", "Linh Chi", "Phương Hà", "Minh Quân", "Khánh Vy"
    ]

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Bảng học sinh"
        view.backgroundColor = .systemBackground
        setupCollectionView()
    }

    private func setupCollectionView() {
        // B1: Khởi tạo layout
        let layout = UICollectionViewFlowLayout()
        layout.scrollDirection = .vertical
        layout.minimumLineSpacing = 12
        layout.minimumInteritemSpacing = 12
        layout.sectionInset = UIEdgeInsets(top: 16, left: 16, bottom: 16, right: 16)

        // B2: Tạo collection view
        collectionView = UICollectionView(frame: .zero, collectionViewLayout: layout)
        collectionView.translatesAutoresizingMaskIntoConstraints = false
        collectionView.backgroundColor = .systemBackground

        // B3: Thêm vào view và ràng buộc
        view.addSubview(collectionView)
        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])

        // B4: Đăng ký cell
        collectionView.register(StudentGridCell.self, forCellWithReuseIdentifier: StudentGridCell.reuseID)

        // B5: Gán delegate & datasource
        collectionView.dataSource = self
        collectionView.delegate = self
    }
}
```

---

## 🧱 3. Tạo Cell tùy chỉnh

**StudentGridCell.swift**

```swift
import UIKit

final class StudentGridCell: UICollectionViewCell {
    static let reuseID = "StudentGridCell"

    private let imageView = UIImageView()
    private let nameLabel = UILabel()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private func setupUI() {
        imageView.image = UIImage(systemName: "person.circle.fill")
        imageView.tintColor = .systemBlue
        imageView.contentMode = .scaleAspectFit
        imageView.translatesAutoresizingMaskIntoConstraints = false
        imageView.heightAnchor.constraint(equalToConstant: 60).isActive = true
        imageView.widthAnchor.constraint(equalToConstant: 60).isActive = true

        nameLabel.font = .systemFont(ofSize: 16, weight: .medium)
        nameLabel.textAlignment = .center
        nameLabel.numberOfLines = 2

        let stack = UIStackView(arrangedSubviews: [imageView, nameLabel])
        stack.axis = .vertical
        stack.alignment = .center
        stack.spacing = 8
        stack.translatesAutoresizingMaskIntoConstraints = false

        contentView.addSubview(stack)
        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 8),
            stack.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 8),
            stack.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -8),
            stack.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -8)
        ])

        contentView.backgroundColor = .secondarySystemBackground
        contentView.layer.cornerRadius = 12
    }

    func configure(name: String) {
        nameLabel.text = name
    }

    override var isHighlighted: Bool {
        didSet { contentView.alpha = isHighlighted ? 0.6 : 1.0 }
    }
}
```

---

## 🧩 4. Gán DataSource & Delegate

**StudentGridViewController.swift (phần mở rộng)**

```swift
extension StudentGridViewController: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        students.count
    }

    func collectionView(_ collectionView: UICollectionView,
                        cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: StudentGridCell.reuseID,
            for: indexPath
        ) as? StudentGridCell else { return UICollectionViewCell() }

        cell.configure(name: students[indexPath.item])
        return cell
    }
}

extension StudentGridViewController: UICollectionViewDelegateFlowLayout {
    // Cấu hình kích thước cell (3 cột)
    func collectionView(_ collectionView: UICollectionView,
                        layout collectionViewLayout: UICollectionViewLayout,
                        sizeForItemAt indexPath: IndexPath) -> CGSize {
        let padding: CGFloat = 16 * 2 + 12 * 2 // insets + spacing
        let availableWidth = collectionView.frame.width - padding
        let width = availableWidth / 3
        return CGSize(width: width, height: 120)
    }

    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        let name = students[indexPath.item]
        print("Bạn chọn: \(name)")
    }
}
```

---

## ⚡ 5. Giải thích chi tiết

| Thành phần                   | Vai trò                                     |
| ---------------------------- | ------------------------------------------- |
| `UICollectionViewFlowLayout` | Layout mặc định, sắp xếp theo dòng & cột    |
| `sizeForItemAt`              | Quy định kích thước cell                    |
| `minimumLineSpacing`         | Khoảng cách giữa hàng                       |
| `minimumInteritemSpacing`    | Khoảng cách giữa cột                        |
| `sectionInset`               | Khoảng cách lề toàn khu vực                 |
| `dequeueReusableCell`        | Tái sử dụng cell – bắt buộc đăng ký reuseID |
| `didSelectItemAt`            | Xử lý khi chọn item                         |

---

## 🧪 6. Mini Project – Grid học sinh

Khi chạy code:

* Mỗi ô hiển thị icon + tên.
* Tự động giãn ra 3 cột đều nhau.
* Bấm chọn in ra console “Bạn chọn: Tên”.

👉 Sau này, em có thể mở rộng:

* Thay hình `UIImage(systemName:)` bằng ảnh thực (`UIImage(named:)`)
* Gắn `navigationController?.pushViewController(...)` để đi chi tiết.

---

## 🧱 7. Tuỳ chỉnh giao diện nâng cao

### 🔹 Màu ngẫu nhiên mỗi cell:

```swift
contentView.backgroundColor = UIColor(
    hue: CGFloat.random(in: 0...1),
    saturation: 0.2,
    brightness: 0.95,
    alpha: 1
)
```

### 🔹 Shadow nhẹ:

```swift
contentView.layer.shadowColor = UIColor.black.cgColor
contentView.layer.shadowOpacity = 0.1
contentView.layer.shadowRadius = 3
contentView.layer.shadowOffset = .init(width: 0, height: 2)
```

### 🔹 Khi chọn → animation:

```swift
override var isSelected: Bool {
    didSet {
        UIView.animate(withDuration: 0.2) {
            self.contentView.transform = self.isSelected ? CGAffineTransform(scaleX: 0.95, y: 0.95) : .identity
        }
    }
}
```

---

## 🏠 BÀI TẬP VỀ NHÀ (UIKit #4) 🎒

| Mức độ        | Đề bài                                                       | Gợi ý                                              |
| ------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| 🟢 Cơ bản     | Tạo 2 cột thay vì 3 cột                                      | Sửa phép chia trong `sizeForItemAt`                |
| 🟡 Trung bình | Thêm `sectionHeader` với tiêu đề “Lớp A”, “Lớp B”            | Dùng `UICollectionReusableView`                    |
| 🔵 Nâng cao   | Khi chọn cell, push sang màn chi tiết có ảnh + text          | Dùng `navigationController?.pushViewController()`  |
| 🟣 Thử thách  | Làm app “Photo Grid”: 30 ảnh, cuộn mượt, chiều cao linh hoạt | Dùng `UIImage(named:)` + `aspectRatio` constraints |

---

## 📚 Tổng kết kiến thức

| Chủ đề                | Tóm tắt                            |
| --------------------- | ---------------------------------- |
| UICollectionView      | Danh sách + lưới                   |
| FlowLayout            | Layout theo hàng/cột               |
| Cell tùy chỉnh        | Kế thừa `UICollectionViewCell`     |
| Delegate & DataSource | Cung cấp dữ liệu + xử lý tương tác |
| Auto Layout bằng code | Giúp cell tự động co giãn          |

---

## 🧭 Kết thúc bài học

✅ Bạn đã:

* Tạo CollectionView 100% code.
* Hiểu FlowLayout và spacing.
* Biết viết cell tùy chỉnh (ảnh + text).
* Làm mini project: danh sách học sinh dạng lưới.

---

🎓 **UIKit – Bài 5 (kế tiếp):**

> *Navigation nâng cao, TabBarController, và truyền dữ liệu giữa ViewController (delegate, closure, notification).*

Bài 5 sẽ giúp em **chuyển dữ liệu 2 chiều** và **tổ chức app nhiều màn hình chuyên nghiệp** (chuẩn iOS project structure).

---

Thầy hỏi nhẹ:
👉 Em có muốn sang luôn **UIKit – Bài 5: Navigation & Data Passing nâng cao** ngay không, để bắt đầu phần “App nhiều màn hình” chuyên nghiệp?
