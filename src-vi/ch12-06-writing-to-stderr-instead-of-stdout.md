## Ghi Thông Báo Lỗi Ra Standard Error Thay Vì Standard Output

Hiện tại, chúng ta đang ghi toàn bộ đầu ra ra terminal bằng macro `println!`.
Trong hầu hết các terminal, có hai loại đầu ra: *standard output* (`stdout`)
cho thông tin chung và *standard error* (`stderr`) cho các thông báo lỗi.
Sự phân biệt này cho phép người dùng chuyển hướng đầu ra thành công của
chương trình vào một file nhưng vẫn hiển thị thông báo lỗi trên màn hình.

Macro `println!` chỉ có khả năng in ra standard output, vì vậy chúng ta
phải dùng thứ khác để in ra standard error.

### Kiểm Tra Nơi Các Lỗi Được Ghi

Trước tiên, hãy quan sát cách nội dung được in bởi `minigrep` hiện đang
được ghi ra standard output, bao gồm cả các thông báo lỗi mà chúng ta muốn
ghi ra standard error. Chúng ta sẽ làm điều đó bằng cách chuyển hướng luồng
standard output vào một file trong khi cố ý gây ra lỗi. Chúng ta sẽ không
chuyển hướng luồng standard error, vì vậy bất kỳ nội dung nào gửi đến
standard error sẽ tiếp tục hiển thị trên màn hình.

Các chương trình dòng lệnh được kỳ vọng gửi thông báo lỗi đến luồng
standard error để chúng ta vẫn có thể thấy thông báo lỗi trên màn hình
ngay cả khi chúng ta chuyển hướng luồng standard output vào một file. Chương
trình của chúng ta hiện tại chưa hoạt động đúng: chúng ta sắp thấy rằng nó
lưu thông báo lỗi vào file thay vì!  

Để minh họa hành vi này, chúng ta sẽ chạy chương trình với `>` và đường dẫn
file, *output.txt*, mà chúng ta muốn chuyển hướng luồng standard output vào.
Chúng ta sẽ không truyền bất kỳ đối số nào, điều này sẽ gây ra lỗi:

```console
$ cargo run > output.txt
```

Cú pháp `>` thông báo cho shell ghi nội dung của standard output vào
*output.txt* thay vì hiển thị trên màn hình. Chúng ta không thấy thông báo
lỗi mà chúng ta mong đợi được in ra màn hình, điều đó có nghĩa là nó chắc
chắn đã kết thúc trong file. Đây là nội dung của *output.txt*:

```text
Problem parsing arguments: not enough arguments
```

Đúng rồi, thông báo lỗi của chúng ta đang được in ra standard output. Sẽ hữu
ích hơn nhiều nếu các thông báo lỗi như thế này được in ra standard error để
chỉ có dữ liệu từ lần chạy thành công mới kết thúc trong file. Chúng ta sẽ
thay đổi điều đó.

### In Lỗi Ra Standard Error

Chúng ta sẽ dùng code trong Listing 12-24 để thay đổi cách các thông báo
lỗi được in. Nhờ việc refactor mà chúng ta đã làm trước đó trong chương
này, tất cả code in thông báo lỗi đều nằm trong một hàm, `main`. Thư viện
chuẩn cung cấp macro `eprintln!` để in ra luồng standard error, vì vậy hãy
thay đổi hai chỗ mà chúng ta đang gọi `println!` để in lỗi sang dùng
`eprintln!` thay vào.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

<span class="caption">Listing 12-24: Ghi thông báo lỗi ra standard error
thay vì standard output bằng cách sử dụng `eprintln!`</span>

Bây giờ, hãy chạy lại chương trình theo cùng cách, không truyền đối số
nào và chuyển hướng standard output bằng `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Bây giờ chúng ta thấy thông báo lỗi trên màn hình và *output.txt* không chứa
gì, đây là hành vi mà chúng ta mong đợi của các chương trình dòng lệnh.

Hãy chạy lại chương trình với các đối số không gây ra lỗi nhưng vẫn
chuyển hướng standard output vào một file, như sau:

```console
$ cargo run -- to poem.txt > output.txt
```

Chúng ta sẽ không thấy bất kỳ đầu ra nào trên terminal, và *output.txt*
sẽ chứa kết quả của chúng ta:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Điều này minh họa rằng bây giờ chúng ta đang sử dụng standard output cho
đầu ra thành công và standard error cho đầu ra lỗi một cách thích hợp.

## Tóm Tắt

Chương này đã ôn lại một số khái niệm chính mà bạn đã học cho đến nay và
giới thiệu cách thực hiện các thao tác I/O phổ biến trong Rust. Bằng cách
sử dụng đối số dòng lệnh, file, biến môi trường, và macro `eprintln!` để
in lỗi, bạn bây giờ đã sẵn sàng viết các ứng dụng dòng lệnh. Kết hợp với
các khái niệm trong các chương trước, code của bạn sẽ được tổ chức tốt,
lưu trữ dữ liệu hiệu quả trong các cấu trúc dữ liệu phù hợp, xử lý lỗi
mượt mà và được kiểm thử kỹ lưỡng.

Tiếp theo, chúng ta sẽ khám phá một số tính năng của Rust chịu ảnh hưởng
từ các ngôn ngữ hàm: closures và iterators.
