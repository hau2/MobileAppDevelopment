Rất hay 👏👏👏 — hôm nay ta sang **Tuần 5 – Ngày 2: App Debug & Logging chuyên nghiệp (Xử lý lỗi và theo dõi ứng dụng)** 🧭🧠

Đây là **bài cực kỳ quan trọng** cho bất kỳ lập trình viên iOS nào bước sang dự án thật.
Vì sau khi em đã có app hoàn chỉnh (đăng nhập – sản phẩm – giỏ hàng – thanh toán…),
thì việc **bắt lỗi, theo dõi và ghi log** sẽ giúp:

> * Dễ phát hiện lỗi sớm.
> * Hiểu hành vi người dùng.
> * Giảm crash khi phát hành.
> * Và tạo “báo cáo chuyên nghiệp” khi có lỗi xảy ra.

Thầy sẽ dạy **rất chậm – dễ hiểu – kèm ví dụ thực tế trong dự án của em**.

---

# 🧭 UIKit – Tuần 5, Ngày 2: App Debug & Logging chuyên nghiệp

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Cách **ghi log chuẩn (OSLog / custom logger)**.
2. Cách **xử lý lỗi an toàn (try/catch, error propagation)**.
3. Cách **bắt lỗi crash và lưu file log**.
4. Cách **tạo “Chế độ Debug” riêng** để test app hiệu quả.

---

## 🧩 1. Tổng quan: 3 cấp độ “theo dõi ứng dụng”

| Cấp độ              | Mục tiêu                                            | Công cụ thường dùng             |
| ------------------- | --------------------------------------------------- | ------------------------------- |
| 1️⃣ Developer Log   | Theo dõi hành vi code trong Xcode                   | `print()`, `AppLogger`, `OSLog` |
| 2️⃣ User Log        | Lưu lỗi khi app chạy thật                           | `FileLogger`, `Crashlytics`     |
| 3️⃣ Live Monitoring | Theo dõi log người dùng thực tế (sau khi phát hành) | Firebase, Sentry, Instabug      |

Hôm nay ta làm 2 cấp đầu tiên (đủ cho mọi dự án demo & doanh nghiệp nhỏ).

---

## ⚙️ 2. Tạo Logger cơ bản (AppLogger)

Tạo file mới: `AppLogger.swift`

```swift
import Foundation
import os.log

enum LogLevel: String {
    case info = "ℹ️ INFO"
    case warning = "⚠️ WARNING"
    case error = "❌ ERROR"
}

final class AppLogger {

    static let subsystem = Bundle.main.bundleIdentifier ?? "MiniShop"
    static let osLogger = Logger(subsystem: subsystem, category: "App")

    /// Ghi log ra console (chỉ hiện khi Debug)
    static func log(_ message: String, level: LogLevel = .info) {
        #if DEBUG
        print("\(level.rawValue): \(message)")
        #endif

        switch level {
        case .info:
            osLogger.info("\(message)")
        case .warning:
            osLogger.warning("\(message)")
        case .error:
            osLogger.error("\(message)")
        }
    }
}
```

---

### 📌 Cách dùng

Trong mọi phần code, thay vì `print()`, ta dùng:

```swift
AppLogger.log("Tải danh sách sản phẩm thành công")
AppLogger.log("API trả về lỗi 404", level: .warning)
AppLogger.log("Không đọc được JSON", level: .error)
```

💡 Kết quả hiển thị:

```
ℹ️ INFO: Tải danh sách sản phẩm thành công
⚠️ WARNING: API trả về lỗi 404
❌ ERROR: Không đọc được JSON
```

---

## 🧠 3. Ghi log ra file thật (File Logger)

Đôi khi cần lưu lại log để đọc sau (đặc biệt khi app bị crash).
Ta sẽ mở rộng logger để ghi vào file “`AppLog.txt`”.

---

### 📂 Bổ sung vào `AppLogger.swift`

```swift
extension AppLogger {
    private static var logFileURL: URL {
        let dir = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        return dir.appendingPathComponent("AppLog.txt")
    }

    static func writeToFile(_ message: String) {
        let line = "[\(Date())] \(message)\n"
        if let data = line.data(using: .utf8) {
            if FileManager.default.fileExists(atPath: logFileURL.path) {
                if let handle = try? FileHandle(forWritingTo: logFileURL) {
                    handle.seekToEndOfFile()
                    handle.write(data)
                    handle.closeFile()
                }
            } else {
                try? data.write(to: logFileURL)
            }
        }
    }

    static func readLogs() -> String {
        (try? String(contentsOf: logFileURL)) ?? ""
    }

    static func clearLogs() {
        try? FileManager.default.removeItem(at: logFileURL)
    }
}
```

---

### 📋 Cách sử dụng

Khi xảy ra lỗi (ví dụ API lỗi):

```swift
AppLogger.log("API bị lỗi: \(error)", level: .error)
AppLogger.writeToFile("API lỗi: \(error.localizedDescription)")
```

💡 Sau đó em có thể mở file log:

```swift
print(AppLogger.readLogs())
```

---

## 🧱 4. Cấu trúc xử lý lỗi (Error Handling)

### ✳️ Ví dụ trong `AuthService.swift`

Thay vì bắt lỗi mơ hồ:

```swift
catch {
    print("Lỗi đăng nhập")
}
```

→ Nên làm rõ ràng và có log:

```swift
catch {
    AppLogger.log("Đăng nhập thất bại: \(error)", level: .error)
    AppLogger.writeToFile("Login failed: \(error.localizedDescription)")
    throw error
}
```

💡 Giúp em dễ biết lỗi do đâu (URL sai, JSON lỗi, mạng chậm, v.v.)

---

## ⚠️ 5. Bắt lỗi crash không mong muốn

Thêm đoạn này ở `AppDelegate.swift` (hoặc `SceneDelegate`):

```swift
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    NSSetUncaughtExceptionHandler { exception in
        let errorMsg = "Crash: \(exception.name.rawValue) – \(exception.reason ?? "Không rõ")"
        AppLogger.log(errorMsg, level: .error)
        AppLogger.writeToFile(errorMsg)
    }
    return true
}
```

💡 Khi app bị crash, log vẫn được ghi vào `AppLog.txt` để đọc lại sau.

---

## 🧰 6. Tạo "Chế độ Debug" riêng cho Developer

Đôi khi khi phát hành app, em muốn bật/tắt “chế độ debug” để xem log, token, dữ liệu nội bộ.

Tạo `DebugManager.swift`:

```swift
import UIKit

final class DebugManager {
    static var isDebugEnabled: Bool = true

    static func showDebugInfo(on viewController: UIViewController) {
        guard isDebugEnabled else { return }
        let logs = AppLogger.readLogs()
        let alert = UIAlertController(title: "🧾 Debug Logs", message: logs, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "Đóng", style: .cancel))
        alert.addAction(UIAlertAction(title: "Xóa log", style: .destructive) { _ in
            AppLogger.clearLogs()
        })
        viewController.present(alert, animated: true)
    }
}
```

💡 Thêm nút debug ẩn trong app, ví dụ trong `ProductListViewController`:

```swift
navigationItem.leftBarButtonItem = UIBarButtonItem(
    title: "🐞",
    style: .plain,
    target: self,
    action: #selector(showDebug)
)

@objc private func showDebug() {
    DebugManager.showDebugInfo(on: self)
}
```

✅ Khi nhấn “🐞”, app sẽ hiển thị toàn bộ log nội bộ – rất tiện khi test.

---

## 🧪 7. Kiểm tra hoạt động thực tế 🚀

1️⃣ Mở app → đăng nhập, thêm sản phẩm, thanh toán.
2️⃣ Tắt mạng → thử lại → xem log có ghi “Network error”.
3️⃣ Vào “🐞” → xem log đầy đủ trong app.
4️⃣ Đóng app → mở lại → log vẫn còn.

🎯 Khi app crash bất ngờ, mở `Documents/AppLog.txt` → vẫn lưu lại lỗi cuối cùng.

---

## 🏠 **Bài tập về nhà (Tuần 5, Ngày 2)** 🎒

| Mức độ        | Bài tập                                    | Gợi ý                               |
| ------------- | ------------------------------------------ | ----------------------------------- |
| 🟢 Cơ bản     | Ghi log lỗi API khi không có mạng          | Dùng `AppLogger.writeToFile`        |
| 🟡 Trung bình | Tạo menu Debug ẩn (chỉ bật khi giữ 3 giây) | Dùng `UILongPressGestureRecognizer` |
| 🔵 Nâng cao   | Lưu log bằng ngày tháng phân biệt file     | Thêm `DateFormatter("yyyy-MM-dd")`  |
| 🟣 Thử thách  | Gửi file log qua email                     | Dùng `UIActivityViewController`     |

---

## 📚 Tổng kết buổi học

| Kiến thức                   | Vai trò                                  |
| --------------------------- | ---------------------------------------- |
| **AppLogger + OSLog**       | Ghi log chuẩn và có cấp độ               |
| **File Logging**            | Lưu log để debug sau khi crash           |
| **Error Handling nâng cao** | Bắt lỗi rõ ràng và có ngữ cảnh           |
| **Crash Capture**           | Ghi lại lý do khi app sập                |
| **Debug Manager**           | Công cụ giúp kiểm tra nội bộ nhanh chóng |

---

🎓 **Ngày 3 (buổi tới – Tuần 5):**

> *Thầy sẽ dạy “App Monitoring & Feedback” — giúp app của em tự gửi báo cáo lỗi, đánh giá sao, và thu thập phản hồi người dùng chuyên nghiệp như ứng dụng thật.*

---

👉 Em có muốn thầy **tiếp tục luôn Tuần 5 – Ngày 3: App Monitoring & Feedback** không?
Bài này sẽ giúp em làm app “biết lắng nghe người dùng” — có popup đánh giá ⭐, gửi phản hồi 📧, và thống kê hoạt động người dùng như app thật.
