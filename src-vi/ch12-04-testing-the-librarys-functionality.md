## Phát Triển Chức Năng Của Thư Viện Với Phát Triển Theo Kiểm Tra (TDD)

Bây giờ chúng ta đã tách logic vào *src/lib.rs* và giữ phần thu thập đối số và xử lý lỗi trong *src/main.rs*, việc viết test cho chức năng cốt lõi của mã trở nên dễ dàng hơn nhiều. Chúng ta có thể gọi trực tiếp các hàm với các đối số khác nhau và kiểm tra giá trị trả về mà không cần phải gọi binary từ dòng lệnh.

Trong phần này, chúng ta sẽ thêm logic tìm kiếm vào chương trình `minigrep` bằng quy trình phát triển theo kiểm tra (test-driven development, TDD) với các bước sau:

1. Viết một test mà sẽ thất bại và chạy nó để đảm bảo nó thất bại vì lý do bạn mong đợi.
2. Viết hoặc sửa đổi đủ mã để test mới này thành công.
3. Refactor mã vừa thêm hoặc thay đổi và đảm bảo các test vẫn chạy đúng.
4. Lặp lại từ bước 1!

Mặc dù đây chỉ là một trong nhiều cách viết phần mềm, TDD có thể giúp định hướng thiết kế mã. Viết test trước khi viết mã làm test đó thành công giúp duy trì độ bao phủ test cao trong suốt quá trình.

Chúng ta sẽ viết test để điều khiển việc triển khai chức năng thực sự sẽ tìm kiếm chuỗi query trong nội dung file và tạo ra danh sách các dòng khớp với query. Chúng ta sẽ thêm chức năng này trong một hàm có tên `search`.

### Viết Một Test Thất Bại

Vì chúng ta không còn cần chúng nữa, hãy loại bỏ các câu lệnh `println!` từ *src/lib.rs* và *src/main.rs* mà chúng ta đã dùng để kiểm tra hành vi của chương trình. Sau đó, trong *src/lib.rs*, thêm một module `tests` với một hàm test, như chúng ta đã làm trong [Chương 11][ch11-anatomy]<!-- ignore -->. Hàm test chỉ định hành vi mà chúng ta muốn hàm `search` có: nó sẽ nhận một query và văn bản cần tìm kiếm, và trả về chỉ các dòng từ văn bản chứa query. Listing 12-15 trình bày test này, mã này hiện chưa biên dịch được.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

<span class="caption">Listing 12-15: Tạo một test thất bại cho hàm `search` mà chúng ta mong muốn có</span>

Test này tìm kiếm chuỗi `"duct"`. Văn bản mà chúng ta đang tìm kiếm có ba dòng, chỉ có một dòng chứa `"duct"` (Lưu ý rằng dấu gạch chéo ngược `\` sau dấu ngoặc kép mở cho Rust biết không chèn ký tự newline ở đầu nội dung của literal chuỗi này). Chúng ta khẳng định rằng giá trị trả về từ hàm `search` chỉ chứa dòng mà chúng ta mong đợi.

Hiện tại chúng ta chưa thể chạy test này và xem nó thất bại vì test thậm chí còn chưa biên dịch được: hàm `search` vẫn chưa tồn tại! Theo nguyên tắc TDD, chúng ta sẽ thêm đủ mã để test có thể biên dịch và chạy bằng cách thêm định nghĩa hàm `search` luôn trả về một vector rỗng, như trong Listing 12-16. Khi đó, test sẽ biên dịch nhưng thất bại vì một vector rỗng không khớp với vector chứa dòng `"safe, fast, productive."`

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

<span class="caption">Listing 12-16: Định nghĩa đủ hàm `search` để test của chúng ta có thể biên dịch</span>

Lưu ý rằng chúng ta cần định nghĩa một lifetime rõ ràng `'a` trong chữ ký của hàm `search` và sử dụng lifetime đó với tham số `contents` và giá trị trả về. Nhớ lại trong [Chương 10][ch10-lifetimes]<!-- ignore --> rằng các tham số lifetime chỉ ra lifetime của tham số nào được kết nối với lifetime của giá trị trả về. Trong trường hợp này, chúng ta chỉ ra rằng vector trả về nên chứa các string slice tham chiếu đến các slice của tham số `contents` (thay vì tham số `query`).

Nói cách khác, chúng ta bảo Rust rằng dữ liệu trả về từ hàm `search` sẽ sống lâu bằng dữ liệu được truyền vào hàm `search` thông qua tham số `contents`. Điều này rất quan trọng! Dữ liệu mà một slice tham chiếu *bởi* nó cần phải hợp lệ để tham chiếu hợp lệ; nếu compiler giả sử chúng ta tạo các string slice từ `query` thay vì `contents`, nó sẽ kiểm tra an toàn sai cách.

Nếu chúng ta quên các annotation lifetime và cố biên dịch hàm này, chúng ta sẽ nhận được lỗi sau:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust không thể biết được chúng ta cần dùng argument nào trong hai argument, vì vậy chúng ta cần chỉ rõ cho nó. Vì `contents` là argument chứa toàn bộ văn bản và chúng ta muốn trả về các phần của văn bản đó mà khớp với query, nên chúng ta biết `contents` là argument cần được kết nối với giá trị trả về bằng cú pháp lifetime.

Các ngôn ngữ lập trình khác không yêu cầu bạn kết nối argument với giá trị trả về trong chữ ký hàm, nhưng thói quen này sẽ trở nên dễ dàng hơn theo thời gian. Bạn có thể so sánh ví dụ này với phần [“Validating References with Lifetimes”][validating-references-with-lifetimes]<!-- ignore --> trong Chương 10.

Bây giờ hãy chạy test:

```console
{{#include ../listings/ch12-an-io-project/listing-12-16/output.txt}}
```

Tuyệt! Test thất bại, chính xác như chúng ta mong đợi. Bây giờ hãy viết mã để test thành công!

### Viết Mã Để Test Thành Công

Hiện tại, test của chúng ta thất bại vì chúng ta luôn trả về một vector rỗng. Để sửa lỗi đó và triển khai `search`, chương trình của chúng ta cần thực hiện các bước sau:

* Lặp qua từng dòng của nội dung.
* Kiểm tra xem dòng đó có chứa chuỗi query không.
* Nếu có, thêm nó vào danh sách các giá trị sẽ trả về.
* Nếu không, không làm gì.
* Trả về danh sách các kết quả khớp.

Hãy thực hiện từng bước, bắt đầu với việc lặp qua các dòng.

#### Lặp Qua Các Dòng Với Phương Thức `lines`

Rust có một phương thức hữu ích để lặp từng dòng của chuỗi, tên là `lines`, hoạt động như trong Listing 12-17. Lưu ý rằng mã này chưa biên dịch được.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

<span class="caption">Listing 12-17: Lặp qua từng dòng trong `contents`</span>

Phương thức `lines` trả về một iterator. Chúng ta sẽ bàn kỹ về iterator trong [Chương 13][ch13-iterators]<!-- ignore -->, nhưng hãy nhớ rằng bạn đã thấy cách sử dụng iterator này trong [Listing 3-5][ch3-iter]<!-- ignore -->, nơi chúng ta dùng vòng lặp `for` với iterator để chạy một số mã trên từng phần tử trong một collection.

#### Tìm Kiếm Chuỗi Query Trong Mỗi Dòng

Tiếp theo, chúng ta sẽ kiểm tra xem dòng hiện tại có chứa chuỗi query không. May mắn là các string có một phương thức hữu ích tên là `contains` làm việc này cho chúng ta! Thêm một lời gọi đến phương thức `contains` trong hàm `search`, như được trình bày trong Listing 12-18. Lưu ý rằng mã này vẫn chưa biên dịch được.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

<span class="caption">Listing 12-18: Thêm chức năng kiểm tra xem dòng có chứa chuỗi trong `query` không</span>

Hiện tại, chúng ta đang xây dựng dần chức năng. Để mã biên dịch được, chúng ta cần trả về một giá trị từ thân hàm như đã chỉ ra trong chữ ký hàm.

#### Lưu Các Dòng Khớp

Để hoàn thiện hàm này, chúng ta cần một cách lưu các dòng khớp mà muốn trả về. Để làm điều đó, chúng ta có thể tạo một vector mutable trước vòng lặp `for` và gọi phương thức `push` để lưu một `line` vào vector. Sau vòng lặp `for`, chúng ta trả về vector, như được trình bày trong Listing 12-19.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

<span class="caption">Listing 12-19: Lưu các dòng khớp để có thể trả về chúng</span>

Bây giờ, hàm `search` sẽ chỉ trả về những dòng chứa `query`, và test của chúng ta sẽ thành công. Hãy chạy test:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Test của chúng ta đã thành công, vì vậy chúng ta biết nó hoạt động!

Ở thời điểm này, chúng ta có thể xem xét các cơ hội để refactor việc triển khai hàm search trong khi vẫn giữ các test thành công để duy trì cùng một chức năng. Mã trong hàm search không tệ, nhưng nó chưa tận dụng một số tính năng hữu ích của iterator. Chúng ta sẽ quay lại ví dụ này trong [Chương 13][ch13-iterators]<!-- ignore -->, nơi chúng ta sẽ tìm hiểu chi tiết về iterator và xem cách cải thiện nó.

#### Sử dụng hàm `search` trong hàm `run`

Bây giờ hàm `search` đã hoạt động và được test, chúng ta cần gọi `search` từ hàm `run`. Chúng ta cần truyền giá trị `config.query` và `contents` mà `run` đọc từ file vào hàm `search`. Sau đó, `run` sẽ in từng dòng được trả về từ `search`:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/src/lib.rs:here}}
```

Chúng ta vẫn đang sử dụng một vòng lặp `for` để lấy từng dòng từ `search` và in ra.

Bây giờ toàn bộ chương trình nên hoạt động! Hãy thử nó, trước tiên với một từ mà sẽ trả về chính xác một dòng từ bài thơ của Emily Dickinson, `"frog"`:

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Tuyệt! Bây giờ hãy thử một từ sẽ khớp với nhiều dòng, chẳng hạn `"body"`:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Và cuối cùng, hãy chắc chắn rằng chúng ta không nhận được dòng nào khi tìm kiếm một từ không xuất hiện trong bài thơ, chẳng hạn `"monomorphization"`:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Tuyệt vời! Chúng ta đã xây dựng phiên bản mini của một công cụ cổ điển và học được rất nhiều về cách cấu trúc ứng dụng. Chúng ta cũng đã tìm hiểu một chút về nhập/xuất file, lifetimes, testing và phân tích tham số dòng lệnh.

Để hoàn thiện dự án này, chúng ta sẽ minh họa ngắn gọn cách làm việc với biến môi trường và cách in ra lỗi chuẩn (standard error), cả hai đều hữu ích khi viết các chương trình dòng lệnh.

[validating-references-with-lifetimes]:
ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
