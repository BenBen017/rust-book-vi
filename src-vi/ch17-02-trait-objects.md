## Using Trait Objects That Allow for Values of Different Types

Trong Chương 8, chúng ta đã đề cập rằng một hạn chế của vector là chúng chỉ có thể
lưu trữ các phần tử của một kiểu duy nhất. Chúng ta đã tạo một cách обход
(workaround) trong Listing 8-9 bằng cách định nghĩa một enum `SpreadsheetCell`
với các biến thể để chứa số nguyên, số thực và văn bản. Điều này cho phép chúng ta
lưu trữ các kiểu dữ liệu khác nhau trong mỗi ô và vẫn có một vector đại diện cho
một hàng các ô. Đây là một giải pháp hoàn toàn phù hợp khi các phần tử có thể thay
thế cho nhau của chúng ta thuộc về một tập kiểu cố định mà ta biết tại thời điểm
biên dịch.

Tuy nhiên, đôi khi chúng ta muốn người dùng thư viện có thể mở rộng tập các kiểu
hợp lệ trong một tình huống cụ thể. Để minh họa cách đạt được điều này, chúng ta sẽ
tạo một ví dụ về công cụ giao diện người dùng đồ họa (GUI) lặp qua một danh sách
các mục và gọi một phương thức `draw` trên mỗi mục để vẽ nó lên màn hình — một kỹ
thuật phổ biến trong các công cụ GUI. Chúng ta sẽ tạo một library crate có tên là
`gui`, chứa cấu trúc của một thư viện GUI. Crate này có thể bao gồm một số kiểu để
mọi người sử dụng, chẳng hạn như `Button` hoặc `TextField`. Ngoài ra, người dùng
`gui` cũng sẽ muốn tạo các kiểu riêng của họ có thể được vẽ: ví dụ, một lập trình
viên có thể thêm `Image` và một người khác có thể thêm `SelectBox`.

Chúng ta sẽ không triển khai một thư viện GUI đầy đủ cho ví dụ này, mà chỉ cho thấy
cách các mảnh ghép sẽ kết hợp với nhau như thế nào. Tại thời điểm viết thư viện,
chúng ta không thể biết trước và định nghĩa tất cả các kiểu mà những lập trình
viên khác có thể muốn tạo ra. Nhưng chúng ta biết rằng `gui` cần theo dõi nhiều giá
trị thuộc các kiểu khác nhau, và nó cần gọi một phương thức `draw` trên mỗi giá trị
có kiểu khác nhau đó. Nó không cần biết chính xác điều gì sẽ xảy ra khi gọi phương
thức `draw`, chỉ cần biết rằng giá trị đó có sẵn phương thức này để chúng ta gọi.

Để làm điều này trong một ngôn ngữ có inheritance, chúng ta có thể định nghĩa một
class tên là `Component` có một phương thức tên là `draw`. Các class khác, chẳng
hạn như `Button`, `Image` và `SelectBox`, sẽ kế thừa từ `Component` và do đó kế thừa
phương thức `draw`. Mỗi class có thể ghi đè phương thức `draw` để định nghĩa hành vi
riêng của mình, nhưng framework có thể đối xử với tất cả các kiểu đó như thể chúng
là các instance của `Component` và gọi `draw` trên chúng. Nhưng vì Rust không có
inheritance, chúng ta cần một cách khác để cấu trúc thư viện `gui` sao cho cho phép
người dùng mở rộng nó bằng các kiểu mới.

### Defining a Trait for Common Behavior

Để triển khai hành vi mà chúng ta muốn `gui` có, chúng ta sẽ định nghĩa một trait
tên là `Draw` với một phương thức duy nhất là `draw`. Sau đó, chúng ta có thể định
nghĩa một vector nhận vào một *trait object*. Một trait object trỏ tới cả một
instance của một kiểu triển khai trait được chỉ định và một bảng được dùng để tra
cứu các phương thức của trait đó tại runtime. Chúng ta tạo một trait object bằng
cách chỉ định một dạng con trỏ nào đó, chẳng hạn như một tham chiếu `&` hoặc một
smart pointer `Box<T>`, sau đó là từ khóa `dyn`, rồi chỉ định trait tương ứng.
(Chúng ta sẽ nói về lý do tại sao trait objects phải sử dụng con trỏ trong Chương
19, ở mục [“Dynamically Sized Types and the `Sized` Trait.”][dynamically-sized]
<!-- ignore -->) Chúng ta có thể sử dụng trait objects thay cho generic hoặc kiểu
cụ thể. Ở bất kỳ đâu chúng ta sử dụng trait object, hệ thống kiểu của Rust sẽ đảm
bảo tại thời điểm biên dịch rằng mọi giá trị được dùng trong ngữ cảnh đó đều triển
khai trait của trait object. Do đó, chúng ta không cần phải biết trước tất cả các
kiểu khả dĩ tại thời điểm biên dịch.

Chúng ta đã đề cập rằng trong Rust, chúng ta tránh gọi struct và enum là “object”
để phân biệt chúng với object trong các ngôn ngữ khác. Trong một struct hoặc enum,
dữ liệu trong các field và hành vi trong các khối `impl` được tách biệt, trong khi
ở các ngôn ngữ khác, dữ liệu và hành vi được kết hợp thành một khái niệm duy nhất
thường được gọi là object. Tuy nhiên, trait objects *thực sự* giống với object
trong các ngôn ngữ khác ở chỗ chúng kết hợp cả dữ liệu và hành vi. Nhưng trait
objects khác với object truyền thống ở điểm là chúng ta không thể thêm dữ liệu vào
một trait object. Trait objects không hữu dụng một cách tổng quát như object trong
các ngôn ngữ khác: mục đích cụ thể của chúng là cho phép trừu tượng hóa trên các
hành vi chung.

Listing 17-3 cho thấy cách định nghĩa một trait tên là `Draw` với một phương thức
tên là `draw`:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-03/src/lib.rs}}
```

<span class="caption">Listing 17-3: Definition of the `Draw` trait</span>

Cú pháp này hẳn sẽ trông quen thuộc từ những thảo luận của chúng ta về cách định
nghĩa trait trong Chương 10. Tiếp theo là một số cú pháp mới: Listing 17-4 định
nghĩa một struct tên là `Screen`, struct này chứa một vector có tên là
`components`. Vector này có kiểu `Box<dyn Draw>`, tức là một trait object; nó
đóng vai trò như một đại diện (stand-in) cho bất kỳ kiểu nào được đặt trong một
`Box` mà triển khai trait `Draw`.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-04/src/lib.rs:here}}
```

<span class="caption">Listing 17-4: Definition of the `Screen` struct with a
`components` field holding a vector of trait objects that implement the `Draw`
trait</span>

Trên struct `Screen`, chúng ta sẽ định nghĩa một phương thức có tên là `run`,
phương thức này sẽ gọi phương thức `draw` trên từng phần tử trong `components`
của nó, như được minh họa trong Listing 17-5:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-05/src/lib.rs:here}}
```

<span class="caption">Listing 17-5: A `run` method on `Screen` that calls the
`draw` method on each component</span>

Cách này hoạt động khác với việc định nghĩa một struct sử dụng tham số kiểu
generic kèm theo trait bound. Một tham số kiểu generic chỉ có thể được thay thế
bằng một kiểu cụ thể tại một thời điểm, trong khi trait objects cho phép
nhiều kiểu cụ thể khác nhau được dùng để thay thế cho trait object tại runtime.
Ví dụ, chúng ta có thể đã định nghĩa struct `Screen` bằng cách sử dụng một kiểu
generic và một trait bound như trong Listing 17-6:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-06/src/lib.rs:here}}
```

<span class="caption">Listing 17-6: An alternate implementation of the `Screen`
struct and its `run` method using generics and trait bounds</span>

Cách này giới hạn chúng ta ở một instance của `Screen` mà trong đó danh sách
các component đều có cùng một kiểu, ví dụ tất cả đều là `Button` hoặc tất cả
đều là `TextField`. Nếu bạn chỉ bao giờ làm việc với các collection đồng nhất
(homogeneous), thì việc sử dụng generics và trait bounds là lựa chọn tốt hơn,
bởi vì các định nghĩa này sẽ được monomorphize tại thời điểm biên dịch để sử
dụng các kiểu cụ thể.

Ngược lại, với phương thức sử dụng trait objects, một instance của `Screen`
có thể chứa một `Vec<T>` bao gồm cả `Box<Button>` lẫn `Box<TextField>`. Hãy cùng
xem cách cơ chế này hoạt động như thế nào, sau đó chúng ta sẽ bàn về các ảnh
hưởng tới hiệu năng tại runtime.

### Implementing the Trait

Bây giờ chúng ta sẽ thêm một số kiểu triển khai trait `Draw`. Chúng ta sẽ cung
cấp kiểu `Button`. Một lần nữa, việc triển khai đầy đủ một thư viện GUI nằm ngoài
phạm vi của cuốn sách này, vì vậy phương thức `draw` sẽ không có phần cài đặt
hữu ích nào trong thân của nó. Để hình dung phương thức này có thể được cài đặt
như thế nào, một struct `Button` có thể có các field như `width`, `height` và
`label`, như được minh họa trong Listing 17-7:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-07/src/lib.rs:here}}
```

<span class="caption">Listing 17-7: A `Button` struct that implements the
`Draw` trait</span>

Các field `width`, `height` và `label` của `Button` sẽ khác với các field của
những component khác; ví dụ, một kiểu `TextField` có thể có các field giống vậy
và thêm một field `placeholder`. Mỗi kiểu mà chúng ta muốn vẽ lên màn hình đều
sẽ triển khai trait `Draw`, nhưng sẽ sử dụng các đoạn mã khác nhau trong phương
thức `draw` để định nghĩa cách vẽ cho kiểu cụ thể đó, như cách `Button` đã làm
ở đây (không bao gồm mã GUI thực tế, như đã đề cập). Ví dụ, kiểu `Button` có thể
có thêm một khối `impl` bổ sung chứa các phương thức liên quan đến việc điều gì
xảy ra khi người dùng nhấn vào nút. Những loại phương thức này sẽ không áp dụng
cho các kiểu như `TextField`.

Nếu một người sử dụng thư viện của chúng ta quyết định triển khai một struct
`SelectBox` có các field `width`, `height` và `options`, thì họ cũng sẽ triển
khai trait `Draw` cho kiểu `SelectBox`, như được minh họa trong Listing 17-8:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-08/src/main.rs:here}}
```

<span class="caption">Listing 17-8: Another crate using `gui` and implementing
the `Draw` trait on a `SelectBox` struct</span>

Người dùng thư viện của chúng ta giờ đây có thể viết hàm `main` của họ để tạo
một instance của `Screen`. Với instance `Screen` này, họ có thể thêm một
`SelectBox` và một `Button` bằng cách đặt mỗi component vào một `Box<T>` để
trở thành một trait object. Sau đó, họ có thể gọi phương thức `run` trên
instance `Screen`, phương thức này sẽ gọi `draw` trên từng component.
Listing 17-9 minh họa cách cài đặt này:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-09/src/main.rs:here}}
```

<span class="caption">Listing 17-9: Using trait objects to store values of
different types that implement the same trait</span>

Khi chúng ta viết thư viện, chúng ta không biết rằng sẽ có người thêm kiểu
`SelectBox`, nhưng phần cài đặt `Screen` của chúng ta vẫn có thể hoạt động với
kiểu mới này và vẽ nó, bởi vì `SelectBox` triển khai trait `Draw`, nghĩa là nó
triển khai phương thức `draw`.

Khái niệm này — chỉ quan tâm đến các “thông điệp” (message) mà một giá trị có
thể phản hồi, thay vì kiểu cụ thể của giá trị đó — tương tự với khái niệm *duck
typing* trong các ngôn ngữ kiểu động: nếu nó đi như vịt và kêu như vịt, thì nó
chắc hẳn là vịt! Trong phần cài đặt `run` của `Screen` ở Listing 17-5, `run`
không cần biết kiểu cụ thể của từng component là gì. Nó không kiểm tra xem một
component có phải là instance của `Button` hay `SelectBox` hay không, mà chỉ
đơn giản gọi phương thức `draw` trên component đó. Bằng cách chỉ định
`Box<dyn Draw>` làm kiểu của các giá trị trong vector `components`, chúng ta đã
định nghĩa rằng `Screen` cần những giá trị mà chúng ta có thể gọi được phương
thức `draw` trên chúng.

Ưu điểm của việc sử dụng trait objects cùng với hệ thống kiểu của Rust để viết
mã tương tự như mã sử dụng duck typing là chúng ta không bao giờ phải kiểm tra
tại runtime xem một giá trị có triển khai một phương thức cụ thể hay không, cũng
không phải lo lắng về việc gặp lỗi khi gọi một phương thức mà giá trị đó không
triển khai. Rust sẽ không biên dịch mã của chúng ta nếu các giá trị không triển
khai các trait mà trait object yêu cầu.

Ví dụ, Listing 17-10 cho thấy điều gì sẽ xảy ra nếu chúng ta cố gắng tạo một
`Screen` với một `String` làm component:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-10/src/main.rs}}
```

<span class="caption">Listing 17-10: Attempting to use a type that doesn’t
implement the trait object’s trait</span>

Chúng ta sẽ nhận được lỗi này bởi vì `String` không triển khai trait `Draw`:

```console
{{#include ../listings/ch17-oop/listing-17-10/output.txt}}
```

Lỗi này cho chúng ta biết rằng hoặc là chúng ta đang truyền một giá trị vào
`Screen` mà không đúng dự định và nên truyền một kiểu khác, hoặc chúng ta
nên triển khai trait `Draw` trên `String` để `Screen` có thể gọi `draw`
trên nó.

### Trait Objects Perform Dynamic Dispatch

Hãy nhớ lại trong mục [“Performance of Code Using
Generics”][performance-of-code-using-generics]<!-- ignore --> ở Chương 10
khi chúng ta thảo luận về quá trình monomorphization do compiler thực hiện
khi sử dụng trait bounds trên generic: compiler sẽ sinh ra các triển khai
không-generic của các hàm và phương thức cho từng kiểu cụ thể mà chúng ta
sử dụng thay cho tham số kiểu generic. Mã kết quả từ monomorphization
sử dụng *static dispatch*, tức là compiler biết chính xác phương thức
bạn đang gọi tại thời điểm biên dịch. Ngược lại, *dynamic dispatch*
xảy ra khi compiler không thể biết trước phương thức nào sẽ được gọi
tại thời điểm biên dịch. Trong trường hợp dynamic dispatch, compiler
sẽ sinh ra mã sao cho tại runtime sẽ xác định phương thức nào cần gọi.

Khi chúng ta sử dụng trait objects, Rust buộc phải dùng dynamic dispatch.
Compiler không biết trước tất cả các kiểu có thể được sử dụng với mã
sử dụng trait objects, vì vậy nó không biết nên gọi phương thức nào được
triển khai trên kiểu nào. Thay vào đó, tại runtime, Rust sử dụng các
con trỏ bên trong trait object để biết phương thức nào cần gọi. Việc tra
cứu này gây ra chi phí tại runtime mà static dispatch không có. Dynamic
dispatch cũng ngăn compiler chọn việc inline mã của phương thức, từ đó
cản trở một số tối ưu hóa. Tuy nhiên, chúng ta có được sự linh hoạt
bổ sung trong mã mà chúng ta viết ở Listing 17-5 và hỗ trợ trong
Listing 17-9, nên đây là một sự đánh đổi cần cân nhắc.

[performance-of-code-using-generics]:
ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait
