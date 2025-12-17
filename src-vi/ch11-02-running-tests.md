## Kiểm soát cách chạy các hàm kiểm tra

Cũng giống như `cargo run` biên dịch mã của bạn và sau đó chạy file nhị phân kết quả, `cargo test` biên dịch mã của bạn ở chế độ kiểm tra và chạy file nhị phân kiểm tra tạo ra. Hành vi mặc định của file nhị phân do `cargo test` tạo ra là chạy tất cả các hàm kiểm tra song song và bắt giữ output được tạo ra trong quá trình chạy kiểm tra, ngăn không cho output hiển thị và giúp dễ đọc các kết quả liên quan đến kiểm tra. Tuy nhiên, bạn có thể chỉ định các tùy chọn dòng lệnh để thay đổi hành vi mặc định này.

Một số tùy chọn dòng lệnh dành cho `cargo test`, và một số dành cho file nhị phân kiểm tra tạo ra. Để tách hai loại đối số này, bạn liệt kê các đối số dành cho `cargo test`, theo sau là dấu phân tách `--`, rồi mới đến các đối số dành cho file nhị phân kiểm tra. Chạy `cargo test --help` sẽ hiển thị các tùy chọn bạn có thể dùng với `cargo test`, và chạy `cargo test -- --help` sẽ hiển thị các tùy chọn bạn có thể dùng sau dấu phân tách.

### Chạy các hàm kiểm tra song song hoặc tuần tự

Khi bạn chạy nhiều hàm kiểm tra, theo mặc định chúng chạy song song bằng các thread, nghĩa là chúng hoàn thành nhanh hơn và bạn nhận phản hồi nhanh hơn. Vì các hàm kiểm tra chạy cùng lúc, bạn phải đảm bảo rằng các hàm kiểm tra không phụ thuộc vào nhau hoặc vào bất kỳ trạng thái chung nào, bao gồm môi trường chung, chẳng hạn như thư mục làm việc hiện tại hoặc biến môi trường.

Ví dụ, giả sử mỗi hàm kiểm tra của bạn chạy một đoạn mã tạo một file trên ổ đĩa tên *test-output.txt* và ghi một số dữ liệu vào file đó. Sau đó mỗi hàm kiểm tra đọc dữ liệu trong file và xác nhận rằng file chứa một giá trị nhất định, khác nhau ở mỗi hàm kiểm tra. Vì các hàm kiểm tra chạy cùng lúc, một hàm kiểm tra có thể ghi đè file trong lúc một hàm kiểm tra khác đang ghi và đọc file. Hàm kiểm tra thứ hai sẽ thất bại, không phải vì mã sai mà vì các hàm kiểm tra đã can thiệp lẫn nhau khi chạy song song. Một giải pháp là đảm bảo mỗi hàm kiểm tra ghi vào một file khác; giải pháp khác là chạy các hàm kiểm tra một lần mỗi lần.

Nếu bạn không muốn chạy các hàm kiểm tra song song hoặc muốn kiểm soát chi tiết hơn số lượng thread sử dụng, bạn có thể gửi flag `--test-threads` kèm số lượng thread muốn dùng cho file nhị phân kiểm tra. Xem ví dụ sau:

```console
$ cargo test -- --test-threads=1
```

Chúng ta đặt số lượng thread kiểm tra là `1`, thông báo cho chương trình không sử dụng song song. Chạy các hàm kiểm tra bằng một thread sẽ mất nhiều thời gian hơn so với chạy song song, nhưng các hàm kiểm tra sẽ không can thiệp lẫn nhau nếu chúng chia sẻ trạng thái.

### Hiển thị output của hàm

Theo mặc định, nếu một hàm kiểm tra thành công, thư viện kiểm tra của Rust sẽ bắt giữ mọi thứ in ra standard output. Ví dụ, nếu chúng ta gọi `println!` trong một hàm kiểm tra và hàm kiểm tra thành công, chúng ta sẽ không thấy output của `println!` trên terminal; chúng ta chỉ thấy dòng báo rằng hàm kiểm tra đã thành công. Nếu một hàm kiểm tra thất bại, chúng ta sẽ thấy mọi thứ được in ra standard output kèm với thông báo thất bại.

Ví dụ, Listing 11-10 có một hàm ngớ ngẩn in giá trị của tham số và trả về 10, cùng với một hàm kiểm tra thành công và một hàm kiểm tra thất bại.

<span class="filename">Filename: src/lib.rs</span>

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

<span class="caption">Listing 11-10: Các hàm kiểm tra cho một hàm gọi `println!`</span>

Khi chúng ta chạy các hàm kiểm tra này với `cargo test`, chúng ta sẽ thấy output như sau:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Lưu ý rằng ở đâu trong output này chúng ta cũng không thấy `I got the value 4`, đây là giá trị được in ra khi hàm kiểm tra thành công chạy. Output này đã bị bắt giữ. Output từ hàm kiểm tra thất bại, `I got the value 8`, xuất hiện trong phần tóm tắt kết quả kiểm tra, cùng với nguyên nhân thất bại của hàm kiểm tra.

Nếu chúng ta muốn thấy giá trị được in ra của các hàm kiểm tra thành công, chúng ta có thể yêu cầu Rust hiển thị output của các kiểm tra thành công bằng `--show-output`.

```console
$ cargo test -- --show-output
```

When we run the tests in Listing 11-10 again with the `--show-output` flag, we
see the following output:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Chạy một tập con các hàm kiểm tra theo tên

Đôi khi, chạy toàn bộ bộ kiểm tra có thể mất nhiều thời gian. Nếu bạn đang làm việc trên một phần cụ thể của mã, bạn có thể chỉ muốn chạy các hàm kiểm tra liên quan đến phần đó. Bạn có thể chọn các hàm kiểm tra để chạy bằng cách truyền tên hoặc các tên của hàm kiểm tra muốn chạy làm đối số cho `cargo test`.

Để minh họa cách chạy một tập con các hàm kiểm tra, trước tiên chúng ta sẽ tạo ba hàm kiểm tra cho hàm `add_two` của mình, như hiển thị trong Listing 11-11, và chọn những hàm nào sẽ chạy.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

<span class="caption">Listing 11-11: Ba hàm kiểm tra với ba tên khác nhau</span>

Nếu chúng ta chạy các hàm kiểm tra mà không truyền bất kỳ đối số nào, như đã thấy trước đó, tất cả các hàm kiểm tra sẽ chạy song song:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Chạy một hàm kiểm tra riêng lẻ

Chúng ta có thể truyền tên bất kỳ hàm kiểm tra nào cho `cargo test` để chỉ chạy hàm kiểm tra đó:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Chỉ hàm kiểm tra có tên `one_hundred` được chạy; hai hàm kiểm tra còn lại không khớp với tên đó. Output của hàm kiểm tra thông báo cho chúng ta rằng còn nhiều hàm kiểm tra khác không chạy bằng cách hiển thị `2 filtered out` ở cuối.

Chúng ta không thể chỉ định tên nhiều hàm kiểm tra theo cách này; chỉ giá trị đầu tiên truyền cho `cargo test` sẽ được sử dụng. Nhưng vẫn có cách để chạy nhiều hàm kiểm tra.

#### Lọc để chạy nhiều hàm kiểm tra

Chúng ta có thể chỉ định một phần của tên hàm kiểm tra, và bất kỳ hàm kiểm tra nào có tên khớp với giá trị đó sẽ được chạy. Ví dụ, vì hai trong số các hàm kiểm tra của chúng ta có tên chứa `add`, chúng ta có thể chạy hai hàm đó bằng cách chạy `cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Lệnh này chạy tất cả các hàm kiểm tra có `add` trong tên và loại bỏ hàm kiểm tra có tên `one_hundred`. Cũng lưu ý rằng module chứa hàm kiểm tra trở thành một phần của tên hàm kiểm tra, vì vậy chúng ta có thể chạy tất cả các hàm kiểm tra trong một module bằng cách lọc theo tên module.

### Bỏ qua một số hàm kiểm tra trừ khi được yêu cầu cụ thể

Đôi khi một vài hàm kiểm tra cụ thể có thể mất nhiều thời gian để thực thi, vì vậy bạn có thể muốn loại chúng ra trong hầu hết các lần chạy `cargo test`. Thay vì liệt kê tất cả các hàm kiểm tra bạn muốn chạy làm đối số, bạn có thể chú thích các hàm kiểm tra tốn thời gian bằng attribute `ignore` để loại chúng ra, như minh họa dưới đây:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs}}
```

Sau `#[test]`, chúng ta thêm dòng `#[ignore]` vào hàm kiểm tra mà chúng ta muốn loại ra. Bây giờ khi chạy các hàm kiểm tra, `it_works` sẽ chạy, nhưng `expensive_test` thì không:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

Hàm `expensive_test` được liệt kê là `ignored`. Nếu chúng ta chỉ muốn chạy các hàm kiểm tra bị bỏ qua, có thể dùng `cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Bằng cách kiểm soát các hàm kiểm tra được chạy, bạn có thể đảm bảo rằng kết quả `cargo test` sẽ nhanh. Khi đến lúc bạn muốn kiểm tra kết quả của các hàm kiểm tra bị `ignored` và có thời gian chờ kết quả, bạn có thể chạy `cargo test -- --ignored`. Nếu bạn muốn chạy tất cả các hàm kiểm tra, bất kể chúng có bị bỏ qua hay không, bạn có thể chạy `cargo test -- --include-ignored`.