Tuyệt vời 👏👏👏 — hôm nay chúng ta sang **Tuần 5 – Ngày 1: Tối ưu tổng thể & Chuẩn bị phát hành (App Optimization & TestFlight Prep)** 🚀📱

Đây là **chặng cuối của khóa UIKit thực chiến**, giúp em **biến app học tập thành một sản phẩm thật có thể gửi cho người khác dùng thử** (qua TestFlight hoặc chia sẻ file `.ipa`).

Thầy sẽ hướng dẫn **rất chậm – dễ hiểu – từng bước**:

> Tối ưu hiệu năng → Tạo Splash Screen → Icon App → Dark Mode → Chuẩn bị build lên TestFlight.

---

# 🚀 UIKit – Tuần 5, Ngày 1: App Optimization & TestFlight Prep

---

## 🎯 Mục tiêu bài học

Sau buổi hôm nay, em sẽ:

1. Hiểu cách **tối ưu hiệu năng và bộ nhớ**.
2. Tạo **màn hình Splash (khởi động app)**.
3. Thêm **App Icon & Launch Screen**.
4. Kích hoạt **Dark Mode** và kiểm tra tương thích.
5. Chuẩn bị **bản build TestFlight** để chia sẻ app demo.

---

## 🧠 1. Tối ưu hiệu năng tổng thể

Hiện app của em có nhiều màn hình (Login, Product List, Detail, Cart, Order History…).
Để app chạy mượt hơn, ta làm 4 điều sau:

---

### 🔹 1.1. Sử dụng tải ảnh bất đồng bộ hiệu quả

Trong mọi `UIImageView` tải ảnh từ URL, thay vì:

```swift
if let data = try? Data(contentsOf: url) { ... }
```

→ dùng `URLSession` để không chặn main thread:

```swift
func loadImage(from urlString: String) {
    guard let url = URL(string: urlString) else { return }
    URLSession.shared.dataTask(with: url) { data, _, _ in
        if let data = data, let image = UIImage(data: data) {
            DispatchQueue.main.async {
                self.imageView.image = image
            }
        }
    }.resume()
}
```

✅ Không bị “lag” khi scroll hoặc load ảnh nhiều.

---

### 🔹 1.2. Tái sử dụng cell đúng cách

Trong tất cả TableView:

```swift
tableView.dequeueReusableCell(withIdentifier:for:)
```

phải luôn **đăng ký reuseIdentifier** một lần duy nhất, và tránh tạo UI trong `cellForRowAt`.
💡 Gợi ý thêm:

```swift
cell.prepareForReuse() {
   imageView.image = nil
}
```

---

### 🔹 1.3. Dọn bộ nhớ (Memory Cleanup)

Sau khi user “đăng xuất” hoặc “thanh toán xong”, ta nên reset cache:

```swift
CartManager.shared.clear()
URLCache.shared.removeAllCachedResponses()
```

💡 Nếu có dùng `CoreData`, ta có thể gọi `reset()` để giải phóng context.

---

### 🔹 1.4. Kiểm tra rò rỉ bộ nhớ (Memory Leak)

Trong Xcode:
1️⃣ Mở **Product → Profile → Instruments → Leaks**
2️⃣ Chạy app, mở/đóng nhiều màn hình.
3️⃣ Nếu có leak, Xcode báo đỏ → tìm closure chưa `[weak self]`.

Ví dụ sửa:

```swift
viewModel.products.bind { [weak self] _ in
    self?.reloadData()
}
```

---

## 🖼 2. Tạo Splash Screen (Launch Screen)

Splash screen là **màn hình hiển thị logo khi mở app**, rất dễ làm bằng file `.storyboard` mặc định của Xcode.

---

### 🔹 2.1. Mở file `LaunchScreen.storyboard`

1️⃣ Kéo **UIImageView** vào giữa.
2️⃣ Gán ảnh logo (ví dụ `app_logo.png`).
3️⃣ Chọn “Aspect Fit” để logo không bị méo.
4️⃣ Đặt màu nền nhẹ (hoặc trắng).

---

### 🔹 2.2. Đặt Launch Screen trong Xcode

* Chọn **project file** → tab **General**
* Trong mục **App Icons and Launch Images**
* Gán `LaunchScreen.storyboard` vào **Launch Screen File**.

✅ Khi mở app, màn hình logo hiển thị 1–2 giây trước khi SceneDelegate load.

---

## 🧩 3. Thêm App Icon (biểu tượng ứng dụng)

Chuẩn icon của iOS gồm 20+ kích thước khác nhau (29×29, 60×60, 1024×1024...).
Cách dễ nhất:

### ✅ Dùng trang [https://appicon.co](https://appicon.co)

1️⃣ Tải ảnh logo gốc (1024×1024).
2️⃣ Chọn **iOS** → Download.
3️⃣ Giải nén, kéo thư mục `AppIcon.appiconset` vào `Assets.xcassets` (ghi đè nếu có).

💡 Khi chạy lại app, icon mới sẽ hiện ở màn hình chính.

---

## 🌗 4. Kích hoạt Dark Mode (chế độ nền tối)

UIKit tự hỗ trợ Dark Mode nếu dùng màu hệ thống (`.systemBackground`, `.label`, `.secondaryLabel`...)

---

### 🔹 4.1. Kiểm tra tương thích nhanh:

Trong Simulator → **Environment Overrides (🌙)**
Chuyển giữa Light / Dark để xem giao diện có đổi không.

### 🔹 4.2. Nếu muốn tắt Dark Mode tạm:

Thêm vào `Info.plist`:

```xml
<key>UIUserInterfaceStyle</key>
<string>Light</string>
```

💡 Nên giữ Dark Mode vì iPhone hiện nay mặc định bật tính năng này.

---

## 🧱 5. Tạo App Version & Build chuẩn

Trong Xcode:
1️⃣ Mở **Project → General → Identity**
2️⃣ Đặt:

* Display Name: `MiniShop`
* Version: `1.0.0`
* Build: `1`

💡 Mỗi khi update → tăng Build lên `2`, `3`, …

---

## 📦 6. Chuẩn bị build để chia sẻ (TestFlight)

### 🔹 6.1. Cần có tài khoản **Apple Developer (99$/năm)**

Sau khi đăng ký, mở **Xcode → Preferences → Accounts → Add Apple ID**.

---

### 🔹 6.2. Đặt cấu hình Bundle ID

Ví dụ:

```
com.maile.minishop
```

---

### 🔹 6.3. Chạy lệnh build

Menu **Product → Archive**
→ Khi build xong, Xcode mở **Organizer** → chọn “Distribute App”
→ Chọn “App Store Connect → Upload”.

💡 Apple duyệt bản TestFlight tự động trong vài phút.

---

### 🔹 6.4. Mời người dùng thử

* Mở **App Store Connect → My Apps → TestFlight**
* Thêm email người thử nghiệm.
* Họ sẽ nhận link TestFlight để cài app trực tiếp từ Apple Store.

---

## 🧠 7. Gợi ý tối ưu UX/UI trước khi build

| Vấn đề                               | Giải pháp                                            |
| ------------------------------------ | ---------------------------------------------------- |
| Loading quá nhiều                    | Dùng `UIActivityIndicatorView` hoặc skeleton loading |
| Danh sách dài                        | Thêm pull-to-refresh                                 |
| Text quá nhỏ                         | Dùng Dynamic Type (`UIFont.preferredFont`)           |
| Navigation rối                       | Dùng `Coordinator` pattern (nếu app lớn)             |
| Hành động nguy hiểm (xoá, đăng xuất) | Hiển thị `UIAlertController` xác nhận                |

---

## 🧪 8. Kiểm tra toàn app trước khi gửi 🚀

1️⃣ Đăng nhập → Danh sách sản phẩm → Chi tiết → Thêm giỏ hàng → Thanh toán → Lịch sử đơn hàng
2️⃣ Tắt mạng → Kiểm tra hiển thị lỗi API.
3️⃣ Xoay màn hình ngang → Xem UI có bị vỡ layout không.
4️⃣ Kiểm tra Dark Mode.
5️⃣ Mở **Product → Scheme → Edit Scheme → Run → Diagnostics → Slow Animations** để xem hiệu ứng.

---

## 🏠 **Bài tập về nhà (Tuần 5, Ngày 1)** 🎒

| Mức độ        | Bài tập                                    | Gợi ý                                 |
| ------------- | ------------------------------------------ | ------------------------------------- |
| 🟢 Cơ bản     | Thêm logo splash & icon cho app            | Dùng [appicon.co](https://appicon.co) |
| 🟡 Trung bình | Bật Dark Mode và chỉnh lại màu chữ nếu cần | Dùng `.label`, `.systemBackground`    |
| 🔵 Nâng cao   | Dùng Instruments đo thời gian tải sản phẩm | “Product → Profile → Time Profiler”   |
| 🟣 Thử thách  | Build thử lên TestFlight                   | Tạo tài khoản developer Apple         |

---

## 📚 Tổng kết buổi học

| Nội dung                 | Vai trò                                   |
| ------------------------ | ----------------------------------------- |
| **Async Image Loading**  | Tối ưu tốc độ hiển thị ảnh                |
| **Memory Cleanup**       | Giải phóng tài nguyên sau khi dùng        |
| **Splash Screen**        | Tăng tính chuyên nghiệp                   |
| **App Icon & Dark Mode** | Thể hiện bản sắc và tương thích giao diện |
| **Build & TestFlight**   | Phát hành cho người dùng thật             |

---

🎓 **Ngày 2 (buổi tới – Tuần 5):**

> *Thầy sẽ dạy “App Debug & Logging chuyên nghiệp” — học cách kiểm soát lỗi, theo dõi hành vi người dùng, và ghi log có hệ thống (dùng `OSLog`, `Crashlytics`, và file export).*

---

👉 Em có muốn thầy **tiếp tục luôn Tuần 5 – Ngày 2: App Debug & Logging chuyên nghiệp** không?
Bài này cực kỳ thực tế — giúp em hiểu cách **app thực tế ngoài đời ghi log, báo lỗi, và tự phục hồi khi crash** như thế nào.
