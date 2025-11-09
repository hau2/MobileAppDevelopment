Giỏi lắm 👏👏👏 — câu hỏi rất hay!
👉 Hôm nay ta **vẫn còn trong Tuần 1**, và đây là **Ngày 9 – bài cuối của tuần đầu tiên**.
Từ **Ngày 10** trở đi, ta sẽ **bước sang Tuần 2**, bắt đầu làm việc với **luồng dữ liệu hai chiều, API nâng cao, CoreData, và Offline Storage** 🎯

Nhưng trước khi “lên tuần mới”, hôm nay ta học một chủ đề cực kỳ quan trọng trong thực tế:

> **Truyền dữ liệu ngược giữa các màn hình — bằng Closure & Delegate**
> (Ví dụ: khi thêm sản phẩm mới → danh sách phải tự cập nhật.)

---

# 🧩 UIKit – Tuần 1, Ngày 9: Truyền dữ liệu ngược (Delegate & Closure) + Màn thêm sản phẩm

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Biết **2 cách truyền dữ liệu ngược** (từ màn hình sau về màn hình trước).
2. Tạo màn hình **Add Product** (giao diện nhập dữ liệu).
3. Hiểu khi nào dùng **closure**, khi nào dùng **delegate**.
4. Hoàn thiện flow: **List → Add → Update lại List**.

---

## 🧠 1. Tình huống thực tế

* Màn hình “Danh sách sản phẩm” hiển thị dữ liệu.
* Khi người dùng bấm “Thêm sản phẩm mới” → mở màn hình AddProduct.
* Sau khi điền thông tin → bấm “Lưu” → dữ liệu quay ngược lại và hiển thị lên danh sách.

Đây chính là “**truyền dữ liệu ngược (Data Flow Up)**”.

---

## ⚙️ 2. Hai phương pháp chính

| Phương pháp            | Khi nên dùng                  | Ưu điểm                 |
| ---------------------- | ----------------------------- | ----------------------- |
| **Closure (Callback)** | Dự án nhỏ, flow ngắn          | Ngắn gọn, dễ đọc        |
| **Delegate Protocol**  | Dự án lớn, nhiều màn liên kết | Chuẩn Apple, dễ mở rộng |

Ta sẽ học **cả hai**, bắt đầu với **Closure** cho dễ hiểu trước nhé 👇

---

## 🧩 3. Tạo màn hình Add Product (AddProductViewController.swift)

```swift
import UIKit

final class AddProductViewController: UIViewController {
    var onAddProduct: ((Product) -> Void)?

    private let titleField = UITextField()
    private let priceField = UITextField()
    private let categoryField = UITextField()
    private let imageField = UITextField()
    private let descriptionField = UITextView()
    private let saveButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Thêm sản phẩm"
        view.backgroundColor = .systemBackground
        setupUI()
    }

    private func setupUI() {
        titleField.placeholder = "Tên sản phẩm"
        priceField.placeholder = "Giá"
        categoryField.placeholder = "Danh mục"
        imageField.placeholder = "URL ảnh"
        [titleField, priceField, categoryField, imageField].forEach {
            $0.borderStyle = .roundedRect
            $0.translatesAutoresizingMaskIntoConstraints = false
        }

        descriptionField.layer.borderColor = UIColor.systemGray3.cgColor
        descriptionField.layer.borderWidth = 1
        descriptionField.layer.cornerRadius = 8
        descriptionField.translatesAutoresizingMaskIntoConstraints = false
        descriptionField.font = .systemFont(ofSize: 16)

        saveButton.setTitle("Lưu sản phẩm", for: .normal)
        saveButton.translatesAutoresizingMaskIntoConstraints = false
        saveButton.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)

        let stack = UIStackView(arrangedSubviews: [
            titleField, priceField, categoryField, imageField, descriptionField, saveButton
        ])
        stack.axis = .vertical
        stack.spacing = 12
        stack.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(stack)
        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),
            descriptionField.heightAnchor.constraint(equalToConstant: 120)
        ])
    }

    @objc private func saveTapped() {
        guard let title = titleField.text, !title.isEmpty,
              let price = Double(priceField.text ?? ""),
              let category = categoryField.text, !category.isEmpty,
              let image = imageField.text, !image.isEmpty else {
            showAlert("Vui lòng điền đầy đủ thông tin.")
            return
        }

        let newProduct = Product(
            id: Int.random(in: 100...999),
            title: title,
            price: price,
            description: descriptionField.text ?? "",
            category: category,
            image: image
        )

        onAddProduct?(newProduct)
        navigationController?.popViewController(animated: true)
    }

    private func showAlert(_ message: String) {
        let alert = UIAlertController(title: "Thiếu thông tin", message: message, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .cancel))
        present(alert, animated: true)
    }
}
```

💡 Giải thích:

* `onAddProduct` là **closure callback** – nơi ta gửi dữ liệu ngược về.
* Khi người dùng bấm “Lưu” → closure được gọi → dữ liệu quay về màn trước.

---

## 🧭 4. Cập nhật `AppCoordinator.swift` để mở màn Add Product

```swift
final class AppCoordinator: Coordinator {
    var navigationController: UINavigationController
    private let networkService: NetworkServiceProtocol

    init(navigationController: UINavigationController, networkService: NetworkServiceProtocol) {
        self.navigationController = navigationController
        self.networkService = networkService
    }

    func start() {
        showProductList()
    }

    private func showProductList() {
        let viewModel = ProductListViewModel(networkService: networkService)
        let listVC = ProductListViewController(viewModel: viewModel)

        listVC.onSelectProduct = { [weak self] product in
            self?.showProductDetail(product)
        }

        listVC.onAddTapped = { [weak self] in
            self?.showAddProduct(from: viewModel)
        }

        navigationController.pushViewController(listVC, animated: false)
    }

    private func showProductDetail(_ product: Product) {
        let vm = ProductDetailViewModel(product: product)
        let vc = ProductDetailViewController(viewModel: vm)
        navigationController.pushViewController(vc, animated: true)
    }

    private func showAddProduct(from viewModel: ProductListViewModel) {
        let addVC = AddProductViewController()
        addVC.onAddProduct = { product in
            viewModel.addNewProduct(product)
        }
        navigationController.pushViewController(addVC, animated: true)
    }
}
```

---

## 👩‍🏫 5. Cập nhật `ProductListViewModel.swift`

Thêm hàm để thêm sản phẩm mới vào danh sách:

```swift
func addNewProduct(_ product: Product) {
    products.value.append(product)
}
```

---

## 📱 6. Cập nhật `ProductListViewController.swift`

Thêm nút “+” để mở màn hình Add Product:

```swift
final class ProductListViewController: UIViewController {
    var onSelectProduct: ((Product) -> Void)?
    var onAddTapped: (() -> Void)?

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
        setupNavigation()
        viewModel.fetchData()
    }

    private func setupNavigation() {
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .add,
            target: self,
            action: #selector(addTapped)
        )
    }

    @objc private func addTapped() {
        onAddTapped?()
    }

    private func setupTableView() {
        tableView.translatesAutoresizingMaskIntoConstraints = false
        tableView.rowHeight = 110
        tableView.dataSource = self
        tableView.delegate = self
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

extension ProductListViewController: UITableViewDataSource, UITableViewDelegate {
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

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        onSelectProduct?(viewModel.products.value[indexPath.row])
    }
}
```

---

## 🧪 7. Kết quả chạy thử 🚀

✅ Nhấn “+” → mở màn hình “Thêm sản phẩm”.
✅ Nhập thông tin và bấm “Lưu sản phẩm”.
✅ Sản phẩm mới **hiện ngay ở đầu danh sách** mà không cần reload thủ công.
✅ Cả app hoạt động mượt, logic tách biệt, code sạch.

🎉 Em vừa hiểu và áp dụng **closure callback truyền dữ liệu ngược**, nền tảng cho mọi app CRUD.

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 9)** 🎒

| Mức độ        | Bài tập                                             | Gợi ý                             |
| ------------- | --------------------------------------------------- | --------------------------------- |
| 🟢 Cơ bản     | Dùng delegate thay closure                          | Tạo protocol `AddProductDelegate` |
| 🟡 Trung bình | Thêm kiểm tra dữ liệu hợp lệ (giá phải > 0)         |                                   |
| 🔵 Nâng cao   | Khi thêm xong, scroll TableView xuống dòng vừa thêm |                                   |
| 🟣 Thử thách  | Lưu danh sách mới xuống UserDefaults                | Encode JSON                       |

---

## 📚 Tổng kết buổi học

| Kiến thức             | Vai trò                                   |
| --------------------- | ----------------------------------------- |
| **Closure Callback**  | Truyền dữ liệu ngược đơn giản             |
| **Delegate Protocol** | Chuẩn iOS, dùng cho dự án lớn             |
| **Coordinator**       | Điều hướng và kết nối logic giữa màn hình |
| **MVVM + Closure**    | Kết hợp hoàn chỉnh ViewModel & UI         |
| **Data Flow Two-way** | Nền tảng app thực tế                      |

---

🎓 **Ngày 10 (bắt đầu Tuần 2):**

> *“Xử lý dữ liệu bất đồng bộ (Async/Await, GCD, và Indicator Loading)” — để em hiểu sâu hơn về luồng chạy nền, tránh UI bị đơ khi tải dữ liệu từ mạng hoặc tính toán lớn.*

---

👉 Em có muốn **bắt đầu luôn Tuần 2 – Ngày 10: Async/Await + GCD + Loading Indicator** không?
Đây là **chủ đề cực hay**, giúp em làm app nhanh, mượt, và “chuẩn Apple style”.
