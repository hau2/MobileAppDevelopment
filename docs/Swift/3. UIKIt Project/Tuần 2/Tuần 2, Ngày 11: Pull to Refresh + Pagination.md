Rất tốt 👏👏👏 — hôm nay ta sang **Tuần 2 – Ngày 11: Pull to Refresh & Pagination trong UITableView**

Đây là **kỹ năng cực kỳ thực tế** mà hầu như mọi ứng dụng có danh sách dữ liệu đều phải có: từ **Shopee**, **Facebook**, đến **TikTok**.
Khi dữ liệu nhiều, ta **không thể tải hết một lần**, mà phải:

* Kéo xuống để **refresh** (cập nhật dữ liệu mới nhất)
* Cuộn tới cuối để **tải thêm (pagination)**

---

# 🧩 UIKit – Tuần 2, Ngày 11: Pull to Refresh + Pagination

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ:

1. Biết tạo **UIRefreshControl** để làm “kéo để làm mới”.
2. Biết cách **phát hiện khi cuộn tới cuối bảng** để tải thêm.
3. Hiểu nguyên lý **Pagination (phân trang)**.
4. Áp dụng song song với API thật (`https://fakestoreapi.com/products`).

---

## 🧠 1. Ôn lại cấu trúc `ProductListViewController`

Chúng ta đang có:

* TableView hiển thị danh sách sản phẩm
* ViewModel quản lý dữ liệu
* API gọi qua NetworkService

Giờ ta mở rộng để:
✅ Kéo xuống → làm mới danh sách
✅ Cuộn đến cuối → tự động tải thêm sản phẩm

---

## ⚙️ 2. Tạo `UIRefreshControl` (kéo để làm mới)

Trong `ProductListViewController` thêm:

```swift
final class ProductListViewController: UIViewController {
    private let tableView = UITableView()
    private let refreshControl = UIRefreshControl()
    private let spinner = UIActivityIndicatorView(style: .large)
    private let viewModel: ProductListViewModel
    private var isLoadingMore = false

    // ... init, setup giữ nguyên

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Sản phẩm"
        view.backgroundColor = .systemBackground
        setupTableView()
        setupRefreshControl()
        setupSpinner()
        bindViewModel()
        fetchDataAsync()
    }

    private func setupRefreshControl() {
        refreshControl.addTarget(self, action: #selector(refreshPulled), for: .valueChanged)
        tableView.refreshControl = refreshControl
    }

    @objc private func refreshPulled() {
        Task {
            do {
                let products = try await viewModel.fetchDataAsync(reset: true)
                DispatchQueue.main.async {
                    self.viewModel.products.value = products
                    self.refreshControl.endRefreshing()
                }
            } catch {
                self.refreshControl.endRefreshing()
                print("❌ Lỗi refresh:", error)
            }
        }
    }
}
```

💬 Giải thích:

* `UIRefreshControl` là control tích hợp sẵn để kéo xuống.
* Khi kéo → gọi hàm `refreshPulled()` để tải lại dữ liệu.
* Dùng `endRefreshing()` để tắt vòng xoay sau khi xong.

---

## 🧩 3. Thêm Pagination (cuộn xuống để tải thêm)

Cập nhật `UITableViewDelegate` trong `ProductListViewController`:

```swift
extension ProductListViewController: UITableViewDelegate, UITableViewDataSource {
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

        // Nếu đã đến dòng cuối cùng, tải thêm dữ liệu
        if indexPath.row == viewModel.products.value.count - 1 {
            loadMoreData()
        }

        return cell
    }
}
```

---

## ⚙️ 4. Cập nhật `ProductListViewModel.swift` để hỗ trợ phân trang

```swift
final class ProductListViewModel {
    private let networkService: NetworkServiceProtocol
    private var currentPage = 1
    private let pageSize = 5
    var products: Observable<[Product]> = Observable([])

    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
    }

    func fetchDataAsync(reset: Bool = false) async throws -> [Product] {
        if reset {
            currentPage = 1
        }

        let allProducts = try await networkService.fetchProductsAsync()
        let start = (currentPage - 1) * pageSize
        let end = min(start + pageSize, allProducts.count)
        let pageProducts = Array(allProducts[start..<end])

        if reset {
            products.value = pageProducts
        } else {
            products.value += pageProducts
        }

        currentPage += 1
        return products.value
    }

    func canLoadMore() -> Bool {
        return products.value.count < 20 // vì API này có 20 sản phẩm
    }
}
```

💡 Giải thích:

* `pageSize` = số sản phẩm tải mỗi lần.
* `currentPage` = trang hiện tại.
* Mỗi lần tải thêm, `currentPage += 1`.

---

## 🧩 5. Viết hàm tải thêm trong ViewController

```swift
extension ProductListViewController {
    private func loadMoreData() {
        guard !isLoadingMore, viewModel.canLoadMore() else { return }
        isLoadingMore = true

        Task {
            do {
                let _ = try await viewModel.fetchDataAsync()
                DispatchQueue.main.async {
                    self.isLoadingMore = false
                    self.tableView.reloadData()
                }
            } catch {
                DispatchQueue.main.async {
                    self.isLoadingMore = false
                }
                print("❌ Lỗi load more:", error)
            }
        }
    }
}
```

💬 Giải thích:

* Dùng `isLoadingMore` để tránh gọi chồng nhiều lần khi cuộn nhanh.
* Mỗi lần chạm cuối bảng → tự tải thêm.

---

## 🧪 6. Chạy thử 🚀

✅ Khi mở app → danh sách tải 5 sản phẩm đầu tiên.
✅ Kéo xuống → vòng xoay refresh xuất hiện, tải lại toàn bộ.
✅ Cuộn đến cuối → tự động tải thêm 5 sản phẩm tiếp theo.
✅ UI vẫn mượt, không “lag”.

🎉 Em vừa hoàn thiện **một danh sách có khả năng làm mới và tải vô hạn** — đây là kỹ năng bắt buộc của mọi app hiển thị dữ liệu động.

---

## 🏠 **Bài tập về nhà (Tuần 2, Ngày 11)** 🎒

| Mức độ        | Bài tập                                              | Gợi ý           |
| ------------- | ---------------------------------------------------- | --------------- |
| 🟢 Cơ bản     | Hiển thị spinner nhỏ ở cuối khi đang tải thêm        | Thêm footerView |
| 🟡 Trung bình | Thêm label “Không còn dữ liệu” khi hết trang         |                 |
| 🔵 Nâng cao   | Dùng Combine hoặc async stream thay Observable       |                 |
| 🟣 Thử thách  | Gọi nhiều API phân trang thực tế (ví dụ newsapi.org) |                 |

---

## 📚 Tổng kết buổi học

| Kiến thức              | Vai trò                                |
| ---------------------- | -------------------------------------- |
| **UIRefreshControl**   | Tạo kéo để làm mới                     |
| **Pagination**         | Tải dữ liệu theo trang, tránh overload |
| **Async/Await**        | Đảm bảo không đơ UI khi tải nhiều lần  |
| **isLoadingMore flag** | Tránh gọi API chồng lặp                |
| **MVVM + Observable**  | UI tự động cập nhật dữ liệu mới        |

---

🎓 **Ngày 12 (buổi tới):**

> *Thầy sẽ dạy “Offline Caching với UserDefaults + Lưu danh sách sản phẩm để xem lại khi không có mạng” – giúp app hoạt động mượt ngay cả khi offline.*

---

👉 Em có muốn thầy **tiếp tục luôn Ngày 12 – Offline Cache bằng UserDefaults và Codable** không?
Bài này cực hay vì giúp app của em hoạt động **ngay cả khi không có Internet** — đúng chuẩn “real-world iOS app”.
