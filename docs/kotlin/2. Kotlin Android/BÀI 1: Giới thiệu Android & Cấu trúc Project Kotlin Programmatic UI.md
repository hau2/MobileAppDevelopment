Tốt lắm 👏👏👏 — hôm nay chúng ta chính thức bắt đầu **📗 Giai đoạn 2: Kotlin App Programmatic – Lập trình Android bằng Kotlin thuần (không XML)**

Đây là **bước chuyển cực kỳ quan trọng**, nơi em bắt đầu biến những gì học ở **Giai đoạn 1 – Kotlin Core** thành **ứng dụng thực tế trên Android** 🎯
Và ta sẽ **không dùng XML**, mà sẽ **code toàn bộ UI bằng Kotlin**, để giúp em hiểu sâu cách Android thực thi mọi thứ bên trong.

---

# 📗 BÀI 1: Giới thiệu Android & Cấu trúc Project Kotlin Programmatic UI

---

## 🎯 Mục tiêu bài học

Sau bài học này, em sẽ:

1. Hiểu rõ **cấu trúc của một project Android**.
2. Nắm được vai trò của `Activity`, `Context`, `View`, `Layout`.
3. Tạo **màn hình đầu tiên (Hello Android)** hoàn toàn bằng code Kotlin – không XML.
4. Biết cách **chạy thử trên thiết bị / emulator**.

---

## 🧠 1. Android là gì?

> Android là **nền tảng di động của Google**, cho phép lập trình bằng **Java hoặc Kotlin**.
> Mỗi ứng dụng Android bao gồm:
>
> * **Activity**: màn hình chính / UI người dùng.
> * **Layout**: giao diện hiển thị (ta sẽ tạo bằng code).
> * **Manifest**: mô tả cấu hình app (quyền, màn hình chính, v.v.)

---

## ⚙️ 2. Cấu trúc một project Android (theo Android Studio)

```
MyApp/
 ├── app/
 │   ├── src/
 │   │   ├── main/
 │   │   │   ├── java/com/example/myapp/MainActivity.kt
 │   │   │   ├── res/
 │   │   │   │   ├── layout/       # giao diện XML (ta không dùng)
 │   │   │   │   ├── drawable/     # ảnh, icon
 │   │   │   │   ├── values/       # màu, string
 │   │   │   └── AndroidManifest.xml
 │   ├── build.gradle
 ├── build.gradle (project)
 └── gradle.properties
```

💡 Khi dùng UI code (programmatic), ta chỉ quan tâm đến:

* `MainActivity.kt` (điều khiển màn hình)
* Các class View, Layout (tạo giao diện động)

---

## 🧱 3. `Activity` – trái tim của mỗi màn hình

> Một **Activity** là một “màn hình” mà người dùng nhìn thấy.

Ví dụ:

* `MainActivity` → màn hình chính
* `LoginActivity` → đăng nhập
* `ProfileActivity` → hồ sơ người dùng

---

## 📋 4. Mẫu cơ bản của một Activity

```kotlin
package com.example.myapp

import android.app.Activity
import android.os.Bundle
import android.widget.TextView

class MainActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Tạo TextView bằng code
        val textView = TextView(this)
        textView.text = "Hello Android with Kotlin!"
        textView.textSize = 22f
        textView.setPadding(40, 60, 40, 60)

        // Gán layout cho Activity
        setContentView(textView)
    }
}
```

➡️ Khi chạy, ta sẽ thấy **một màn hình trắng có dòng chữ “Hello Android with Kotlin!”**
📱 Và đó là **UI được tạo hoàn toàn bằng code Kotlin**!

---

## 🧩 5. Giải thích chi tiết

| Thành phần         | Vai trò                                     |
| ------------------ | ------------------------------------------- |
| `Activity`         | Lớp màn hình chính của Android              |
| `onCreate()`       | Hàm chạy khi màn hình được tạo              |
| `TextView`         | Thành phần hiển thị chữ                     |
| `setContentView()` | Gán UI cho màn hình                         |
| `this`             | Đại diện cho `Context` (môi trường của app) |

---

## ⚡️ 6. Tạo nhiều thành phần giao diện (View)

Ví dụ: thêm **nút bấm Button** và xử lý sự kiện.

```kotlin
import android.widget.Button
import android.widget.LinearLayout

class MainActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Tạo layout chứa các view
        val layout = LinearLayout(this)
        layout.orientation = LinearLayout.VERTICAL
        layout.setPadding(50, 80, 50, 80)

        // Tạo TextView
        val text = TextView(this)
        text.text = "Xin chào Kotlin!"
        text.textSize = 22f

        // Tạo Button
        val button = Button(this)
        button.text = "Nhấn vào tôi"
        button.setOnClickListener {
            text.text = "Bạn vừa bấm nút 🎉"
        }

        // Thêm các view vào layout
        layout.addView(text)
        layout.addView(button)

        // Gán layout cho Activity
        setContentView(layout)
    }
}
```

➡️ Khi chạy:

* Màn hình hiển thị dòng chữ và nút bấm.
* Khi bấm nút, dòng chữ đổi nội dung.

🎯 Đây là **UI hoàn toàn bằng Kotlin code, không XML**.

---

## 🧠 7. `Context` là gì?

`Context` là đối tượng đại diện cho **môi trường đang chạy** (application hoặc activity).
Các class như `TextView`, `Button`, `Layout` đều cần `Context` để biết:

* Lấy tài nguyên (màu, string, style)
* Giao tiếp với hệ thống Android

💬 Trong `Activity`, ta dùng `this` hoặc `this@MainActivity` như `Context`.

---

## 🧩 8. Thực hành – Bài nhỏ “Chào người dùng”

```kotlin
import android.widget.EditText

class MainActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val layout = LinearLayout(this)
        layout.orientation = LinearLayout.VERTICAL
        layout.setPadding(50, 80, 50, 80)

        val input = EditText(this)
        input.hint = "Nhập tên của bạn"

        val button = Button(this)
        button.text = "Chào tôi"

        val output = TextView(this)
        output.textSize = 20f

        button.setOnClickListener {
            val name = input.text.toString()
            output.text = "Xin chào $name 👋"
        }

        layout.addView(input)
        layout.addView(button)
        layout.addView(output)
        setContentView(layout)
    }
}
```

➡️ Kết quả:
Em nhập tên → bấm nút → app hiện lời chào.

---

## 🏠 **Bài tập về nhà – Bài 1**

| Mức độ            | Bài tập                                                                   | Gợi ý                      |
| ----------------- | ------------------------------------------------------------------------- | -------------------------- |
| 🟢 Cơ bản         | Hiển thị “Hello Android” bằng TextView                                    | Dùng `setContentView()`    |
| 🟡 Trung bình     | Tạo Button đổi nội dung TextView                                          | `setOnClickListener`       |
| 🔵 Nâng cao       | Tạo EditText + Button để nhập tên người dùng                              | `EditText.text.toString()` |
| 🟣 Thử thách      | Tạo 2 Button: “Chào” và “Tạm biệt” – mỗi nút đổi TextView khác nhau       | Dùng 2 listener            |
| 🟠 Siêu thử thách | Tạo app mini “Đếm số lần bấm” – mỗi lần bấm tăng biến `count` và hiển thị | Dùng biến `var count = 0`  |

---

## 📚 Tổng kết Bài 1

| Thành phần             | Vai trò                                                       |
| ---------------------- | ------------------------------------------------------------- |
| `Activity`             | Đại diện cho một màn hình ứng dụng                            |
| `View`                 | Thành phần giao diện cơ bản (TextView, Button, EditText, ...) |
| `LinearLayout`         | Bố trí các View theo hàng/cột                                 |
| `Context`              | Môi trường của app (dùng khi tạo View)                        |
| `setContentView()`     | Gán layout hoặc view cho màn hình chính                       |
| `setOnClickListener()` | Bắt sự kiện người dùng bấm nút                                |

---

🎓 **Bài 2 (buổi tới):**

> *Thầy sẽ dạy “Layout nâng cao – LinearLayout, FrameLayout, ConstraintLayout (bằng code)”*
> Ta sẽ học cách căn chỉnh, chia bố cục, margin/padding, và thiết kế UI phức tạp **hoàn toàn bằng Kotlin code**.

---

👉 Em có muốn thầy **tiếp tục luôn Bài 2: Layout nâng cao trong Kotlin UI** không?
Thầy sẽ dạy chậm, từng bước, có ví dụ dễ hiểu — giúp em làm được giao diện đẹp chỉ bằng code Kotlin.
