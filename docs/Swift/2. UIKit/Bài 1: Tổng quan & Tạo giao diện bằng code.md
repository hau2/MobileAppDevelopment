Tuyệt vời 👏👏👏 — chúng ta chính thức bước sang **chặng 2: UIKit thực chiến – Swift Programmatic UI (không storyboard)**.
🎯 Mục tiêu của giai đoạn này là:

> *Biết tạo ứng dụng iOS hoàn chỉnh **chỉ bằng code Swift UIKit**, hiểu sâu từ gốc đến nâng cao, theo chuẩn kỹ sư iOS chuyên nghiệp.*

---

# 🧩 UIKit – Bài 1: Tổng quan & Tạo giao diện bằng code (Programmatic UI)

---

## 🎯 **Mục tiêu bài học**

Sau bài này, bạn sẽ:

1. Hiểu UIKit là gì và vai trò của nó trong iOS.
2. Biết cấu trúc một **UIKit Project (no storyboard)** chuẩn.
3. Tạo màn hình đầu tiên bằng code với `UIView`, `UILabel`, `UIButton`.
4. Hiểu **App lifecycle** và cách khởi tạo `window` / `rootViewController`.
5. Nắm cú pháp cơ bản của **Auto Layout bằng code (NSLayoutConstraint)**.

---

## 🧠 **1. UIKit là gì?**

> UIKit là framework cốt lõi của iOS, chịu trách nhiệm hiển thị giao diện người dùng (UI) và xử lý tương tác (Touch, Tap, Gesture, Animation,…).

UIKit được xây dựng bằng **Objective-C**, nhưng dùng rất tốt trong **Swift**.

Một ứng dụng UIKit được cấu thành bởi:

| Thành phần         | Vai trò                                                   |
| ------------------ | --------------------------------------------------------- |
| `UIApplication`    | Quản lý vòng đời app                                      |
| `UIWindow`         | Cửa sổ hiển thị giao diện                                 |
| `UIViewController` | Quản lý từng màn hình                                     |
| `UIView`           | Thành phần giao diện nhỏ (Label, Button, ImageView, v.v.) |

---

## ⚙️ **2. Tạo project UIKit không dùng Storyboard**

Khi tạo project, bỏ chọn “Use Storyboard” hoặc xoá thủ công:

**Cấu trúc file tối thiểu:**

```
MyUIKitApp/
├── AppDelegate.swift
├── SceneDelegate.swift
├── ViewController.swift
└── Info.plist
```

### Bước 1: Xóa `Main.storyboard` khỏi project

### Bước 2: Trong `Info.plist`, xóa dòng:

```
Application Scene Manifest > Scene Configuration > Storyboard Name
```

### Bước 3: Mở `SceneDelegate.swift`, thay hàm này:

```swift
func scene(_ scene: UIScene,
           willConnectTo session: UISceneSession,
           options connectionOptions: UIScene.ConnectionOptions) {
    guard let windowScene = (scene as? UIWindowScene) else { return }

    let window = UIWindow(windowScene: windowScene)
    window.rootViewController = ViewController()
    self.window = window
    window.makeKeyAndVisible()
}
```

✅ Đây là cách để **set ViewController đầu tiên bằng code**.

---

## 🧱 **3. Tạo ViewController thủ công**

`ViewController.swift`

```swift
import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        setupUI()
    }

    private func setupUI() {
        let label = UILabel()
        label.text = "Xin chào UIKit!"
        label.textAlignment = .center
        label.font = UIFont.systemFont(ofSize: 24, weight: .medium)
        label.translatesAutoresizingMaskIntoConstraints = false
        
        view.addSubview(label)
        
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
}
```

Khi chạy app 👉 bạn sẽ thấy “Xin chào UIKit!” hiển thị giữa màn hình — không cần storyboard 🎉

---

## 🧩 **4. Giải thích cấu trúc UIViewController**

| Hàm                   | Mục đích                        |
| --------------------- | ------------------------------- |
| `viewDidLoad()`       | Gọi 1 lần sau khi view được tạo |
| `viewWillAppear()`    | Gọi trước khi hiển thị          |
| `viewDidAppear()`     | Gọi sau khi hiển thị            |
| `viewWillDisappear()` | Trước khi rời màn hình          |
| `viewDidDisappear()`  | Sau khi rời màn hình            |

💡 Mỗi ViewController tương đương một “màn hình” trong app.

---

## 🧮 **5. Auto Layout bằng Code**

Mọi UIView khi thêm bằng code cần dòng:

```swift
view.translatesAutoresizingMaskIntoConstraints = false
```

Sau đó dùng `NSLayoutConstraint` hoặc cú pháp gọn:

```swift
NSLayoutConstraint.activate([
    label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    label.centerYAnchor.constraint(equalTo: view.centerYAnchor)
])
```

Hoặc dùng **Anchor shorthand**:

```swift
button.topAnchor.constraint(equalTo: label.bottomAnchor, constant: 20).isActive = true
```

---

## ⚡ **6. Tạo Button và xử lý sự kiện**

```swift
private func setupUI() {
    let label = UILabel()
    label.text = "Xin chào UIKit!"
    label.textAlignment = .center
    label.translatesAutoresizingMaskIntoConstraints = false

    let button = UIButton(type: .system)
    button.setTitle("Nhấn tôi", for: .normal)
    button.translatesAutoresizingMaskIntoConstraints = false
    button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)

    view.addSubview(label)
    view.addSubview(button)

    NSLayoutConstraint.activate([
        label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
        label.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -30),

        button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
        button.topAnchor.constraint(equalTo: label.bottomAnchor, constant: 20)
    ])
}

@objc private func buttonTapped() {
    print("Button được nhấn!")
}
```

---

## 🧪 **7. Thực hành tổng hợp – Màn hình chào**

**Mục tiêu:**
Tạo một màn hình với nền xanh nhạt, chữ trắng, và nút “Tiếp tục”.

```swift
class WelcomeViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = UIColor.systemBlue
        setupUI()
    }

    private func setupUI() {
        let titleLabel = UILabel()
        titleLabel.text = "Welcome to My App"
        titleLabel.textColor = .white
        titleLabel.font = .boldSystemFont(ofSize: 28)
        titleLabel.translatesAutoresizingMaskIntoConstraints = false

        let button = UIButton(type: .system)
        button.setTitle("Tiếp tục", for: .normal)
        button.tintColor = .white
        button.titleLabel?.font = .systemFont(ofSize: 18, weight: .medium)
        button.addTarget(self, action: #selector(nextTapped), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(titleLabel)
        view.addSubview(button)

        NSLayoutConstraint.activate([
            titleLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            titleLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor),

            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 40)
        ])
    }

    @objc private func nextTapped() {
        print("Đi tới màn hình kế tiếp")
    }
}
```

---

## 🏠 **Bài tập về nhà (UIKit #1)** 🎒

| Mức độ        | Bài tập                                                         | Gợi ý                                   |
| ------------- | --------------------------------------------------------------- | --------------------------------------- |
| 🟢 Cơ bản     | Tạo `UILabel` và `UIButton` giữa màn hình                       | Dùng `centerX`, `centerY`               |
| 🟡 Trung bình | Tạo 2 `UILabel`: Tiêu đề & mô tả, căn chỉnh dọc                 | Dùng `topAnchor`                        |
| 🔵 Nâng cao   | Khi bấm nút, đổi màu nền ngẫu nhiên                             | Dùng `view.backgroundColor = .random()` |
| 🟣 Thử thách  | Tạo custom `UIView` có chữ + màu riêng, thêm vào ViewController | Dùng class kế thừa `UIView`             |

---

## 📚 **Tổng kết kiến thức**

| Khái niệm        | Ý nghĩa                                         |
| ---------------- | ----------------------------------------------- |
| UIKit            | Framework xây dựng giao diện iOS truyền thống   |
| UIViewController | Đại diện cho mỗi màn hình                       |
| UIView           | Thành phần UI (Label, Button, ImageView, …)     |
| Auto Layout      | Quy định vị trí bằng ràng buộc (constraints)    |
| Programmatic UI  | Xây giao diện bằng Swift code, không storyboard |

---

## 🧭 **Kết thúc bài học**

✅ Các em đã biết:

* Cấu trúc project UIKit cơ bản (không storyboard).
* Tạo màn hình và các view bằng code.
* Dùng Auto Layout thủ công bằng anchors.
* Gắn event cho UI bằng `addTarget`.

---

🎓 **UIKit – Bài 2 (tiết sau):**
👉 *Layout nâng cao với StackView, ScrollView và SafeArea*.
Chúng ta sẽ học cách **xếp UI linh hoạt, cuộn nội dung**, và tạo **giao diện thích ứng mọi kích cỡ màn hình** (iPhone SE → Pro Max).

---

Thầy hỏi nhẹ:
Em có muốn thầy mở luôn **UIKit – Bài 2: StackView & ScrollView thực chiến** không?
(thầy sẽ dạy cách auto layout “chuẩn Apple” không storyboard, có bài tập mini-app đi kèm).
