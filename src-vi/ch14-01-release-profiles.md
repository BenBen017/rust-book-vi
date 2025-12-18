## Tùy Chỉnh Build với Release Profiles

Trong Rust, *release profiles* là các profile được định nghĩa sẵn và có thể tùy chỉnh với các cấu hình khác nhau, cho phép lập trình viên kiểm soát nhiều tùy chọn khi biên dịch mã. Mỗi profile được cấu hình độc lập với các profile khác.

Cargo có hai profile chính: profile `dev` mà Cargo sử dụng khi bạn chạy `cargo build` và profile `release` mà Cargo sử dụng khi bạn chạy `cargo build --release`. Profile `dev` được định nghĩa với các giá trị mặc định tốt cho phát triển, còn profile `release` có các giá trị mặc định tốt cho việc build khi phát hành.

Những tên profile này có thể quen thuộc từ kết quả đầu ra của các lần build:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

Profile `dev` và `release` là hai profile khác nhau mà trình biên dịch sử dụng.

Cargo có các thiết lập mặc định cho mỗi profile, áp dụng khi bạn chưa thêm bất kỳ phần `[profile.*]` nào trong file *Cargo.toml* của dự án. Bằng cách thêm các phần `[profile.*]` cho bất kỳ profile nào bạn muốn tùy chỉnh, bạn sẽ ghi đè một tập hợp con của các thiết lập mặc định. Ví dụ, dưới đây là các giá trị mặc định của thiết lập `opt-level` cho profile `dev` và `release`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

Thiết lập `opt-level` điều khiển số lượng tối ưu hóa mà Rust sẽ áp dụng cho mã của bạn, với phạm vi từ 0 đến 3. Áp dụng nhiều tối ưu hóa hơn sẽ làm tăng thời gian biên dịch, vì vậy nếu bạn đang phát triển và biên dịch thường xuyên, bạn sẽ muốn ít tối ưu hóa hơn để biên dịch nhanh hơn mặc dù mã chạy chậm hơn. Do đó, `opt-level` mặc định cho profile `dev` là `0`. Khi bạn sẵn sàng phát hành mã, tốt nhất là dành nhiều thời gian hơn cho việc biên dịch. Bạn chỉ biên dịch ở chế độ release một lần, nhưng sẽ chạy chương trình biên dịch nhiều lần, vì vậy chế độ release đánh đổi thời gian biên dịch lâu hơn để mã chạy nhanh hơn. Đó là lý do tại sao `opt-level` mặc định cho profile `release` là `3`.

Bạn có thể ghi đè thiết lập mặc định bằng cách thêm một giá trị khác trong *Cargo.toml*. Ví dụ, nếu muốn sử dụng mức tối ưu hóa 1 trong profile phát triển, bạn có thể thêm hai dòng sau vào file *Cargo.toml* của dự án:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Mã này ghi đè thiết lập mặc định là `0`. Bây giờ khi chạy `cargo build`, Cargo sẽ sử dụng các giá trị mặc định cho profile `dev` cùng với tùy chỉnh `opt-level` của chúng ta. Vì chúng ta đặt `opt-level` là `1`, Cargo sẽ áp dụng nhiều tối ưu hóa hơn mặc định, nhưng không nhiều bằng trong bản build release.

Để xem danh sách đầy đủ các tùy chọn cấu hình và giá trị mặc định cho từng profile, hãy tham khảo 
[tài liệu của Cargo](https://doc.rust-lang.org/cargo/reference/profiles.html).
