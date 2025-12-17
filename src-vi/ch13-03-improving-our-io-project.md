## Cải thiện Dự án I/O của Chúng ta

Với kiến thức mới về iterators, chúng ta có thể cải thiện dự án I/O
trong Chương 12 bằng cách sử dụng iterators để làm cho các chỗ trong
code rõ ràng và ngắn gọn hơn. Hãy xem cách iterators có thể cải thiện
cách triển khai hàm `Config::build` và hàm `search`.

### Loại bỏ việc `clone` bằng Iterator

Trong Listing 12-6, chúng ta đã thêm code lấy một slice của các giá trị
`String` và tạo một instance của struct `Config` bằng cách index vào
slice và clone các giá trị, cho phép struct `Config` sở hữu các giá trị
này. Trong Listing 13-17, chúng ta đã tái hiện cách triển khai hàm
`Config::build` như trong Listing 12-23:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/lib.rs:ch13}}
```

<span class="caption">Listing 13-17: Tái hiện hàm `Config::build`
từ Listing 12-23</span>

Lúc đó, chúng ta nói không cần lo lắng về các lần gọi `clone` kém hiệu
quả vì sẽ loại bỏ chúng trong tương lai. Giờ thì thời điểm đó đã đến!

Chúng ta cần `clone` ở đây vì chúng ta có một slice với các phần tử
`String` trong tham số `args`, nhưng hàm `build` không sở hữu `args`.
Để trả quyền sở hữu một instance của `Config`, chúng ta phải clone
các giá trị từ các trường `query` và `filename` của `Config` để
instance `Config` có thể sở hữu các giá trị của nó.

Với kiến thức mới về iterators, chúng ta có thể thay đổi hàm `build`
để nhận quyền sở hữu của một iterator làm tham số thay vì mượn một slice.
Chúng ta sẽ dùng chức năng của iterator thay vì code kiểm tra độ dài
slice và index vào các vị trí cụ thể. Điều này sẽ làm rõ việc
hàm `Config::build` đang làm gì vì iterator sẽ truy cập các giá trị.

Khi `Config::build` nhận quyền sở hữu iterator và ngừng dùng các
phép toán indexing mượn, chúng ta có thể di chuyển các giá trị `String`
từ iterator vào `Config` thay vì gọi `clone` và tạo một cấp phát mới.

#### Sử dụng Iterator trả về trực tiếp

Mở file *src/main.rs* của dự án I/O của bạn, file này sẽ trông như sau:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Chúng ta sẽ bắt đầu bằng cách thay đổi phần đầu của hàm `main`
mà chúng ta có trong Listing 12-24 thành code trong Listing 13-18,
lần này sử dụng iterator. Code này sẽ chưa biên dịch được cho đến
khi chúng ta cập nhật hàm `Config::build`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

Hàm `env::args` trả về một iterator! Thay vì thu thập các giá trị
của iterator vào một vector rồi truyền một slice vào `Config::build`,
bây giờ chúng ta truyền quyền sở hữu của iterator trả về từ `env::args`
trực tiếp vào `Config::build`.

Tiếp theo, chúng ta cần cập nhật định nghĩa của `Config::build`.
Trong file *src/lib.rs* của dự án I/O, hãy thay đổi chữ ký của
`Config::build` như trong Listing 13-19. Code này vẫn chưa biên dịch được
vì chúng ta cần cập nhật phần thân hàm.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/lib.rs:here}}
```

Tài liệu của thư viện chuẩn cho hàm `env::args` cho thấy kiểu của
iterator mà nó trả về là `std::env::Args`, và kiểu này triển khai
trait `Iterator` và trả về các giá trị `String`.

Chúng ta đã cập nhật chữ ký của hàm `Config::build` sao cho tham số
`args` có kiểu tổng quát với ràng buộc trait
`impl Iterator<Item = String>` thay vì `&[String]`. Cách sử dụng
cú pháp `impl Trait` mà chúng ta đã thảo luận trong phần
[“Traits as Parameters”][impl-trait]<!-- ignore --> của Chương 10
có nghĩa là `args` có thể là bất kỳ kiểu nào triển khai trait `Iterator`
và trả về các phần tử kiểu `String`.

Vì chúng ta đang nhận quyền sở hữu `args` và sẽ thay đổi `args` bằng
cách lặp qua nó, chúng ta có thể thêm từ khóa `mut` vào khai báo
tham số `args` để làm nó có thể thay đổi được.

#### Sử dụng các phương thức của trait `Iterator` thay vì đánh chỉ số

Tiếp theo, chúng ta sẽ sửa phần thân của `Config::build`. Vì `args`
triển khai trait `Iterator`, chúng ta biết có thể gọi phương thức
`next` trên nó! Listing 13-20 cập nhật code từ Listing 12-23
sử dụng phương thức `next`:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/lib.rs:here}}
```

Nhớ rằng giá trị đầu tiên trong giá trị trả về của `env::args` là tên
chương trình. Chúng ta muốn bỏ qua giá trị này và lấy giá trị tiếp theo,
vì vậy trước tiên chúng ta gọi `next` nhưng không làm gì với giá trị trả về.
Tiếp theo, chúng ta gọi `next` để lấy giá trị mà chúng ta muốn gán vào
trường `query` của `Config`. Nếu `next` trả về `Some`, chúng ta dùng
`match` để lấy giá trị. Nếu trả về `None`, nghĩa là không đủ tham số,
và chúng ta trả về sớm với giá trị `Err`. Chúng ta làm tương tự cho giá
trị `filename`.

### Làm code rõ ràng hơn với các iterator adaptor

Chúng ta cũng có thể tận dụng iterator trong hàm `search` của dự án
I/O, được tái hiện ở đây trong Listing 13-21 như trong Listing 12-19:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

Chúng ta có thể viết đoạn code này một cách ngắn gọn hơn bằng cách dùng
các phương thức iterator adaptor. Cách làm này cũng giúp chúng ta tránh
phải tạo một vector `results` trung gian có thể thay đổi. Phong cách lập
trình hàm hướng (functional programming) ưu tiên giảm thiểu trạng thái
có thể thay đổi để làm code rõ ràng hơn. Việc loại bỏ trạng thái có thể
thay đổi cũng có thể cho phép nâng cấp trong tương lai để thực hiện
tìm kiếm song song, vì chúng ta sẽ không phải quản lý truy cập đồng
thời đến vector `results`. Listing 13-22 minh họa thay đổi này:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

Hãy nhớ rằng mục đích của hàm `search` là trả về tất cả các dòng trong
`contents` chứa `query`. Tương tự như ví dụ `filter` trong Listing 13-16,
đoạn code này sử dụng adaptor `filter` để giữ lại chỉ những dòng mà
`line.contains(query)` trả về `true`. Sau đó, chúng ta thu thập các dòng
phù hợp vào một vector khác bằng `collect`. Đơn giản hơn nhiều! Bạn có
thể thực hiện thay đổi tương tự để dùng phương thức iterator trong hàm
`search_case_insensitive`.

### Chọn giữa vòng lặp và iterator

Câu hỏi tiếp theo là nên chọn phong cách nào trong code của bạn và tại sao:
cài đặt ban đầu trong Listing 13-21 hay phiên bản dùng iterator trong Listing 13-22.
Hầu hết lập trình viên Rust thích dùng phong cách iterator. Ban đầu có thể
khó làm quen, nhưng khi bạn hiểu các iterator adaptor và chức năng của chúng,
iterator thường dễ hiểu hơn. Thay vì loay hoay với các vòng lặp và tạo
vector mới, code tập trung vào mục tiêu cao cấp của vòng lặp. Điều này
trừu tượng hóa một phần code thông thường, giúp dễ nhìn thấy các khái niệm
đặc trưng cho đoạn code này, như điều kiện lọc mà mỗi phần tử trong iterator
phải thỏa mãn.

Nhưng liệu hai cài đặt này có thực sự tương đương? Giả định trực quan
có thể là vòng lặp mức thấp sẽ nhanh hơn. Hãy nói về hiệu năng.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
