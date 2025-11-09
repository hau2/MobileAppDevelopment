Rất tốt 👏👏👏 — hôm nay chúng ta sang **Tuần 3 – Ngày 18: App Finalization & Tối ưu hiệu năng (App tối ưu và hoàn thiện)** 🚀

Đây là **bài tổng kết toàn bộ khóa UIKit cơ bản – trung cấp – thực chiến**, giúp em:
✅ Gom tất cả những gì đã học thành **một ứng dụng hoàn chỉnh**.
✅ Hiểu cách **tối ưu tốc độ, bộ nhớ, và UX** để app chạy mượt.
✅ Chuẩn bị kiến thức để **chuyển sang dự án thực tế hoặc App Store.**

Thầy sẽ đi **chậm, rõ ràng, dễ hiểu từng phần**, vì bài này vừa quan trọng, vừa là nền tảng để em tự tin làm app thực chiến.

---

# 🚀 UIKit – Tuần 3, Ngày 18: App Finalization & Tối ưu hiệu năng

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Cách **tổ chức dự án chuẩn MVVM + Coordinator**.
2. Cách **giảm lag, tiết kiệm bộ nhớ, tránh crash.**
3. Cách **thêm Loading Indicator, Empty State, Alert hợp lý.**
4. Cách **ghi log (logging), xử lý lỗi, và chuẩn bị cho App Store.**

---

## 🧩 1. Ôn lại cấu trúc chuẩn của dự án

Cấu trúc thư mục chuẩn cho app UIKit hiện đại:

```
ProductApp/
├── Application/
│   ├── AppDelegate.swift
│   ├── SceneDelegate.swift
│   ├── Coordinator/
│   └── Extensions/
│
├── Data/
│   ├── Models/
│   ├── Network/
│   ├── CoreData/
│   ├── Cache/
│
├── Features/
│   ├── ProductList/
│   │   ├── View/
│   │   ├── ViewModel/
│   │   ├── Controller/
│   ├── ProductDetail/
│   ├── AddProduct/
│
├── Resources/
│   ├── Assets.xcassets
│   ├── Info.plist
│
└── Utils/
    ├── ImageStorageManager.swift
    ├── NotificationManager.swift
    ├── PermissionManager.swift
```

💡 Lợi ích:

* Mỗi tính năng riêng biệt → dễ mở rộng, bảo trì.
* Dễ test từng module → chuyên nghiệp như team iOS thật.

---

## 🧠 2. Loading Indicator (hiển thị khi đang tải dữ liệu)

Khi gọi API, người dùng **phải biết app đang làm việc** → tránh ấn nhiều lần.

Tạo `LoadingIndicator.swift`:

```swift
import UIKit

final class LoadingIndicator {
    static let shared = LoadingIndicator()
    private var spinner: UIActivityIndicatorView?

    func show(on view: UIView) {
        DispatchQueue.main.async {
            if self.spinner == nil {
                let spinner = UIActivityIndicatorView(style: .large)
                spinner.center = view.center
                spinner.startAnimating()
                spinner.color = .systemBlue
                view.addSubview(spinner)
                self.spinner = spinner
            }
        }
    }

    func hide() {
        DispatchQueue.main.async {
            self.spinner?.stopAnimating()
            self.spinner?.removeFromSuperview()
            self.spinner = nil
        }
    }
}
```

💬 Giải thích:

* Dùng singleton để hiển thị spinner ở bất kỳ màn hình nào.
* Gọi `LoadingIndicator.shared.show(on: self.view)` trước khi fetch dữ liệu,
  và `LoadingIndicator.shared.hide()` sau khi hoàn tất.

---

## 🧱 3. Empty State (khi không có dữ liệu hiển thị)

Trong `ProductListViewController`, thêm hàm:

```swift
private func updateEmptyState() {
    if viewModel.products.value.isEmpty {
        let label = UILabel()
        label.text = "📭 Không có sản phẩm nào."
        label.textAlignment = .center
        label.textColor = .systemGray
        label.font = .systemFont(ofSize: 17, weight: .medium)
        tableView.backgroundView = label
    } else {
        tableView.backgroundView = nil
    }
}
```

Và gọi nó sau khi reload dữ liệu:

```swift
viewModel.products.bind { [weak self] _ in
    self?.tableView.reloadData()
    self?.updateEmptyState()
}
```

💡 Người dùng sẽ thấy một thông điệp thân thiện khi chưa có dữ liệu.

---

## ⚙️ 4. Ghi log & quản lý lỗi (Error Handling)

Tạo `AppLogger.swift`:

```swift
import Foundation

enum LogLevel: String {
    case info = "ℹ️"
    case warning = "⚠️"
    case error = "❌"
}

final class AppLogger {
    static func log(_ message: String, level: LogLevel = .info) {
        #if DEBUG
        print("\(level.rawValue) [\(Date())] \(message)")
        #endif
    }
}
```

Sử dụng:

```swift
AppLogger.log("Bắt đầu tải dữ liệu")
AppLogger.log("Không có kết nối mạng", level: .warning)
AppLogger.log("Lỗi phân tích JSON", level: .error)
```

💡 Khi build bản release, các log sẽ tự ẩn — giữ app sạch và an toàn.

---

## 🧩 5. Tối ưu hiệu năng TableView (rất quan trọng)

| Vấn đề           | Cách khắc phục                                           |
| ---------------- | -------------------------------------------------------- |
| Ảnh load chậm    | Dùng Image Cache hoặc thư viện `SDWebImage`              |
| Scroll lag       | Tải ảnh nền bằng async queue                             |
| Cell nặng        | Giới hạn số lượng subviews trong cell                    |
| Reload toàn bảng | Dùng `reloadRows(at:)` thay vì `reloadData()` khi có thể |
| Dữ liệu lớn      | Áp dụng pagination (đã học ở Ngày 11)                    |

Ví dụ: tải ảnh async trong cell:

```swift
func configure(with product: Product) {
    titleLabel.text = product.title
    priceLabel.text = "$\(String(format: "%.2f", product.price))"

    productImageView.image = UIImage(systemName: "photo")
    DispatchQueue.global().async {
        if let url = URL(string: product.image),
           let data = try? Data(contentsOf: url),
           let image = UIImage(data: data) {
            DispatchQueue.main.async {
                self.productImageView.image = image
            }
        }
    }
}
```

---

## ⚙️ 6. Bộ nhớ & ARC (Automatic Reference Counting)

💡 Nguyên tắc cơ bản:

* **Mỗi object** được giữ lại (retain) khi có biến tham chiếu đến.
* Khi không ai giữ → hệ thống tự giải phóng.

Để tránh **retain cycle**:

* Dùng `[weak self]` trong closure.
* Với ViewModel → tránh giữ ViewController.

Ví dụ:

```swift
viewModel.products.bind { [weak self] _ in
    self?.tableView.reloadData()
}
```

---

## 📱 7. UX tối ưu: Alert + Feedback

Khi người dùng lưu sản phẩm:

```swift
let alert = UIAlertController(
    title: "🎉 Thành công",
    message: "Sản phẩm đã được lưu!",
    preferredStyle: .alert
)
alert.addAction(UIAlertAction(title: "OK", style: .default))
present(alert, animated: true)
```

Kết hợp với **haptic feedback**:

```swift
let generator = UINotificationFeedbackGenerator()
generator.notificationOccurred(.success)
```

💡 Người dùng cảm thấy “app có hồn” hơn khi có phản hồi xúc giác.

---

## 🧩 8. Chuẩn bị cho App Store

Khi app hoàn thiện:

1. Xóa hoặc tắt các **log debug**.
2. Kiểm tra lại **quyền Info.plist** (Camera, Photos, Notification...).
3. Chạy **Product → Archive** trong Xcode.
4. Kiểm tra bằng **TestFlight** trước khi gửi duyệt App Store.

---

## 🏠 **Bài tập về nhà (Tuần 3, Ngày 18)** 🎒

| Mức độ        | Bài tập                                                       | Gợi ý |
| ------------- | ------------------------------------------------------------- | ----- |
| 🟢 Cơ bản     | Thêm LoadingIndicator vào khi fetch API                       |       |
| 🟡 Trung bình | Hiển thị Empty State khi không có sản phẩm                    |       |
| 🔵 Nâng cao   | Ghi log cho từng bước xử lý                                   |       |
| 🟣 Thử thách  | Dùng Instruments (trong Xcode) để đo bộ nhớ và CPU khi scroll |       |

---

## 📚 Tổng kết buổi học

| Kiến thức                    | Vai trò                                    |
| ---------------------------- | ------------------------------------------ |
| **LoadingIndicator**         | Cải thiện trải nghiệm tải dữ liệu          |
| **Empty State**              | Giao diện thân thiện khi không có nội dung |
| **Logging & Error Handling** | Theo dõi lỗi và hành vi                    |
| **ARC & Weak Reference**     | Giữ app không rò rỉ bộ nhớ                 |
| **UX Feedback & Alert**      | Nâng cao trải nghiệm người dùng            |
| **App Store Prep**           | Chuẩn bị phát hành sản phẩm thật           |

---

🎓 **Tuần 4 (sắp tới):**

> *Thầy sẽ bắt đầu chuỗi “Dự án thực tế – Xây dựng Mini E-Commerce App hoàn chỉnh” với: đăng nhập, giỏ hàng, thanh toán, và lưu trữ bằng CoreData + API giả lập.*

---

👉 Em có muốn thầy **bắt đầu Tuần 4 – Dự án thực tế (Mini E-Commerce App)** luôn không?
Bài tới, thầy sẽ hướng dẫn em thiết kế **màn hình đăng nhập (Login)** và **luồng người dùng (User Flow)** đầu tiên, vẫn 100% bằng **UIKit code thuần**, cực kỳ dễ hiểu.
