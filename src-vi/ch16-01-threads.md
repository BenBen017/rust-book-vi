## Sử Dụng Luồng để Chạy Mã Đồng Thời

Trong hầu hết các hệ điều hành hiện nay, mã của một chương trình khi thực thi được chạy trong một *process*, và hệ điều hành sẽ quản lý nhiều process cùng lúc. Trong một chương trình, bạn cũng có thể có các phần độc lập chạy đồng thời. Các tính năng chạy các phần độc lập này được gọi là *threads* (luồng). Ví dụ, một web server có thể có nhiều luồng để có thể phản hồi nhiều yêu cầu cùng một lúc.

Chia việc tính toán trong chương trình của bạn thành nhiều luồng để chạy nhiều tác vụ cùng một lúc có thể cải thiện hiệu suất, nhưng cũng làm tăng độ phức tạp. Vì các luồng có thể chạy đồng thời, không có đảm bảo nào về thứ tự mà các phần mã trên các luồng khác nhau sẽ chạy. Điều này có thể dẫn đến các vấn đề như:

* Race conditions, nơi các luồng truy cập dữ liệu hoặc tài nguyên theo thứ tự không nhất quán
* Deadlocks, nơi hai luồng chờ lẫn nhau, khiến cả hai luồng không thể tiếp tục
* Lỗi chỉ xảy ra trong một số tình huống nhất định và khó tái tạo cũng như sửa chữa một cách đáng tin cậy

Rust cố gắng giảm thiểu các tác động tiêu cực khi sử dụng luồng, nhưng lập trình trong bối cảnh đa luồng vẫn đòi hỏi suy nghĩ cẩn thận và yêu cầu cấu trúc mã khác với các chương trình chạy trong một luồng đơn.

Các ngôn ngữ lập trình triển khai luồng theo một vài cách khác nhau, và nhiều hệ điều hành cung cấp một API mà ngôn ngữ có thể gọi để tạo luồng mới. Thư viện chuẩn của Rust sử dụng mô hình *1:1* trong việc triển khai luồng, theo đó một chương trình sử dụng một luồng hệ điều hành cho mỗi luồng trong ngôn ngữ. Có các crate triển khai các mô hình luồng khác, đưa ra các đánh đổi khác so với mô hình 1:1.

### Tạo Luồng Mới với `spawn`

Để tạo một luồng mới, chúng ta gọi hàm `thread::spawn` 
và truyền vào một closure (chúng ta đã nói về closures trong Chương 13) 
chứa mã mà chúng ta muốn chạy trong luồng mới. 
Ví dụ trong Listing 16-1 in ra một số văn bản từ luồng chính và văn bản khác từ một luồng mới:


<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

<span class="caption">Listing 16-1: Tạo một luồng mới để in một thứ trong khi luồng chính in thứ khác</span>

Lưu ý rằng khi luồng chính của một chương trình Rust hoàn tất, tất cả các luồng được sinh ra sẽ bị đóng, bất kể chúng đã chạy xong hay chưa. Đầu ra từ chương trình này có thể hơi khác nhau mỗi lần chạy, nhưng sẽ trông tương tự như sau:

<!-- Không trích xuất đầu ra vì các thay đổi trong đầu ra này không quan trọng; các thay đổi có khả năng do các luồng chạy khác nhau chứ không phải do thay đổi trong trình biên dịch -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Các lần gọi `thread::sleep` buộc một luồng phải tạm dừng thực thi trong một khoảng thời gian ngắn, cho phép luồng khác chạy. Các luồng có thể sẽ chạy lần lượt, nhưng điều đó không được đảm bảo: nó phụ thuộc vào cách hệ điều hành của bạn lập lịch các luồng. Trong lần chạy này, luồng chính in ra trước, mặc dù câu lệnh in từ luồng được sinh ra xuất hiện trước trong mã. Và mặc dù chúng ta bảo luồng được sinh ra in cho đến khi `i` là 9, nó chỉ in được đến 5 trước khi luồng chính kết thúc.

Nếu bạn chạy mã này và chỉ thấy đầu ra từ luồng chính, hoặc không thấy sự chồng lấn nào, hãy thử tăng các con số trong các phạm vi để tạo thêm cơ hội cho hệ điều hành chuyển đổi giữa các luồng.

### Chờ tất cả các luồng hoàn thành bằng cách sử dụng `Join` Handles

Mã trong Listing 16-1 không chỉ dừng luồng được sinh ra sớm hơn hầu hết thời gian do luồng chính kết thúc, mà vì không có đảm bảo về thứ tự các luồng chạy, chúng ta cũng không thể đảm bảo rằng luồng được sinh ra sẽ chạy được chút nào!

Chúng ta có thể giải quyết vấn đề luồng được sinh ra không chạy hoặc kết thúc sớm bằng cách lưu giá trị trả về của `thread::spawn` vào một biến. Kiểu trả về của `thread::spawn` là `JoinHandle`. Một `JoinHandle` là một giá trị sở hữu mà khi chúng ta gọi phương thức `join` trên nó, sẽ chờ cho luồng của nó hoàn thành. Listing 16-2 minh họa cách sử dụng `JoinHandle` của luồng chúng ta tạo trong Listing 16-1 và gọi `join` để đảm bảo luồng được sinh ra kết thúc trước khi `main` thoát:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

<span class="caption">Listing 16-2: Lưu `JoinHandle` từ `thread::spawn`
để đảm bảo luồng được chạy đến khi hoàn thành</span>

Gọi `join` trên handle sẽ chặn luồng đang chạy cho đến khi luồng được handle đại diện kết thúc. *Chặn* một luồng có nghĩa là luồng đó bị ngăn không thực hiện công việc hoặc thoát. Vì chúng ta đặt lời gọi `join` sau vòng lặp `for` của luồng chính, chạy Listing 16-2 sẽ tạo ra đầu ra tương tự như sau:

<!-- Không trích xuất đầu ra vì các thay đổi đối với đầu ra này không quan trọng;
các thay đổi có khả năng do các luồng chạy khác nhau thay vì do thay đổi trong trình biên dịch -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Hai luồng tiếp tục xen kẽ, nhưng luồng chính sẽ chờ vì lời gọi `handle.join()` và không kết thúc cho đến khi luồng được spawn hoàn tất.

Nhưng hãy xem điều gì xảy ra nếu chúng ta chuyển `handle.join()` lên trước vòng lặp `for` trong `main`, như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

Luồng chính sẽ chờ cho luồng được spawn hoàn tất, sau đó mới chạy vòng lặp `for` của nó, vì vậy kết quả sẽ không còn xen kẽ nữa, như minh họa dưới đây:

<!-- Không trích xuất output vì các thay đổi của output này không quan trọng;
các thay đổi có thể do các luồng chạy khác nhau hơn là do thay đổi trong compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Những chi tiết nhỏ, chẳng hạn như nơi gọi `join`, có thể ảnh hưởng đến việc các luồng của bạn có chạy cùng lúc hay không.

### Sử dụng Closure với `move` trong Threads

Chúng ta thường sử dụng từ khóa `move` với các closure được truyền vào `thread::spawn` vì khi đó closure sẽ nhận quyền sở hữu các giá trị mà nó sử dụng từ môi trường, do đó chuyển quyền sở hữu các giá trị đó từ một luồng sang luồng khác. Trong phần [“Capturing References or Moving Ownership”][capture]<!-- ignore --> của Chương 13, chúng ta đã thảo luận về `move` trong bối cảnh của closures. Bây giờ, chúng ta sẽ tập trung hơn vào sự tương tác giữa `move` và `thread::spawn`.

Chú ý trong Listing 16-1 rằng closure mà chúng ta truyền vào `thread::spawn` không nhận tham số nào: chúng ta không sử dụng bất kỳ dữ liệu nào từ luồng chính trong code của luồng được spawn. Để sử dụng dữ liệu từ luồng chính trong luồng được spawn, closure của luồng được spawn phải capture các giá trị mà nó cần. Listing 16-3 cho thấy một cố gắng tạo một vector trong luồng chính và sử dụng nó trong luồng được spawn. Tuy nhiên, điều này chưa hoạt động, như bạn sẽ thấy ngay sau đây.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

<span class="caption">Listing 16-3: Cố gắng sử dụng một vector được tạo bởi luồng chính trong một luồng khác</span>

Closure sử dụng `v`, do đó nó sẽ capture `v` và làm nó trở thành một phần của môi trường của closure. Vì `thread::spawn` chạy closure này trong một luồng mới, chúng ta sẽ mong muốn có thể truy cập `v` bên trong luồng mới đó. Nhưng khi biên dịch ví dụ này, chúng ta nhận được lỗi sau:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust *suy diễn* cách capture `v`, và vì `println!` chỉ cần một tham chiếu đến `v`, closure cố gắng mượn `v`. Tuy nhiên, có một vấn đề: Rust không thể biết luồng được spawn sẽ chạy trong bao lâu, nên nó không biết liệu tham chiếu đến `v` có luôn hợp lệ hay không.

Listing 16-4 đưa ra một kịch bản có khả năng cao hơn là tham chiếu đến `v` sẽ không hợp lệ:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

<span class="caption">Listing 16-4: Một luồng với closure cố gắng capture tham chiếu đến `v` từ luồng chính, nơi `v` đã bị drop</span>

Nếu Rust cho phép chúng ta chạy đoạn mã này, có khả năng luồng được spawn sẽ ngay lập tức bị đặt vào nền mà không chạy gì cả. Luồng được spawn có một tham chiếu đến `v` bên trong, nhưng luồng chính ngay lập tức drop `v`, sử dụng hàm `drop` mà chúng ta đã thảo luận trong Chương 15. Sau đó, khi luồng được spawn bắt đầu thực thi, `v` không còn hợp lệ nữa, nên tham chiếu đến nó cũng không hợp lệ. Ôi không!

Để sửa lỗi compiler trong Listing 16-3, chúng ta có thể dùng lời khuyên từ thông báo lỗi:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Bằng cách thêm từ khóa `move` trước closure, chúng ta buộc closure nhận quyền sở hữu của các giá trị mà nó sử dụng thay vì để Rust suy đoán rằng nó nên mượn các giá trị đó. Sự chỉnh sửa đối với Listing 16-3 được thể hiện trong Listing 16-5 sẽ biên dịch và chạy như chúng ta mong muốn:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

Chúng ta có thể bị cám dỗ để thử làm điều tương tự để sửa mã trong Listing 16-4, 
nơi luồng chính gọi `drop`, bằng cách sử dụng một closure `move`. 
Tuy nhiên, cách sửa này sẽ không hoạt động vì những gì Listing 16-4 
đang cố gắng làm bị cấm vì một lý do khác. Nếu chúng ta thêm `move` vào closure, 
chúng ta sẽ chuyển `v` vào môi trường của closure, và sẽ không thể gọi `drop` 
trên nó trong luồng chính nữa. Thay vào đó, chúng ta sẽ nhận được lỗi biên dịch như sau:


```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Các quy tắc sở hữu của Rust lại cứu chúng ta một lần nữa! Chúng ta nhận được lỗi từ mã trong Listing 16-3 vì Rust thận trọng và chỉ mượn `v` cho luồng, điều này có nghĩa là luồng chính về lý thuyết có thể làm cho tham chiếu trong luồng spawn trở nên không hợp lệ. Bằng cách bảo Rust chuyển quyền sở hữu `v` sang luồng spawn, chúng ta đảm bảo với Rust rằng luồng chính sẽ không sử dụng `v` nữa. Nếu chúng ta thay đổi Listing 16-4 theo cách tương tự, chúng ta sẽ vi phạm quy tắc sở hữu khi cố gắng sử dụng `v` trong luồng chính. Từ khóa `move` ghi đè mặc định thận trọng của Rust là mượn; nó không cho phép chúng ta vi phạm các quy tắc sở hữu.

Với hiểu biết cơ bản về luồng và API của luồng, hãy xem chúng ta có thể *làm gì* với các luồng.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
