Ok, qua **Bài 9 – Closure Pattern trong UITableViewCell** 👨‍🏫
Đây là một trong những bài **QUAN TRỌNG NHẤT** trong toàn bộ lộ trình UIKit + MVVM, vì:

* 🟦 90% ứng dụng iOS đều có list/table
* 🟦 Cell thường có button / image / action riêng
* 🟦 ViewController cần biết khi người dùng bấm vào cell → xử lý logic
* 🟦 Delegate rườm rà → **Closure Pattern** là xu hướng hiện đại

Thầy sẽ dạy **rất chậm – dễ hiểu – có comment – code chạy được ngay.**

---

# 🎯 **MỤC TIÊU BÀI 9**

Sau bài này, em sẽ hiểu:

✔ Cách tạo callback trong cell
✔ Cách truyền event từ cell → VC
✔ Khi nào dùng weak self
✔ Vì sao closure trong cell là thứ bắt buộc phải biết
✔ Vì sao MVVM rất cần kỹ thuật này
✔ Code full chạy được (copy/paste OK)

---

# 🧠 **1. Tại sao phải dùng Closure trong UITableViewCell?**

Giả sử em có cell như sau:

```
+-----------------------------------------+
|  product image   Product Name      [♥] |
+-----------------------------------------+
```

Khi user bấm ♥ (favorite), cell cần báo cho VC:

* Đánh dấu sản phẩm yêu thích
* Cập nhật UI
* Lưu vào database

**Cell KHÔNG được biết logic xử lý**, vì đó là trách nhiệm của:

* ViewController
* hoặc ViewModel (MVVM)

→ Cell chỉ là UI.

### Các cách truyền event từ cell → VC:

| Cách                | Ưu                          | Nhược                           |
| ------------------- | --------------------------- | ------------------------------- |
| Delegate            | mạnh, linh hoạt             | quá dài dòng                    |
| Notification        | broadcast, không rõ ràng    | dễ rối, khó debug               |
| Target-Action       | OK cho button đơn           | không truyền data tốt           |
| **Closure Pattern** | **ngắn, rõ ràng, hiện đại** | dễ sai weak self nếu không biết |

👉 Vì vậy, **Closure Pattern** là cách tốt nhất cho UIKit modern.

---

# 🧱 **2. Tạo CustomCell có closure**

Tạo file **ProductCell.swift**

Copy toàn bộ:

```swift
import UIKit

/// Đây là cell hiển thị sản phẩm.
/// Nó có 1 ảnh, 1 label tên sản phẩm, và 1 nút "❤️ Favorite".
/// Khi nút được bấm → gọi closure onFavoriteTapped()
class ProductCell: UITableViewCell {

    // MARK: - Callback Closure

    /// Closure gọi ra ViewController khi user bấm nút "Favorite"
    ///
    /// Tại sao dùng closure?
    /// - Cell không biết phải xử lý gì
    /// - VC hoặc ViewModel quyết định hành vi
    /// - Giảm phụ thuộc, tăng tái sử dụng
    var onFavoriteTapped: (() -> Void)?

    // MARK: - UI Components

    private let productImageView: UIImageView = {
        let iv = UIImageView()
        iv.image = UIImage(systemName: "photo")
        iv.tintColor = .systemBlue
        iv.contentMode = .scaleAspectFit
        iv.clipsToBounds = true
        return iv
    }()

    private let nameLabel: UILabel = {
        let label = UILabel()
        label.font = .systemFont(ofSize: 16, weight: .medium)
        label.numberOfLines = 2
        return label
    }()

    private let favoriteButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("❤️", for: .normal)
        button.titleLabel?.font = .systemFont(ofSize: 24)
        return button
    }()

    // MARK: - Init

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)

        contentView.addSubview(productImageView)
        contentView.addSubview(nameLabel)
        contentView.addSubview(favoriteButton)

        setupConstraints()

        // Gán action cho nút
        favoriteButton.addTarget(self, action: #selector(handleFavorite), for: .touchUpInside)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - AutoLayout

    private func setupConstraints() {
        [productImageView, nameLabel, favoriteButton].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
        }

        NSLayoutConstraint.activate([
            // 1. Ảnh sản phẩm
            productImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 12),
            productImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            productImageView.widthAnchor.constraint(equalToConstant: 60),
            productImageView.heightAnchor.constraint(equalToConstant: 60),

            // 2. Label sản phẩm
            nameLabel.leadingAnchor.constraint(equalTo: productImageView.trailingAnchor, constant: 12),
            nameLabel.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),

            // 3. Nút ❤️
            favoriteButton.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -12),
            favoriteButton.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }

    // MARK: - Public Method

    func configure(name: String) {
        nameLabel.text = name
    }

    // MARK: - Action

    /// Khi user bấm "❤️"
    @objc private func handleFavorite() {
        print("Button ❤️ được bấm trong Cell")

        // GỌI CLOSURE
        onFavoriteTapped?()
    }
}
```

---

# 🧱 **3. Sử dụng custom cell + closure trong ViewController**

Thay toàn bộ code **ViewController.swift** bằng đoạn sau:

```swift
import UIKit

class ViewController: UIViewController {

    private let products = [
        "iPhone 16 Pro",
        "MacBook M4",
        "AirPods Pro 3",
        "Apple Watch S10",
        "iPad Pro OLED"
    ]

    private let tableView = UITableView()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        title = "Bài 9 - Closure trong Cell"

        view.addSubview(tableView)

        setupTable()
        setupConstraints()
    }

    private func setupTable() {
        tableView.translatesAutoresizingMaskIntoConstraints = false
        tableView.rowHeight = 80
        tableView.register(ProductCell.self, forCellReuseIdentifier: "ProductCell")

        tableView.dataSource = self
        tableView.delegate = self
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

        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: "ProductCell",
            for: indexPath
        ) as? ProductCell else { return UITableViewCell() }

        let name = products[indexPath.row]
        cell.configure(name: name)

        // GÁN CLOSURE HANDLER
        cell.onFavoriteTapped = { [weak self] in
            guard let self = self else { return }
            print("VC nhận sự kiện ❤️ của: \(name)")

            // Ví dụ: hiện alert
            let alert = UIAlertController(
                title: "Yêu thích",
                message: "\(name) đã được yêu thích!",
                preferredStyle: .alert
            )
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            self.present(alert, animated: true)
        }

        return cell
    }
}

// MARK: - UITableViewDelegate

extension ViewController: UITableViewDelegate {}
```

---

# 🧪 **KẾT QUẢ CHẠY**

Khi chạy app:

✔ Mỗi cell có hình + tên + biểu tượng ❤️
✔ Khi bấm ❤️ trong cell → VC nhận callback
✔ VC hiện alert “đã yêu thích”
✔ Không cần protocol
✔ Không cần delegate
✔ Code sạch – dễ hiểu – đúng chuẩn UIKit modern

---

# 🎯 **4. Vì sao Closure Pattern trong Cell QUAN TRỌNG CHO MVVM?**

Trong MVVM:

* Cell → thông báo event → ViewController
* ViewController → forward event → ViewModel

Nhờ closure pattern:

### Cell không biết:

* ViewModel là gì
* ViewController là gì
* Business logic là gì

→ Tính **Independent**, **Reusable**, **Clean Architecture**.

Closure chính là “đường ống” truyền sự kiện từ UI → logic.

---

# 🎁 Bài tập cuối bài (rất quan trọng)

### **Bài 1:**

Thêm 1 button "🛒 Add to cart" → tạo closure `onAddTapped`.

### **Bài 2:**

Truyền indexPath thay vì name, để VC biết bấm vào cell nào.

Ví dụ:

```swift
var onFavoriteTapped: ((IndexPath) -> Void)?
```

### **Bài 3:**

Thay closure bằng struct Event:

```swift
struct Event {
    var onFavorite: (() -> Void)?
    var onAddToCart: (() -> Void)?
}
```

---

# 👉 Sẵn sàng qua **Bài 10 – Capture List, Weak Self & tránh memory leak** chưa?

Chỉ cần nói:
**“Qua bài 10 đi thầy”**
