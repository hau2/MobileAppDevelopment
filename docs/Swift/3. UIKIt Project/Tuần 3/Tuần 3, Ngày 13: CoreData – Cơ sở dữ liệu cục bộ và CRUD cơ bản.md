Tốt lắm 👏👏👏 — hôm nay thầy và em **chính thức bước sang Tuần 3 – Ngày 13: CoreData – Lưu trữ dữ liệu nâng cao (CRUD)**

Đây là **cột mốc quan trọng** giúp em bước từ “ứng dụng demo” sang **ứng dụng thực tế có cơ sở dữ liệu thật** 💾
Sau bài này, em sẽ hiểu và làm được:

* Tạo database cục bộ (offline bền vững).
* Thêm / sửa / xóa / truy vấn dữ liệu ngay trong app.
* Lưu trữ hàng nghìn bản ghi mà không lo mất khi tắt app.

---

# 💾 UIKit – Tuần 3, Ngày 13: CoreData – Cơ sở dữ liệu cục bộ và CRUD cơ bản

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ:

1. Hiểu rõ **CoreData là gì và khác gì UserDefaults**.
2. Biết tạo **Data Model, Entity, Attribute**.
3. Biết CRUD (Create, Read, Update, Delete) bằng code.
4. Áp dụng vào app quản lý sản phẩm mà ta đã xây từ tuần trước.

---

## 🧠 1. CoreData là gì?

CoreData là **ORM (Object Relational Mapping)** của Apple, giúp ta:

* Lưu object Swift như “Product”, “User”, “Note”… vào database thật (SQLite).
* Tự động ánh xạ giữa **object** ↔ **bảng trong DB**.
* Tìm kiếm, lọc, sắp xếp, cập nhật nhanh và an toàn.

🟢 **UserDefaults** → Dành cho dữ liệu nhỏ, dạng cặp key–value.
🔵 **CoreData** → Dành cho dữ liệu lớn, có cấu trúc, quan hệ.

---

## ⚙️ 2. Chuẩn bị Project CoreData

1️⃣ Trong Xcode, mở `ProductApp.xcodeproj`.
2️⃣ Vào menu **File → New → File → Data Model → ProductApp.xcdatamodeld**
3️⃣ Trong model này, tạo **Entity mới** tên `ProductEntity`.

**Thuộc tính (Attributes):**

| Tên      | Kiểu dữ liệu |
| -------- | ------------ |
| id       | Integer 64   |
| title    | String       |
| price    | Double       |
| desc     | String       |
| category | String       |
| image    | String       |

💡 Em có thể xem nó như bảng “Product” trong database thật.

---

## 🧩 3. Tạo CoreDataManager.swift

Tạo lớp quản lý chung cho mọi truy vấn CoreData.

```swift
import CoreData
import UIKit

final class CoreDataManager {
    static let shared = CoreDataManager()
    private init() {}

    // MARK: - Persistent Container
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "ProductApp")
        container.loadPersistentStores { (_, error) in
            if let error = error {
                fatalError("❌ Lỗi load CoreData: \(error)")
            }
        }
        return container
    }()

    var context: NSManagedObjectContext {
        persistentContainer.viewContext
    }

    // MARK: - Save Context
    func saveContext() {
        if context.hasChanges {
            do {
                try context.save()
                print("✅ Đã lưu thay đổi vào CoreData.")
            } catch {
                print("❌ Lỗi khi lưu context:", error)
            }
        }
    }
}
```

💬 Giải thích:

* `NSPersistentContainer` là “trái tim” của CoreData, quản lý toàn bộ DB.
* `context` là nơi ta thao tác CRUD.
* `saveContext()` dùng để commit thay đổi xuống DB thật.

---

## 🧩 4. Tạo model ánh xạ: ProductEntity+Extension.swift

```swift
import CoreData

extension ProductEntity {
    func toProduct() -> Product {
        Product(
            id: Int(self.id),
            title: self.title ?? "",
            price: self.price,
            description: self.desc ?? "",
            category: self.category ?? "",
            image: self.image ?? ""
        )
    }

    func update(from product: Product) {
        self.id = Int64(product.id)
        self.title = product.title
        self.price = product.price
        self.desc = product.description
        self.category = product.category
        self.image = product.image
    }
}
```

💡 Mục tiêu:

* Chuyển đổi giữa **ProductEntity (CoreData)** và **Product (Swift struct)**.
* Khi đọc từ DB → `toProduct()`
* Khi ghi → `update(from:)`

---

## ⚙️ 5. Viết CRUD trong CoreDataManager

```swift
extension CoreDataManager {
    // CREATE
    func insert(product: Product) {
        let entity = ProductEntity(context: context)
        entity.update(from: product)
        saveContext()
    }

    // READ
    func fetchAll() -> [Product] {
        let request: NSFetchRequest<ProductEntity> = ProductEntity.fetchRequest()
        do {
            return try context.fetch(request).map { $0.toProduct() }
        } catch {
            print("❌ Lỗi fetch:", error)
            return []
        }
    }

    // UPDATE
    func update(product: Product) {
        let request: NSFetchRequest<ProductEntity> = ProductEntity.fetchRequest()
        request.predicate = NSPredicate(format: "id == %d", product.id)
        do {
            if let entity = try context.fetch(request).first {
                entity.update(from: product)
                saveContext()
            }
        } catch {
            print("❌ Lỗi update:", error)
        }
    }

    // DELETE
    func delete(product: Product) {
        let request: NSFetchRequest<ProductEntity> = ProductEntity.fetchRequest()
        request.predicate = NSPredicate(format: "id == %d", product.id)
        do {
            if let entity = try context.fetch(request).first {
                context.delete(entity)
                saveContext()
            }
        } catch {
            print("❌ Lỗi delete:", error)
        }
    }
}
```

✅ Em đã có đầy đủ CRUD:

* `insert()` → Thêm sản phẩm
* `fetchAll()` → Đọc danh sách
* `update()` → Cập nhật
* `delete()` → Xóa

---

## 🧩 6. Áp dụng vào `ProductListViewModel`

Thêm CoreData song song với API & cache:

```swift
final class ProductListViewModel {
    private let networkService: NetworkServiceProtocol
    var products: Observable<[Product]> = Observable([])

    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
        loadFromCoreData()
    }

    private func loadFromCoreData() {
        let saved = CoreDataManager.shared.fetchAll()
        if !saved.isEmpty {
            products.value = saved
        }
    }

    func fetchDataAsync(reset: Bool = false) async throws -> [Product] {
        let newProducts = try await networkService.fetchProductsAsync()
        products.value = newProducts

        // Lưu xuống CoreData
        newProducts.forEach {
            CoreDataManager.shared.insert(product: $0)
        }

        return products.value
    }

    func deleteProduct(_ product: Product) {
        CoreDataManager.shared.delete(product: product)
        products.value.removeAll { $0.id == product.id }
    }
}
```

💡 Khi app tải dữ liệu mới:

* Hiển thị ngay từ DB.
* Sau đó tải online và cập nhật DB.

---

## 🧪 7. Kiểm tra thực tế 🚀

### 🔹 Bước 1:

Chạy app (có mạng) → sản phẩm hiển thị, CoreData ghi lại.
Console:

```
✅ Đã lưu thay đổi vào CoreData.
```

### 🔹 Bước 2:

Tắt mạng, mở lại app → sản phẩm vẫn còn.
Console:

```
✅ Đã tải dữ liệu từ CoreData.
```

🎉 App của em giờ **chính thức có database thật** — lưu dữ liệu lâu dài, mượt và ổn định.

---

## 🏠 **Bài tập về nhà (Tuần 3, Ngày 13)** 🎒

| Mức độ        | Bài tập                                   | Gợi ý                 |
| ------------- | ----------------------------------------- | --------------------- |
| 🟢 Cơ bản     | Thêm nút “Xóa sản phẩm” trong danh sách   | Gọi `deleteProduct()` |
| 🟡 Trung bình | Thêm tính năng “Sửa thông tin sản phẩm”   | Dùng `update()`       |
| 🔵 Nâng cao   | Thêm trường “ngày thêm” (Date) vào Entity | Kiểu Date             |
| 🟣 Thử thách  | Tạo quan hệ 1-nhiều (Category – Product)  | Entity liên kết       |

---

## 📚 Tổng kết buổi học

| Kiến thức           | Vai trò                                 |
| ------------------- | --------------------------------------- |
| **CoreData**        | Cơ sở dữ liệu cục bộ mạnh mẽ trên iOS   |
| **NSManagedObject** | Object ánh xạ sang DB                   |
| **NSFetchRequest**  | Truy vấn dữ liệu                        |
| **CRUD**            | Tạo, đọc, cập nhật, xóa dữ liệu         |
| **MVVM + CoreData** | App hoạt động online/offline hoàn chỉnh |

---

🎓 **Ngày 14 (buổi tới):**

> *Thầy sẽ dạy “Tìm kiếm (SearchBar) + Lọc dữ liệu (Filter) + Sắp xếp (Sort)” — để app của em có chức năng tìm nhanh, lọc theo danh mục, và sắp xếp giá tăng/giảm.*

---

👉 Em có muốn thầy **tiếp tục luôn Ngày 14 – Tìm kiếm & Lọc dữ liệu trong TableView + CoreData** không?
Bài này rất thú vị, vì ta sẽ làm **search bar hoạt động realtime** và **lọc dữ liệu trực tiếp từ CoreData** giống các app chuyên nghiệp.
