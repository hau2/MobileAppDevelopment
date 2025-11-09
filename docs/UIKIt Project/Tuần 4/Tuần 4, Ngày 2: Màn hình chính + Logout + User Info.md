Rất tốt 👏👏👏 — hôm nay thầy và em sang **Tuần 4 – Ngày 2: Màn hình chính (Product List) + Đăng xuất + Thông tin người dùng** 🛒

Đây là buổi **chuyển tiếp sau đăng nhập**, ta sẽ làm cho app hoạt động **thực tế như một ứng dụng thương mại điện tử mini**:

> Khi người dùng đăng nhập → app hiển thị danh sách sản phẩm từ API thật, và có thể đăng xuất trở lại màn hình Login.

Thầy sẽ **giải thích chậm, từng bước, dễ hiểu**, để em nắm chắc cả phần giao diện lẫn logic luồng người dùng (user flow).

---

# 🛍 UIKit – Tuần 4, Ngày 2: Màn hình chính + Logout + User Info

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Hiểu rõ **luồng sau khi đăng nhập**: hiển thị danh sách sản phẩm & thông tin user.
2. Hiểu cách tạo nút **Đăng xuất (Logout)** và điều hướng ngược lại Login.
3. Dùng API `GET /users/{id}` để lấy thông tin người dùng thật.
4. Quản lý navigation giữa các màn hình an toàn, không bị lỗi.

---

## 🧠 1. Luồng hoạt động sau đăng nhập

```
LoginViewController  →  ProductListViewController
                              ↑
                             Logout
```

* Khi người dùng đăng nhập thành công:

  * Lưu token vào `UserDefaults`.
  * Chuyển sang **màn hình danh sách sản phẩm**.
* Khi bấm **Đăng xuất**:

  * Xóa token.
  * Quay lại màn hình đăng nhập.

---

## ⚙️ 2. Cập nhật `ProductListViewController`

Ta đã có file này từ trước, giờ ta sẽ thêm:

* Nút “Đăng xuất” ở góc phải trên thanh điều hướng.
* Khi nhấn → gọi `UserSessionManager.logout()` và quay về Login.
* Khi vừa vào màn hình → gọi API lấy danh sách sản phẩm (nếu chưa có).

---

### ✳️ Bước 1 – thêm nút đăng xuất

Trong `viewDidLoad()` của `ProductListViewController`, thêm:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    title = "Sản phẩm"
    view.backgroundColor = .systemBackground
    setupTableView()
    setupLogoutButton()
    bindViewModel()
    fetchData()
}
```

Rồi tạo hàm `setupLogoutButton()`:

```swift
private func setupLogoutButton() {
    navigationItem.rightBarButtonItem = UIBarButtonItem(
        title: "Đăng xuất",
        style: .plain,
        target: self,
        action: #selector(logoutTapped)
    )
}

@objc private func logoutTapped() {
    let alert = UIAlertController(
        title: "Xác nhận",
        message: "Bạn có chắc muốn đăng xuất?",
        preferredStyle: .alert
    )
    alert.addAction(UIAlertAction(title: "Hủy", style: .cancel))
    alert.addAction(UIAlertAction(title: "Đăng xuất", style: .destructive) { _ in
        UserSessionManager.shared.logout()
        self.navigateToLogin()
    })
    present(alert, animated: true)
}

private func navigateToLogin() {
    let loginVC = LoginViewController()
    let nav = UINavigationController(rootViewController: loginVC)
    (UIApplication.shared.connectedScenes.first as? UIWindowScene)?
        .windows.first?.rootViewController = nav
}
```

💡 Giải thích:

* Khi bấm “Đăng xuất” → hiển thị cảnh báo xác nhận.
* Nếu đồng ý → xóa token, rồi đặt lại rootViewController về `LoginViewController`.
* Dùng cách đổi root thay vì `pop`, để đảm bảo không bị quay lại bằng gesture.

---

### ✳️ Bước 2 – gọi API danh sách sản phẩm

Phần này ta đã có từ các tuần trước, giờ chỉ cần gọi `fetchData()` sau khi load màn hình.

```swift
private func fetchData() {
    Task {
        LoadingIndicator.shared.show(on: view)
        do {
            let products = try await viewModel.fetchDataAsync()
            DispatchQueue.main.async {
                self.viewModel.products.value = products
                LoadingIndicator.shared.hide()
            }
        } catch {
            DispatchQueue.main.async {
                LoadingIndicator.shared.hide()
                print("❌ Lỗi tải sản phẩm:", error)
            }
        }
    }
}
```

---

## 🧩 3. Lấy thông tin người dùng thật từ API

FakeStoreAPI có endpoint:

```
GET https://fakestoreapi.com/users/1
```

Trả về:

```json
{
  "id": 1,
  "email": "john@gmail.com",
  "username": "johnd",
  "name": { "firstname": "John", "lastname": "Doe" },
  "address": {
    "city": "kilcoole",
    "street": "Lovers Ln",
    "zipcode": "12926-3874"
  },
  "phone": "1-570-236-7033"
}
```

---

### ✳️ Bước 1 – tạo `UserService.swift`

```swift
import Foundation

struct User: Codable {
    let id: Int
    let email: String
    let username: String
    let name: Name
    let address: Address
    let phone: String
}

struct Name: Codable {
    let firstname: String
    let lastname: String
}

struct Address: Codable {
    let city: String
    let street: String
    let zipcode: String
}

final class UserService {
    static let shared = UserService()
    private init() {}

    func fetchUser(id: Int) async throws -> User {
        guard let url = URL(string: "https://fakestoreapi.com/users/\(id)") else {
            throw URLError(.badURL)
        }
        let (data, response) = try await URLSession.shared.data(from: url)
        guard let httpResponse = response as? HTTPURLResponse,
              httpResponse.statusCode == 200 else {
            throw URLError(.badServerResponse)
        }
        return try JSONDecoder().decode(User.self, from: data)
    }
}
```

---

### ✳️ Bước 2 – hiển thị thông tin user trên màn hình

Thêm 1 nút “👤 Thông tin” cạnh nút “Đăng xuất”:

```swift
private func setupLogoutButton() {
    let logoutButton = UIBarButtonItem(
        title: "Đăng xuất",
        style: .plain,
        target: self,
        action: #selector(logoutTapped)
    )

    let profileButton = UIBarButtonItem(
        title: "👤",
        style: .plain,
        target: self,
        action: #selector(showUserInfo)
    )

    navigationItem.rightBarButtonItems = [logoutButton, profileButton]
}
```

Rồi tạo hàm `showUserInfo()`:

```swift
@objc private func showUserInfo() {
    Task {
        LoadingIndicator.shared.show(on: view)
        do {
            let user = try await UserService.shared.fetchUser(id: 1)
            await MainActor.run {
                LoadingIndicator.shared.hide()
                self.presentUserInfo(user)
            }
        } catch {
            await MainActor.run {
                LoadingIndicator.shared.hide()
                print("❌ Lỗi tải user:", error)
            }
        }
    }
}

private func presentUserInfo(_ user: User) {
    let msg = """
    👤 \(user.name.firstname.capitalized) \(user.name.lastname.capitalized)
    📧 \(user.email)
    ☎️ \(user.phone)
    🏠 \(user.address.street), \(user.address.city)
    """
    let alert = UIAlertController(title: "Thông tin người dùng", message: msg, preferredStyle: .alert)
    alert.addAction(UIAlertAction(title: "Đóng", style: .default))
    present(alert, animated: true)
}
```

💡 Khi nhấn vào nút “👤” → app gọi API thật → hiển thị thông tin user bằng alert.

---

## 🧪 4. Kiểm tra thực tế 🚀

### 🔹 Bước 1:

Đăng nhập bằng:

```
username: mor_2314
password: 83r5^_
```

→ Thành công → chuyển sang màn hình sản phẩm.

### 🔹 Bước 2:

* Bấm “👤” → hiện popup “Thông tin người dùng”.
* Bấm “Đăng xuất” → quay lại màn hình đăng nhập.

✅ App của em giờ đã có **luồng người dùng hoàn chỉnh**:
Login → Danh sách → Logout → quay lại Login.

---

## 🏠 **Bài tập về nhà (Tuần 4, Ngày 2)** 🎒

| Mức độ        | Bài tập                                                   | Gợi ý                                                      |
| ------------- | --------------------------------------------------------- | ---------------------------------------------------------- |
| 🟢 Cơ bản     | Hiển thị thêm ảnh đại diện user (fake image)              | Dùng `randomuser.me/api/portraits/men/1.jpg`               |
| 🟡 Trung bình | Cho phép chọn user ID khi đăng nhập (để test nhiều người) | Dùng `UserDefaults` lưu userID                             |
| 🔵 Nâng cao   | Thêm màn hình “Hồ sơ cá nhân” riêng thay vì alert         | Tạo `UserProfileViewController`                            |
| 🟣 Thử thách  | Khi đăng xuất → xoá cả cache và CoreData                  | Gọi `CacheManager.clearCache()` và `CoreDataManager` reset |

---

## 📚 Tổng kết buổi học

| Kiến thức                     | Vai trò                        |
| ----------------------------- | ------------------------------ |
| **Navigation Flow**           | Luồng giữa Login ↔ ProductList |
| **User Session**              | Lưu / Xóa token đăng nhập      |
| **Async/Await API**           | Gọi API user thật              |
| **UIAlertController**         | Hiển thị thông tin gọn gàng    |
| **Root ViewController Reset** | Điều hướng an toàn khi logout  |

---

🎓 **Ngày 3 (buổi tới – Tuần 4):**

> *Thầy sẽ hướng dẫn “Chi tiết sản phẩm (Product Detail View) + Thêm vào giỏ hàng (Cart)” – tiếp tục xây dựng luồng mua hàng thực tế trong app.*

---

👉 Em có muốn thầy **tiếp tục luôn Tuần 4 – Ngày 3: Product Detail + Cart Flow** không?
Bài này rất vui và quan trọng — ta sẽ học cách **truyền dữ liệu giữa màn hình**, **hiển thị chi tiết sản phẩm**, và **thêm vào giỏ hàng** như app thật.
