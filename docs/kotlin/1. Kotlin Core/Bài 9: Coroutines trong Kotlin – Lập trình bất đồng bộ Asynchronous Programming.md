Tốt lắm 👏 — hôm nay chúng ta sang **📘 Bài 9: Coroutines trong Kotlin – Lập trình bất đồng bộ (Asynchronous Programming)**

Đây là **một trong những phần nâng cao và thực tế nhất**, vì khi em lập trình Android hay backend, hầu như mọi tác vụ (API, database, tải hình, xử lý file) đều phải **chạy song song mà không làm “đơ” giao diện**.

---

# 📘 BÀI 9: Coroutines trong Kotlin – Xử lý Bất Đồng Bộ

---

## 🎯 Mục tiêu bài học

Sau bài học này, em sẽ:

1. Hiểu **coroutine là gì** và tại sao cần nó.
2. Biết dùng `launch`, `async`, `delay`, `runBlocking`.
3. Hiểu cách viết **hàm suspend (hàm tạm dừng)**.
4. Viết được ví dụ thực tế: tải dữ liệu song song.

---

## 🧠 1. Coroutines là gì?

> **Coroutine** là một “luồng nhẹ” (lightweight thread) giúp ta chạy các tác vụ song song **mà không chặn main thread**.

💡 Giống như khi app tải dữ liệu từ server — ta muốn giao diện vẫn chạy mượt, không bị đứng.

---

## ⚙️ 2. Import thư viện

Trong dự án Kotlin hoặc Android, cần thêm dependency (nếu chạy ngoài IDE Kotlin Playground):

```gradle
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.0")
```

---

## 🧩 3. Ví dụ đầu tiên với `runBlocking` và `launch`

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    println("Bắt đầu trên thread: ${Thread.currentThread().name}")

    launch {
        delay(1000L)
        println("Chạy trong coroutine 1")
    }

    launch {
        delay(500L)
        println("Chạy trong coroutine 2")
    }

    println("Kết thúc chương trình chính")
}
```

➡️ Output (thứ tự không cố định):

```
Bắt đầu trên thread: main
Kết thúc chương trình chính
Chạy trong coroutine 2
Chạy trong coroutine 1
```

💡 `launch` tạo coroutine mới và chạy song song.
`delay()` **chỉ tạm dừng coroutine đó**, không chặn thread chính.

---

## 🔁 4. `launch` vs `async`

| Hàm      | Dùng khi                               | Trả về gì     |
| -------- | -------------------------------------- | ------------- |
| `launch` | Chạy song song, không cần kết quả      | `Job`         |
| `async`  | Chạy song song, **cần kết quả trả về** | `Deferred<T>` |

---

### 📋 Ví dụ: `async` – chạy song song và lấy kết quả

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val task1 = async {
        delay(1000)
        println("Tải dữ liệu A xong")
        100
    }

    val task2 = async {
        delay(800)
        println("Tải dữ liệu B xong")
        200
    }

    val total = task1.await() + task2.await()
    println("Tổng dữ liệu: $total")
}
```

➡️ Output:

```
Tải dữ liệu B xong
Tải dữ liệu A xong
Tổng dữ liệu: 300
```

💡 `await()` chờ kết quả từ coroutine khác.

---

## 🧮 5. `suspend function` – hàm có thể “tạm dừng”

Một **hàm bình thường** không thể gọi `delay()` hoặc thao tác bất đồng bộ.
Ta phải khai báo `suspend`.

```kotlin
suspend fun fetchData(): String {
    delay(1000L)
    return "Dữ liệu từ server"
}

fun main() = runBlocking {
    println("Đang tải...")
    val result = fetchData()
    println("Kết quả: $result")
}
```

💡 `suspend` chỉ có thể gọi từ **coroutine hoặc hàm suspend khác**.

---

## 🧠 6. Coroutine Scope & Context

Mọi coroutine đều chạy trong **scope** – nơi quản lý vòng đời của chúng.

| Scope            | Dùng khi                                  | Ví dụ                            |
| ---------------- | ----------------------------------------- | -------------------------------- |
| `GlobalScope`    | chạy toàn cục (hiếm dùng trong app)       | `GlobalScope.launch { ... }`     |
| `runBlocking`    | dùng trong hàm main/test                  | `runBlocking { ... }`            |
| `CoroutineScope` | dùng trong Android ViewModel / Repository | `CoroutineScope(Dispatchers.IO)` |

---

## ⚡️ 7. Dispatchers – chọn luồng chạy

| Dispatcher               | Chạy trên thread nào | Dùng khi                |
| ------------------------ | -------------------- | ----------------------- |
| `Dispatchers.Main`       | Main UI thread       | Android UI              |
| `Dispatchers.IO`         | I/O thread           | đọc file, API, database |
| `Dispatchers.Default`    | CPU-bound            | xử lý tính toán nặng    |
| `Dispatchers.Unconfined` | tuỳ vào ngữ cảnh     | ít dùng                 |

📋 Ví dụ:

```kotlin
launch(Dispatchers.IO) {
    val data = fetchData()
    println("Tải xong trong thread: ${Thread.currentThread().name}")
}
```

---

## 🔄 8. `coroutineScope` – tạo phạm vi con

Giúp gom nhiều coroutine lại, nếu một cái lỗi thì cả nhóm dừng.

```kotlin
suspend fun getData() = coroutineScope {
    launch { delay(1000); println("Task 1 xong") }
    launch { delay(500); println("Task 2 xong") }
    println("Đang tải dữ liệu...")
}
```

---

## 🧪 **Thực hành mini**

### 📋 Bài 1 – Chạy 2 tác vụ song song

Viết 2 coroutine:

* 1 in ra “Đang tải dữ liệu người dùng…”
* 1 in ra “Đang tải thông báo…”
  Sau 1 giây in “Tất cả đã xong!”

---

### 📋 Bài 2 – Giả lập API

```kotlin
suspend fun getUser() : String {
    delay(1500)
    return "Người dùng: Mai Lê"
}

suspend fun getPosts() : String {
    delay(1000)
    return "Danh sách bài viết"
}

fun main() = runBlocking {
    val user = async { getUser() }
    val posts = async { getPosts() }

    println(user.await())
    println(posts.await())
    println("Hoàn tất tải dữ liệu ✅")
}
```

---

## 🏠 **Bài tập về nhà – Bài 9**

| Mức độ            | Bài tập                                                                                    | Gợi ý                       |
| ----------------- | ------------------------------------------------------------------------------------------ | --------------------------- |
| 🟢 Cơ bản         | Dùng `launch` để in 3 thông báo cách nhau 1s                                               | Dùng `delay()`              |
| 🟡 Trung bình     | Dùng `async` để tính tổng 2 hàm chạy song song                                             | `async {}` + `await()`      |
| 🔵 Nâng cao       | Viết `suspend fun fetchData(name: String)` → trả về sau 1s                                 | Gọi trong `runBlocking`     |
| 🟣 Thử thách      | Dùng `Dispatchers.IO` để giả lập tải file                                                  | In tên thread               |
| 🟠 Siêu thử thách | Viết chương trình chạy 3 coroutine song song (user, message, product) và in thời gian tổng | Dùng `measureTimeMillis {}` |

---

## 📚 Tổng kết Bài 9

| Kiến thức         | Vai trò                               |
| ----------------- | ------------------------------------- |
| `launch`, `async` | Tạo coroutine mới                     |
| `delay`           | Tạm dừng coroutine, không chặn thread |
| `suspend fun`     | Hàm có thể bị tạm dừng                |
| `Dispatcher`      | Xác định luồng chạy                   |
| `coroutineScope`  | Gom nhiều coroutine an toàn           |

---

🎓 **Bài 10 (buổi tới):**

> *Thầy sẽ dạy “Xử lý JSON, File và Networking trong Kotlin – với thư viện Gson, Retrofit và Coroutines”, để em kết nối được dữ liệu thật từ Internet như một app Android chuyên nghiệp.*

---

👉 Em có muốn thầy **tiếp tục luôn Bài 10: Xử lý JSON & Networking trong Kotlin (với Retrofit + Coroutines)** không?
Bài này cực kỳ thực tế — ta sẽ tạo hàm tải API thật, parse JSON, và hiển thị dữ liệu ra console.
