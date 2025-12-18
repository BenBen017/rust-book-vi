## Publishing a Crate to Crates.io

Chúng ta đã sử dụng các package từ [crates.io](https://crates.io/)<!-- ignore --> làm dependencies cho dự án của mình, nhưng bạn cũng có thể chia sẻ mã của mình với người khác bằng cách xuất bản package riêng. Registry của crate tại [crates.io](https://crates.io/)<!-- ignore --> phân phối mã nguồn của package của bạn, chủ yếu là mã nguồn mở.

Rust và Cargo có các tính năng giúp package bạn xuất bản dễ được tìm thấy và sử dụng hơn. Chúng ta sẽ nói về một số tính năng này và sau đó giải thích cách xuất bản package.

### Viết Bình Luận Tài Liệu Hữu Ích

Việc tài liệu hóa chính xác package sẽ giúp người dùng khác biết cách và khi nào sử dụng chúng, vì vậy đáng để đầu tư thời gian viết tài liệu. Trong Chương 3, chúng ta đã thảo luận cách comment code Rust sử dụng hai dấu gạch chéo, `//`. Rust cũng có một kiểu comment đặc biệt cho tài liệu, gọi tiện là *documentation comment*, sẽ tạo ra tài liệu HTML. HTML sẽ hiển thị nội dung của các comment tài liệu cho các API public, nhằm hướng dẫn lập trình viên cách *sử dụng* crate thay vì cách crate được *triển khai*.

Documentation comment sử dụng ba dấu gạch chéo, `///`, thay vì hai, và hỗ trợ Markdown để định dạng văn bản. Đặt comment ngay trước item mà nó tài liệu hóa. Listing 14-1 minh họa các comment tài liệu cho một hàm `add_one` trong crate tên là `my_crate`.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

<span class="caption">Listing 14-1: A documentation comment for a
function</span>

Ở đây, chúng ta cung cấp mô tả về chức năng của hàm `add_one`, bắt đầu một
section với tiêu đề `Examples`, và sau đó cung cấp code minh họa cách sử dụng
hàm `add_one`. Chúng ta có thể tạo tài liệu HTML từ comment này bằng cách chạy
`cargo doc`. Lệnh này sẽ chạy công cụ `rustdoc` đi kèm Rust và đặt tài liệu HTML
tạo ra trong thư mục *target/doc*.

Để tiện lợi, chạy `cargo doc --open` sẽ build HTML cho tài liệu crate hiện tại
(cũng như tài liệu cho tất cả dependencies của crate) và mở kết quả trong trình
duyệt web. Điều hướng đến hàm `add_one` và bạn sẽ thấy cách văn bản trong
comment được hiển thị, như minh họa trong Hình 14-1:

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figure 14-1: HTML documentation for the `add_one`
function</span>

#### Các Section Thường Dùng

Chúng ta đã sử dụng tiêu đề Markdown `# Examples` trong Listing 14-1 để tạo
một section trong HTML với tên “Examples.” Dưới đây là một số section khác mà
tác giả crate thường dùng trong tài liệu:

* **Panics**: Các tình huống mà hàm được tài liệu hóa có thể panic. Những
  người gọi hàm mà không muốn chương trình của họ panic cần đảm bảo không
  gọi hàm trong những tình huống này.
* **Errors**: Nếu hàm trả về `Result`, mô tả các loại lỗi có thể xảy ra và
  điều kiện nào gây ra lỗi sẽ giúp người dùng viết code xử lý các lỗi khác
  nhau một cách phù hợp.
* **Safety**: Nếu hàm là `unsafe` để gọi (chúng ta sẽ bàn về unsafety ở Chương
  19), cần có một section giải thích tại sao hàm là unsafe và các invariant mà
  hàm mong người gọi tuân thủ.

Hầu hết các comment tài liệu không cần tất cả các section này, nhưng đây là
một checklist hữu ích nhắc nhở bạn về những khía cạnh của code mà người dùng
sẽ quan tâm.

#### Documentation Comments as Tests

Thêm các khối code ví dụ trong comment tài liệu có thể giúp minh họa cách sử
dụng thư viện của bạn, và điều này còn có một lợi ích thêm: chạy `cargo test`
sẽ thực thi các ví dụ trong tài liệu như các test! Không gì tốt hơn tài liệu
có ví dụ minh họa. Nhưng cũng không gì tệ hơn các ví dụ không hoạt động vì
code đã thay đổi kể từ khi tài liệu được viết. Nếu chúng ta chạy `cargo test`
với tài liệu của hàm `add_one` từ Listing 14-1, chúng ta sẽ thấy một section
trong kết quả test như sau:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Bây giờ nếu chúng ta thay đổi hàm hoặc ví dụ sao cho `assert_eq!` trong ví dụ
panic và chạy lại `cargo test`, chúng ta sẽ thấy rằng các doc test phát hiện
ra rằng ví dụ và code không còn đồng bộ với nhau nữa!

#### Comment cho các Item Chứa Nó

Kiểu comment tài liệu `//!` sẽ thêm tài liệu vào item chứa các comment thay
vì các item theo sau. Chúng ta thường dùng kiểu doc comment này bên trong
file gốc của crate (*src/lib.rs* theo quy ước) hoặc bên trong một module để
document toàn bộ crate hoặc module.

Ví dụ, để thêm tài liệu mô tả mục đích của crate `my_crate` chứa hàm
`add_one`, chúng ta thêm các doc comment bắt đầu với `//!` vào đầu file
*src/lib.rs*, như được minh họa trong Listing 14-2:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

<span class="caption">Listing 14-2: Documentation cho toàn bộ crate `my_crate`</span>

Chú ý rằng không có code nào sau dòng cuối cùng bắt đầu bằng `//!`. Vì chúng
ta bắt đầu comment với `//!` thay vì `///`, chúng ta đang document item chứa
comment này thay vì một item theo sau. Trong trường hợp này, item đó là file
*src/lib.rs*, tức là crate root. Các comment này mô tả toàn bộ crate.

Khi chạy `cargo doc --open`, các comment này sẽ hiển thị trên trang chính
của tài liệu crate `my_crate` phía trên danh sách các item public trong crate,
như được minh họa trong Figure 14-2:

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Figure 14-2: Tài liệu được render cho `my_crate`,
bao gồm comment mô tả toàn bộ crate</span>

Các documentation comment bên trong các item rất hữu ích để mô tả crates
và modules. Dùng chúng để giải thích mục đích tổng thể của container để
giúp người dùng hiểu tổ chức crate.

### Export một Public API thuận tiện với `pub use`

Cấu trúc của public API là yếu tố quan trọng khi xuất bản crate. Người dùng
crate của bạn không quen với cấu trúc bằng bạn và có thể gặp khó khăn khi
tìm các phần họ muốn dùng nếu crate có hierarchy module lớn.

Trong Chapter 7, chúng ta đã học cách làm item public với từ khóa `pub`,
và đưa item vào scope với từ khóa `use`. Tuy nhiên, cấu trúc hợp lý cho bạn
khi phát triển crate có thể không thuận tiện cho người khác. Bạn có thể
muốn tổ chức các struct theo một hierarchy nhiều cấp, nhưng người dùng muốn
dùng một type trong cấp sâu có thể khó biết type đó tồn tại. Họ cũng có
thể phiền khi phải viết `use my_crate::some_module::another_module::UsefulType;`
thay vì `use my_crate::UsefulType;`.

Tin tốt là nếu cấu trúc *không* tiện lợi cho người khác dùng từ thư viện
khác, bạn không cần phải thay đổi tổ chức nội bộ: thay vào đó, bạn có thể
re-export item để tạo cấu trúc public khác với cấu trúc private bằng cách dùng
`pub use`. Re-export sẽ lấy một public item ở một nơi và làm nó public ở nơi
khác, như thể item được định nghĩa ở vị trí khác đó.

Ví dụ, giả sử chúng ta tạo một library tên là `art` để mô hình hóa các
khái niệm nghệ thuật. Trong library này có hai module: module `kinds` chứa
hai enum `PrimaryColor` và `SecondaryColor`, và module `utils` chứa hàm `mix`,
như được minh họa trong Listing 14-3:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

<span class="caption">Listing 14-3: Một thư viện `art` với các item được tổ chức
trong các module `kinds` và `utils`</span>

Figure 14-3 minh họa trang chính của tài liệu crate này được tạo bởi
`cargo doc`:

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Figure 14-3: Trang chính của tài liệu crate `art`
liệt kê các module `kinds` và `utils`</span>

Chú ý rằng các type `PrimaryColor` và `SecondaryColor` không được liệt kê
trên trang chính, cũng như hàm `mix`. Chúng ta phải click vào `kinds` và
`utils` để xem chúng.

Một crate khác phụ thuộc vào thư viện này sẽ cần các câu lệnh `use` để
đưa các item từ `art` vào scope, chỉ ra cấu trúc module hiện tại. Listing 14-4
minh họa một ví dụ crate sử dụng các item `PrimaryColor` và `mix` từ crate
`art`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

<span class="caption">Listing 14-4: Một crate sử dụng các item của crate `art` với
cấu trúc nội bộ của nó</span>

Tác giả của code trong Listing 14-4, sử dụng crate `art`, phải xác định rằng
`PrimaryColor` nằm trong module `kinds` và `mix` nằm trong module `utils`. 
Cấu trúc module của crate `art` quan trọng hơn với các developer làm việc
trên crate `art` hơn là với những người dùng nó. Cấu trúc nội bộ không chứa
thông tin hữu ích cho ai đó đang cố gắng hiểu cách sử dụng crate `art`,
mà ngược lại gây nhầm lẫn vì các developer phải tìm xem nên look vào đâu
và phải chỉ rõ tên module trong các câu lệnh `use`.

Để loại bỏ cấu trúc nội bộ khỏi API công khai, chúng ta có thể chỉnh sửa
code crate `art` trong Listing 14-3 để thêm các câu lệnh `pub use` để
re-export các item lên cấp top-level, như minh họa trong Listing 14-5:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

<span class="caption">Listing 14-5: Thêm các câu lệnh `pub use` để re-export
các item</span>

Tài liệu API mà `cargo doc` tạo ra cho crate này giờ sẽ liệt kê và liên kết
các re-export trên trang chính, như minh họa trong Figure 14-4, giúp
các type `PrimaryColor` và `SecondaryColor` cũng như hàm `mix` dễ tìm hơn.

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Figure 14-4: Trang chính của tài liệu crate `art`
liệt kê các re-exports</span>

Người dùng crate `art` vẫn có thể xem và sử dụng cấu trúc nội bộ từ Listing
14-3 như minh họa trong Listing 14-4, hoặc họ có thể sử dụng cấu trúc
tiện lợi hơn trong Listing 14-5, như minh họa trong Listing 14-6:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

<span class="caption">Listing 14-6: Một chương trình sử dụng các item được re-export từ
crate `art`</span>

Trong trường hợp có nhiều module lồng nhau, việc re-export các type ở cấp
trên cùng với `pub use` có thể cải thiện đáng kể trải nghiệm của người
dùng crate. Một cách sử dụng phổ biến khác của `pub use` là re-export
các định nghĩa từ một dependency trong crate hiện tại để biến các
định nghĩa đó thành một phần của public API crate của bạn.

Tạo một cấu trúc public API hữu ích thường là nghệ thuật hơn là khoa học,
và bạn có thể lặp lại để tìm ra API phù hợp nhất với người dùng của bạn.
Chọn `pub use` giúp bạn linh hoạt trong cách cấu trúc crate nội bộ và
tách biệt cấu trúc nội bộ đó khỏi những gì bạn trình bày cho người dùng.
Hãy xem một số code của các crate bạn đã cài để xem liệu cấu trúc nội bộ
của chúng có khác với public API không.

### Thiết lập tài khoản Crates.io

Trước khi bạn có thể publish bất kỳ crate nào, bạn cần tạo một tài khoản
trên [crates.io](https://crates.io/)<!-- ignore --> và lấy API token. Để làm
điều này, truy cập trang chủ tại [crates.io](https://crates.io/)<!-- ignore -->
và đăng nhập qua tài khoản GitHub. (Hiện tại tài khoản GitHub là bắt buộc,
nhưng trang web có thể hỗ trợ các cách tạo tài khoản khác trong tương lai.)
Khi đã đăng nhập, truy cập cài đặt tài khoản của bạn tại
[https://crates.io/me/](https://crates.io/me/)<!-- ignore --> và lấy API key.
Sau đó chạy lệnh `cargo login` với API key của bạn, như sau:

```console
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

Lệnh này sẽ thông báo cho Cargo về API token của bạn và lưu trữ nó cục bộ
tại *~/.cargo/credentials*. Lưu ý rằng token này là một *bí mật*: không
chia sẻ nó với bất kỳ ai khác. Nếu bạn đã chia sẻ token với người khác vì
bất kỳ lý do gì, bạn nên thu hồi nó và tạo một token mới trên
[crates.io](https://crates.io/)<!-- ignore -->.

### Thêm Metadata cho Crate Mới

Giả sử bạn có một crate mà bạn muốn publish. Trước khi publish, bạn cần
thêm một số metadata vào phần `[package]` trong file *Cargo.toml* của crate.

Crate của bạn cần có một tên duy nhất. Khi làm việc với crate cục bộ, bạn
có thể đặt tên crate tùy ý. Tuy nhiên, tên crate trên
[crates.io](https://crates.io/)<!-- ignore --> được cấp theo cơ chế
first-come, first-served. Khi tên crate đã được dùng, không ai khác có thể
publish crate với tên đó. Trước khi thử publish, hãy tìm kiếm tên bạn muốn
sử dụng. Nếu tên đó đã được dùng, bạn sẽ cần tìm tên khác và chỉnh sửa
trường `name` trong file *Cargo.toml* ở phần `[package]` để sử dụng tên mới
khi publish, như sau:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Ngay cả khi bạn đã chọn một tên duy nhất, khi chạy `cargo publish` để
publish crate vào thời điểm này, bạn sẽ nhận được một cảnh báo và sau đó là
một lỗi:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error: missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for how to upload metadata
```

Lỗi này xảy ra vì bạn đang thiếu một số thông tin quan trọng: bạn phải có
`description` và `license` để người khác biết crate của bạn làm gì và dưới
điều kiện nào họ có thể sử dụng. Trong *Cargo.toml*, thêm một `description`
chỉ vài câu ngắn, vì nó sẽ hiển thị trong kết quả tìm kiếm crate của bạn.
Đối với trường `license`, bạn cần cung cấp một *license identifier*.
[Linux Foundation’s Software Package Data Exchange (SPDX)][spdx] liệt kê các
identifier mà bạn có thể dùng. Ví dụ, để chỉ ra rằng bạn cấp phép crate
bằng MIT License, thêm identifier `MIT`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Nếu bạn muốn dùng một license không có trong danh sách SPDX, bạn cần đặt
nội dung license đó vào một file, thêm file đó vào dự án, và dùng `license-file`
để chỉ định tên file đó thay vì dùng khóa `license`.

Hướng dẫn chọn license phù hợp cho dự án của bạn nằm ngoài phạm vi cuốn
sách này. Nhiều người trong cộng đồng Rust cấp phép dự án của họ giống như
Rust bằng cách dùng dual license `MIT OR Apache-2.0`. Cách làm này cũng
chứng minh rằng bạn có thể chỉ định nhiều license bằng cách dùng `OR` để
cho phép nhiều license cho dự án của bạn.

Với tên duy nhất, phiên bản, description, và license đã thêm vào, file
*Cargo.toml* cho một dự án sẵn sàng publish có thể trông như sau:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo’s documentation](https://doc.rust-lang.org/cargo/) mô tả các metadata
khác mà bạn có thể chỉ định để giúp người khác dễ dàng tìm thấy và sử dụng
crate của bạn hơn.

### Publishing to Crates.io

Bây giờ bạn đã tạo tài khoản, lưu API token, chọn tên cho crate của bạn,
và chỉ định các metadata cần thiết, bạn đã sẵn sàng để publish! Việc publish
một crate sẽ tải một phiên bản cụ thể lên [crates.io](https://crates.io/) 
cho người khác sử dụng.

Hãy cẩn thận, vì việc publish là *vĩnh viễn*. Phiên bản đó không thể bị
ghi đè, và code không thể bị xóa. Một mục tiêu chính của [crates.io](https://crates.io/) 
là làm một kho lưu trữ vĩnh viễn của code để các build của mọi dự án phụ thuộc
vào các crate từ [crates.io](https://crates.io/) vẫn tiếp tục hoạt động.
Cho phép xóa phiên bản sẽ phá vỡ mục tiêu này. Tuy nhiên, không có giới
hạn về số phiên bản crate mà bạn có thể publish.

Chạy lệnh `cargo publish` lần nữa. Lúc này lệnh sẽ thành công:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Chúc mừng! Bây giờ bạn đã chia sẻ code của mình với cộng đồng Rust, và
bất kỳ ai cũng có thể dễ dàng thêm crate của bạn làm dependency trong dự án
của họ.

### Publishing a New Version of an Existing Crate

Khi bạn đã thực hiện thay đổi cho crate và sẵn sàng phát hành phiên bản
mới, hãy thay đổi giá trị `version` trong *Cargo.toml* của bạn và publish
lại. Sử dụng các quy tắc [Semantic Versioning][semver] để quyết định số
phiên bản tiếp theo phù hợp dựa trên các loại thay đổi bạn đã thực hiện.
Sau đó chạy `cargo publish` để tải phiên bản mới lên.

### Deprecating Versions from Crates.io with `cargo yank`

Mặc dù bạn không thể xóa các phiên bản trước của crate, bạn có thể ngăn
các dự án mới thêm chúng làm dependency mới. Điều này hữu ích khi một
phiên bản crate bị lỗi vì lý do nào đó. Trong những tình huống này, Cargo
hỗ trợ *yanking* một phiên bản crate.

Yanking một phiên bản ngăn các dự án mới phụ thuộc vào phiên bản đó
trong khi vẫn cho phép tất cả các dự án hiện có đang phụ thuộc vào nó
tiếp tục hoạt động. Về cơ bản, yank đảm bảo rằng tất cả các dự án có
*Cargo.lock* sẽ không bị phá vỡ, và bất kỳ *Cargo.lock* mới nào được
tạo ra sẽ không sử dụng phiên bản bị yank.

Để yank một phiên bản crate, trong thư mục của crate mà bạn đã publish
trước đó, chạy `cargo yank` và chỉ định phiên bản bạn muốn yank. Ví dụ,
nếu chúng ta đã publish crate có tên `guessing_game` phiên bản 1.0.1
và muốn yank nó, trong thư mục dự án của `guessing_game` chúng ta sẽ chạy:

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Bằng cách thêm `--undo` vào lệnh, bạn cũng có thể hoàn tác một yank và
cho phép các dự án bắt đầu phụ thuộc vào phiên bản đó một lần nữa:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Một yank *không* xóa bất kỳ mã nào. Ví dụ, nó không thể xóa các thông tin bí mật
bị tải lên nhầm. Nếu điều đó xảy ra, bạn phải ngay lập tức đặt lại những thông tin bí mật đó.

[spdx]: http://spdx.org/licenses/
[semver]: http://semver.org/
