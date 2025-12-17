## Làm việc với Biến Môi Trường

Chúng ta sẽ cải tiến `minigrep` bằng cách thêm một tính năng: tùy chọn tìm kiếm không phân biệt chữ hoa chữ thường, mà người dùng có thể bật thông qua biến môi trường. Chúng ta có thể biến tính năng này thành một tùy chọn dòng lệnh và yêu cầu người dùng nhập nó mỗi lần muốn sử dụng, nhưng bằng cách dùng biến môi trường, chúng ta cho phép người dùng thiết lập một lần và tất cả các tìm kiếm sau đó trong phiên terminal đó sẽ không phân biệt chữ hoa chữ thường.

### Viết Test Thất Bại cho Hàm `search` Không Phân Biệt Chữ Hoa

Trước tiên, chúng ta thêm một hàm mới `search_case_insensitive` sẽ được gọi khi biến môi trường có giá trị. Chúng ta sẽ tiếp tục theo quy trình TDD, vì vậy bước đầu tiên là viết một test thất bại. Chúng ta sẽ thêm một test mới cho hàm `search_case_insensitive` và đổi tên test cũ từ `one_result` thành `case_sensitive` để làm rõ sự khác biệt giữa hai test, như minh họa trong Listing 12-20.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

<span class="caption">Listing 12-20: Thêm một test thất bại mới cho
hàm không phân biệt chữ hoa/chữ thường mà chúng ta sắp thêm</span>

Lưu ý rằng chúng ta cũng đã chỉnh sửa `contents` của test cũ. Chúng ta đã thêm một dòng mới với nội dung `"Duct tape."` sử dụng chữ D viết hoa, dòng này không nên khớp với query `"duct"` khi chúng ta tìm kiếm theo kiểu phân biệt chữ hoa/chữ thường. Việc thay đổi test cũ theo cách này giúp đảm bảo rằng chúng ta không vô tình phá vỡ chức năng tìm kiếm phân biệt chữ hoa/chữ thường mà chúng ta đã triển khai. Test này nên chạy được và sẽ tiếp tục chạy khi chúng ta làm việc trên chức năng tìm kiếm không phân biệt chữ hoa/chữ thường.

Test mới cho tìm kiếm không phân biệt chữ hoa/chữ thường sử dụng `"rUsT"` làm query. Trong hàm `search_case_insensitive` mà chúng ta sắp thêm, query `"rUsT"` sẽ khớp với dòng chứa `"Rust:"` với chữ R viết hoa và cũng khớp với dòng `"Trust me."` mặc dù cả hai có cách viết khác với query. Đây là test thất bại của chúng ta, và nó sẽ không biên dịch được vì chúng ta chưa định nghĩa hàm `search_case_insensitive`. Bạn có thể thêm một triển khai skeleton trả về luôn một vector rỗng, tương tự như cách chúng ta làm với hàm `search` trong Listing 12-16 để thấy test biên dịch và thất bại.

### Triển khai hàm `search_case_insensitive`

Hàm `search_case_insensitive`, như minh họa trong Listing 12-21, sẽ gần như giống hệt hàm `search`. Điểm khác duy nhất là chúng ta sẽ chuyển `query` và từng `line` về chữ thường, để dù đầu vào có kiểu chữ nào, chúng cũng sẽ cùng kiểu chữ khi kiểm tra xem dòng có chứa query hay không.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

<span class="caption">Listing 12-21: Định nghĩa hàm `search_case_insensitive`
để chuyển query và dòng về chữ thường trước khi so sánh</span>

Trước tiên, chúng ta chuyển chuỗi `query` về chữ thường và lưu vào một biến
shadow cùng tên. Việc gọi `to_lowercase` trên query là cần thiết để dù query
của người dùng là `"rust"`, `"RUST"`, `"Rust"`, hay `"rUsT"`, chúng ta sẽ
xử lý query như thể nó là `"rust"` và không phân biệt chữ hoa/chữ thường.
Mặc dù `to_lowercase` sẽ xử lý Unicode cơ bản, nhưng không hoàn toàn chính
xác 100%. Nếu chúng ta viết một ứng dụng thực sự, sẽ cần xử lý kỹ hơn ở
phần này, nhưng mục tiêu của phần này là biến môi trường, không phải Unicode,
nên chúng ta sẽ tạm dừng ở đây.

Lưu ý rằng `query` bây giờ là một `String` thay vì một slice của chuỗi, vì
việc gọi `to_lowercase` tạo ra dữ liệu mới thay vì tham chiếu dữ liệu có sẵn.
Ví dụ, nếu query là `"rUsT"`, slice này không chứa chữ `u` hay `t` viết
thường để chúng ta sử dụng, nên chúng ta phải cấp phát một `String` mới
chứa `"rust"`. Khi truyền `query` làm đối số cho phương thức `contains`, chúng
ta cần thêm dấu & vì chữ ký của `contains` được định nghĩa để nhận một slice
của chuỗi.

Tiếp theo, chúng ta gọi `to_lowercase` trên từng `line` để chuyển tất cả
ký tự về chữ thường. Giờ đây, khi `line` và `query` đã được chuyển về chữ
thường, chúng ta sẽ tìm thấy các dòng khớp bất kể query có chữ hoa hay
chữ thường.

Hãy xem liệu triển khai này có vượt qua các test không:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Tuyệt vời! Các test đã vượt qua. Bây giờ, chúng ta sẽ gọi hàm mới
`search_case_insensitive` từ hàm `run`. Trước tiên, chúng ta sẽ thêm một
tùy chọn cấu hình vào struct `Config` để chuyển đổi giữa tìm kiếm phân biệt
chữ hoa/chữ thường và không phân biệt chữ hoa/chữ thường. Việc thêm trường
này sẽ gây ra lỗi biên dịch vì chúng ta chưa khởi tạo trường này ở đâu cả:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:here}}
```

Chúng ta đã thêm trường `ignore_case` mà chứa một Boolean. Tiếp theo, chúng ta cần hàm `run`
để kiểm tra giá trị của trường `ignore_case` và dùng nó để quyết định
có gọi hàm `search` hay hàm `search_case_insensitive`
như được hiển thị trong Listing 12-22. Điều này vẫn chưa thể compile.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:there}}
```

<span class="caption">Listing 12-22: Gọi `search` hoặc
`search_case_insensitive` dựa trên giá trị trong `config.ignore_case`</span>

Cuối cùng, chúng ta cần kiểm tra biến môi trường. Các hàm làm việc với
biến môi trường nằm trong module `env` của thư viện chuẩn, vì vậy chúng ta
mang module đó vào phạm vi ở đầu *src/lib.rs*. Sau đó, chúng ta sẽ sử dụng
hàm `var` từ module `env` để kiểm tra xem có giá trị nào đã được đặt cho
biến môi trường có tên `IGNORE_CASE` hay không, như được hiển thị trong
Listing 12-23.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/lib.rs:here}}
```

<span class="caption">Listing 12-23: Kiểm tra giá trị trong biến môi trường
có tên `IGNORE_CASE`</span>

Ở đây, chúng ta tạo một biến mới `ignore_case`. Để gán giá trị cho nó, chúng
ta gọi hàm `env::var` và truyền vào tên của biến môi trường `IGNORE_CASE`.
Hàm `env::var` trả về một `Result` sẽ là biến thể `Ok` chứa giá trị của
biến môi trường nếu biến môi trường đó được đặt bất kỳ giá trị nào. Nó sẽ
trả về biến thể `Err` nếu biến môi trường không được đặt.

Chúng ta sử dụng phương thức `is_ok` trên `Result` để kiểm tra xem biến
môi trường có được đặt hay không, nghĩa là chương trình sẽ thực hiện tìm
kiếm không phân biệt chữ hoa chữ thường. Nếu biến môi trường `IGNORE_CASE`
không được đặt giá trị gì, `is_ok` sẽ trả về false và chương trình sẽ thực
hiện tìm kiếm phân biệt chữ hoa chữ thường. Chúng ta không quan tâm đến
*giá trị* của biến môi trường, chỉ quan tâm xem nó đã được đặt hay chưa, nên
chúng ta kiểm tra `is_ok` thay vì sử dụng `unwrap`, `expect` hay bất kỳ
phương thức nào khác mà chúng ta đã thấy trên `Result`.

Chúng ta truyền giá trị trong biến `ignore_case` vào instance của `Config` để
hàm `run` có thể đọc giá trị đó và quyết định có gọi `search_case_insensitive`
hay `search`, như chúng ta đã triển khai trong Listing 12-22.

Hãy thử xem! Trước tiên, chúng ta sẽ chạy chương trình mà không đặt biến
môi trường và với truy vấn `to`, điều này sẽ khớp với bất kỳ dòng nào
chứa từ “to” viết thường:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Có vẻ như chương trình vẫn hoạt động! Bây giờ, hãy chạy chương trình với
`IGNORE_CASE` được đặt thành `1` nhưng với cùng truy vấn `to`.

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Nếu bạn đang sử dụng PowerShell, bạn sẽ cần đặt biến môi trường và
chạy chương trình như hai lệnh riêng biệt:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Điều này sẽ làm cho `IGNORE_CASE` tồn tại trong suốt phiên làm việc shell
của bạn. Nó có thể được hủy bằng cmdlet `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Chúng ta sẽ nhận được các dòng chứa “to” mà có thể có chữ hoa:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Tuyệt vời, chúng ta cũng nhận được các dòng chứa “To”! Chương trình `minigrep`
của chúng ta bây giờ có thể thực hiện tìm kiếm không phân biệt chữ hoa chữ
thường được điều khiển bởi một biến môi trường. Bây giờ bạn đã biết cách
quản lý các tùy chọn được đặt thông qua đối số dòng lệnh hoặc biến môi
trường.

Một số chương trình cho phép sử dụng cả đối số *và* biến môi trường cho
cùng một cấu hình. Trong những trường hợp đó, chương trình sẽ quyết định
biến nào được ưu tiên. Để luyện tập thêm, bạn có thể thử điều khiển
việc phân biệt chữ hoa chữ thường thông qua đối số dòng lệnh hoặc biến
môi trường. Quyết định xem đối số dòng lệnh hay biến môi trường sẽ được
ưu tiên nếu chương trình được chạy với một cái đặt phân biệt chữ hoa
chữ thường và cái còn lại đặt không phân biệt.

Module `std::env` còn chứa nhiều tính năng hữu ích khác để làm việc với
các biến môi trường: hãy tham khảo tài liệu của nó để xem những gì
có sẵn.
