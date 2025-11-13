Ok! Qua **Bài 5 – UITableView (Custom Cell bằng code)** 👨‍🏫
Từ bài 5 trở đi chúng ta sẽ làm việc giống dự án thực tế hơn:

* Tạo file riêng cho Cell
* Custom UI trong cell
* Dùng AutoLayout trong cell
* Code đầy đủ + comment chi tiết
* Copy/paste chạy được ngay

Đây là bước CHUYỂN GIAI ĐOẠN từ “bài học cơ bản” → “UIKit thực chiến”.

---

# 🎯 **Mục tiêu Bài 5**

Sau bài này em sẽ nắm được:

✔ Tạo một custom cell bằng code (không storyboard)
✔ Setup UI trong cell
✔ Thêm ảnh + label trong cell
✔ Layout cell bằng AutoLayout
✔ Register + sử dụng cell trong UITableView
✔ Clean code theo style chuyên nghiệp

---

# 🧱 **1. Tạo file Custom Cell**

Trong Xcode → `File → New → File… → Swift File`
Đặt tên: **ProductCell.swift**

---

# 🧱 **2. CODE CHI TIẾT – ProductCell.swift**

Copy toàn bộ đoạn này:

```swift
import UIKit

/// Custom cell dùng để hiển thị thông tin sản phẩm.
/// Cell này gồm:
/// - Hình sản phẩm (UIImageView)
/// - Tên sản phẩm (UILabel)
/// - Giá sản phẩm (UILabel)
class ProductCell: UITableViewCell {

    // MARK: - UI Components

    /// Ảnh sản phẩm (placeholder đơn giản)
    private let productImageView: UIImageView = {
        let imageView = UIImageView()
        imageView.image = UIImage(systemName: "photo")   // Icon mặc định
        imageView.contentMode = .scaleAspectFit          // Giữ tỷ lệ hình
        imageView.clipsToBounds = true
        imageView.tintColor = .systemBlue
        return imageView
    }()

    /// Tên sản phẩm
    private let nameLabel: UILabel = {
        let label = UILabel()
        label.font = .systemFont(ofSize: 17, weight: .semibold)
        label.textColor = .label
        label.numberOfLines = 2                       // Cho phép xuống dòng
        return label
    }()

    /// Giá sản phẩm
    private let priceLabel: UILabel = {
        let label = UILabel()
        label.font = .systemFont(ofSize: 15)
        label.textColor = .systemGreen
        return label
    }()

    // MARK: - Init

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)

        // Thêm UI vào contentView của cell
        contentView.addSubview(productImageView)
        contentView.addSubview(nameLabel)
        contentView.addSubview(priceLabel)

        // Setup AutoLayout
        setupConstraints()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - AutoLayout

    /// Thiết lập layout cho các thành phần trong cell
    private func setupConstraints() {

        // Tắt autoresizingMask để dùng AutoLayout
        [productImageView, nameLabel, priceLabel].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
        }

        NSLayoutConstraint.activate([

            // ===== 1. Ảnh sản phẩm =====
            productImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            productImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            productImageView.widthAnchor.constraint(equalToConstant: 60),
            productImageView.heightAnchor.constraint(equalToConstant: 60),

            // ===== 2. Tên sản phẩm =====
            nameLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 12),
            nameLabel.leadingAnchor.constraint(equalTo: productImageView.trailingAnchor, constant: 12),
            nameLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),

            // ===== 3. Giá sản phẩm =====
            priceLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 6),
            priceLabel.leadingAnchor.constraint(equalTo: nameLabel.leadingAnchor),
            priceLabel.trailingAnchor.constraint(equalTo: nameLabel.trailingAnchor),
            priceLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -12)
        ])
    }

    // MARK: - Public Method

    /// Hàm này để gán dữ liệu từ ViewController vào cell
    func configure(name: String, price: String) {
        nameLabel.text = name
        priceLabel.text = price
    }
}
```

---

# 🧱 **3. Cập nhật ViewController.swift để dùng Custom Cell**

Thay toàn bộ file **ViewController.swift** bằng code dưới (đã tích hợp luôn ProductCell):

```swift
import UIKit

class ViewController: UIViewController {

    // MARK: - Dummy Data
    /// Dữ liệu sản phẩm mẫu (tạm thời chưa dùng API)
    private let products: [(name: String, price: Double)] = [
        ("iPhone 16 Pro Max", 2999),
        ("MacBook Pro 16\" M4", 4899),
        ("iPad Pro OLED", 1499),
        ("AirPods Pro 3", 399),
        ("Apple Watch Series 10", 499),
        ("Apple Vision Pro", 3499),
        ("HomePod mini", 99),
        ("Magic Keyboard", 199)
    ]

    // MARK: - TableView
    private let tableView: UITableView = {
        let table = UITableView()
        table.tableFooterView = UIView()
        table.rowHeight = 90   // Fixed height cell
        return table
    }()

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .systemBackground
        title = "Bài 5 - Custom Cell"

        view.addSubview(tableView)

        setupTable()
        setupConstraints()
    }

    private func setupTable() {
        tableView.translatesAutoresizingMaskIntoConstraints = false

        tableView.dataSource = self
        tableView.delegate = self

        // Đăng ký custom cell
        tableView.register(ProductCell.self, forCellReuseIdentifier: "ProductCell")
    }

    private func setupConstraints() {
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
}

// MARK: - UITableViewDataSource
extension ViewController: UITableViewDataSource {

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return products.count
    }

    func tableView(
        _ tableView: UITableView,
        cellForRowAt indexPath: IndexPath
    ) -> UITableViewCell {

        // Lấy ra custom cell
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "ProductCell", for: indexPath) as? ProductCell else {
            return UITableViewCell()
        }

        let item = products[indexPath.row]

        // Gọi hàm configure để gán dữ liệu vào cell
        cell.configure(name: item.name, price: "$\(item.price)")

        return cell
    }
}

// MARK: - UITableViewDelegate
extension ViewController: UITableViewDelegate {

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)

        let item = products[indexPath.row]
        print("Tapped: \(item.name)")
    }
}
```

---

# 🧪 **Kết quả khi chạy**

✔ Mỗi cell có hình (icon “photo”), tên sản phẩm, giá
✔ UI đẹp – gọn – spacing hợp lý
✔ Cell tương tác được
✔ Không crash
✔ Code sạch, comment rõ

---

# 🎁 Bài tập mở rộng (rất quan trọng)

Làm 1 trong 3 bài sau:

### **Bài 1 – Thay icon `photo` bằng 1 màu nền vuông**

Gợi ý:

```swift
imageView.backgroundColor = .systemGray4
```

### **Bài 2 – Thêm 1 nút “Add to cart” trong mỗi cell**

### **Bài 3 – Làm cell “chạm highlight”**

Gợi ý override:

```swift
override func setHighlighted(_ highlighted: Bool, animated: Bool)
```

---

# 🎓 Bài tiếp theo?

Nếu em muốn qua tiếp:

👉 **Bài 6 – UICollectionView cơ bản**
hoặc
👉 Bắt đầu phần UIScrollView, hoặc phần Combine

Hãy nói: **“Qua bài 6 đi thầy”** để tiếp tục.
