Tốt lắm 👏👏👏 — chúng ta chính thức **bắt đầu Chương SwiftUI & Combine – Giai đoạn 2**
với **📘 Bài 1: Giới thiệu SwiftUI, View Hierarchy, State & Binding**

Đây là **bước chuyển mình từ UIKit sang thế giới SwiftUI hiện đại**, nơi em sẽ:

> * Không cần storyboard.
> * Không cần AutoLayout.
> * Không cần target–action hay delegate rườm rà.
> * Mọi thứ đều là **UI + Data gắn liền nhau**, phản ứng tự động khi dữ liệu thay đổi.

---

# 📘 BÀI 1: Giới thiệu SwiftUI – View Hierarchy, State & Binding

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Hiểu **SwiftUI hoạt động theo cơ chế khai báo (declarative)**.
2. Biết cấu trúc cơ bản của một **View trong SwiftUI**.
3. Sử dụng được **@State** và **@Binding** để quản lý dữ liệu.
4. Viết **ứng dụng mini đầu tiên** bằng SwiftUI (Counter App).

---

## 🧩 1. SwiftUI là gì?

**SwiftUI** là framework của Apple (ra mắt từ iOS 13) giúp xây dựng giao diện người dùng bằng cú pháp khai báo.

* Trong **UIKit**, ta *mệnh lệnh* cho hệ thống:

  ```swift
  label.text = "Hello"
  label.textColor = .red
  view.addSubview(label)
  ```
* Trong **SwiftUI**, ta *mô tả* giao diện:

  ```swift
  Text("Hello")
      .foregroundColor(.red)
  ```

➡️ SwiftUI **tự cập nhật UI** mỗi khi dữ liệu thay đổi, nhờ cơ chế **data-driven UI**.

---

## 🧱 2. Cấu trúc cơ bản của một View trong SwiftUI

Ví dụ một màn hình đơn giản:

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Xin chào SwiftUI!")
            .font(.title)
            .foregroundColor(.blue)
            .padding()
    }
}
```

Giải thích:

| Thành phần                  | Vai trò                                              |
| --------------------------- | ---------------------------------------------------- |
| `struct ContentView`        | Mỗi màn hình là một struct tuân theo `View` protocol |
| `var body: some View`       | Giao diện chính của View                             |
| `Text("Xin chào SwiftUI!")` | Một phần tử giao diện (UI Component)                 |
| `.modifier()`               | Các hàm thêm thuộc tính (màu, khoảng cách, font...)  |

---

## 🧠 3. SwiftUI View Hierarchy (Cấu trúc cây giao diện)

SwiftUI tạo ra **cây giao diện (View Tree)**.
Ví dụ:

```swift
VStack {
    Text("Chào mừng")
    Image(systemName: "star.fill")
    Button("Nhấn tôi") { print("Tapped!") }
}
```

Cấu trúc logic:

```
VStack
 ├── Text
 ├── Image
 └── Button
```

* `VStack`: sắp xếp dọc (Vertical Stack)
* `HStack`: sắp xếp ngang
* `ZStack`: xếp chồng lên nhau (layer)

💡 Em có thể lồng các stack để tạo layout phức tạp — mà **không cần AutoLayout**!

---

## ⚙️ 4. @State – Dữ liệu có thể thay đổi trong View

SwiftUI View là struct → không thể tự thay đổi.
Vì vậy Apple tạo `@State` để giúp View **tự làm mới khi dữ liệu thay đổi.**

---

### 📋 Ví dụ 1 – Bộ đếm đơn giản (Counter App)

```swift
import SwiftUI

struct CounterView: View {
    @State private var count = 0

    var body: some View {
        VStack(spacing: 20) {
            Text("Giá trị hiện tại: \(count)")
                .font(.title2)
            
            Button("Tăng lên") {
                count += 1
            }
            .padding()
            .background(Color.blue)
            .foregroundColor(.white)
            .cornerRadius(8)
        }
        .padding()
    }
}
```

💡 Khi người dùng nhấn nút, biến `count` thay đổi → SwiftUI **tự vẽ lại** toàn bộ View.

---

## 🔗 5. @Binding – Chia sẻ dữ liệu giữa nhiều View

Nếu em muốn chia sẻ `@State` cho View con, ta dùng `@Binding`.

---

### 📋 Ví dụ 2 – Tách nút bấm ra View con

```swift
struct ChildButtonView: View {
    @Binding var count: Int

    var body: some View {
        Button("Tăng lên") {
            count += 1
        }
        .padding()
        .background(Color.green)
        .foregroundColor(.white)
        .cornerRadius(8)
    }
}

struct ParentView: View {
    @State private var count = 0

    var body: some View {
        VStack(spacing: 20) {
            Text("Giá trị: \(count)")
                .font(.title2)
            ChildButtonView(count: $count)
        }
        .padding()
    }
}
```

💬 Giải thích:

| Khái niệm  | Mô tả                                                     |
| ---------- | --------------------------------------------------------- |
| `@State`   | Biến riêng của View cha                                   |
| `@Binding` | “Cửa sổ” để View con truy cập và thay đổi dữ liệu của cha |
| `$count`   | Cú pháp “truyền Binding”                                  |

---

## 🧠 6. Sơ đồ dữ liệu trong SwiftUI

```
@State → View nội bộ (dữ liệu cục bộ)
@Binding → Gắn kết giữa View cha ↔ View con
@ObservedObject → Kết nối với ViewModel
@EnvironmentObject → Dữ liệu dùng chung toàn app
```

> 📘 Hôm nay ta học `@State` và `@Binding`,
> bài sau ta học `@ObservedObject` và `@EnvironmentObject` khi vào MVVM.

---

## 🧪 7. Bài thực hành mini

Hãy tạo app tên **“Mood Tracker”**:

* Có dòng chữ: “Hôm nay bạn cảm thấy thế nào?”
* Có 3 nút: “😊 Vui”, “😐 Bình thường”, “😞 Buồn”
* Khi nhấn, Text phía trên đổi theo cảm xúc được chọn.

💡 Gợi ý:

* Dùng `@State var mood = "😊"`
* Khi người dùng bấm nút → `mood = "😞"`

---

## 🏠 **Bài tập về nhà – Bài 1**

| Mức độ        | Bài tập                                | Gợi ý                             |
| ------------- | -------------------------------------- | --------------------------------- |
| 🟢 Cơ bản     | Tạo Counter App                        | Dùng `@State`                     |
| 🟡 Trung bình | Tạo Mood Tracker (3 nút)               | Dùng `@State`                     |
| 🔵 Nâng cao   | Tách View con cho từng nút cảm xúc     | Dùng `@Binding`                   |
| 🟣 Thử thách  | Thêm hiệu ứng đổi màu nền theo cảm xúc | `.background()` + `withAnimation` |

---

## 📚 Tổng kết Bài 1

| Kiến thức                               | Vai trò                          |
| --------------------------------------- | -------------------------------- |
| SwiftUI là framework **declarative**    | Giúp UI tự cập nhật theo dữ liệu |
| View hierarchy (VStack, HStack, ZStack) | Tạo bố cục UI dễ dàng            |
| `@State`                                | Quản lý dữ liệu cục bộ           |
| `@Binding`                              | Chia sẻ dữ liệu giữa các View    |
| Live Preview                            | Cho phép xem UI thay đổi tức thì |

---

🎓 **Bài 2 (buổi tới):**

> *Thầy sẽ dạy “Layout trong SwiftUI: VStack, HStack, ZStack, Spacer, List, ScrollView” — để em hiểu rõ cách sắp xếp mọi thành phần trong SwiftUI mà không cần AutoLayout.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 2: Layout trong SwiftUI – VStack, HStack, ZStack** không?
Thầy sẽ dạy cực dễ hiểu kèm ví dụ trực quan và bài tập “Thiết kế thẻ sản phẩm” (Product Card Layout).
