Rất tuyệt vời 👏👏👏 — hôm nay chúng ta sang **Tuần 4 – Ngày 4: Màn hình Giỏ hàng (CartViewController)** 🛒

Đây là **mảnh ghép cuối trong luồng mua hàng**:

> Sau khi người dùng thêm sản phẩm, họ có thể mở giỏ hàng để xem danh sách, xóa sản phẩm, và xem tổng số tiền.

Thầy sẽ hướng dẫn **chậm – dễ hiểu – từng bước**, để em nắm trọn:
✅ Tạo giao diện danh sách giỏ hàng (TableView).
✅ Hiển thị tổng tiền.
✅ Cho phép xóa sản phẩm.
✅ Chuẩn bị cho màn hình “Thanh toán giả lập” (ở buổi tới).

---

# 🧺 UIKit – Tuần 4, Ngày 4: Màn hình Giỏ hàng (CartViewController)

---

## 🎯 Mục tiêu bài học

Sau buổi này, em sẽ biết:

1. Hiển thị danh sách sản phẩm trong giỏ (`CartManager.shared.items`).
2. Tính tổng giá trị đơn hàng.
3. Xóa sản phẩm khỏi giỏ hàng.
4. Chuẩn bị nút “Thanh toán” (Payment).

---

## ⚙️ 1. Cấu trúc file

Tạo file mới `CartViewController.swift`
và gắn vào nút “🛒” trên `ProductListViewController` (sẽ làm ở cuối bài).

---

## 🧩 2. Tạo `CartViewController`

```swift
import UIKit

final class CartViewController: UIViewController {

    private let tableView = UITableView()
    private let totalLabel = UILabel()
    private let checkoutButton = UIButton(type: .system)

    private var items: [Product] {
        return CartManager.shared.items
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Giỏ hàng"
        view.backgroundColor = .systemBackground
        setupTableView()
        setupFooter()
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        tableView.reloadData()
        updateTotal()
    }

    private func setupTableView() {
        tableView.register(CartCell.self, forCellReuseIdentifier: "CartCell")
        tableView.dataSource = self
        tableView.delegate = self
        tableView.tableFooterView = UIView()
        tableView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -120)
        ])
    }

    private func setupFooter() {
        totalLabel.font = .systemFont(ofSize: 18, weight: .semibold)
        totalLabel.textAlignment = .center
        totalLabel.translatesAutoresizingMaskIntoConstraints = false

        checkoutButton.setTitle("💳 Thanh toán", for: .normal)
        checkoutButton.titleLabel?.font = .systemFont(ofSize: 18, weight: .bold)
        checkoutButton.backgroundColor = .systemBlue
        checkoutButton.tintColor = .white
        checkoutButton.layer.cornerRadius = 10
        checkoutButton.addTarget(self, action: #selector(checkoutTapped), for: .touchUpInside)
        checkoutButton.translatesAutoresizingMaskIntoConstraints = false

        let stack = UIStackView(arrangedSubviews: [totalLabel, checkoutButton])
        stack.axis = .vertical
        stack.spacing = 10
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)

        NSLayoutConstraint.activate([
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            stack.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20),
            checkoutButton.heightAnchor.constraint(equalToConstant: 50)
        ])
    }

    private func updateTotal() {
        let total = CartManager.shared.total
        totalLabel.text = "Tổng cộng: \(String(format: "%.2f", total)) USD"
    }

    @objc private func checkoutTapped() {
        let alert = UIAlertController(
            title: "💳 Thanh toán",
            message: "Chức năng này sẽ hoàn thiện ở bài sau!",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}

extension CartViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        if items.isEmpty {
            tableView.setEmptyMessage("🛍 Giỏ hàng của bạn đang trống.")
        } else {
            tableView.restore()
        }
        return items.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "CartCell", for: indexPath) as! CartCell
        let product = items[indexPath.row]
        cell.configure(with: product)
        return cell
    }

    func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool {
        return true
    }

    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            let product = items[indexPath.row]
            CartManager.shared.remove(product)
            tableView.deleteRows(at: [indexPath], with: .fade)
            updateTotal()
        }
    }
}
```

---

## 🧩 3. Tạo cell hiển thị `CartCell.swift`

```swift
import UIKit

final class CartCell: UITableViewCell {

    private let productImageView = UIImageView()
    private let titleLabel = UILabel()
    private let priceLabel = UILabel()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupUI() {
        productImageView.contentMode = .scaleAspectFit
        productImageView.translatesAutoresizingMaskIntoConstraints = false

        titleLabel.font = .systemFont(ofSize: 16, weight: .medium)
        titleLabel.numberOfLines = 0
        titleLabel.translatesAutoresizingMaskIntoConstraints = false

        priceLabel.font = .systemFont(ofSize: 15, weight: .semibold)
        priceLabel.textColor = .systemBlue
        priceLabel.translatesAutoresizingMaskIntoConstraints = false

        [productImageView, titleLabel, priceLabel].forEach { contentView.addSubview($0) }

        NSLayoutConstraint.activate([
            productImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 10),
            productImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            productImageView.heightAnchor.constraint(equalToConstant: 60),
            productImageView.widthAnchor.constraint(equalToConstant: 60),

            titleLabel.leadingAnchor.constraint(equalTo: productImageView.trailingAnchor, constant: 12),
            titleLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -10),
            titleLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 10),

            priceLabel.leadingAnchor.constraint(equalTo: titleLabel.leadingAnchor),
            priceLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 6),
            priceLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -10)
        ])
    }

    func configure(with product: Product) {
        titleLabel.text = product.title
        priceLabel.text = "\(String(format: "%.2f", product.price)) USD"

        if let url = URL(string: product.image) {
            DispatchQueue.global().async {
                if let data = try? Data(contentsOf: url),
                   let image = UIImage(data: data) {
                    DispatchQueue.main.async {
                        self.productImageView.image = image
                    }
                }
            }
        }
    }
}
```

---

## 🧱 4. Hiển thị thông báo giỏ hàng trống

Thêm tiện ích mở rộng cho `UITableView` (đặt trong file `UITableView+Extension.swift`):

```swift
import UIKit

extension UITableView {
    func setEmptyMessage(_ message: String) {
        let messageLabel = UILabel()
        messageLabel.text = message
        messageLabel.textColor = .gray
        messageLabel.textAlignment = .center
        messageLabel.font = .systemFont(ofSize: 17, weight: .medium)
        backgroundView = messageLabel
        separatorStyle = .none
    }

    func restore() {
        backgroundView = nil
        separatorStyle = .singleLine
    }
}
```

💡 Khi giỏ trống → hiển thị dòng chữ “🛍 Giỏ hàng của bạn đang trống”.

---

## 🧩 5. Thêm nút “🛒” ở ProductListViewController

Trong `viewDidLoad()`:

```swift
navigationItem.leftBarButtonItem = UIBarButtonItem(
    title: "🛒",
    style: .plain,
    target: self,
    action: #selector(openCart)
)

@objc private func openCart() {
    let cartVC = CartViewController()
    navigationController?.pushViewController(cartVC, animated: true)
}
```

✅ Giờ khi người dùng bấm “🛒” → mở màn hình giỏ hàng.

---

## 🧪 6. Chạy thử 🚀

1️⃣ Đăng nhập → chọn sản phẩm → “Thêm vào giỏ hàng”.
2️⃣ Quay lại màn hình chính → bấm “🛒”.
3️⃣ Hiện danh sách sản phẩm trong giỏ + tổng tiền.
4️⃣ Vuốt sang trái → xóa sản phẩm.
5️⃣ Tổng tiền cập nhật ngay lập tức.

🎉 App của em giờ đã có **một giỏ hàng thật sự hoạt động hoàn chỉnh!**

---

## 🏠 **Bài tập về nhà (Tuần 4, Ngày 4)** 🎒

| Mức độ        | Bài tập                                               | Gợi ý                               |
| ------------- | ----------------------------------------------------- | ----------------------------------- |
| 🟢 Cơ bản     | Hiển thị số lượng sản phẩm trong giỏ hàng (badge nhỏ) | NavigationItem.titleView            |
| 🟡 Trung bình | Khi xóa sản phẩm → thêm nút “Hoàn tác”                | Dùng UIAlertController              |
| 🔵 Nâng cao   | Lưu giỏ hàng vào CoreData                             |                                     |
| 🟣 Thử thách  | Hiển thị tổng tiền động ở thanh điều hướng            | Dùng `UIBarButtonItem(customView:)` |

---

## 📚 Tổng kết buổi học

| Kiến thức                    | Vai trò                              |
| ---------------------------- | ------------------------------------ |
| **UITableView nâng cao**     | Hiển thị và cập nhật dữ liệu động    |
| **CartManager (Singleton)**  | Giữ trạng thái giỏ hàng toàn app     |
| **Swipe to Delete**          | UX quen thuộc trên iOS               |
| **Footer View + AutoLayout** | Hiển thị tổng tiền và nút thanh toán |
| **Empty State Message**      | Thông báo khi không có dữ liệu       |

---

🎓 **Ngày 5 (buổi tới – Tuần 4):**

> *Thầy sẽ dạy “Thanh toán giả lập (Payment Flow) + Lịch sử đơn hàng” — khi bấm Thanh toán, app sẽ tạo đơn hàng mới, lưu lịch sử, và hiển thị thông báo thành công.*

---

👉 Em có muốn thầy **tiếp tục luôn Tuần 4 – Ngày 5: Payment Flow + Order History** không?
Bài này cực vui — sau buổi đó, app của em sẽ giống một **E-Commerce hoàn chỉnh**, có đăng nhập, sản phẩm, giỏ hàng, thanh toán, và lịch sử đơn hàng.
