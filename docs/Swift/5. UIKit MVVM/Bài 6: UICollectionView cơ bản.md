Ok, qua **Bài 6 – UICollectionView cơ bản** 👨‍🏫
Bài này cực quan trọng vì UICollectionView là nền tảng cho:

* Grid layout
* Gallery
* IG/FB style feed
* Horizontal carousel
* Layout động
* Section-based UI (sau này Combine + Diffable Data Source)

Thầy sẽ dạy **chậm – dễ hiểu – code đầy đủ – comment chi tiết**.

---

# 🎯 MỤC TIÊU BÀI 6

Sau bài này, em sẽ biết:

✔ Tạo UICollectionView bằng code
✔ Hiểu `UICollectionViewFlowLayout`
✔ Tạo custom cell
✔ Layout dạng **grid 2 cột** phổ biến
✔ Hiển thị dữ liệu cơ bản
✔ Copy/paste chạy được ngay

---

# 🧱 1. Tạo file Cell riêng (ProductGridCell.swift)

Trong Xcode → **File → New → File… → Swift File**
Đặt tên: **ProductGridCell.swift**

Copy toàn bộ code sau:

```swift
import UIKit

/// Custom cell dạng grid: gồm hình + tên sản phẩm.
/// Sau này ta có thể thêm giá, rating, nút, v.v.
class ProductGridCell: UICollectionViewCell {

    // MARK: - UI Components

    /// Ảnh sản phẩm (tạm icon hệ thống)
    private let productImageView: UIImageView = {
        let imageView = UIImageView()
        imageView.image = UIImage(systemName: "photo")
        imageView.contentMode = .scaleAspectFit
        imageView.tintColor = .systemBlue
        imageView.clipsToBounds = true
        imageView.backgroundColor = .systemGray6
        imageView.layer.cornerRadius = 8
        return imageView
    }()

    /// Tên sản phẩm
    private let nameLabel: UILabel = {
        let label = UILabel()
        label.font = .systemFont(ofSize: 15, weight: .medium)
        label.numberOfLines = 2
        label.textAlignment = .center
        return label
    }()

    // MARK: - Init

    override init(frame: CGRect) {
        super.init(frame: frame)

        // Thêm ui vào contentView
        contentView.addSubview(productImageView)
        contentView.addSubview(nameLabel)

        setupConstraints()

        // Style cell
        contentView.layer.cornerRadius = 10
        contentView.backgroundColor = .secondarySystemBackground
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - Constraints

    private func setupConstraints() {
        productImageView.translatesAutoresizingMaskIntoConstraints = false
        nameLabel.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            // Ảnh nằm trên, chiếm 70% chiều cao cell
            productImageView.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 8),
            productImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 8),
            productImageView.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -8),
            productImageView.heightAnchor.constraint(equalTo: contentView.heightAnchor, multiplier: 0.65),

            // Label nằm dưới ảnh
            nameLabel.topAnchor.constraint(equalTo: productImageView.bottomAnchor, constant: 6),
            nameLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 4),
            nameLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -4),
            nameLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -6)
        ])
    }

    // MARK: - Public Method

    func configure(name: String) {
        nameLabel.text = name
    }
}
```

---

# 🧱 2. Cập nhật ViewController để dùng CollectionView

Thay file **ViewController.swift** bằng code sau:

```swift
import UIKit

class ViewController: UIViewController {

    // MARK: - Dummy Data
    private let products: [String] = [
        "iPhone 16 Pro Max",
        "MacBook Pro M4",
        "iPad Pro OLED",
        "AirPods Pro 3",
        "Vision Pro",
        "Apple Watch S10",
        "HomePod Mini",
        "Magic Keyboard",
        "Magic Mouse",
        "Apple TV 4K",
        "iMac M4",
        "Mac Studio"
    ]

    // MARK: - CollectionView Layout

    /// CollectionView Flow Layout = kiểu layout dạng grid/hình chữ nhật
    private let layout: UICollectionViewFlowLayout = {
        let layout = UICollectionViewFlowLayout()

        layout.scrollDirection = .vertical   // cuộn dọc
        layout.minimumLineSpacing = 16       // khoảng cách giữa các hàng
        layout.minimumInteritemSpacing = 12  // khoảng cách giữa các cột
        layout.sectionInset = UIEdgeInsets(top: 16, left: 16, bottom: 16, right: 16)

        return layout
    }()

    // MARK: - CollectionView

    private lazy var collectionView: UICollectionView = {
        let cv = UICollectionView(frame: .zero, collectionViewLayout: layout)
        cv.backgroundColor = .systemBackground

        // Đăng ký custom cell
        cv.register(ProductGridCell.self, forCellWithReuseIdentifier: "ProductGridCell")

        cv.dataSource = self
        cv.delegate = self

        return cv
    }()

    // MARK: - View Lifecycle

    override func viewDidLoad() {
        super.viewDidLoad()

        title = "Bài 6 - UICollectionView"
        view.backgroundColor = .systemBackground

        view.addSubview(collectionView)
        setupConstraints()
    }

    private func setupConstraints() {
        collectionView.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
}

// MARK: - UICollectionViewDataSource

extension ViewController: UICollectionViewDataSource {

    /// Số lượng cell
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return products.count
    }

    /// Tạo cell + gán dữ liệu
    func collectionView(
        _ collectionView: UICollectionView,
        cellForItemAt indexPath: IndexPath
    ) -> UICollectionViewCell {

        guard let cell = collectionView.dequeueReusableCell(
            withReuseIdentifier: "ProductGridCell",
            for: indexPath
        ) as? ProductGridCell else {
            return UICollectionViewCell()
        }

        let productName = products[indexPath.item]
        cell.configure(name: productName)

        return cell
    }
}

// MARK: - UICollectionViewDelegateFlowLayout

extension ViewController: UICollectionViewDelegateFlowLayout {

    /// Kích thước cell (grid 2 cột)
    func collectionView(
        _ collectionView: UICollectionView,
        layout collectionViewLayout: UICollectionViewLayout,
        sizeForItemAt indexPath: IndexPath
    ) -> CGSize {

        // Tính toán width cho 2 cột
        let totalSpacing: CGFloat = 16 + 16 + 12 // left + right + spacing giữa 2 cột
        let width = (collectionView.frame.width - totalSpacing) / 2

        return CGSize(width: width, height: width * 1.4)
    }

    /// Khi tap vào cell
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        print("Tapped:", products[indexPath.item])
    }
}
```

---

# 🧪 KẾT QUẢ CHẠY

✔ Grid 2 cột đẹp
✔ Cell có ảnh + tên
✔ Spacing chuẩn 16–12–16
✔ Tự cuộn mượt
✔ Tap vào cell → in log

---

# 🎁 Bài tập mở rộng

1️⃣ **Thêm giá vào cell**
→ chỉnh lại UI + AutoLayout

2️⃣ **Thêm shadow cho cell**

```swift
contentView.layer.shadowColor = UIColor.black.cgColor
contentView.layer.shadowOpacity = 0.1
contentView.layer.shadowRadius = 4
```

3️⃣ **Chuyển layout sang horizontal (carousel)**

```swift
layout.scrollDirection = .horizontal
```

4️⃣ **Thêm highlight khi chọn cell**
Override:

```swift
override var isHighlighted: Bool {
    didSet { ... }
}
```

---

# 🎯 Tiếp theo em muốn học gì?

👉 **Bài 7 – Closure Pattern (cơ bản → nâng cao)**
👉 **hay muốn học Combine trước?**

Chỉ cần nói:
**“Qua bài 7 đi thầy”**
