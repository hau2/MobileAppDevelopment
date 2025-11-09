Rất tốt 👏👏👏 — hôm nay thầy và em bước sang **Tuần 2 – Ngày 12: Lưu dữ liệu offline với UserDefaults + Codable**

Đây là **bước cực kỳ quan trọng** để app có thể hoạt động ngay cả khi **không có Internet** — điều mà các app thực tế (Shopee, Zalo, Facebook, v.v.) đều phải làm.

Sau bài này, app của em sẽ:

* Tự lưu danh sách sản phẩm đã tải xuống.
* Khi khởi động lại hoặc mất mạng, vẫn hiển thị dữ liệu cũ.
* Hoạt động hoàn toàn **offline-first**.

---

# 💾 UIKit – Tuần 2, Ngày 12: Offline Caching bằng UserDefaults + Codable

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Hiểu nguyên lý **offline caching (lưu đệm dữ liệu)**.
2. Biết dùng **UserDefaults + Codable** để lưu danh sách sản phẩm.
3. Tạo **CacheManager** chuyên trách lưu & đọc dữ liệu.
4. Kết hợp caching với ViewModel → app vừa online vừa offline.

---

## 🧠 1. Khái niệm “Offline Caching”

Khi người dùng mở app:

1. App tải dữ liệu từ **cache (local)** trước → hiển thị ngay.
2. Sau đó gọi API **cập nhật dữ liệu mới**.
3. Nếu mạng yếu → app vẫn có dữ liệu hiển thị (offline).

💡 Cơ chế này giúp app **chạy nhanh hơn**, **ổn định hơn**, **trải nghiệm người dùng mượt hơn**.

---

## 🧱 2. Tạo lớp `CacheManager.swift`

```swift
import Foundation

final class CacheManager {
    static let shared = CacheManager()
    private let defaults = UserDefaults.standard
    private let cacheKey = "cached_products"

    private init() {}

    func saveProducts(_ products: [Product]) {
        do {
            let data = try JSONEncoder().encode(products)
            defaults.set(data, forKey: cacheKey)
            print("✅ Đã lưu \(products.count) sản phẩm vào cache.")
        } catch {
            print("❌ Lỗi khi lưu cache:", error)
        }
    }

    func loadProducts() -> [Product] {
        guard let data = defaults.data(forKey: cacheKey) else { return [] }
        do {
            let products = try JSONDecoder().decode([Product].self, from: data)
            print("✅ Đã tải \(products.count) sản phẩm từ cache.")
            return products
        } catch {
            print("❌ Lỗi khi đọc cache:", error)
            return []
        }
    }

    func clearCache() {
        defaults.removeObject(forKey: cacheKey)
        print("🗑️ Đã xóa cache.")
    }
}
```

💬 Giải thích:

* Dữ liệu được lưu ở `UserDefaults` dưới dạng `Data` (đã mã hóa JSON).
* Khi khởi động app, ta đọc lại bằng `loadProducts()`.
* `CacheManager.shared` là singleton dùng chung toàn app.

---

## ⚙️ 3. Cập nhật `ProductListViewModel.swift`

Ta thêm logic:

* Khi tải dữ liệu từ API → lưu xuống cache.
* Khi khởi động → nếu cache có → hiển thị ngay.

```swift
final class ProductListViewModel {
    private let networkService: NetworkServiceProtocol
    var products: Observable<[Product]> = Observable([])

    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
        loadFromCache()
    }

    private func loadFromCache() {
        let cached = CacheManager.shared.loadProducts()
        if !cached.isEmpty {
            products.value = cached
        }
    }

    func fetchDataAsync(reset: Bool = false) async throws -> [Product] {
        let allProducts = try await networkService.fetchProductsAsync()
        products.value = allProducts
        CacheManager.shared.saveProducts(allProducts)
        return products.value
    }

    func addNewProduct(_ product: Product) {
        products.value.append(product)
        CacheManager.shared.saveProducts(products.value)
    }
}
```

💡 Giải thích:

* Khi khởi tạo, ViewModel tự động **tải cache cũ** nếu có.
* Khi có dữ liệu mới từ API hoặc thêm sản phẩm, tự **ghi lại cache**.
* Không cần thao tác thủ công, app tự lưu – tự đọc.

---

## 🧩 4. Cập nhật `ProductListViewController.swift`

Không cần thay đổi nhiều, chỉ cần đảm bảo:

* Khi mở app, ViewModel đã tự load cache.
* Lần đầu vào app, nếu không có mạng → vẫn thấy dữ liệu trước đó.

---

## 🧪 5. Kiểm tra thực tế 🚀

### 🔹 Bước 1:

Chạy app lần đầu (có mạng):

* Dữ liệu tải từ API thật.
* Console in:

  ```
  ✅ Đã lưu 20 sản phẩm vào cache.
  ```

### 🔹 Bước 2:

Tắt Internet, chạy lại app:

* App vẫn hiển thị danh sách đầy đủ.
* Console in:

  ```
  ✅ Đã tải 20 sản phẩm từ cache.
  ```

🎉 App của em giờ **hoạt động được cả online & offline** —
đây là tiêu chuẩn của mọi app iOS chuyên nghiệp!

---

## 🧩 6. Thêm nút “Xóa cache” (tuỳ chọn)

Để luyện tập, thêm một nút ở navigation bar:

```swift
navigationItem.leftBarButtonItem = UIBarButtonItem(
    title: "Clear Cache",
    style: .plain,
    target: self,
    action: #selector(clearCacheTapped)
)

@objc private func clearCacheTapped() {
    CacheManager.shared.clearCache()
}
```

---

## 🏠 **Bài tập về nhà (Tuần 2, Ngày 12)** 🎒

| Mức độ        | Bài tập                                       | Gợi ý                     |
| ------------- | --------------------------------------------- | ------------------------- |
| 🟢 Cơ bản     | Thêm hiển thị ngày cache cuối cùng            | Lưu `Date()` cùng dữ liệu |
| 🟡 Trung bình | Xóa cache sau 3 ngày                          | So sánh thời gian lưu     |
| 🔵 Nâng cao   | Lưu cache bằng file JSON thay vì UserDefaults | Dùng `FileManager`        |
| 🟣 Thử thách  | Tự động xóa cache khi user logout             |                           |

---

## 📚 Tổng kết buổi học

| Thành phần               | Vai trò                         |
| ------------------------ | ------------------------------- |
| **UserDefaults**         | Lưu dữ liệu nhỏ, nhanh, nhẹ     |
| **Codable**              | Chuyển object → Data (JSON)     |
| **CacheManager**         | Quản lý đọc/ghi dữ liệu offline |
| **MVVM + Cache**         | App mượt, chạy offline          |
| **Offline-first design** | Chuẩn app hiện đại, UX cao      |

---

🎓 **Ngày 13 (buổi tới):**

> *Thầy sẽ dạy “Lưu trữ dữ liệu nâng cao bằng CoreData – tạo, đọc, cập nhật, xóa (CRUD)” — để em quản lý dữ liệu lớn, có quan hệ và bền vững hơn UserDefaults.*

---

👉 Em có muốn thầy **tiếp tục luôn Ngày 13 – Lưu trữ dữ liệu với CoreData (CRUD)** không?
Bài này là “bước đột phá”, giúp em từ app demo tiến sang **app có database thực thụ** — như quản lý người dùng, sản phẩm, đơn hàng thật.
