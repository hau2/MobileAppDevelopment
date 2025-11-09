Rất tốt 👏👏👏 — hôm nay chúng ta bước sang **UIKit – Bài 20: ScrollView & Paging nâng cao – Cuộn, Zoom và Hiển thị nhiều màn trong một View**.
Đây là **một trong những kỹ năng nền tảng quan trọng nhất** của iOS Developer, vì hầu hết các ứng dụng hiện đại đều có dạng **cuộn nội dung**, **zoom ảnh**, hoặc **trang onboarding lật ngang** như App Store, Instagram, hay eBook.

> Sau bài này, em sẽ hiểu sâu về `UIScrollView`, `UIStackView` kết hợp, **paging**, và **zoom** như Photos App.

---

# 🧩 UIKit – Bài 20: ScrollView & Paging (Cuộn + Zoom + Trang lật)

---

## 🎯 Mục tiêu bài học

Sau buổi này, em sẽ biết:

1. Cấu trúc và nguyên lý hoạt động của `UIScrollView`.
2. Cách cuộn nội dung dọc và ngang.
3. Cách tạo màn hình **Onboarding / PagingView** dạng lật trang.
4. Kết hợp ScrollView + StackView để làm layout động.
5. Zoom ảnh bằng gesture.

---

## 🧠 1. Hiểu cơ chế `UIScrollView`

`UIScrollView` hoạt động dựa trên ba thành phần:

| Thành phần         | Ý nghĩa                                        |
| ------------------ | ---------------------------------------------- |
| **Frame**          | Kích thước hiển thị (khung nhìn)               |
| **Content Size**   | Tổng kích thước nội dung bên trong             |
| **Content Offset** | Vị trí hiện tại của khung nhìn so với nội dung |

Khi `contentSize` lớn hơn `frame`, ScrollView sẽ cho phép **cuộn**.

---

## ⚙️ 2. Ví dụ cơ bản: Cuộn nội dung dọc

**ScrollDemoViewController.swift**

```swift
import UIKit

final class ScrollDemoViewController: UIViewController {
    private let scrollView = UIScrollView()
    private let contentView = UIView()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Scroll Demo"
        view.backgroundColor = .systemBackground
        setupScrollView()
        setupContent()
    }

    private func setupScrollView() {
        scrollView.translatesAutoresizingMaskIntoConstraints = false
        scrollView.backgroundColor = .systemGroupedBackground
        view.addSubview(scrollView)

        NSLayoutConstraint.activate([
            scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])

        contentView.translatesAutoresizingMaskIntoConstraints = false
        scrollView.addSubview(contentView)

        NSLayoutConstraint.activate([
            contentView.topAnchor.constraint(equalTo: scrollView.topAnchor),
            contentView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
            contentView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
            contentView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),

            // Rất quan trọng: phải có width = scrollView width
            contentView.widthAnchor.constraint(equalTo: scrollView.widthAnchor)
        ])
    }

    private func setupContent() {
        var lastLabel: UILabel?
        for i in 1...15 {
            let label = UILabel()
            label.text = "Dòng nội dung \(i)"
            label.textAlignment = .center
            label.font = .preferredFont(forTextStyle: .title3)
            label.translatesAutoresizingMaskIntoConstraints = false
            contentView.addSubview(label)

            NSLayoutConstraint.activate([
                label.leadingAnchor.constraint(equalTo: contentView.leadingAnchor),
                label.trailingAnchor.constraint(equalTo: contentView.trailingAnchor),
                label.heightAnchor.constraint(equalToConstant: 50),
                label.topAnchor.constraint(equalTo: lastLabel?.bottomAnchor ?? contentView.topAnchor, constant: 20)
            ])
            lastLabel = label
        }

        if let lastLabel = lastLabel {
            lastLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -20).isActive = true
        }
    }
}
```

✅ Bây giờ, chạy app → kéo dọc để xem 15 dòng nội dung.

---

## 🧩 3. Kết hợp ScrollView + StackView (chuẩn Apple)

Cách tối ưu nhất hiện nay để xây dựng layout động là:

> ScrollView → ContentView → StackView → Subviews

```swift
private func setupStackLayout() {
    let scrollView = UIScrollView()
    let stackView = UIStackView()
    stackView.axis = .vertical
    stackView.spacing = 12
    stackView.translatesAutoresizingMaskIntoConstraints = false
    scrollView.translatesAutoresizingMaskIntoConstraints = false

    view.addSubview(scrollView)
    scrollView.addSubview(stackView)

    NSLayoutConstraint.activate([
        scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
        scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
        scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor),

        stackView.topAnchor.constraint(equalTo: scrollView.topAnchor, constant: 20),
        stackView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor, constant: 16),
        stackView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor, constant: -16),
        stackView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
        stackView.widthAnchor.constraint(equalTo: scrollView.widthAnchor, constant: -32)
    ])

    for i in 1...10 {
        let label = UILabel()
        label.text = "📘 Mục \(i)"
        label.font = .preferredFont(forTextStyle: .title3)
        label.textAlignment = .center
        label.backgroundColor = .secondarySystemBackground
        label.layer.cornerRadius = 10
        label.layer.masksToBounds = true
        label.heightAnchor.constraint(equalToConstant: 80).isActive = true
        stackView.addArrangedSubview(label)
    }
}
```

💡 Ưu điểm:

* Tự động co giãn theo nội dung.
* Rất ít constraint thủ công.
* Dễ ẩn/hiện phần tử động.

---

## 🎨 4. Paging – Lật trang ngang (Onboarding, Gallery)

Tạo ScrollView có nhiều “màn” lật ngang.

```swift
private func setupPagingScroll() {
    let scrollView = UIScrollView()
    scrollView.isPagingEnabled = true
    scrollView.showsHorizontalScrollIndicator = false
    scrollView.translatesAutoresizingMaskIntoConstraints = false
    view.addSubview(scrollView)

    NSLayoutConstraint.activate([
        scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
        scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
        scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
    ])

    var lastView: UIView?
    for i in 1...3 {
        let page = UIView()
        page.backgroundColor = [UIColor.systemRed, .systemGreen, .systemBlue][i-1]
        page.translatesAutoresizingMaskIntoConstraints = false
        scrollView.addSubview(page)

        NSLayoutConstraint.activate([
            page.topAnchor.constraint(equalTo: scrollView.topAnchor),
            page.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
            page.widthAnchor.constraint(equalTo: view.widthAnchor),
            page.heightAnchor.constraint(equalTo: view.heightAnchor),
            page.leadingAnchor.constraint(equalTo: lastView?.trailingAnchor ?? scrollView.leadingAnchor)
        ])
        lastView = page
    }
    lastView?.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor).isActive = true
}
```

➡️ Giờ em có thể **vuốt ngang** giữa các trang như slide giới thiệu app 🎯.

---

## 🔍 5. Zoom ảnh (Photos App style)

```swift
private func setupZoomImage() {
    let scrollView = UIScrollView()
    let imageView = UIImageView(image: UIImage(named: "landscape"))
    imageView.contentMode = .scaleAspectFit

    scrollView.delegate = self
    scrollView.minimumZoomScale = 1.0
    scrollView.maximumZoomScale = 3.0
    scrollView.translatesAutoresizingMaskIntoConstraints = false
    imageView.translatesAutoresizingMaskIntoConstraints = false

    view.addSubview(scrollView)
    scrollView.addSubview(imageView)

    NSLayoutConstraint.activate([
        scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
        scrollView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
        scrollView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        scrollView.bottomAnchor.constraint(equalTo: view.bottomAnchor),

        imageView.topAnchor.constraint(equalTo: scrollView.topAnchor),
        imageView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
        imageView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
        imageView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
        imageView.widthAnchor.constraint(equalTo: scrollView.widthAnchor)
    ])
}

extension ScrollDemoViewController: UIScrollViewDelegate {
    func viewForZooming(in scrollView: UIScrollView) -> UIView? {
        return scrollView.subviews.first // ảnh
    }
}
```

📸 Giờ em có thể **chụm 2 ngón để zoom ảnh** — cực kỳ mượt và chuẩn native!

---

## 🧪 6. Kiểm thử thực tế

✅ Cuộn dọc mượt và không bị giới hạn.
✅ Onboarding slide lật ngang đúng kích thước.
✅ Zoom ảnh tự nhiên, không vỡ hình.
✅ Layout chạy tốt trên mọi thiết bị (iPhone, iPad, ngang/dọc).

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #20)** 🎒

| Mức độ        | Bài tập                         | Gợi ý                    |
| ------------- | ------------------------------- | ------------------------ |
| 🟢 Cơ bản     | Cuộn dọc danh sách 20 phần tử   | ScrollView + ContentView |
| 🟡 Trung bình | Dùng StackView trong ScrollView | ScrollView → StackView   |
| 🔵 Nâng cao   | Tạo paging ngang 3 trang        | isPagingEnabled = true   |
| 🟣 Thử thách  | Zoom ảnh với gesture            | `UIScrollViewDelegate`   |

---

## 📚 Tổng kết

| Thành phần           | Vai trò                           |
| -------------------- | --------------------------------- |
| ScrollView           | Cho phép cuộn nội dung lớn        |
| Content Size         | Kích thước thực của nội dung      |
| StackView            | Dễ dàng tạo bố cục động bên trong |
| Paging               | Lật giữa các trang                |
| Zoom                 | Phóng to / thu nhỏ nội dung       |
| UIScrollViewDelegate | Điều khiển hành vi zoom & cuộn    |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Làm cuộn dọc và ngang trong UIKit.
* Dùng StackView để xây layout động trong ScrollView.
* Tạo Onboarding view có paging.
* Zoom ảnh tự nhiên như Photos App.

---

🎓 **UIKit – Bài 21 (buổi tới):**

> *TableView nâng cao – Cell động, Custom Header/Footer, Reload, Diffable.*

Thầy sẽ dạy:

* Cách tạo cell động, nhiều loại trong 1 table.
* Header/Footer tùy chỉnh.
* Reload dữ liệu mượt.
* Diffable Data Source cho TableView (iOS 15+).

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 21: TableView nâng cao (Custom Cell + Diffable)** không?
