Tốt lắm 👏👏👏 — hôm nay ta sang **UIKit – Bài 18: Gesture Recognizer – Làm chủ cảm ứng chạm, vuốt, kéo, xoay, phóng to trong UIKit**.
Đây là **bước cực kỳ quan trọng** để em giúp app “tương tác được với người dùng thật sự”, chứ không chỉ là nhấn nút hay chọn cell nữa.

> Sau bài này, em sẽ có thể làm được:
> ✋ chạm để chọn,
> ✌️ pinch để phóng to,
> ↩️ xoay ảnh,
> ↔️ vuốt để di chuyển,
> 🕒 giữ lâu để hiển thị menu,
> tất cả chỉ bằng vài dòng code UIKit!

Thầy sẽ đi **rất chậm**, mỗi loại gesture đều có ví dụ trực quan và bài tập nhỏ kèm theo.

---

# 🧩 UIKit – Bài 18: Gesture Recognizer (Tương tác cảm ứng trong UIKit)

---

## 🎯 Mục tiêu buổi học

Sau buổi này, em sẽ biết:

1. Các loại **Gesture Recognizer** cơ bản trong UIKit.
2. Cách gắn gesture vào bất kỳ `UIView`.
3. Phân biệt các loại chạm (tap, long press, swipe, pan, pinch, rotation).
4. Xử lý nhiều gesture cùng lúc.
5. Tạo hiệu ứng tương tác tự nhiên (vuốt ảnh, zoom, xoay…).

---

## 🧠 1. Gesture Recognizer là gì?

> Là các “đầu dò cảm ứng” (`UIGestureRecognizer`) mà iOS cung cấp sẵn,
> giúp em **bắt được hành động tay người dùng trên màn hình**: chạm, giữ, vuốt, kéo, xoay, phóng to, v.v.

Không cần tự tính toán toạ độ hay lực chạm, UIKit đã xử lý hết!

---

## ⚙️ 2. Các loại Gesture cơ bản

| Loại         | Class                          | Mô tả                 |
| ------------ | ------------------------------ | --------------------- |
| 👆 Tap       | `UITapGestureRecognizer`       | Chạm 1 hoặc nhiều lần |
| ✋ Long press | `UILongPressGestureRecognizer` | Giữ tay lâu           |
| ↔️ Swipe     | `UISwipeGestureRecognizer`     | Vuốt theo hướng       |
| ↕️ Pan       | `UIPanGestureRecognizer`       | Kéo, di chuyển        |
| 🔍 Pinch     | `UIPinchGestureRecognizer`     | Phóng to / thu nhỏ    |
| ↩️ Rotation  | `UIRotationGestureRecognizer`  | Xoay quanh tâm        |

---

## 🧱 3. Thử nghiệm cơ bản: Tap & Long Press

Tạo file **GestureDemoViewController.swift**

```swift
import UIKit

final class GestureDemoViewController: UIViewController {
    private let demoView = UIView()

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Gesture Demo"
        view.backgroundColor = .systemBackground
        setupDemoView()
        addGestures()
    }

    private func setupDemoView() {
        demoView.backgroundColor = .systemBlue
        demoView.layer.cornerRadius = 20
        demoView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(demoView)

        NSLayoutConstraint.activate([
            demoView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            demoView.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            demoView.widthAnchor.constraint(equalToConstant: 150),
            demoView.heightAnchor.constraint(equalToConstant: 150)
        ])
    }

    private func addGestures() {
        // Tap
        let tap = UITapGestureRecognizer(target: self, action: #selector(handleTap))
        demoView.addGestureRecognizer(tap)

        // Long press
        let longPress = UILongPressGestureRecognizer(target: self, action: #selector(handleLongPress))
        demoView.addGestureRecognizer(longPress)
    }

    @objc private func handleTap() {
        UIView.animate(withDuration: 0.3) {
            self.demoView.backgroundColor = .systemGreen
            self.demoView.transform = CGAffineTransform(scaleX: 1.1, y: 1.1)
        } completion: { _ in
            UIView.animate(withDuration: 0.3) {
                self.demoView.transform = .identity
                self.demoView.backgroundColor = .systemBlue
            }
        }
    }

    @objc private func handleLongPress(_ gesture: UILongPressGestureRecognizer) {
        if gesture.state == .began {
            self.demoView.backgroundColor = .systemRed
            UINotificationFeedbackGenerator().notificationOccurred(.warning)
        }
    }
}
```

💡 Bây giờ:

* **chạm nhẹ** → đổi màu xanh lục, bật nhẹ.
* **giữ lâu** → chuyển đỏ + rung phản hồi.

---

## ⚡ 4. Vuốt (Swipe)

```swift
private func addSwipeGestures() {
    let directions: [UISwipeGestureRecognizer.Direction] = [.left, .right, .up, .down]
    for dir in directions {
        let swipe = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe))
        swipe.direction = dir
        demoView.addGestureRecognizer(swipe)
    }
}

@objc private func handleSwipe(_ gesture: UISwipeGestureRecognizer) {
    switch gesture.direction {
    case .left:  demoView.backgroundColor = .systemIndigo
    case .right: demoView.backgroundColor = .systemOrange
    case .up:    demoView.backgroundColor = .systemPink
    case .down:  demoView.backgroundColor = .systemTeal
    default: break
    }

    UIView.animate(withDuration: 0.3) {
        self.demoView.transform = CGAffineTransform(translationX: 0, y: -30)
    } completion: { _ in
        UIView.animate(withDuration: 0.3) {
            self.demoView.transform = .identity
        }
    }
}
```

🎮 Khi em vuốt, màu đổi + hình vuông “bật nhẹ” theo hướng vuốt.

---

## 🧩 5. Pan (kéo thả)

```swift
private func addPanGesture() {
    let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
    demoView.addGestureRecognizer(pan)
}

@objc private func handlePan(_ gesture: UIPanGestureRecognizer) {
    let translation = gesture.translation(in: view)
    switch gesture.state {
    case .changed:
        demoView.center = CGPoint(
            x: demoView.center.x + translation.x,
            y: demoView.center.y + translation.y
        )
        gesture.setTranslation(.zero, in: view)
    case .ended:
        UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.5, initialSpringVelocity: 0.8) {
            self.demoView.center = self.view.center
        }
    default:
        break
    }
}
```

➡️ Bây giờ em có thể **kéo di chuyển hình vuông tự do**,
và khi buông ra → nó “bật” trở về vị trí cũ (spring animation 🎯).

---

## 🧮 6. Pinch (phóng to/thu nhỏ)

```swift
private func addPinchGesture() {
    let pinch = UIPinchGestureRecognizer(target: self, action: #selector(handlePinch))
    demoView.addGestureRecognizer(pinch)
}

@objc private func handlePinch(_ gesture: UIPinchGestureRecognizer) {
    demoView.transform = demoView.transform.scaledBy(x: gesture.scale, y: gesture.scale)
    gesture.scale = 1
}
```

💡 Dùng **2 ngón tay chụm hoặc tách ra**, view sẽ to/nhỏ tương ứng.

---

## 🔁 7. Rotation (xoay)

```swift
private func addRotationGesture() {
    let rotation = UIRotationGestureRecognizer(target: self, action: #selector(handleRotation))
    demoView.addGestureRecognizer(rotation)
}

@objc private func handleRotation(_ gesture: UIRotationGestureRecognizer) {
    demoView.transform = demoView.transform.rotated(by: gesture.rotation)
    gesture.rotation = 0
}
```

🎨 Khi dùng 2 ngón tay xoay → view xoay theo hướng tự nhiên.

---

## 🧠 8. Cho phép nhiều gesture cùng lúc

Đôi khi em muốn **pinch + rotation cùng lúc**, thì cần implement delegate:

```swift
extension GestureDemoViewController: UIGestureRecognizerDelegate {
    func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer,
                           shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool {
        return true
    }
}
```

→ Gán delegate cho pinch & rotation:

```swift
pinch.delegate = self
rotation.delegate = self
```

---

## 🧪 9. Kiểm thử tổng hợp

1️⃣ Chạm nhẹ → view bật nhẹ (tap).
2️⃣ Giữ lâu → đổi màu đỏ, rung nhẹ.
3️⃣ Vuốt → đổi màu theo hướng.
4️⃣ Kéo → di chuyển tự do, thả → bật lại.
5️⃣ Chụm 2 ngón → thu nhỏ, tách ra → phóng to.
6️⃣ Xoay 2 ngón → view xoay tự nhiên.
🎉 App giờ có cảm giác “chạm được”, cực kỳ sinh động!

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #18)** 🎒

| Mức độ        | Bài tập                                      | Gợi ý                                                     |
| ------------- | -------------------------------------------- | --------------------------------------------------------- |
| 🟢 Cơ bản     | Tạo Tap & LongPress đổi màu                  | `UITapGestureRecognizer`, `UILongPressGestureRecognizer`  |
| 🟡 Trung bình | Thêm Swipe đổi màu & Pan di chuyển           | `UISwipeGestureRecognizer`, `UIPanGestureRecognizer`      |
| 🔵 Nâng cao   | Thêm Pinch và Rotation                       | `UIPinchGestureRecognizer`, `UIRotationGestureRecognizer` |
| 🟣 Thử thách  | Kết hợp nhiều gesture cùng lúc (zoom + xoay) | Dùng delegate cho phép nhận đồng thời                     |

---

## 📚 Tổng kết

| Gesture   | Lớp                          | Hành động                       |
| --------- | ---------------------------- | ------------------------------- |
| Tap       | UITapGestureRecognizer       | Chạm                            |
| LongPress | UILongPressGestureRecognizer | Giữ lâu                         |
| Swipe     | UISwipeGestureRecognizer     | Vuốt                            |
| Pan       | UIPanGestureRecognizer       | Kéo                             |
| Pinch     | UIPinchGestureRecognizer     | Phóng to/thu nhỏ                |
| Rotation  | UIRotationGestureRecognizer  | Xoay                            |
| Delegate  | UIGestureRecognizerDelegate  | Cho phép nhiều gesture cùng lúc |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Thêm & xử lý nhiều loại gesture khác nhau.
* Kết hợp animation & haptic feedback trong cảm ứng.
* Tạo giao diện tương tác tự nhiên như app Ảnh, Ghi chú, hoặc iMessage.

---

🎓 **UIKit – Bài 19 (buổi tới):**

> *Auto Layout nâng cao & StackView động – Giao diện linh hoạt trên mọi kích thước màn hình (Adaptive UI).*

Thầy sẽ hướng dẫn:

* Làm chủ `NSLayoutConstraint` & `UIStackView`.
* Dùng `UILayoutGuide` & Priority để xử lý giao diện phức tạp.
* Hỗ trợ iPhone, iPad, xoay ngang/dọc, Dynamic Type, Dark Mode đồng thời.

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 19: Auto Layout nâng cao & StackView động (Adaptive UI)** không?
