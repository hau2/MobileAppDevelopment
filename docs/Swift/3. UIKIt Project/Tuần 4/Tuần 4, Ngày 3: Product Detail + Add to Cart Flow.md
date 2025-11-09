Rất tốt 👏👏👏 — hôm nay ta tiếp tục sang **Tuần 4 – Ngày 3: Chi tiết sản phẩm (Product Detail) + Thêm vào giỏ hàng (Cart Flow)** 🛍️

Đây là **một trong những bài quan trọng nhất** của dự án E-Commerce:

> Sau khi người dùng xem danh sách sản phẩm, họ có thể nhấn vào từng sản phẩm → xem chi tiết, mô tả, hình ảnh, giá, rồi bấm **“Thêm vào giỏ hàng”** 🛒

Thầy sẽ hướng dẫn **chậm rãi, từng bước**, giải thích rõ cả phần **truyền dữ liệu giữa các màn hình**, **hiển thị UI**, và **xử lý logic giỏ hàng**.

---

# 🛒 UIKit – Tuần 4, Ngày 3: Product Detail + Add to Cart Flow

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Cách truyền dữ liệu từ `ProductListViewController` → `ProductDetailViewController`.
2. Tạo giao diện chi tiết sản phẩm hoàn toàn bằng code.
3. Xử lý nút **“Thêm vào giỏ hàng”** và lưu tạm trong CoreData hoặc bộ nhớ.
4. Chuẩn bị sẵn luồng để mở màn hình “Giỏ hàng” ở bài sau.

---

## 🧱 1. Chuẩn bị dữ liệu (Model)

Ta đã có model `Product` rồi (từ các bài trước), cấu trúc như sau:

```swift
struct Product: Codable {
    let id: Int
    let title: String
    let price: Double
    let description: String
    let category: String
    let image: String
}
```

---

## ⚙️ 2. Tạo màn hình chi tiết `ProductDetailViewController.swift`

```swift
import UIKit

final class ProductDetailViewController: UIViewController {

    private let product: Product
    private let scrollView = UIScrollView()
    private let contentView = UIView()

    private let imageView = UIImageView()
    private let titleLabel = UILabel()
    private let priceLabel = UILabel()
    private let descriptionLabel = UILabel()
    private let addButton = UIButton(type: .system)

    init(product: Product) {
        self.product = product
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        title = "Chi tiết sản phẩm"
        setupUI()
        populateData()
    }

    private func setupUI() {
        scrollView.translatesAutoresizingMaskIntoConstraints = false
        contentView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(scrollView)
        scrollView.addSubview(contentView)

        NSLayoutConstraint.activate([
            scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),

            contentView.topAnchor.constraint(equalTo: scrollView.topAnchor),
            contentView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
            contentView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
            contentView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
            contentView.widthAnchor.constraint(equalTo: scrollView.widthAnchor)
        ])

        imageView.contentMode = .scaleAspectFit
        imageView.translatesAutoresizingMaskIntoConstraints = false

        titleLabel.font = .boldSystemFont(ofSize: 20)
        titleLabel.numberOfLines = 0
        titleLabel.translatesAutoresizingMaskIntoConstraints = false

        priceLabel.font = .systemFont(ofSize: 18, weight: .semibold)
        priceLabel.textColor = .systemBlue
        priceLabel.translatesAutoresizingMaskIntoConstraints = false

        descriptionLabel.font = .systemFont(ofSize: 16)
        descriptionLabel.textColor = .darkGray
        descriptionLabel.numberOfLines = 0
        descriptionLabel.translatesAutoresizingMaskIntoConstraints = false

        addButton.setTitle("🛒 Thêm vào giỏ hàng", for: .normal)
        addButton.titleLabel?.font = .systemFont(ofSize: 17, weight: .bold)
        addButton.backgroundColor = .systemBlue
        addButton.tintColor = .white
        addButton.layer.cornerRadius = 10
        addButton.addTarget(self, action: #selector(addToCartTapped), for: .touchUpInside)
        addButton.translatesAutoresizingMaskIntoConstraints = false

        [imageView, titleLabel, priceLabel, descriptionLabel, addButton].forEach {
            contentView.addSubview($0)
        }

        NSLayoutConstraint.activate([
            imageView.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 20),
            imageView.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            imageView.heightAnchor.constraint(equalToConstant: 220),
            imageView.widthAnchor.constraint(equalToConstant: 220),

            titleLabel.topAnchor.constraint(equalTo: imageView.bottomAnchor, constant: 16),
            titleLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            titleLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),

            priceLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 8),
            priceLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),

            descriptionLabel.topAnchor.constraint(equalTo: priceLabel.bottomAnchor, constant: 12),
            descriptionLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),
            descriptionLabel.trailingAnchor.constraint(equalTo: titleLabel.trailingAnchor),

            addButton.topAnchor.constraint(equalTo: descriptionLabel.bottomAnchor, constant: 20),
            addButton.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
            addButton.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -40),
            addButton.heightAnchor.constraint(equalToConstant: 50),
            addButton.widthAnchor.constraint(equalToConstant: 220)
        ])
    }

    private func populateData() {
        titleLabel.text = product.title
        priceLabel.text = "💰 \(String(format: "%.2f", product.price)) USD"
        descriptionLabel.text = product.description

        if let url = URL(string: product.image) {
            DispatchQueue.global().async {
                if let data = try? Data(contentsOf: url),
                   let img = UIImage(data: data) {
                    DispatchQueue.main.async {
                        self.imageView.image = img
                    }
                }
            }
        }
    }

    @objc private func addToCartTapped() {
        CartManager.shared.add(product)
        let generator = UINotificationFeedbackGenerator()
        generator.notificationOccurred(.success)
        let alert = UIAlertController(
            title: "🛒 Đã thêm vào giỏ hàng",
            message: "\(product.title)",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

💡 Giải thích:

* Dùng `init(product:)` để nhận sản phẩm từ màn hình trước.
* Hiển thị ảnh, tên, giá, mô tả.
* Khi nhấn nút “Thêm vào giỏ hàng” → gọi `CartManager.shared.add(product)`.

---

## 🧠 3. Tạo `CartManager.swift`

Đây là lớp trung tâm quản lý các sản phẩm trong giỏ hàng.

```swift
import Foundation

final class CartManager {
    static let shared = CartManager()
    private(set) var items: [Product] = []

    private init() {}

    func add(_ product: Product) {
        items.append(product)
    }

    func remove(_ product: Product) {
        items.removeAll { $0.id == product.id }
    }

    func clear() {
        items.removeAll()
    }

    var total: Double {
        items.reduce(0) { $0 + $1.price }
    }
}
```

💡 Ở bài sau, ta sẽ hiển thị `items` này trong màn hình **CartViewController**.

---

## 🧩 4. Truyền dữ liệu từ ProductList → ProductDetail

Trong `ProductListViewController`, khi nhấn vào cell:

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    tableView.deselectRow(at: indexPath, animated: true)
    let product = viewModel.products.value[indexPath.row]
    let detailVC = ProductDetailViewController(product: product)
    navigationController?.pushViewController(detailVC, animated: true)
}
```

✅ Dữ liệu `product` được truyền trực tiếp qua hàm khởi tạo `ProductDetailViewController`.

---

## 🧪 5. Chạy thử 🚀

1️⃣ Mở app → Đăng nhập thành công
2️⃣ Chọn một sản phẩm → Mở màn hình chi tiết
3️⃣ Ảnh, tên, giá, mô tả hiển thị đầy đủ
4️⃣ Bấm **“Thêm vào giỏ hàng”** → hiện thông báo “Đã thêm”
5️⃣ Mở console → in ra danh sách giỏ hàng:

```swift
print(CartManager.shared.items)
```

🎉 App của em giờ đã có **luồng mua hàng thực tế: Xem chi tiết → Thêm vào giỏ hàng**!

---

## 🏠 **Bài tập về nhà (Tuần 4, Ngày 3)** 🎒

| Mức độ        | Bài tập                                             | Gợi ý                          |
| ------------- | --------------------------------------------------- | ------------------------------ |
| 🟢 Cơ bản     | Thêm label “Số lượng trong giỏ” vào ProductDetail   | Giả lập đếm số lượng           |
| 🟡 Trung bình | Không cho thêm trùng sản phẩm (nếu đã có trong giỏ) | Check `items.contains(where:)` |
| 🔵 Nâng cao   | Lưu giỏ hàng bằng CoreData                          |                                |
| 🟣 Thử thách  | Tạo nút “Xem giỏ hàng” ở góc phải ProductList       | Push sang CartViewController   |

---

## 📚 Tổng kết buổi học

| Kiến thức                              | Vai trò                              |
| -------------------------------------- | ------------------------------------ |
| **Truyền dữ liệu giữa ViewController** | Giao tiếp logic giữa các màn hình    |
| **Chi tiết sản phẩm (Detail Screen)**  | Giao diện mô tả nội dung đầy đủ      |
| **CartManager (Singleton)**            | Quản lý trạng thái giỏ hàng toàn app |
| **User Feedback (Alert + Haptic)**     | Tăng trải nghiệm người dùng          |
| **MVVM + Navigation Flow**             | App có luồng logic hoàn chỉnh        |

---

🎓 **Ngày 4 (buổi tới – Tuần 4):**

> *Thầy sẽ dạy “Màn hình Giỏ hàng (CartViewController)” – hiển thị danh sách sản phẩm trong giỏ, cho phép xóa từng sản phẩm, và tính tổng tiền.*

---

👉 Em có muốn thầy **tiếp tục luôn Tuần 4 – Ngày 4: CartViewController (Giỏ hàng)** không?
Thầy sẽ hướng dẫn cực chậm, từng bước dựng UI bảng, hiển thị sản phẩm trong giỏ, tính tổng, và thêm nút “Thanh toán giả lập” cho app hoàn chỉnh.
