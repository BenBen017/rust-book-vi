## Tắt và Dọn dẹp Một cách Êm ái

Code trong Listing 20-20 đang xử lý các request một cách bất đồng bộ thông qua
việc sử dụng thread pool, đúng như chúng ta mong muốn. Chúng ta nhận một số
cảnh báo về các trường `workers`, `id`, và `thread` mà chúng ta không sử dụng
trực tiếp, nhắc nhở rằng chúng ta chưa dọn dẹp gì cả. Khi sử dụng phương pháp 
ít tinh tế hơn <span class="keystroke">ctrl-c</span> để dừng thread chính, tất 
cả các thread khác cũng bị dừng ngay lập tức, ngay cả khi chúng đang xử lý một 
request.

Tiếp theo, chúng ta sẽ triển khai trait `Drop` để gọi `join` trên từng thread trong pool 
để chúng có thể hoàn tất các request mà chúng đang xử lý trước khi đóng. Sau đó, 
chúng ta sẽ triển khai một cách để báo cho các thread biết rằng chúng nên ngừng 
nhận request mới và tắt. Để xem code này hoạt động, chúng ta sẽ chỉnh sửa server 
chỉ chấp nhận hai request trước khi tắt thread pool một cách êm ái.

### Triển khai trait `Drop` trên `ThreadPool`

Hãy bắt đầu với việc triển khai `Drop` cho thread pool. Khi pool bị drop, tất cả 
các thread của chúng ta nên join để đảm bảo chúng hoàn thành công việc. Listing 20-22 
cho thấy một lần thử đầu tiên của việc triển khai `Drop`; code này vẫn chưa chạy được.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-22/src/lib.rs:here}}
```

<span class="caption">Listing 20-22: Gọi `join` trên từng thread khi thread pool
ra khỏi phạm vi</span>

Trước tiên, chúng ta lặp qua từng `worker` trong thread pool. Chúng ta sử dụng `&mut` 
vì `self` là một tham chiếu có thể thay đổi, và chúng ta cũng cần khả năng thay đổi 
`worker`. Với mỗi worker, chúng ta in ra một thông báo nói rằng worker cụ thể này 
đang tắt, sau đó gọi `join` trên thread của worker đó. Nếu việc gọi `join` thất bại, 
chúng ta sử dụng `unwrap` để Rust panic và thực hiện một việc tắt không êm ái.

Dưới đây là lỗi mà chúng ta nhận được khi biên dịch code này:

```console
{{#include ../listings/ch20-web-server/listing-20-22/output.txt}}
```

Lỗi này cho chúng ta biết rằng không thể gọi `join` vì chúng ta chỉ có một 
tham chiếu thay đổi (`mutable borrow`) tới mỗi `worker`, trong khi `join` 
yêu cầu quyền sở hữu (`ownership`) của đối số. Để giải quyết vấn đề này, 
chúng ta cần di chuyển thread ra khỏi instance `Worker` đang sở hữu `thread` 
để `join` có thể tiêu thụ thread đó. Chúng ta đã làm điều này trong Listing 17-15: 
nếu `Worker` giữ một `Option<thread::JoinHandle<()>>`, chúng ta có thể gọi 
phương thức `take` trên `Option` để di chuyển giá trị ra khỏi biến `Some` 
và để lại `None` ở vị trí đó. Nói cách khác, một `Worker` đang chạy sẽ có 
biến `Some` trong `thread`, và khi muốn dọn dẹp một `Worker`, chúng ta sẽ 
thay `Some` bằng `None` để `Worker` không còn thread nào để chạy.

Vì vậy, chúng ta biết rằng cần cập nhật định nghĩa của `Worker` như sau:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/no-listing-04-update-worker-definition/src/lib.rs:here}}
```

Bây giờ chúng ta hãy dựa vào compiler để tìm các chỗ khác cần thay đổi. 
Khi kiểm tra mã này, chúng ta nhận được hai lỗi:

```console
{{#include ../listings/ch20-web-server/no-listing-04-update-worker-definition/output.txt}}
```

Hãy giải quyết lỗi thứ hai, liên quan đến mã ở cuối `Worker::new`; chúng ta cần
bao `thread` trong `Some` khi tạo một `Worker` mới. Thực hiện các thay đổi sau
để sửa lỗi này:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/no-listing-05-fix-worker-new/src/lib.rs:here}}
```

Lỗi đầu tiên nằm trong phần triển khai `Drop`. Chúng ta đã đề cập trước đó rằng 
chúng ta dự định gọi `take` trên giá trị `Option` để di chuyển `thread` ra khỏi 
`worker`. Các thay đổi sau sẽ thực hiện điều đó:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/no-listing-06-fix-threadpool-drop/src/lib.rs:here}}
```

Như đã thảo luận trong Chương 17, phương thức `take` trên `Option` sẽ lấy giá trị 
`Some` ra và để lại `None` ở chỗ đó. Chúng ta dùng `if let` để phá cấu trúc 
`Some` và lấy ra `thread`; sau đó gọi `join` trên `thread` đó. Nếu `thread` của 
một worker đã là `None`, nghĩa là worker đó đã được dọn dẹp trước, nên trong 
trường hợp này sẽ không có gì xảy ra.

### Thông báo cho các Thread dừng lắng nghe Job

Với tất cả những thay đổi trên, code của chúng ta đã biên dịch mà không có cảnh báo. 
Tuy nhiên, vấn đề là code hiện tại vẫn chưa hoạt động đúng như mong muốn. 
Điểm then chốt nằm ở logic trong các closure mà các thread của `Worker` đang chạy: 
hiện tại chúng ta chỉ gọi `join`, nhưng điều này sẽ không tắt các thread 
bởi vì chúng `loop` vô tận để chờ job. Nếu thử `drop` `ThreadPool` với 
cách triển khai hiện tại, thread chính sẽ bị block vô thời hạn chờ thread đầu tiên kết thúc.

Để khắc phục vấn đề này, chúng ta cần thay đổi trong triển khai `drop` của 
`ThreadPool` và thay đổi trong vòng lặp của `Worker`.

Trước tiên, chúng ta sẽ thay đổi `drop` của `ThreadPool` để chủ động 
`drop` `sender` trước khi chờ các thread hoàn tất. Listing 20-23 minh họa 
các thay đổi trong `ThreadPool` để chủ động `drop` `sender`. Chúng ta sử dụng 
cùng kỹ thuật `Option` và `take` như với thread để có thể di chuyển `sender` 
ra khỏi `ThreadPool`:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-23/src/lib.rs:here}}
```

Việc `drop` `sender` sẽ đóng kênh, báo hiệu rằng sẽ không còn thông điệp nào 
được gửi nữa. Khi điều này xảy ra, tất cả các lần gọi `recv` trong vòng lặp 
vô hạn của các worker sẽ trả về lỗi. Trong Listing 20-24, chúng ta thay đổi 
vòng lặp của `Worker` để thoát vòng lặp một cách an toàn khi nhận lỗi, 
nghĩa là các thread sẽ kết thúc khi `ThreadPool` gọi `join` trên chúng.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-24/src/lib.rs:here}}
```

<span class="caption">Listing 20-24: Thoát vòng lặp một cách rõ ràng khi `recv` trả về lỗi</span>

Để thấy đoạn code này hoạt động, hãy chỉnh sửa `main` sao cho server chỉ nhận tối đa hai request 
trước khi đóng server một cách gọn gàng, như minh họa trong Listing 20-25.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/listing-20-25/src/main.rs:here}}
```

<span class="caption">Listing 20-25: Tắt server sau khi phục vụ hai request bằng cách thoát khỏi vòng lặp</span>

Trong thực tế, bạn sẽ không muốn một web server tắt sau khi 
chỉ phục vụ hai request. Đoạn code này chỉ nhằm minh họa 
rằng cơ chế tắt server gọn gàng và dọn dẹp tài nguyên đang hoạt động.

Phương thức `take` được định nghĩa trong trait `Iterator` 
và giới hạn số lần lặp tối đa là hai. `ThreadPool` sẽ ra khỏi phạm vi 
khi kết thúc `main`, và implementation của `drop` sẽ được gọi.

Chạy server bằng `cargo run`, và gửi ba request. Request thứ ba sẽ gặp lỗi, 
và trên terminal bạn sẽ thấy đầu ra tương tự như sau:

<!-- manual-regeneration
cd listings/ch20-web-server/listing-20-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Bạn có thể thấy thứ tự in ra của các worker và các message sẽ khác nhau. 
Chúng ta có thể hiểu cách hoạt động của đoạn code này qua các thông báo: 
worker 0 và 3 nhận hai request đầu tiên. Server ngừng chấp nhận kết nối 
sau kết nối thứ hai, và implementation của `Drop` trên `ThreadPool` 
bắt đầu thực thi trước khi worker 3 kịp bắt đầu công việc của nó. 
Việc drop `sender` sẽ ngắt kết nối tất cả các worker và báo cho chúng tắt. 
Mỗi worker sẽ in ra một thông báo khi chúng bị ngắt kết nối, 
sau đó thread pool gọi `join` để chờ các thread worker kết thúc.

Lưu ý một điểm thú vị trong lần thực thi này: `ThreadPool` drop `sender`, 
và trước khi bất kỳ worker nào nhận được lỗi, chúng ta đã thử join worker 0. 
Worker 0 chưa nhận lỗi từ `recv`, nên thread chính bị chặn chờ worker 0 hoàn thành. 
Trong lúc đó, worker 3 nhận được một job và sau đó tất cả các thread nhận lỗi. 
Khi worker 0 kết thúc, thread chính chờ các worker còn lại hoàn thành. 
Lúc đó, tất cả đã thoát khỏi vòng lặp và dừng lại.

Chúc mừng! Chúng ta đã hoàn thành dự án; chúng ta có một web server 
cơ bản sử dụng thread pool để phản hồi bất đồng bộ. 
Chúng ta có thể thực hiện tắt server một cách gọn gàng, dọn dẹp tất cả các thread trong pool.

Dưới đây là toàn bộ code để tham khảo:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/no-listing-07-final-code/src/main.rs}}
```

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-07-final-code/src/lib.rs}}
```

Chúng ta vẫn có thể làm nhiều hơn nữa! Nếu bạn muốn tiếp tục cải tiến dự án này, đây là một số ý tưởng:

* Thêm nhiều tài liệu hơn cho `ThreadPool` và các phương thức public của nó.
* Thêm các bài test cho chức năng của thư viện.
* Thay các lần gọi `unwrap` bằng cơ chế xử lý lỗi chắc chắn hơn.
* Sử dụng `ThreadPool` để thực hiện một nhiệm vụ khác ngoài việc phục vụ web request.
* Tìm một crate thread pool trên [crates.io](https://crates.io/) và triển khai một web server tương tự bằng crate đó. Sau đó so sánh API và độ ổn định với thread pool mà chúng ta tự implement.

## Tóm tắt

Xuất sắc! Bạn đã đi đến cuối cuốn sách! Chúng tôi cảm ơn bạn 
đã đồng hành cùng chúng tôi trong chuyến tham quan Rust. 
Bây giờ bạn đã sẵn sàng để triển khai các dự án Rust của riêng mình 
và hỗ trợ các dự án của người khác. Hãy nhớ rằng cộng đồng Rustacean 
luôn chào đón và sẵn sàng giúp bạn vượt qua bất kỳ thử thách nào trên hành trình học Rust của bạn.
