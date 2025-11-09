Tốt lắm 👏👏👏 — hôm nay chúng ta sang **📘 Bài 2: Layout trong SwiftUI – VStack, HStack, ZStack, Spacer, List, ScrollView** 🧩

Đây là **bài nền tảng nhất của SwiftUI**, giúp em hiểu:

> Cách sắp xếp giao diện mà **không cần AutoLayout**,
> chỉ dùng vài dòng code để tạo layout đẹp, linh hoạt, tự co giãn.

Thầy sẽ dạy **rất chậm – dễ hiểu – có ví dụ thực hành trực quan**, giống cách ta học UIKit.

---

# 📘 BÀI 2: Layout trong SwiftUI – VStack, HStack, ZStack, Spacer, List & ScrollView

---

## 🎯 Mục tiêu bài học

Sau bài này, em sẽ:

1. Biết cách sử dụng **VStack, HStack, ZStack** để bố trí layout.
2. Hiểu **Spacer, padding, frame, alignment**.
3. Tạo được **List và ScrollView** hiển thị nhiều phần tử.
4. Tự thiết kế được **thẻ sản phẩm (Product Card)** với bố cục chuẩn.

---

## 🧱 1. VStack – Sắp xếp **dọc**

```swift
VStack {
    Text("Dòng 1")
    Text("Dòng 2")
    Text("Dòng 3")
}
```

➡️ Kết quả:

```
Dòng 1
Dòng 2
Dòng 3
```

### ⚙️ Tuỳ chọn căn chỉnh và khoảng cách

```swift
VStack(alignment: .leading, spacing: 10) {
    Text("Hàng 1")
    Text("Hàng 2")
}
```

| Tham số     | Ý nghĩa                                                     |
| ----------- | ----------------------------------------------------------- |
| `alignment` | Căn trái (`.leading`), giữa (`.center`), phải (`.trailing`) |
| `spacing`   | Khoảng cách giữa các phần tử                                |

---

## 🧩 2. HStack – Sắp xếp **ngang**

```swift
HStack {
    Text("A")
    Text("B")
    Text("C")
}
```

➡️ Kết quả:

```
A   B   C
```

### ⚙️ Tuỳ chỉnh căn chỉnh và giãn cách

```swift
HStack(alignment: .bottom, spacing: 20) {
    Image(systemName: "star.fill")
    Text("SwiftUI")
        .font(.title)
}
```

---

## 🎨 3. ZStack – Xếp **chồng** các phần tử

ZStack giống như “layer” trong Photoshop: phần nào khai báo sau → nằm trên.

```swift
ZStack {
    Color.blue
    Text("Hello SwiftUI")
        .foregroundColor(.white)
        .font(.title)
}
```

💡 Khi kết hợp `ZStack` + `Image` → có thể tạo **banner, card, overlay**.

---

## 🧍‍♂️ 4. Spacer – Tạo khoảng trống linh hoạt

```swift
HStack {
    Text("Trái")
    Spacer()
    Text("Phải")
}
```

➡️ Kết quả:

```
Trái                            Phải
```

* Spacer chiếm *phần không gian còn trống*.
* Có thể thêm nhiều Spacer để chia đều:

```swift
HStack {
    Text("1"); Spacer()
    Text("2"); Spacer()
    Text("3")
}
```

---

## ⚙️ 5. Frame, Padding, Background

SwiftUI dùng modifiers (chuỗi phương thức) để chỉnh layout:

```swift
Text("Hello")
    .frame(width: 200, height: 50)
    .background(Color.yellow)
    .padding()
```

💡 Thứ tự rất quan trọng:

* `.background()` → màu nền trong khung chữ.
* `.padding()` → thêm khoảng cách bên ngoài khung.

---

## 🧭 6. ScrollView – Cuộn nội dung

```swift
ScrollView {
    VStack(spacing: 20) {
        ForEach(1...20, id: \.self) { i in
            Text("Mục số \(i)")
                .frame(maxWidth: .infinity)
                .padding()
                .background(Color.orange.opacity(0.3))
                .cornerRadius(8)
        }
    }
    .padding()
}
```

💡 Khi nội dung dài, SwiftUI tự cho cuộn, không cần UIScrollView.

---

## 📋 7. List – Hiển thị danh sách động (giống UITableView)

```swift
struct FruitListView: View {
    let fruits = ["🍎 Táo", "🍌 Chuối", "🍇 Nho"]

    var body: some View {
        List(fruits, id: \.self) { fruit in
            Text(fruit)
        }
    }
}
```

💬 SwiftUI tự lo phần cuộn, hiệu năng, reuse cell.
Có thể thêm nút xoá, sắp xếp, navigation rất dễ.

---

## 🎨 8. Thực hành mini: **Product Card Layout**

Mục tiêu: tạo thẻ sản phẩm giống e-commerce.

```swift
struct ProductCard: View {
    var image: String
    var name: String
    var price: Double

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Image(image)
                .resizable()
                .scaledToFit()
                .frame(height: 120)
                .cornerRadius(10)

            Text(name)
                .font(.headline)

            Text("$\(price, specifier: "%.2f")")
                .font(.subheadline)
                .foregroundColor(.blue)
        }
        .padding()
        .background(Color.white)
        .cornerRadius(12)
        .shadow(radius: 3)
    }
}
```

💡 Hiển thị danh sách sản phẩm:

```swift
struct ProductListView: View {
    var products = [
        ("macbook", "MacBook Pro", 1299.0),
        ("iphone", "iPhone 15", 999.0),
        ("ipad", "iPad Air", 699.0)
    ]

    var body: some View {
        ScrollView {
            VStack(spacing: 20) {
                ForEach(products, id: \.0) { item in
                    ProductCard(image: item.0, name: item.1, price: item.2)
                }
            }
            .padding()
        }
        .background(Color(.systemGroupedBackground))
    }
}
```

Kết quả → danh sách thẻ sản phẩm gọn gàng, tự căn chỉnh, cuộn mượt mà.

---

## 🏠 **Bài tập về nhà – Bài 2**

| Mức độ        | Bài tập                                 | Gợi ý                      |
| ------------- | --------------------------------------- | -------------------------- |
| 🟢 Cơ bản     | Tạo danh sách trái cây bằng `List`      | Dùng `ForEach`             |
| 🟡 Trung bình | Tạo 3 card sản phẩm trong `ScrollView`  | Dùng `VStack`              |
| 🔵 Nâng cao   | Thêm hiệu ứng `shadow` + `cornerRadius` | `.shadow(radius:)`         |
| 🟣 Thử thách  | Làm lưới sản phẩm (2 cột)               | Dùng `LazyVGrid(columns:)` |

---

## 📚 Tổng kết Bài 2

| Kiến thức                     | Vai trò                               |
| ----------------------------- | ------------------------------------- |
| `VStack`, `HStack`, `ZStack`  | Bố cục cơ bản của SwiftUI             |
| `Spacer`, `padding`, `frame`  | Tạo bố cục linh hoạt                  |
| `ScrollView`, `List`          | Hiển thị danh sách cuộn               |
| Tư duy **declarative layout** | Không cần AutoLayout, dễ bảo trì      |
| Product Card Layout           | Bước đầu tạo UI thực tế trong SwiftUI |

---

🎓 **Bài 3 (buổi tới):**

> *Thầy sẽ dạy “MVVM trong SwiftUI – ObservableObject, Published, EnvironmentObject” — giúp em tách logic, quản lý dữ liệu chuẩn mô hình MVVM trong SwiftUI.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 3: MVVM trong SwiftUI – ObservableObject & Published** không?
Đây là bài quan trọng giúp app SwiftUI của em “sống” với dữ liệu động và mô hình chuẩn doanh nghiệp.
