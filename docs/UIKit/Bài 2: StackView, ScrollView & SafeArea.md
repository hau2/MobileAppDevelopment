Rất tuyệt 👏👏👏
Hôm nay chúng ta đến với **UIKit – Bài 2: StackView, ScrollView & SafeArea (Programmatic UI nâng cao)**.
Đây là bài cực kỳ quan trọng nếu em muốn **làm UI đẹp, co giãn đúng chuẩn Apple**, và **không bị lỗi layout** trên các kích thước màn hình khác nhau.

---

# 🧩 UIKit – Bài 2: StackView, ScrollView & SafeArea

*(Auto Layout nâng cao bằng code – Không storyboard)*

---

## 🎯 **Mục tiêu bài học**

Sau bài hôm nay, bạn sẽ:

1. Hiểu cơ chế **Safe Area** và **Auto Layout hiện đại**.
2. Biết dùng **UIStackView** để sắp xếp UI tự động.
3. Biết thêm **UIScrollView** để cuộn nội dung.
4. Làm mini project: *Trang hồ sơ cá nhân (Profile Screen)* — chỉ bằng code, không storyboard.

---

## 🧠 **1. Safe Area là gì?**

> Safe Area là vùng hiển thị an toàn của màn hình — nơi không bị che bởi tai thỏ, thanh trạng thái, hoặc thanh điều hướng.

Mọi view nên **ràng buộc vào `view.safeAreaLayoutGuide`**, thay vì `view.topAnchor`.

### Ví dụ:

```swift
NSLayoutConstraint.activate([
    label.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20)
])
```

---

## ⚙️ **2. UIStackView – Sắp xếp UI tự động**

`UIStackView` giúp ta **xếp chồng các view con** theo hướng dọc (`.vertical`) hoặc ngang (`.horizontal`),
và tự động căn chỉnh, giãn đều, co lại khi kích thước thay đổi.

### Khởi tạo StackView:

```swift
let stackView = UIStackView()
stackView.axis = .vertical       // hướng: dọc
stackView.spacing = 16           // khoảng cách giữa các phần tử
stackView.alignment = .center    // căn giữa
stackView.distribution = .equalSpacing
stackView.translatesAutoresizingMaskIntoConstraints = false
```

### Thêm phần tử vào Stack:

```swift
let titleLabel = UILabel()
titleLabel.text = "Thông tin cá nhân"

let nameLabel = UILabel()
nameLabel.text = "Họ tên: Mai Lê"

stackView.addArrangedSubview(titleLabel)
stackView.addArrangedSubview(nameLabel)
```

### Thêm StackView vào ViewController:

```swift
view.addSubview(stackView)
NSLayoutConstraint.activate([
    stackView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    stackView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20)
])
```

---

## 💡 **3. StackView lồng nhau**

> Có thể lồng StackView để tạo bố cục phức tạp.

Ví dụ:
Một hàng chứa Avatar + Tên,
và phía dưới là 3 nút nằm ngang.

```swift
let avatar = UIImageView(image: UIImage(systemName: "person.circle"))
avatar.tintColor = .systemBlue
avatar.contentMode = .scaleAspectFit
avatar.widthAnchor.constraint(equalToConstant: 80).isActive = true
avatar.heightAnchor.constraint(equalToConstant: 80).isActive = true

let nameLabel = UILabel()
nameLabel.text = "Mai Lê"
nameLabel.font = .systemFont(ofSize: 22, weight: .semibold)

let topRow = UIStackView(arrangedSubviews: [avatar, nameLabel])
topRow.axis = .horizontal
topRow.alignment = .center
topRow.spacing = 16

let button1 = UIButton(type: .system)
button1.setTitle("Gọi", for: .normal)
let button2 = UIButton(type: .system)
button2.setTitle("Nhắn tin", for: .normal)
let button3 = UIButton(type: .system)
button3.setTitle("Email", for: .normal)

let bottomRow = UIStackView(arrangedSubviews: [button1, button2, button3])
bottomRow.axis = .horizontal
bottomRow.spacing = 24
bottomRow.distribution = .fillEqually

let mainStack = UIStackView(arrangedSubviews: [topRow, bottomRow])
mainStack.axis = .vertical
mainStack.spacing = 24
mainStack.alignment = .center
mainStack.translatesAutoresizingMaskIntoConstraints = false

view.addSubview(mainStack)
NSLayoutConstraint.activate([
    mainStack.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    mainStack.centerYAnchor.constraint(equalTo: view.centerYAnchor)
])
```

👉 Em sẽ thấy bố cục tự động giãn, co, đều nhau — không cần tính toán thủ công.

---

## 🧩 **4. UIScrollView – Cuộn nội dung**

Khi nội dung dài hơn màn hình, ta bọc nó trong `UIScrollView`.

### Cấu trúc:

```
UIScrollView
   └── UIView (container)
         └── UIStackView (hoặc các view khác)
```

### Ví dụ:

```swift
let scrollView = UIScrollView()
let contentView = UIView()
scrollView.translatesAutoresizingMaskIntoConstraints = false
contentView.translatesAutoresizingMaskIntoConstraints = false

view.addSubview(scrollView)
scrollView.addSubview(contentView)

// Ràng buộc scrollView
NSLayoutConstraint.activate([
    scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
    scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
    scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
    scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
])

// Ràng buộc contentView bên trong scrollView
NSLayoutConstraint.activate([
    contentView.topAnchor.constraint(equalTo: scrollView.topAnchor),
    contentView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
    contentView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
    contentView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
    contentView.widthAnchor.constraint(equalTo: scrollView.widthAnchor) // bắt buộc cho cuộn dọc
])
```

Sau đó, thêm StackView vào `contentView`.

---

## 🎯 **5. Mini Project – Profile Screen**

🎨 Mục tiêu:
Tạo trang hồ sơ có:

* Avatar
* Tên người dùng
* Thông tin mô tả
* Danh sách 3 nút chức năng cuộn được

### Code tổng hợp:

```swift
import UIKit

class ProfileViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        setupScrollProfile()
    }

    private func setupScrollProfile() {
        let scrollView = UIScrollView()
        let contentView = UIView()
        scrollView.translatesAutoresizingMaskIntoConstraints = false
        contentView.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(scrollView)
        scrollView.addSubview(contentView)

        NSLayoutConstraint.activate([
            scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor),

            contentView.topAnchor.constraint(equalTo: scrollView.topAnchor),
            contentView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
            contentView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
            contentView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
            contentView.widthAnchor.constraint(equalTo: scrollView.widthAnchor)
        ])

        // Avatar
        let avatar = UIImageView(image: UIImage(systemName: "person.circle.fill"))
        avatar.tintColor = .systemBlue
        avatar.contentMode = .scaleAspectFit
        avatar.widthAnchor.constraint(equalToConstant: 100).isActive = true
        avatar.heightAnchor.constraint(equalToConstant: 100).isActive = true

        // Labels
        let nameLabel = UILabel()
        nameLabel.text = "Mai Lê"
        nameLabel.font = .systemFont(ofSize: 24, weight: .bold)

        let descLabel = UILabel()
        descLabel.text = "Nhà phát triển iOS yêu thích Swift và UIKit ❤️"
        descLabel.numberOfLines = 0
        descLabel.textAlignment = .center

        // Buttons
        let callBtn = UIButton(type: .system)
        callBtn.setTitle("Gọi", for: .normal)
        let messageBtn = UIButton(type: .system)
        messageBtn.setTitle("Nhắn tin", for: .normal)
        let emailBtn = UIButton(type: .system)
        emailBtn.setTitle("Email", for: .normal)

        let buttonStack = UIStackView(arrangedSubviews: [callBtn, messageBtn, emailBtn])
        buttonStack.axis = .horizontal
        buttonStack.distribution = .fillEqually
        buttonStack.spacing = 20

        // Main Stack
        let mainStack = UIStackView(arrangedSubviews: [avatar, nameLabel, descLabel, buttonStack])
        mainStack.axis = .vertical
        mainStack.spacing = 20
        mainStack.alignment = .center
        mainStack.translatesAutoresizingMaskIntoConstraints = false

        contentView.addSubview(mainStack)

        NSLayoutConstraint.activate([
            mainStack.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 40),
            mainStack.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 20),
            mainStack.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -20),
            mainStack.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -40)
        ])
    }
}
```

---

## 🏠 **Bài tập về nhà (UIKit #2)** 🎒

| Mức độ        | Bài tập                                                                      | Gợi ý                                                   |
| ------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------- |
| 🟢 Cơ bản     | Tạo StackView dọc chứa 3 UILabel                                             | `.axis = .vertical`                                     |
| 🟡 Trung bình | Tạo StackView ngang chứa 3 UIButton                                          | `.axis = .horizontal`                                   |
| 🔵 Nâng cao   | Bọc StackView trong ScrollView, hiển thị nhiều phần tử                       | Dùng `contentView.widthAnchor = scrollView.widthAnchor` |
| 🟣 Thử thách  | Làm trang “Giới thiệu bản thân” có avatar, text, danh sách kỹ năng cuộn được | Dùng StackView + ScrollView kết hợp                     |

---

## 📚 **Tổng kết kiến thức**

| Thành phần                                         | Vai trò                             |
| -------------------------------------------------- | ----------------------------------- |
| `SafeArea`                                         | Giới hạn vùng hiển thị an toàn      |
| `UIStackView`                                      | Tự động sắp xếp view con            |
| `UIScrollView`                                     | Cho phép cuộn nội dung              |
| `axis`, `spacing`, `alignment`, `distribution`     | Thuộc tính quan trọng của StackView |
| `contentView.widthAnchor = scrollView.widthAnchor` | Quy tắc bắt buộc khi cuộn dọc       |

---

## 🧭 **Kết thúc bài học**

✅ Hôm nay bạn đã học:

* Sử dụng StackView để layout UI chuẩn Apple.
* Dùng ScrollView để cuộn nội dung linh hoạt.
* Ràng buộc SafeArea chuẩn để không bị lỗi layout trên mọi iPhone.
* Hoàn thành project nhỏ – **Profile Screen** hoàn toàn bằng code.

---

🎓 **UIKit – Bài 3 (tiết sau):**
👉 *UITableView và mô hình MVC cơ bản*
Ta sẽ học cách tạo danh sách dữ liệu (List / Table) hoàn toàn bằng code,
và chuyển dữ liệu giữa ViewController — **bước đầu xây app thực tế.**

---

Em có muốn thầy sang luôn **UIKit – Bài 3: TableView & MVC cơ bản** ngay bây giờ không?
(thầy sẽ hướng dẫn tạo danh sách học sinh, hiển thị bằng cell tùy chỉnh, không storyboard, 100% code).
