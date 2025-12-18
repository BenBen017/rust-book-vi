## Shared-State Concurrency

Concurrency dựa trên việc truyền thông điệp (message passing) là một cách tốt để
xử lý đồng thời, nhưng đó không phải là cách duy nhất. Một phương pháp khác là
cho nhiều thread cùng truy cập vào một vùng dữ liệu được chia sẻ. Hãy xem lại
một phần trong khẩu hiệu của tài liệu ngôn ngữ Go: “do not communicate by sharing
memory.”

Vậy việc giao tiếp bằng cách chia sẻ bộ nhớ sẽ trông như thế nào? Ngoài ra, tại
sao những người ủng hộ message passing lại cảnh báo không nên sử dụng việc chia
sẻ bộ nhớ?

Ở một khía cạnh nào đó, channel trong bất kỳ ngôn ngữ lập trình nào cũng tương tự
như single ownership, bởi vì một khi bạn đã chuyển một giá trị qua channel, bạn
không nên sử dụng lại giá trị đó nữa. Concurrency dựa trên bộ nhớ dùng chung
(shared memory concurrency) thì giống với multiple ownership: nhiều thread có
thể truy cập cùng một vị trí bộ nhớ tại cùng một thời điểm. Như bạn đã thấy trong
Chương 15, nơi smart pointer cho phép multiple ownership, thì multiple ownership
có thể làm tăng độ phức tạp vì các “chủ sở hữu” khác nhau này cần được quản lý.
Hệ thống kiểu dữ liệu và các quy tắc ownership của Rust hỗ trợ rất nhiều trong
việc quản lý này một cách chính xác. Để lấy ví dụ, chúng ta hãy cùng xem mutex,
một trong những primitive đồng thời phổ biến nhất cho bộ nhớ dùng chung.

### Sử Dụng Mutex Để Cho Phép Truy Cập Dữ Liệu Từ Một Thread Tại Một Thời Điểm

*Mutex* là từ viết tắt của *mutual exclusion* (loại trừ lẫn nhau), nghĩa là mutex
chỉ cho phép **một thread** truy cập vào một phần dữ liệu tại bất kỳ thời điểm
nào. Để truy cập dữ liệu bên trong mutex, một thread trước hết phải báo hiệu
rằng nó muốn truy cập bằng cách yêu cầu chiếm giữ (*acquire*) *lock* của mutex.
Lock là một cấu trúc dữ liệu thuộc về mutex, dùng để theo dõi thread nào hiện
đang có quyền truy cập độc quyền vào dữ liệu. Vì vậy, mutex được mô tả là đang
*canh giữ* (guarding) dữ liệu mà nó nắm giữ thông qua cơ chế khóa.

Mutex nổi tiếng là khó sử dụng vì bạn phải ghi nhớ hai quy tắc:

* Bạn phải cố gắng acquire lock trước khi sử dụng dữ liệu.
* Khi đã sử dụng xong dữ liệu mà mutex đang canh giữ, bạn phải mở khóa (unlock)
  dữ liệu đó để các thread khác có thể acquire lock.

Để lấy một phép ẩn dụ trong đời thực cho mutex, hãy tưởng tượng một buổi thảo
luận tại hội nghị chỉ có một chiếc micro. Trước khi một diễn giả có thể phát
biểu, họ phải xin hoặc ra hiệu rằng họ muốn sử dụng micro. Khi đã có micro, họ
có thể nói bao lâu tùy thích, rồi chuyển micro cho diễn giả tiếp theo đang muốn
phát biểu. Nếu một diễn giả quên không chuyển micro sau khi nói xong, sẽ không ai
khác có thể nói được. Nếu việc quản lý chiếc micro dùng chung này xảy ra sai
sót, buổi thảo luận sẽ không thể diễn ra như dự kiến!

Việc quản lý mutex có thể cực kỳ phức tạp để làm cho đúng, đó là lý do vì sao rất
nhiều người hứng thú với channel. Tuy nhiên, nhờ hệ thống kiểu dữ liệu và các
quy tắc ownership của Rust, bạn không thể khóa và mở khóa sai được.

#### API của `Mutex<T>`

Để minh họa cách sử dụng mutex, chúng ta hãy bắt đầu bằng việc dùng mutex trong
bối cảnh single-thread, như được minh họa trong Listing 16-12:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

<span class="caption">Listing 16-12: Khám phá API của `Mutex<T>` trong bối cảnh
single-thread để đơn giản hóa</span>

Cũng như nhiều kiểu dữ liệu khác, chúng ta tạo một `Mutex<T>` bằng cách sử dụng
hàm liên kết (associated function) `new`. Để truy cập dữ liệu bên trong mutex,
chúng ta dùng phương thức `lock` để acquire lock. Lời gọi này sẽ chặn (block)
thread hiện tại, khiến nó không thể làm việc gì cho đến khi tới lượt nó nắm giữ
lock.

Lời gọi `lock` sẽ thất bại nếu một thread khác đang giữ lock bị panic. Trong
trường hợp đó, sẽ không ai có thể lấy được lock nữa, vì vậy chúng ta đã chọn
cách gọi `unwrap` và để thread hiện tại panic nếu rơi vào tình huống này.

Sau khi đã acquire được lock, chúng ta có thể coi giá trị trả về—trong trường
hợp này được đặt tên là `num`—như một mutable reference tới dữ liệu bên trong.
Hệ thống kiểu dữ liệu đảm bảo rằng chúng ta phải acquire lock trước khi sử dụng
giá trị bên trong `m`. Kiểu của `m` là `Mutex<i32>`, chứ không phải `i32`, nên
chúng ta *bắt buộc* phải gọi `lock` thì mới có thể sử dụng giá trị `i32`. Chúng
ta không thể quên bước này; hệ thống kiểu dữ liệu sẽ không cho phép truy cập vào
`i32` bên trong nếu không làm như vậy.

Như bạn có thể đoán, `Mutex<T>` là một smart pointer. Chính xác hơn, lời gọi
`lock` *trả về* một smart pointer có tên là `MutexGuard`, được bọc trong một
`LockResult` mà chúng ta đã xử lý bằng cách gọi `unwrap`. Smart pointer
`MutexGuard` triển khai trait `Deref` để trỏ tới dữ liệu bên trong; đồng thời nó
cũng có một implementation của `Drop` để tự động giải phóng lock khi
`MutexGuard` đi ra khỏi scope, điều này xảy ra ở cuối scope bên trong. Kết quả
là chúng ta không có nguy cơ quên giải phóng lock và chặn mutex không cho các
thread khác sử dụng, bởi vì việc mở khóa diễn ra một cách tự động.

Sau khi lock được drop, chúng ta có thể in giá trị của mutex ra và thấy rằng
chúng ta đã thay đổi được giá trị `i32` bên trong thành 6.

#### Chia Sẻ `Mutex<T>` Giữa Nhiều Thread

Bây giờ, hãy thử chia sẻ một giá trị giữa nhiều thread bằng cách sử dụng
`Mutex<T>`. Chúng ta sẽ tạo ra 10 thread và để mỗi thread tăng giá trị của một
counter lên 1, để counter đi từ 0 lên 10. Ví dụ tiếp theo trong Listing 16-13
sẽ gây ra một lỗi biên dịch, và chúng ta sẽ dùng lỗi đó để tìm hiểu thêm về cách
sử dụng `Mutex<T>` cũng như cách Rust giúp chúng ta dùng nó một cách chính xác.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

<span class="caption">Listing 16-13: Mười thread, mỗi thread tăng một counter
được canh giữ bởi `Mutex<T>`</span>

Chúng ta tạo một biến `counter` để chứa một giá trị `i32` bên trong một
`Mutex<T>`, giống như đã làm trong Listing 16-12. Tiếp theo, chúng ta tạo ra
10 thread bằng cách lặp qua một khoảng số. Chúng ta sử dụng `thread::spawn` và
truyền cho tất cả các thread cùng một closure: closure này move `counter` vào
trong thread, acquire lock trên `Mutex<T>` bằng cách gọi phương thức `lock`,
sau đó cộng thêm 1 vào giá trị bên trong mutex. Khi một thread chạy xong
closure của nó, `num` sẽ đi ra khỏi scope và giải phóng lock để thread khác có
thể acquire nó.

Trong thread chính, chúng ta thu thập tất cả các join handle. Sau đó, giống như
đã làm trong Listing 16-2, chúng ta gọi `join` trên từng handle để đảm bảo tất
cả các thread đều kết thúc. Tại thời điểm đó, thread chính sẽ acquire lock và
in ra kết quả của chương trình này.

Chúng ta đã gợi ý rằng ví dụ này sẽ không biên dịch được. Bây giờ hãy cùng tìm
hiểu lý do vì sao!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Thông báo lỗi cho biết rằng giá trị `counter` đã bị move ở lần lặp trước đó của
vòng lặp. Rust đang nói với chúng ta rằng chúng ta không thể move quyền sở hữu
của `counter` vào nhiều thread khác nhau. Hãy sửa lỗi biên dịch này bằng một
phương pháp multiple ownership mà chúng ta đã thảo luận trong Chương 15.

#### Multiple Ownership với Nhiều Thread

Trong Chương 15, chúng ta đã tạo cho một giá trị nhiều owner bằng cách sử dụng
smart pointer `Rc<T>` để tạo ra một giá trị được đếm tham chiếu (reference
counted). Hãy làm điều tương tự ở đây và xem điều gì sẽ xảy ra. Chúng ta sẽ bọc
`Mutex<T>` bên trong `Rc<T>` trong Listing 16-14 và clone `Rc<T>` trước khi move
quyền sở hữu vào thread.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

<span class="caption">Listing 16-14: Thử sử dụng `Rc<T>` để cho phép
nhiều thread cùng sở hữu `Mutex<T>`</span>

Một lần nữa, chúng ta biên dịch và lại nhận được… những lỗi khác! Trình biên
dịch đang dạy chúng ta rất nhiều điều.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Wow, thông báo lỗi này thật là dài dòng! Đây là phần quan trọng cần tập trung:
`` `Rc<Mutex<i32>>` cannot be sent between threads safely ``. Trình biên dịch
cũng cho chúng ta biết lý do vì sao: ``the trait `Send` is not implemented for
`Rc<Mutex<i32>>` ``. Chúng ta sẽ nói về `Send` trong phần tiếp theo: đây là một
trong những trait đảm bảo rằng các kiểu dữ liệu chúng ta dùng với thread là
được thiết kế để sử dụng trong các tình huống đồng thời.

Đáng tiếc là `Rc<T>` không an toàn để chia sẻ giữa các thread. Khi `Rc<T>` quản
lý bộ đếm tham chiếu (reference count), nó tăng bộ đếm mỗi khi `clone` được gọi
và giảm bộ đếm khi mỗi bản clone bị drop. Tuy nhiên, nó không sử dụng bất kỳ
primitive đồng thời nào để đảm bảo rằng các thay đổi đối với bộ đếm không bị
ngắt quãng bởi thread khác. Điều này có thể dẫn đến bộ đếm sai—những lỗi tinh vi
mà sau đó có thể gây ra rò rỉ bộ nhớ hoặc khiến một giá trị bị drop trước khi
chúng ta dùng xong nó. Thứ chúng ta cần là một kiểu dữ liệu giống hệt `Rc<T>`
nhưng thực hiện các thay đổi đối với bộ đếm tham chiếu theo cách an toàn với
thread.

#### Đếm Tham Chiếu Nguyên Tử với `Arc<T>`

May mắn thay, `Arc<T>` *là* một kiểu dữ liệu giống như `Rc<T>` nhưng an toàn để
sử dụng trong các tình huống đồng thời. Chữ *a* là viết tắt của *atomic*, nghĩa
là đây là một kiểu *được đếm tham chiếu một cách nguyên tử* (atomically
reference counted). Atomics là một loại primitive đồng thời bổ sung mà chúng ta
sẽ không đi sâu vào chi tiết ở đây: hãy xem tài liệu thư viện chuẩn cho
[`std::sync::atomic`][atomic]<!-- ignore --> để biết thêm chi tiết. Ở thời điểm
này, bạn chỉ cần biết rằng atomics hoạt động giống như các kiểu primitive
thông thường nhưng an toàn để chia sẻ giữa các thread.

Lúc này bạn có thể thắc mắc vì sao tất cả các kiểu primitive không phải đều là
atomic, và vì sao các kiểu trong thư viện chuẩn không được triển khai để dùng
`Arc<T>` theo mặc định. Lý do là vì tính an toàn với thread đi kèm với một chi
phí hiệu năng, và bạn chỉ nên trả chi phí đó khi thực sự cần thiết. Nếu bạn chỉ
thực hiện các thao tác trên giá trị trong phạm vi một thread duy nhất, chương
trình của bạn có thể chạy nhanh hơn nếu không phải áp đặt các đảm bảo mà
atomics cung cấp.

Hãy quay lại ví dụ của chúng ta: `Arc<T>` và `Rc<T>` có cùng API, vì vậy chúng ta
sửa chương trình bằng cách thay đổi dòng `use`, lời gọi `new`, và lời gọi
`clone`. Đoạn mã trong Listing 16-15 cuối cùng cũng sẽ biên dịch và chạy được:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

<span class="caption">Listing 16-15: Sử dụng `Arc<T>` để bọc `Mutex<T>`
nhằm có thể chia sẻ quyền sở hữu giữa nhiều thread</span>

Đoạn mã này sẽ in ra kết quả sau:

<!-- Không trích xuất output vì những thay đổi trong output này không đáng kể;
các thay đổi nhiều khả năng là do các thread chạy khác nhau, thay vì do
những thay đổi trong trình biên dịch -->

```text
Result: 10
```

Chúng ta đã làm được rồi! Chúng ta đã đếm từ 0 lên 10 — nghe có vẻ không mấy ấn
tượng, nhưng nó đã dạy cho chúng ta rất nhiều điều về `Mutex<T>` và tính an toàn
với thread. Bạn cũng có thể sử dụng cấu trúc của chương trình này để thực hiện
những thao tác phức tạp hơn nhiều so với việc chỉ tăng một counter. Với chiến
lược này, bạn có thể chia một phép tính thành các phần độc lập, phân chia các
phần đó cho nhiều thread, và sau đó sử dụng `Mutex<T>` để mỗi thread cập nhật
kết quả cuối cùng bằng phần việc của mình.

Lưu ý rằng nếu bạn chỉ thực hiện các phép toán số học đơn giản, thì có những
kiểu dữ liệu đơn giản hơn `Mutex<T>` được cung cấp bởi module
[`std::sync::atomic` của thư viện chuẩn][atomic]<!-- ignore -->. Những kiểu dữ
liệu này cung cấp khả năng truy cập đồng thời, an toàn và nguyên tử (atomic)
đối với các kiểu primitive. Chúng tôi chọn sử dụng `Mutex<T>` với một kiểu
primitive trong ví dụ này để có thể tập trung vào cách `Mutex<T>` hoạt động.

### Sự Tương Đồng Giữa `RefCell<T>`/`Rc<T>` và `Mutex<T>`/`Arc<T>`

Bạn có thể đã nhận ra rằng `counter` là immutable nhưng chúng ta vẫn có thể lấy
được một mutable reference tới giá trị bên trong nó; điều này có nghĩa là
`Mutex<T>` cung cấp *interior mutability*, tương tự như họ kiểu `Cell`. Cũng
giống như cách chúng ta đã dùng `RefCell<T>` trong Chương 15 để cho phép mutate
nội dung bên trong một `Rc<T>`, thì ở đây chúng ta dùng `Mutex<T>` để mutate nội
dung bên trong một `Arc<T>`.

Một chi tiết khác cần lưu ý là Rust không thể bảo vệ bạn khỏi mọi loại lỗi logic
khi bạn sử dụng `Mutex<T>`. Hãy nhớ lại trong Chương 15 rằng việc sử dụng
`Rc<T>` đi kèm với rủi ro tạo ra các chu trình tham chiếu (reference cycles), khi
hai giá trị `Rc<T>` tham chiếu lẫn nhau, dẫn đến rò rỉ bộ nhớ. Tương tự như vậy,
`Mutex<T>` cũng đi kèm với rủi ro tạo ra *deadlock*. Deadlock xảy ra khi một thao
tác cần khóa hai tài nguyên và hai thread mỗi bên đã chiếm giữ một lock, khiến
chúng chờ nhau vô thời hạn. Nếu bạn quan tâm đến deadlock, hãy thử viết một
chương trình Rust có deadlock; sau đó tìm hiểu các chiến lược giảm thiểu
deadlock cho mutex trong bất kỳ ngôn ngữ nào và thử triển khai chúng bằng Rust.
Tài liệu API của thư viện chuẩn cho `Mutex<T>` và `MutexGuard` cung cấp nhiều
thông tin hữu ích.

Chúng ta sẽ khép lại chương này bằng việc nói về các trait `Send` và `Sync`, và
cách chúng ta có thể sử dụng chúng với các kiểu dữ liệu tự định nghĩa.

[atomic]: ../std/sync/atomic/index.html
