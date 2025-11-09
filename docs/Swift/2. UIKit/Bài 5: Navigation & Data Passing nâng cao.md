Rất tốt 👏 — hôm nay ta sang **UIKit – Bài 5: Navigation & Truyền dữ liệu giữa ViewController (Delegate, Closure, Notification)**.
Thầy sẽ dạy **chậm – có giải thích – có ví dụ dễ hiểu – có code chạy được**.

---

# 🧩 UIKit – Bài 5: Navigation & Data Passing (chuyển màn hình & truyền dữ liệu)

---

## 🎯 **Mục tiêu buổi học**

Sau bài này, em sẽ biết:

1. Cách **chuyển màn hình** (push / modal).
2. Cách **truyền dữ liệu từ A → B** (forward data).
3. Cách **truyền ngược từ B → A** (backward data) bằng:

   * Delegate
   * Closure
   * NotificationCenter
4. Hiểu rõ luồng dữ liệu trong mô hình UIKit.

---

## 🧠 1. Khởi đầu – điều hướng bằng `UINavigationController`

Khi app có nhiều màn hình, `UINavigationController` là “xương sống”.

Ví dụ: `MainViewController` → `DetailViewController`.

### Cách khởi tạo Navigation

```swift
let nav = UINavigationController(rootViewController: MainViewController())
window.rootViewController = nav
```

### Cách **push sang màn hình khác**

```swift
let detailVC = DetailViewController()
navigationController?.pushViewController(detailVC, animated: true)
```

### Cách **trở lại màn trước**

```swift
navigationController?.popViewController(animated: true)
```

---

## 🧩 2. Truyền dữ liệu **A → B**

Ví dụ: từ màn danh sách học sinh → chi tiết học sinh.

**StudentListViewController.swift**

```swift
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    let student = students[indexPath.row]
    let detailVC = StudentDetailViewController(student: student)
    navigationController?.pushViewController(detailVC, animated: true)
}
```

**StudentDetailViewController.swift**

```swift
final class StudentDetailViewController: UIViewController {
    private let student: Student
    
    init(student: Student) {
        self.student = student
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }
}
```

✅ Cách này là **truyền dữ liệu một chiều** — từ A → B (dễ và an toàn).

---

## ⚙️ 3. Truyền dữ liệu **ngược lại (B → A)**

Giả sử ta có màn hình `EditScoreViewController` → sửa điểm của học sinh,
và muốn khi “Lưu” xong, quay lại `StudentDetailViewController` thì điểm hiển thị được cập nhật.

Có **3 cách chính** để làm điều đó.

---

### 🧩 Cách 1 – Delegate (chuẩn Apple)

> Dùng **protocol** để B gửi dữ liệu ngược về A.
> Tư duy: “Báo lại cho người ủy quyền (delegate) biết mình đã thay đổi gì.”

---

#### 1️⃣ Tạo protocol trong `EditScoreViewController`

```swift
protocol EditScoreDelegate: AnyObject {
    func didUpdateScore(_ newScore: Double)
}
```

---

#### 2️⃣ Trong `EditScoreViewController`

```swift
final class EditScoreViewController: UIViewController {
    weak var delegate: EditScoreDelegate?
    private var currentScore: Double

    init(currentScore: Double) {
        self.currentScore = currentScore
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        setupUI()
    }

    private func setupUI() {
        let button = UIButton(type: .system)
        button.setTitle("Lưu điểm mới", for: .normal)
        button.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(button)
        NSLayoutConstraint.activate([
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            button.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    @objc private func saveTapped() {
        let newScore = Double.random(in: 7...10)
        delegate?.didUpdateScore(newScore)
        navigationController?.popViewController(animated: true)
    }
}
```

---

#### 3️⃣ Trong `StudentDetailViewController`

```swift
final class StudentDetailViewController: UIViewController, EditScoreDelegate {
    private var scoreLabel = UILabel()
    private var score: Double

    init(score: Double) {
        self.score = score
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        title = "Chi tiết điểm"
        setupUI()
    }

    private func setupUI() {
        scoreLabel.text = "Điểm hiện tại: \(score)"
        scoreLabel.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(scoreLabel)
        NSLayoutConstraint.activate([
            scoreLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            scoreLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])

        navigationItem.rightBarButtonItem = UIBarButtonItem(
            title: "Sửa",
            style: .plain,
            target: self,
            action: #selector(editTapped)
        )
    }

    @objc private func editTapped() {
        let editVC = EditScoreViewController(currentScore: score)
        editVC.delegate = self
        navigationController?.pushViewController(editVC, animated: true)
    }

    func didUpdateScore(_ newScore: Double) {
        score = newScore
        scoreLabel.text = "Điểm hiện tại: \(score)"
    }
}
```

✅ Ưu điểm: **Chuẩn mực, rõ ràng, dễ quản lý** (Apple khuyên dùng).
❌ Nhược: cần định nghĩa thêm protocol.

---

### 🧩 Cách 2 – Closure (gọn hơn)

> Thay vì dùng delegate, truyền ngược dữ liệu bằng **closure callback** (hàm ẩn danh).

**EditScoreViewController.swift**

```swift
final class EditScoreViewController: UIViewController {
    var onScoreChanged: ((Double) -> Void)?

    @objc private func saveTapped() {
        let newScore = Double.random(in: 6...10)
        onScoreChanged?(newScore)
        navigationController?.popViewController(animated: true)
    }
}
```

**StudentDetailViewController.swift**

```swift
@objc private func editTapped() {
    let editVC = EditScoreViewController()
    editVC.onScoreChanged = { [weak self] newScore in
        self?.score = newScore
        self?.scoreLabel.text = "Điểm hiện tại: \(newScore)"
    }
    navigationController?.pushViewController(editVC, animated: true)
}
```

✅ Ưu điểm: **Gọn, dễ hiểu, code cùng chỗ.**
❌ Nhược: Dễ quên `[weak self]` → memory leak.

---

### 🧩 Cách 3 – NotificationCenter (broadcast)

> Dùng khi nhiều nơi trong app cần biết sự thay đổi (global event).

**EditScoreViewController**

```swift
@objc private func saveTapped() {
    let newScore = Double.random(in: 7...10)
    NotificationCenter.default.post(
        name: .scoreUpdated,
        object: nil,
        userInfo: ["score": newScore]
    )
    navigationController?.popViewController(animated: true)
}
```

**StudentDetailViewController**

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    NotificationCenter.default.addObserver(
        self,
        selector: #selector(scoreChanged(_:)),
        name: .scoreUpdated,
        object: nil
    )
}

@objc private func scoreChanged(_ notification: Notification) {
    if let score = notification.userInfo?["score"] as? Double {
        scoreLabel.text = "Điểm hiện tại: \(score)"
    }
}
```

**Extension**

```swift
extension Notification.Name {
    static let scoreUpdated = Notification.Name("scoreUpdated")
}
```

✅ Ưu điểm: dễ phát cho nhiều màn hình.
❌ Nhược: khó kiểm soát, dễ lỗi nếu không remove observer.

---

## 🧩 4. Tổng kết 3 cách truyền dữ liệu ngược

| Cách             | Ưu điểm                        | Khi dùng              |
| ---------------- | ------------------------------ | --------------------- |
| **Delegate**     | Chuẩn mực, an toàn, dùng nhiều | Khi 1–1 giữa 2 VC     |
| **Closure**      | Gọn, dễ hiểu                   | Khi callback đơn giản |
| **Notification** | Phát cho nhiều nơi cùng nghe   | Khi cần broadcast     |

---

## 🧪 5. Mini Project: Quản lý điểm học sinh

### Chức năng:

* Màn `StudentDetail` → xem điểm.
* Nhấn “Sửa điểm” → `EditScoreViewController`.
* Sửa xong → cập nhật ngược về detail.

👉 Làm theo từng bước ở ví dụ delegate hoặc closure,
khi chạy sẽ thấy: điểm tự đổi, không cần reload lại toàn màn hình.

---

## 🏠 **Bài tập về nhà (UIKit #5)** 🎒

| Mức độ        | Bài tập                                                                                         | Gợi ý                       |
| ------------- | ----------------------------------------------------------------------------------------------- | --------------------------- |
| 🟢 Cơ bản     | Thực hành truyền dữ liệu từ A → B (push chi tiết)                                               | Dùng init()                 |
| 🟡 Trung bình | Truyền dữ liệu ngược lại (B → A) bằng delegate                                                  | Dùng protocol               |
| 🔵 Nâng cao   | Dùng closure thay delegate                                                                      | Biến `onDataChanged`        |
| 🟣 Thử thách  | Dùng NotificationCenter để phát “điểm trung bình lớp thay đổi” cho nhiều màn hình cùng cập nhật | Extension Notification.Name |

---

## 📚 **Tổng kết kiến thức**

| Chủ đề               | Ý nghĩa                             | Ví dụ                                        |
| -------------------- | ----------------------------------- | -------------------------------------------- |
| `pushViewController` | Chuyển sang màn hình khác           | `navigationController?.pushViewController()` |
| Delegate             | Truyền ngược qua protocol           | `delegate?.didUpdate(...)`                   |
| Closure              | Truyền ngược qua callback           | `onChanged?()`                               |
| NotificationCenter   | Phát thông báo toàn cục             | `post(name:userInfo:)`                       |
| weak self            | Tránh retain cycle khi dùng closure | `[weak self] in`                             |

---

## 🧭 **Kết thúc bài học**

✅ Em đã hiểu:

* Cách chuyển màn hình trong UIKit.
* Cách truyền dữ liệu hai chiều.
* Cách chọn kỹ thuật phù hợp cho từng tình huống.

---

🎓 **UIKit – Bài 6 (buổi sau):**
👉 *Form nhập dữ liệu & bàn phím: UITextField, UITextView, delegate nhập liệu, dismiss bàn phím, validate dữ liệu.*

Buổi sau thầy sẽ hướng dẫn **tạo form đăng ký học sinh** có:

* Trường nhập tên, lớp, điểm.
* Nút “Lưu” có validate hợp lệ.
* Ẩn bàn phím khi chạm ra ngoài.
* Dùng delegate để kiểm soát input.

---

Thầy hỏi nhỏ:
👉 Em có muốn thầy sang luôn **Bài 6: Form nhập dữ liệu (UITextField, Validation, Keyboard)** không, để nối tiếp phần truyền dữ liệu này?
