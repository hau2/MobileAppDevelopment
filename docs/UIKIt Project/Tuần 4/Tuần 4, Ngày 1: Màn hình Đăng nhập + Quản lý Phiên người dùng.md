Tuyệt vời 👏👏👏 — hôm nay ta **bắt đầu Tuần 4 – Ngày 1: Màn hình Đăng nhập (Login) + Quản lý Phiên người dùng (User Session)**

Đây là **bước mở đầu của Dự án thực tế Mini E-Commerce App** 🛍️
Sau bài này, em sẽ có **màn hình đăng nhập thật**, gọi API tới `https://fakestoreapi.com/auth/login`,
xác thực người dùng, lưu token, và chuyển sang **màn hình sản phẩm chính**.

---

# 🔐 UIKit – Tuần 4, Ngày 1: Màn hình Đăng nhập + Quản lý Phiên người dùng

---

## 🎯 Mục tiêu buổi học

Sau bài này, em sẽ biết:

1. Tạo form đăng nhập bằng UIKit (không storyboard).
2. Gọi API login thật bằng `URLSession`.
3. Validate dữ liệu người dùng nhập.
4. Lưu token đăng nhập bằng `UserDefaults`.
5. Chuyển hướng sang màn hình chính sau khi login thành công.

---

## 🧠 1. Chuẩn bị dữ liệu test

FakeStoreAPI cung cấp user thật:

| username | password |
| -------- | -------- |
| mor_2314 | 83r5^_   |
| johnd    | m38rmF$  |

Ta sẽ dùng user này để test đăng nhập.

API endpoint:

```
POST https://fakestoreapi.com/auth/login
Body (JSON):
{
  "username": "mor_2314",
  "password": "83r5^_"
}
```

Response:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6..."
}
```

---

## ⚙️ 2. Tạo `AuthService.swift`

Đây là lớp chịu trách nhiệm gọi API đăng nhập.

```swift
import Foundation

struct LoginRequest: Codable {
    let username: String
    let password: String
}

struct LoginResponse: Codable {
    let token: String
}

final class AuthService {
    static let shared = AuthService()
    private init() {}

    func login(username: String, password: String) async throws -> String {
        guard let url = URL(string: "https://fakestoreapi.com/auth/login") else {
            throw URLError(.badURL)
        }

        let requestBody = LoginRequest(username: username, password: password)
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = try JSONEncoder().encode(requestBody)

        let (data, response) = try await URLSession.shared.data(for: request)
        guard let httpResponse = response as? HTTPURLResponse,
              httpResponse.statusCode == 200 else {
            throw URLError(.badServerResponse)
        }

        let result = try JSONDecoder().decode(LoginResponse.self, from: data)
        return result.token
    }
}
```

💡 Giải thích:

* Dùng `async/await` để code ngắn gọn, rõ ràng.
* Nếu đăng nhập thành công, trả về token JWT.
* Token này sẽ được lưu lại để xác định người dùng đã login.

---

## 🧱 3. Tạo `UserSessionManager.swift`

Đây là nơi quản lý đăng nhập/đăng xuất, lưu token vào `UserDefaults`.

```swift
import Foundation

final class UserSessionManager {
    static let shared = UserSessionManager()
    private let tokenKey = "user_token"
    private init() {}

    var isLoggedIn: Bool {
        return UserDefaults.standard.string(forKey: tokenKey) != nil
    }

    func saveToken(_ token: String) {
        UserDefaults.standard.set(token, forKey: tokenKey)
    }

    func getToken() -> String? {
        UserDefaults.standard.string(forKey: tokenKey)
    }

    func logout() {
        UserDefaults.standard.removeObject(forKey: tokenKey)
    }
}
```

💡 Dùng singleton để có thể gọi `UserSessionManager.shared` ở mọi nơi.

---

## 🪄 4. Tạo giao diện `LoginViewController.swift`

Form đăng nhập gồm:

* TextField: username
* TextField: password
* Button: Login
* Label: thông báo lỗi

```swift
import UIKit

final class LoginViewController: UIViewController {

    private let usernameField = UITextField()
    private let passwordField = UITextField()
    private let loginButton = UIButton(type: .system)
    private let messageLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Đăng nhập"
        view.backgroundColor = .systemBackground
        setupUI()
    }

    private func setupUI() {
        usernameField.placeholder = "Tên đăng nhập"
        usernameField.borderStyle = .roundedRect
        usernameField.autocapitalizationType = .none

        passwordField.placeholder = "Mật khẩu"
        passwordField.borderStyle = .roundedRect
        passwordField.isSecureTextEntry = true

        loginButton.setTitle("Đăng nhập", for: .normal)
        loginButton.addTarget(self, action: #selector(loginTapped), for: .touchUpInside)
        loginButton.backgroundColor = .systemBlue
        loginButton.tintColor = .white
        loginButton.layer.cornerRadius = 8

        messageLabel.textColor = .systemRed
        messageLabel.numberOfLines = 0
        messageLabel.textAlignment = .center

        let stack = UIStackView(arrangedSubviews: [
            usernameField, passwordField, loginButton, messageLabel
        ])
        stack.axis = .vertical
        stack.spacing = 16
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)

        NSLayoutConstraint.activate([
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 24),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -24),
            stack.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            loginButton.heightAnchor.constraint(equalToConstant: 44)
        ])
    }

    @objc private func loginTapped() {
        Task {
            await loginAction()
        }
    }

    private func validate() -> (String, String)? {
        guard let username = usernameField.text, !username.isEmpty else {
            messageLabel.text = "Vui lòng nhập tên đăng nhập."
            return nil
        }
        guard let password = passwordField.text, !password.isEmpty else {
            messageLabel.text = "Vui lòng nhập mật khẩu."
            return nil
        }
        return (username, password)
    }

    private func showLoading(_ show: Bool) {
        if show {
            LoadingIndicator.shared.show(on: view)
        } else {
            LoadingIndicator.shared.hide()
        }
    }

    private func showError(_ msg: String) {
        messageLabel.text = msg
        let generator = UINotificationFeedbackGenerator()
        generator.notificationOccurred(.error)
    }

    private func navigateToMain() {
        let mainVC = ProductListViewController(viewModel: ProductListViewModel(networkService: NetworkService()))
        navigationController?.setViewControllers([mainVC], animated: true)
    }

    private func loginAction() async {
        guard let (username, password) = validate() else { return }
        showLoading(true)

        do {
            let token = try await AuthService.shared.login(username: username, password: password)
            UserSessionManager.shared.saveToken(token)
            await MainActor.run {
                self.showLoading(false)
                self.navigateToMain()
            }
        } catch {
            await MainActor.run {
                self.showLoading(false)
                self.showError("Sai tài khoản hoặc mật khẩu.")
            }
        }
    }
}
```

💡 Giải thích:

* `validate()` kiểm tra người dùng đã nhập đủ.
* `AuthService` gọi API thật.
* Khi thành công → lưu token và điều hướng sang màn hình sản phẩm chính.

---

## 🧪 5. Kết nối vào luồng khởi động (AppDelegate / SceneDelegate)

Trong `SceneDelegate.swift`:

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }
    window = UIWindow(windowScene: windowScene)

    let rootVC: UIViewController
    if UserSessionManager.shared.isLoggedIn {
        rootVC = ProductListViewController(viewModel: ProductListViewModel(networkService: NetworkService()))
    } else {
        rootVC = LoginViewController()
    }

    let nav = UINavigationController(rootViewController: rootVC)
    window?.rootViewController = nav
    window?.makeKeyAndVisible()
}
```

✅ Khi app mở:

* Nếu chưa đăng nhập → hiện màn hình Login.
* Nếu đã có token → vào thẳng màn hình chính.

---

## 🏠 **Bài tập về nhà (Tuần 4, Ngày 1)** 🎒

| Mức độ        | Bài tập                                                         | Gợi ý                             |
| ------------- | --------------------------------------------------------------- | --------------------------------- |
| 🟢 Cơ bản     | Hiển thị lỗi nếu login sai                                      | Dùng `messageLabel`               |
| 🟡 Trung bình | Thêm nút “Đăng xuất” trong ProductList                          | Gọi `UserSessionManager.logout()` |
| 🔵 Nâng cao   | Dùng Combine hoặc async stream để theo dõi trạng thái đăng nhập |                                   |
| 🟣 Thử thách  | Làm chức năng “Ghi nhớ đăng nhập” (Remember Me)                 | Dùng `UserDefaults` lưu thêm cờ   |

---

## 📚 Tổng kết buổi học

| Kiến thức                    | Vai trò                           |
| ---------------------------- | --------------------------------- |
| **Form Login bằng UIKit**    | Giao diện người dùng cơ bản       |
| **URLSession + async/await** | Gọi API thật                      |
| **UserDefaults**             | Lưu token đăng nhập               |
| **Navigation Flow**          | Chuyển màn hình sau khi đăng nhập |
| **MVVM + Auth Layer**        | Phân tách logic chuyên nghiệp     |

---

🎓 **Ngày 2 (buổi tới – Tuần 4):**

> *Thầy sẽ dạy “Màn hình chính – Product List + Logout + Hiển thị thông tin User”*
> Khi đăng nhập xong, ta sẽ hiển thị sản phẩm và thông tin người dùng thật từ API `/users/{id}`.

---

👉 Em có muốn thầy **tiếp tục luôn Tuần 4 – Ngày 2: Product List + Logout + User Info** không?
Bài này là bước nối tiếp để app trở thành **ứng dụng hoàn chỉnh**: login → hiển thị sản phẩm → đăng xuất.
