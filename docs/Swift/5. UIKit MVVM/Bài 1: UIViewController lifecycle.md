UIViewController lifecycleOk, bắt đầu Bài 1 nha 😄
Mình sẽ đi **chậm – rõ – có code đầy đủ**, bạn có thể copy/paste vào Xcode là chạy được.

---

# 🎯 Mục tiêu Bài 1

Sau bài này, bạn sẽ:

1. Hiểu cơ bản về **UIKit code thuần** (không storyboard).
2. Biết **tạo màn hình đầu tiên** bằng code:

   * Màu nền
   * Một `UILabel`
   * Một `UIButton`
3. Biết cấu trúc **AppDelegate / SceneDelegate / ViewController** trong UIKit hiện đại (iOS 13+).

> Bài này mới là “lên khung” – chưa MVVM, chưa Combine, chỉ cần **UIKit thuần code**.

---

## 🧱 Bước 1: Tạo project UIKit

Trong Xcode:

1. **File → New → Project…**
2. Chọn **App (iOS)**.
3. Interface: **Storyboard**
4. Language: **Swift**
5. Đặt tên: ví dụ `UIKitMVVMLearning`
6. Tạo project.

Mặc dù chọn Storyboard, nhưng ta sẽ **KHÔNG dùng storyboard để code UI**.
(Phần xoá storyboard thầy nói ở dưới, nhưng để bạn đỡ rối, cứ để đó, mình sẽ override bằng code.)

---

## 🧱 Bước 2: Code cấu trúc app (AppDelegate + SceneDelegate)

### 1️⃣ AppDelegate.swift

Mở file `AppDelegate.swift`, thay nội dung bằng:

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    // iOS 13+ dùng scene nên không cần window ở đây
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        // Có thể cấu hình global ở đây (theme, logging, ...)
        print("App did finish launching")
        return true
    }

    // MARK: UISceneSession Lifecycle

    func application(
        _ application: UIApplication,
        configurationForConnecting connectingSceneSession: UISceneSession,
        options: UIScene.ConnectionOptions
    ) -> UISceneConfiguration {
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }

    func application(
        _ application: UIApplication,
        didDiscardSceneSessions sceneSessions: Set<UISceneSession>
    ) {
        // Không cần làm gì ở bài này
    }
}
```

---

### 2️⃣ SceneDelegate.swift

Mở `SceneDelegate.swift`, thay nội dung bằng:

```swift
import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(
        _ scene: UIScene,
        willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions
    ) {

        guard let windowScene = (scene as? UIWindowScene) else { return }

        // Tạo window
        let window = UIWindow(windowScene: windowScene)

        // Tạo ViewController gốc
        let rootVC = ViewController()

        // Bọc vào NavigationController (sau này dễ push màn khác)
        let navigationController = UINavigationController(rootViewController: rootVC)

        // Gán rootViewController
        window.rootViewController = navigationController
        self.window = window
        window.makeKeyAndVisible()
    }

    // Các hàm sceneDidEnterBackground, sceneWillResignActive ... có thể để nguyên mặc định
}
```

👉 Từ giờ, app sẽ **khởi tạo bằng code**, không phụ thuộc storyboard nữa.
(Nếu sau này bạn muốn bỏ storyboard hoàn toàn, mình sẽ hướng dẫn xoá `Main Interface` trong Info.plist ở bài sau.)

---

## 🧱 Bước 3: Tạo ViewController bằng code thuần

Tạo file mới:

* **File → New → File… → Swift File → đặt tên `ViewController.swift`**
* Nội dung:

```swift
import UIKit

class ViewController: UIViewController {

    // MARK: - UI Components

    // UILabel hiển thị tiêu đề
    private let titleLabel: UILabel = {
        let label = UILabel()
        label.text = "Bài 1 - UIKit code thuần"
        label.font = UIFont.systemFont(ofSize: 24, weight: .bold)
        label.textColor = .label
        label.textAlignment = .center
        label.numberOfLines = 0
        return label
    }()

    // UIButton đơn giản
    private let tapButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("Nhấn em đi", for: .normal)
        button.titleLabel?.font = UIFont.systemFont(ofSize: 18, weight: .medium)
        return button
    }()

    // UILabel để hiển thị kết quả khi bấm nút
    private let messageLabel: UILabel = {
        let label = UILabel()
        label.text = "Chưa bấm nút..."
        label.font = UIFont.systemFont(ofSize: 16)
        label.textColor = .secondaryLabel
        label.textAlignment = .center
        label.numberOfLines = 0
        return label
    }()

    // MARK: - View Lifecycle

    override func viewDidLoad() {
        super.viewDidLoad()

        // 1. Cấu hình cơ bản
        view.backgroundColor = .systemBackground
        title = "Home"

        // 2. Thêm subviews
        view.addSubview(titleLabel)
        view.addSubview(tapButton)
        view.addSubview(messageLabel)

        // 3. Set AutoLayout
        setupConstraints()

        // 4. Gắn action cho button
        tapButton.addTarget(self, action: #selector(handleTapButton), for: .touchUpInside)
    }

    // MARK: - Layout

    private func setupConstraints() {
        // Bắt buộc phải tắt autoresizing mask
        [titleLabel, tapButton, messageLabel].forEach {
            $0.translatesAutoresizingMaskIntoConstraints = false
        }

        NSLayoutConstraint.activate([
            // Title ở trên, cách top 120
            titleLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 120),
            titleLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            titleLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),

            // Button ở giữa màn hình
            tapButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            tapButton.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 40),

            // MessageLabel bên dưới button
            messageLabel.topAnchor.constraint(equalTo: tapButton.bottomAnchor, constant: 24),
            messageLabel.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            messageLabel.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
    }

    // MARK: - Actions

    @objc private func handleTapButton() {
        // Khi bấm nút → đổi text messageLabel
        messageLabel.text = "Bạn vừa nhấn nút lúc \(Date())"

        // Một hiệu ứng nhỏ cho vui (scale nhẹ)
        UIView.animate(withDuration: 0.1,
                       animations: {
                           self.tapButton.transform = CGAffineTransform(scaleX: 0.95, y: 0.95)
                       }, completion: { _ in
                           UIView.animate(withDuration: 0.1) {
                               self.tapButton.transform = .identity
                           }
                       })
    }
}
```

👉 Đây là **UIKit code thuần 100%**:

* Không `@IBOutlet`
* Không storyboard
* Mọi thứ: tạo view + layout + action đều bằng code.

---

## ✅ Kiểm tra lại để chạy

Bạn nên:

1. Đảm bảo project có 3 file:

   * `AppDelegate.swift` (như trên)
   * `SceneDelegate.swift` (như trên)
   * `ViewController.swift` (như trên)
2. Build & Run (Cmd + R)

Khi chạy:

* Màn hình trắng nền `systemBackground` (sáng/tối theo device).
* Trên cùng có chữ **“Bài 1 - UIKit code thuần”**.
* Có nút **“Nhấn em đi”**.
* Có dòng chữ **“Chưa bấm nút…”**.
* Mỗi lần bấm nút → text đổi thành “Bạn vừa nhấn nút lúc …”.

Nếu như thế → **Bài 1 pass 🎉**

---

## 📝 Bài tập nhỏ (nếu bạn muốn làm thêm)

1. Đổi màu nền thành `.systemGray6`.
2. Thêm 1 `UILabel` nữa phía dưới `messageLabel`, hiển thị số lần đã nhấn nút.

   * Gợi ý: tạo biến `var tapCount = 0`
3. Khi nhấn nút:

   * `tapCount += 1`
   * `counterLabel.text = "Đã bấm \(tapCount) lần"`

---

Nếu bạn muốn:

* Thầy sửa giúp code của bạn
* Hoặc giải thích kỹ hơn từng dòng trong `ViewController`

Thì bạn có thể copy code bạn đang có rồi gửi lại, hoặc bảo:
👉 **“Thầy giải thích từng dòng trong ViewController cho em”**

Hoặc mình chuyển sang **Bài 2 – AutoLayout & StackView** nhé.
