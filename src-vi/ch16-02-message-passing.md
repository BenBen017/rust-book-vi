## Sử dụng Message Passing để Chuyển Dữ liệu Giữa Các Luồng

Một phương pháp ngày càng phổ biến để đảm bảo concurrency an toàn là *message passing*, nơi các luồng hoặc actor giao tiếp bằng cách gửi cho nhau các thông điệp chứa dữ liệu. Ý tưởng này được tóm tắt trong một câu slogan từ [tài liệu ngôn ngữ Go](https://golang.org/doc/effective_go.html#concurrency): 
“Đừng giao tiếp bằng cách chia sẻ bộ nhớ; thay vào đó, hãy chia sẻ bộ nhớ bằng cách giao tiếp.”

Để thực hiện concurrency dựa trên gửi thông điệp, thư viện chuẩn của Rust cung cấp một triển khai của *channels*. Một channel là một khái niệm lập trình tổng quát, theo đó dữ liệu được gửi từ một luồng sang luồng khác.

Bạn có thể tưởng tượng một channel trong lập trình giống như một dòng nước có hướng, chẳng hạn như một con suối hoặc một dòng sông. Nếu bạn đặt một thứ gì đó như một con vịt cao su vào sông, nó sẽ trôi xuôi dòng đến cuối con đường nước.

Một channel có hai nửa: nửa truyền và nửa nhận. Nửa truyền là vị trí thượng lưu nơi bạn đặt các con vịt cao su vào sông, và nửa nhận là nơi con vịt cao su sẽ đến hạ lưu. Một phần mã của bạn gọi các phương thức trên nửa truyền với dữ liệu bạn muốn gửi, và phần khác kiểm tra đầu nhận để nhận các thông điệp đến. Một channel được coi là *đóng* nếu một trong hai nửa truyền hoặc nhận bị thả.

Ở đây, chúng ta sẽ phát triển một chương trình có một luồng tạo giá trị và gửi chúng xuống một channel, và một luồng khác sẽ nhận các giá trị đó và in ra. Chúng ta sẽ gửi các giá trị đơn giản giữa các luồng sử dụng channel để minh họa tính năng. Khi bạn đã quen với kỹ thuật này, bạn có thể sử dụng channels cho bất kỳ luồng nào cần giao tiếp với nhau, chẳng hạn như hệ thống chat hoặc một hệ thống nơi nhiều luồng thực hiện các phần của phép tính và gửi các phần đó đến một luồng tổng hợp kết quả.

Đầu tiên, trong Listing 16-6, chúng ta sẽ tạo một channel nhưng chưa làm gì với nó. Lưu ý rằng điều này sẽ chưa biên dịch vì Rust chưa biết loại giá trị nào chúng ta muốn gửi qua channel.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

<span class="caption">Listing 16-6: Tạo một channel và gán hai nửa cho `tx` và `rx`</span>

Chúng ta tạo một channel mới bằng cách sử dụng hàm `mpsc::channel`; `mpsc` viết tắt của *multiple producer, single consumer* (nhiều người sản xuất, một người tiêu thụ). Tóm lại, cách thư viện chuẩn của Rust triển khai channels nghĩa là một channel có thể có nhiều *sending* ends tạo ra giá trị nhưng chỉ có một *receiving* end tiêu thụ các giá trị đó. Hãy tưởng tượng nhiều dòng suối hợp lại thành một dòng sông lớn: tất cả mọi thứ được gửi từ bất kỳ dòng suối nào sẽ kết thúc trong một dòng sông ở cuối. Hiện tại, chúng ta sẽ bắt đầu với một producer duy nhất, nhưng sau đó sẽ thêm nhiều producer khi ví dụ này hoạt động.

Hàm `mpsc::channel` trả về một tuple, phần tử đầu tiên là nửa gửi -- transmitter -- và phần tử thứ hai là nửa nhận -- receiver. Các viết tắt `tx` và `rx` thường được sử dụng trong nhiều lĩnh vực cho *transmitter* và *receiver* tương ứng, vì vậy chúng ta đặt tên biến như vậy để chỉ định mỗi đầu. Chúng ta sử dụng một câu lệnh `let` với pattern để destructure tuple; chúng ta sẽ bàn về việc sử dụng pattern trong các câu lệnh `let` và destructuring trong Chương 18. Hiện tại, hãy biết rằng việc sử dụng câu lệnh `let` theo cách này là một cách tiện lợi để trích xuất các phần tử của tuple do `mpsc::channel` trả về.

Hãy di chuyển nửa truyền vào một luồng được spawn và cho nó gửi một chuỗi để luồng spawn có thể giao tiếp với luồng chính, như được hiển thị trong Listing 16-7. Điều này giống như đặt một con vịt cao su vào dòng sông thượng lưu hoặc gửi một tin nhắn chat từ một luồng sang luồng khác.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

<span class="caption">Listing 16-7: Di chuyển `tx` vào luồng spawn và gửi “hi”</span>

Một lần nữa, chúng ta sử dụng `thread::spawn` để tạo một luồng mới và sau đó dùng `move` để di chuyển `tx` vào closure sao cho luồng spawn sở hữu `tx`. Luồng spawn cần sở hữu transmitter để có thể gửi tin nhắn qua channel. Transmitter có phương thức `send` nhận giá trị mà chúng ta muốn gửi. Phương thức `send` trả về kiểu `Result<T, E>`, vì vậy nếu receiver đã bị drop và không có nơi nào để gửi giá trị, thao tác gửi sẽ trả về lỗi. Trong ví dụ này, chúng ta gọi `unwrap` để panic nếu có lỗi. Nhưng trong một ứng dụng thực tế, chúng ta sẽ xử lý nó một cách thích hợp: quay lại Chương 9 để xem các chiến lược xử lý lỗi đúng cách.

Trong Listing 16-8, chúng ta sẽ lấy giá trị từ receiver trong luồng chính. Điều này giống như lấy con vịt cao su từ cuối dòng sông hoặc nhận một tin nhắn chat.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

<span class="caption">Listing 16-8: Nhận giá trị “hi” trong luồng chính và in ra</span>

Receiver có hai phương thức hữu ích: `recv` và `try_recv`. Chúng ta đang sử dụng `recv`, viết tắt của *receive*, phương thức này sẽ block luồng chính và chờ cho đến khi một giá trị được gửi xuống channel. Khi có giá trị được gửi, `recv` sẽ trả về giá trị đó dưới dạng `Result<T, E>`. Khi transmitter đóng, `recv` sẽ trả về lỗi để báo rằng không còn giá trị nào nữa.

Phương thức `try_recv` không block, mà thay vào đó sẽ trả về ngay lập tức một `Result<T, E>`: giá trị `Ok` chứa tin nhắn nếu có tin nhắn, và giá trị `Err` nếu lần này không có tin nhắn nào. Sử dụng `try_recv` hữu ích nếu luồng này còn công việc khác để làm trong khi chờ tin nhắn: chúng ta có thể viết một vòng lặp gọi `try_recv` theo thời gian, xử lý tin nhắn nếu có, và nếu không thì làm công việc khác một thời gian ngắn trước khi kiểm tra lại.

Chúng ta đã dùng `recv` trong ví dụ này cho đơn giản; luồng chính không còn công việc gì khác ngoài chờ tin nhắn, nên việc block luồng chính là hợp lý.

Khi chạy code trong Listing 16-8, chúng ta sẽ thấy giá trị được in ra từ luồng chính:

<!-- Không trích xuất đầu ra vì các thay đổi với đầu ra này không đáng kể;
các thay đổi có khả năng do các luồng chạy khác nhau chứ không phải do compiler thay đổi -->

```text
Got: hi
```

Perfect!

### Kênh và Việc Chuyển Giao Quyền Sở Hữu

Các quy tắc sở hữu đóng vai trò then chốt trong việc gửi tin nhắn 
vì chúng giúp bạn viết mã đồng thời an toàn. Ngăn ngừa lỗi trong 
lập trình đồng thời là lợi ích của việc luôn xem xét quyền sở hữu 
trong toàn bộ chương trình Rust của bạn. Hãy làm một thí nghiệm để 
minh họa cách kênh và quyền sở hữu phối hợp ngăn ngừa vấn đề: chúng ta 
sẽ thử sử dụng một giá trị `val` trong luồng được tạo ra *sau khi* đã gửi nó qua kênh. 
Hãy thử biên dịch mã trong Listing 16-9 để thấy tại sao mã này không được phép:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

<span class="caption">Listing 16-9: Thử sử dụng `val` sau khi chúng ta đã gửi nó
qua kênh</span>

Ở đây, chúng ta cố gắng in ra `val` sau khi đã gửi nó xuống kênh thông qua
`tx.send`. Việc cho phép điều này sẽ là một ý tưởng tồi: một khi giá trị đã
được gửi sang một luồng khác, luồng đó có thể sửa đổi hoặc hủy (drop) nó trước
khi chúng ta cố gắng sử dụng lại giá trị này. Rất có thể, các thay đổi từ luồng
kia sẽ gây ra lỗi hoặc những kết quả không mong muốn do dữ liệu không nhất
quán hoặc thậm chí không còn tồn tại. Tuy nhiên, Rust sẽ báo lỗi khi chúng ta
cố gắng biên dịch mã trong Listing 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Lỗi đồng thời (concurrency) của chúng ta đã gây ra một lỗi tại thời điểm biên dịch. Hàm `send`
sẽ lấy quyền sở hữu (take ownership) của tham số truyền vào, và khi giá trị bị move, phía nhận
(receiver) sẽ nắm quyền sở hữu của nó. Điều này ngăn chúng ta vô tình sử dụng lại giá trị sau khi
đã gửi đi; hệ thống ownership sẽ kiểm tra để đảm bảo mọi thứ đều hợp lệ.

### Gửi Nhiều Giá Trị và Quan Sát Receiver Đang Chờ

Đoạn mã trong Listing 16-8 đã biên dịch và chạy được, nhưng nó chưa cho thấy rõ rằng
hai thread riêng biệt đang giao tiếp với nhau thông qua channel. Trong Listing
16-10, chúng ta đã thực hiện một số chỉnh sửa để chứng minh rằng đoạn mã trong
Listing 16-8 thực sự đang chạy đồng thời: thread được spawn giờ đây sẽ gửi nhiều
thông điệp và tạm dừng một giây giữa mỗi lần gửi.

<span class="filename">Filename: src/main.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

<span class="caption">Listing 16-10: Gửi nhiều thông điệp và tạm dừng
giữa mỗi lần gửi</span>

Lần này, thread được spawn có một vector các chuỗi mà chúng ta muốn gửi tới
thread chính. Chúng ta lặp qua từng phần tử, gửi từng giá trị một, và tạm dừng
giữa mỗi lần gửi bằng cách gọi hàm `thread::sleep` với một giá trị `Duration`
là 1 giây.

Trong thread chính, chúng ta không còn gọi hàm `recv` một cách tường minh nữa:
thay vào đó, chúng ta coi `rx` như một iterator. Với mỗi giá trị nhận được,
chúng ta in nó ra. Khi channel bị đóng, quá trình lặp sẽ kết thúc.

Khi chạy đoạn mã trong Listing 16-10, bạn sẽ thấy output sau, với khoảng dừng
1 giây giữa mỗi dòng:

<!-- Không trích xuất output vì những thay đổi trong output này không đáng kể;
các thay đổi nhiều khả năng là do các thread chạy khác nhau, thay vì do
những thay đổi trong trình biên dịch -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Bởi vì trong vòng lặp `for` ở thread chính chúng ta không có bất kỳ đoạn mã nào
để tạm dừng hay trì hoãn, nên có thể thấy rằng thread chính đang chờ để nhận
các giá trị được gửi từ thread được spawn.

### Tạo Nhiều Producer bằng Cách Clone Transmitter

Trước đó, chúng ta đã đề cập rằng `mpsc` là từ viết tắt của *multiple producer,
single consumer* (nhiều bên gửi, một bên nhận). Bây giờ hãy áp dụng `mpsc` và
mở rộng đoạn mã trong Listing 16-10 để tạo ra nhiều thread, tất cả đều gửi giá
trị tới cùng một receiver. Chúng ta có thể làm điều này bằng cách clone
transmitter, như được minh họa trong Listing 16-11:

<span class="filename">Filename: src/main.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

<span class="caption">Listing 16-11: Gửi nhiều thông điệp từ nhiều producer</span>

Lần này, trước khi tạo thread được spawn đầu tiên, chúng ta gọi `clone` trên
transmitter. Việc này sẽ tạo ra một transmitter mới để chúng ta truyền vào
thread được spawn thứ nhất. Transmitter gốc sẽ được truyền cho thread được spawn
thứ hai. Kết quả là chúng ta có hai thread, mỗi thread gửi các thông điệp khác
nhau tới cùng một receiver.

Khi bạn chạy đoạn mã, output sẽ trông tương tự như sau:

<!-- Không trích xuất output vì những thay đổi trong output này không đáng kể;
các thay đổi nhiều khả năng là do các thread chạy khác nhau, thay vì do
những thay đổi trong trình biên dịch -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Bạn có thể sẽ thấy các giá trị xuất hiện theo một thứ tự khác, tùy thuộc vào hệ
thống của bạn. Đây chính là điều khiến concurrency vừa thú vị vừa khó khăn. Nếu
bạn thử nghiệm với `thread::sleep`, truyền cho nó các giá trị khác nhau ở các
thread khác nhau, thì mỗi lần chạy chương trình sẽ càng trở nên không xác định
(nondeterministic) hơn và tạo ra output khác nhau mỗi lần.

Bây giờ, sau khi đã xem cách các channel hoạt động, chúng ta hãy cùng xem một
phương pháp concurrency khác.
