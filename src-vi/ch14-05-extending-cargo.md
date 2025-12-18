## Extending Cargo with Custom Commands

Cargo được thiết kế để bạn có thể mở rộng với các lệnh con mới mà không cần phải
sửa đổi Cargo. Nếu một binary trong `$PATH` của bạn có tên là `cargo-something`,
bạn có thể chạy nó như thể nó là một lệnh con của Cargo bằng cách chạy
`cargo something`. Các lệnh tùy chỉnh như thế này cũng được liệt kê khi bạn chạy
`cargo --list`. Khả năng sử dụng `cargo install` để cài đặt các tiện ích mở rộng
và sau đó chạy chúng giống như các công cụ Cargo tích hợp là một lợi ích cực kỳ
tiện lợi của thiết kế Cargo!

## Tóm tắt

Chia sẻ mã với Cargo và [crates.io](https://crates.io/)<!-- ignore --> là một phần
làm cho hệ sinh thái Rust hữu ích cho nhiều nhiệm vụ khác nhau. Thư viện chuẩn
của Rust nhỏ và ổn định, nhưng các crate rất dễ chia sẻ, sử dụng và cải tiến theo
một dòng thời gian khác với ngôn ngữ. Đừng ngần ngại chia sẻ mã mà bạn thấy hữu
ích trên [crates.io](https://crates.io/)<!-- ignore -->; rất có thể nó cũng sẽ hữu
ích với người khác!
