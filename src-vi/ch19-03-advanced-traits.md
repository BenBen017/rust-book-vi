## Advanced Traits

Chúng ta đã lần đầu tiên tìm hiểu về traits trong phần [“Traits: Defining Shared Behavior”][traits-defining-shared-behavior]<!-- ignore --> ở Chương 10, nhưng chưa thảo luận về các chi tiết nâng cao hơn. Giờ khi bạn đã biết thêm về Rust, chúng ta có thể đi vào các chi tiết kỹ thuật.

### Chỉ định loại placeholder trong định nghĩa Trait với Associated Types

*Associated types* kết nối một placeholder type với một trait sao cho các định nghĩa phương thức của trait có thể sử dụng các loại placeholder này trong chữ ký của chúng. Người triển khai một trait sẽ chỉ định loại cụ thể sẽ được sử dụng thay cho placeholder type cho triển khai đó. Nhờ đó, chúng ta có thể định nghĩa một trait sử dụng một số loại mà không cần biết chính xác các loại đó là gì cho đến khi trait được triển khai.

Chúng tôi đã mô tả hầu hết các tính năng nâng cao trong chương này là hiếm khi cần đến. Associated types nằm ở mức trung gian: chúng được sử dụng ít hơn các tính năng được giải thích trong phần còn lại của cuốn sách nhưng phổ biến hơn nhiều so với các tính năng khác được thảo luận trong chương này.

Một ví dụ về trait với associated type là trait `Iterator` mà thư viện chuẩn cung cấp. Associated type được đặt tên là `Item` và đại diện cho kiểu giá trị mà kiểu triển khai trait `Iterator` đang lặp qua. Định nghĩa của trait `Iterator` được trình bày như trong Listing 19-12.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-12/src/lib.rs}}
```

<span class="caption">Listing 19-12: Định nghĩa trait `Iterator` với associated type `Item`</span>

Kiểu `Item` là một placeholder, và định nghĩa phương thức `next` cho thấy nó sẽ trả về các giá trị kiểu `Option<Self::Item>`. Người triển khai trait `Iterator` sẽ chỉ định kiểu cụ thể cho `Item`, và phương thức `next` sẽ trả về một `Option` chứa giá trị của kiểu cụ thể đó.

Associated types có vẻ giống với generics, vì generics cho phép chúng ta định nghĩa một hàm mà không cần chỉ định các kiểu mà nó có thể xử lý. Để xem sự khác biệt giữa hai khái niệm này, chúng ta sẽ xem một triển khai trait `Iterator` trên một kiểu có tên là `Counter` mà chỉ định kiểu `Item` là `u32`:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

Cú pháp này có vẻ tương tự như cú pháp của generics. Vậy tại sao không định nghĩa trait `Iterator` với generics, như được trình bày trong Listing 19-13?

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-13/src/lib.rs}}
```

<span class="caption">Listing 19-13: Một định nghĩa giả tưởng của trait `Iterator` sử dụng generics</span>

Sự khác biệt là khi sử dụng generics, như trong Listing 19-13, chúng ta phải chú thích các kiểu trong mỗi triển khai; vì chúng ta cũng có thể triển khai `Iterator<String> for Counter` hoặc bất kỳ kiểu nào khác, chúng ta có thể có nhiều triển khai của `Iterator` cho `Counter`. Nói cách khác, khi một trait có tham số generic, nó có thể được triển khai cho một kiểu nhiều lần, thay đổi các kiểu cụ thể của các tham số generic mỗi lần. Khi chúng ta sử dụng phương thức `next` trên `Counter`, chúng ta phải cung cấp chú thích kiểu để chỉ ra triển khai `Iterator` nào muốn sử dụng.

Với associated types, chúng ta không cần chú thích kiểu vì chúng ta không thể triển khai một trait trên cùng một kiểu nhiều lần. Trong Listing 19-12 với định nghĩa sử dụng associated types, chúng ta chỉ có thể chọn kiểu của `Item` một lần, vì chỉ có một `impl Iterator for Counter`. Chúng ta không cần phải chỉ định rằng muốn một iterator của các giá trị `u32` ở mọi nơi mà gọi `next` trên `Counter`.

Associated types cũng trở thành một phần của hợp đồng của trait: người triển khai trait phải cung cấp một kiểu để thay thế cho placeholder của associated type. Associated types thường có một tên mô tả cách sử dụng kiểu đó, và việc tài liệu hóa associated type trong tài liệu API là một thực hành tốt.

### Tham số kiểu Generic mặc định và Nạp chồng toán tử (Operator Overloading)

Khi sử dụng tham số kiểu generic, chúng ta có thể chỉ định một kiểu cụ thể mặc định cho generic type. Điều này loại bỏ nhu cầu người triển khai trait phải chỉ định kiểu cụ thể nếu kiểu mặc định phù hợp. Bạn chỉ định kiểu mặc định khi khai báo một kiểu generic với cú pháp `<PlaceholderType=ConcreteType>`.

Một ví dụ tuyệt vời về tình huống mà kỹ thuật này hữu ích là với *operator overloading*, trong đó bạn tùy chỉnh hành vi của một toán tử (chẳng hạn `+`) trong các tình huống cụ thể.

Rust không cho phép bạn tạo toán tử riêng hoặc nạp chồng các toán tử tùy ý. Nhưng bạn có thể nạp chồng các phép toán và trait tương ứng được liệt kê trong `std::ops` bằng cách triển khai các trait liên quan đến toán tử đó. Ví dụ, trong Listing 19-14 chúng ta nạp chồng toán tử `+` để cộng hai thể hiện `Point` với nhau. Chúng ta làm điều này bằng cách triển khai trait `Add` trên struct `Point`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-14/src/main.rs}}
```

<span class="caption">Listing 19-14: Triển khai trait `Add` để nạp chồng toán tử `+` cho các thể hiện `Point`</span>

Phương thức `add` cộng các giá trị `x` của hai thể hiện `Point` và các giá trị `y` của hai thể hiện `Point` để tạo ra một `Point` mới. Trait `Add` có một associated type tên là `Output` xác định kiểu được trả về từ phương thức `add`.

Tham số kiểu generic mặc định trong mã này nằm trong trait `Add`. Đây là định nghĩa của nó:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Mã này có vẻ khá quen thuộc: một trait với một phương thức và một associated type. Phần mới là `Rhs=Self`: cú pháp này được gọi là *default type parameters*. Tham số kiểu generic `Rhs` (viết tắt của “right hand side”) xác định kiểu của tham số `rhs` trong phương thức `add`. Nếu chúng ta không chỉ định một kiểu cụ thể cho `Rhs` khi triển khai trait `Add`, kiểu của `Rhs` sẽ mặc định là `Self`, tức là kiểu mà chúng ta đang triển khai `Add`.

Khi chúng ta triển khai `Add` cho `Point`, chúng ta sử dụng mặc định cho `Rhs` vì muốn cộng hai thể hiện `Point`. Hãy xem một ví dụ về việc triển khai trait `Add` mà chúng ta muốn tùy chỉnh kiểu `Rhs` thay vì sử dụng mặc định.

Chúng ta có hai struct, `Millimeters` và `Meters`, chứa các giá trị ở các đơn vị khác nhau. Việc bao bọc mỏng một kiểu hiện có trong một struct khác được gọi là *newtype pattern*, mà chúng ta mô tả chi tiết hơn trong phần [“Using the Newtype Pattern to Implement External Traits on External Types”][newtype]<!-- ignore -->. Chúng ta muốn cộng các giá trị millimeters với các giá trị meters và đảm bảo việc triển khai `Add` thực hiện chuyển đổi đúng. Chúng ta có thể triển khai `Add` cho `Millimeters` với `Meters` là `Rhs`, như được trình bày trong Listing 19-15.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-15/src/lib.rs}}
```

<span class="caption">Listing 19-15: Triển khai trait `Add` trên `Millimeters` để cộng `Millimeters` với `Meters`</span>

Để cộng `Millimeters` và `Meters`, chúng ta chỉ định `impl Add<Meters>` để đặt giá trị của tham số kiểu `Rhs` thay vì sử dụng mặc định là `Self`.

Bạn sẽ sử dụng default type parameters theo hai cách chính:

* Mở rộng một kiểu mà không phá vỡ mã hiện có
* Cho phép tùy chỉnh trong các trường hợp cụ thể mà hầu hết người dùng không cần đến

Trait `Add` trong thư viện chuẩn là một ví dụ cho mục đích thứ hai: thường thì bạn sẽ cộng hai kiểu giống nhau, nhưng trait `Add` cung cấp khả năng tùy chỉnh vượt ra ngoài điều đó. Việc sử dụng default type parameter trong định nghĩa trait `Add` có nghĩa là bạn không cần chỉ định tham số bổ sung hầu hết thời gian. Nói cách khác, một chút boilerplate khi triển khai không cần thiết, giúp trait dễ sử dụng hơn.

Mục đích thứ nhất tương tự mục đích thứ hai nhưng ngược lại: nếu bạn muốn thêm một tham số kiểu vào một trait hiện có, bạn có thể cung cấp giá trị mặc định để mở rộng chức năng của trait mà không phá vỡ mã triển khai hiện có.

### Cú pháp đầy đủ để phân biệt: Gọi phương thức cùng tên

Rust không ngăn cản một trait có phương thức cùng tên với phương thức của trait khác, và cũng không ngăn cản bạn triển khai cả hai trait trên cùng một kiểu. Cũng có thể triển khai phương thức trực tiếp trên kiểu với cùng tên như các phương thức từ trait.

Khi gọi các phương thức cùng tên, bạn cần thông báo cho Rust biết bạn muốn sử dụng phương thức nào. Xem xét mã trong Listing 19-16, nơi chúng ta đã định nghĩa hai trait, `Pilot` và `Wizard`, đều có phương thức gọi là `fly`. Sau đó, chúng ta triển khai cả hai trait trên kiểu `Human`, mà kiểu này đã có một phương thức tên là `fly` được triển khai trực tiếp. Mỗi phương thức `fly` thực hiện một hành động khác nhau.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-16/src/main.rs:here}}
```

<span class="caption">Listing 19-16: Hai trait được định nghĩa có phương thức `fly` và được triển khai trên kiểu `Human`, đồng thời một phương thức `fly` được triển khai trực tiếp trên `Human`</span>

Khi chúng ta gọi `fly` trên một thể hiện của `Human`, trình biên dịch mặc định sẽ gọi phương thức được triển khai trực tiếp trên kiểu, như được trình bày trong Listing 19-17.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-17/src/main.rs:here}}
```

<span class="caption">Listing 19-17: Gọi `fly` trên một thể hiện của `Human`</span>

Chạy mã này sẽ in ra `*waving arms furiously*`, cho thấy Rust đã gọi phương thức `fly` được triển khai trực tiếp trên `Human`.

Để gọi các phương thức `fly` từ trait `Pilot` hoặc trait `Wizard`, chúng ta cần sử dụng cú pháp rõ ràng hơn để chỉ định phương thức `fly` mà chúng ta muốn. Listing 19-18 minh họa cú pháp này.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-18/src/main.rs:here}}
```

<span class="caption">Listing 19-18: Chỉ định phương thức `fly` của trait nào mà chúng ta muốn gọi</span>

Việc chỉ định tên trait trước tên phương thức giúp Rust hiểu rõ chúng ta muốn gọi triển khai `fly` nào. Chúng ta cũng có thể viết `Human::fly(&person)`, điều này tương đương với `person.fly()` mà chúng ta đã sử dụng trong Listing 19-18, nhưng cú pháp này dài hơn một chút nếu không cần phân biệt.

Chạy mã này sẽ in ra:

```console
{{#include ../listings/ch19-advanced-features/listing-19-18/output.txt}}
```

Vì phương thức `fly` nhận một tham số `self`, nếu chúng ta có hai *kiểu* đều triển khai cùng một *trait*, Rust có thể xác định triển khai của trait nào sẽ được sử dụng dựa trên kiểu của `self`.

Tuy nhiên, các hàm liên quan (associated functions) mà không phải là phương thức thì không có tham số `self`. Khi có nhiều kiểu hoặc trait định nghĩa các hàm không phải phương thức với cùng tên hàm, Rust không luôn biết bạn muốn dùng kiểu nào trừ khi bạn sử dụng *fully qualified syntax*. Ví dụ, trong Listing 19-19, chúng ta tạo một trait cho một trại động vật muốn đặt tên tất cả chó con là *Spot*. Chúng ta tạo trait `Animal` với một hàm liên quan không phải phương thức `baby_name`. Trait `Animal` được triển khai cho struct `Dog`, trên đó chúng ta cũng cung cấp một hàm liên quan không phải phương thức `baby_name` trực tiếp.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-19/src/main.rs}}
```

<span class="caption">Listing 19-19: Một trait với một associated function và một kiểu có associated function cùng tên, đồng thời triển khai trait đó</span>

Chúng ta triển khai mã để đặt tên tất cả chó con là Spot trong hàm liên quan `baby_name` được định nghĩa trực tiếp trên `Dog`. Kiểu `Dog` cũng triển khai trait `Animal`, mô tả các đặc điểm mà tất cả động vật có. Chó con được gọi là puppies, và điều này được thể hiện trong việc triển khai trait `Animal` trên `Dog` trong hàm `baby_name` liên quan đến trait `Animal`.

Trong `main`, chúng ta gọi hàm `Dog::baby_name`, phương thức này gọi hàm liên quan được định nghĩa trực tiếp trên `Dog`. Mã này sẽ in ra:

```console
{{#include ../listings/ch19-advanced-features/listing-19-19/output.txt}}
```

Kết quả này không phải là điều chúng ta muốn. Chúng ta muốn gọi hàm `baby_name` là một phần của trait `Animal` mà chúng ta đã triển khai trên `Dog` để mã in ra `A baby dog is called a puppy`. Kỹ thuật chỉ định tên trait mà chúng ta đã sử dụng trong Listing 19-18 không giúp được ở đây; nếu chúng ta thay đổi `main` thành mã trong Listing 19-20, sẽ xảy ra lỗi biên dịch.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-20/src/main.rs:here}}
```

<span class="caption">Listing 19-20: Thử gọi hàm `baby_name` từ trait `Animal`, nhưng Rust không biết triển khai nào sẽ sử dụng</span>

Vì `Animal::baby_name` không có tham số `self`, và có thể có các kiểu khác triển khai trait `Animal`, Rust không thể xác định triển khai nào của `Animal::baby_name` mà chúng ta muốn sử dụng. Chúng ta sẽ nhận được lỗi biên dịch sau:

```console
{{#include ../listings/ch19-advanced-features/listing-19-20/output.txt}}
```

Để phân biệt và cho Rust biết rằng chúng ta muốn sử dụng triển khai của `Animal` cho `Dog` thay vì triển khai của `Animal` cho một kiểu khác, chúng ta cần sử dụng cú pháp đầy đủ (fully qualified syntax). Listing 19-21 minh họa cách sử dụng cú pháp đầy đủ.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-21/src/main.rs:here}}
```

<span class="caption">Listing 19-21: Sử dụng cú pháp đầy đủ để chỉ định rằng chúng ta muốn gọi hàm `baby_name` từ trait `Animal` được triển khai trên `Dog`</span>

Chúng ta cung cấp cho Rust một chú thích kiểu trong dấu ngoặc nhọn, cho biết chúng ta muốn gọi phương thức `baby_name` từ trait `Animal` như được triển khai trên `Dog` bằng cách nói rằng chúng ta muốn coi kiểu `Dog` như một `Animal` cho lần gọi hàm này. Mã này bây giờ sẽ in ra kết quả như mong muốn:

```console
{{#include ../listings/ch19-advanced-features/listing-19-21/output.txt}}
```

Nói chung, cú pháp đầy đủ (fully qualified syntax) được định nghĩa như sau:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Đối với các hàm liên quan (associated functions) không phải là phương thức, sẽ không có `receiver`: chỉ có danh sách các tham số khác. Bạn có thể sử dụng cú pháp đầy đủ ở bất cứ nơi nào gọi hàm hoặc phương thức. Tuy nhiên, bạn được phép bỏ bất kỳ phần nào của cú pháp này mà Rust có thể suy ra từ các thông tin khác trong chương trình. Bạn chỉ cần sử dụng cú pháp dài dòng hơn này trong những trường hợp có nhiều triển khai sử dụng cùng một tên và Rust cần giúp xác định triển khai nào bạn muốn gọi.

### Sử dụng Supertraits để yêu cầu chức năng của một trait trong một trait khác

Đôi khi, bạn có thể viết một định nghĩa trait phụ thuộc vào một trait khác: để một kiểu triển khai trait đầu tiên, bạn muốn yêu cầu kiểu đó cũng phải triển khai trait thứ hai. Bạn làm điều này để định nghĩa trait của bạn có thể sử dụng các thành phần liên quan (associated items) của trait thứ hai. Trait mà định nghĩa trait của bạn dựa vào được gọi là *supertrait* của trait đó.

Ví dụ, giả sử chúng ta muốn tạo một trait `OutlinePrint` với một phương thức `outline_print` sẽ in một giá trị sao cho được khung bởi các dấu sao. Tức là, với struct `Point` triển khai trait `Display` của thư viện chuẩn để hiển thị `(x, y)`, khi chúng ta gọi `outline_print` trên một thể hiện `Point` có `x = 1` và `y = 3`, nó sẽ in ra:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Trong việc triển khai phương thức `outline_print`, chúng ta muốn sử dụng chức năng của trait `Display`. Do đó, chúng ta cần chỉ định rằng trait `OutlinePrint` sẽ chỉ hoạt động cho các kiểu cũng triển khai `Display` và cung cấp các chức năng mà `OutlinePrint` cần. Chúng ta có thể làm điều đó trong định nghĩa trait bằng cách chỉ định `OutlinePrint: Display`. Kỹ thuật này tương tự như việc thêm một trait bound vào trait. Listing 19-22 trình bày một triển khai của trait `OutlinePrint`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-22/src/main.rs:here}}
```

<span class="caption">Listing 19-22: Triển khai trait `OutlinePrint` yêu cầu chức năng từ `Display`</span>

Vì chúng ta đã chỉ định rằng `OutlinePrint` yêu cầu trait `Display`, chúng ta có thể sử dụng hàm `to_string` được triển khai tự động cho bất kỳ kiểu nào triển khai `Display`. Nếu chúng ta thử sử dụng `to_string` mà không thêm dấu hai chấm và chỉ định trait `Display` sau tên trait, chúng ta sẽ nhận được lỗi nói rằng không tìm thấy phương thức nào tên là `to_string` cho kiểu `&Self` trong phạm vi hiện tại.

Hãy xem điều gì xảy ra khi chúng ta cố triển khai `OutlinePrint` trên một kiểu không triển khai `Display`, chẳng hạn như struct `Point`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

Chúng ta sẽ nhận được lỗi nói rằng trait `Display` là bắt buộc nhưng chưa được triển khai:

```console
{{#include ../listings/ch19-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Để khắc phục, chúng ta triển khai trait `Display` trên `Point` và thỏa mãn ràng buộc mà `OutlinePrint` yêu cầu, như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

Sau đó, việc triển khai trait `OutlinePrint` trên `Point` sẽ biên dịch thành công, và chúng ta có thể gọi `outline_print` trên một thể hiện của `Point` để hiển thị nó trong một khung dấu sao.

### Sử dụng Newtype Pattern để triển khai các trait bên ngoài trên các kiểu bên ngoài

Trong Chương 10, trong phần [“Implementing a Trait on a Type”][implementing-a-trait-on-a-type]<!-- ignore -->, chúng ta đã đề cập đến quy tắc orphan (orphan rule) cho rằng chúng ta chỉ được phép triển khai một trait trên một kiểu nếu trait hoặc kiểu đó là cục bộ trong crate của chúng ta. Có thể vượt qua giới hạn này bằng cách sử dụng *newtype pattern*, liên quan đến việc tạo một kiểu mới trong một tuple struct. (Chúng ta đã học về tuple structs trong phần [“Using Tuple Structs without Named Fields to Create Different Types”][tuple-structs]<!-- ignore --> của Chương 5.) Tuple struct sẽ có một trường và là một wrapper mỏng quanh kiểu mà chúng ta muốn triển khai trait. Khi đó, kiểu wrapper là cục bộ trong crate của chúng ta, và chúng ta có thể triển khai trait trên wrapper. Thuật ngữ *newtype* xuất phát từ ngôn ngữ lập trình Haskell. Việc sử dụng pattern này không ảnh hưởng tới hiệu năng khi chạy, và kiểu wrapper sẽ bị loại bỏ tại thời điểm biên dịch.

Ví dụ, giả sử chúng ta muốn triển khai `Display` trên `Vec<T>`, điều mà quy tắc orphan ngăn cản chúng ta làm trực tiếp vì trait `Display` và kiểu `Vec<T>` được định nghĩa bên ngoài crate của chúng ta. Chúng ta có thể tạo một struct `Wrapper` chứa một thể hiện của `Vec<T>`; sau đó triển khai `Display` trên `Wrapper` và sử dụng giá trị `Vec<T>`, như được minh họa trong Listing 19-23.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-23/src/main.rs}}
```

<span class="caption">Listing 19-23: Tạo một kiểu `Wrapper` quanh `Vec<String>` để triển khai `Display`</span>

Việc triển khai `Display` sử dụng `self.0` để truy cập vào `Vec<T>` bên trong, vì `Wrapper` là một tuple struct và `Vec<T>` là phần tử ở chỉ số 0 trong tuple. Sau đó, chúng ta có thể sử dụng các chức năng của kiểu `Display` trên `Wrapper`.

Nhược điểm của kỹ thuật này là `Wrapper` là một kiểu mới, vì vậy nó không có các phương thức của giá trị mà nó đang chứa. Chúng ta sẽ phải triển khai tất cả các phương thức của `Vec<T>` trực tiếp trên `Wrapper` sao cho các phương thức ủy quyền (delegate) tới `self.0`, điều này cho phép chúng ta sử dụng `Wrapper` giống hệt như một `Vec<T>`. Nếu muốn kiểu mới có mọi phương thức mà kiểu bên trong có, việc triển khai trait `Deref` (được thảo luận trong Chương 15, phần [“Treating Smart Pointers Like Regular References with the `Deref` Trait”][smart-pointer-deref]<!-- ignore -->) trên `Wrapper` để trả về kiểu bên trong sẽ là một giải pháp. Nếu không muốn kiểu `Wrapper` có tất cả các phương thức của kiểu bên trong — ví dụ, để hạn chế hành vi của kiểu `Wrapper` — chúng ta chỉ cần triển khai thủ công những phương thức mà chúng ta muốn.

Pattern newtype này cũng hữu ích ngay cả khi không có trait liên quan. Bây giờ hãy chuyển sang và xem một số cách nâng cao để tương tác với hệ thống kiểu của Rust.

[newtype]: ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
[implementing-a-trait-on-a-type]:
ch10-02-traits.html#implementing-a-trait-on-a-type
[traits-defining-shared-behavior]:
ch10-02-traits.html#traits-defining-shared-behavior
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
