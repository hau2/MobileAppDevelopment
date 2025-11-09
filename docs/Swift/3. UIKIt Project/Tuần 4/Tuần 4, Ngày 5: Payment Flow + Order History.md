Rất tốt 👏👏👏 — hôm nay ta sang **Tuần 4 – Ngày 5: Thanh toán giả lập (Payment Flow) + Lịch sử đơn hàng (Order History)** 💳📦

Đây là **bài hoàn thiện giai đoạn 1 của dự án Mini E-Commerce App**, giúp app của em trở thành **một ứng dụng hoàn chỉnh có thể demo thật cho người khác xem**:

> Sau khi người dùng thêm sản phẩm vào giỏ → họ bấm **Thanh toán**, app tạo một “đơn hàng” giả lập và hiển thị lịch sử mua hàng.

Thầy sẽ hướng dẫn **từ tốn, dễ hiểu, từng bước**, bao gồm cả mô hình dữ liệu, logic lưu trữ, giao diện, và thao tác thanh toán.

---

# 💳 UIKit – Tuần 4, Ngày 5: Payment Flow + Order History

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Cách **xử lý thanh toán giả lập (mock payment)** trong app.
2. Cách **tạo và lưu lịch sử đơn hàng (Order)**.
3. Cách **hiển thị danh sách đơn hàng đã mua** bằng TableView.
4. Cách **xoá giỏ hàng sau khi thanh toán**.

---

## 🧱 1. Mô hình dữ liệu đơn hàng

Tạo file `Order.swift`:

```swift
import Foundation

struct Order {
    let id: UUID
    let date: Date
    let items: [Product]
    let total: Double
}
```

💡 Đơn giản nhưng đủ dùng cho app demo:

* `id` để phân biệt các đơn hàng.
* `date` ghi thời gian thanh toán.
* `items` là danh sách sản phẩm đã mua.
* `total` là tổng tiền đơn hàng.

---

## 🧩 2. Tạo `OrderManager.swift`

Đây là singleton quản lý danh sách đơn hàng đã mua.

```swift
import Foundation

final class OrderManager {
    static let shared = OrderManager()
    private(set) var orders: [Order] = []
    private init() {}

    func addOrder(items: [Product]) {
        let order = Order(
            id: UUID(),
            date: Date(),
            items: items,
            total: items.reduce(0) { $0 + $1.price }
        )
        orders.insert(order, at: 0) // đơn mới ở đầu danh sách
    }

    func clear() {
        orders.removeAll()
    }
}
```

💡 Sau này ta có thể lưu danh sách `orders` này vào file hoặc CoreData.

---

## ⚙️ 3. Cập nhật `CartViewController` → xử lý Thanh toán

Trong phần `checkoutTapped()` ta sẽ thay alert tạm bằng logic thật:

```swift
@objc private func checkoutTapped() {
    guard !CartManager.shared.items.isEmpty else {
        let alert = UIAlertController(title: "⚠️ Giỏ hàng trống", message: "Hãy thêm sản phẩm trước khi thanh toán!", preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
        return
    }

    let total = CartManager.shared.total
    let alert = UIAlertController(
        title: "Xác nhận thanh toán",
        message: "Bạn có muốn thanh toán \(String(format: "%.2f", total)) USD không?",
        preferredStyle: .alert
    )

    alert.addAction(UIAlertAction(title: "Hủy", style: .cancel))
    alert.addAction(UIAlertAction(title: "Thanh toán", style: .default) { _ in
        self.performPayment()
    })
    present(alert, animated: true)
}

private func performPayment() {
    // 1️⃣ Tạo đơn hàng
    OrderManager.shared.addOrder(items: CartManager.shared.items)

    // 2️⃣ Xoá giỏ hàng
    CartManager.shared.clear()

    // 3️⃣ Gọi phản hồi người dùng
    let generator = UINotificationFeedbackGenerator()
    generator.notificationOccurred(.success)

    // 4️⃣ Hiển thị thông báo thành công
    let alert = UIAlertController(
        title: "✅ Thanh toán thành công!",
        message: "Đơn hàng của bạn đã được tạo.",
        preferredStyle: .alert
    )
    alert.addAction(UIAlertAction(title: "Xem lịch sử", style: .default) { _ in
        self.showOrderHistory()
    })
    alert.addAction(UIAlertAction(title: "OK", style: .cancel) { _ in
        self.navigationController?.popViewController(animated: true)
    })
    present(alert, animated: true)
}

private func showOrderHistory() {
    let vc = OrderHistoryViewController()
    navigationController?.pushViewController(vc, animated: true)
}
```

💡 Giải thích:

* Khi nhấn “Thanh toán” → app sẽ tạo `Order` mới.
* Giỏ hàng được xóa.
* Hiển thị popup xác nhận và cho người dùng chọn “Xem lịch sử”.

---

## 🧾 4. Tạo `OrderHistoryViewController.swift`

```swift
import UIKit

final class OrderHistoryViewController: UIViewController {

    private let tableView = UITableView()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Lịch sử đơn hàng"
        view.backgroundColor = .systemBackground
        setupTableView()
    }

    private func setupTableView() {
        tableView.register(OrderCell.self, forCellReuseIdentifier: "OrderCell")
        tableView.dataSource = self
        tableView.delegate = self
        tableView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
}

extension OrderHistoryViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        let count = OrderManager.shared.orders.count
        if count == 0 {
            tableView.setEmptyMessage("🧾 Chưa có đơn hàng nào.")
        } else {
            tableView.restore()
        }
        return count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "OrderCell", for: indexPath) as! OrderCell
        let order = OrderManager.shared.orders[indexPath.row]
        cell.configure(with: order)
        return cell
    }

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        let order = OrderManager.shared.orders[indexPath.row]
        let detailVC = OrderDetailViewController(order: order)
        navigationController?.pushViewController(detailVC, animated: true)
    }
}
```

---

## 🧩 5. Tạo cell hiển thị đơn hàng `OrderCell.swift`

```swift
import UIKit

final class OrderCell: UITableViewCell {

    private let idLabel = UILabel()
    private let dateLabel = UILabel()
    private let totalLabel = UILabel()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupUI() {
        [idLabel, dateLabel, totalLabel].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            contentView.addSubview($0)
        }

        idLabel.font = .systemFont(ofSize: 14)
        idLabel.textColor = .secondaryLabel

        dateLabel.font = .systemFont(ofSize: 15)
        totalLabel.font = .systemFont(ofSize: 16, weight: .semibold)
        totalLabel.textColor = .systemBlue

        NSLayoutConstraint.activate([
            idLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            idLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 8),

            dateLabel.leadingAnchor.constraint(equalTo: idLabel.leadingAnchor),
            dateLabel.topAnchor.constraint(equalTo: idLabel.bottomAnchor, constant: 4),

            totalLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            totalLabel.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
        ])
    }

    func configure(with order: Order) {
        let formatter = DateFormatter()
        formatter.dateFormat = "dd/MM/yyyy HH:mm"
        idLabel.text = "Mã đơn: \(order.id.uuidString.prefix(6))"
        dateLabel.text = formatter.string(from: order.date)
        totalLabel.text = "\(String(format: "%.2f", order.total)) USD"
    }
}
```

---

## 🧾 6. (Tuỳ chọn) Xem chi tiết đơn hàng `OrderDetailViewController.swift`

```swift
import UIKit

final class OrderDetailViewController: UIViewController {
    private let order: Order
    private let textView = UITextView()

    init(order: Order) {
        self.order = order
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Chi tiết đơn hàng"
        view.backgroundColor = .systemBackground
        setupTextView()
    }

    private func setupTextView() {
        textView.isEditable = false
        textView.font = .systemFont(ofSize: 16)
        textView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(textView)

        NSLayoutConstraint.activate([
            textView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            textView.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            textView.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            textView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])

        var content = "🧾 Đơn hàng \(order.id.uuidString.prefix(6))\n\n"
        for (index, product) in order.items.enumerated() {
            content += "\(index + 1). \(product.title) — \(String(format: "%.2f", product.price)) USD\n"
        }
        content += "\nTổng tiền: \(String(format: "%.2f", order.total)) USD"
        textView.text = content
    }
}
```

---

## 🧪 7. Chạy thử 🚀

1️⃣ Đăng nhập → thêm vài sản phẩm vào giỏ.
2️⃣ Mở giỏ hàng → bấm “Thanh toán”.
3️⃣ Popup hiện ra → chọn **Thanh toán** → app hiển thị “✅ Thanh toán thành công!”.
4️⃣ Bấm “Xem lịch sử” → hiện danh sách đơn hàng vừa tạo.
5️⃣ Nhấn vào một đơn → xem chi tiết các sản phẩm trong đơn.

🎉 Giờ app của em đã có:
✅ Đăng nhập thật
✅ Danh sách sản phẩm
✅ Chi tiết + Thêm giỏ hàng
✅ Giỏ hàng có thể xoá
✅ Thanh toán + Lịch sử đơn hàng

---

## 🏠 **Bài tập về nhà (Tuần 4, Ngày 5)** 🎒

| Mức độ        | Bài tập                                                    | Gợi ý                             |
| ------------- | ---------------------------------------------------------- | --------------------------------- |
| 🟢 Cơ bản     | Lưu danh sách `orders` xuống file JSON                     | Dùng `FileManager`                |
| 🟡 Trung bình | Hiển thị tổng số đơn hàng ở navigation                     | Dùng `titleView`                  |
| 🔵 Nâng cao   | Tạo biểu đồ thống kê doanh thu                             | Dùng `Charts` library             |
| 🟣 Thử thách  | Gửi thông báo “Thanh toán thành công” (Local Notification) | Dùng `NotificationManager` bài cũ |

---

## 📚 Tổng kết buổi học

| Kiến thức                    | Vai trò                                 |
| ---------------------------- | --------------------------------------- |
| **Mock Payment Flow**        | Giả lập thanh toán thật                 |
| **OrderManager (Singleton)** | Lưu và truy cập đơn hàng toàn app       |
| **Order History Screen**     | Theo dõi đơn hàng đã mua                |
| **Navigation Hierarchy**     | Điều hướng nâng cao giữa nhiều màn hình |
| **Alert & Feedback**         | Tăng trải nghiệm người dùng             |

---

🎓 **Bước tiếp theo (Tuần 5):**

> *Thầy sẽ hướng dẫn “Tối ưu tổng thể & Chuẩn bị phát hành” — gồm splash screen, icon app, dark mode, test user flow, và export build để chia sẻ qua TestFlight.*

---

👉 Em có muốn thầy **tiếp tục luôn Tuần 5 – App Optimization & TestFlight Prep** không?
Bài đó giúp em biến app này thành **phiên bản demo chuyên nghiệp hoàn chỉnh** như một sản phẩm thật.
