Rất tốt 👏👏👏 — hôm nay chúng ta sang **Tuần 3 – Ngày 16: App Lifecycle & Quyền truy cập (Camera, Photos, Permission Handling)** 🛡️📱

Bài này giúp em hiểu **vòng đời của ứng dụng iOS** và **cách quản lý quyền truy cập hệ thống (camera, thư viện ảnh, thông báo, v.v.)** — điều kiện **bắt buộc để app được duyệt lên App Store**.

---

# 🧭 UIKit – Tuần 3, Ngày 16: App Lifecycle & Quyền truy cập hệ thống

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Hiểu rõ **vòng đời (lifecycle)** của một ứng dụng iOS.
2. Biết cách **lắng nghe trạng thái app (active, background, terminated)**.
3. Cấu hình quyền **Camera / Photos / Microphone / Notifications** đúng chuẩn Apple.
4. Quản lý quyền **động bằng code** (runtime request).

---

## 🧩 1. Vòng đời ứng dụng iOS (App Lifecycle)

Mỗi app iOS có **3 trạng thái chính:**

| Trạng thái     | Ý nghĩa                                | Ví dụ thực tế                    |
| -------------- | -------------------------------------- | -------------------------------- |
| **Active**     | App đang chạy, người dùng tương tác    | Người dùng đang dùng app         |
| **Inactive**   | Chuẩn bị chuyển trạng thái             | Có cuộc gọi đến                  |
| **Background** | App chạy nền, tạm dừng UI              | Người dùng thoát ra Home         |
| **Suspended**  | App bị dừng hoàn toàn (không dùng CPU) | Sau vài phút không hoạt động     |
| **Terminated** | Hệ thống đóng app                      | Người dùng “swipe up” để tắt app |

---

## ⚙️ 2. Quan sát vòng đời bằng AppDelegate

Mở `AppDelegate.swift`, thêm các hàm sau:

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        print("🚀 App khởi động xong.")
        return true
    }

    func applicationDidBecomeActive(_ application: UIApplication) {
        print("✅ App đang hoạt động (Active).")
    }

    func applicationWillResignActive(_ application: UIApplication) {
        print("⏸ App sắp ngừng hoạt động (Inactive).")
    }

    func applicationDidEnterBackground(_ application: UIApplication) {
        print("🌙 App vào nền (Background).")
    }

    func applicationWillEnterForeground(_ application: UIApplication) {
        print("🌅 App sắp quay lại foreground.")
    }

    func applicationWillTerminate(_ application: UIApplication) {
        print("💀 App bị tắt hoàn toàn (Terminated).")
    }
}
```

💡 Em có thể chạy app, bấm **Home**, **mở lại**, hoặc **kill app**, rồi quan sát console log.

---

## 🧠 3. App Lifecycle trong SceneDelegate (từ iOS 13+)

Nếu app dùng `SceneDelegate.swift`, thêm:

```swift
func sceneWillEnterForeground(_ scene: UIScene) {
    print("🌅 Scene: vào foreground")
}

func sceneDidEnterBackground(_ scene: UIScene) {
    print("🌙 Scene: vào background")
}
```

⚠️ Lưu ý: iOS 13 trở đi, **AppDelegate** và **SceneDelegate** cùng tồn tại.

---

## 🧱 4. Quyền truy cập hệ thống (Permissions)

### 📸 Camera

Để dùng camera, phải khai báo trước trong **Info.plist**:

```xml
<key>NSCameraUsageDescription</key>
<string>Ứng dụng cần truy cập camera để chụp ảnh sản phẩm.</string>
```

### 🖼 Photos

Để chọn ảnh từ thư viện:

```xml
<key>NSPhotoLibraryUsageDescription</key>
<string>Ứng dụng cần quyền truy cập thư viện ảnh để chọn ảnh sản phẩm.</string>
```

### 🎙 Microphone (nếu cần ghi âm)

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Ứng dụng cần quyền truy cập micro để ghi âm.</string>
```

### 🔔 Notifications

```xml
<key>NSUserNotificationsUsageDescription</key>
<string>Ứng dụng cần gửi thông báo để nhắc người dùng.</string>
```

💡 Nếu **không khai báo**, app **sẽ crash** khi gọi chức năng tương ứng.

---

## ⚙️ 5. Kiểm tra & xin quyền Camera / Photos bằng code

Tạo lớp `PermissionManager.swift`:

```swift
import AVFoundation
import Photos
import UIKit

final class PermissionManager {
    static let shared = PermissionManager()
    private init() {}

    // MARK: - Camera
    func checkCameraPermission(completion: @escaping (Bool) -> Void) {
        let status = AVCaptureDevice.authorizationStatus(for: .video)
        switch status {
        case .authorized:
            completion(true)
        case .notDetermined:
            AVCaptureDevice.requestAccess(for: .video) { granted in
                DispatchQueue.main.async { completion(granted) }
            }
        default:
            completion(false)
        }
    }

    // MARK: - Photos
    func checkPhotoPermission(completion: @escaping (Bool) -> Void) {
        let status = PHPhotoLibrary.authorizationStatus(for: .readWrite)
        switch status {
        case .authorized, .limited:
            completion(true)
        case .notDetermined:
            PHPhotoLibrary.requestAuthorization { newStatus in
                DispatchQueue.main.async {
                    completion(newStatus == .authorized || newStatus == .limited)
                }
            }
        default:
            completion(false)
        }
    }

    func showSettingsAlert(on vc: UIViewController) {
        let alert = UIAlertController(
            title: "Quyền bị từ chối",
            message: "Vui lòng bật quyền truy cập trong Cài đặt để tiếp tục.",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "Mở Cài đặt", style: .default) { _ in
            if let url = URL(string: UIApplication.openSettingsURLString) {
                UIApplication.shared.open(url)
            }
        })
        alert.addAction(UIAlertAction(title: "Hủy", style: .cancel))
        vc.present(alert, animated: true)
    }
}
```

---

## 🧩 6. Gọi kiểm tra trước khi mở Camera

Trong `AddProductViewController`, thay đoạn mở camera bằng:

```swift
@objc private func selectImageTapped() {
    let alert = UIAlertController(title: "Chọn nguồn ảnh", message: nil, preferredStyle: .actionSheet)
    alert.addAction(UIAlertAction(title: "Thư viện", style: .default) { _ in
        PermissionManager.shared.checkPhotoPermission { granted in
            granted ? self.openImagePicker(source: .photoLibrary)
                    : PermissionManager.shared.showSettingsAlert(on: self)
        }
    })
    alert.addAction(UIAlertAction(title: "Camera", style: .default) { _ in
        PermissionManager.shared.checkCameraPermission { granted in
            granted ? self.openImagePicker(source: .camera)
                    : PermissionManager.shared.showSettingsAlert(on: self)
        }
    })
    alert.addAction(UIAlertAction(title: "Hủy", style: .cancel))
    present(alert, animated: true)
}
```

✅ Giờ app **tự động xin quyền lần đầu** và **nhắc người dùng mở Cài đặt** nếu bị từ chối.

---

## 🧪 7. Thử nghiệm thực tế 🚀

1. Chạy app, chọn **chụp ảnh** → hiện popup xin quyền Camera.
2. Từ chối → thử lại → hiện alert mở Cài đặt.
3. Cho phép → Camera hoạt động bình thường.
4. Tắt app → mở lại → log hiện trạng thái `background`, `active`, `terminate`.

🎉 Em vừa hoàn thiện tính năng **quản lý quyền truy cập chuẩn App Store** và hiểu sâu **App Lifecycle** của iOS.

---

## 🏠 **Bài tập về nhà (Tuần 3, Ngày 16)** 🎒

| Mức độ        | Bài tập                                                                                 | Gợi ý |
| ------------- | --------------------------------------------------------------------------------------- | ----- |
| 🟢 Cơ bản     | Thêm quyền “PhotoLibraryAddUsageDescription” để app có thể ghi ảnh vào thư viện         |       |
| 🟡 Trung bình | Lưu trạng thái quyền người dùng vào UserDefaults để không hỏi lại                       |       |
| 🔵 Nâng cao   | Hiển thị popup tùy chỉnh thay vì alert mặc định                                         |       |
| 🟣 Thử thách  | Tạo màn hình tổng hợp hiển thị trạng thái từng quyền (Camera, Photo, Mic, Notification) |       |

---

## 📚 Tổng kết buổi học

| Kiến thức                       | Vai trò                            |
| ------------------------------- | ---------------------------------- |
| **App Lifecycle**               | Quản lý trạng thái app             |
| **AppDelegate / SceneDelegate** | Bắt sự kiện chuyển trạng thái      |
| **Permission Handling**         | Xin và kiểm tra quyền hệ thống     |
| **Info.plist Usage Keys**       | Khai báo quyền bắt buộc            |
| **Open Settings**               | Hướng dẫn người dùng bật lại quyền |

---

🎓 **Ngày 17 (buổi tới):**

> *Thầy sẽ dạy “Local Notifications + App Background Tasks” — để app gửi thông báo nhắc người dùng và xử lý tác vụ ngầm (ví dụ: tải dữ liệu định kỳ, đồng bộ hóa).*

---

👉 Em có muốn thầy **tiếp tục luôn Ngày 17 – Local Notification + Background Task** không?
Đây là bài giúp app của em **tự động hoạt động khi người dùng không mở app**, như nhắc việc, gửi thông báo hoặc cập nhật dữ liệu định kỳ.
