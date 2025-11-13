Ok, qua **Bài 4 – UITableView (phần 1)** nha 🎓
Từ giờ thầy sẽ **comment chi tiết trong code** để em đọc như giáo trình luôn.

Bài này:

* Tạo **UITableView bằng code**
* Hiển thị danh sách dữ liệu đơn giản (fake data)
* Dùng `dataSource` + `delegate` cơ bản
* Code **copy/paste là chạy được**

> AppDelegate + SceneDelegate giữ y nguyên như Bài 1–3.
> Thầy chỉ thay file `ViewController.swift`.

---

## 🎯 Mục tiêu Bài 4

Sau bài này, em sẽ:

* Biết tạo `UITableView` bằng code
* Biết implement `UITableViewDataSource`
* Biết `numberOfRowsInSection` và `cellForRowAt`
* Biết cách **đăng ký cell** (`register`) và **tái sử dụng cell** (`dequeueReusableCell`)

---

## 📌 `ViewController.swift` – Bài 4 (FULL CODE + COMMENT)

> Em hãy **thay toàn bộ file `ViewController.swift`** bằng đoạn dưới, rồi chạy app.

```swift
import UIKit

/// Đây là màn hình chính hiển thị danh sách sản phẩm giả bằng UITableView
class ViewController: UIViewController {

    // MARK: - Dummy Data (Dữ liệu mẫu để đổ vào table)
    /// Mảng dữ liệu đơn giản để hiển thị.
    /// Sau này chúng ta sẽ thay bằng data thật từ API/Repository.
    private let products: [String] = [
        "iPhone 16 Pro Max",
        "MacBook Pro 16\" M4",
        "iPad Pro OLED",
        "AirPods Pro 3",
        "Apple Watch Series 10",
        "Apple Vision Pro",
        "HomePod mini",
        "Apple TV 4K",
        "Magic Keyboard",
        "Magic Mouse"
    ]

    // MARK: - UI Components

    /// UITableView dùng để hiển thị danh sách
    private let tableView: UITableView = {
        // Khởi tạo table view với kiểu plain mặc định
        let table = UITableView(frame: .zero, style: .plain)
        // Ẩn dòng kẻ thừa dưới cùng khi không đủ cell
        table.tableFooterView = UIView()
        // Đặt estimated row height (tạm, sau này có thể dùng auto layout)
        table.estimatedRowHeight = 60
        // Đặt rowHeight = automaticDimension để cell tự co giãn theo nội dung nếu cần
        table.rowHeight = UITableView.automaticDimension
        return table
    }()

    // MARK: - View Lifecycle

    override func viewDidLoad() {
        super.viewDidLoad()

        // Màu nền hệ thống (tự động dark/light mode)
        view.backgroundColor = .systemBackground

        // Đặt title cho navigation bar
        title = "Bài 4 - UITableView"

        // Thêm tableView vào view chính
        view.addSubview(tableView)

        // Gọi hàm cấu hình table view (delegate, dataSource, register cell)
        setupTableView()

        // Gọi hàm đặt AutoLayout cho tableView
        setupConstraints()
    }

    // MARK: - Setup TableView

    /// Hàm cấu hình tableView: set delegate, dataSource và đăng ký cell
    private func setupTableView() {
        // Cho phép dùng AutoLayout (bắt buộc phải false)
        tableView.translatesAutoresizingMaskIntoConstraints = false

        // Gán dataSource và delegate cho ViewController
        // ViewController sẽ chịu trách nhiệm cung cấp dữ liệu và xử lý sự kiện của tableView
        tableView.dataSource = self
        tableView.delegate = self

        // Đăng ký loại cell cơ bản của UITableView.
        // "cell" là reuseIdentifier dùng để dequeue ở cellForRowAt.
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
    }

    // MARK: - Layout

    /// Đặt constraint cho tableView để nó chiếm toàn bộ màn hình
    private func setupConstraints() {
        NSLayoutConstraint.activate([
            // Kéo 4 cạnh của tableView dính với 4 cạnh safeArea của view
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
}

// MARK: - UITableViewDataSource

/// Extension này giúp tách phần implement DataSource ra cho gọn code.
/// UITableViewDataSource = nơi cung cấp dữ liệu (số row, nội dung cell)
extension ViewController: UITableViewDataSource {

    /// Hỏi: Trong section này có bao nhiêu dòng?
    /// Ở ví dụ đơn giản này, ta chỉ có 1 section nên trả về số phần tử trong mảng products.
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return products.count
    }

    /// Hỏi: Tại indexPath (hàng thứ row) thì cell hiển thị như thế nào?
    /// Đây là nơi ta dequeue (lấy ra) một cell và gán dữ liệu cho nó.
    func tableView(
        _ tableView: UITableView,
        cellForRowAt indexPath: IndexPath
    ) -> UITableViewCell {

        // Lấy ra một cell đã đăng ký với identifier "cell".
        // dequeueReusableCell giúp tái sử dụng cell cũ, tiết kiệm bộ nhớ.
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)

        // Lấy tên sản phẩm tương ứng với hàng hiện tại
        let productName = products[indexPath.row]

        // Gán text cho cell
        cell.textLabel?.text = productName

        // Đặt font, style nếu muốn
        cell.textLabel?.font = .systemFont(ofSize: 16, weight: .medium)

        // Thêm accessory loại ">" ở bên phải, nhìn giống như có thể bấm vào để xem chi tiết
        cell.accessoryType = .disclosureIndicator

        return cell
    }
}

// MARK: - UITableViewDelegate

/// UITableViewDelegate = nơi xử lý sự kiện liên quan tới việc tương tác với cell (chạm, chọn, chiều cao, v.v.)
extension ViewController: UITableViewDelegate {

    /// Hàm này được gọi khi người dùng chọn (tap) vào một dòng trong tableView.
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        // Bỏ highlight sau khi chọn để UI không giữ màu xám
        tableView.deselectRow(at: indexPath, animated: true)

        // Lấy tên sản phẩm được chọn
        let selectedProduct = products[indexPath.row]

        // In ra console cho dễ debug
        print("Bạn đã chọn sản phẩm: \(selectedProduct)")

        // Hiện một alert đơn giản
        let alert = UIAlertController(
            title: "Đã chọn",
            message: "Bạn chọn: \(selectedProduct)",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))

        present(alert, animated: true, completion: nil)
    }
}
```

---

## ✅ Khi chạy app, em sẽ thấy:

* Một màn hình có title: **“Bài 4 - UITableView”**
* Một danh sách các sản phẩm Apple giả (10 item)
* Tap vào 1 dòng → hiện alert **“Bạn chọn: …”**

Nếu đúng như vậy → **Bài 4 hoàn thành ✔**

---

## 🧠 Giải thích nhanh dòng quan trọng

* `tableView.dataSource = self`
  → ViewController chịu trách nhiệm cung cấp data.

* `tableView.delegate = self`
  → ViewController xử lý sự kiện, ví dụ khi bấm vào cell.

* `tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")`
  → Đăng ký kiểu cell để hệ thống tạo/tái sử dụng.

* `dequeueReusableCell(withIdentifier: "cell", for: indexPath)`
  → Lấy cell từ pool (nếu có sẵn) hoặc tạo mới (nếu chưa có).

---

## 🎁 Bài tập gợi ý (nếu muốn làm thêm)

1. **Đổi kiểu dữ liệu:**
   Thay `products: [String]` thành mảng `Product`:

   ```swift
   struct Product {
       let name: String
       let price: Double
   }
   ```

2. **Hiển thị giá:**
   Gán `cell.detailTextLabel?.text = "Giá: \(price)"` (cần đổi style cell).

3. **Đổi kiểu cell:**
   Dùng `UITableViewCell(style: .subtitle, reuseIdentifier: "cell")` trong `cellForRowAt` (thầy có thể hướng dẫn ở bài sau).

---

Nếu em muốn:

* Thầy giải thích lại từng hàm `numberOfRowsInSection`, `cellForRowAt`, `didSelectRowAt`
* Hay giúp refactor sang struct `Product` cho giống dự án thật

cứ bảo: **“Giải thích chi tiết DataSource/Delegate đi thầy”**.

Còn nếu em thấy ổn rồi, mình sẽ qua:
👉 **Bài 5 – UITableView (custom cell bằng code + comment chi tiết)**
