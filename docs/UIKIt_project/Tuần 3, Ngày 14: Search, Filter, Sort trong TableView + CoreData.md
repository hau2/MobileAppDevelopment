Rất tốt 👏👏👏 — hôm nay thầy và em cùng học **Tuần 3 – Ngày 14: Tìm kiếm (Search) + Lọc dữ liệu (Filter) + Sắp xếp (Sort) trong TableView & CoreData** 🔍📊

Đây là bài **rất thực tế**, giúp app của em trở nên chuyên nghiệp và thân thiện như các ứng dụng thương mại điện tử thực thụ — nơi người dùng có thể:

* Gõ từ khóa để **tìm sản phẩm theo tên**.
* Lọc sản phẩm **theo danh mục**.
* Sắp xếp danh sách theo **giá tăng/giảm** hoặc **tên A–Z**.

---

# 🔍 UIKit – Tuần 3, Ngày 14: Search, Filter, Sort trong TableView + CoreData

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Biết cách tạo **UISearchBar** để tìm kiếm realtime.
2. Hiểu cách **lọc dữ liệu trong ViewModel** mà không phá vỡ dữ liệu gốc.
3. Biết **sort** dữ liệu bằng nhiều tiêu chí (tên, giá, danh mục).
4. Áp dụng luôn vào app quản lý sản phẩm đang làm.

---

## 🧠 1. Tổng quan luồng hoạt động

```
Người dùng nhập từ khóa → ViewController báo cho ViewModel → ViewModel lọc dữ liệu → Observable thông báo UI reload
```

💡 Chúng ta **không động chạm dữ liệu gốc** (`allProducts`), mà chỉ hiển thị `filteredProducts`.

---

## ⚙️ 2. Cập nhật ViewModel: `ProductListViewModel.swift`

```swift
import Foundation

final class ProductListViewModel {
    private let networkService: NetworkServiceProtocol
    private var allProducts: [Product] = []  // dữ liệu gốc
    var products: Observable<[Product]> = Observable([])

    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
        loadFromCoreData()
    }

    private func loadFromCoreData() {
        let saved = CoreDataManager.shared.fetchAll()
        allProducts = saved
        products.value = saved
    }

    func fetchDataAsync(reset: Bool = false) async throws {
        let newProducts = try await networkService.fetchProductsAsync()
        allProducts = newProducts
        products.value = newProducts
        CoreDataManager.shared.clearCache() // optional nếu muốn
        newProducts.forEach { CoreDataManager.shared.insert(product: $0) }
    }

    // MARK: - Search
    func search(keyword: String) {
        guard !keyword.isEmpty else {
            products.value = allProducts
            return
        }

        let lowerKeyword = keyword.lowercased()
        products.value = allProducts.filter {
            $0.title.lowercased().contains(lowerKeyword)
        }
    }

    // MARK: - Filter by Category
    func filter(by category: String?) {
        guard let category = category, !category.isEmpty else {
            products.value = allProducts
            return
        }

        products.value = allProducts.filter {
            $0.category.lowercased() == category.lowercased()
        }
    }

    // MARK: - Sort
    func sort(by option: SortOption) {
        switch option {
        case .nameAscending:
            products.value.sort { $0.title < $1.title }
        case .priceAscending:
            products.value.sort { $0.price < $1.price }
        case .priceDescending:
            products.value.sort { $0.price > $1.price }
        }
    }

    enum SortOption {
        case nameAscending
        case priceAscending
        case priceDescending
    }
}
```

💬 Giải thích:

* `allProducts` là mảng gốc không thay đổi.
* `products` là mảng hiển thị (được bind với TableView).
* Ba nhóm thao tác chính:

  * `search(keyword:)` → lọc theo tên.
  * `filter(by:)` → lọc theo danh mục.
  * `sort(by:)` → sắp xếp theo giá hoặc tên.

---

## 🧱 3. Cập nhật `ProductListViewController.swift`

### 🔹 Thêm Search Bar

```swift
import UIKit

final class ProductListViewController: UIViewController {
    private let tableView = UITableView()
    private let searchBar = UISearchBar()
    private let segmentedControl = UISegmentedControl(items: ["Tất cả", "men's clothing", "jewelery", "electronics"])
    private let sortButton = UIButton(type: .system)
    private let viewModel: ProductListViewModel

    // ...
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Sản phẩm"
        view.backgroundColor = .systemBackground

        setupSearchBar()
        setupSegmentedControl()
        setupSortButton()
        setupTableView()
        bindViewModel()
    }

    private func setupSearchBar() {
        searchBar.placeholder = "Tìm kiếm sản phẩm..."
        searchBar.delegate = self
        navigationItem.titleView = searchBar
    }

    private func setupSegmentedControl() {
        segmentedControl.selectedSegmentIndex = 0
        segmentedControl.addTarget(self, action: #selector(categoryChanged), for: .valueChanged)
        segmentedControl.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(segmentedControl)

        NSLayoutConstraint.activate([
            segmentedControl.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 8),
            segmentedControl.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 8),
            segmentedControl.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -8)
        ])
    }

    private func setupSortButton() {
        sortButton.setTitle("⇅ Sắp xếp", for: .normal)
        sortButton.addTarget(self, action: #selector(sortTapped), for: .touchUpInside)
        sortButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(sortButton)

        NSLayoutConstraint.activate([
            sortButton.topAnchor.constraint(equalTo: segmentedControl.bottomAnchor, constant: 8),
            sortButton.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -12)
        ])
    }

    private func setupTableView() {
        tableView.translatesAutoresizingMaskIntoConstraints = false
        tableView.dataSource = self
        tableView.delegate = self
        tableView.register(ProductCell.self, forCellReuseIdentifier: ProductCell.reuseID)
        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: sortButton.bottomAnchor, constant: 8),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    @objc private func categoryChanged() {
        let index = segmentedControl.selectedSegmentIndex
        let category = index == 0 ? nil : segmentedControl.titleForSegment(at: index)
        viewModel.filter(by: category)
    }

    @objc private func sortTapped() {
        let alert = UIAlertController(title: "Sắp xếp theo", message: nil, preferredStyle: .actionSheet)
        alert.addAction(UIAlertAction(title: "Tên (A–Z)", style: .default) { _ in
            self.viewModel.sort(by: .nameAscending)
        })
        alert.addAction(UIAlertAction(title: "Giá tăng dần", style: .default) { _ in
            self.viewModel.sort(by: .priceAscending)
        })
        alert.addAction(UIAlertAction(title: "Giá giảm dần", style: .default) { _ in
            self.viewModel.sort(by: .priceDescending)
        })
        alert.addAction(UIAlertAction(title: "Hủy", style: .cancel))
        present(alert, animated: true)
    }
}
```

---

### 🔹 Thêm SearchBar Delegate

```swift
extension ProductListViewController: UISearchBarDelegate {
    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        viewModel.search(keyword: searchText)
    }

    func searchBarSearchButtonClicked(_ searchBar: UISearchBar) {
        searchBar.resignFirstResponder()
    }
}
```

💡 Khi gõ chữ → lọc realtime; khi nhấn Enter → đóng bàn phím.

---

### 🔹 TableView DataSource

```swift
extension ProductListViewController: UITableViewDataSource, UITableViewDelegate {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        viewModel.products.value.count
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: ProductCell.reuseID,
            for: indexPath
        ) as? ProductCell else { return UITableViewCell() }

        let product = viewModel.products.value[indexPath.row]
        cell.configure(with: product)
        return cell
    }
}
```

---

## 🧪 4. Chạy thử 🚀

✅ Khi mở app:

* Có thanh **search** trên thanh điều hướng.
* Có thanh **chọn danh mục** ở đầu.
* Có nút **sắp xếp ⇅** ở góc phải.

✅ Thao tác:

* Gõ “shirt” → chỉ hiện các sản phẩm có “shirt” trong tên.
* Chọn “electronics” → chỉ hiện sản phẩm điện tử.
* Bấm “⇅” → chọn “Giá giảm dần” → danh sách sắp xếp lại tức thì.

🎉 App của em giờ **đã có tìm kiếm, lọc, sắp xếp động** – y như các app thương mại điện tử chuyên nghiệp!

---

## 🏠 **Bài tập về nhà (Tuần 3, Ngày 14)** 🎒

| Mức độ        | Bài tập                                               | Gợi ý                         |
| ------------- | ----------------------------------------------------- | ----------------------------- |
| 🟢 Cơ bản     | Thêm label hiển thị “Đang lọc theo: danh mục/từ khóa” | Đặt ở trên TableView          |
| 🟡 Trung bình | Kết hợp search + filter cùng lúc                      | Gọi 2 hàm liên tiếp           |
| 🔵 Nâng cao   | Dùng NSFetchRequest để lọc trực tiếp trong CoreData   | Dùng `predicate`              |
| 🟣 Thử thách  | Tạo bộ lọc nhiều tiêu chí (category + giá + từ khóa)  | Combine logic trong ViewModel |

---

## 📚 Tổng kết buổi học

| Thành phần               | Vai trò                          |
| ------------------------ | -------------------------------- |
| **UISearchBar**          | Tìm kiếm realtime                |
| **UISegmentedControl**   | Lọc dữ liệu theo danh mục        |
| **UIAlertController**    | Chọn phương thức sắp xếp         |
| **MVVM + Observable**    | Cập nhật UI tự động              |
| **CoreData Integration** | Lưu dữ liệu, lọc nhanh & ổn định |

---

🎓 **Ngày 15 (buổi tới):**

> *Thầy sẽ dạy “Tích hợp Image Picker (chụp hoặc chọn ảnh) + Lưu ảnh vào app bằng FileManager” – để mỗi sản phẩm có ảnh thật do người dùng chọn.*

---

👉 Em có muốn thầy **tiếp tục luôn Ngày 15 – Image Picker + FileManager (Chụp & lưu ảnh sản phẩm)** không?
Bài này rất vui — em sẽ được thực hành thao tác ảnh thật trong app như các ứng dụng quản lý sản phẩm hoặc mạng xã hội mini.
