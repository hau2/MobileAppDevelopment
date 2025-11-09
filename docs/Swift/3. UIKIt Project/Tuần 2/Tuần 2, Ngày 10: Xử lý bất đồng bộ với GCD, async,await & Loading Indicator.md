Rất tốt 👏👏👏 — hôm nay chúng ta **bước sang Tuần 2**, nghĩa là em đã chính thức rời khỏi giai đoạn “làm quen” và bước vào **lập trình UIKit chuyên sâu thực chiến**.

Trong tuần này, em sẽ được học những kỹ năng **bắt buộc để app chạy mượt, phản hồi nhanh**, không bị “đơ” khi tải dữ liệu hoặc ảnh.
Và bài hôm nay chính là nền tảng:

---

# ⚙️ UIKit – Tuần 2, Ngày 10: Xử lý bất đồng bộ với GCD, async/await & Loading Indicator

---

## 🎯 Mục tiêu buổi học

Sau buổi học này, em sẽ:

1. Hiểu khái niệm **luồng chính (main thread)** và **luồng nền (background thread)**.
2. Biết dùng **DispatchQueue (GCD)** để chạy tác vụ nền.
3. Biết dùng **async/await** trong Swift 5.5+ (cực gọn, hiện đại).
4. Biết cách **hiển thị Loading Indicator (UIActivityIndicatorView)** khi đang tải dữ liệu.

---

## 🧠 1. Vấn đề “đơ UI” trong app iOS

Khi em gọi API hoặc tải ảnh bằng `URLSession`, đó là **tác vụ tốn thời gian**.
Nếu ta chạy trên **main thread** (nơi xử lý UI), thì UI sẽ **bị đơ**, không cuộn được, người dùng tưởng app “treo”.

🧩 Giải pháp:

* **Chạy tác vụ nặng ở background thread (DispatchQueue.global)**
* Sau đó **quay lại main thread (DispatchQueue.main)** để cập nhật UI.

---

## ⚙️ 2. Giới thiệu GCD (Grand Central Dispatch)

GCD giúp em **quản lý luồng thực thi (thread)** trong Swift.
Cú pháp cơ bản:

```swift
DispatchQueue.global(qos: .background).async {
    // 👉 Chạy ở luồng nền
    let data = downloadSomething()

    DispatchQueue.main.async {
        // 👉 Quay về luồng chính để update UI
        self.label.text = data
    }
}
```

💡 GCD = cách “thủ công” nhưng rất mạnh, dùng được trong mọi phiên bản iOS.

---

## 🧩 3. Giới thiệu async/await (Swift 5.5+)

Từ iOS 15 trở lên, Swift có cú pháp hiện đại hơn, **async/await**, giúp viết code bất đồng bộ dễ đọc như code bình thường.

Ví dụ:

```swift
func fetchData() async throws -> [Product] {
    let url = URL(string: "https://fakestoreapi.com/products")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([Product].self, from: data)
}
```

Gọi hàm này:

```swift
Task {
    do {
        let products = try await fetchData()
        print(products.count)
    } catch {
        print("Lỗi:", error)
    }
}
```

✅ Không cần callback hay closure phức tạp.
✅ Dễ đọc, dễ test, code “phẳng” hơn nhiều.

---

## 🧱 4. Áp dụng vào `NetworkService.swift`

Ta cải tiến file này để hỗ trợ **async/await** song song với kiểu cũ:

```swift
import Foundation

protocol NetworkServiceProtocol {
    func fetchProducts(completion: @escaping (Result<[Product], Error>) -> Void)
    func fetchProductsAsync() async throws -> [Product]
}

final class NetworkService: NetworkServiceProtocol {
    func fetchProducts(completion: @escaping (Result<[Product], Error>) -> Void) {
        guard let url = URL(string: "https://fakestoreapi.com/products") else {
            completion(.failure(NSError(domain: "Invalid URL", code: 0)))
            return
        }

        URLSession.shared.dataTask(with: url) { data, _, error in
            if let error = error {
                completion(.failure(error))
                return
            }

            guard let data = data else {
                completion(.failure(NSError(domain: "No data", code: 0)))
                return
            }

            do {
                let products = try JSONDecoder().decode([Product].self, from: data)
                completion(.success(products))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }

    func fetchProductsAsync() async throws -> [Product] {
        let url = URL(string: "https://fakestoreapi.com/products")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([Product].self, from: data)
    }
}
```

---

## 🧩 5. Tạo Loading Indicator (UIActivityIndicatorView)

Chúng ta thêm vào `ProductListViewController` để app thân thiện hơn:

```swift
final class ProductListViewController: UIViewController {
    private let tableView = UITableView()
    private let spinner = UIActivityIndicatorView(style: .large)
    private let viewModel: ProductListViewModel

    // ... init, setupTableView giữ nguyên

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Sản phẩm"
        view.backgroundColor = .systemBackground
        setupTableView()
        setupSpinner()
        bindViewModel()
        fetchDataAsync()
    }

    private func setupSpinner() {
        spinner.translatesAutoresizingMaskIntoConstraints = false
        spinner.hidesWhenStopped = true
        view.addSubview(spinner)

        NSLayoutConstraint.activate([
            spinner.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            spinner.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    private func fetchDataAsync() {
        spinner.startAnimating()

        Task {
            do {
                let products = try await viewModel.fetchDataAsync()
                DispatchQueue.main.async {
                    self.spinner.stopAnimating()
                    self.viewModel.products.value = products
                }
            } catch {
                DispatchQueue.main.async {
                    self.spinner.stopAnimating()
                    print("❌ Lỗi:", error)
                }
            }
        }
    }
}
```

---

## 🧠 6. Cập nhật ViewModel hỗ trợ async

```swift
final class ProductListViewModel {
    private let networkService: NetworkServiceProtocol
    var products: Observable<[Product]> = Observable([])

    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
    }

    func fetchDataAsync() async throws -> [Product] {
        return try await networkService.fetchProductsAsync()
    }
}
```

---

## 🧪 7. Kết quả chạy thử 🚀

✅ Khi mở app → **Loading Indicator xoay** giữa màn hình.
✅ Sau vài giây → danh sách sản phẩm hiện ra.
✅ UI luôn mượt, không “treo”.
✅ Kéo xuống refresh → vẫn mượt.
✅ Network chậm → app vẫn phản hồi bình thường.

🎉 Em vừa hiểu và áp dụng **bất đồng bộ hiện đại (async/await)** trong UIKit — một kỹ năng quan trọng mà hầu hết app thực tế đều cần.

---

## 🏠 **Bài tập về nhà (Tuần 2, Ngày 10)** 🎒

| Mức độ        | Bài tập                                    | Gợi ý                      |
| ------------- | ------------------------------------------ | -------------------------- |
| 🟢 Cơ bản     | Thêm nút “Tải lại” ở navigation bar        | Gọi lại `fetchDataAsync()` |
| 🟡 Trung bình | Hiển thị thông báo khi lỗi mạng            | Dùng `UIAlertController`   |
| 🔵 Nâng cao   | Dùng `async let` để tải nhiều API cùng lúc | 2 endpoint khác nhau       |
| 🟣 Thử thách  | Dùng TaskGroup để gọi song song 3 API      | In kết quả ra console      |

---

## 📚 Tổng kết buổi học

| Khái niệm                   | Vai trò                               |
| --------------------------- | ------------------------------------- |
| **Main Thread**             | Nơi chạy UI, phải luôn mượt           |
| **DispatchQueue**           | Quản lý luồng nền và luồng chính      |
| **async/await**             | Cú pháp bất đồng bộ hiện đại, dễ đọc  |
| **UIActivityIndicatorView** | Loading spinner trong UIKit           |
| **MVVM + Async**            | Mẫu chuẩn để kết nối dữ liệu với View |

---

🎓 **Ngày 11 (buổi tới):**

> *Thầy sẽ dạy “Kéo để làm mới (Pull to Refresh) + Pagination (Tải thêm khi cuộn)” — kỹ năng bắt buộc trong mọi app hiển thị danh sách lớn.*

---

👉 Em có muốn thầy **tiếp tục luôn Ngày 11 – Pull to Refresh + Pagination** không?
Thầy sẽ dạy thật chậm, minh họa bằng code cụ thể và giải thích luồng chạy từng bước để em thấy rõ cách UIKit xử lý danh sách lớn nhé.
