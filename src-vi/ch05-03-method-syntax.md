## Method Syntax

*Methods* tương tự như functions: chúng được khai báo với từ khóa `fn` và một
tên, chúng có thể có parameters và một return value, và chúng chứa một số code
được chạy khi method được gọi từ một nơi khác. Không giống như functions,
methods được định nghĩa trong ngữ cảnh của một struct (hoặc một enum hoặc một trait
object, chúng ta đề cập lần lượt trong [Chapter 6][enums]<!-- ignore --> và [Chapter
17][trait-objects]<!-- ignore -->), và parameter đầu tiên của chúng luôn là
`self`, đại diện cho instance của struct mà method đang được gọi trên đó.


### Defining Methods

Hãy thay đổi function `area` đang có một instance `Rectangle` làm parameter
và thay vào đó tạo một method `area` được định nghĩa trên struct `Rectangle`,
như được minh họa trong Listing 5-13.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

<span class="caption">Listing 5-13: Định nghĩa một method `area` trên
`Rectangle` struct</span>

Để định nghĩa function trong ngữ cảnh của `Rectangle`, chúng ta bắt đầu một
block `impl` (implementation) cho `Rectangle`. Mọi thứ bên trong block `impl`
này sẽ được gắn với type `Rectangle`. Sau đó chúng ta di chuyển function `area`
vào bên trong các dấu ngoặc nhọn của `impl` và thay đổi parameter đầu tiên (và
trong trường hợp này là parameter duy nhất) thành `self` trong signature và ở
mọi nơi trong body. Trong `main`, nơi chúng ta đã gọi function `area` và truyền
`rect1` làm một argument, chúng ta có thể thay vào đó sử dụng *method syntax*
để gọi method `area` trên instance `Rectangle` của chúng ta. Method syntax
được viết sau một instance: chúng ta thêm một dấu chấm theo sau là tên method,
dấu ngoặc tròn, và bất kỳ argument nào.

Trong signature của `area`, chúng ta sử dụng `&self` thay vì `rectangle: &Rectangle`.
`&self` thực chất là dạng viết tắt của `self: &Self`. Bên trong một block `impl`,
type `Self` là một alias cho type mà block `impl` đó áp dụng. Các method bắt buộc
phải có một parameter tên là `self` với type `Self` làm parameter đầu tiên, vì vậy
Rust cho phép bạn rút gọn điều này chỉ còn tên `self` ở vị trí parameter đầu tiên.
Lưu ý rằng chúng ta vẫn cần sử dụng dấu `&` phía trước dạng viết tắt `self` để
chỉ ra rằng method này mượn (borrow) instance `Self`, giống như cách chúng ta đã
làm với `rectangle: &Rectangle`. Các method có thể nhận quyền sở hữu của `self`,
mượn `self` một cách bất biến (immutable) như chúng ta đã làm ở đây, hoặc mượn
`self` một cách khả biến (mutable), giống như với bất kỳ parameter nào khác.

Chúng ta chọn `&self` ở đây vì cùng lý do đã dùng `&Rectangle` trong phiên bản
function: chúng ta không muốn lấy quyền sở hữu, và chỉ muốn đọc dữ liệu trong
struct chứ không ghi vào nó. Nếu chúng ta muốn thay đổi instance mà method được
gọi trên đó như một phần công việc của method, chúng ta sẽ dùng `&mut self` làm
parameter đầu tiên. Việc có một method lấy quyền sở hữu của instance bằng cách
chỉ sử dụng `self` làm parameter đầu tiên là khá hiếm; kỹ thuật này thường được
dùng khi method biến đổi `self` thành một thứ gì đó khác và bạn muốn ngăn không
cho người gọi tiếp tục sử dụng instance ban đầu sau khi quá trình biến đổi kết
thúc.

Lý do chính để sử dụng methods thay vì functions, ngoài việc cung cấp method
syntax và không phải lặp lại type của `self` trong signature của mỗi method,
là để tổ chức mã nguồn. Chúng ta đã đặt tất cả những gì có thể làm với một
instance của một type vào trong một block `impl` duy nhất, thay vì để những
người dùng mã của chúng ta trong tương lai phải đi tìm các khả năng của
`Rectangle` ở nhiều vị trí khác nhau trong thư viện mà chúng ta cung cấp.

Lưu ý rằng chúng ta có thể chọn đặt cho một method cùng tên với một trong các
field của struct. Ví dụ, chúng ta có thể định nghĩa một method trên `Rectangle`
mà cũng có tên là `width`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

Ở đây, chúng ta chọn để method `width` trả về `true` nếu giá trị trong field
`width` của instance lớn hơn `0`, và trả về `false` nếu giá trị là `0`: chúng ta
có thể sử dụng một field bên trong một method cùng tên cho bất kỳ mục đích nào.
Trong `main`, khi chúng ta theo sau `rect1.width` bằng dấu ngoặc tròn, Rust hiểu
rằng chúng ta đang nói đến method `width`. Khi chúng ta không dùng dấu ngoặc
tròn, Rust hiểu rằng chúng ta đang nói đến field `width`.

Thường thì, nhưng không phải lúc nào cũng vậy, khi chúng ta đặt cho một method
cùng tên với một field, chúng ta muốn nó chỉ trả về giá trị của field đó và
không làm gì khác. Những method như vậy được gọi là *getters*, và Rust không tự
động triển khai chúng cho các field của struct như một số ngôn ngữ khác.
Getters rất hữu ích vì bạn có thể để field ở chế độ private nhưng method thì ở
chế độ public, từ đó cho phép truy cập chỉ-đọc (read-only) tới field đó như một
phần của public API của type. Chúng ta sẽ thảo luận về public và private là gì
và cách chỉ định một field hoặc method là public hay private trong [Chapter
7][public]<!-- ignore -->.

> ### Toán tử `->` ở đâu?
>
> Trong C và C++, có hai toán tử khác nhau được dùng để gọi method: bạn dùng
> `.` nếu gọi method trực tiếp trên object và dùng `->` nếu gọi method trên
> một con trỏ tới object và cần dereference con trỏ trước. Nói cách khác, nếu
> `object` là một con trỏ, thì `object->something()` tương tự với
> `(*object).something()`.
>
> Rust không có toán tử tương đương với `->`; thay vào đó, Rust có một tính
> năng gọi là *automatic referencing and dereferencing*. Việc gọi method là
> một trong số ít những nơi trong Rust có hành vi này.
>
> Cách hoạt động như sau: khi bạn gọi một method với `object.something()`,
> Rust sẽ tự động thêm `&`, `&mut`, hoặc `*` để `object` khớp với signature của
> method. Nói cách khác, các cách gọi sau là giống nhau:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> Cách gọi đầu tiên trông gọn gàng hơn nhiều. Hành vi tự động reference này hoạt
> động được là vì các method có một receiver rõ ràng — chính là type của `self`.
> Dựa vào receiver và tên của method, Rust có thể xác định một cách dứt khoát
> liệu method đang đọc (`&self`), thay đổi (`&mut self`), hay tiêu thụ (`self`).
> Việc Rust làm cho cơ chế mượn (borrowing) trở nên ngầm định đối với receiver
> của method là một phần quan trọng giúp mô hình ownership trở nên dễ dùng hơn
> trong thực tế.

### Methods with More Parameters

Hãy thực hành sử dụng methods bằng cách triển khai một method thứ hai trên
struct `Rectangle`. Lần này, chúng ta muốn một instance của `Rectangle` nhận
một instance khác của `Rectangle` và trả về `true` nếu `Rectangle` thứ hai có
thể nằm hoàn toàn bên trong `self` (tức là `Rectangle` đầu tiên); nếu không,
nó sẽ trả về `false`. Nói cách khác, sau khi chúng ta đã định nghĩa method
`can_hold`, chúng ta muốn có thể viết chương trình như được minh họa trong
Listing 5-14.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

<span class="caption">Listing 5-14: Sử dụng method `can_hold` chưa được viết</span>

Kết quả mong đợi sẽ trông như sau vì cả hai kích thước của `rect2` đều nhỏ hơn
các kích thước của `rect1`, nhưng `rect3` thì rộng hơn `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Chúng ta biết rằng mình muốn định nghĩa một method, vì vậy nó sẽ nằm trong block
`impl Rectangle`. Tên method sẽ là `can_hold`, và nó sẽ nhận một immutable borrow
của một `Rectangle` khác làm parameter. Chúng ta có thể biết type của parameter
này bằng cách nhìn vào đoạn code gọi method:
`rect1.can_hold(&rect2)` truyền vào `&rect2`, tức là một immutable borrow tới
`rect2`, một instance của `Rectangle`. Điều này là hợp lý vì chúng ta chỉ cần
đọc `rect2` (thay vì ghi, điều đó sẽ yêu cầu một mutable borrow), và chúng ta
muốn `main` vẫn giữ quyền sở hữu của `rect2` để có thể dùng lại nó sau khi gọi
method `can_hold`. Giá trị trả về của `can_hold` sẽ là một Boolean, và phần
triển khai sẽ kiểm tra xem chiều rộng và chiều cao của `self` có lớn hơn chiều
rộng và chiều cao của `Rectangle` còn lại hay không. Hãy thêm method `can_hold`
mới vào block `impl` từ Listing 5-13, như được minh họa trong Listing 5-15.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

<span class="caption">Listing 5-15: Triển khai method `can_hold` trên
`Rectangle`, method này nhận một instance `Rectangle` khác làm parameter</span>

Khi chúng ta chạy đoạn code này với hàm `main` trong Listing 5-14, chúng ta sẽ
nhận được kết quả mong muốn. Methods có thể nhận nhiều parameter; chúng ta chỉ
cần thêm chúng vào signature sau parameter `self`, và các parameter đó hoạt
động giống hệt như parameter trong functions.

### Associated Functions

Tất cả các function được định nghĩa bên trong một block `impl` đều được gọi là
*các associated functions* vì chúng được gắn với type được nêu sau từ khóa
`impl`. Chúng ta có thể định nghĩa các associated function không có `self` làm
parameter đầu tiên (và do đó không phải là methods), bởi vì chúng không cần một
instance của type để hoạt động. Chúng ta đã từng sử dụng một function như vậy:
function `String::from` được định nghĩa trên type `String`.

Các associated function không phải là methods thường được dùng làm constructor
để trả về một instance mới của struct. Chúng thường được đặt tên là `new`, nhưng
`new` không phải là một tên đặc biệt và cũng không được tích hợp sẵn trong ngôn
ngữ. Ví dụ, chúng ta có thể chọn cung cấp một associated function tên là
`square`, function này sẽ nhận một parameter kích thước và dùng giá trị đó cho
cả chiều rộng lẫn chiều cao, từ đó giúp việc tạo một `Rectangle` hình vuông trở
nên dễ dàng hơn thay vì phải chỉ định cùng một giá trị hai lần:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Từ khóa `Self` trong kiểu trả về và trong body của function là alias cho type
xuất hiện sau từ khóa `impl`, trong trường hợp này là `Rectangle`.

Để gọi associated function này, chúng ta sử dụng cú pháp `::` với tên struct;
`let sq = Rectangle::square(3);` là một ví dụ. Function này được đặt trong
namespace của struct: cú pháp `::` được dùng cho cả associated functions và
các namespace được tạo bởi modules. Chúng ta sẽ thảo luận về modules trong
[Chapter 7][modules]<!-- ignore -->.

### Multiple `impl` Blocks

Mỗi struct được phép có nhiều `impl` block. Ví dụ, Listing
5-15 tương đương với đoạn code được trình bày trong Listing 5-16, trong đó
mỗi method nằm trong một `impl` block riêng.

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

<span class="caption">Listing 5-16: Viết lại Listing 5-15 bằng cách sử dụng nhiều `impl`
blocks</span>

Không có lý do bắt buộc nào để tách các method này ra thành nhiều `impl` block
trong trường hợp này, nhưng đây là cú pháp hợp lệ. Chúng ta sẽ thấy một trường
hợp mà nhiều `impl` block trở nên hữu ích ở Chapter 10, nơi chúng ta thảo luận
về generic types và traits.

## Summary

Struct cho phép bạn tạo ra các custom types có ý nghĩa đối với domain của bạn.
Bằng cách sử dụng struct, bạn có thể giữ các mảnh dữ liệu liên quan được gắn
kết với nhau và đặt tên cho từng mảnh để làm cho code của bạn rõ ràng hơn.
Trong các `impl` block, bạn có thể định nghĩa các function được liên kết với
type của mình, và methods là một dạng associated function cho phép bạn chỉ rõ
hành vi mà các instance của struct đó có.

Nhưng struct không phải là cách duy nhất để bạn tạo custom types: hãy chuyển
sang feature enum của Rust để bổ sung thêm một công cụ nữa vào toolbox của bạn.

[enums]: ch06-00-enums.html
[trait-objects]: ch17-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
