## Cách Viết Tests

Bài kiểm tra là các hàm trong Rust dùng để xác minh rằng code không phải kiểm tra hoạt động đúng như mong đợi. Nội dung của các hàm kiểm tra thường thực hiện ba bước sau:

1. Thiết lập dữ liệu hoặc trạng thái cần thiết.
2. Chạy đoạn code bạn muốn kiểm tra.
3. Khẳng định kết quả là như mong đợi.

Hãy xem các tính năng mà Rust cung cấp để viết các bài kiểm tra thực hiện các bước này, bao gồm attribute `test`, một số macro, và attribute `should_panic`.

### Cấu Trúc Của Một Hàm Kiểm Tra

Ở mức đơn giản nhất, một bài kiểm tra trong Rust là một hàm được chú thích với attribute `test`. Attributes là metadata về các phần code Rust; một ví dụ là attribute `derive` mà chúng ta đã dùng với struct trong Chương 5. Để biến một hàm thành hàm kiểm tra, thêm `#[test]` ngay trước `fn`. Khi bạn chạy kiểm tra với lệnh `cargo test`, Rust sẽ xây dựng một binary chạy kiểm tra, chạy các hàm được chú thích và báo cáo xem mỗi hàm kiểm tra pass hay fail.

Mỗi khi chúng ta tạo một dự án thư viện mới với Cargo, một module kiểm tra cùng với một hàm kiểm tra trong đó được tự động tạo sẵn. Module này cung cấp cho bạn một mẫu để viết các bài kiểm tra, nhờ đó bạn không phải tra cứu cấu trúc và cú pháp chính xác mỗi khi bắt đầu dự án mới. Bạn có thể thêm bao nhiêu hàm kiểm tra và module kiểm tra tùy ý.

Chúng ta sẽ khám phá một số khía cạnh về cách hoạt động của kiểm tra bằng cách thử nghiệm với mẫu kiểm tra trước khi thực sự kiểm tra bất kỳ code nào. Sau đó, chúng ta sẽ viết các bài kiểm tra thực tế, gọi các code đã viết và khẳng định rằng hành vi của chúng là đúng.

Hãy tạo một dự án thư viện mới có tên là `adder` dùng để cộng hai số:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

The contents of the *src/lib.rs* file in your `adder` library should look like
Listing 11-1.

<span class="filename">Filename: src/lib.rs</span>

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
cargo test
git co output.txt
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

<span class="caption">Listing 11-1: Module kiểm thử và hàm được tạo tự động bởi `cargo new`</span>

Hiện tại, hãy bỏ qua hai dòng đầu và tập trung vào hàm. Chú ý annotation `#[test]`: attribute này cho biết đây là một hàm kiểm thử, vì vậy test runner sẽ biết xử lý hàm này như một kiểm thử. Chúng ta cũng có thể có các hàm không phải kiểm thử trong module `tests` để thiết lập các kịch bản chung hoặc thực hiện các thao tác chung, vì vậy chúng ta luôn cần chỉ rõ hàm nào là kiểm thử.

Nội dung hàm ví dụ sử dụng macro `assert_eq!` để khẳng định rằng `result`, chứa kết quả của việc cộng 2 và 2, bằng 4. Khẳng định này là một ví dụ về định dạng của một hàm kiểm thử điển hình. Hãy chạy nó để xem kiểm thử này pass.

Lệnh `cargo test` sẽ chạy tất cả các kiểm thử trong dự án của chúng ta, như minh họa trong Listing 11-2.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

<span class="caption">Listing 11-2: Kết quả khi chạy kiểm thử được tạo tự động</span>

Cargo đã biên dịch và chạy kiểm thử. Chúng ta thấy dòng `running 1 test`. Dòng tiếp theo hiển thị tên của hàm kiểm thử được tạo, gọi là `it_works`, và kết quả chạy kiểm thử là `ok`. Tóm tắt tổng thể `test result: ok.` có nghĩa là tất cả các kiểm thử đều pass, và phần `1 passed; 0 failed` cho biết số lượng kiểm thử pass hoặc fail.

Có thể đánh dấu một kiểm thử là ignored để nó không chạy trong một trường hợp cụ thể; phần này sẽ được nói đến trong mục [“Ignoring Some Tests Unless Specifically Requested”][ignoring] sau trong chương. Vì ở đây chúng ta chưa làm điều đó, tóm tắt hiển thị `0 ignored`. Chúng ta cũng có thể truyền một đối số cho lệnh `cargo test` để chỉ chạy các kiểm thử có tên khớp với một chuỗi; điều này gọi là *filtering* và sẽ được nói đến trong mục [“Running a Subset of Tests by Name”][subset]. Ở đây chúng ta cũng chưa lọc các kiểm thử đang chạy, vì vậy cuối tóm tắt hiển thị `0 filtered out`.

Thống kê `0 measured` dành cho các kiểm thử benchmark đo hiệu năng. Các kiểm thử benchmark hiện tại chỉ khả dụng trong Rust nightly. Xem [tài liệu về benchmark tests][bench] để tìm hiểu thêm.

Phần tiếp theo của kết quả kiểm thử, bắt đầu từ `Doc-tests adder`, là kết quả của các kiểm thử tài liệu. Chúng ta chưa có kiểm thử tài liệu nào, nhưng Rust có thể biên dịch bất kỳ ví dụ code nào xuất hiện trong tài liệu API. Tính năng này giúp đồng bộ giữa tài liệu và code! Chúng ta sẽ thảo luận cách viết kiểm thử tài liệu trong mục [“Documentation Comments as Tests”][doc-comments] ở Chương 14. Hiện tại, chúng ta sẽ bỏ qua kết quả `Doc-tests`.

Hãy bắt đầu tùy chỉnh kiểm thử theo nhu cầu của chúng ta. Trước hết, đổi tên hàm `it_works` sang một tên khác, ví dụ `exploration`, như sau:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Sau đó chạy lại `cargo test` lần nữa. Kết quả bây giờ hiển thị `exploration` thay vì `it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Bây giờ chúng ta sẽ thêm một hàm để kiểm tra khác, nhưng lần này sẽ tạo một hàm để kiểm tra thất bại! Hàm kiểm tra sẽ thất bại khi có điều gì đó trong hàm gây ra `panic`. Mỗi hàm kiểm tra được chạy trong một thread mới, và khi thread chính nhận thấy một thread kiểm tra đã kết thúc, hàm kiểm tra đó sẽ được đánh dấu là thất bại. Trong Chương 9, chúng ta đã nói về cách đơn giản nhất để gây `panic` là gọi macro `panic!`.  

Nhập hàm kiểm tra mới dưới dạng một hàm tên là `another`, sao cho file *src/lib.rs* của bạn trông giống như Listing 11-3.

<span class="filename">Filename: src/lib.rs</span>

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs:here}}
```

<span class="caption">Listing 11-3: Thêm một hàm kiểm tra thứ hai sẽ thất bại vì chúng ta gọi macro `panic!`</span>

Chạy lại các hàm kiểm tra bằng cách sử dụng `cargo test`. Kết quả sẽ trông giống như Listing 11-4, cho thấy hàm kiểm tra `exploration` của chúng ta đã thành công và `another` thất bại.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

<span class="caption">Listing 11-4: Kết quả kiểm tra khi một hàm kiểm tra thành công và một hàm kiểm tra thất bại</span>

Thay vì hiển thị `ok`, dòng `test tests::another` sẽ hiển thị `FAILED`. Hai phần mới xuất hiện giữa kết quả từng hàm kiểm tra và phần tóm tắt: phần đầu tiên hiển thị lý do chi tiết cho mỗi lỗi của hàm kiểm tra. Trong trường hợp này, chúng ta nhận được chi tiết rằng hàm `another` thất bại vì nó `panicked at 'Make this test fail'` ở dòng 10 trong file *src/lib.rs*. Phần tiếp theo chỉ liệt kê tên của tất cả các hàm kiểm tra thất bại, điều này hữu ích khi có nhiều hàm kiểm tra và nhiều kết quả lỗi chi tiết. Chúng ta có thể sử dụng tên của hàm kiểm tra thất bại để chạy riêng hàm đó, giúp gỡ lỗi dễ hơn; chúng ta sẽ bàn kỹ hơn về các cách chạy kiểm tra trong phần [“Controlling How Tests Are Run”][controlling-how-tests-are-run]<!-- ignore -->.

Dòng tóm tắt xuất hiện ở cuối: tổng thể, kết quả kiểm tra của chúng ta là `FAILED`. Chúng ta có một hàm kiểm tra thành công và một hàm kiểm tra thất bại.

Bây giờ khi bạn đã thấy kết quả kiểm tra trông như thế nào trong các kịch bản khác nhau, hãy cùng xem một số macro khác ngoài `panic!` mà hữu ích trong các hàm kiểm tra.

### Kiểm tra kết quả với macro `assert!`

Macro `assert!`, được cung cấp bởi thư viện chuẩn, hữu ích khi bạn muốn đảm bảo rằng một điều kiện trong hàm kiểm tra đánh giá là `true`. Chúng ta truyền cho macro `assert!` một biểu thức trả về Boolean. Nếu giá trị là `true`, không có gì xảy ra và hàm kiểm tra thành công. Nếu giá trị là `false`, macro `assert!` gọi `panic!` để làm cho hàm kiểm tra thất bại. Sử dụng macro `assert!` giúp chúng ta kiểm tra rằng mã của mình hoạt động theo đúng ý định.

Trong Chương 5, Listing 5-15, chúng ta đã sử dụng struct `Rectangle` và phương thức `can_hold`, được lặp lại ở đây trong Listing 11-5. Hãy đặt đoạn mã này vào file *src/lib.rs*, sau đó viết một số hàm kiểm tra cho nó bằng cách sử dụng macro `assert!`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs:here}}
```

<span class="caption">Listing 11-5: Sử dụng struct `Rectangle` và phương thức `can_hold` từ Chương 5</span>

Phương thức `can_hold` trả về một giá trị Boolean, điều này có nghĩa là nó là một trường hợp sử dụng hoàn hảo cho macro `assert!`. Trong Listing 11-6, chúng ta viết một hàm kiểm tra để thử phương thức `can_hold` bằng cách tạo một instance của `Rectangle` có chiều rộng 8 và chiều cao 7, và xác nhận rằng nó có thể chứa một instance khác của `Rectangle` có chiều rộng 5 và chiều cao 1.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

<span class="caption">Listing 11-6: Một hàm kiểm tra `can_hold` xác định xem một hình chữ nhật lớn hơn có thể chứa một hình chữ nhật nhỏ hơn hay không</span>

Lưu ý rằng chúng ta đã thêm một dòng mới bên trong module `tests`: `use super::*;`. Module `tests` là một module bình thường tuân theo các quy tắc hiển thị thông thường mà chúng ta đã đề cập trong Chương 7 ở phần [“Paths for Referring to an Item in the Module Tree”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->. Vì module `tests` là một inner module, chúng ta cần đưa mã đang kiểm tra trong module bên ngoài vào phạm vi của inner module. Ở đây chúng ta sử dụng glob để bất kỳ thứ gì được định nghĩa trong module bên ngoài cũng có sẵn cho module `tests`.

Chúng ta đặt tên cho hàm kiểm tra là `larger_can_hold_smaller`, và đã tạo hai instance của `Rectangle` cần thiết. Sau đó chúng ta gọi macro `assert!` và truyền vào kết quả của việc gọi `larger.can_hold(&smaller)`. Biểu thức này dự kiến trả về `true`, vì vậy hàm kiểm tra của chúng ta nên thành công. Hãy cùng kiểm tra!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Nó đã thành công! Bây giờ hãy thêm một hàm kiểm tra khác, lần này xác nhận rằng một hình chữ nhật nhỏ hơn **không thể** chứa một hình chữ nhật lớn hơn:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Bởi vì kết quả đúng của hàm `can_hold` trong trường hợp này là `false`, chúng ta cần phủ định kết quả đó trước khi truyền nó vào macro `assert!`. Kết quả là, hàm kiểm tra của chúng ta sẽ thành công nếu `can_hold` trả về `false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Hai hàm kiểm tra đều thành công! Bây giờ hãy xem điều gì sẽ xảy ra với kết quả kiểm tra khi chúng ta giới thiệu một lỗi trong mã của mình. Chúng ta sẽ thay đổi cài đặt của phương thức `can_hold` bằng cách thay dấu lớn hơn (`>`) bằng dấu nhỏ hơn (`<`) khi so sánh chiều rộng:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Chạy các hàm kiểm tra bây giờ sẽ cho kết quả như sau:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Các hàm kiểm tra của chúng ta đã phát hiện ra lỗi! Bởi vì `larger.width` là 8 và `smaller.width` là 5, so sánh chiều rộng trong `can_hold` bây giờ trả về `false`: 8 không nhỏ hơn 5.

### Kiểm tra bằng macro `assert_eq!` và `assert_ne!`

Một cách phổ biến để xác minh chức năng là kiểm tra sự bằng nhau giữa kết quả của mã đang kiểm tra và giá trị mà bạn mong mã trả về. Bạn có thể làm điều này bằng cách sử dụng macro `assert!` và truyền cho nó một biểu thức sử dụng toán tử `==`. Tuy nhiên, đây là một kiểu kiểm tra rất phổ biến nên thư viện chuẩn cung cấp một cặp macro — `assert_eq!` và `assert_ne!` — để thực hiện kiểm tra này thuận tiện hơn. Các macro này so sánh hai đối số để kiểm tra bằng nhau hoặc không bằng nhau, tương ứng. Chúng cũng sẽ in ra hai giá trị nếu phép kiểm tra thất bại, giúp dễ dàng nhận thấy *tại sao* hàm kiểm tra thất bại; ngược lại, macro `assert!` chỉ cho biết rằng biểu thức `==` trả về `false`, mà không in ra các giá trị dẫn đến `false`.

Trong Listing 11-7, chúng ta viết một hàm tên là `add_two` để cộng `2` vào tham số của nó, sau đó kiểm tra hàm này bằng macro `assert_eq!`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

<span class="caption">Listing 11-7: Kiểm tra hàm `add_two` bằng macro `assert_eq!`</span>

Hãy kiểm tra xem nó có thành công không!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Chúng ta truyền `4` làm đối số cho `assert_eq!`, bằng với kết quả của việc gọi `add_two(2)`. Dòng cho hàm kiểm tra này là `test tests::it_adds_two ... ok`, và chữ `ok` cho thấy hàm kiểm tra của chúng ta đã thành công!

Hãy giới thiệu một lỗi vào mã của chúng ta để xem `assert_eq!` trông như thế nào khi thất bại. Thay đổi cài đặt của hàm `add_two` để thay vào đó cộng `3`:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Chạy lại các hàm kiểm tra:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Các hàm kiểm tra của chúng ta đã phát hiện ra lỗi! Hàm kiểm tra `it_adds_two` thất bại, và thông báo cho chúng ta biết phép kiểm tra thất bại là `` assertion failed: `(left == right)` `` cùng với giá trị của `left` và `right`. Thông báo này giúp chúng ta bắt đầu gỡ lỗi: đối số `left` là `4` nhưng đối số `right`, nơi chúng ta gọi `add_two(2)`, là `5`. Bạn có thể tưởng tượng điều này đặc biệt hữu ích khi chúng ta có nhiều hàm kiểm tra.

Lưu ý rằng trong một số ngôn ngữ và framework kiểm tra, các tham số của hàm kiểm tra bằng nhau được gọi là `expected` và `actual`, và thứ tự truyền đối số rất quan trọng. Tuy nhiên, trong Rust, chúng được gọi là `left` và `right`, và thứ tự chúng ta chỉ định giá trị mong đợi và giá trị mà mã trả về **không quan trọng**. Chúng ta có thể viết phép kiểm tra trong ví dụ này là `assert_eq!(add_two(2), 4)`, và kết quả sẽ giống nhau với thông báo thất bại `` assertion failed: `(left == right)` ``.

Macro `assert_ne!` sẽ thành công nếu hai giá trị không bằng nhau và thất bại nếu chúng bằng nhau. Macro này hữu ích nhất khi chúng ta không chắc giá trị *sẽ* là gì, nhưng chúng ta biết giá trị đó chắc chắn *không nên* là gì. Ví dụ, nếu chúng ta kiểm tra một hàm đảm bảo thay đổi đầu vào theo một cách nào đó, nhưng cách thay đổi phụ thuộc vào ngày trong tuần khi chạy kiểm tra, điều tốt nhất để kiểm tra có thể là đầu ra của hàm không bằng đầu vào.

Ở mức cơ bản, các macro `assert_eq!` và `assert_ne!` sử dụng các toán tử `==` và `!=`, tương ứng. Khi các phép kiểm tra thất bại, các macro này in các đối số bằng **debug formatting**, nghĩa là các giá trị được so sánh phải triển khai các trait `PartialEq` và `Debug`. Tất cả các kiểu nguyên thủy và hầu hết các kiểu trong thư viện chuẩn đều triển khai các trait này. Đối với các struct và enum do bạn định nghĩa, bạn cần triển khai `PartialEq` để kiểm tra sự bằng nhau của các kiểu đó. Bạn cũng cần triển khai `Debug` để in giá trị khi phép kiểm tra thất bại. Vì cả hai trait đều có thể được derive, như đã đề cập trong Listing 5-12 ở Chương 5, điều này thường đơn giản chỉ cần thêm annotation `#[derive(PartialEq, Debug)]` vào định nghĩa struct hoặc enum của bạn. Xem Phụ lục C, [“Derivable Traits,”][derivable-traits]<!-- ignore --> để biết thêm chi tiết về các trait có thể derive và các trait khác.

### Thêm thông báo lỗi tùy chỉnh

Bạn cũng có thể thêm thông báo tùy chỉnh được in cùng với thông báo thất bại như các đối số tùy chọn cho các macro `assert!`, `assert_eq!`, và `assert_ne!`. Bất kỳ đối số nào được chỉ định sau các đối số bắt buộc sẽ được chuyển cho macro `format!` (được thảo luận trong Chương 8 ở phần [“Concatenation with the `+` Operator or the `format!` Macro”][concatenation-with-the--operator-or-the-format-macro]<!-- ignore -->), vì vậy bạn có thể truyền một chuỗi định dạng chứa các placeholder `{}` và các giá trị để điền vào đó. Thông báo tùy chỉnh hữu ích để ghi chú ý nghĩa của một phép kiểm tra; khi một hàm kiểm tra thất bại, bạn sẽ dễ dàng biết vấn đề là gì trong mã.

Ví dụ, giả sử chúng ta có một hàm chào người theo tên và muốn kiểm tra rằng tên truyền vào hàm xuất hiện trong đầu ra:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Các yêu cầu cho chương trình này vẫn chưa được thống nhất, và chúng ta khá chắc rằng đoạn văn bản `Hello` ở đầu lời chào sẽ thay đổi. Chúng ta quyết định không muốn phải cập nhật hàm kiểm tra khi các yêu cầu thay đổi, vì vậy thay vì kiểm tra sự bằng nhau chính xác với giá trị trả về từ hàm `greeting`, chúng ta chỉ xác nhận rằng đầu ra chứa văn bản của tham số đầu vào.

Bây giờ hãy giới thiệu một lỗi vào mã này bằng cách thay đổi hàm `greeting` để bỏ qua `name` để xem thông báo thất bại mặc định của hàm kiểm tra trông như thế nào:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Chạy hàm kiểm tra này sẽ cho kết quả như sau:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Kết quả này chỉ cho biết rằng phép kiểm tra thất bại và dòng chứa phép kiểm tra đó. Một thông báo thất bại hữu ích hơn sẽ in giá trị trả về từ hàm `greeting`. Hãy thêm một thông báo thất bại tùy chỉnh gồm một chuỗi định dạng với placeholder được điền bằng giá trị thực tế mà chúng ta nhận được từ hàm `greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Bây giờ khi chúng ta chạy hàm kiểm tra, chúng ta sẽ nhận được một thông báo lỗi chi tiết hơn:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Chúng ta có thể thấy giá trị thực tế mà chúng ta nhận được trong kết quả kiểm tra, điều này sẽ giúp gỡ lỗi xem chuyện gì đã xảy ra thay vì những gì chúng ta mong đợi.

### Kiểm tra panic với `should_panic`

Ngoài việc kiểm tra giá trị trả về, việc kiểm tra rằng mã của chúng ta xử lý các điều kiện lỗi như mong đợi cũng rất quan trọng. Ví dụ, hãy xem xét kiểu `Guess` mà chúng ta đã tạo trong Chương 9, Listing 9-13. Các đoạn mã khác sử dụng `Guess` phụ thuộc vào đảm bảo rằng các instance của `Guess` chỉ chứa các giá trị từ 1 đến 100. Chúng ta có thể viết một hàm kiểm tra để đảm bảo rằng việc cố gắng tạo một instance của `Guess` với giá trị ngoài khoảng đó sẽ gây panic.

Chúng ta làm điều này bằng cách thêm attribute `should_panic` vào hàm kiểm tra. Hàm kiểm tra sẽ thành công nếu mã bên trong hàm gây panic; hàm kiểm tra sẽ thất bại nếu mã bên trong hàm không gây panic.

Listing 11-8 hiển thị một hàm kiểm tra đảm bảo rằng các điều kiện lỗi của `Guess::new` xảy ra khi chúng ta mong đợi.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

<span class="caption">Listing 11-8: Kiểm tra rằng một điều kiện sẽ gây ra `panic!`</span>

Chúng ta đặt attribute `#[should_panic]` sau attribute `#[test]` và trước hàm kiểm tra mà nó áp dụng. Hãy xem kết quả khi hàm kiểm tra này thành công:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Trông ổn rồi! Bây giờ hãy giới thiệu một lỗi vào mã của chúng ta bằng cách loại bỏ điều kiện mà hàm `new` sẽ panic nếu giá trị lớn hơn 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Khi chúng ta chạy hàm kiểm tra trong Listing 11-8, nó sẽ thất bại:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Trong trường hợp này, chúng ta không nhận được thông báo hữu ích lắm, nhưng khi xem hàm kiểm tra, chúng ta thấy nó được chú thích bằng `#[should_panic]`. Thất bại mà chúng ta nhận được có nghĩa là mã trong hàm kiểm tra không gây ra panic.

Các hàm kiểm tra sử dụng `should_panic` có thể không chính xác. Một hàm kiểm tra `should_panic` sẽ vẫn thành công ngay cả khi panic xảy ra vì lý do khác với lý do mà chúng ta mong đợi. Để làm cho các hàm kiểm tra `should_panic` chính xác hơn, chúng ta có thể thêm một tham số tùy chọn `expected` vào attribute `should_panic`. Trình chạy kiểm tra sẽ đảm bảo rằng thông báo thất bại chứa văn bản được cung cấp. Ví dụ, hãy xem đoạn mã `Guess` đã được chỉnh sửa trong Listing 11-9, nơi hàm `new` panic với các thông điệp khác nhau tùy thuộc vào việc giá trị quá nhỏ hay quá lớn.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

<span class="caption">Listing 11-9: Kiểm tra `panic!` với thông báo panic chứa một chuỗi con xác định</span>

Hàm kiểm tra này sẽ thành công vì giá trị mà chúng ta đặt trong tham số `expected` của attribute `should_panic` là một chuỗi con của thông báo mà hàm `Guess::new` panic. Chúng ta có thể chỉ định toàn bộ thông báo panic mà chúng ta mong đợi, trong trường hợp này sẽ là `Guess value must be less than or equal to 100, got 200.` Việc bạn chọn chỉ định cái gì phụ thuộc vào việc phần nào của thông báo panic là duy nhất hoặc thay đổi và mức độ chính xác mà bạn muốn cho hàm kiểm tra. Trong trường hợp này, một chuỗi con của thông báo panic là đủ để đảm bảo rằng mã trong hàm kiểm tra thực thi nhánh `else if value > 100`.

Để xem điều gì xảy ra khi một hàm kiểm tra `should_panic` với thông báo `expected` thất bại, hãy lại giới thiệu một lỗi vào mã của chúng ta bằng cách hoán đổi nội dung của các khối `if value < 1` và `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Lần này khi chúng ta chạy hàm kiểm tra `should_panic`, nó sẽ thất bại:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Thông báo thất bại cho thấy hàm kiểm tra này thực sự đã panic như chúng ta mong đợi, nhưng thông báo panic không chứa chuỗi mong đợi `'Guess value must be less than or equal to 100'`. Thông báo panic mà chúng ta nhận được trong trường hợp này là `Guess value must be greater than or equal to 1, got 200.` Bây giờ chúng ta có thể bắt đầu xác định vị trí lỗi của mình!

### Sử dụng `Result<T, E>` trong các hàm kiểm tra

Cho đến nay các hàm kiểm tra của chúng ta đều panic khi thất bại. Chúng ta cũng có thể viết các hàm kiểm tra sử dụng `Result<T, E>`! Đây là hàm kiểm tra từ Listing 11-1, được viết lại để sử dụng `Result<T, E>` và trả về `Err` thay vì panic:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs}}
```

Hàm `it_works` bây giờ có kiểu trả về `Result<(), String>`. Trong thân hàm, thay vì gọi macro `assert_eq!`, chúng ta trả về `Ok(())` khi hàm kiểm tra thành công và một `Err` chứa `String` khi hàm kiểm tra thất bại.

Việc viết các hàm kiểm tra trả về `Result<T, E>` cho phép bạn sử dụng toán tử dấu hỏi `?` trong thân hàm kiểm tra, đây có thể là một cách tiện lợi để viết các hàm kiểm tra mà sẽ thất bại nếu bất kỳ thao tác nào bên trong trả về biến thể `Err`.

Bạn không thể sử dụng annotation `#[should_panic]` trên các hàm kiểm tra sử dụng `Result<T, E>`. Để kiểm tra rằng một thao tác trả về biến thể `Err`, không sử dụng toán tử dấu hỏi `?` trên giá trị `Result<T, E>`. Thay vào đó, hãy dùng `assert!(value.is_err())`.

Bây giờ khi bạn đã biết nhiều cách để viết các hàm kiểm tra, hãy xem điều gì xảy ra khi chúng ta chạy các hàm kiểm tra và khám phá các tùy chọn khác nhau mà chúng ta có thể sử dụng với `cargo test`.

[concatenation-with-the--operator-or-the-format-macro]:
ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]:
ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
