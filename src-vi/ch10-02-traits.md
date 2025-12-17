## Traits: Định nghĩa Hành vi Chung

Một *trait* định nghĩa các chức năng mà một type cụ thể có và có thể chia sẻ
với các type khác. Chúng ta có thể sử dụng traits để định nghĩa hành vi
chung theo cách trừu tượng. Chúng ta có thể dùng *trait bounds* để chỉ định
rằng một generic type có thể là bất kỳ type nào mà có hành vi nhất định.

> Lưu ý: Traits tương tự như một tính năng thường được gọi là *interfaces*
> trong các ngôn ngữ khác, mặc dù có một số khác biệt.

### Defining a Trait

Hành vi của một type bao gồm các method mà chúng ta có thể gọi trên type đó.
Các type khác nhau chia sẻ cùng một hành vi nếu chúng ta có thể gọi cùng
một phương thức trên tất cả các type đó. Định nghĩa trait là một cách để
nhóm các method signature lại với nhau nhằm xác định một tập hợp hành vi
cần thiết để thực hiện một mục đích nào đó.

Ví dụ, giả sử chúng ta có nhiều struct lưu trữ các loại và lượng text khác
nhau: một struct `NewsArticle` chứa một bài báo được lưu trữ ở một địa
điểm cụ thể, và một `Tweet` có tối đa 280 ký tự cùng với metadata chỉ
ra đó là tweet mới, retweet, hay trả lời một tweet khác.

Chúng ta muốn tạo một media aggregator library crate tên là `aggregator`
có thể hiển thị tóm tắt dữ liệu có thể được lưu trong một instance của
`NewsArticle` hoặc `Tweet`. Để làm điều này, chúng ta cần một bản tóm
tắt từ mỗi type, và chúng ta sẽ yêu cầu bản tóm tắt đó bằng cách gọi
method `summarize` trên một instance. Listing 10-12 hiển thị định nghĩa
của một public `Summary` trait biểu thị hành vi này.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

<span class="caption">Listing 10-12: Một `Summary` trait bao gồm hành vi được
cung cấp bởi method `summarize`</span>

Ở đây, chúng ta khai báo một trait bằng từ khóa `trait` và sau đó là tên
của trait, trong trường hợp này là `Summary`. Chúng ta cũng khai báo trait
là `pub` để các crate phụ thuộc vào crate này cũng có thể sử dụng trait,
như chúng ta sẽ thấy trong một vài ví dụ. Bên trong cặp ngoặc nhọn, chúng
ta khai báo các method signature mô tả hành vi của các type triển khai trait
này, trong trường hợp này là `fn summarize(&self) -> String`.

Sau method signature, thay vì cung cấp phần implementation bên trong cặp
ngoặc nhọn, chúng ta dùng dấu chấm phẩy. Mỗi type triển khai trait này
phải cung cấp hành vi riêng cho thân của method. Compiler sẽ đảm bảo rằng
bất kỳ type nào có trait `Summary` sẽ có method `summarize` được định nghĩa
với signature chính xác như vậy.

Một trait có thể có nhiều method trong thân của nó: các method signature
được liệt kê từng dòng một và mỗi dòng kết thúc bằng dấu chấm phẩy.

### Triển khai Trait trên một Type

Bây giờ chúng ta đã định nghĩa các signature mong muốn của các method trong
trait `Summary`, chúng ta có thể triển khai nó trên các type trong media
aggregator của mình. Listing 10-13 hiển thị một triển khai của trait
`Summary` trên struct `NewsArticle` sử dụng headline, author và location
để tạo giá trị trả về của `summarize`. Đối với struct `Tweet`, chúng ta
định nghĩa `summarize` là username theo sau là toàn bộ nội dung tweet,
giả sử nội dung tweet đã bị giới hạn tối đa 280 ký tự.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

<span class="caption">Listing 10-13: Triển khai trait `Summary` trên các type
`NewsArticle` và `Tweet`</span>

Triển khai một trait trên một type tương tự như triển khai các method
thường. Sự khác biệt là sau `impl`, chúng ta đặt tên trait muốn triển khai,
sau đó dùng từ khóa `for`, và rồi chỉ định tên của type mà chúng ta muốn
triển khai trait cho. Bên trong khối `impl`, chúng ta đặt các method
signature mà định nghĩa trait đã định nghĩa. Thay vì thêm dấu chấm phẩy
sau mỗi signature, chúng ta dùng cặp ngoặc nhọn và điền phần thân của
method với hành vi cụ thể mà chúng ta muốn các method của trait có trên
type cụ thể đó.

Bây giờ khi thư viện đã triển khai trait `Summary` trên `NewsArticle` và
`Tweet`, người dùng của crate có thể gọi các method của trait trên các
instance của `NewsArticle` và `Tweet` theo cùng cách chúng ta gọi các
method thường. Sự khác biệt duy nhất là người dùng phải đưa trait vào
scope cũng như các type. Dưới đây là ví dụ về cách một binary crate có
thể sử dụng thư viện `aggregator` của chúng ta:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Đoạn code này in ra `1 new tweet: horse_ebooks: of course, as you probably already
know, people`.

Các crate khác phụ thuộc vào crate `aggregator` cũng có thể đưa trait `Summary`
vào scope để triển khai `Summary` trên các type của riêng họ. Một hạn chế cần
lưu ý là chúng ta chỉ có thể triển khai một trait trên một type nếu ít nhất
một trong trait hoặc type là local với crate của chúng ta. Ví dụ, chúng ta
có thể triển khai các trait trong standard library như `Display` trên một
type tùy chỉnh như `Tweet` như một phần của chức năng crate `aggregator` của
chúng ta, vì type `Tweet` là local với crate `aggregator` của chúng ta. Chúng
ta cũng có thể triển khai `Summary` trên `Vec<T>` trong crate `aggregator` của
chúng ta, vì trait `Summary` là local với crate `aggregator`.

Nhưng chúng ta không thể triển khai các trait bên ngoài trên các type bên ngoài.
Ví dụ, chúng ta không thể triển khai trait `Display` trên `Vec<T>` trong crate
`aggregator` của chúng ta, vì `Display` và `Vec<T>` đều được định nghĩa trong
standard library và không phải là local với crate `aggregator`. Hạn chế này
là một phần của một đặc tính gọi là *coherence*, và cụ thể hơn là *orphan rule*,
được gọi như vậy vì type cha không có mặt. Quy tắc này đảm bảo rằng code của
người khác không thể phá vỡ code của bạn và ngược lại. Nếu không có quy tắc này,
hai crate có thể triển khai cùng một trait cho cùng một type, và Rust sẽ không
biết triển khai nào để sử dụng.

### Default Implementations

Đôi khi sẽ hữu ích khi có hành vi mặc định cho một số hoặc tất cả các method
trong một trait thay vì yêu cầu phải triển khai tất cả các method trên mọi type.
Khi đó, khi triển khai trait trên một type cụ thể, chúng ta có thể giữ nguyên
hoặc ghi đè hành vi mặc định của từng method.

Trong Listing 10-14, chúng ta xác định một chuỗi mặc định cho method `summarize`
của trait `Summary` thay vì chỉ định nghĩa chữ ký của method, như chúng ta đã
làm trong Listing 10-12.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

<span class="caption">Listing 10-14: Định nghĩa trait `Summary` với
cài đặt mặc định cho method `summarize`</span>

Để sử dụng cài đặt mặc định để tóm tắt các instance của `NewsArticle`, chúng ta
xác định một `impl` block rỗng với `impl Summary for NewsArticle {}`.

Mặc dù bây giờ chúng ta không còn định nghĩa method `summarize` trực tiếp trên
`NewsArticle`, chúng ta đã cung cấp một cài đặt mặc định và chỉ định rằng
`NewsArticle` triển khai trait `Summary`. Kết quả là, chúng ta vẫn có thể gọi
method `summarize` trên một instance của `NewsArticle`, như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Mã này sẽ in ra `New article available! (Read more...)`.

Việc tạo một cài đặt mặc định không yêu cầu chúng ta thay đổi gì về
cách triển khai trait `Summary` trên `Tweet` trong Listing 10-13. Lý do là cú pháp
để ghi đè một cài đặt mặc định giống hệt cú pháp để triển khai một method của
trait mà không có cài đặt mặc định.

Các cài đặt mặc định có thể gọi các method khác trong cùng một trait, ngay cả
khi các method đó không có cài đặt mặc định. Theo cách này, một trait có thể
cung cấp nhiều chức năng hữu ích và chỉ yêu cầu người triển khai định nghĩa
một phần nhỏ trong số đó. Ví dụ, chúng ta có thể định nghĩa trait `Summary`
có một method `summarize_author` mà bắt buộc phải triển khai, rồi sau đó
định nghĩa method `summarize` có cài đặt mặc định gọi đến `summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Để sử dụng phiên bản `Summary` này, chúng ta chỉ cần định nghĩa `summarize_author`
khi triển khai trait trên một kiểu:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

Sau khi chúng ta định nghĩa `summarize_author`, 
chúng ta có thể gọi `summarize` trên các instance của struct `Tweet`, 
và implementation mặc định của `summarize` sẽ gọi định nghĩa của `summarize_author` mà chúng ta đã cung cấp. 
Bởi vì chúng ta đã triển khai `summarize_author`, trait `Summary` 
đã cung cấp cho chúng ta hành vi của phương thức `summarize` mà không yêu cầu phải viết thêm bất kỳ code nào.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Mã này in ra `1 new tweet: (Read more from @horse_ebooks...)`.

Lưu ý rằng không thể gọi triển khai mặc định từ một triển khai ghi đè của cùng phương thức đó.

### Traits như Tham Số

Bây giờ bạn đã biết cách định nghĩa và triển khai traits, chúng ta có thể khám phá cách sử dụng traits để định nghĩa các hàm nhận nhiều loại khác nhau. Chúng ta sẽ sử dụng trait `Summary` mà chúng ta đã triển khai trên các kiểu `NewsArticle` và `Tweet` trong Listing 10-13 để định nghĩa một hàm `notify` gọi phương thức `summarize` trên tham số `item` của nó, mà là một kiểu nào đó triển khai trait `Summary`. Để làm điều này, chúng ta sử dụng cú pháp `impl Trait`, như sau:


```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

Thay vì một kiểu cụ thể cho tham số `item`, chúng ta chỉ định từ khóa `impl` và tên trait.
 Tham số này chấp nhận bất kỳ kiểu nào triển khai trait được chỉ định. 
 Trong thân hàm `notify`, chúng ta có thể gọi bất kỳ phương thức nào trên `item` mà đến từ trait `Summary`, 
 chẳng hạn như `summarize`. Chúng ta có thể gọi `notify` và truyền vào bất kỳ thể hiện nào của `NewsArticle` hoặc `Tweet`. 
 Mã gọi hàm với bất kỳ kiểu nào khác, chẳng hạn như `String` hoặc `i32`, 
 sẽ không biên dịch vì các kiểu đó không triển khai `Summary`.

<!-- Old headings. Do not remove or links may break. -->
<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Cú pháp Trait Bound

Cú pháp `impl Trait` hoạt động cho các trường hợp đơn giản nhưng thực ra là cú pháp ngắn gọn cho một dạng dài hơn được gọi là *trait bound*; nó trông như sau:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Phiên bản dài hơn này tương đương với ví dụ trong phần trước nhưng chi tiết hơn. 
Chúng ta đặt ràng buộc trait cùng với khai báo của tham số kiểu generic ngay sau dấu hai chấm và bên trong dấu ngoặc nhọn (`<>`).


Cú pháp `impl Trait` tiện lợi và giúp viết code ngắn gọn hơn trong những trường hợp đơn giản, 
trong khi cú pháp ràng buộc trait đầy đủ có thể diễn đạt sự phức tạp hơn trong các trường hợp khác. 
Ví dụ, chúng ta có thể có hai tham số thực thi `Summary`. Việc làm này với cú pháp `impl Trait` sẽ trông như sau:


```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Việc sử dụng `impl Trait` là phù hợp nếu chúng ta muốn hàm này cho phép `item1` và
`item2` có các kiểu khác nhau (miễn là cả hai kiểu đều thực thi `Summary`). Tuy nhiên, nếu
chúng ta muốn buộc cả hai tham số phải cùng kiểu, thì phải sử dụng trait bound, như sau:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

Kiểu generic `T` được chỉ định làm kiểu của các tham số `item1` và `item2` giới hạn hàm sao cho kiểu cụ thể của giá trị được truyền vào `item1` và `item2` phải giống nhau.

#### Chỉ định nhiều ràng buộc trait với cú pháp `+`

Chúng ta cũng có thể chỉ định nhiều hơn một ràng buộc trait. Giả sử chúng ta muốn `notify` sử dụng cả định dạng hiển thị lẫn tóm tắt trên `item`: chúng ta chỉ định trong định nghĩa `notify` rằng `item` phải thực thi cả `Display` và `Summary`. Chúng ta có thể làm điều này bằng cú pháp `+`:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

Cú pháp `+` cũng hợp lệ với các ràng buộc trait trên các kiểu generic:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

Với hai ràng buộc trait được chỉ định, thân của `notify` có thể gọi `summarize` và sử dụng `{}` để định dạng `item`.

#### Ràng buộc trait rõ ràng hơn với câu lệnh `where`

Sử dụng quá nhiều ràng buộc trait cũng có nhược điểm. Mỗi kiểu generic có các ràng buộc trait riêng, vì vậy các hàm với nhiều tham số kiểu generic có thể chứa rất nhiều thông tin ràng buộc trait giữa tên hàm và danh sách tham số, làm cho chữ ký hàm khó đọc. Vì lý do này, Rust có cú pháp thay thế để chỉ định ràng buộc trait bên trong một câu lệnh `where` sau chữ ký hàm. Thay vì viết như sau:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

we can use a `where` clause, like this:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

Chữ ký của hàm này gọn gàng hơn: tên hàm, danh sách tham số và kiểu trả về được đặt gần nhau, 
tương tự như một hàm không có nhiều ràng buộc trait.

### Trả về các kiểu thực thi trait

Chúng ta cũng có thể sử dụng cú pháp `impl Trait` ở vị trí trả về để trả về một giá trị của kiểu nào đó thực thi một trait, như minh họa sau:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Bằng cách sử dụng `impl Summary` cho kiểu trả về, chúng ta chỉ định rằng hàm `returns_summarizable` 
trả về một kiểu nào đó thực thi trait `Summary` mà không cần đặt tên kiểu cụ thể. Trong trường hợp này, 
`returns_summarizable` trả về một `Tweet`, nhưng mã gọi hàm này không cần biết điều đó.

Khả năng chỉ định kiểu trả về chỉ dựa trên trait mà nó thực thi đặc biệt hữu ích trong bối cảnh closures và iterators, được đề cập trong Chương 13. Closures và iterators tạo ra các kiểu mà chỉ trình biên dịch biết hoặc các kiểu rất dài để chỉ định. Cú pháp `impl Trait` cho phép bạn chỉ định một cách ngắn gọn rằng một hàm trả về một kiểu nào đó thực thi trait `Iterator` mà không cần viết ra kiểu rất dài.

Tuy nhiên, bạn chỉ có thể sử dụng `impl Trait` nếu trả về một kiểu duy nhất. Ví dụ, đoạn mã này trả về hoặc một `NewsArticle` hoặc một `Tweet` với kiểu trả về được chỉ định là `impl Summary` sẽ không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Việc trả về hoặc một `NewsArticle` hoặc một `Tweet` không được phép do các hạn chế liên quan đến cách cú pháp `impl Trait` được triển khai trong trình biên dịch. Chúng ta sẽ đề cập cách viết một hàm với hành vi này trong phần [“Sử dụng Trait Objects cho các giá trị có kiểu khác nhau”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore --> của Chương 17.

### Sử dụng ràng buộc trait để triển khai phương thức có điều kiện

Bằng cách sử dụng ràng buộc trait với một khối `impl` sử dụng tham số kiểu generic, chúng ta có thể triển khai phương thức có điều kiện cho các kiểu thực thi các trait đã chỉ định. Ví dụ, kiểu `Pair<T>` trong Listing 10-15 luôn triển khai hàm `new` để trả về một thể hiện mới của `Pair<T>` (nhớ lại từ phần [“Định nghĩa phương thức”][methods]<!-- ignore --> của Chương 5 rằng `Self` là bí danh cho kiểu của khối `impl`, trong trường hợp này là `Pair<T>`). Nhưng trong khối `impl` tiếp theo, `Pair<T>` chỉ triển khai phương thức `cmp_display` nếu kiểu bên trong `T` thực thi trait `PartialOrd` cho phép so sánh *và* trait `Display` cho phép in ra.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

<span class="caption">Listing 10-15: Triển khai phương thức có điều kiện trên một kiểu generic tùy thuộc vào ràng buộc trait</span>

Chúng ta cũng có thể triển khai một trait có điều kiện cho bất kỳ kiểu nào thực thi một trait khác. Các triển khai của một trait trên bất kỳ kiểu nào thỏa mãn các ràng buộc trait được gọi là *blanket implementations* và được sử dụng rộng rãi trong thư viện chuẩn của Rust. Ví dụ, thư viện chuẩn triển khai trait `ToString` trên bất kỳ kiểu nào thực thi trait `Display`. Khối `impl` trong thư viện chuẩn trông tương tự như đoạn mã này:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Vì thư viện chuẩn có triển khai blanket này, chúng ta có thể gọi phương thức `to_string` được định nghĩa bởi trait `ToString` trên bất kỳ kiểu nào thực thi trait `Display`. Ví dụ, chúng ta có thể chuyển các số nguyên thành giá trị `String` tương ứng như sau vì các số nguyên thực thi `Display`:

```rust
let s = 3.to_string();
```

Các triển khai blanket xuất hiện trong tài liệu của trait ở phần “Implementors”.

Traits và ràng buộc trait cho phép chúng ta viết code sử dụng tham số kiểu generic để giảm sự trùng lặp, đồng thời chỉ định cho trình biên dịch rằng chúng ta muốn kiểu generic có hành vi cụ thể. Trình biên dịch sau đó có thể sử dụng thông tin ràng buộc trait để kiểm tra rằng tất cả các kiểu cụ thể được sử dụng với code của chúng ta cung cấp hành vi đúng. Trong các ngôn ngữ kiểu động, chúng ta sẽ nhận lỗi tại thời điểm chạy nếu gọi một phương thức trên kiểu không định nghĩa phương thức đó. Nhưng Rust chuyển các lỗi này sang thời điểm biên dịch, buộc chúng ta phải sửa lỗi trước khi code có thể chạy. Thêm vào đó, chúng ta không phải viết code kiểm tra hành vi tại thời điểm chạy vì đã kiểm tra ở thời điểm biên dịch. Việc này cải thiện hiệu năng mà không phải hy sinh tính linh hoạt của generic.

[using-trait-objects-that-allow-for-values-of-different-types]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
