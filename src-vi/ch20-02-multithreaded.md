## Biến Server Chạy Một Luồng Thành Server Nhiều Luồng

Hiện tại, server sẽ xử lý từng yêu cầu theo lượt, nghĩa là nó sẽ không xử lý
kết nối thứ hai cho đến khi yêu cầu đầu tiên hoàn tất. Nếu server nhận được
ngày càng nhiều yêu cầu, việc thực thi tuần tự này sẽ càng kém hiệu quả.
Nếu server nhận được một yêu cầu mất nhiều thời gian để xử lý, các yêu cầu
sau sẽ phải chờ cho đến khi yêu cầu lâu hoàn tất, ngay cả khi các yêu cầu
mới có thể xử lý nhanh. Chúng ta sẽ cần khắc phục điều này, nhưng trước
hết, hãy xem vấn đề này trong thực tế.

### Mô phỏng Một Yêu Cầu Chậm Trong Cài Đặt Server Hiện Tại

Chúng ta sẽ xem cách một yêu cầu xử lý chậm có thể ảnh hưởng đến các yêu cầu
khác gửi đến server hiện tại. Listing 20-10 triển khai xử lý yêu cầu tới
*/sleep* với phản hồi giả lập chậm, khiến server ngủ 5 giây trước khi
phản hồi.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-10/src/main.rs:here}}
```

<span class="caption">Listing 20-10: Mô phỏng một yêu cầu chậm bằng cách
ngủ 5 giây</span>

Chúng ta chuyển từ `if` sang `match` vì bây giờ có ba trường hợp. Chúng ta
cần match rõ ràng trên một slice của `request_line` để so khớp với các
giá trị literal của chuỗi; `match` không tự động tham chiếu và giải tham
chiếu như phương thức so sánh bằng (`==`) làm.

Nhánh đầu tiên giống như khối `if` trong Listing 20-9. Nhánh thứ hai match
với yêu cầu tới */sleep*. Khi nhận được yêu cầu đó, server sẽ ngủ 5 giây
trước khi hiển thị trang HTML thành công. Nhánh thứ ba giống như khối `else`
trong Listing 20-9.

Bạn có thể thấy server của chúng ta còn rất thô sơ: các thư viện thực tế sẽ
xử lý nhận diện nhiều yêu cầu một cách ít tốn lời hơn nhiều!

Khởi động server bằng `cargo run`. Sau đó mở hai cửa sổ trình duyệt: một cho
*http://127.0.0.1:7878/* và một cho *http://127.0.0.1:7878/sleep*. Nếu bạn
nhập URI */* vài lần, như trước, bạn sẽ thấy phản hồi nhanh. Nhưng nếu
nhập */sleep* rồi load */*, bạn sẽ thấy */* phải chờ cho đến khi `sleep`
ngủ đủ 5 giây mới tải xong.

Có nhiều kỹ thuật để tránh các yêu cầu bị kẹt phía sau một yêu cầu chậm; kỹ
thuật chúng ta sẽ triển khai là thread pool.

### Cải thiện Throughput với Thread Pool

Một *thread pool* là một nhóm các thread đã được sinh ra, đang chờ và sẵn sàng
xử lý một nhiệm vụ. Khi chương trình nhận một nhiệm vụ mới, nó sẽ gán một
thread trong pool cho nhiệm vụ đó, và thread đó sẽ xử lý nhiệm vụ. Các
thread còn lại trong pool sẽ sẵn sàng xử lý các nhiệm vụ khác đến trong
khi thread đầu tiên đang xử lý. Khi thread đầu tiên hoàn tất nhiệm vụ của
nó, nó được trả lại pool của các thread rỗi, sẵn sàng xử lý nhiệm vụ mới.
Thread pool cho phép bạn xử lý các kết nối đồng thời, tăng throughput của
server.

Chúng ta sẽ giới hạn số lượng thread trong pool ở một con số nhỏ để bảo
vệ khỏi các cuộc tấn công Denial of Service (DoS); nếu chương trình tạo một
thread mới cho mỗi yêu cầu khi nó đến, một người gửi 10 triệu yêu cầu
tới server của chúng ta có thể gây hỗn loạn bằng cách sử dụng hết tất cả
tài nguyên server và làm tắc nghẽn xử lý yêu cầu.

Thay vì sinh ra số lượng thread không giới hạn, chúng ta sẽ có một số
thread cố định chờ trong pool. Các yêu cầu đến được gửi vào pool để xử lý.
Pool sẽ duy trì một hàng đợi các yêu cầu đến. Mỗi thread trong pool sẽ lấy
một yêu cầu từ hàng đợi này, xử lý yêu cầu, và sau đó yêu cầu một yêu cầu
khác từ hàng đợi. Với thiết kế này, chúng ta có thể xử lý đồng thời tối đa
`N` yêu cầu, trong đó `N` là số lượng thread. Nếu mỗi thread đang phản hồi
một yêu cầu chạy lâu, các yêu cầu tiếp theo vẫn có thể xếp vào hàng đợi,
nhưng chúng ta đã tăng số lượng yêu cầu chạy lâu có thể xử lý trước khi
đạt đến điểm đó.

Kỹ thuật này chỉ là một trong nhiều cách để cải thiện throughput của
web server. Các lựa chọn khác bạn có thể khám phá là *fork/join model*, 
*mô hình async I/O đơn luồng*, hoặc *mô hình async I/O đa luồng*. Nếu bạn
quan tâm đến chủ đề này, bạn có thể đọc thêm về các giải pháp khác và thử
triển khai; với một ngôn ngữ mức thấp như Rust, tất cả các lựa chọn này
đều khả thi.

Trước khi bắt đầu triển khai thread pool, hãy bàn về cách sử dụng pool
nên trông như thế nào. Khi thiết kế code, viết trước giao diện client có
thể giúp hướng dẫn thiết kế. Viết API của code sao cho cấu trúc phù hợp
với cách bạn muốn gọi nó; sau đó triển khai chức năng bên trong cấu trúc
đó thay vì triển khai chức năng trước rồi mới thiết kế API công khai.

Tương tự như cách chúng ta sử dụng phát triển hướng kiểm thử (TDD) trong
dự án ở Chương 12, ở đây chúng ta sẽ sử dụng phát triển dựa trên compiler.
Chúng ta sẽ viết code gọi các hàm muốn dùng, và sau đó xem lỗi từ compiler
để xác định điều gì cần thay đổi tiếp theo để code hoạt động. Tuy nhiên,
trước khi làm điều đó, chúng ta sẽ khám phá kỹ thuật mà chúng ta sẽ không
dùng làm điểm khởi đầu.

<!-- Old headings. Do not remove or links may break. -->
<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Sinh Một Thread Cho Mỗi Yêu Cầu

Trước hết, hãy xem code của chúng ta sẽ trông như thế nào nếu tạo một thread
mới cho mỗi kết nối. Như đã đề cập trước đó, đây không phải là kế hoạch
cuối cùng của chúng ta do các vấn đề có thể phát sinh từ việc sinh số lượng
thread không giới hạn, nhưng đây là điểm khởi đầu để có một server đa luồng
hoạt động. Sau đó chúng ta sẽ thêm thread pool như một cải tiến, và so sánh
hai giải pháp sẽ dễ hơn. Listing 20-11 cho thấy những thay đổi cần thực hiện
trong `main` để sinh một thread mới xử lý mỗi stream trong vòng lặp `for`.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-11/src/main.rs:here}}
```

<span class="caption">Listing 20-11: Sinh một thread mới cho mỗi stream</span>

Như bạn đã học trong Chương 16, `thread::spawn` sẽ tạo một thread mới và
chạy code trong closure trên thread mới đó. Nếu bạn chạy code này và load
*/sleep* trong trình duyệt, rồi */* trong hai tab trình duyệt khác, bạn sẽ
thấy rằng các yêu cầu tới */* không phải chờ */sleep* hoàn tất. Tuy nhiên,
như đã đề cập, điều này cuối cùng sẽ làm quá tải hệ thống vì bạn đang tạo
các thread mới mà không giới hạn.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Tạo Một Số Lượng Thread Hạn Chế

Chúng ta muốn thread pool hoạt động theo cách tương tự và quen thuộc để
việc chuyển từ threads sang thread pool không yêu cầu thay đổi lớn trong
code sử dụng API của chúng ta. Listing 20-12 cho thấy giao diện giả định
cho struct `ThreadPool` mà chúng ta muốn dùng thay vì `thread::spawn`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-12/src/main.rs:here}}
```

<span class="caption">Listing 20-12: Giao diện `ThreadPool` lý tưởng của chúng ta</span>

Chúng ta dùng `ThreadPool::new` để tạo một thread pool mới với số lượng
thread có thể cấu hình, trong trường hợp này là bốn. Sau đó, trong vòng
lặp `for`, `pool.execute` có giao diện tương tự như `thread::spawn` ở chỗ
nó nhận một closure mà pool sẽ chạy cho mỗi stream. Chúng ta cần triển khai
`pool.execute` sao cho nó nhận closure và giao nó cho một thread trong pool
để chạy. Code này chưa thể biên dịch ngay, nhưng chúng ta sẽ thử để compiler
hướng dẫn cách sửa.

<!-- Old headings. Do not remove or links may break. -->
<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Xây dựng `ThreadPool` Sử Dụng Phát Triển Dựa Trên Compiler

Thực hiện các thay đổi trong Listing 20-12 vào *src/main.rs*, và sau đó
sử dụng các lỗi từ `cargo check` để hướng dẫn quá trình phát triển. Đây là
lỗi đầu tiên mà chúng ta nhận được:

```console
{{#include ../listings/ch20-web-server/listing-20-12/output.txt}}
```

Tuyệt vời! Lỗi này cho chúng ta biết cần có một kiểu hoặc module `ThreadPool`,
vì vậy chúng ta sẽ xây dựng một cái ngay bây giờ. Việc triển khai `ThreadPool`
sẽ độc lập với loại công việc mà web server của chúng ta đang thực hiện. Vì
vậy, hãy chuyển crate `hello` từ binary crate sang library crate để chứa
việc triển khai `ThreadPool`. Sau khi chuyển sang library crate, chúng ta
cũng có thể sử dụng thư viện thread pool riêng cho bất kỳ công việc nào
muốn làm với thread pool, không chỉ giới hạn cho việc phục vụ các yêu cầu
web.

Tạo một file *src/lib.rs* chứa nội dung sau, đây là định nghĩa đơn giản nhất
của struct `ThreadPool` mà chúng ta có thể có hiện tại:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

Sau đó, chỉnh sửa file *main.rs* để đưa `ThreadPool` vào phạm vi sử dụng từ
library crate bằng cách thêm đoạn code sau lên đầu *src/main.rs*:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

Code này vẫn chưa hoạt động, nhưng hãy kiểm tra lại để nhận lỗi tiếp theo
mà chúng ta cần xử lý:

```console
{{#include ../listings/ch20-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Lỗi này cho thấy bước tiếp theo là chúng ta cần tạo một hàm liên kết (associated
function) tên là `new` cho `ThreadPool`. Chúng ta cũng biết rằng `new` cần
một tham số có thể nhận `4` làm đối số và nên trả về một instance của
`ThreadPool`. Hãy triển khai hàm `new` đơn giản nhất có các đặc điểm đó:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

Chúng ta chọn `usize` làm kiểu của tham số `size`, vì một số lượng thread
âm là vô nghĩa. Chúng ta cũng biết rằng chúng ta sẽ dùng số 4 này làm số
lượng phần tử trong một tập hợp các thread, và đó là mục đích của kiểu
`usize`, như đã thảo luận trong phần [“Integer Types”][integer-types]<!--
ignore --> của Chương 3.

Hãy kiểm tra lại code:

```console
{{#include ../listings/ch20-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Bây giờ lỗi xuất hiện vì chúng ta chưa có phương thức `execute` trên
`ThreadPool`. Hãy nhớ lại trong phần [“Tạo Một Số Lượng Thread Hạn Chế”](#creating-a-finite-number-of-threads)<!-- ignore --> rằng
chúng ta quyết định thread pool nên có giao diện tương tự `thread::spawn`.
Ngoài ra, chúng ta sẽ triển khai hàm `execute` sao cho nó nhận closure được
truyền và giao nó cho một thread rỗi trong pool để chạy.

Chúng ta sẽ định nghĩa phương thức `execute` trên `ThreadPool` để nhận
một closure làm tham số. Hãy nhớ lại trong phần [“Di chuyển giá trị
bắt được ra khỏi closure và các trait `Fn`”][fn-traits]<!-- ignore --> ở
Chương 13 rằng chúng ta có thể nhận closures làm tham số với ba trait
khác nhau: `Fn`, `FnMut`, và `FnOnce`. Chúng ta cần quyết định loại closure
sẽ dùng ở đây. Chúng ta biết rằng cuối cùng sẽ làm điều gì đó tương tự
như triển khai `thread::spawn` trong thư viện chuẩn, vì vậy có thể xem
giới hạn mà chữ ký của `thread::spawn` đặt trên tham số của nó. Tài liệu
cho chúng ta thấy như sau:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Tham số kiểu `F` là thứ chúng ta quan tâm ở đây; tham số kiểu `T` liên quan
đến giá trị trả về, và chúng ta không quan tâm đến điều đó. Chúng ta có thể
thấy rằng `spawn` sử dụng `FnOnce` làm trait bound trên `F`. Đây có lẽ là
cái chúng ta cũng muốn, vì cuối cùng chúng ta sẽ truyền đối số nhận được
trong `execute` cho `spawn`. Chúng ta có thể tự tin hơn rằng `FnOnce` là
trait cần dùng bởi vì thread chạy một yêu cầu sẽ chỉ thực thi closure của
yêu cầu đó một lần, phù hợp với `Once` trong `FnOnce`.

Tham số kiểu `F` cũng có trait bound `Send` và lifetime bound `'static`,
các bound này hữu ích trong tình huống của chúng ta: cần `Send` để chuyển
closure từ thread này sang thread khác và `'static` vì chúng ta không biết
thread sẽ mất bao lâu để thực thi. Hãy tạo phương thức `execute` trên
`ThreadPool` nhận một tham số generic kiểu `F` với các bound này:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

Chúng ta vẫn dùng `()` sau `FnOnce` vì `FnOnce` này đại diện cho một closure
không nhận tham số và trả về kiểu unit `()`. Giống như định nghĩa hàm, kiểu
trả về có thể bỏ qua trong chữ ký, nhưng ngay cả khi không có tham số, chúng
ta vẫn cần dấu ngoặc đơn.

Một lần nữa, đây là triển khai đơn giản nhất của phương thức `execute`: nó
không làm gì cả, nhưng mục tiêu hiện tại chỉ là làm cho code có thể biên
dịch. Hãy kiểm tra lại:

```console
{{#include ../listings/ch20-web-server/no-listing-03-define-execute/output.txt}}
```

Nó biên dịch được rồi! Nhưng lưu ý rằng nếu bạn thử `cargo run` và gửi yêu cầu
trong trình duyệt, bạn sẽ thấy các lỗi trong trình duyệt như đã thấy ở đầu
chương. Thư viện của chúng ta vẫn chưa thực sự gọi closure được truyền vào
`execute`!

> Lưu ý: Một câu nói bạn có thể nghe về các ngôn ngữ với compiler nghiêm ngặt,
> như Haskell và Rust, là “nếu code biên dịch được, thì nó chạy được.” Nhưng
> câu nói này không hoàn toàn đúng. Dự án của chúng ta biên dịch được, nhưng
> hoàn toàn không làm gì cả! Nếu chúng ta đang xây dựng một dự án thực sự,
> hoàn chỉnh, đây sẽ là thời điểm tốt để bắt đầu viết unit test để kiểm tra
> rằng code biên dịch *và* có hành vi chúng ta mong muốn.

#### Xác thực Số Lượng Thread trong `new`

Chúng ta chưa làm gì với các tham số của `new` và `execute`. Hãy triển khai
nội dung của các hàm này với hành vi mà chúng ta muốn. Để bắt đầu, hãy nghĩ
về `new`. Trước đó, chúng ta đã chọn kiểu unsigned cho tham số `size`,
bởi vì một pool với số thread âm là vô nghĩa. Tuy nhiên, một pool với số
thread bằng 0 cũng vô nghĩa, nhưng zero là một giá trị `usize` hoàn toàn
hợp lệ. Chúng ta sẽ thêm code để kiểm tra rằng `size` lớn hơn 0 trước khi
trả về một instance `ThreadPool` và làm chương trình panic nếu nhận được 0
bằng cách sử dụng macro `assert!`, như minh họa trong Listing 20-13.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-13/src/lib.rs:here}}
```

<span class="caption">Listing 20-13: Triển khai `ThreadPool::new` để panic nếu
`size` bằng 0</span>

Chúng ta cũng đã thêm một số tài liệu cho `ThreadPool` bằng doc comments.
Lưu ý rằng chúng ta đã tuân theo các thực hành tài liệu tốt bằng cách thêm
một phần nêu ra các tình huống mà hàm có thể panic, như đã thảo luận trong
Chương 14. Thử chạy `cargo doc --open` và nhấp vào struct `ThreadPool` để
xem tài liệu được sinh ra cho `new` trông như thế nào!

Thay vì thêm macro `assert!` như chúng ta đã làm ở đây, chúng ta có thể
đổi `new` thành `build` và trả về một `Result` giống như với `Config::build`
trong dự án I/O ở Listing 12-9. Nhưng trong trường hợp này, chúng ta quyết
định rằng việc cố tạo một thread pool mà không có thread nào nên là lỗi
không thể khôi phục. Nếu bạn cảm thấy tham vọng, thử viết một hàm tên là
`build` với chữ ký sau để so sánh với hàm `new`:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Tạo Không Gian Lưu Trữ Các Thread

Bây giờ chúng ta đã có cách để biết rằng số lượng thread hợp lệ để lưu trong
pool, chúng ta có thể tạo các thread đó và lưu chúng trong struct `ThreadPool`
trước khi trả về struct. Nhưng làm thế nào để “lưu” một thread? Hãy cùng
nhìn lại chữ ký của `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Hàm `spawn` trả về một `JoinHandle<T>`, trong đó `T` là kiểu mà closure trả
về. Hãy thử sử dụng `JoinHandle` và xem điều gì xảy ra. Trong trường hợp của
chúng ta, các closure được truyền vào thread pool sẽ xử lý kết nối và không
trả về gì, nên `T` sẽ là kiểu unit `()`.

Code trong Listing 20-14 sẽ biên dịch được nhưng vẫn chưa tạo bất kỳ thread
nào. Chúng ta đã thay đổi định nghĩa của `ThreadPool` để giữ một vector các
instance `thread::JoinHandle<()>`, khởi tạo vector với dung lượng `size`,
thiết lập một vòng lặp `for` để chạy code tạo các thread, và trả về một
instance `ThreadPool` chứa chúng.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-14/src/lib.rs:here}}
```

<span class="caption">Listing 20-14: Tạo một vector cho `ThreadPool` để giữ các thread</span>

Chúng ta đã đưa `std::thread` vào phạm vi sử dụng trong library crate, vì
chúng ta dùng `thread::JoinHandle` làm kiểu của các phần tử trong vector của
`ThreadPool`.

Khi nhận được một kích thước hợp lệ, `ThreadPool` của chúng ta tạo một
vector mới có thể chứa `size` phần tử. Hàm `with_capacity` thực hiện cùng
một nhiệm vụ như `Vec::new` nhưng có một khác biệt quan trọng: nó cấp phát
trước không gian trong vector. Vì chúng ta biết cần lưu `size` phần tử
trong vector, việc cấp phát này trước giúp hiệu quả hơn một chút so với
`Vec::new`, vốn sẽ tự thay đổi kích thước khi các phần tử được chèn vào.

Khi bạn chạy lại `cargo check`, nó sẽ thành công.

#### Struct `Worker` Chịu Trách Nhiệm Gửi Code từ `ThreadPool` tới Thread

Chúng ta đã để lại một comment trong vòng lặp `for` ở Listing 20-14 liên
quan đến việc tạo các thread. Ở đây, chúng ta sẽ xem cách thực sự tạo thread.
Thư viện chuẩn cung cấp `thread::spawn` như một cách để tạo thread, và
`thread::spawn` mong muốn nhận được một đoạn code mà thread sẽ chạy ngay
khi thread được tạo. Tuy nhiên, trong trường hợp của chúng ta, chúng ta muốn
tạo các thread và để chúng *chờ* code mà chúng ta sẽ gửi sau. Việc triển
khai thread trong thư viện chuẩn không bao gồm cách làm này; chúng ta phải
tự triển khai.

Chúng ta sẽ triển khai hành vi này bằng cách giới thiệu một cấu trúc dữ
liệu mới giữa `ThreadPool` và các thread để quản lý hành vi này. Chúng ta
sẽ gọi cấu trúc dữ liệu này là *Worker*, một thuật ngữ phổ biến trong các
triển khai pooling. Worker sẽ nhận code cần chạy và thực thi code đó trên
thread của chính Worker. Hãy tưởng tượng những người làm việc trong bếp
tại một nhà hàng: các worker sẽ chờ cho đến khi có đơn hàng từ khách, rồi
chịu trách nhiệm nhận đơn và thực hiện nó.

Thay vì lưu một vector các instance `JoinHandle<()>` trong thread pool, chúng
ta sẽ lưu các instance của struct `Worker`. Mỗi `Worker` sẽ lưu một instance
`JoinHandle<()>`. Sau đó, chúng ta sẽ triển khai một phương thức trên `Worker`
nhận một closure để chạy và gửi nó đến thread đang chạy để thực thi. Chúng
ta cũng sẽ cấp cho mỗi worker một `id` để phân biệt các worker khác nhau
trong pool khi ghi log hoặc gỡ lỗi.

Dưới đây là quy trình mới xảy ra khi tạo một `ThreadPool`. Chúng ta sẽ
triển khai code gửi closure tới thread sau khi đã thiết lập `Worker` như
sau:

1. Định nghĩa một struct `Worker` giữ `id` và `JoinHandle<()>`.
2. Thay đổi `ThreadPool` để giữ một vector các instance `Worker`.
3. Định nghĩa hàm `Worker::new` nhận một số `id` và trả về một instance
   `Worker` chứa `id` và một thread được spawn với một closure rỗng.
4. Trong `ThreadPool::new`, dùng bộ đếm vòng lặp `for` để tạo `id`, tạo
   một `Worker` mới với `id` đó, và lưu worker vào vector.

Nếu bạn muốn thử thách, hãy triển khai những thay đổi này trước khi xem
code trong Listing 20-15.

Sẵn sàng chưa? Đây là Listing 20-15 với một cách để thực hiện các sửa đổi
nêu trên.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-15/src/lib.rs:here}}
```

<span class="caption">Listing 20-15: Sửa đổi `ThreadPool` để giữ các instance `Worker` thay vì giữ trực tiếp các thread</span>

Chúng ta đã đổi tên trường trong `ThreadPool` từ `threads` thành `workers`
vì bây giờ nó giữ các instance `Worker` thay vì các instance `JoinHandle<()>`.
Chúng ta dùng bộ đếm trong vòng lặp `for` làm đối số cho `Worker::new`, và
lưu từng `Worker` mới vào vector có tên `workers`.

Code bên ngoài (như server của chúng ta trong *src/main.rs*) không cần
biết chi tiết triển khai về việc sử dụng struct `Worker` trong `ThreadPool`,
vì vậy chúng ta làm struct `Worker` và hàm `new` của nó là private. Hàm
`Worker::new` dùng `id` được truyền vào và lưu một instance `JoinHandle<()>`
được tạo ra bằng cách spawn một thread mới với closure rỗng.

> Lưu ý: Nếu hệ điều hành không thể tạo thread vì không đủ tài nguyên hệ
> thống, `thread::spawn` sẽ panic. Điều này sẽ khiến toàn bộ server của
> chúng ta panic, mặc dù một số thread có thể tạo thành công. Vì đơn giản,
> hành vi này là ổn, nhưng trong triển khai thread pool thực tế, bạn có thể
> muốn dùng [`std::thread::Builder`][builder]<!-- ignore --> và phương thức
> [`spawn`][builder-spawn]<!-- ignore --> của nó, trả về `Result` thay vì panic.

Code này sẽ biên dịch và lưu số lượng instance `Worker` mà chúng ta chỉ định
làm đối số cho `ThreadPool::new`. Nhưng chúng ta vẫn *chưa* xử lý closure
mà ta nhận được trong `execute`. Hãy xem cách làm tiếp theo.

#### Gửi Yêu Cầu tới Thread thông qua Channels

Vấn đề tiếp theo là các closure được truyền cho `thread::spawn` hiện không
làm gì cả. Hiện tại, chúng ta nhận closure muốn thực thi trong phương thức
`execute`. Nhưng chúng ta cần truyền một closure cho `thread::spawn` khi
tạo mỗi `Worker` trong quá trình tạo `ThreadPool`.

Chúng ta muốn các struct `Worker` vừa tạo lấy code để chạy từ một queue
được giữ trong `ThreadPool` và gửi code đó tới thread của nó để thực thi.

Các channel mà chúng ta đã học trong Chương 16 — một cách đơn giản để
giao tiếp giữa hai thread — sẽ hoàn hảo cho trường hợp này. Chúng ta sẽ
dùng một channel làm queue các job, và `execute` sẽ gửi một job từ
`ThreadPool` tới các instance `Worker`, và `Worker` sẽ gửi job tới thread
của nó. Kế hoạch như sau:

1. `ThreadPool` sẽ tạo một channel và giữ sender.
2. Mỗi `Worker` sẽ giữ receiver.
3. Chúng ta sẽ tạo một struct `Job` mới để giữ các closure mà muốn gửi qua channel.
4. Phương thức `execute` sẽ gửi job mà nó muốn thực thi qua sender.
5. Trong thread của nó, `Worker` sẽ lặp qua receiver và thực thi các closure
   của bất kỳ job nào nó nhận được.

Hãy bắt đầu bằng việc tạo một channel trong `ThreadPool::new` và giữ sender
trong instance `ThreadPool`, như minh họa trong Listing 20-16. Struct `Job`
hiện tại chưa giữ gì nhưng sẽ là kiểu của item mà chúng ta gửi qua channel.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-16/src/lib.rs:here}}
```

<span class="caption">Listing 20-16: Sửa đổi `ThreadPool` để lưu sender của một channel truyền các instance `Job`</span>

Trong `ThreadPool::new`, chúng ta tạo channel mới và cho pool giữ sender. Điều
này sẽ biên dịch thành công.

Hãy thử truyền receiver của channel vào từng worker khi thread pool tạo
channel. Chúng ta biết rằng muốn sử dụng receiver trong thread mà worker
spawn, vì vậy sẽ tham chiếu tới tham số `receiver` trong closure. Code trong
Listing 20-17 vẫn chưa thể biên dịch hoàn chỉnh.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-17/src/lib.rs:here}}
```

<span class="caption">Listing 20-17: Truyền receiver tới các worker</span>

Chúng ta đã thực hiện một số thay đổi nhỏ và đơn giản: truyền receiver vào
`Worker::new`, rồi sử dụng nó bên trong closure.

Khi thử kiểm tra code này, chúng ta nhận được lỗi sau:

```console
{{#include ../listings/ch20-web-server/listing-20-17/output.txt}}
```

Code đang cố gắng truyền `receiver` tới nhiều instance `Worker`. Điều này sẽ
không hoạt động, như bạn đã học trong Chương 16: triển khai channel mà Rust
cung cấp là *nhiều producer*, một *consumer*. Điều này có nghĩa là chúng ta
không thể chỉ clone đầu nhận của channel để sửa code này. Chúng ta cũng không
muốn gửi một message nhiều lần tới nhiều consumer; chúng ta muốn một danh
sách message với nhiều worker sao cho mỗi message chỉ được xử lý một lần.

Ngoài ra, việc lấy một job ra khỏi queue của channel liên quan tới việc
biến đổi `receiver`, vì vậy các thread cần một cách an toàn để chia sẻ và
sửa đổi `receiver`; nếu không, chúng ta có thể gặp các race condition (như
đã đề cập trong Chương 16).

Hãy nhớ lại các smart pointer an toàn với thread được thảo luận trong Chương
16: để chia sẻ quyền sở hữu giữa nhiều thread và cho phép các thread thay
đổi giá trị, chúng ta cần dùng `Arc<Mutex<T>>`. Kiểu `Arc` cho phép nhiều
worker sở hữu receiver, và `Mutex` đảm bảo rằng chỉ có một worker lấy job
từ receiver tại một thời điểm. Listing 20-18 trình bày các thay đổi cần thực
hiện.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-18/src/lib.rs:here}}
```

<span class="caption">Listing 20-18: Chia sẻ receiver giữa các worker sử dụng `Arc` và `Mutex`</span>

Trong `ThreadPool::new`, chúng ta đặt receiver vào trong một `Arc` và một `Mutex`. 
Với mỗi worker mới, chúng ta clone `Arc` để tăng reference count, cho phép các 
worker cùng sở hữu receiver.

Với những thay đổi này, code đã biên dịch được! Chúng ta đang tiến gần tới mục tiêu!

#### Triển khai phương thức `execute`

Cuối cùng, hãy triển khai phương thức `execute` trên `ThreadPool`. Chúng ta cũng 
sẽ thay `Job` từ một struct sang type alias cho một trait object, chứa kiểu closure 
mà `execute` nhận vào. Như đã thảo luận trong phần [“Creating Type Synonyms with Type Aliases”][creating-type-synonyms-with-type-aliases]<!-- ignore --> 
trong Chương 19, type alias giúp chúng ta rút gọn các kiểu dài để tiện sử dụng. 
Xem ví dụ trong Listing 20-19.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-19/src/lib.rs:here}}
```

<span class="caption">Listing 20-19: Tạo type alias `Job` cho một `Box` giữ từng closure và gửi job xuống channel</span>

Sau khi tạo một instance `Job` mới sử dụng closure nhận được trong `execute`, 
chúng ta gửi job đó xuống đầu gửi (sending end) của channel. Chúng ta gọi 
`unwrap` trên `send` trong trường hợp gửi thất bại. Điều này có thể xảy ra 
nếu, ví dụ, chúng ta dừng tất cả các thread, nghĩa là đầu nhận (receiving end) 
không còn nhận message mới. Hiện tại, chúng ta không thể dừng các thread: 
các thread sẽ tiếp tục thực thi miễn là pool tồn tại. Lý do chúng ta dùng 
`unwrap` là vì biết trường hợp thất bại sẽ không xảy ra, nhưng compiler không biết điều đó.

Nhưng chúng ta vẫn chưa hoàn tất! Trong worker, closure được truyền cho 
`thread::spawn` hiện tại vẫn chỉ *tham chiếu* đến đầu nhận của channel. 
Thay vào đó, chúng ta cần closure lặp vô hạn, hỏi đầu nhận của channel về 
một job và chạy job khi nhận được. Hãy thực hiện thay đổi như trong Listing 20-20 
trong `Worker::new`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-20/src/lib.rs:here}}
```

<span class="caption">Listing 20-20: Nhận và thực thi các job trong thread của worker</span>

Ở đây, trước tiên chúng ta gọi `lock` trên `receiver` để lấy mutex, rồi gọi 
`unwrap` để panic nếu có lỗi. Việc lấy lock có thể thất bại nếu mutex ở trạng 
thái *poisoned*, điều này xảy ra nếu một thread khác panic trong khi giữ lock 
thay vì thả lock. Trong tình huống này, gọi `unwrap` để thread này panic là 
hành động đúng. Bạn có thể thay `unwrap` bằng `expect` với thông báo lỗi 
có ý nghĩa với bạn.

Nếu chúng ta lấy được lock trên mutex, gọi `recv` để nhận một `Job` từ channel. 
Một `unwrap` cuối cùng bỏ qua mọi lỗi tại đây, có thể xảy ra nếu thread giữ 
sender đã tắt, tương tự như cách `send` trả về `Err` nếu receiver tắt.

Lệnh gọi `recv` sẽ block, vì vậy nếu chưa có job, thread hiện tại sẽ chờ cho 
đến khi có job. `Mutex<T>` đảm bảo chỉ một thread `Worker` tại một thời điểm 
thử yêu cầu job.

Thread pool của chúng ta giờ đã hoạt động! Hãy chạy `cargo run` và thực hiện 
một vài request:

<!-- manual-regeneration
cd listings/ch20-web-server/listing-20-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never read: `workers`
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: field is never read: `id`
  --> src/lib.rs:48:5
   |
48 |     id: usize,
   |     ^^^^^^^^^

warning: field is never read: `thread`
  --> src/lib.rs:49:5
   |
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: `hello` (lib) generated 3 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Thành công! Giờ chúng ta đã có một thread pool thực thi các kết nối một cách
bất đồng bộ. Không bao giờ có hơn bốn thread được tạo, vì vậy hệ thống sẽ không
bị quá tải nếu server nhận nhiều request. Nếu chúng ta gửi một request tới 
*/sleep*, server vẫn có thể phục vụ các request khác nhờ một thread khác 
thực thi chúng.

> Lưu ý: nếu bạn mở */sleep* trên nhiều cửa sổ trình duyệt cùng lúc, chúng
> có thể tải lần lượt với khoảng cách 5 giây. Một số trình duyệt thực thi
> nhiều instance của cùng một request theo thứ tự để phục vụ caching. 
> Giới hạn này không phải do server web của chúng ta gây ra.

Sau khi học về vòng lặp `while let` trong Chương 18, bạn có thể tự hỏi 
tại sao chúng ta không viết code thread của worker như trong Listing 20-21.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-21/src/lib.rs:here}}
```

<span class="caption">Listing 20-21: Một cách triển khai khác của `Worker::new` sử dụng `while let`</span>

Code này biên dịch và chạy được nhưng không mang lại hành vi threading như mong muốn: 
một request chậm vẫn sẽ khiến các request khác phải chờ xử lý. Nguyên nhân khá tinh tế: 
struct `Mutex` không có phương thức `unlock` công khai vì quyền sở hữu lock dựa trên 
vòng đời của `MutexGuard<T>` trong `LockResult<MutexGuard<T>>` mà phương thức `lock` trả về. 
Tại thời điểm biên dịch, borrow checker sẽ thực thi quy tắc rằng một tài nguyên được 
bảo vệ bởi `Mutex` không thể truy cập nếu chúng ta không giữ lock. Tuy nhiên, cách 
triển khai này cũng có thể khiến lock bị giữ lâu hơn dự kiến nếu chúng ta không chú ý 
đến vòng đời của `MutexGuard<T>`.

Code trong Listing 20-20 sử dụng `let job = receiver.lock().unwrap().recv().unwrap();` 
hoạt động vì với `let`, bất kỳ giá trị tạm thời nào dùng trong biểu thức bên phải 
dấu bằng sẽ bị drop ngay khi câu lệnh `let` kết thúc. Tuy nhiên, `while let` (và `if let` 
và `match`) không drop các giá trị tạm thời cho đến khi kết thúc block liên quan. Trong 
Listing 20-21, lock vẫn bị giữ trong suốt cuộc gọi tới `job()`, nghĩa là các worker khác 
không thể nhận job.

[creating-type-synonyms-with-type-aliases]:
ch19-04-advanced-types.html#creating-type-synonyms-with-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[fn-traits]:
ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
