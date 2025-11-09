Tốt lắm 👏👏👏 — hôm nay thầy và em bước sang **Tuần 1 – Ngày 6: Networking trong UIKit – Gọi API thật bằng URLSession và Codable**.
Đây là **một trong những kỹ năng bắt buộc** của iOS Developer, vì gần như **mọi ứng dụng thực tế đều phải lấy dữ liệu từ server** — như danh sách sản phẩm, học sinh, bài viết, tin tức, v.v.

Thầy sẽ dạy **rất chậm, dễ hiểu, từng bước một** để em nắm chắc nền tảng, không bị “ngợp” như nhiều người khi mới học gọi API.

---

# 🌐 UIKit – Tuần 1, Ngày 6: Gọi API bằng URLSession và Codable

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Hiểu nguyên lý hoạt động của **Networking** trong iOS.
2. Biết cách gọi API thật bằng **URLSession**.
3. Biết cách parse JSON thành model bằng **Codable**.
4. Biết cách hiển thị dữ liệu từ API lên TableView.

---

## 🧠 1. Networking là gì?

“Networking” nghĩa là **ứng dụng của em kết nối với máy chủ (server)** để lấy dữ liệu.
Quá trình này gồm 3 bước:

```
1️⃣ Gửi yêu cầu (Request) →  
2️⃣ Nhận phản hồi (Response, dạng JSON) →  
3️⃣ Giải mã JSON → hiển thị lên UI
```

---

## 💡 2. API thật để chúng ta luyện tập

Ta sẽ dùng API thật miễn phí:

🔗 `https://fakestoreapi.com/products`

Khi em mở link này, sẽ thấy dữ liệu JSON dạng:

```json
[
  {
    "id": 1,
    "title": "Fjallraven Backpack",
    "price": 109.95,
    "description": "Your perfect pack for everyday use",
    "category": "men's clothing",
    "image": "https://fakestoreapi.com/img/81fPKd-2AYL._AC_SL1500_.jpg"
  },
  ...
]
```

---

## 🧱 3. Tạo model để nhận dữ liệu

Tạo file `Product.swift` trong thư mục **Models**:

```swift
import Foundation

struct Product: Codable {
    let id: Int
    let title: String
    let price: Double
    let description: String
    let category: String
    let image: String
}
```

> ✅ `Codable` là “bộ giải mã tự động” trong Swift, giúp ta chuyển đổi JSON → object cực dễ.

---

## ⚙️ 4. Tạo lớp `NetworkService` để gọi API

Tạo file `NetworkService.swift` trong thư mục **Services**:

```swift
import Foundation

protocol NetworkServiceProtocol {
    func fetchProducts(completion: @escaping (Result<[Product], Error>) -> Void)
}

final class NetworkService: NetworkServiceProtocol {
    func fetchProducts(completion: @escaping (Result<[Product], Error>) -> Void) {
        guard let url = URL(string: "https://fakestoreapi.com/products") else {
            completion(.failure(NSError(domain: "Invalid URL", code: 0)))
            return
        }

        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            // Nếu lỗi kết nối
            if let error = error {
                completion(.failure(error))
                return
            }

            // Kiểm tra dữ liệu
            guard let data = data else {
                completion(.failure(NSError(domain: "No data", code: 0)))
                return
            }

            do {
                // Giải mã JSON
                let products = try JSONDecoder().decode([Product].self, from: data)
                completion(.success(products))
            } catch {
                completion(.failure(error))
            }
        }

        task.resume() // Bắt đầu gọi API
    }
}
```

💬 Giải thích:

* `URLSession.shared.dataTask` là **hàm gọi mạng** trong iOS.
* Dữ liệu trả về dạng `Data` (nhị phân), ta **giải mã bằng JSONDecoder** thành `[Product]`.
* Dùng `Result` để báo **thành công (success)** hoặc **thất bại (failure)**.

---

## 👩‍🏫 5. Tạo ViewModel để quản lý dữ liệu

Tạo `ProductListViewModel.swift`:

```swift
import Foundation

final class ProductListViewModel {
    private let networkService: NetworkServiceProtocol
    var products: Observable<[Product]> = Observable([])

    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
    }

    func fetchData() {
        networkService.fetchProducts { [weak self] result in
            switch result {
            case .success(let data):
                self?.products.value = data
            case .failure(let error):
                print("❌ Lỗi tải dữ liệu:", error)
            }
        }
    }
}
```

✅ Khi ViewModel gọi `fetchData()`, nó sẽ tự động lấy danh sách sản phẩm và cập nhật vào `Observable`, từ đó **UI sẽ tự động reload** (nhờ bài học về *binding* hôm qua).

---

## 🧩 6. Tạo `ProductListViewController.swift`

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
        tableView.dataSource = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
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
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        let product = viewModel.products.value[indexPath.row]
        cell.textLabel?.text = product.title
        return cell
    }
}
```

✅ Khi app chạy:

* Màn hình gọi `viewModel.fetchData()`.
* ViewModel gọi `networkService.fetchProducts()`.
* Khi có dữ liệu → Observable thay đổi → TableView tự reload.

---

## 🧱 7. Cấu hình entry point

Cập nhật `SceneDelegate.swift`:

```swift
func scene(_ scene: UIScene,
            willConnectTo session: UISceneSession,
            options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }
    let window = UIWindow(windowScene: windowScene)
    let nav = UINavigationController()

    let network = NetworkService()
    let viewModel = ProductListViewModel(networkService: network)
    let vc = ProductListViewController(viewModel: viewModel)
    nav.viewControllers = [vc]

    window.rootViewController = nav
    window.makeKeyAndVisible()
    self.window = window
}
```

---

## 🧪 8. Chạy thử 🧠

✅ App khởi động → hiện danh sách sản phẩm thật từ API.
✅ Mỗi hàng hiển thị tên sản phẩm.
✅ Mở Console → thấy log “fetch thành công”.
✅ Internet chậm → TableView vẫn ổn (không crash).

🎉 Em vừa hoàn thiện **ứng dụng UIKit đầu tiên gọi API thật** bằng `URLSession` và `Codable`.

---

## 🏠 **Bài tập về nhà (Tuần 1, Ngày 6)** 🎒

| Mức độ        | Bài tập                                        | Gợi ý                                      |
| ------------- | ---------------------------------------------- | ------------------------------------------ |
| 🟢 Cơ bản     | Hiển thị thêm `price` dưới tên sản phẩm        | `cell.detailTextLabel`                     |
| 🟡 Trung bình | Tạo `ProductCell` tùy chỉnh có ảnh + tên + giá | Dùng `UIImageView` + `URLSession.dataTask` |
| 🔵 Nâng cao   | Thêm `ActivityIndicator` khi đang tải dữ liệu  | Hiển thị khi `fetchData` chưa xong         |
| 🟣 Thử thách  | Tạo cache ảnh sản phẩm (UIImage cache)         | Dùng `NSCache` hoặc `URLCache`             |

---

## 📚 Tổng kết buổi học

| Thành phần            | Vai trò                                |
| --------------------- | -------------------------------------- |
| **URLSession**        | Gọi API HTTP trong iOS                 |
| **Codable**           | Giải mã JSON thành object Swift        |
| **Observable**        | Binding dữ liệu giữa ViewModel và View |
| **`Result<T, Error>`**  | Trả kết quả thành công / lỗi           |
| **MVVM + Networking** | Mẫu chuẩn trong app thật               |

---

🎓 **Ngày 7 (buổi tới):**

> *Ta sẽ học “Hiển thị ảnh từ URL + tạo custom cell chuyên nghiệp trong TableView”, kết hợp cache để app nhanh hơn và tiết kiệm dữ liệu mạng.*

---

👉 Em có muốn thầy tiếp tục luôn **Ngày 7 – Custom Cell + Hiển thị ảnh từ URL + Caching ảnh** không?
Thầy sẽ hướng dẫn thật dễ hiểu, từng dòng một — đây là bài cực vui, vì app của em bắt đầu **hiện ảnh sản phẩm thật, giống app thương mại điện tử mini**.
