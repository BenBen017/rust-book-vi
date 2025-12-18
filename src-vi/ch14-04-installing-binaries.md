<!-- Old link, do not remove -->
<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Cài Binaries với `cargo install`

Lệnh `cargo install` cho phép bạn cài đặt và sử dụng các binary crate **cục bộ**. Lệnh này không nhằm thay thế các gói hệ thống; nó chỉ là một cách tiện lợi cho các lập trình viên Rust để cài đặt các công cụ mà người khác chia sẻ trên [crates.io](https://crates.io/)<!-- ignore -->. 

Lưu ý rằng bạn chỉ có thể cài đặt các gói có **binary targets**. Một *binary target* là chương trình có thể chạy được được tạo ra nếu crate có file *src/main.rs* hoặc một file khác được chỉ định là binary, trái ngược với **library target** không thể chạy độc lập mà chỉ phù hợp để sử dụng trong các chương trình khác. Thông thường, các crate sẽ ghi chú trong file *README* liệu crate là library, có binary target, hay cả hai.

Tất cả các binaries được cài bằng `cargo install` sẽ được lưu trong thư mục *bin* của **installation root**. Nếu bạn cài Rust bằng *rustup.rs* và không cấu hình gì thêm, thư mục này sẽ là `$HOME/.cargo/bin`. Hãy đảm bảo thư mục này có trong `$PATH` để bạn có thể chạy các chương trình đã cài bằng `cargo install`.

Ví dụ, trong Chương 12, chúng ta đã nhắc tới công cụ `grep` viết bằng Rust có tên là `ripgrep` dùng để tìm kiếm trong file. Để cài `ripgrep`, ta chạy:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v13.0.0
  Downloaded 1 crate (243.3 KB) in 0.88s
  Installing ripgrep v13.0.0
--snip--
   Compiling ripgrep v13.0.0
    Finished release [optimized + debuginfo] target(s) in 3m 10s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v13.0.0` (executable `rg`)
```

Dòng thứ hai từ cuối của output hiển thị vị trí và tên của binary đã cài đặt, trong trường hợp của `ripgrep` là `rg`. Miễn là thư mục cài đặt nằm trong `$PATH`, như đã đề cập trước đó, bạn có thể chạy `rg --help` và bắt đầu sử dụng công cụ tìm kiếm file nhanh hơn và “Rustier” này!
