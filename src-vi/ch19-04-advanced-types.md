## Kiểu nâng cao

Hệ thống kiểu của Rust có một số tính năng mà chúng ta đã đề cập nhưng chưa bàn chi tiết. Chúng ta sẽ bắt đầu bằng việc thảo luận về newtypes nói chung và xem tại sao newtypes hữu ích như các kiểu. Sau đó, chúng ta sẽ chuyển sang type aliases, một tính năng tương tự newtypes nhưng có ngữ nghĩa hơi khác. Chúng ta cũng sẽ thảo luận về kiểu `!` và các kiểu có kích thước động (dynamically sized types).

### Sử dụng Newtype Pattern để đảm bảo an toàn kiểu và trừu tượng hóa

> Lưu ý: Phần này giả định bạn đã đọc phần trước [“Using the Newtype Pattern to Implement External Traits on External Types.”][using-the-newtype-pattern]<!-- ignore -->

Newtype pattern cũng hữu ích cho các nhiệm vụ vượt ra ngoài những gì chúng ta đã thảo luận, bao gồm việc đảm bảo tại thời gian biên dịch rằng các giá trị không bị nhầm lẫn và chỉ định đơn vị của một giá trị. Bạn đã thấy một ví dụ về việc sử dụng newtypes để chỉ định đơn vị trong Listing 19-15: nhớ rằng các struct `Millimeters` và `Meters` bọc các giá trị `u32` trong một newtype. Nếu chúng ta viết một hàm với tham số kiểu `Millimeters`, chúng ta sẽ không thể biên dịch một chương trình mà vô tình cố gọi hàm đó với giá trị kiểu `Meters` hoặc một `u32` thông thường.

Chúng ta cũng có thể sử dụng newtype pattern để trừu tượng hóa một số chi tiết triển khai của một kiểu: kiểu mới có thể cung cấp một API công khai khác với API của kiểu bên trong riêng tư.

Newtypes cũng có thể che giấu triển khai bên trong. Ví dụ, chúng ta có thể cung cấp kiểu `People` để bọc một `HashMap<i32, String>` lưu trữ ID của một người liên kết với tên của họ. Mã sử dụng `People` chỉ tương tác với API công khai mà chúng ta cung cấp, chẳng hạn như một phương thức thêm tên vào tập hợp `People`; mã đó không cần biết rằng chúng ta gán một ID `i32` cho tên bên trong. Newtype pattern là một cách nhẹ để đạt được đóng gói (encapsulation) nhằm che giấu chi tiết triển khai, như chúng ta đã thảo luận trong phần [“Encapsulation that Hides Implementation Details”][encapsulation-that-hides-implementation-details]<!-- ignore --> của Chương 17.

### Tạo bí danh kiểu với Type Aliases

Rust cung cấp khả năng khai báo một *type alias* để đặt một tên khác cho một kiểu hiện có. Chúng ta sử dụng từ khóa `type` cho việc này. Ví dụ, chúng ta có thể tạo bí danh `Kilometers` cho `i32` như sau:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Bây giờ, bí danh `Kilometers` là một *tên đồng nghĩa* với `i32`; khác với các kiểu `Millimeters` và `Meters` mà chúng ta đã tạo trong Listing 19-15, `Kilometers` không phải là một kiểu mới, riêng biệt. Các giá trị có kiểu `Kilometers` sẽ được xử lý giống như các giá trị kiểu `i32`:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Vì `Kilometers` và `i32` là cùng một kiểu, chúng ta có thể cộng các giá trị của cả hai kiểu và có thể truyền các giá trị `Kilometers` vào các hàm nhận tham số kiểu `i32`. Tuy nhiên, sử dụng phương pháp này, chúng ta không có lợi ích kiểm tra kiểu (type checking) như khi sử dụng newtype pattern đã thảo luận trước đó. Nói cách khác, nếu chúng ta nhầm lẫn giữa các giá trị `Kilometers` và `i32` ở đâu đó, trình biên dịch sẽ không báo lỗi.

Trường hợp sử dụng chính của các bí danh kiểu là giảm sự lặp lại. Ví dụ, chúng ta có thể có một kiểu dài như sau:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Việc viết kiểu dài này trong các chữ ký hàm và chú thích kiểu khắp nơi trong mã có thể gây mệt mỏi và dễ sai sót. Hãy tưởng tượng một dự án đầy các đoạn mã như trong Listing 19-24.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-24/src/main.rs:here}}
```

<span class="caption">Listing 19-24: Sử dụng một kiểu dài ở nhiều nơi</span>

Một bí danh kiểu giúp mã này dễ quản lý hơn bằng cách giảm sự lặp lại. Trong Listing 19-25, chúng ta đã tạo một bí danh tên là `Thunk` cho kiểu dài dòng và có thể thay thế tất cả các lần sử dụng kiểu này bằng bí danh ngắn hơn `Thunk`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-25/src/main.rs:here}}
```

<span class="caption">Listing 19-25: Giới thiệu bí danh kiểu `Thunk` để giảm sự lặp lại</span>

Mã này dễ đọc và viết hơn rất nhiều! Việc chọn một tên có ý nghĩa cho một bí danh kiểu cũng giúp truyền đạt ý định của bạn (*thunk* là một từ chỉ đoạn mã sẽ được đánh giá sau, nên nó là tên phù hợp cho một closure được lưu trữ).

Các bí danh kiểu cũng thường được sử dụng với kiểu `Result<T, E>` để giảm sự lặp lại. Hãy xem xét module `std::io` trong thư viện chuẩn. Các thao tác I/O thường trả về `Result<T, E>` để xử lý các tình huống khi thao tác không thành công. Thư viện này có struct `std::io::Error` đại diện cho tất cả các lỗi I/O có thể xảy ra. Nhiều hàm trong `std::io` sẽ trả về `Result<T, E>` với `E` là `std::io::Error`, ví dụ như các hàm trong trait `Write`:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

Kiểu `Result<..., Error>` được lặp lại rất nhiều. Do đó, `std::io` có khai báo bí danh kiểu như sau:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Vì khai báo này nằm trong module `std::io`, chúng ta có thể sử dụng bí danh kiểu đầy đủ `std::io::Result<T>`; nghĩa là một `Result<T, E>` với `E` được điền là `std::io::Error`. Chữ ký các hàm trong trait `Write` sẽ trông như sau:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Bí danh kiểu hữu ích theo hai cách: nó làm cho mã dễ viết hơn *và* cung cấp một giao diện nhất quán trên toàn bộ `std::io`. Vì nó chỉ là một bí danh, nó vẫn là `Result<T, E>`, nghĩa là chúng ta có thể sử dụng bất kỳ phương thức nào hoạt động trên `Result<T, E>` với nó, cũng như các cú pháp đặc biệt như toán tử `?`.

### Kiểu Never không bao giờ trả về

Rust có một kiểu đặc biệt tên là `!`, trong lý thuyết kiểu được gọi là *kiểu rỗng* vì nó không có giá trị nào. Chúng ta thích gọi nó là *never type* vì nó đứng ở vị trí kiểu trả về khi một hàm sẽ không bao giờ trả về. Đây là một ví dụ:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Mã này được đọc là “hàm `bar` không bao giờ trả về.” Các hàm trả về kiểu never được gọi là *diverging functions*. Chúng ta không thể tạo giá trị của kiểu `!`, nên `bar` sẽ không bao giờ có thể trả về.

Nhưng một kiểu mà bạn không bao giờ có thể tạo giá trị của nó có tác dụng gì? Hãy nhớ lại mã từ Listing 2-5, một phần của trò chơi đoán số; chúng tôi đã tái hiện một phần ở đây trong Listing 19-26.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

<span class="caption">Listing 19-26: Một `match` với nhánh kết thúc bằng `continue`</span>

Lúc đó, chúng ta đã bỏ qua một số chi tiết trong mã này. Trong Chương 6, trong phần [“Toán tử điều khiển luồng `match`”][the-match-control-flow-operator]<!-- ignore -->, chúng ta đã thảo luận rằng tất cả các nhánh của `match` phải trả về cùng một kiểu. Vì vậy, ví dụ, mã sau đây sẽ không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Kiểu của `guess` trong mã này sẽ phải vừa là một số nguyên *vừa* là một chuỗi, và Rust yêu cầu `guess` chỉ có một kiểu duy nhất. Vậy `continue` trả về gì? Làm sao chúng ta được phép trả về một `u32` từ một nhánh và có một nhánh khác kết thúc bằng `continue` trong Listing 19-26?

Như bạn có thể đoán, `continue` có giá trị kiểu `!`. Nghĩa là khi Rust tính toán kiểu của `guess`, nó nhìn vào cả hai nhánh của `match`, nhánh trước có giá trị là `u32` và nhánh sau có giá trị là `!`. Vì `!` không bao giờ có giá trị, Rust quyết định rằng kiểu của `guess` là `u32`.

Cách chính thức để mô tả hành vi này là các biểu thức có kiểu `!` có thể được ép sang bất kỳ kiểu nào khác. Chúng ta được phép kết thúc nhánh `match` này bằng `continue` vì `continue` không trả về giá trị; thay vào đó, nó đưa luồng điều khiển quay trở lại đầu vòng lặp, nên trong trường hợp `Err`, chúng ta không bao giờ gán giá trị cho `guess`.

Kiểu never cũng hữu ích với macro `panic!`. Hãy nhớ lại hàm `unwrap` mà chúng ta gọi trên các giá trị `Option<T>` để tạo ra một giá trị hoặc gây panic với định nghĩa sau:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Trong mã này, điều tương tự xảy ra như trong `match` ở Listing 19-26: Rust thấy rằng `val` có kiểu `T` và `panic!` có kiểu `!`, nên kết quả của toàn bộ biểu thức `match` là `T`. Mã này hoạt động vì `panic!` không tạo ra giá trị; nó kết thúc chương trình. Trong trường hợp `None`, chúng ta sẽ không trả về giá trị từ `unwrap`, nên mã này là hợp lệ.

Một biểu thức cuối cùng có kiểu `!` là `loop`:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Ở đây, vòng lặp không bao giờ kết thúc, nên `!` là giá trị của biểu thức. Tuy nhiên, điều này sẽ không đúng nếu chúng ta bao gồm một `break`, vì vòng lặp sẽ kết thúc khi gặp `break`.

### Kiểu có Kích thước Động và Trait `Sized`

Rust cần biết một số chi tiết về các kiểu của nó, chẳng hạn như bao nhiêu bộ nhớ cần cấp phát cho một giá trị của kiểu cụ thể. Điều này để lại một khía cạnh hơi khó hiểu lúc đầu trong hệ thống kiểu của Rust: khái niệm *các kiểu có kích thước động*. Đôi khi được gọi là *DSTs* hoặc *unsized types*, các kiểu này cho phép chúng ta viết mã với các giá trị mà kích thước chỉ biết được khi chạy chương trình.

Hãy đi sâu vào chi tiết về một kiểu có kích thước động gọi là `str`, mà chúng ta đã sử dụng trong suốt cuốn sách. Đúng vậy, không phải `&str`, mà chính `str` là một DST. Chúng ta không thể biết độ dài của chuỗi cho đến khi chạy chương trình, nghĩa là chúng ta không thể tạo biến có kiểu `str`, cũng không thể nhận đối số kiểu `str`. Xem xét mã sau, mã này không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust cần biết lượng bộ nhớ cần cấp phát cho bất kỳ giá trị nào của một kiểu cụ thể, và tất cả các giá trị của một kiểu phải sử dụng cùng một lượng bộ nhớ. Nếu Rust cho phép chúng ta viết đoạn mã này, hai giá trị `str` này sẽ cần chiếm cùng một lượng không gian. Nhưng chúng có độ dài khác nhau: `s1` cần 12 byte bộ nhớ và `s2` cần 15 byte. Đây là lý do tại sao không thể tạo một biến chứa kiểu có kích thước động.

Vậy chúng ta làm gì? Trong trường hợp này, bạn đã biết câu trả lời: chúng ta đặt kiểu của `s1` và `s2` là `&str` thay vì `str`. Nhớ lại từ phần [“String Slices”][string-slices]<!-- ignore --> trong Chương 4 rằng cấu trúc dữ liệu slice chỉ lưu vị trí bắt đầu và độ dài của slice. Vì vậy, mặc dù một `&T` là một giá trị duy nhất lưu địa chỉ bộ nhớ nơi `T` nằm, một `&str` lại là *hai* giá trị: địa chỉ của `str` và độ dài của nó. Như vậy, chúng ta có thể biết kích thước của một giá trị `&str` tại thời gian biên dịch: nó gấp đôi kích thước của một `usize`. Tức là, chúng ta luôn biết kích thước của `&str`, bất kể chuỗi mà nó tham chiếu dài bao nhiêu. Nói chung, đây là cách các kiểu có kích thước động được sử dụng trong Rust: chúng có một phần metadata bổ sung lưu kích thước của thông tin động. Quy tắc vàng của các kiểu có kích thước động là chúng ta phải luôn đặt các giá trị của kiểu có kích thước động sau một con trỏ nào đó.

Chúng ta có thể kết hợp `str` với tất cả các loại con trỏ: ví dụ, `Box<str>` hoặc `Rc<str>`. Thực tế, bạn đã thấy điều này trước đây nhưng với một kiểu có kích thước động khác: traits. Mỗi trait là một kiểu có kích thước động mà chúng ta có thể tham chiếu bằng cách sử dụng tên của trait. Trong Chương 17, phần [“Using Trait Objects That Allow for Values of Different Types”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore -->, chúng ta đã đề cập rằng để sử dụng trait như trait objects, chúng ta phải đặt chúng sau một con trỏ, chẳng hạn như `&dyn Trait` hoặc `Box<dyn Trait>` (`Rc<dyn Trait>` cũng được).

Để làm việc với DSTs, Rust cung cấp trait `Sized` để xác định xem kích thước của một kiểu có được biết tại thời gian biên dịch hay không. Trait này được triển khai tự động cho mọi thứ có kích thước được biết tại thời gian biên dịch. Ngoài ra, Rust ngầm thêm một ràng buộc `Sized` vào mọi hàm generic. Tức là, một định nghĩa hàm generic như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

thực ra được coi như chúng ta đã viết như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Mặc định, các hàm generic chỉ hoạt động trên các kiểu có kích thước được biết tại thời gian biên dịch. Tuy nhiên, bạn có thể sử dụng cú pháp đặc biệt sau để nới lỏng hạn chế này:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

Một ràng buộc trait với `?Sized` có nghĩa là “`T` có thể có hoặc không có kích thước xác định” và ký hiệu này ghi đè mặc định rằng các kiểu generic phải có kích thước được biết tại thời gian biên dịch. Cú pháp `?Trait` với ý nghĩa này chỉ áp dụng cho `Sized`, không áp dụng cho bất kỳ trait nào khác.

Cũng lưu ý rằng chúng ta đã chuyển kiểu của tham số `t` từ `T` sang `&T`. Vì kiểu này có thể không phải là `Sized`, chúng ta cần sử dụng nó thông qua một loại con trỏ nào đó. Trong trường hợp này, chúng ta đã chọn sử dụng một reference.

Tiếp theo, chúng ta sẽ nói về các hàm và closures!

[encapsulation-that-hides-implementation-details]:
ch17-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-operator]:
ch06-02-match.html#the-match-control-flow-operator
[using-trait-objects-that-allow-for-values-of-different-types]:
ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[using-the-newtype-pattern]: ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
