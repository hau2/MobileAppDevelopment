Rất tốt 👏👏👏 — hôm nay chúng ta bước sang **Tuần 1 – Ngày 8: Màn hình chi tiết sản phẩm (Product Detail Screen) + Truyền dữ liệu giữa ViewController qua Coordinator** 🧭

Đây là **bước hoàn thiện đầu tiên của app thương mại điện tử mini** — app em sẽ hiển thị **danh sách sản phẩm → nhấn vào 1 sản phẩm → xem chi tiết đầy đủ (ảnh lớn, tên, mô tả, giá)**.
Đồng thời hôm nay em học **cách truyền dữ liệu giữa các màn hình một cách chuyên nghiệp** thông qua *Coordinator pattern* (đã học ở bài 4).

---

# 🧩 UIKit – Tuần 1, Ngày 8: Product Detail + Data Flow qua Coordinator

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ:

1. Hiểu rõ cách truyền dữ liệu giữa các màn hình.
2. Biết cách mở màn hình chi tiết bằng Coordinator.
3. Hiển thị chi tiết sản phẩm (ảnh lớn, mô tả, giá, category).
4. Áp dụng hoàn chỉnh flow: **List → Detail**.

---

## 🧱 1. Cấu trúc dự án hôm nay

```
ProductApp
├── Models
│   └── Product.swift
├── Services
│   └── NetworkService.swift
│   └── ImageLoader.swift
├── ViewModels
│   ├── ProductListViewModel.swift
│   └── ProductDetailViewModel.swift   👈 (mới)
├── Views
│   ├── ProductCell.swift
│   └── ProductDetailView.swift        👈 (mới)
├── Screens
│   ├── ProductListViewController.swift
│   └── ProductDetailViewController.swift 👈 (mới)
└── Coordinators
    └── AppCoordinator.swift
```

---

## 👩‍🏫 2. Ôn lại cách truyền dữ liệu giữa các màn hình

### ❌ Cách cũ (khi chưa có Coordinator)

```swift
let detailVC = ProductDetailViewController(product: product)
navigationController?.pushViewController(detailVC, animated: true)
```

→ ViewController **tự tạo, tự điều hướng**, dễ rối khi app lớn.

---

### ✅ Cách chuẩn (dùng Coordinator)

```swift
listVC.onSelectProduct = { [weak self] product in
    self?.showProductDetail(product)
}
```

→ ViewController chỉ báo “người dùng chọn sản phẩm này”,
→ Coordinator **nhận và điều hướng** tới màn chi tiết.

---

## ⚙️ 3. Tạo `ProductDetailViewModel.swift`

```swift
import Foundation

final class ProductDetailViewModel {
    private let product: Product

    init(product: Product) {
        self.product = product
    }

    var title: String {
        product.title
    }

    var price: String {
        "$\(String(format: "%.2f", product.price))"
    }

    var description: String {
        product.description
    }

    var category: String {
        "Danh mục: \(product.category)"
    }

    var imageURL: String {
        product.image
    }
}
```

💡 ViewModel giữ toàn bộ dữ liệu và format sẵn để hiển thị lên View.

---

## 🧩 4. Tạo giao diện chi tiết `ProductDetailView.swift`

```swift
import UIKit

final class ProductDetailView: UIView {
    let imageView = UIImageView()
    let titleLabel = UILabel()
    let priceLabel = UILabel()
    let categoryLabel = UILabel()
    let descriptionLabel = UILabel()
    let scrollView = UIScrollView()
    let contentStack = UIStackView()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }

    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private func setupUI() {
        backgroundColor = .systemBackground

        // Ảnh sản phẩm
        imageView.contentMode = .scaleAspectFit
        imageView.layer.cornerRadius = 8
        imageView.clipsToBounds = true
        imageView.heightAnchor.constraint(equalToConstant: 250).isActive = true

        // Tiêu đề
        titleLabel.font = .systemFont(ofSize: 20, weight: .semibold)
        titleLabel.numberOfLines = 0
        titleLabel.textAlignment = .center

        // Giá
        priceLabel.font = .systemFont(ofSize: 18, weight: .bold)
        priceLabel.textColor = .systemGreen
        priceLabel.textAlignment = .center

        // Danh mục
        categoryLabel.font = .systemFont(ofSize: 16, weight: .medium)
        categoryLabel.textColor = .secondaryLabel
        categoryLabel.textAlignment = .center

        // Mô tả
        descriptionLabel.font = .systemFont(ofSize: 15)
        descriptionLabel.numberOfLines = 0
        descriptionLabel.textAlignment = .justified

        // Stack chứa nội dung
        contentStack.axis = .vertical
        contentStack.spacing = 12
        contentStack.alignment = .fill
        contentStack.translatesAutoresizingMaskIntoConstraints = false
        contentStack.addArrangedSubview(imageView)
        contentStack.addArrangedSubview(titleLabel)
        contentStack.addArrangedSubview(priceLabel)
        contentStack.addArrangedSubview(categoryLabel)
        contentStack.addArrangedSubview(descriptionLabel)

        scrollView.translatesAutoresizingMaskIntoConstraints = false
        scrollView.addSubview(contentStack)
        addSubview(scrollView)

        NSLayoutConstraint.activate([
            scrollView.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor),
            scrollView.leadingAnchor.constraint(equalTo: leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: trailingAnchor),
            scrollView.bottomAnchor.constraint(equalTo: bottomAnchor),

            contentStack.topAnchor.constraint(equalTo: scrollView.topAnchor, constant: 20),
            contentStack.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor, constant: 16),
            contentStack.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor, constant: -16),
            contentStack.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor, constant: -20),
            contentStack.widthAnchor.constraint(equalTo: scrollView.widthAnchor, constant: -32)
        ])
    }

    func configure(with viewModel: ProductDetailViewModel) {
        titleLabel.text = viewModel.title
        priceLabel.text = viewModel.price
        categoryLabel.text = viewModel.category
        descriptionLabel.text = viewModel.description

        imageView.image = UIImage(systemName: "photo")
        ImageLoader.shared.loadImage(from: viewModel.imageURL) { [weak self] image in
            self?.imageView.image = image
        }
    }
}
```

💬 Giải thích:

* `UIScrollView` để màn hình chi tiết có thể cuộn.
* `UIStackView` giúp sắp xếp các thành phần gọn gàng.
* Hàm `configure(with:)` nhận dữ liệu từ ViewModel và hiển thị.

---

## 📱 5. Tạo `ProductDetailViewController.swift`

```swift
import UIKit

final class ProductDetailViewController: UIViewController {
    private let contentView = ProductDetailView()
    private let viewModel: ProductDetailViewModel

    init(viewModel: ProductDetailViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    override func loadView() {
        view = contentView
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Chi tiết sản phẩm"
        contentView.configure(with: viewModel)
    }
}
```

---

## 🧭 6. Cập nhật `AppCoordinator.swift`

```swift
final class AppCoordinator: Coordinator {
    var navigationController: UINavigationController
    private let networkService: NetworkServiceProtocol

    init(navigationController: UINavigationController, networkService: NetworkServiceProtocol) {
        self.navigationController = navigationController
        self.networkService = networkService
    }

    func start() {
        showProductList()
    }

    private func showProductList() {
        let viewModel = ProductListViewModel(networkService: networkService)
        let listVC = ProductListViewController(viewModel: viewModel)

        listVC.onSelectProduct = { [weak self] product in
            self?.showProductDetail(product)
        }

        navigationController.pushViewController(listVC, animated: false)
    }

    private func showProductDetail(_ product: Product) {
        let detailVM = ProductDetailViewModel(product: product)
        let detailVC = ProductDetailViewController(viewModel: detailVM)
        navigationController.pushViewController(detailVC, animated: true)
    }
}
```

---

## 📘 7. Cập nhật `ProductListViewController.swift`

Thêm closure báo cho Coordinator khi người dùng chọn sản phẩm:

```swift
final class ProductListViewController: UIViewController {
    var onSelectProduct: ((Product) -> Void)?

    // ... các phần cũ giữ nguyên

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let product = viewModel.products.value[indexPath.row]
        onSelectProduct?(product)
    }
}
```

---

## 🧪 8. Chạy thử 🚀

✅ Mở app → danh sách sản phẩm xuất hiện.
✅ Chạm vào 1 sản phẩm → chuyển sang màn chi tiết hiển thị ảnh lớn, tên, giá, mô tả, danh mục.
✅ Quay lại → vẫn giữ nguyên vị trí danh sách (Coordinator giữ state).
✅ Mọi thứ hoạt động **rất mượt và chuyên nghiệp**.

🎉 Em vừa hoàn thiện **flow cơ bản của một e-commerce app thật**:
**List → Detail**, tách biệt rõ **ViewModel / View / Coordinator**.

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 8)** 🎒

| Mức độ        | Bài tập                                                    | Gợi ý                                |
| ------------- | ---------------------------------------------------------- | ------------------------------------ |
| 🟢 Cơ bản     | Thêm nút “Mua ngay” dưới phần mô tả                        | UIButton trong `ProductDetailView`   |
| 🟡 Trung bình | Khi nhấn “Mua ngay”, hiển thị `UIAlertController` xác nhận |                                      |
| 🔵 Nâng cao   | Làm hiệu ứng phóng to ảnh khi nhấn (Zoom in)               | `UITapGestureRecognizer` + Animation |
| 🟣 Thử thách  | Dùng Coordinator để mở màn “Giỏ hàng”                      | tạo `CartCoordinator` riêng          |

---

## 📚 Tổng kết buổi học

| Thành phần              | Vai trò                             |
| ----------------------- | ----------------------------------- |
| **Coordinator**         | Quản lý điều hướng (List → Detail)  |
| **ViewModel**           | Cung cấp dữ liệu cho View           |
| **View (UIView riêng)** | Hiển thị dữ liệu (UI thuần code)    |
| **Closure Callback**    | Gửi tín hiệu giữa VC và Coordinator |
| **Async Image Loading** | Hiển thị ảnh lớn từ URL             |

---

🎓 **Ngày 9 (buổi tới):**

> *Thầy sẽ dạy “Tạo màn hình thêm sản phẩm (Add Product) + truyền dữ liệu ngược từ Detail về List bằng closure hoặc delegate” — đây là nền tảng để hiểu cơ chế dữ liệu hai chiều trong app thực tế.*

---

👉 Em có muốn thầy dạy luôn **Ngày 9 – Màn thêm sản phẩm & truyền dữ liệu ngược (Delegate/Closure)** không?
Thầy sẽ đi chậm, có ví dụ minh họa rõ ràng để em nắm chắc cơ chế “truyền dữ liệu ngược giữa các màn hình”.
