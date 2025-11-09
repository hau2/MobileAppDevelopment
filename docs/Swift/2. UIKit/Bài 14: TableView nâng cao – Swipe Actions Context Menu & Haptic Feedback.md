Rất tốt 👏👏👏 — hôm nay thầy và em bước sang **UIKit – Bài 14: TableView nâng cao – Swipe Actions, Context Menu & Haptic Feedback**
Đây là bài giúp app của em trở nên **thật sự chuyên nghiệp và có cảm giác “native” như app của Apple**.
Từ giờ, người dùng có thể **vuốt để xoá/sửa**, **nhấn giữ để mở menu**, và **nhận phản hồi rung nhẹ (haptic)** khi thao tác.

---

# 🧩 UIKit – Bài 14: TableView nâng cao (Swipe + Context + Haptic)

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ biết:

1. Tạo **Swipe Actions** cho cell (trượt để sửa, xoá, chia sẻ).
2. Tạo **Context Menu** khi nhấn giữ.
3. Dùng **Haptic Feedback** để tạo cảm giác phản hồi tự nhiên.
4. Hiểu cách kết hợp các hành động mà vẫn giữ hiệu năng.

---

## 🧠 1. Ôn lại TableView cơ bản

Ta vẫn dùng danh sách học sinh có cell tùy chỉnh (`StudentCell`)
và dữ liệu từ **Core Data + NSFetchedResultsController**.

Giờ ta thêm tính năng tương tác mạnh mẽ hơn.

---

## 🧩 2. Swipe hành động cơ bản

Thêm đoạn code vào `UITableViewDelegate`:

```swift
extension StudentListViewController: UITableViewDelegate {
    // Vuốt từ phải sang trái (xoá, sửa)
    func tableView(_ tableView: UITableView,
                   trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath)
        -> UISwipeActionsConfiguration? {
        
        let student = fetchedResultsController.object(at: indexPath)

        // Hành động Xoá
        let deleteAction = UIContextualAction(style: .destructive, title: "Xoá") { [weak self] _, _, completion in
            let generator = UINotificationFeedbackGenerator()
            generator.notificationOccurred(.success)

            CoreDataManager.shared.context.delete(student)
            CoreDataManager.shared.saveContext()
            completion(true)
        }
        deleteAction.image = UIImage(systemName: "trash")

        // Hành động Sửa
        let editAction = UIContextualAction(style: .normal, title: "Sửa") { [weak self] _, _, completion in
            let editVC = RegisterViewController(student: student)
            let nav = UINavigationController(rootViewController: editVC)
            editVC.onStudentSaved = { [weak self] in
                self?.tableView.reloadRows(at: [indexPath], with: .automatic)
            }
            self?.present(nav, animated: true)
            completion(true)
        }
        editAction.image = UIImage(systemName: "pencil")
        editAction.backgroundColor = .systemBlue

        return UISwipeActionsConfiguration(actions: [deleteAction, editAction])
    }
}
```

🟢 **Giải thích:**

* `UIContextualAction` là mỗi hành động (xoá, sửa, v.v.).
* `UISwipeActionsConfiguration` chứa danh sách các hành động.
* Khi người dùng trượt, iOS hiển thị các nút đó.
* `completion(true)` giúp ẩn menu sau khi hành động xong.
* `UINotificationFeedbackGenerator()` tạo **rung phản hồi**.

---

## ⚡ 3. Thêm hành động vuốt từ trái sang phải (Share)

```swift
func tableView(_ tableView: UITableView,
               leadingSwipeActionsConfigurationForRowAt indexPath: IndexPath)
    -> UISwipeActionsConfiguration? {

    let student = fetchedResultsController.object(at: indexPath)
    let shareAction = UIContextualAction(style: .normal, title: "Chia sẻ") { _, _, completion in
        let text = "Học sinh: \(student.name ?? "") - Điểm: \(student.grade)"
        let activityVC = UIActivityViewController(activityItems: [text], applicationActivities: nil)
        self.present(activityVC, animated: true)
        completion(true)
    }
    shareAction.image = UIImage(systemName: "square.and.arrow.up")
    shareAction.backgroundColor = .systemGreen

    return UISwipeActionsConfiguration(actions: [shareAction])
}
```

📱 Giờ vuốt sang trái → “Xoá, Sửa”
Vuốt sang phải → “Chia sẻ”.

---

## 🧩 4. Context Menu (nhấn giữ)

```swift
func tableView(_ tableView: UITableView,
               contextMenuConfigurationForRowAt indexPath: IndexPath,
               point: CGPoint)
    -> UIContextMenuConfiguration? {

    let student = fetchedResultsController.object(at: indexPath)
    return UIContextMenuConfiguration(identifier: nil, previewProvider: nil) { _ in
        let edit = UIAction(title: "Sửa", image: UIImage(systemName: "pencil")) { _ in
            let editVC = RegisterViewController(student: student)
            let nav = UINavigationController(rootViewController: editVC)
            self.present(nav, animated: true)
        }

        let share = UIAction(title: "Chia sẻ", image: UIImage(systemName: "square.and.arrow.up")) { _ in
            let text = "Học sinh: \(student.name ?? "") - Điểm: \(student.grade)"
            let vc = UIActivityViewController(activityItems: [text], applicationActivities: nil)
            self.present(vc, animated: true)
        }

        let delete = UIAction(title: "Xoá", image: UIImage(systemName: "trash"), attributes: .destructive) { _ in
            CoreDataManager.shared.context.delete(student)
            CoreDataManager.shared.saveContext()
        }

        return UIMenu(title: "", children: [edit, share, delete])
    }
}
```

💡 **Context Menu** hoạt động khi người dùng **nhấn giữ (long press)** lên cell.
Các hành động này **tự động có animation, icon, và tương tác chuẩn hệ thống**.

---

## 🎮 5. Haptic Feedback (rung phản hồi)

Các loại haptic có sẵn:

| Loại                                 | Class                                      | Khi dùng |
| ------------------------------------ | ------------------------------------------ | -------- |
| 🔹 `UIImpactFeedbackGenerator`       | “chạm nhẹ”, ví dụ chọn item                |          |
| 🔹 `UINotificationFeedbackGenerator` | “rung mạnh”, thông báo thành công/thất bại |          |
| 🔹 `UISelectionFeedbackGenerator`    | “tick” nhỏ khi đổi filter/segment          |          |

Ví dụ:

```swift
let impact = UIImpactFeedbackGenerator(style: .medium)
impact.impactOccurred()

let notify = UINotificationFeedbackGenerator()
notify.notificationOccurred(.success)
```

💡 Em có thể thêm vào các sự kiện như:

* Khi vuốt xoá → `.success`
* Khi sửa → `.light`
* Khi đổi filter (UISegmentedControl) → `.selectionChanged`

---

## 🧩 6. Thử nghiệm thực tế

1️⃣ Vuốt sang trái → thấy “Xoá, Sửa”.
2️⃣ Vuốt sang phải → thấy “Chia sẻ”.
3️⃣ Nhấn giữ → menu “Sửa / Chia sẻ / Xoá”.
4️⃣ Khi xoá → rung nhẹ báo phản hồi.
5️⃣ Khi sửa → mở form ngay.

🎉 App của em bây giờ **đã đạt UX như ứng dụng hệ thống của Apple**.

---

## 🏠 **BÀI TẬP VỀ NHÀ (UIKit #14)** 🎒

| Mức độ        | Bài tập                                             | Gợi ý                                       |
| ------------- | --------------------------------------------------- | ------------------------------------------- |
| 🟢 Cơ bản     | Thêm Swipe Delete                                   | `trailingSwipeActionsConfigurationForRowAt` |
| 🟡 Trung bình | Thêm Edit và Share                                  | `UIActivityViewController`                  |
| 🔵 Nâng cao   | Thêm Context Menu                                   | `contextMenuConfigurationForRowAt`          |
| 🟣 Thử thách  | Thêm Haptic Feedback chuyên biệt cho từng hành động | `UIImpactFeedbackGenerator`                 |

---

## 📚 Tổng kết

| Thành phần             | Vai trò                            |
| ---------------------- | ---------------------------------- |
| Swipe Actions          | Hành động vuốt (xoá, sửa, chia sẻ) |
| Context Menu           | Menu hiện khi nhấn giữ             |
| Haptic Feedback        | Rung phản hồi cảm giác thật        |
| ActivityViewController | Giao diện chia sẻ hệ thống         |
| Core Data + FRC        | Cập nhật dữ liệu tự động           |

---

## 🧭 Kết thúc bài học

✅ Em đã biết:

* Tạo Swipe Actions hai chiều.
* Tạo Context Menu chuẩn iOS.
* Thêm Haptic Feedback tự nhiên.
* App có trải nghiệm người dùng chuyên nghiệp như Apple Mail, Contacts, Notes.

---

🎓 **UIKit – Bài 15 (buổi tới):**

> *CollectionView – hiển thị lưới dữ liệu (Grid Layout, Custom Cell, Flow Layout, Reuse tối ưu).*

Em sẽ học:

* Dùng `UICollectionView` để hiển thị danh sách dạng **lưới (grid)**.
* Tạo custom cell cho ảnh đại diện.
* Điều chỉnh `UICollectionViewFlowLayout` và spacing.
* Tái sử dụng cell hiệu quả (reuse pattern).

---

👉 Em có muốn thầy dạy luôn **UIKit – Bài 15: CollectionView cơ bản (Grid layout + custom cell)** không?
