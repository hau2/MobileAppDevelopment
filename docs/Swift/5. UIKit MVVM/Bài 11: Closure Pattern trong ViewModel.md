Ok, qua **Bài 11 – Closure Pattern trong ViewModel** 👨‍🏫
Đây là **bài CỰC KỲ QUAN TRỌNG**, vì toàn bộ mô hình **MVVM + UIKit + Repository** sau này của em sẽ dựa vào đúng kỹ thuật này.

Thầy sẽ dạy:

* Chậm
* Rõ ràng
* Giải thích “Vì sao phải dùng”
* Code thực hành đầy đủ + comment để copy/paste chạy được

---

# 🎯 **MỤC TIÊU BÀI 11 – Closure Pattern trong ViewModel**

Sau bài này, em sẽ:

✔ Hiểu vì sao ViewModel cần closure
✔ Biết cách tạo ViewModel có callback “output event”
✔ Biết cách truyền event ViewModel → ViewController
✔ Biết cách dùng weak self khi bind
✔ Hiểu 3 loại event thường gặp: loading – success – error
✔ Có ví dụ chạy được ngay

---

# 🧠 **1. TẠI SAO VIEWMODEL PHẢI DÙNG CLOSURE?**

### MVVM có 2 chiều:

```
View (VC) ------------> ViewModel   (Input)
ViewModel ------------> View (VC)   (Output)
```

**VC → VM (input)**: gọi hàm
**VM → VC (output)**: báo data đã load, báo lỗi, báo xong…

---

### 🟥 VẤN ĐỀ:

ViewModel **không được import UIKit**
→ Không thể `reloadData()`
→ Không thể show alert
→ Không được giữ VC

Nhưng ViewModel vẫn cần báo:

* Đã load xong dữ liệu
* Lỗi xảy ra
* Loading start/stop
* Cập nhật danh sách sản phẩm

---

### 🟩 GIẢI PHÁP HIỆN ĐẠI: **Closure Pattern trong ViewModel**

ViewModel khai báo:

```swift
var onLoadingChange: ((Bool) -> Void)?
var onError: ((String) -> Void)?
var onDataLoaded: (([Product]) -> Void)?
```

ViewController bind:

```swift
viewModel.onDataLoaded = { [weak self] products in
    self?.tableView.reloadData()
}
```

ViewModel gọi:

```swift
onDataLoaded?(products)
```

👉 Đây chính là cách data “chảy” từ ViewModel → View.

---

# 🧱 **2. BẮT ĐẦU VI DỤ – ViewModel với Closure**

Tạo file mới: **ProductViewModel.swift**

```swift
import Foundation

/// ViewModel này làm nhiệm vụ xử lý logic của màn hình danh sách sản phẩm.
/// Nó không biết UIKit, không biết ViewController.
/// Nó chỉ cung cấp DATA và EVENT qua closure.
class ProductViewModel {

    // MARK: - Output events (đưa dữ liệu ra ngoài)

    /// Báo trạng thái loading (true = đang load, false = dừng)
    var onLoadingChange: ((Bool) -> Void)?

    /// Báo lỗi nếu có
    var onError: ((String) -> Void)?

    /// Báo danh sách sản phẩm khi load xong
    var onProductsLoaded: (([String]) -> Void)?

    // MARK: - Fake data (chưa API)
    private let fakeProducts = [
        "iPhone 16 Pro",
        "iPad Pro OLED",
        "MacBook Pro M4",
        "AirPods Pro 3",
        "Vision Pro"
    ]

    // MARK: - Public API cho View gọi vào
    func loadProducts() {

        // 1. Bắt đầu loading
        onLoadingChange?(true)

        // 2. Giả lập xử lý bất đồng bộ (giống gọi API)
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) { [weak self] in
            guard let self = self else { return }

            // 3. Dừng loading
            self.onLoadingChange?(false)

            // 4. Gửi data ra View (success)
            self.onProductsLoaded?(self.fakeProducts)

            // Nếu muốn test lỗi:
            // self.onError?("Không thể tải dữ liệu.")
        }
    }
}
```

---

# 🧩 **3. Bind ViewModel vào ViewController (full code + comment)**

Mở file **ViewController.swift** và thay bằng đoạn dưới:

```swift
import UIKit

class ViewController: UIViewController {

    private let viewModel = ProductViewModel()

    private var products: [String] = []

    private let tableView = UITableView()

    private let loadingIndicator: UIActivityIndicatorView = {
        let indicator = UIActivityIndicatorView(style: .large)
        indicator.hidesWhenStopped = true
        return indicator
    }()

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .systemBackground
        title = "Bài 11 - Closure trong ViewModel"

        view.addSubview(tableView)
        view.addSubview(loadingIndicator)

        setupTable()
        setupConstraints()
        bindViewModel()

        // Gọi load data
        viewModel.loadProducts()
    }

    private func setupTable() {
        tableView.translatesAutoresizingMaskIntoConstraints = false
        tableView.dataSource = self
        tableView.rowHeight = 60

        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
    }

    private func setupConstraints() {
        loadingIndicator.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor),

            loadingIndicator.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            loadingIndicator.centerYAnchor.constraint(equalTo: view.centerYAnchor),
        ])
    }

    // MARK: - BIND VIEWMODEL

    private func bindViewModel() {

        // 1️⃣ Loading state
        viewModel.onLoadingChange = { [weak self] isLoading in
            guard let self = self else { return }
            if isLoading {
                self.loadingIndicator.startAnimating()
            } else {
                self.loadingIndicator.stopAnimating()
            }
        }

        // 2️⃣ Data loaded
        viewModel.onProductsLoaded = { [weak self] products in
            guard let self = self else { return }
            self.products = products
            self.tableView.reloadData()
        }

        // 3️⃣ Error
        viewModel.onError = { [weak self] message in
            guard let self = self else { return }

            let alert = UIAlertController(
                title: "Lỗi",
                message: message,
                preferredStyle: .alert
            )
            alert.addAction(UIAlertAction(title: "OK", style: .default))
            self.present(alert, animated: true)
        }
    }
}

// MARK: - UITableViewDataSource

extension ViewController: UITableViewDataSource {

    func tableView(
        _ tableView: UITableView,
        numberOfRowsInSection section: Int
    ) -> Int {
        return products.count
    }

    func tableView(
        _ tableView: UITableView,
        cellForRowAt indexPath: IndexPath
    ) -> UITableViewCell {

        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = products[indexPath.row]
        return cell
    }
}
```

---

# 🧪 **KẾT QUẢ CHẠY**

Khi chạy app:

✔ Loading spinner hiện
✔ 1.5 giây sau → data load thành công
✔ TableView hiển thị danh sách sản phẩm
✔ Không crash
✔ Không leak
✔ MVVM chạy đúng luồng dữ liệu

---

# 🎯 **VÌ SAO CLOSURE TRONG VIEWMODEL THỰC SỰ QUAN TRỌNG?**

### 🟩 1. Tách biệt UI logic và business logic

VC chỉ hiển thị UI → không xử lý dữ liệu
ViewModel chỉ xử lý dữ liệu → không liên quan UI

Closure giúp:

```
VM xử lý → callback → VC nhận → update UI
```

---

### 🟩 2. Giảm phụ thuộc (loose coupling)

* ViewModel không biết ViewController là ai
* ViewController không biết logic bên trong ViewModel

→ Clean Architecture.

---

### 🟩 3. Thay thế cho RxSwift / Combine trong MVVM UIKit

MVVM có 2 cách truyền event:

| Kiểu              | Đặc điểm                         |
| ----------------- | -------------------------------- |
| Closure Pattern   | đơn giản, dễ học, không thư viện |
| Combine / RxSwift | mạnh hơn, reactive               |

Bắt đầu MVVM bằng closure là chuẩn.

---

# 🎁 BÀI TẬP CUỐI BÀI

**Bài 1:**
Thêm một event:

```swift
var onEmptyData: (() -> Void)?
```

Khi mảng rỗng → gọi onEmptyData.

---

**Bài 2:**
Đổi fake data thành struct:

```swift
struct Product {
    let name: String
    let price: Double
}
```

---

**Bài 3 (khó):**
Trong ViewModel:

* thêm trạng thái:

```swift
enum State {
    case idle
    case loading
    case loaded([Product])
    case error(String)
}
```

* Tạo closure:

```swift
var onStateChange: ((State) -> Void)?
```

→ Đây là nền tảng MVVM hướng State rất mạnh.

---

# 👉 Sẵn sàng qua **Bài 12 – Combine căn bản cho MVVM** chưa?

Chỉ cần nói:
**“Qua bài 12 đi thầy”**
