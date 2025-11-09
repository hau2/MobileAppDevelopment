Rất tốt 👏👏👏 — hôm nay chúng ta sang **Tuần 3 – Ngày 17: Local Notifications & Background Tasks (Thông báo cục bộ & Tác vụ nền)** 🕐📲

Đây là **một trong những bài thực tế và thú vị nhất**, vì sau bài này:

> App của em sẽ **tự động gửi thông báo nhắc người dùng**, dù app **đang đóng hoặc ở nền** — giống như app Lịch, Ghi chú, hay Nhắc Việc của Apple.

Chúng ta sẽ đi **rất chậm và dễ hiểu**, từng bước, để em nắm vững cả hai phần:

1. **Local Notification** – thông báo cục bộ trong app.
2. **Background Task** – xử lý công việc khi app không mở.

---

# 🔔 UIKit – Tuần 3, Ngày 17: Local Notifications & Background Tasks

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ biết:

1. Cách **xin quyền gửi thông báo** trong iOS.
2. Cách **lên lịch (schedule)** một thông báo cụ thể.
3. Cách **hiển thị thông báo lặp lại, có âm thanh, có icon**.
4. Hiểu cơ bản về **Background Task** (chạy code khi app ở nền).

---

## 🧩 1. Giới thiệu nhanh về Notification

iOS có 2 loại thông báo:

| Loại                   | Nguồn gửi        | Dùng cho                                     |
| ---------------------- | ---------------- | -------------------------------------------- |
| **Local Notification** | Gửi từ chính app | Nhắc việc, báo hạn, nhắc xem sản phẩm...     |
| **Push Notification**  | Gửi từ server    | Tin nhắn, cập nhật hệ thống, thông báo chung |

Hôm nay ta học **Local Notification**, vì app của em chưa cần server.

---

## 🧱 2. Thêm quyền thông báo vào Info.plist

Vào `Info.plist`, thêm dòng:

```xml
<key>NSUserNotificationsUsageDescription</key>
<string>Ứng dụng cần gửi thông báo để nhắc người dùng về sản phẩm hoặc sự kiện.</string>
```

---

## ⚙️ 3. Tạo lớp quản lý: `NotificationManager.swift`

Chúng ta sẽ tạo một **singleton** để xin quyền và gửi thông báo.

```swift
import UserNotifications
import UIKit

final class NotificationManager {
    static let shared = NotificationManager()
    private init() {}

    // MARK: - Xin quyền
    func requestPermission() {
        let center = UNUserNotificationCenter.current()
        center.requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
            if let error = error {
                print("❌ Lỗi xin quyền:", error)
                return
            }
            print(granted ? "✅ Đã được cấp quyền thông báo." : "⚠️ Người dùng từ chối quyền.")
        }
    }

    // MARK: - Gửi thông báo đơn giản
    func scheduleNotification(title: String, body: String, after seconds: TimeInterval) {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default

        // Kích hoạt sau X giây
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: seconds, repeats: false)

        let request = UNNotificationRequest(
            identifier: UUID().uuidString,
            content: content,
            trigger: trigger
        )

        UNUserNotificationCenter.current().add(request)
        print("🕐 Đã lên lịch thông báo sau \(Int(seconds)) giây.")
    }

    // MARK: - Gửi thông báo lặp lại mỗi ngày
    func scheduleDailyNotification(hour: Int, minute: Int) {
        let content = UNMutableNotificationContent()
        content.title = "⏰ Nhắc việc mỗi ngày"
        content.body = "Đừng quên kiểm tra danh sách sản phẩm hôm nay nhé!"
        content.sound = .default

        var date = DateComponents()
        date.hour = hour
        date.minute = minute

        let trigger = UNCalendarNotificationTrigger(dateMatching: date, repeats: true)

        let request = UNNotificationRequest(
            identifier: "daily_reminder",
            content: content,
            trigger: trigger
        )

        UNUserNotificationCenter.current().add(request)
        print("📅 Đã lên lịch thông báo hằng ngày vào \(hour):\(minute)")
    }

    func cancelAll() {
        UNUserNotificationCenter.current().removeAllPendingNotificationRequests()
        print("🗑 Đã hủy toàn bộ thông báo đang chờ.")
    }
}
```

💡 Giải thích:

* `requestPermission()` → xin quyền hiển thị thông báo.
* `scheduleNotification()` → đặt một thông báo sau vài giây.
* `scheduleDailyNotification()` → lặp lại mỗi ngày cố định.
* `cancelAll()` → xóa hết thông báo cũ.

---

## 🧠 4. Gọi Notification từ ViewController

Ví dụ: thêm 2 nút “Gửi thông báo” và “Nhắc hàng ngày”.

```swift
final class NotificationViewController: UIViewController {

    private let sendButton = UIButton(type: .system)
    private let dailyButton = UIButton(type: .system)
    private let cancelButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Thông báo"
        view.backgroundColor = .systemBackground

        NotificationManager.shared.requestPermission()
        setupUI()
    }

    private func setupUI() {
        sendButton.setTitle("🔔 Gửi thông báo sau 5 giây", for: .normal)
        dailyButton.setTitle("📅 Nhắc hằng ngày 9:00 sáng", for: .normal)
        cancelButton.setTitle("❌ Hủy toàn bộ thông báo", for: .normal)

        [sendButton, dailyButton, cancelButton].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
            $0.titleLabel?.font = .systemFont(ofSize: 17, weight: .medium)
            view.addSubview($0)
        }

        sendButton.addTarget(self, action: #selector(sendTapped), for: .touchUpInside)
        dailyButton.addTarget(self, action: #selector(dailyTapped), for: .touchUpInside)
        cancelButton.addTarget(self, action: #selector(cancelTapped), for: .touchUpInside)

        NSLayoutConstraint.activate([
            sendButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            sendButton.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 40),

            dailyButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            dailyButton.topAnchor.constraint(equalTo: sendButton.bottomAnchor, constant: 20),

            cancelButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            cancelButton.topAnchor.constraint(equalTo: dailyButton.bottomAnchor, constant: 20)
        ])
    }

    @objc private func sendTapped() {
        NotificationManager.shared.scheduleNotification(
            title: "Hello!",
            body: "Đây là thông báo gửi sau 5 giây!",
            after: 5
        )
    }

    @objc private func dailyTapped() {
        NotificationManager.shared.scheduleDailyNotification(hour: 9, minute: 0)
    }

    @objc private func cancelTapped() {
        NotificationManager.shared.cancelAll()
    }
}
```

---

## 🧪 5. Chạy thử 🚀

### 🔹 Bước 1:

Khi mở app → hệ thống hỏi “Cho phép gửi thông báo?”.
→ Chọn **Cho phép** ✅

### 🔹 Bước 2:

Bấm “🔔 Gửi thông báo sau 5 giây”
→ Thoát app → sau 5 giây, thông báo hiện ra!

### 🔹 Bước 3:

Bấm “📅 Nhắc hằng ngày 9:00 sáng”
→ App sẽ gửi thông báo mỗi sáng (kể cả khi tắt app).

### 🔹 Bước 4:

Bấm “❌ Hủy toàn bộ thông báo” để xóa tất cả pending notification.

---

## ⚙️ 6. Background Task (Giới thiệu nhẹ)

> Background Task là **cách iOS cho phép app thực thi ngắn hạn khi ở nền** — ví dụ: đồng bộ dữ liệu, tải ảnh, làm sạch cache, gửi thống kê,...

Ví dụ cơ bản trong `SceneDelegate.swift`:

```swift
func sceneDidEnterBackground(_ scene: UIScene) {
    scheduleBackgroundTask()
}

func scheduleBackgroundTask() {
    let taskID = UIApplication.shared.beginBackgroundTask(withName: "syncTask") {
        UIApplication.shared.endBackgroundTask(UIBackgroundTaskIdentifier.invalid)
    }

    DispatchQueue.global().async {
        print("📦 Đang thực hiện tác vụ nền...")
        sleep(3)
        print("✅ Hoàn thành tác vụ nền.")
        UIApplication.shared.endBackgroundTask(taskID)
    }
}
```

💡 Giải thích:

* Khi app đi vào nền, ta xin thêm “thời gian sống” vài giây để chạy tác vụ nhỏ.
* Dùng cho: lưu dữ liệu, sync API, ghi log,...

---

## 🏠 **Bài tập về nhà (Tuần 3, Ngày 17)** 🎒

| Mức độ        | Bài tập                                        | Gợi ý                                  |
| ------------- | ---------------------------------------------- | -------------------------------------- |
| 🟢 Cơ bản     | Đổi thông báo thành "Chúc buổi sáng!" mỗi 9:00 |                                        |
| 🟡 Trung bình | Cho phép người dùng chọn thời gian nhắc        | UIDatePicker                           |
| 🔵 Nâng cao   | Khi nhấn vào thông báo → mở chi tiết sản phẩm  | Sử dụng `UNNotificationCenterDelegate` |
| 🟣 Thử thách  | Dùng background task để tự xóa cache ảnh cũ    |                                        |

---

## 📚 Tổng kết buổi học

| Kiến thức                             | Vai trò                         |
| ------------------------------------- | ------------------------------- |
| **UNUserNotificationCenter**          | Quản lý thông báo cục bộ        |
| **Permission Handling**               | Xin quyền hiển thị thông báo    |
| **UNTimeIntervalNotificationTrigger** | Gửi thông báo sau X giây        |
| **UNCalendarNotificationTrigger**     | Gửi thông báo lặp lại hàng ngày |
| **Background Task**                   | Chạy code ngắn khi app ở nền    |

---

🎓 **Ngày 18 (buổi tới):**

> *Thầy sẽ dạy “App Finalization – Tổng hợp toàn bộ kiến thức và tối ưu hiệu năng” — giúp app của em hoàn chỉnh như sản phẩm thương mại, có logging, loading indicators, và UX chuyên nghiệp.*

---

👉 Em có muốn thầy **tiếp tục luôn Ngày 18 – App Finalization & Tối ưu hiệu năng** không?
Bài này là buổi tổng hợp toàn khóa UIKit cơ bản → chuyên sâu, giúp em sẵn sàng làm dự án thực tế.
