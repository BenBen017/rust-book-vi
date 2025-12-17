## Tổ chức các hàm kiểm tra

Như đã đề cập ở đầu chương, kiểm tra là một lĩnh vực phức tạp, và những người khác nhau sử dụng thuật ngữ và cách tổ chức khác nhau. Cộng đồng Rust chia các hàm kiểm tra thành hai loại chính: unit tests và integration tests. *Unit tests* nhỏ và tập trung hơn, kiểm tra một module riêng lẻ tại một thời điểm và có thể kiểm tra các giao diện private. *Integration tests* hoàn toàn nằm ngoài thư viện của bạn và sử dụng mã của bạn giống như bất kỳ mã bên ngoài nào khác, chỉ sử dụng giao diện public và có thể kiểm tra nhiều module trong một hàm kiểm tra.

Việc viết cả hai loại kiểm tra là quan trọng để đảm bảo rằng các phần của thư viện hoạt động như bạn mong đợi, riêng lẻ và kết hợp với nhau.

### Unit Tests

Mục đích của unit tests là kiểm tra từng đơn vị mã một cách riêng biệt với phần còn lại của mã để nhanh chóng xác định vị trí mã đang hoạt động hoặc không hoạt động như mong đợi. Bạn sẽ đặt unit tests trong thư mục *src* trong mỗi file chứa mã mà chúng đang kiểm tra. Quy ước là tạo một module có tên `tests` trong mỗi file để chứa các hàm kiểm tra và chú thích module bằng `cfg(test)`.

#### Module Tests và `#[cfg(test)]`

Chú thích `#[cfg(test)]` trên module tests thông báo cho Rust chỉ biên dịch và chạy mã kiểm tra khi bạn chạy `cargo test`, không khi chạy `cargo build`. Điều này tiết kiệm thời gian biên dịch khi bạn chỉ muốn build thư viện và tiết kiệm dung lượng trong file nhị phân kết quả vì các hàm kiểm tra không được bao gồm. Bạn sẽ thấy rằng vì các integration tests nằm ở thư mục khác, chúng không cần chú thích `#[cfg(test)]`. Tuy nhiên, vì unit tests nằm trong cùng file với mã, bạn sẽ dùng `#[cfg(test)]` để chỉ định rằng chúng không nên được bao gồm trong kết quả biên dịch.

Hãy nhớ rằng khi chúng ta tạo dự án mới `adder` trong phần đầu tiên của chương này, Cargo đã tạo ra đoạn mã sau cho chúng ta:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Đoạn mã này là module kiểm tra được tạo tự động. Attribute `cfg` là viết tắt của *configuration* và thông báo cho Rust rằng mục tiếp theo chỉ nên được bao gồm khi có một tùy chọn cấu hình nhất định. Trong trường hợp này, tùy chọn cấu hình là `test`, được Rust cung cấp để biên dịch và chạy các hàm kiểm tra. Bằng cách sử dụng attribute `cfg`, Cargo chỉ biên dịch mã kiểm tra của chúng ta khi chúng ta thực sự chạy các hàm kiểm tra bằng `cargo test`. Điều này bao gồm bất kỳ hàm trợ giúp nào nằm trong module này, cũng như các hàm được chú thích bằng `#[test]`.

#### Kiểm tra các hàm private

Trong cộng đồng kiểm tra, có tranh luận về việc có nên kiểm tra trực tiếp các hàm private hay không, và các ngôn ngữ khác thường khó hoặc không thể kiểm tra các hàm private. Bất kể bạn theo quan điểm kiểm tra nào, quy tắc private của Rust cho phép bạn kiểm tra các hàm private. Hãy xem xét đoạn mã trong Listing 11-12 với hàm private `internal_adder`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

<span class="caption">Listing 11-12: Kiểm tra một hàm private</span>

Lưu ý rằng hàm `internal_adder` không được đánh dấu là `pub`. Các hàm kiểm tra chỉ là mã Rust, và module `tests` chỉ là một module khác. Như chúng ta đã thảo luận trong phần [“Paths for Referring to an Item in the Module Tree”][paths]<!-- ignore -->, các phần tử trong module con có thể sử dụng các phần tử trong module cha của chúng. Trong kiểm tra này, chúng ta đưa tất cả các phần tử của module cha vào phạm vi với `use super::*`, và sau đó hàm kiểm tra có thể gọi `internal_adder`. Nếu bạn không cho rằng các hàm private nên được kiểm tra, Rust không ép buộc bạn phải làm điều đó.

### Integration Tests

Trong Rust, integration tests hoàn toàn nằm ngoài thư viện của bạn. Chúng sử dụng thư viện của bạn giống như bất kỳ mã nào khác, có nghĩa là chúng chỉ có thể gọi các hàm là một phần của API public của thư viện. Mục đích của chúng là kiểm tra xem nhiều phần của thư viện có hoạt động cùng nhau đúng không. Các đơn vị mã hoạt động đúng riêng lẻ có thể gặp vấn đề khi tích hợp, vì vậy việc bao phủ kiểm tra mã tích hợp cũng quan trọng. Để tạo integration tests, trước tiên bạn cần tạo thư mục *tests*.

#### Thư mục *tests*

Chúng ta tạo thư mục *tests* ở cấp cao nhất của dự án, cạnh thư mục *src*. Cargo biết tìm các file kiểm tra tích hợp trong thư mục này. Chúng ta có thể tạo bao nhiêu file kiểm tra tùy thích, và Cargo sẽ biên dịch từng file như một crate riêng lẻ.

Hãy tạo một integration test. Với mã trong Listing 11-12 vẫn nằm trong file *src/lib.rs*, tạo thư mục *tests* và tạo một file mới tên *tests/integration_test.rs*. Cấu trúc thư mục của bạn sẽ trông như sau:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Enter the code in Listing 11-13 into the *tests/integration_test.rs* file:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

<span class="caption">Listing 11-13: Một integration test cho một hàm trong crate `adder`</span>

Mỗi file trong thư mục `tests` là một crate riêng biệt, vì vậy chúng ta cần đưa thư viện của mình vào phạm vi của từng test crate. Vì lý do đó, chúng ta thêm `use adder` ở đầu mã, điều mà trong unit tests chúng ta không cần.

Chúng ta không cần chú thích bất kỳ mã nào trong *tests/integration_test.rs* với `#[cfg(test)]`. Cargo xử lý thư mục `tests` đặc biệt và chỉ biên dịch các file trong thư mục này khi chạy `cargo test`. Bây giờ hãy chạy `cargo test`:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Ba phần của output bao gồm unit tests, integration test và doc tests. Lưu ý rằng nếu bất kỳ hàm kiểm tra nào trong một phần thất bại, các phần tiếp theo sẽ không được chạy. Ví dụ, nếu một unit test thất bại, sẽ không có output cho integration và doc tests vì các test đó chỉ được chạy nếu tất cả unit tests đều thành công.

Phần đầu tiên cho unit tests giống như những gì chúng ta đã thấy: một dòng cho mỗi unit test (một test tên là `internal` mà chúng ta đã thêm trong Listing 11-12) và sau đó là một dòng tóm tắt cho các unit tests.

Phần integration tests bắt đầu với dòng `Running tests/integration_test.rs`. Tiếp theo, có một dòng cho mỗi hàm kiểm tra trong integration test đó và một dòng tóm tắt kết quả của integration test ngay trước khi phần `Doc-tests adder` bắt đầu.

Mỗi file integration test có phần riêng của nó, vì vậy nếu chúng ta thêm nhiều file trong thư mục *tests*, sẽ có thêm các phần cho integration tests.

Chúng ta vẫn có thể chạy một hàm kiểm tra integration cụ thể bằng cách chỉ định tên hàm kiểm tra làm đối số cho `cargo test`. Để chạy tất cả các test trong một file integration test cụ thể, dùng đối số `--test` của `cargo test` theo sau là tên file:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Lệnh này chỉ chạy các hàm kiểm tra trong file *tests/integration_test.rs*.

#### Submodules trong Integration Tests

Khi bạn thêm nhiều integration tests, bạn có thể muốn tạo thêm các file trong thư mục *tests* để giúp tổ chức chúng; ví dụ, bạn có thể nhóm các hàm kiểm tra theo chức năng mà chúng đang kiểm tra. Như đã đề cập trước đó, mỗi file trong thư mục *tests* được biên dịch như một crate riêng biệt, điều này hữu ích để tạo các phạm vi riêng nhằm mô phỏng gần hơn cách người dùng cuối sẽ sử dụng crate của bạn. Tuy nhiên, điều này có nghĩa là các file trong thư mục *tests* không chia sẻ cùng hành vi như các file trong *src*, như bạn đã học ở Chương 7 về cách tách mã thành các module và file.

Sự khác biệt trong hành vi của các file trong thư mục *tests* rõ ràng nhất khi bạn có một bộ hàm trợ giúp dùng trong nhiều file integration test và bạn cố gắng theo các bước trong phần [“Separating Modules into Different Files”][separating-modules-into-files]<!-- ignore --> của Chương 7 để trích xuất chúng thành một module chung. Ví dụ, nếu chúng ta tạo file *tests/common.rs* và đặt một hàm tên là `setup` trong đó, chúng ta có thể thêm một số mã vào `setup` mà muốn gọi từ nhiều hàm kiểm tra trong nhiều file test:

<span class="filename">Filename: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Khi chúng ta chạy lại các hàm kiểm tra, chúng ta sẽ thấy một phần mới trong output cho file *common.rs*, mặc dù file này không chứa bất kỳ hàm kiểm tra nào và chúng ta cũng không gọi hàm `setup` từ đâu cả:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Việc `common` xuất hiện trong kết quả kiểm tra với thông báo `running 0 tests` không phải là điều chúng ta mong muốn. Chúng ta chỉ muốn chia sẻ một số mã với các file integration test khác.

Để tránh `common` xuất hiện trong output của kiểm tra, thay vì tạo file *tests/common.rs*, chúng ta sẽ tạo *tests/common/mod.rs*. Cấu trúc thư mục dự án bây giờ trông như sau:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Đây là quy ước đặt tên cũ mà Rust vẫn hiểu, như chúng ta đã đề cập trong phần [“Alternate File Paths”][alt-paths]<!-- ignore --> của Chương 7. Đặt tên file theo cách này thông báo cho Rust rằng không nên coi module `common` là một file integration test. Khi chúng ta chuyển mã hàm `setup` vào *tests/common/mod.rs* và xóa file *tests/common.rs*, phần hiển thị trong output kiểm tra sẽ không còn xuất hiện. Các file trong các thư mục con của *tests* sẽ không được biên dịch như các crate riêng biệt và cũng không xuất hiện trong output kiểm tra.

Sau khi tạo xong *tests/common/mod.rs*, chúng ta có thể sử dụng nó từ bất kỳ file integration test nào như một module. Dưới đây là ví dụ gọi hàm `setup` từ test `it_adds_two` trong *tests/integration_test.rs*:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Lưu ý rằng khai báo `mod common;` giống như khai báo module mà chúng ta đã minh họa trong Listing 7-21. Sau đó, trong hàm kiểm tra, chúng ta có thể gọi hàm `common::setup()`.

#### Integration Tests cho Binary Crates

Nếu dự án của chúng ta là một binary crate chỉ chứa file *src/main.rs* và không có file *src/lib.rs*, chúng ta không thể tạo integration tests trong thư mục *tests* và đưa các hàm được định nghĩa trong file *src/main.rs* vào phạm vi với câu lệnh `use`. Chỉ các library crate mới cung cấp các hàm mà crate khác có thể sử dụng; binary crate được thiết kế để chạy độc lập.

Đây là một trong những lý do mà các dự án Rust cung cấp binary thường có file *src/main.rs* đơn giản, gọi các logic nằm trong file *src/lib.rs*. Sử dụng cấu trúc đó, integration tests *có thể* kiểm tra library crate với `use` để làm cho các chức năng quan trọng có thể sử dụng được. Nếu các chức năng quan trọng hoạt động, phần mã nhỏ trong file *src/main.rs* cũng sẽ hoạt động, và phần mã nhỏ đó không cần phải kiểm tra.

## Tóm tắt

Các tính năng kiểm tra của Rust cung cấp cách để xác định cách mã nên hoạt động nhằm đảm bảo nó tiếp tục hoạt động như bạn mong đợi, ngay cả khi bạn thực hiện các thay đổi. Unit tests kiểm tra các phần khác nhau của thư viện một cách riêng biệt và có thể kiểm tra các chi tiết triển khai riêng tư. Integration tests kiểm tra rằng nhiều phần của thư viện hoạt động cùng nhau chính xác, và chúng sử dụng API công khai của thư viện để kiểm tra mã giống như cách mã bên ngoài sẽ sử dụng. Mặc dù hệ thống kiểu và quy tắc sở hữu của Rust giúp ngăn ngừa một số loại lỗi, nhưng việc kiểm tra vẫn quan trọng để giảm thiểu các lỗi logic liên quan đến cách mã của bạn được mong đợi hoạt động.

Hãy kết hợp kiến thức bạn học được trong chương này và các chương trước để làm việc trên một dự án!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]:
ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
