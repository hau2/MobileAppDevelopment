Chuẩn, mình làm lại **Bài 12** cho “đã cái nư”, tập trung đúng 3 thứ:

> **Publisher – Subscriber – Cancellable**,
> và giải thích kỹ **`.sink`** và **`.store(in: &cancellables)`** là gì, dùng để làm gì, tại sao *bắt buộc* phải dùng.

Thầy sẽ đi theo thứ tự:

1. Khái niệm & mục đích (tại sao xài)
2. Ví dụ nhỏ (Playground style / Controller đơn)
3. Ví dụ UIKit + ViewModel nhỏ dùng Combine

---

# 🧠 TỔNG QUAN NHANH

Trong Combine:

* **Publisher** = thằng PHÁT dữ liệu (stream “chảy ra”).
* **Subscriber** = thằng NGHE dữ liệu (đăng ký nhận stream).
* **Cancellable** = “vé đăng ký”, giữ sống mối quan hệ Publisher–Subscriber.
* **`.sink {}`** = cách đơn giản nhất để *tạo Subscriber*.
* **`.store(in: &cancellables)`** = giữ lại Cancellable để không bị huỷ sớm.

---

# I. PUBLISHER – “THẰNG PHÁT SÓNG”

## 1. Publisher là gì?

Publisher là **object phát ra dữ liệu theo thời gian**, ví dụ:

* phát ra 1 giá trị rồi xong
* phát ra nhiều lần (sự kiện)
* có thể phát error hoặc complete

Nó giống như **Observable** trong RxJS / RxSwift.

**Ví dụ Publisher đơn giản:**

```swift
import Combine

let publisher = Just("Hello Combine")
```

* `Just` là 1 Publisher rất đơn giản → phát đúng **1 giá trị**, rồi complete.
* Kiểu của nó là: `Just<String>`.

**Publisher trong ViewModel thường gặp:**

* `@Published var isLoading: Bool`  → cũng sinh ra Publisher (đằng sau là `Published<Bool>.Publisher`)
* `PassthroughSubject<String, Never>` → Publisher cho event
* `CurrentValueSubject<Int, Never>` → Publisher có “giá trị hiện tại”

---

# II. SUBSCRIBER – “THẰNG ĐĂNG KÝ NGHE”

## 1. Subscriber là gì?

Subscriber là **thằng đăng ký** với Publisher để nhận:

* **Value** (dữ liệu)
* **Completion** (xong / error)

Chúng ta không hay tạo `Subscriber` custom, mà dùng các operator tiện lợi, trong đó **hay dùng nhất là `.sink`**.

---

# III. `.sink` – CÁCH ĐƠN GIẢN NHẤT ĐỂ SUBSCRIBE

## 1. `.sink` là gì?

`.sink` là một hàm extension trên Publisher, giúp:

* tạo **Subscriber**
* đăng ký lắng nghe value từ Publisher
* optional lắng nghe completion

Cú pháp:

```swift
publisher.sink { completion in
    // xử lý completion (.finished / .failure)
} receiveValue: { value in
    // xử lý value
}
```

Hoặc rút gọn (chỉ care value):

```swift
publisher.sink { value in
    // xử lý value
}
```

---

## 2. Ví dụ nhỏ với `Just` + `.sink`

Đoạn này có thể chạy trong Playground hoặc trong `viewDidLoad`:

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    var cancellables = Set<AnyCancellable>()

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .systemBackground

        // 1. Tạo Publisher
        let publisher = Just("Hello Combine")

        // 2. Subscribe bằng sink
        publisher
            .sink { value in
                print("Nhận được:", value)
            }
            .store(in: &cancellables)
    }
}
```

**Giải thích:**

* `Just("Hello Combine")` → Publisher phát 1 string
* `.sink { value in ... }` → tạo 1 subscriber nhận giá trị
* `.store(in: &cancellables)` → lưu lại subscription trong `cancellables`

### Tại sao phải `.store(in: &cancellables)`?

Vì `.sink` trả về 1 **AnyCancellable** (object đại diện 1 subscription).
Nếu em **không giữ reference** tới nó, subscription sẽ bị huỷ ngay → không nhận được gì.

---

# IV. CANCELLABLE – “CÁI VÉ SUBSCRIBE”

## 1. Cancellable là gì?

`AnyCancellable` là object đại diện cho **việc đăng ký** (subscription).
Nó có nhiệm vụ:

* Giữ cho subscription sống
* Khi bị deinit hoặc gọi `.cancel()`, thì hủy việc lắng nghe publisher

### Vì sao phải giữ nó?

Vì Combine dùng ARC. Nếu không giữ tham chiếu, subscription sẽ bị giải phóng, giống như:

```swift
func foo() {
    let s = publisher.sink { ... }  // s biến local
} // ra ngoài hàm, s bị deinit → subscription chết
```

👉 Do đó, trong ViewController hoặc ViewModel, ta thường có:

```swift
var cancellables = Set<AnyCancellable>()
```

Rồi:

```swift
publisher
    .sink { ... }
    .store(in: &cancellables)
```

`.store(in: &cancellables)` đơn giản là:

* cho Cancellable vào Set
* khi ViewController bị deinit → Set bị deinit → toàn bộ subscription bị cancel **đúng chuẩn, không leak**

---

# V. Ví dụ nhỏ: Publisher + Subscriber + Cancellable phối hợp

Làm demo **đếm số lần bấm nút** bằng Combine.

## 1. ViewModel dùng `@Published` (Publisher)

```swift
import Foundation
import Combine

class CounterViewModel {

    /// Khi count thay đổi, Combine sẽ phát (publish) giá trị mới.
    @Published var count: Int = 0

    func increment() {
        count += 1
    }
}
```

* `@Published` biến `count` thành 1 **Publisher**: `Published<Int>.Publisher`.
* Mỗi lần `count` thay đổi → Publisher phát event mới.

---

## 2. ViewController subscribe bằng `.sink` + lưu `AnyCancellable`

```swift
import UIKit
import Combine

class ViewController: UIViewController {

    private let viewModel = CounterViewModel()
    private var cancellables = Set<AnyCancellable>()

    private let countLabel: UILabel = {
        let label = UILabel()
        label.font = .systemFont(ofSize: 30, weight: .bold)
        label.textAlignment = .center
        label.text = "0"
        return label
    }()

    private let button: UIButton = {
        let btn = UIButton(type: .system)
        btn.setTitle("Tăng count", for: .normal)
        btn.titleLabel?.font = .systemFont(ofSize: 20, weight: .medium)
        return btn
    }()

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .systemBackground
        layoutUI()
        bindViewModel()

        button.addTarget(self, action: #selector(handleTap), for: .touchUpInside)
    }

    private func layoutUI() {
        view.addSubview(countLabel)
        view.addSubview(button)

        countLabel.translatesAutoresizingMaskIntoConstraints = false
        button.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            countLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            countLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor, constant: -40),

            button.topAnchor.constraint(equalTo: countLabel.bottomAnchor, constant: 20),
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }

    private func bindViewModel() {
        // viewModel.$count là 1 Publisher
        viewModel.$count
            .sink { [weak self] newCount in
                // Subscriber – nhận dữ liệu mỗi lần count thay đổi
                self?.countLabel.text = "\(newCount)"
            }
            .store(in: &cancellables)  // giữ subscription
    }

    @objc private func handleTap() {
        viewModel.increment()
    }
}
```

### Dòng quan trọng:

```swift
viewModel.$count
    .sink { [weak self] newCount in
        self?.countLabel.text = "\(newCount)"
    }
    .store(in: &cancellables)
```

* `viewModel.$count` → Publisher (phát ra Int mỗi lần count thay đổi)
* `.sink { ... }` → tạo Subscriber, đăng ký nhận giá trị mới
* `.store(in: &cancellables)` → giữ subscription sống cùng với ViewController

---

# VI. TÓM TẮT BÀI 12 (phiên bản chi tiết)

### 1. Publisher

* Thằng phát dữ liệu
* Ví dụ: `Just`, `@Published`, `PassthroughSubject`, `CurrentValueSubject`
* Dùng trong ViewModel để **expose state** ra View

### 2. Subscriber

* Thằng nhận dữ liệu
* Thầy cho dùng `.sink` là cách dễ nhất tạo Subscriber
* ViewController subscribe để update UI

### 3. Cancellable

* “Vé đăng ký” giữa Publisher – Subscriber
* Phải giữ lại (thường trong `var cancellables = Set<AnyCancellable>()`)
* `.store(in: &cancellables)` = cách chuẩn để giữ tất cả subscription và tự hủy khi VC deinit

---

Nếu em muốn, bài sau thầy sẽ:

* Cho ví dụ **dùng `PassthroughSubject` + Combine**
* Hoặc **bắt đầu gắn Combine vào Repository + API thật** (FakeStoreAPI)

Giờ em có thể chọn:

👉 “**Thầy cho ví dụ về PassthroughSubject**”
hoặc
👉 “**Qua bài 13 – Repository + Combine**”
