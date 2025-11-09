Tuyệt vời 👏👏👏 — hôm nay chúng ta bước sang **Tuần 3 – Ngày 15: Image Picker + Lưu ảnh với FileManager** 📸🗂️

Đây là **bước cực kỳ quan trọng** trong lập trình app thực tế: mọi ứng dụng đều cần thao tác với **ảnh do người dùng chọn hoặc chụp** — từ quản lý sản phẩm, hồ sơ cá nhân, đến mạng xã hội.

Sau bài này, em sẽ biết:

* Mở thư viện ảnh hoặc camera để chọn ảnh thật.
* Lưu ảnh đó vào **bộ nhớ cục bộ của app (FileManager)**.
* Hiển thị ảnh đã chọn và đọc lại khi app khởi động.

---

# 📸 UIKit – Tuần 3, Ngày 15: Image Picker + Lưu ảnh bằng FileManager

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ:

1. Biết cách dùng **UIImagePickerController** để chọn hoặc chụp ảnh.
2. Biết cách **lưu ảnh xuống bộ nhớ app** (Documents folder).
3. Hiểu cách **đọc lại ảnh** và gắn vào sản phẩm.
4. Áp dụng vào app hiện tại — thêm ảnh cho từng sản phẩm.

---

## 🧠 1. Kiến thức nền: FileManager trong iOS

Trong iOS, mỗi app có một **vùng lưu trữ riêng biệt**, gồm 3 thư mục chính:

| Tên thư mục   | Mục đích                              |
| ------------- | ------------------------------------- |
| **Documents** | Dữ liệu người dùng (ảnh, file, JSON…) |
| **Caches**    | Dữ liệu tạm (cache ảnh, API)          |
| **Tmp**       | Dữ liệu ngắn hạn, hệ thống có thể xoá |

Chúng ta sẽ dùng **Documents/** để lưu ảnh thật người dùng chọn, đảm bảo:

* Ảnh tồn tại kể cả khi tắt app.
* Không bị xoá bởi hệ thống.

---

## ⚙️ 2. Tạo `ImageStorageManager.swift`

Đây là lớp trung gian giữa app và hệ thống file.

```swift
import UIKit

final class ImageStorageManager {
    static let shared = ImageStorageManager()
    private init() {}

    private var documentsURL: URL {
        FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    }

    func saveImage(_ image: UIImage, withName name: String) -> String? {
        let fileURL = documentsURL.appendingPathComponent("\(name).jpg")
        guard let data = image.jpegData(compressionQuality: 0.8) else { return nil }

        do {
            try data.write(to: fileURL)
            print("✅ Lưu ảnh: \(fileURL.path)")
            return fileURL.lastPathComponent
        } catch {
            print("❌ Lỗi lưu ảnh:", error)
            return nil
        }
    }

    func loadImage(named fileName: String) -> UIImage? {
        let fileURL = documentsURL.appendingPathComponent(fileName)
        return UIImage(contentsOfFile: fileURL.path)
    }

    func deleteImage(named fileName: String) {
        let fileURL = documentsURL.appendingPathComponent(fileName)
        try? FileManager.default.removeItem(at: fileURL)
    }
}
```

💡 Giải thích:

* `saveImage()` → lưu ảnh với tên cụ thể.
* `loadImage()` → đọc ảnh từ bộ nhớ.
* `deleteImage()` → xoá khi cần.

---

## 🧱 3. Cập nhật `AddProductViewController.swift`

Thêm một **UIImageView** để hiển thị ảnh, và nút để chọn ảnh từ camera/thư viện.

```swift
import UIKit

final class AddProductViewController: UIViewController, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    var onAddProduct: ((Product) -> Void)?

    private let imageView = UIImageView()
    private let selectImageButton = UIButton(type: .system)
    private let titleField = UITextField()
    private let priceField = UITextField()
    private let categoryField = UITextField()
    private let descriptionField = UITextView()
    private let saveButton = UIButton(type: .system)

    private var selectedImage: UIImage?

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Thêm sản phẩm"
        view.backgroundColor = .systemBackground
        setupUI()
    }

    private func setupUI() {
        imageView.contentMode = .scaleAspectFill
        imageView.clipsToBounds = true
        imageView.layer.cornerRadius = 8
        imageView.backgroundColor = .systemGray5
        imageView.heightAnchor.constraint(equalToConstant: 150).isActive = true
        imageView.translatesAutoresizingMaskIntoConstraints = false

        selectImageButton.setTitle("📸 Chọn ảnh", for: .normal)
        selectImageButton.addTarget(self, action: #selector(selectImageTapped), for: .touchUpInside)
        selectImageButton.translatesAutoresizingMaskIntoConstraints = false

        [titleField, priceField, categoryField].forEach {
            $0.borderStyle = .roundedRect
            $0.translatesAutoresizingMaskIntoConstraints = false
        }

        descriptionField.layer.borderWidth = 1
        descriptionField.layer.cornerRadius = 8
        descriptionField.layer.borderColor = UIColor.systemGray4.cgColor
        descriptionField.translatesAutoresizingMaskIntoConstraints = false

        saveButton.setTitle("💾 Lưu sản phẩm", for: .normal)
        saveButton.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)
        saveButton.translatesAutoresizingMaskIntoConstraints = false

        let stack = UIStackView(arrangedSubviews: [
            imageView, selectImageButton,
            titleField, priceField, categoryField, descriptionField, saveButton
        ])
        stack.axis = .vertical
        stack.spacing = 12
        stack.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(stack)

        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20),
            stack.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            stack.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16)
        ])
    }

    // MARK: - Image Picker
    @objc private func selectImageTapped() {
        let alert = UIAlertController(title: "Chọn nguồn ảnh", message: nil, preferredStyle: .actionSheet)
        alert.addAction(UIAlertAction(title: "Thư viện", style: .default) { _ in self.openImagePicker(source: .photoLibrary) })
        alert.addAction(UIAlertAction(title: "Camera", style: .default) { _ in self.openImagePicker(source: .camera) })
        alert.addAction(UIAlertAction(title: "Hủy", style: .cancel))
        present(alert, animated: true)
    }

    private func openImagePicker(source: UIImagePickerController.SourceType) {
        guard UIImagePickerController.isSourceTypeAvailable(source) else { return }
        let picker = UIImagePickerController()
        picker.delegate = self
        picker.sourceType = source
        present(picker, animated: true)
    }

    func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
        picker.dismiss(animated: true)
        if let image = info[.originalImage] as? UIImage {
            imageView.image = image
            selectedImage = image
        }
    }

    // MARK: - Save Product
    @objc private func saveTapped() {
        guard let title = titleField.text, !title.isEmpty,
              let price = Double(priceField.text ?? ""),
              let category = categoryField.text, !category.isEmpty else {
            showAlert("Vui lòng điền đầy đủ thông tin.")
            return
        }

        var imageName = "default.jpg"
        if let image = selectedImage {
            imageName = UUID().uuidString
            _ = ImageStorageManager.shared.saveImage(image, withName: imageName)
        }

        let product = Product(
            id: Int.random(in: 100...999),
            title: title,
            price: price,
            description: descriptionField.text,
            category: category,
            image: imageName
        )

        onAddProduct?(product)
        navigationController?.popViewController(animated: true)
    }

    private func showAlert(_ msg: String) {
        let alert = UIAlertController(title: "Thông báo", message: msg, preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

💡 Giải thích:

* `UIImagePickerController` dùng để chọn hoặc chụp ảnh.
* Ảnh được lưu bằng `ImageStorageManager`.
* Ta lưu **tên file ảnh** trong Product, chứ không lưu `UIImage`.

---

## 🧩 4. Hiển thị lại ảnh đã lưu trong `ProductCell.swift`

```swift
func configure(with product: Product) {
    titleLabel.text = product.title
    priceLabel.text = "$\(String(format: "%.2f", product.price))"

    if let localImage = ImageStorageManager.shared.loadImage(named: product.image) {
        productImageView.image = localImage
    } else {
        ImageLoader.shared.loadImage(from: product.image) { [weak self] img in
            self?.productImageView.image = img
        }
    }
}
```

✅ Kết quả:

* Nếu ảnh là file local → hiển thị từ `FileManager`.
* Nếu là URL (ảnh online từ API) → tải qua mạng.

---

## 🧪 5. Chạy thử 🚀

✅ Nhấn **“+” → Thêm sản phẩm**
✅ Chọn **Chụp ảnh hoặc chọn từ Thư viện**
✅ Ảnh hiển thị ngay trên màn hình thêm sản phẩm
✅ Nhấn “💾 Lưu sản phẩm” → ảnh được lưu trong thư mục Documents
✅ Quay lại danh sách → ảnh hiển thị đúng
✅ Đóng app, mở lại → ảnh vẫn còn (đọc từ FileManager)

🎉 App của em giờ **đã hỗ trợ ảnh thật từ người dùng**, giống các ứng dụng quản lý sản phẩm hoặc hồ sơ thực tế.

---

## 🏠 **Bài tập về nhà (Tuần 3, Ngày 15)** 🎒

| Mức độ        | Bài tập                                            | Gợi ý                                    |
| ------------- | -------------------------------------------------- | ---------------------------------------- |
| 🟢 Cơ bản     | Thêm nút “Xóa ảnh” khi người dùng chọn nhầm        | Dùng `ImageStorageManager.deleteImage()` |
| 🟡 Trung bình | Khi sửa sản phẩm, cho phép thay ảnh mới            | Xóa ảnh cũ rồi lưu ảnh mới               |
| 🔵 Nâng cao   | Tạo thư mục riêng `ProductImages/` trong Documents | `FileManager.createDirectory`            |
| 🟣 Thử thách  | Tối ưu kích thước ảnh khi lưu                      | Resize bằng CoreGraphics                 |

---

## 📚 Tổng kết buổi học

| Kiến thức                     | Vai trò                                |
| ----------------------------- | -------------------------------------- |
| **UIImagePickerController**   | Chọn hoặc chụp ảnh                     |
| **FileManager**               | Lưu và đọc file ảnh trong app          |
| **ImageStorageManager**       | Quản lý ảnh local                      |
| **MVVM + Local Files**        | Lưu ảnh gắn với dữ liệu sản phẩm       |
| **Offline Image Persistence** | Giữ ảnh lâu dài, không mất khi tắt app |

---

🎓 **Ngày 16 (buổi tới):**

> *Thầy sẽ dạy “App Lifecycle + Quản lý quyền truy cập (Camera, Photos, Permission Handling)” — để app của em tuân thủ chuẩn Apple khi dùng camera, lưu ảnh, và hoạt động an toàn.*

---

👉 Em có muốn thầy **tiếp tục luôn Ngày 16 – App Lifecycle & Quyền truy cập Camera/Photos** không?
Bài này cực kỳ quan trọng vì giúp app em **được duyệt khi publish lên App Store**, và hiểu sâu cách iOS quản lý vòng đời ứng dụng.
