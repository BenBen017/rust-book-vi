## Xử lý Smart Pointer như Tham Chiếu Thường với Trait `Deref`

Triển khai trait `Deref` cho phép bạn tùy chỉnh hành vi của *toán tử dereference* `*` (không nhầm lẫn với toán tử nhân hoặc glob). Bằng cách triển khai `Deref` sao cho smart pointer có thể được xử lý như một tham chiếu thông thường, bạn có thể viết code hoạt động trên các tham chiếu và cũng dùng code đó với smart pointer.

Trước tiên, chúng ta sẽ xem cách toán tử dereference hoạt động với tham chiếu thông thường. Sau đó, chúng ta sẽ thử định nghĩa một kiểu tùy chỉnh hoạt động giống như `Box<T>`, và xem lý do tại sao toán tử dereference không hoạt động như một tham chiếu với kiểu mới định nghĩa của chúng ta. Chúng ta sẽ khám phá cách triển khai trait `Deref` giúp smart pointer hoạt động tương tự tham chiếu. Sau đó, chúng ta sẽ xem tính năng *deref coercion* của Rust và cách nó cho phép làm việc với cả tham chiếu lẫn smart pointer.

> Lưu ý: có một điểm khác biệt lớn giữa kiểu `MyBox<T>` mà chúng ta sắp xây dựng và `Box<T>` thật: phiên bản của chúng ta sẽ không lưu dữ liệu trên heap. Chúng ta tập trung vào ví dụ này về `Deref`, vì vậy nơi dữ liệu thực sự được lưu không quan trọng bằng hành vi giống con trỏ.

<!-- Old link, do not remove -->
<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>

### Theo Dấu Con Trỏ đến Giá Trị

Một tham chiếu thông thường là một loại con trỏ, và một cách để nghĩ về con trỏ là như một mũi tên đến một giá trị được lưu ở nơi khác. Trong Listing 15-6, chúng ta tạo một tham chiếu đến một giá trị `i32` và sau đó sử dụng toán tử dereference để theo dấu tham chiếu đến giá trị:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

<span class="caption">Listing 15-6: Sử dụng toán tử dereference để theo dấu
một tham chiếu đến giá trị `i32`</span>

Biến `x` giữ giá trị `i32` là `5`. Chúng ta gán `y` bằng một tham chiếu đến `x`. Chúng ta có thể xác nhận rằng `x` bằng `5`. Tuy nhiên, nếu muốn xác nhận giá trị trong `y`, chúng ta phải sử dụng `*y` để theo dấu tham chiếu đến giá trị mà nó đang trỏ tới (tức là *dereference*) để trình biên dịch có thể so sánh giá trị thực tế. Khi chúng ta dereference `y`, chúng ta sẽ truy cập được giá trị số nguyên mà `y` đang trỏ tới và có thể so sánh với `5`.

Nếu chúng ta cố viết `assert_eq!(5, y);` thì sẽ nhận được lỗi biên dịch sau:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

So sánh một số và một tham chiếu đến số không được phép vì chúng là các loại khác nhau. Chúng ta phải sử dụng toán tử dereference để theo dấu tham chiếu đến giá trị mà nó đang trỏ tới.

### Sử dụng `Box<T>` giống như tham chiếu

Chúng ta có thể viết lại mã trong Listing 15-6 để sử dụng `Box<T>` thay vì một tham chiếu; toán tử dereference được sử dụng trên `Box<T>` trong Listing 15-7 hoạt động cùng cách với toán tử dereference được sử dụng trên tham chiếu trong Listing 15-6:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

<span class="caption">Listing 15-7: Sử dụng toán tử dereference trên một
`Box<i32>`</span>

Sự khác biệt chính giữa Listing 15-7 và Listing 15-6 là ở đây chúng ta gán
`y` thành một thể hiện của `Box<T>` trỏ tới một bản sao của giá trị `x`
thay vì một tham chiếu trỏ tới giá trị của `x`. Trong câu lệnh assert cuối,
chúng ta có thể sử dụng toán tử dereference để theo dấu con trỏ của `Box<T>`
cùng cách mà chúng ta đã làm khi `y` là một tham chiếu. Tiếp theo, chúng ta
sẽ khám phá điều gì đặc biệt ở `Box<T>` cho phép chúng ta sử dụng toán tử
dereference bằng cách định nghĩa kiểu riêng của mình.

### Định nghĩa Smart Pointer của riêng chúng ta

Hãy xây dựng một smart pointer tương tự như kiểu `Box<T>` do thư viện
chuẩn cung cấp để trải nghiệm cách mà smart pointer hoạt động khác với
các tham chiếu theo mặc định. Sau đó, chúng ta sẽ xem cách thêm khả năng
sử dụng toán tử dereference.

Kiểu `Box<T>` cuối cùng được định nghĩa như một tuple struct với một phần tử,
vì vậy Listing 15-8 định nghĩa một kiểu `MyBox<T>` theo cùng cách. Chúng ta
cũng sẽ định nghĩa một hàm `new` để tương ứng với hàm `new` được định nghĩa
trên `Box<T>`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

<span class="caption">Listing 15-8: Định nghĩa kiểu `MyBox<T>`</span>

Chúng ta định nghĩa một struct tên là `MyBox` và khai báo một tham số generic `T`,
vì chúng ta muốn kiểu của mình có thể chứa giá trị của bất kỳ loại nào. Kiểu
`MyBox` là một tuple struct với một phần tử có kiểu `T`. Hàm `MyBox::new` nhận
một tham số kiểu `T` và trả về một thể hiện `MyBox` chứa giá trị được truyền vào.

Hãy thử thêm hàm `main` trong Listing 15-7 vào Listing 15-8 và thay đổi
nó để sử dụng kiểu `MyBox<T>` mà chúng ta đã định nghĩa thay vì `Box<T>`.
Mã trong Listing 15-9 sẽ không biên dịch vì Rust không biết cách dereference
`MyBox`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

<span class="caption">Listing 15-9: Thử sử dụng `MyBox<T>` theo cùng cách
chúng ta đã dùng references và `Box<T>`</span>

Dưới đây là lỗi biên dịch mà chúng ta nhận được:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Kiểu `MyBox<T>` của chúng ta không thể dereference được vì chúng ta chưa triển khai
khả năng đó cho kiểu này. Để cho phép dereferencing với toán tử `*`, chúng ta
cần triển khai trait `Deref`.

### Xử lý một kiểu như một reference bằng cách triển khai trait `Deref`

Như đã thảo luận trong phần [“Implementing a Trait on a Type”][impl-trait]<!-- ignore -->
của Chương 10, để triển khai một trait, chúng ta cần cung cấp các
cài đặt cho các phương thức bắt buộc của trait đó. Trait `Deref`, được cung cấp
bởi thư viện chuẩn, yêu cầu chúng ta triển khai một phương thức tên là `deref`
mượn `self` và trả về một reference đến dữ liệu bên trong. Listing 15-10
chứa cài đặt của `Deref` để thêm vào định nghĩa của `MyBox`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

<span class="caption">Listing 15-10: Triển khai `Deref` trên `MyBox<T>`</span>

Cú pháp `type Target = T;` định nghĩa một associated type cho trait `Deref` sử dụng.
Associated types là một cách hơi khác để khai báo tham số generic, nhưng hiện
tại bạn không cần lo lắng về chúng; chúng ta sẽ tìm hiểu chi tiết hơn trong
Chương 19.

Chúng ta điền thân phương thức `deref` với `&self.0` để `deref` trả về một
reference đến giá trị mà chúng ta muốn truy cập với toán tử `*`; nhớ lại từ
phần [“Using Tuple Structs without Named Fields to Create Different Types”][tuple-structs]<!-- ignore -->
của Chương 5 rằng `.0` truy cập giá trị đầu tiên trong tuple struct. Hàm `main`
trong Listing 15-9 gọi `*` trên giá trị `MyBox<T>` bây giờ đã biên dịch được,
và các assertion đều pass!

Nếu không có trait `Deref`, compiler chỉ có thể dereference các reference `&`.
Phương thức `deref` cung cấp cho compiler khả năng lấy một giá trị của bất kỳ
kiểu nào triển khai `Deref` và gọi phương thức `deref` để có được một reference
`&` mà nó biết cách dereference.

Khi chúng ta nhập `*y` trong Listing 15-9, phía sau, Rust thực sự chạy đoạn
mã sau:

```rust,ignore
*(y.deref())
```

Rust thay thế toán tử `*` bằng một lời gọi tới phương thức `deref` và sau đó
một dereference bình thường, vì vậy chúng ta không cần phải nghĩ xem có cần
gọi phương thức `deref` hay không. Tính năng này của Rust cho phép chúng ta
viết mã hoạt động giống hệt, dù chúng ta đang dùng một reference thông thường
hay một kiểu triển khai `Deref`.

Lý do phương thức `deref` trả về một reference tới giá trị, và dereference
bên ngoài dấu ngoặc trong `*(y.deref())` vẫn cần thiết, liên quan tới hệ
thống ownership. Nếu phương thức `deref` trả về giá trị trực tiếp thay vì
một reference tới giá trị, giá trị đó sẽ bị move ra khỏi `self`. Chúng ta
không muốn chiếm quyền sở hữu giá trị bên trong `MyBox<T>` trong trường
hợp này hoặc trong hầu hết các trường hợp sử dụng toán tử dereference.

Lưu ý rằng toán tử `*` được thay thế bằng một lời gọi tới `deref` và sau đó
một lời gọi tới `*` chỉ một lần, mỗi khi chúng ta sử dụng `*` trong mã.
Vì việc thay thế toán tử `*` không lặp vô hạn, chúng ta cuối cùng nhận được
dữ liệu kiểu `i32`, khớp với giá trị `5` trong `assert_eq!` ở Listing 15-9.

### Chuyển đổi Deref ngầm định với Hàm và Phương thức

*Deref coercion* chuyển một reference tới một kiểu triển khai trait `Deref`
thành một reference tới kiểu khác. Ví dụ, deref coercion có thể chuyển
`&String` thành `&str` vì `String` triển khai `Deref` để trả về `&str`.
Deref coercion là một tiện ích mà Rust thực hiện trên các đối số của hàm
và phương thức, và chỉ hoạt động trên các kiểu triển khai trait `Deref`.
Nó xảy ra tự động khi chúng ta truyền một reference tới giá trị của một
kiểu nhất định làm đối số cho một hàm hoặc phương thức mà kiểu tham số
trong định nghĩa hàm/phương thức không khớp. Một chuỗi các lời gọi tới
phương thức `deref` chuyển kiểu chúng ta cung cấp thành kiểu mà tham số
cần.

Deref coercion được thêm vào Rust để lập trình viên khi viết các lời gọi
hàm và phương thức không cần thêm quá nhiều reference và dereference
tường minh với `&` và `*`. Tính năng này cũng cho phép viết nhiều mã
hơn có thể hoạt động với cả reference hoặc smart pointer.

Để thấy deref coercion hoạt động, chúng ta hãy dùng kiểu `MyBox<T>` đã
định nghĩa trong Listing 15-8 cùng với việc triển khai `Deref` mà chúng
ta đã thêm trong Listing 15-10. Listing 15-11 hiển thị định nghĩa một hàm
có tham số là string slice:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

<span class="caption">Listing 15-11: Hàm `hello` có tham số `name` kiểu `&str`</span>

Chúng ta có thể gọi hàm `hello` với một string slice làm đối số, ví dụ:
`hello("Rust");`. Deref coercion cho phép gọi hàm `hello` với một reference
tới giá trị kiểu `MyBox<String>`, như được minh họa trong Listing 15-12:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

<span class="caption">Listing 15-12: Gọi hàm `hello` với reference tới giá trị
`MyBox<String>`, hoạt động nhờ deref coercion</span>

Ở đây chúng ta gọi hàm `hello` với đối số `&m`, là một reference tới giá trị
`MyBox<String>`. Bởi vì chúng ta đã triển khai trait `Deref` trên `MyBox<T>`
trong Listing 15-10, Rust có thể chuyển `&MyBox<String>` thành `&String` bằng
cách gọi `deref`. Thư viện chuẩn cung cấp một triển khai `Deref` trên `String`
trả về một string slice, và điều này có trong tài liệu API của `Deref`. Rust
lại gọi `deref` lần nữa để chuyển `&String` thành `&str`, phù hợp với định
nghĩa của hàm `hello`.

Nếu Rust không thực hiện deref coercion, chúng ta sẽ phải viết mã như trong
Listing 15-13 thay vì mã trong Listing 15-12 để gọi `hello` với một giá trị
kiểu `&MyBox<String>`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

<span class="caption">Listing 15-13: Mã mà chúng ta phải viết nếu Rust
không có deref coercion</span>

`(*m)` dereference `MyBox<String>` thành một `String`. Sau đó, `&` và `[..]`
lấy một string slice của `String` bằng toàn bộ chuỗi để phù hợp với chữ ký
của hàm `hello`. Mã này nếu không có deref coercion sẽ khó đọc, viết và hiểu
với tất cả các ký hiệu phức tạp này. Deref coercion cho phép Rust xử lý
tự động các chuyển đổi này.

Khi trait `Deref` được định nghĩa cho các kiểu liên quan, Rust sẽ phân tích
các kiểu và sử dụng `Deref::deref` nhiều lần tùy cần để có một reference
phù hợp với kiểu của tham số. Số lần `Deref::deref` được chèn được quyết
định tại thời gian biên dịch, nên không có chi phí runtime khi sử dụng
deref coercion!

### Cách Deref Coercion Tương Tác với Mutability

Tương tự như cách bạn dùng trait `Deref` để override toán tử `*` trên
reference không thay đổi, bạn có thể dùng trait `DerefMut` để override
toán tử `*` trên reference có thể thay đổi.

Rust thực hiện deref coercion trong ba trường hợp:

* Từ `&T` sang `&U` khi `T: Deref<Target=U>`
* Từ `&mut T` sang `&mut U` khi `T: DerefMut<Target=U>`
* Từ `&mut T` sang `&U` khi `T: Deref<Target=U>`

Hai trường hợp đầu giống nhau ngoại trừ trường hợp thứ hai áp dụng cho
mutable references. Trường hợp đầu nói rằng nếu bạn có một `&T`, và `T`
triển khai `Deref` tới kiểu `U`, bạn có thể nhận được một `&U` một cách
trong suốt. Trường hợp thứ hai cũng xảy ra tương tự cho mutable references.

Trường hợp thứ ba phức tạp hơn: Rust cũng sẽ coercion một mutable reference
thành immutable reference. Nhưng ngược lại *không* thực hiện được: immutable
references sẽ không bao giờ coercion thành mutable references. Theo quy tắc
mượn, nếu bạn có một mutable reference, reference đó phải là reference duy
nhất tới dữ liệu đó (nếu không, chương trình sẽ không biên dịch). Việc
chuyển một mutable reference thành immutable reference sẽ không vi phạm
quy tắc mượn. Ngược lại, chuyển một immutable reference thành mutable
reference sẽ yêu cầu reference immutable ban đầu là reference duy nhất tới
dữ liệu đó, nhưng quy tắc mượn không đảm bảo điều này. Do đó, Rust không
thể giả định rằng chuyển đổi immutable reference thành mutable reference là
khả thi.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
