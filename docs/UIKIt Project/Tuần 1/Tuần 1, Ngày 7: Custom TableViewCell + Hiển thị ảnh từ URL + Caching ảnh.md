Rất tốt 👏👏👏 — hôm nay thầy và em bước sang **Tuần 1 – Ngày 7: Custom TableViewCell + Hiển thị ảnh từ URL + Caching ảnh** 🎨📱

Đây là bài rất quan trọng để app của em **trông thật sự “đẹp và chuyên nghiệp”** — không chỉ có chữ nữa mà **hiển thị ảnh sản phẩm thật**, giống Shopee, Lazada, hay Tiki.

Thầy sẽ dạy **rất chậm, rõ ràng, từng dòng một**, để em nắm chắc toàn bộ cách làm việc với hình ảnh trong UIKit, bao gồm:

* Tải ảnh qua URL
* Hiển thị ảnh trong custom cell
* Cache ảnh lại để lần sau không tải lại mạng

---

# 🎨 UIKit – Tuần 1, Ngày 7: Custom Cell + Hiển thị ảnh từ URL + Image Caching

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Biết cách **tạo TableViewCell tùy chỉnh (Custom Cell)**.
2. Biết cách **tải ảnh từ URL** bằng `URLSession`.
3. Biết cách **cache ảnh** bằng `NSCache` để tăng tốc.
4. Áp dụng tất cả vào app hiển thị sản phẩm (API thật từ bài trước).

---

## 🧱 1. Cấu trúc project hôm nay

```
ProductApp
├── Models
│   └── Product.swift
├── Services
│   └── NetworkService.swift
│   └── ImageLoader.swift   👈 (mới)
├── ViewModels
│   └── ProductListViewModel.swift
├── Views
│   └── ProductCell.swift   👈 (mới)
├── Screens
│   └── ProductListViewController.swift
└── AppDelegate / SceneDelegate
```

---

## 🧩 2. Tạo lớp tải ảnh `ImageLoader.swift`

```swift
import UIKit

final class ImageLoader {
    static let shared = ImageLoader()

    private let cache = NSCache<NSString, UIImage>()

    private init() {}

    func loadImage(from urlString: String, completion: @escaping (UIImage?) -> Void) {
        // Nếu ảnh đã cache → trả luôn
        if let cachedImage = cache.object(forKey: urlString as NSString) {
            completion(cachedImage)
            return
        }

        // Nếu chưa có → tải từ mạng
        guard let url = URL(string: urlString) else {
            completion(nil)
            return
        }

        URLSession.shared.dataTask(with: url) { data, response, error in
            guard let data = data,
                  let image = UIImage(data: data),
                  error == nil else {
                DispatchQueue.main.async { completion(nil) }
                return
            }

            // Lưu cache
            self.cache.setObject(image, forKey: urlString as NSString)

            DispatchQueue.main.async {
                completion(image)
            }
        }.resume()
    }
}
```

💡 Giải thích:

* `NSCache` giúp lưu ảnh trong RAM, lần sau dùng lại không cần tải.
* `ImageLoader.shared` là **singleton**, dùng chung toàn app.
* Kết quả tải xong luôn gọi `completion` trên **main thread** (để update UI).

---

## 🧩 3. Tạo Custom Cell: `ProductCell.swift`

```swift
import UIKit

final class ProductCell: UITableViewCell {
    static let reuseID = "ProductCell"

    private let productImageView = UIImageView()
    private let titleLabel = UILabel()
    private let priceLabel = UILabel()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }

    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    private func setupUI() {
        // Cấu hình ảnh
        productImageView.contentMode = .scaleAspectFit
        productImageView.translatesAutoresizingMaskIntoConstraints = false
        productImageView.clipsToBounds = true
        productImageView.layer.cornerRadius = 8

        // Cấu hình tiêu đề
        titleLabel.font = .systemFont(ofSize: 16, weight: .medium)
        titleLabel.numberOfLines = 2
        titleLabel.translatesAutoresizingMaskIntoConstraints = false

        // Cấu hình giá
        priceLabel.font = .systemFont(ofSize: 15, weight: .semibold)
        priceLabel.textColor = .systemGreen
        priceLabel.translatesAutoresizingMaskIntoConstraints = false

        contentView.addSubview(productImageView)
        contentView.addSubview(titleLabel)
        contentView.addSubview(priceLabel)

        NSLayoutConstraint.activate([
            productImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 12),
            productImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            productImageView.widthAnchor.constraint(equalToConstant: 80),
            productImageView.heightAnchor.constraint(equalToConstant: 80),

            titleLabel.leadingAnchor.constraint(equalTo: productImageView.trailingAnchor, constant: 12),
            titleLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -12),
            titleLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 16),

            priceLabel.leadingAnchor.constraint(equalTo: productImageView.trailingAnchor, constant: 12),
            priceLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 8),
            priceLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -16)
        ])
    }

    func configure(with product: Product) {
        titleLabel.text = product.title
        priceLabel.text = "$\(String(format: "%.2f", product.price))"

        productImageView.image = UIImage(systemName: "photo") // ảnh tạm

        ImageLoader.shared.loadImage(from: product.image) { [weak self] image in
            self?.productImageView.image = image
        }
    }
}
```

💬 Giải thích:

* `configure(with:)` nhận `Product` và hiển thị ảnh, tên, giá.
* Dùng `ImageLoader` để tải ảnh, có cache để tránh tải lại.
* Dùng ảnh mặc định “photo” khi đang load để tránh ô trống.

---

## 📱 4. Cập nhật `ProductListViewController.swift`

```swift
import UIKit

final class ProductListViewController: UIViewController {
    private let tableView = UITableView()
    private let viewModel: ProductListViewModel

    init(viewModel: ProductListViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Sản phẩm"
        view.backgroundColor = .systemBackground
        setupTableView()
        bindViewModel()
        viewModel.fetchData()
    }

    private func setupTableView() {
        tableView.translatesAutoresizingMaskIntoConstraints = false
        tableView.rowHeight = 110
        tableView.dataSource = self
        tableView.register(ProductCell.self, forCellReuseIdentifier: ProductCell.reuseID)
        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    private func bindViewModel() {
        viewModel.products.bind { [weak self] _ in
            DispatchQueue.main.async {
                self?.tableView.reloadData()
            }
        }
    }
}

extension ProductListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        viewModel.products.value.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: ProductCell.reuseID,
            for: indexPath
        ) as? ProductCell else {
            return UITableViewCell()
        }

        let product = viewModel.products.value[indexPath.row]
        cell.configure(with: product)
        return cell
    }
}
```

💡 Giờ mỗi ô (cell) trong bảng hiển thị:

* Ảnh sản phẩm
* Tên sản phẩm
* Giá sản phẩm

---

## ⚙️ 5. Chạy thử 🎬

✅ Khi app chạy → gọi API lấy danh sách sản phẩm.
✅ Mỗi dòng hiện ảnh + tên + giá.
✅ Lần đầu tải ảnh hơi chậm → lần sau mượt vì **đã cache**.
✅ Cuộn nhanh → ảnh không nhấp nháy (cache hoạt động).

🎉 Em vừa hoàn thiện **app hiển thị danh sách sản phẩm thật + ảnh thật + caching** —
đây chính là **nền tảng cho mọi app thương mại điện tử hoặc tin tức.**

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 7)** 🎒

| Mức độ        | Bài tập                                                        | Gợi ý                                |
| ------------- | -------------------------------------------------------------- | ------------------------------------ |
| 🟢 Cơ bản     | Thêm label hiển thị “category”                                 | Dưới `priceLabel`                    |
| 🟡 Trung bình | Tạo hiệu ứng làm mờ ảnh khi tải xong                           | Dùng `UIView.animate(withDuration:)` |
| 🔵 Nâng cao   | Thêm `UIActivityIndicatorView` trong cell khi đang load ảnh    |                                      |
| 🟣 Thử thách  | Tạo cơ chế cache ảnh xuống disk (dạng file, không chỉ NSCache) | Dùng `FileManager`                   |

---

## 📚 Tổng kết buổi học

| Thành phần                    | Vai trò                            |
| ----------------------------- | ---------------------------------- |
| **Custom Cell (ProductCell)** | Tùy chỉnh UI từng hàng             |
| **UIImageView + URLSession**  | Tải ảnh từ Internet                |
| **NSCache**                   | Lưu ảnh trong RAM để tái sử dụng   |
| **Binding (Observable)**      | Cập nhật UI tự động khi có dữ liệu |
| **Async Image Loading**       | Giúp UI không bị đơ khi tải ảnh    |

---

🎓 **Ngày 8 (buổi tới):**

> *Thầy sẽ dạy “Tạo màn hình chi tiết sản phẩm (Product Detail Screen) + truyền dữ liệu giữa ViewController qua Coordinator” — để app của em trở thành một mini e-commerce app hoàn chỉnh.*

---

👉 Em có muốn thầy dạy luôn **Ngày 8 – Màn chi tiết sản phẩm + truyền dữ liệu qua Coordinator** không?
Thầy sẽ giải thích từng dòng code, có ví dụ thực tế và hình minh họa luồng dữ liệu nhé.
