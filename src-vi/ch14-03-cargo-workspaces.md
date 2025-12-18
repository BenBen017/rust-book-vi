## Cargo Workspaces

Trong Chương 12, chúng ta đã xây dựng một package bao gồm một crate nhị phân (binary crate) và một crate thư viện (library crate). Khi dự án phát triển, bạn có thể thấy rằng crate thư viện ngày càng lớn và muốn tách package thành nhiều crate thư viện hơn. Cargo cung cấp một tính năng gọi là *workspaces* giúp quản lý nhiều package liên quan được phát triển đồng thời.

### Tạo Workspace

Một *workspace* là một tập hợp các package chia sẻ cùng một *Cargo.lock* và thư mục đầu ra (output directory). Hãy tạo một dự án sử dụng workspace — chúng ta sẽ dùng mã đơn giản để tập trung vào cấu trúc của workspace. Có nhiều cách để cấu trúc workspace, ở đây chúng ta sẽ giới thiệu một cách phổ biến:

- Workspace sẽ chứa một crate nhị phân và hai crate thư viện.
- Crate nhị phân cung cấp chức năng chính và sẽ phụ thuộc vào hai crate thư viện.
- Một crate thư viện sẽ cung cấp hàm `add_one`, và crate thư viện thứ hai cung cấp hàm `add_two`.

Ba crate này sẽ nằm trong cùng một workspace. Trước tiên, chúng ta tạo một thư mục mới cho workspace:

```console
$ mkdir add
$ cd add
```

Tiếp theo, trong thư mục *add*, chúng ta tạo tệp *Cargo.toml* để cấu hình toàn bộ workspace. Tệp này sẽ **không** có phần `[package]`. Thay vào đó, nó sẽ bắt đầu với phần `[workspace]` cho phép chúng ta thêm các thành viên (members) vào workspace bằng cách chỉ định đường dẫn tới package chứa crate nhị phân; trong ví dụ này, đường dẫn là *adder*:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace-with-adder-crate/add/Cargo.toml}}
```

Tiếp theo, chúng ta sẽ tạo crate nhị phân `adder` bằng cách chạy `cargo new` trong thư mục *add*:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
```

Tại thời điểm này, chúng ta có thể build workspace bằng lệnh `cargo build`. Các tệp trong thư mục *add* của bạn sẽ trông như sau:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Workspace này có một thư mục *target* duy nhất ở cấp trên, nơi các artifact được biên dịch sẽ được đặt; gói `adder` không có thư mục *target* riêng. Ngay cả khi chúng ta chạy `cargo build` từ bên trong thư mục *adder*, các artifact biên dịch vẫn sẽ nằm trong *add/target* thay vì *add/adder/target*. Cargo cấu trúc thư mục *target* trong một workspace như vậy vì các crate trong workspace dự định phụ thuộc lẫn nhau. Nếu mỗi crate có thư mục *target* riêng, mỗi crate sẽ phải biên dịch lại tất cả các crate khác trong workspace để đặt artifact vào thư mục *target* riêng của nó. Bằng cách chia sẻ một thư mục *target*, các crate có thể tránh việc biên dịch lại không cần thiết.

### Tạo Gói Thứ Hai Trong Workspace

Tiếp theo, chúng ta sẽ tạo một gói thành viên khác trong workspace và gọi nó là `add_one`. Thay đổi tệp *Cargo.toml* ở cấp trên để chỉ định đường dẫn *add_one* trong danh sách `members`:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Then generate a new library crate named `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
```

Your *add* directory should now have these directories and files:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Trong tệp *add_one/src/lib.rs*, chúng ta thêm một hàm `add_one`:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Bây giờ, gói `adder` với binary có thể phụ thuộc vào gói `add_one` chứa thư viện của chúng ta. Trước tiên, chúng ta cần thêm một *path dependency* tới `add_one` trong tệp *adder/Cargo.toml*:

<span class="filename">Filename: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo không tự động cho rằng các crate trong một workspace sẽ phụ thuộc lẫn nhau, vì vậy chúng ta cần chỉ rõ mối quan hệ phụ thuộc.

Tiếp theo, hãy sử dụng hàm `add_one` (từ crate `add_one`) trong crate `adder`. Mở tệp *adder/src/main.rs* và thêm một dòng `use` ở đầu để đưa crate thư viện `add_one` vào scope. Sau đó, thay đổi hàm `main` để gọi hàm `add_one`, như trong Listing 14-7.

<span class="filename">Filename: adder/src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

<span class="caption">Listing 14-7: Sử dụng crate thư viện `add_one` từ crate `adder`</span>

Hãy biên dịch toàn bộ workspace bằng cách chạy lệnh `cargo build` trong thư mục cấp cao *add*!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68s
```

Để chạy crate nhị phân từ thư mục *add*, chúng ta có thể chỉ định gói muốn chạy trong workspace bằng cách sử dụng tham số `-p` cùng tên gói với lệnh `cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Điều này sẽ chạy code trong *adder/src/main.rs*, crate nhị phân này phụ thuộc vào crate `add_one`.

#### Phụ thuộc vào Gói Bên Ngoài trong Workspace

Lưu ý rằng workspace chỉ có một file *Cargo.lock* ở cấp cao nhất, thay vì mỗi crate có một *Cargo.lock* riêng. Điều này đảm bảo tất cả các crate đang sử dụng cùng một phiên bản của tất cả các dependencies. 

Nếu chúng ta thêm gói `rand` vào cả *adder/Cargo.toml* và *add_one/Cargo.toml*, Cargo sẽ giải quyết cả hai về một phiên bản của `rand` và ghi lại trong cùng một *Cargo.lock*. Việc tất cả các crate trong workspace sử dụng cùng một phiên bản dependency đảm bảo các crate luôn tương thích với nhau.

Hãy thêm crate `rand` vào phần `[dependencies]` trong file *add_one/Cargo.toml* để có thể sử dụng crate `rand` trong crate `add_one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Filename: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Bây giờ chúng ta có thể thêm `use rand;` vào file *add_one/src/lib.rs*, và khi build toàn bộ workspace bằng cách chạy `cargo build` trong thư mục *add*, Cargo sẽ tự động tải và biên dịch crate `rand`. Chúng ta sẽ nhận được một cảnh báo vì hiện tại chưa sử dụng đến `rand` đã được đưa vào scope:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18s
```

File *Cargo.lock* ở cấp trên cùng bây giờ chứa thông tin về việc crate `add_one` phụ thuộc vào `rand`. Tuy nhiên, mặc dù `rand` được sử dụng trong một crate trong workspace, chúng ta không thể dùng nó trong các crate khác trừ khi thêm `rand` vào file *Cargo.toml* của các crate đó. Ví dụ, nếu thêm `use rand;` vào file *adder/src/main.rs* của crate `adder`, chúng ta sẽ gặp lỗi:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Để khắc phục vấn đề này, chỉnh sửa file *Cargo.toml* của package `adder` và chỉ ra rằng `rand` cũng là một dependency cho nó. Việc build package `adder` sẽ thêm `rand` vào danh sách dependencies của `adder` trong *Cargo.lock*, nhưng sẽ không tải thêm bản sao nào của `rand`. Cargo đảm bảo rằng mọi crate trong mọi package trong workspace sử dụng package `rand` đều dùng cùng một phiên bản, tiết kiệm không gian và đảm bảo các crate trong workspace tương thích với nhau.

#### Thêm Test vào Workspace

Một cải tiến khác, hãy thêm một test cho hàm `add_one::add_one` trong crate `add_one`:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Bây giờ chạy `cargo test` trong thư mục *add* cấp cao nhất. Chạy `cargo test` 
trong một workspace được cấu trúc như thế này sẽ chạy các test cho tất cả các crate trong workspace:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.27s
     Running unittests src/lib.rs (target/debug/deps/add_one-f0253159197f7841)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-49979ff40686fa8e)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Phần đầu ra đầu tiên cho thấy test `it_works` trong crate `add_one` đã chạy thành công. 
Phần tiếp theo cho thấy không có test nào được tìm thấy trong crate `adder`, 
và phần cuối cùng cho thấy không có test tài liệu nào được tìm thấy trong crate `add_one`.

Chúng ta cũng có thể chạy test cho một crate cụ thể trong workspace từ thư mục cấp cao bằng cách sử dụng flag `-p` 
và chỉ định tên crate mà ta muốn test:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-b3235fea9a156f74)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Phần đầu ra này cho thấy `cargo test` chỉ chạy các test cho crate `add_one` và không chạy các test của crate `adder`.

Nếu bạn muốn publish các crate trong workspace lên [crates.io](https://crates.io/), mỗi crate sẽ cần được publish riêng biệt. Tương tự như `cargo test`, bạn có thể publish một crate cụ thể trong workspace bằng cách sử dụng flag `-p` và chỉ định tên crate mà bạn muốn publish.

Để luyện tập thêm, bạn có thể thêm một crate `add_two` vào workspace này theo cách tương tự như crate `add_one`.

Khi dự án của bạn lớn lên, hãy cân nhắc sử dụng workspace: việc hiểu các thành phần nhỏ, riêng lẻ sẽ dễ hơn là một khối code lớn. Hơn nữa, giữ các crate trong workspace sẽ giúp việc phối hợp giữa các crate dễ dàng hơn nếu chúng thường được thay đổi cùng lúc.
