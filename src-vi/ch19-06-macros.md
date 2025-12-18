## Macros

Chúng ta đã sử dụng các macro như `println!` trong suốt cuốn sách này, nhưng chúng ta chưa khám phá đầy đủ macro là gì và cách chúng hoạt động. Thuật ngữ *macro* đề cập đến một tập hợp các tính năng trong Rust: *macro khai báo* với `macro_rules!` và ba loại *macro thủ tục*:

* Macro `#[derive]` tùy chỉnh, xác định mã được thêm với thuộc tính `derive` sử dụng trên struct và enum
* Macro kiểu thuộc tính, định nghĩa các thuộc tính tùy chỉnh có thể sử dụng trên bất kỳ item nào
* Macro kiểu hàm, trông giống như các lời gọi hàm nhưng hoạt động trên các token được chỉ định làm đối số

Chúng ta sẽ lần lượt nói về từng loại, nhưng trước tiên, hãy xem lý do tại sao chúng ta cần macro khi đã có hàm.

### Sự khác biệt giữa Macros và Functions

Về cơ bản, macros là cách viết mã mà tự nó viết ra mã khác, điều này được gọi là *metaprogramming*. Trong Phụ lục C, chúng ta thảo luận về thuộc tính `derive`, tạo ra việc triển khai các trait khác nhau cho bạn. Chúng ta cũng đã sử dụng các macro `println!` và `vec!` trong suốt cuốn sách. Tất cả các macro này *mở rộng* để tạo ra nhiều mã hơn so với mã bạn viết thủ công.

Metaprogramming hữu ích để giảm lượng mã bạn phải viết và duy trì, điều này cũng là một trong những vai trò của hàm. Tuy nhiên, macros có một số khả năng bổ sung mà hàm không có.

Một chữ ký hàm phải khai báo số lượng và kiểu của các tham số mà hàm có. Ngược lại, macros có thể nhận một số lượng biến đổi của các tham số: chúng ta có thể gọi `println!("hello")` với một đối số hoặc `println!("hello {}", name)` với hai đối số. Ngoài ra, macros được mở rộng trước khi trình biên dịch giải thích ý nghĩa của mã, vì vậy một macro có thể, ví dụ, triển khai một trait trên một kiểu nhất định. Một hàm không thể làm được điều đó, vì nó được gọi tại runtime và một trait cần được triển khai tại compile time.

Nhược điểm của việc triển khai macro thay vì hàm là các định nghĩa macro phức tạp hơn định nghĩa hàm vì bạn đang viết mã Rust mà tự nó viết ra mã Rust khác. Do sự gián tiếp này, các định nghĩa macro thường khó đọc, hiểu và duy trì hơn so với định nghĩa hàm.

Một khác biệt quan trọng khác giữa macros và hàm là bạn phải định nghĩa macro hoặc đưa chúng vào phạm vi *trước* khi gọi chúng trong một file, khác với các hàm mà bạn có thể định nghĩa ở bất cứ đâu và gọi ở bất cứ đâu.

### Macros Khai Báo với `macro_rules!` cho Metaprogramming Chung

Hình thức macro được sử dụng rộng rãi nhất trong Rust là *macro khai báo*. Chúng đôi khi còn được gọi là “macros theo ví dụ”, “`macro_rules!` macros,” hoặc đơn giản là “macros.” Về bản chất, macro khai báo cho phép bạn viết một cái gì đó tương tự như biểu thức `match` trong Rust. Như đã thảo luận trong Chương 6, biểu thức `match` là các cấu trúc điều khiển nhận một biểu thức, so sánh giá trị kết quả của biểu thức với các pattern, và sau đó chạy mã liên quan đến pattern khớp. Macro cũng so sánh một giá trị với các pattern được liên kết với mã cụ thể: trong trường hợp này, giá trị là mã nguồn Rust literal được truyền vào macro; các pattern được so sánh với cấu trúc của mã nguồn đó; và mã liên kết với mỗi pattern, khi khớp, sẽ thay thế mã được truyền vào macro. Tất cả điều này xảy ra trong quá trình biên dịch.

Để định nghĩa một macro, bạn sử dụng cấu trúc `macro_rules!`. Hãy cùng khám phá cách sử dụng `macro_rules!` bằng cách xem cách macro `vec!` được định nghĩa. Chương 8 đã đề cập cách chúng ta có thể sử dụng macro `vec!` để tạo một vector mới với các giá trị cụ thể. Ví dụ, macro sau tạo một vector mới chứa ba số nguyên:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Chúng ta cũng có thể sử dụng macro `vec!` để tạo một vector gồm hai số nguyên hoặc một vector gồm năm string slice. Chúng ta sẽ không thể sử dụng hàm để làm điều tương tự vì không biết trước số lượng hoặc kiểu của các giá trị.

Listing 19-28 cho thấy một định nghĩa hơi đơn giản hóa của macro `vec!`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-28/src/lib.rs}}
```

<span class="caption">Listing 19-28: Phiên bản đơn giản hóa của định nghĩa macro `vec!`</span>

> Lưu ý: Định nghĩa thực tế của macro `vec!` trong thư viện chuẩn bao gồm mã để cấp phát trước lượng bộ nhớ chính xác. Mã đó là một tối ưu hóa mà chúng ta không bao gồm ở đây để làm ví dụ đơn giản hơn.

Chú thích `#[macro_export]` chỉ ra rằng macro này nên có sẵn bất cứ khi nào crate chứa macro được đưa vào phạm vi. Nếu không có chú thích này, macro sẽ không thể được đưa vào phạm vi.

Chúng ta bắt đầu định nghĩa macro với `macro_rules!` và tên của macro mà chúng ta đang định nghĩa *không có* dấu chấm than. Tên, trong trường hợp này là `vec`, được theo sau bởi dấu ngoặc nhọn xác định thân của định nghĩa macro.

Cấu trúc trong thân `vec!` tương tự cấu trúc của một biểu thức `match`. Ở đây chúng ta có một nhánh với pattern `( $( $x:expr ),* )`, theo sau bởi `=>` và khối mã liên kết với pattern này. Nếu pattern khớp, khối mã liên kết sẽ được tạo ra. Vì đây là pattern duy nhất trong macro này, chỉ có một cách hợp lệ để khớp; bất kỳ pattern nào khác sẽ dẫn đến lỗi. Các macro phức tạp hơn sẽ có nhiều nhánh hơn.

Cú pháp pattern hợp lệ trong định nghĩa macro khác với cú pháp pattern được trình bày trong Chương 18 vì các macro pattern được khớp với cấu trúc mã Rust thay vì giá trị. Hãy cùng phân tích các thành phần pattern trong Listing 19-28; để xem toàn bộ cú pháp pattern của macro, tham khảo [Rust Reference][ref].

Đầu tiên, chúng ta sử dụng một cặp dấu ngoặc để bao quanh toàn bộ pattern. Chúng ta sử dụng dấu đô la (`$`) để khai báo một biến trong hệ thống macro sẽ chứa mã Rust khớp với pattern. Dấu đô la làm rõ rằng đây là một biến macro, khác với biến Rust thông thường. Tiếp theo là một cặp dấu ngoặc để bắt các giá trị khớp với pattern bên trong dấu ngoặc để sử dụng trong mã thay thế. Bên trong `$()` là `$x:expr`, khớp với bất kỳ biểu thức Rust nào và gán tên `$x` cho biểu thức đó.

Dấu phẩy theo sau `$()` chỉ ra rằng một ký tự dấu phẩy thực tế có thể xuất hiện tùy chọn sau mã khớp với `$()`. Dấu `*` chỉ ra rằng pattern khớp với không hoặc nhiều lần bất cứ thứ gì đứng trước `*`.

Khi chúng ta gọi macro này với `vec![1, 2, 3];`, pattern `$x` khớp ba lần với ba biểu thức `1`, `2`, và `3`.

Bây giờ hãy xem pattern trong thân mã liên kết với nhánh này: `temp_vec.push()` trong `$()*` được sinh ra cho mỗi phần khớp với `$()` trong pattern không hoặc nhiều lần tùy thuộc vào số lần pattern khớp. `$x` được thay thế bằng mỗi biểu thức khớp. Khi chúng ta gọi macro này với `vec![1, 2, 3];`, mã được sinh ra thay thế cho lời gọi macro này sẽ là:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Chúng ta đã định nghĩa một macro có thể nhận bất kỳ số lượng đối số nào với bất kỳ kiểu nào và có thể sinh mã để tạo một vector chứa các phần tử được chỉ định.

Để tìm hiểu thêm về cách viết macro, tham khảo tài liệu trực tuyến hoặc các nguồn khác, chẳng hạn như [“The Little Book of Rust Macros”][tlborm] do Daniel Keep khởi xướng và tiếp tục bởi Lukas Wirth.

### Macros Thủ Tục để Sinh Mã từ Thuộc Tính

Hình thức thứ hai của macro là *macro thủ tục*, hoạt động giống như một hàm (và là một loại thủ tục). Macros thủ tục nhận một số mã làm đầu vào, thao tác trên mã đó, và tạo ra một số mã làm đầu ra thay vì so khớp với các pattern và thay thế mã như các macro khai báo. Ba loại macros thủ tục là custom derive, attribute-like, và function-like, và tất cả hoạt động theo cách tương tự.

Khi tạo macros thủ tục, định nghĩa phải nằm trong một crate riêng với loại crate đặc biệt. Điều này do các lý do kỹ thuật phức tạp mà chúng ta hy vọng sẽ loại bỏ trong tương lai. Trong Listing 19-29, chúng ta minh họa cách định nghĩa một macro thủ tục, trong đó `some_attribute` là một chỗ giữ chỗ cho việc sử dụng một loại macro cụ thể.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

<span class="caption">Listing 19-29: Ví dụ về việc định nghĩa một procedural macro</span>

Hàm định nghĩa một procedural macro nhận một `TokenStream` làm đầu vào và tạo ra một `TokenStream` làm đầu ra. Kiểu `TokenStream` được định nghĩa bởi crate `proc_macro` đi kèm với Rust và đại diện cho một chuỗi các token. Đây là phần cốt lõi của macro: mã nguồn mà macro thao tác tạo thành `TokenStream` đầu vào, và mã mà macro sinh ra là `TokenStream` đầu ra. Hàm cũng có một thuộc tính đi kèm chỉ ra loại procedural macro mà chúng ta đang tạo. Chúng ta có thể có nhiều loại procedural macro trong cùng một crate.

Hãy xem các loại procedural macro khác nhau. Chúng ta sẽ bắt đầu với một custom derive macro và sau đó giải thích những khác biệt nhỏ khiến các dạng khác khác nhau.

### Cách Viết Custom `derive` Macro

Hãy tạo một crate có tên `hello_macro` định nghĩa một trait có tên `HelloMacro` với một hàm liên kết là `hello_macro`. Thay vì yêu cầu người dùng triển khai trait `HelloMacro` cho từng kiểu của họ, chúng ta sẽ cung cấp một procedural macro để người dùng có thể chú thích kiểu của họ với `#[derive(HelloMacro)]` để nhận triển khai mặc định của hàm `hello_macro`. Triển khai mặc định sẽ in ra `Hello, Macro! My name is TypeName!`, trong đó `TypeName` là tên của kiểu mà trait này được định nghĩa. Nói cách khác, chúng ta sẽ viết một crate cho phép lập trình viên khác viết mã như Listing 19-30 sử dụng crate của chúng ta.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-30/src/main.rs}}
```

<span class="caption">Listing 19-30: Mã mà người dùng crate của chúng ta có thể viết khi sử dụng procedural macro</span>

Đoạn mã này sẽ in ra `Hello, Macro! My name is Pancakes!` khi hoàn tất. Bước đầu tiên là tạo một crate thư viện mới, như sau:

```console
$ cargo new hello_macro --lib
```

Tiếp theo, chúng ta sẽ định nghĩa trait `HelloMacro` và hàm liên kết của nó:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/hello_macro/src/lib.rs}}
```

Chúng ta đã có một trait và hàm của nó. Tại thời điểm này, người dùng crate của chúng ta có thể triển khai trait để đạt được chức năng mong muốn, như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

Tuy nhiên, họ sẽ phải viết khối triển khai cho từng kiểu mà họ muốn sử dụng với `hello_macro`; chúng ta muốn giúp họ khỏi phải làm việc này.

Ngoài ra, chúng ta chưa thể cung cấp hàm `hello_macro` với triển khai mặc định in ra tên của kiểu mà trait được triển khai trên đó: Rust không có khả năng reflection, vì vậy không thể tra cứu tên kiểu tại runtime. Chúng ta cần một macro để sinh mã tại thời gian biên dịch.

Bước tiếp theo là định nghĩa procedural macro. Tại thời điểm viết, các procedural macro cần nằm trong một crate riêng. Trong tương lai, hạn chế này có thể được loại bỏ. Quy ước cấu trúc crate và crate macro như sau: đối với một crate có tên `foo`, crate procedural macro custom derive sẽ được gọi là `foo_derive`. Hãy bắt đầu một crate mới có tên `hello_macro_derive` bên trong dự án `hello_macro` của chúng ta:

```console
$ cargo new hello_macro_derive --lib
```

Hai crate của chúng ta có mối quan hệ chặt chẽ, vì vậy chúng ta tạo crate procedural macro bên trong thư mục của crate `hello_macro`. Nếu chúng ta thay đổi định nghĩa trait trong `hello_macro`, chúng ta cũng sẽ phải thay đổi triển khai của procedural macro trong `hello_macro_derive`. Hai crate sẽ cần được phát hành riêng, và lập trình viên sử dụng các crate này sẽ cần thêm cả hai làm dependencies và đưa cả hai vào phạm vi. Chúng ta có thể để crate `hello_macro` sử dụng `hello_macro_derive` làm dependency và re-export mã procedural macro. Tuy nhiên, cách chúng ta cấu trúc dự án cho phép lập trình viên sử dụng `hello_macro` ngay cả khi họ không cần chức năng `derive`.

Chúng ta cần khai báo crate `hello_macro_derive` là một procedural macro crate. Chúng ta cũng sẽ cần chức năng từ các crate `syn` và `quote`, như bạn sẽ thấy ngay sau đây, vì vậy cần thêm chúng làm dependencies. Thêm các dòng sau vào file *Cargo.toml* của `hello_macro_derive`:

<span class="filename">Filename: hello_macro_derive/Cargo.toml</span>

```toml
{{#include ../listings/ch19-advanced-features/listing-19-31/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

Để bắt đầu định nghĩa procedural macro, đặt mã trong Listing 19-31 vào file *src/lib.rs* của crate `hello_macro_derive`. Lưu ý rằng mã này sẽ chưa biên dịch cho đến khi chúng ta thêm định nghĩa cho hàm `impl_hello_macro`.

<span class="filename">Filename: hello_macro_derive/src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-31/hello_macro/hello_macro_derive/src/lib.rs}}
```

<span class="caption">Listing 19-31: Mã mà hầu hết các crate procedural macro sẽ cần để xử lý mã Rust</span>

Lưu ý rằng chúng ta đã tách mã thành hàm `hello_macro_derive`, chịu trách nhiệm phân tích `TokenStream`, và hàm `impl_hello_macro`, chịu trách nhiệm biến đổi cây cú pháp: điều này giúp việc viết procedural macro thuận tiện hơn. Mã trong hàm ngoài (`hello_macro_derive` trong trường hợp này) sẽ giống nhau cho hầu hết mọi crate procedural macro mà bạn thấy hoặc tạo. Mã bạn chỉ định trong thân của hàm bên trong (`impl_hello_macro` trong trường hợp này) sẽ khác nhau tùy thuộc vào mục đích của procedural macro của bạn.

Chúng ta đã giới thiệu ba crate mới: `proc_macro`, [`syn`] và [`quote`]. Crate `proc_macro` đi kèm với Rust, vì vậy chúng ta không cần thêm nó vào dependencies trong *Cargo.toml*. Crate `proc_macro` là API của compiler cho phép chúng ta đọc và thao tác mã Rust từ mã của chính chúng ta.

Crate `syn` phân tích mã Rust từ một chuỗi thành một cấu trúc dữ liệu mà chúng ta có thể thao tác. Crate `quote` biến các cấu trúc dữ liệu của `syn` trở lại thành mã Rust. Các crate này làm cho việc phân tích bất kỳ loại mã Rust nào mà chúng ta muốn xử lý trở nên đơn giản hơn nhiều: viết một parser đầy đủ cho mã Rust không phải là việc đơn giản.

Hàm `hello_macro_derive` sẽ được gọi khi người dùng thư viện của chúng ta chỉ định `#[derive(HelloMacro)]` trên một kiểu. Điều này khả thi vì chúng ta đã chú thích hàm `hello_macro_derive` với `proc_macro_derive` và chỉ định tên `HelloMacro`, khớp với tên trait của chúng ta; đây là quy ước mà hầu hết procedural macro tuân theo.

Hàm `hello_macro_derive` trước tiên chuyển `input` từ `TokenStream` thành một cấu trúc dữ liệu mà chúng ta có thể diễn giải và thao tác. Đây là nơi crate `syn` phát huy tác dụng. Hàm `parse` trong `syn` nhận một `TokenStream` và trả về một struct `DeriveInput` đại diện cho mã Rust đã được phân tích. Listing 19-32 hiển thị các phần liên quan của struct `DeriveInput` mà chúng ta nhận được từ việc phân tích chuỗi `struct Pancakes;`:

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

<span class="caption">Listing 19-32: Instance `DeriveInput` chúng ta nhận được khi phân tích mã có thuộc tính macro trong Listing 19-30</span>

Các trường của struct này cho thấy mã Rust mà chúng ta đã phân tích là một unit struct với `ident` (identifier, nghĩa là tên) là `Pancakes`. Có nhiều trường khác trong struct này để mô tả đủ loại mã Rust; xem tài liệu [`syn` về `DeriveInput`][syn-docs] để biết thêm thông tin.

Chúng ta sắp định nghĩa hàm `impl_hello_macro`, nơi chúng ta sẽ xây dựng mã Rust mới mà muốn thêm vào. Nhưng trước khi làm điều đó, lưu ý rằng đầu ra của macro derive của chúng ta cũng là một `TokenStream`. `TokenStream` trả về được thêm vào mã mà người dùng crate của chúng ta viết, vì vậy khi họ biên dịch crate của họ, họ sẽ nhận được chức năng bổ sung mà chúng ta cung cấp trong `TokenStream` đã được sửa đổi.

Bạn có thể nhận thấy chúng ta đang gọi `unwrap` để làm cho hàm `hello_macro_derive` panic nếu lời gọi đến hàm `syn::parse` thất bại ở đây. Điều này cần thiết vì procedural macro của chúng ta phải panic khi có lỗi vì các hàm `proc_macro_derive` phải trả về `TokenStream` thay vì `Result` để tuân theo API procedural macro. Chúng ta đã đơn giản hóa ví dụ này bằng cách sử dụng `unwrap`; trong mã sản xuất, bạn nên cung cấp thông báo lỗi cụ thể hơn về những gì đã xảy ra bằng cách sử dụng `panic!` hoặc `expect`.

Bây giờ khi chúng ta có mã để chuyển đổi mã Rust được chú thích từ `TokenStream` thành một instance `DeriveInput`, hãy sinh mã thực thi trait `HelloMacro` trên kiểu được chú thích, như được trình bày trong Listing 19-33.

<span class="filename">Filename: hello_macro_derive/src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-33/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

<span class="caption">Listing 19-33: Triển khai trait `HelloMacro` sử dụng mã Rust đã được phân tích</span>

Chúng ta nhận được một instance struct `Ident` chứa tên (identifier) của kiểu được chú thích bằng cách sử dụng `ast.ident`. Struct trong Listing 19-32 cho thấy khi chạy hàm `impl_hello_macro` trên mã trong Listing 19-30, `ident` nhận được sẽ có trường `ident` với giá trị `"Pancakes"`. Do đó, biến `name` trong Listing 19-33 sẽ chứa một instance struct `Ident` mà khi in ra sẽ là chuỗi `"Pancakes"`, tên của struct trong Listing 19-30.

Macro `quote!` cho phép chúng ta định nghĩa mã Rust mà chúng ta muốn trả về. Compiler mong đợi thứ gì đó khác so với kết quả trực tiếp của việc thực thi macro `quote!`, vì vậy chúng ta cần chuyển đổi nó thành `TokenStream`. Chúng ta làm điều này bằng cách gọi phương thức `into`, phương thức này tiêu thụ biểu diễn trung gian và trả về giá trị kiểu `TokenStream` yêu cầu.

Macro `quote!` cũng cung cấp cơ chế templating rất tiện lợi: chúng ta có thể viết `#name`, và `quote!` sẽ thay thế bằng giá trị trong biến `name`. Bạn thậm chí có thể thực hiện một số lặp lại tương tự cách hoạt động của các macro thông thường. Xem [tài liệu crate `quote`][quote-docs] để có giới thiệu chi tiết.

Chúng ta muốn procedural macro sinh ra triển khai trait `HelloMacro` cho kiểu mà người dùng đã chú thích, điều này có thể lấy bằng cách sử dụng `#name`. Triển khai trait có một hàm là `hello_macro`, thân hàm chứa chức năng mà chúng ta muốn cung cấp: in ra `Hello, Macro! My name is` và sau đó là tên của kiểu được chú thích.

Macro `stringify!` được sử dụng ở đây là built-in trong Rust. Nó nhận một biểu thức Rust, chẳng hạn như `1 + 2`, và tại thời gian biên dịch chuyển biểu thức đó thành một literal string, ví dụ `"1 + 2"`. Điều này khác với `format!` hoặc `println!`, các macro này sẽ đánh giá biểu thức rồi chuyển kết quả thành một `String`. Có khả năng input `#name` là một biểu thức cần in ra nguyên văn, vì vậy chúng ta dùng `stringify!`. Sử dụng `stringify!` cũng tiết kiệm một phép cấp phát bằng cách chuyển `#name` thành literal string tại thời gian biên dịch.

Ở thời điểm này, `cargo build` nên hoàn tất thành công cả trong `hello_macro` và `hello_macro_derive`. Hãy kết nối các crate này với mã trong Listing 19-30 để xem procedural macro hoạt động! Tạo một dự án nhị phân mới trong thư mục *projects* bằng lệnh `cargo new pancakes`. Chúng ta cần thêm `hello_macro` và `hello_macro_derive` làm dependencies trong *Cargo.toml* của crate `pancakes`. Nếu bạn xuất bản các phiên bản của `hello_macro` và `hello_macro_derive` lên [crates.io](https://crates.io/), chúng sẽ là dependencies thông thường; nếu không, bạn có thể chỉ định chúng là dependencies theo `path` như sau:

```toml
{{#include ../listings/ch19-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:7:9}}
```

Đặt mã trong Listing 19-30 vào *src/main.rs*, và chạy `cargo run`: nó sẽ in ra `Hello, Macro! My name is Pancakes!`. Triển khai trait `HelloMacro` từ procedural macro đã được thêm vào mà crate `pancakes` không cần tự triển khai; `#[derive(HelloMacro)]` đã thêm triển khai trait này.

Tiếp theo, hãy khám phá cách các loại procedural macro khác khác với custom derive macros.

### Macros kiểu Attribute

Macros kiểu attribute tương tự như custom derive macros, nhưng thay vì sinh mã cho thuộc tính `derive`, chúng cho phép bạn tạo các thuộc tính mới. Chúng cũng linh hoạt hơn: `derive` chỉ hoạt động cho structs và enums; các attribute có thể áp dụng cho các item khác, chẳng hạn như các hàm. Dưới đây là ví dụ sử dụng macro kiểu attribute: giả sử bạn có một attribute tên `route` để chú thích các hàm khi sử dụng một framework ứng dụng web:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Attribute `#[route]` này sẽ được framework định nghĩa như một procedural macro. Chữ ký của hàm định nghĩa macro sẽ trông như sau:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Ở đây, chúng ta có hai tham số kiểu `TokenStream`. Tham số đầu tiên dành cho nội dung của attribute: phần `GET, "/"`. Tham số thứ hai là thân của item mà attribute được gắn vào: trong trường hợp này là `fn index() {}` và phần còn lại của thân hàm.

Ngoài ra, macros kiểu attribute hoạt động giống hệt như custom derive macros: bạn tạo một crate với loại crate `proc-macro` và triển khai một hàm sinh ra mã bạn muốn!

### Macros kiểu Function

Macros kiểu function định nghĩa các macro trông giống như lời gọi hàm. Tương tự như các macro `macro_rules!`, chúng linh hoạt hơn các hàm thông thường; ví dụ, chúng có thể nhận số lượng đối số không xác định. Tuy nhiên, các macro `macro_rules!` chỉ có thể được định nghĩa bằng cú pháp giống match mà chúng ta đã thảo luận trong phần [“Declarative Macros with `macro_rules!` for General Metaprogramming”][decl]<!-- ignore --> trước đó. Các macro kiểu function nhận một tham số `TokenStream` và định nghĩa của chúng thao tác với `TokenStream` đó bằng mã Rust giống như hai loại procedural macro khác. Một ví dụ về macro kiểu function là macro `sql!` có thể được gọi như sau:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Macro này sẽ phân tích câu lệnh SQL bên trong nó và kiểm tra xem cú pháp có đúng hay không, điều này phức tạp hơn nhiều so với những gì một macro `macro_rules!` có thể làm. Macro `sql!` sẽ được định nghĩa như sau:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Định nghĩa này tương tự như chữ ký của macro custom derive: chúng ta nhận các token bên trong dấu ngoặc và trả về mã mà chúng ta muốn sinh ra.

## Tóm tắt

Hú! Bây giờ bạn đã có một số tính năng Rust trong hộp công cụ của mình mà có thể bạn sẽ không dùng thường xuyên, nhưng bạn sẽ biết chúng khả dụng trong những tình huống rất cụ thể. Chúng ta đã giới thiệu một số chủ đề phức tạp để khi gặp chúng trong gợi ý lỗi hoặc trong mã của người khác, bạn sẽ nhận biết được các khái niệm và cú pháp này. Sử dụng chương này như một tài liệu tham khảo để hướng dẫn bạn đến các giải pháp.

Tiếp theo, chúng ta sẽ đưa tất cả những gì đã thảo luận trong suốt cuốn sách vào thực hành và làm thêm một dự án nữa!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[`syn`]: https://crates.io/crates/syn
[`quote`]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/1.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
