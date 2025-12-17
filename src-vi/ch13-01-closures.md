<!-- Old heading. Do not remove or links may break. -->
<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

## Closures: Hàm Ẩn Danh Có Khả Năng Bắt Giá Trị Môi Trường

Closures trong Rust là các hàm ẩn danh mà bạn có thể lưu vào biến hoặc
truyền làm đối số cho các hàm khác. Bạn có thể định nghĩa closure ở một
nơi và sau đó gọi nó ở nơi khác để thực thi trong một ngữ cảnh khác.
Khác với các hàm thông thường, closures có thể “bắt” các giá trị từ phạm
vi nơi chúng được tạo. Chúng ta sẽ minh họa cách các tính năng này của
closures giúp tái sử dụng code và tùy chỉnh hành vi chương trình.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

### Bắt Giá Trị Môi Trường với Closures

Trước tiên, chúng ta sẽ xem cách closures có thể “bắt” các giá trị từ môi
trường nơi chúng được định nghĩa để dùng sau này. Ví dụ sau đây: Thỉnh
thoảng, công ty áo thun của chúng ta sẽ tặng một chiếc áo phiên bản giới
hạn cho một người trong danh sách email như một chương trình khuyến mãi.
Người trong danh sách email có thể tùy chọn thêm màu áo yêu thích vào hồ
sơ của họ. Nếu người được chọn đã có màu yêu thích, họ sẽ nhận áo màu
đó. Nếu chưa, họ sẽ nhận màu áo mà công ty đang còn nhiều nhất trong kho.

Có nhiều cách để triển khai điều này. Trong ví dụ này, chúng ta sẽ dùng
enum `ShirtColor` với hai biến thể `Red` và `Blue` (giới hạn số màu cho
đơn giản). Kho hàng được biểu diễn bằng struct `Inventory`, có trường
`shirts` chứa `Vec<ShirtColor>` đại diện cho các màu áo hiện có trong kho.
Phương thức `giveaway` của `Inventory` nhận tùy chọn màu áo của người
thắng cuộc và trả về màu áo mà họ sẽ nhận. Cấu trúc này được minh họa
trong Listing 13-1:

<span class="filename">Filename: src/main.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

<span class="caption">Listing 13-1: Tình huống tặng áo của công ty</span>

`store` được định nghĩa trong `main` còn hai áo màu xanh và một áo màu đỏ
để phân phát trong chương trình khuyến mãi phiên bản giới hạn này. Chúng
ta gọi phương thức `giveaway` cho một người dùng có sở thích áo màu đỏ
và một người không có sở thích nào.

Lại một lần nữa, đoạn code này có thể được triển khai theo nhiều cách,
và ở đây, để tập trung vào closures, chúng ta chỉ sử dụng những khái niệm
bạn đã học, ngoại trừ phần thân của phương thức `giveaway` sử dụng closure.
Trong phương thức `giveaway`, chúng ta nhận sở thích của người dùng dưới
dạng tham số kiểu `Option<ShirtColor>` và gọi phương thức `unwrap_or_else`
trên `user_preference`. Phương thức [`unwrap_or_else` trên `Option<T>`][unwrap-or-else]<!-- ignore --> 
được định nghĩa bởi thư viện chuẩn. Nó nhận một đối số: một closure không
có tham số và trả về giá trị kiểu `T` (cùng kiểu được lưu trong biến thể
`Some` của `Option<T>`, trong trường hợp này là `ShirtColor`). Nếu `Option<T>`
là biến thể `Some`, `unwrap_or_else` trả về giá trị bên trong `Some`. Nếu
`Option<T>` là biến thể `None`, `unwrap_or_else` sẽ gọi closure và trả về
giá trị mà closure trả về.

Chúng ta chỉ định biểu thức closure `|| self.most_stocked()` làm đối số
cho `unwrap_or_else`. Đây là một closure không nhận tham số nào (nếu closure
có tham số, chúng sẽ xuất hiện giữa hai dấu gạch đứng). Thân của closure
gọi `self.most_stocked()`. Chúng ta định nghĩa closure ở đây, và
cài đặt của `unwrap_or_else` sẽ thực thi closure sau nếu kết quả được
cần thiết.

Chạy đoạn code này sẽ in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Một điểm thú vị ở đây là chúng ta đã truyền một closure gọi `self.most_stocked()`
trên instance `Inventory` hiện tại. Thư viện chuẩn không cần biết gì về các
kiểu `Inventory` hay `ShirtColor` mà chúng ta định nghĩa, cũng như logic mà
chúng ta muốn sử dụng trong tình huống này. Closure sẽ bắt một tham chiếu
bất biến đến instance `self` của `Inventory` và truyền nó cùng với đoạn code
chúng ta chỉ định đến phương thức `unwrap_or_else`. Ngược lại, các hàm
thường không thể bắt môi trường của chúng theo cách này.

### Suy Luận Kiểu và Ghi Chú Kiểu cho Closures

Còn có nhiều khác biệt giữa hàm và closures. Closures thường không yêu cầu
bạn phải ghi chú kiểu cho các tham số hoặc giá trị trả về như các hàm `fn`.
Ghi chú kiểu là cần thiết cho hàm vì các kiểu là một phần của giao diện
rõ ràng được cung cấp cho người dùng. Định nghĩa giao diện này một cách
cứng nhắc giúp đảm bảo mọi người đều hiểu các kiểu giá trị mà hàm sử dụng
và trả về. Closures, ngược lại, không được dùng trong một giao diện lộ ra
như vậy: chúng được lưu trong biến và sử dụng mà không cần đặt tên hay
tiếp cận bởi người dùng thư viện.

Closures thường ngắn gọn và chỉ liên quan trong một ngữ cảnh hẹp, thay vì
trong bất kỳ tình huống tùy ý nào. Trong những ngữ cảnh hạn chế này,
compiler có thể suy luận kiểu của các tham số và kiểu trả về, tương tự
như cách nó suy luận kiểu hầu hết các biến (có những trường hợp hiếm
rằng compiler cũng cần ghi chú kiểu cho closure).

Giống như với biến, chúng ta có thể thêm ghi chú kiểu nếu muốn tăng tính
rõ ràng và minh bạch, mặc dù sẽ làm code dài hơn mức cần thiết. Việc ghi
chú kiểu cho một closure sẽ trông như định nghĩa trong Listing 13-2. Trong
ví dụ này, chúng ta định nghĩa một closure và lưu nó vào biến thay vì định
nghĩa closure ngay tại chỗ truyền làm đối số như trong Listing 13-1.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

<span class="caption">Listing 13-2: Thêm ghi chú kiểu tùy chọn cho các kiểu
tham số và giá trị trả về trong closure</span>

Khi thêm ghi chú kiểu, cú pháp của closures trông giống hơn với cú pháp
của các hàm. Ở đây, chúng ta định nghĩa một hàm cộng 1 vào tham số của
nó và một closure có cùng hành vi, để so sánh. Chúng ta đã thêm một số
dấu cách để căn các phần liên quan. Điều này minh họa cách cú pháp
closure tương tự cú pháp hàm, ngoại trừ việc sử dụng các dấu ống (`| |`)
và mức độ cú pháp tùy chọn.

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Dòng đầu tiên hiển thị định nghĩa một hàm, và dòng thứ hai hiển thị định
nghĩa một closure với đầy đủ ghi chú kiểu. Ở dòng thứ ba, chúng ta loại
bỏ ghi chú kiểu khỏi định nghĩa closure. Ở dòng thứ tư, chúng ta bỏ
cặp dấu ngoặc nhọn, vốn tùy chọn vì thân closure chỉ có một biểu thức.
Tất cả các định nghĩa này đều hợp lệ và sẽ cho cùng một hành vi khi
được gọi. Các dòng `add_one_v3` và `add_one_v4` yêu cầu các closure được
đánh giá trước khi biên dịch vì các kiểu sẽ được suy luận từ cách chúng
được sử dụng. Điều này tương tự như `let v = Vec::new();` cần ghi chú
kiểu hoặc các giá trị của một kiểu nào đó được thêm vào `Vec` để Rust
có thể suy luận kiểu.

Đối với định nghĩa closure, compiler sẽ suy luận một kiểu cụ thể cho mỗi
tham số và cho giá trị trả về của chúng. Ví dụ, Listing 13-3 hiển thị
định nghĩa một closure ngắn chỉ trả về giá trị mà nó nhận làm tham số.
Closure này không hữu ích lắm ngoài mục đích minh họa. Lưu ý rằng chúng
ta không thêm bất kỳ ghi chú kiểu nào cho định nghĩa. Vì không có ghi
chú kiểu, chúng ta có thể gọi closure với bất kỳ kiểu nào, như lần đầu
chúng ta gọi với `String`. Nếu sau đó thử gọi `example_closure` với một
số nguyên, chúng ta sẽ nhận được lỗi.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

<span class="caption">Listing 13-3: Thử gọi một closure có kiểu được suy luận
với hai kiểu khác nhau</span>

Compiler sẽ đưa ra lỗi sau:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Lần đầu tiên chúng ta gọi `example_closure` với giá trị `String`, compiler
suy luận kiểu của `x` và kiểu trả về của closure là `String`. Những kiểu
này sau đó được “khóa” trong closure ở `example_closure`, và chúng ta sẽ
nhận được lỗi kiểu khi cố gắng sử dụng một kiểu khác với cùng closure.

### Bắt Tham Chiếu hoặc Chuyển Quyền Sở Hữu

Closures có thể bắt các giá trị từ môi trường của chúng theo ba cách, tương
ứng trực tiếp với ba cách một hàm có thể nhận tham số: mượn bất biến,
mượn có thể thay đổi, và nhận quyền sở hữu. Closure sẽ quyết định sử dụng
cách nào dựa trên những gì thân hàm làm với các giá trị bị bắt.

Trong Listing 13-4, chúng ta định nghĩa một closure bắt một tham chiếu
bất biến đến vector có tên `list` vì nó chỉ cần tham chiếu bất biến để in
giá trị:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

<span class="caption">Listing 13-4: Định nghĩa và gọi một closure bắt tham chiếu bất biến</span>

Ví dụ này cũng minh họa rằng một biến có thể liên kết với một định nghĩa
closure, và chúng ta có thể gọi closure sau này bằng cách sử dụng tên
biến và dấu ngoặc, như thể tên biến là tên hàm.

Vì chúng ta có thể có nhiều tham chiếu bất biến đến `list` cùng lúc,
`list` vẫn có thể truy cập được từ code trước khi định nghĩa closure,
sau khi định nghĩa closure nhưng trước khi gọi closure, và sau khi gọi
closure. Đoạn code này biên dịch được, chạy được và in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Tiếp theo, trong Listing 13-5, chúng ta thay đổi thân closure sao cho nó
thêm một phần tử vào vector `list`. Closure bây giờ bắt một tham chiếu
có thể thay đổi:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

<span class="caption">Listing 13-5: Định nghĩa và gọi một closure bắt tham chiếu có thể thay đổi</span>

Đoạn code này biên dịch được, chạy được và in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Lưu ý rằng không còn `println!` nào giữa định nghĩa và gọi closure
`borrows_mutably`: khi `borrows_mutably` được định nghĩa, nó bắt một
tham chiếu có thể thay đổi đến `list`. Chúng ta không sử dụng closure
nữa sau khi gọi, nên việc mượn có thể thay đổi kết thúc. Giữa định
nghĩa closure và gọi closure, việc mượn bất biến để in ra không được
cho phép vì khi có một tham chiếu có thể thay đổi, không có mượn nào
khác được phép. Hãy thử thêm một `println!` vào đó để xem thông báo
lỗi bạn nhận được!

Nếu bạn muốn buộc closure nhận quyền sở hữu các giá trị nó sử dụng
trong môi trường mặc dù thân closure không nhất thiết phải cần quyền
sở hữu, bạn có thể dùng từ khóa `move` trước danh sách tham số.

Kỹ thuật này chủ yếu hữu ích khi truyền một closure vào một luồng mới
để chuyển dữ liệu sao cho nó thuộc sở hữu của luồng mới. Chúng ta sẽ
thảo luận chi tiết về luồng và lý do muốn dùng chúng trong Chương 16
khi bàn về concurrency, nhưng bây giờ, hãy cùng khám phá ngắn gọn việc
khởi tạo một luồng mới sử dụng closure cần từ khóa `move`. Listing 13-6
hiển thị Listing 13-4 được chỉnh sửa để in vector trong một luồng mới
thay vì luồng chính:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

<span class="caption">Listing 13-6: Sử dụng `move` để buộc closure của luồng mới nhận quyền sở hữu `list`</span>

Chúng ta khởi tạo một luồng mới, truyền cho luồng một closure để chạy
như một đối số. Thân closure in ra `list`. Trong Listing 13-4, closure chỉ
bắt `list` bằng một tham chiếu bất biến vì đó là mức truy cập tối thiểu
cần thiết để in. Trong ví dụ này, mặc dù thân closure vẫn chỉ cần
một tham chiếu bất biến, chúng ta cần chỉ định rằng `list` nên được
chuyển vào closure bằng cách đặt từ khóa `move` ở đầu định nghĩa closure.
Luồng mới có thể kết thúc trước khi luồng chính kết thúc, hoặc luồng chính
có thể kết thúc trước. Nếu luồng chính vẫn giữ quyền sở hữu `list` nhưng
kết thúc trước luồng mới và hủy `list`, tham chiếu bất biến trong luồng mới
sẽ không còn hợp lệ. Do đó, compiler yêu cầu `list` phải được chuyển vào
closure của luồng mới để tham chiếu được hợp lệ. Hãy thử bỏ từ khóa `move`
hoặc sử dụng `list` trong luồng chính sau khi định nghĩa closure để xem
các lỗi compiler bạn nhận được!

<!-- Old headings. Do not remove or links may break. -->
<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

### Chuyển Giá Trị Bắt Được Ra Khỏi Closures và Các Trait `Fn`

Khi một closure đã bắt một tham chiếu hoặc quyền sở hữu của một giá trị
từ môi trường nơi closure được định nghĩa (ảnh hưởng đến việc có gì,
nếu có, được chuyển *vào* closure), thì đoạn code trong thân closure sẽ
xác định điều gì xảy ra với các tham chiếu hoặc giá trị khi closure được
thực thi sau này (ảnh hưởng đến việc có gì, nếu có, được chuyển *ra khỏi*
closure). Thân closure có thể thực hiện bất kỳ điều nào sau đây: chuyển
giá trị bắt được ra khỏi closure, thay đổi giá trị bắt được, không chuyển
không thay đổi giá trị, hoặc không bắt gì từ môi trường ngay từ đầu.

Cách closure bắt và xử lý các giá trị từ môi trường ảnh hưởng đến các trait
mà closure thực thi, và traits là cách các hàm và struct có thể chỉ định
loại closures mà chúng có thể sử dụng. Closures sẽ tự động triển khai một,
hai, hoặc cả ba trait `Fn` này, theo cách cộng dồn, tùy thuộc vào cách thân
closure xử lý các giá trị:

1. `FnOnce` áp dụng cho các closures chỉ có thể được gọi một lần. Tất cả
   closures đều triển khai ít nhất trait này, vì tất cả closures có thể được
   gọi. Một closure mà chuyển các giá trị bắt được ra khỏi thân của nó
   sẽ chỉ triển khai `FnOnce` và không triển khai các trait `Fn` khác,
   vì nó chỉ có thể được gọi một lần.
2. `FnMut` áp dụng cho các closures không chuyển giá trị bắt được ra khỏi
   thân, nhưng có thể thay đổi các giá trị bắt được. Những closures này
   có thể được gọi nhiều lần.
3. `Fn` áp dụng cho các closures không chuyển giá trị bắt được ra khỏi thân
   và không thay đổi các giá trị bắt được, cũng như các closures không bắt
   gì từ môi trường. Những closures này có thể được gọi nhiều lần mà không
   làm thay đổi môi trường, điều này quan trọng trong các trường hợp như
   gọi closure nhiều lần đồng thời.

Hãy xem định nghĩa của phương thức `unwrap_or_else` trên `Option<T>` mà
chúng ta đã sử dụng trong Listing 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Nhớ rằng `T` là kiểu tổng quát đại diện cho kiểu giá trị trong biến thể
`Some` của một `Option`. Kiểu `T` cũng là kiểu trả về của hàm
`unwrap_or_else`: ví dụ, code gọi `unwrap_or_else` trên một
`Option<String>` sẽ nhận về một `String`.

Tiếp theo, lưu ý rằng hàm `unwrap_or_else` có thêm tham số kiểu tổng quát
`F`. Kiểu `F` là kiểu của tham số `f`, chính là closure mà chúng ta cung
cấp khi gọi `unwrap_or_else`.

Ràng buộc trait được chỉ định trên kiểu tổng quát `F` là `FnOnce() -> T`,
có nghĩa `F` phải có thể được gọi một lần, không nhận tham số, và trả
về một `T`. Việc dùng `FnOnce` trong ràng buộc trait thể hiện hạn chế
rằng `unwrap_or_else` chỉ gọi `f` tối đa một lần. Trong thân hàm
`unwrap_or_else`, nếu `Option` là `Some`, `f` sẽ không được gọi.
Nếu `Option` là `None`, `f` sẽ được gọi một lần. Vì tất cả closures
triển khai `FnOnce`, `unwrap_or_else` chấp nhận đa dạng nhất các loại
closures và linh hoạt tối đa.

> Lưu ý: Các hàm cũng có thể triển khai cả ba trait `Fn`. Nếu việc
> chúng ta muốn làm không yêu cầu bắt giá trị từ môi trường, chúng ta
> có thể sử dụng tên hàm thay vì closure nơi cần một thứ triển khai
> một trong các trait `Fn`. Ví dụ, trên một giá trị `Option<Vec<T>>`,
> ta có thể gọi `unwrap_or_else(Vec::new)` để nhận một vector mới rỗng
> nếu giá trị là `None`.

Bây giờ, hãy xem phương thức thư viện chuẩn `sort_by_key` định nghĩa
trên các slices, để thấy sự khác biệt với `unwrap_or_else` và lý do
tại sao `sort_by_key` dùng `FnMut` thay vì `FnOnce` cho ràng buộc trait.
Closure nhận một tham số dưới dạng tham chiếu đến phần tử hiện tại
trong slice đang xét, và trả về một giá trị kiểu `K` có thể sắp xếp.
Hàm này hữu ích khi muốn sắp xếp slice theo một thuộc tính cụ thể
của mỗi phần tử. Trong Listing 13-7, chúng ta có một danh sách
các instance `Rectangle` và dùng `sort_by_key` để sắp xếp theo
thuộc tính `width` từ nhỏ đến lớn:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

<span class="caption">Listing 13-7: Sử dụng `sort_by_key` để sắp xếp các hình chữ nhật theo chiều rộng</span>

Đoạn code này in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

Lý do `sort_by_key` được định nghĩa để nhận một closure `FnMut` là vì
nó gọi closure nhiều lần: một lần cho mỗi phần tử trong slice. Closure
`|r| r.width` không bắt, thay đổi, hay chuyển ra bất kỳ giá trị nào
từ môi trường, nên nó thỏa mãn các yêu cầu của ràng buộc trait.

Ngược lại, Listing 13-8 cho thấy một ví dụ về closure chỉ triển khai
trait `FnOnce`, vì nó chuyển một giá trị ra khỏi môi trường. Compiler
sẽ không cho phép chúng ta sử dụng closure này với `sort_by_key`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

<span class="caption">Listing 13-8: Thử sử dụng một closure `FnOnce` với `sort_by_key`</span>

Đây là một cách dựng lên rườm rà và phức tạp (không hoạt động) để cố
gắng đếm số lần `sort_by_key` được gọi khi sắp xếp `list`. Đoạn code
này cố gắng đếm bằng cách đẩy `value`—một `String` từ môi trường của
closure—vào vector `sort_operations`. Closure bắt `value` rồi chuyển
`value` ra khỏi closure bằng cách chuyển quyền sở hữu của `value` vào
vector `sort_operations`. Closure này chỉ có thể được gọi một lần;
nếu cố gọi lần thứ hai sẽ không được vì `value` không còn trong
môi trường để đẩy vào `sort_operations` nữa! Do đó, closure này chỉ
triển khai `FnOnce`. Khi cố biên dịch đoạn code này, chúng ta nhận được
lỗi rằng `value` không thể được chuyển ra khỏi closure vì closure
phải triển khai `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Lỗi này chỉ ra dòng trong thân closure chuyển `value` ra khỏi môi trường.
Để khắc phục, chúng ta cần thay đổi thân closure sao cho không chuyển
giá trị ra khỏi môi trường. Để đếm số lần `sort_by_key` được gọi, việc
giữ một biến đếm trong môi trường và tăng giá trị của nó trong thân
closure là cách trực quan hơn để tính toán. Closure trong Listing 13-9
hoạt động với `sort_by_key` vì nó chỉ bắt một tham chiếu có thể thay
đổi đến biến đếm `num_sort_operations` và do đó có thể được gọi nhiều
lần:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

<span class="caption">Listing 13-9: Sử dụng closure `FnMut` với `sort_by_key` được phép</span>

Các trait `Fn` rất quan trọng khi định nghĩa hoặc sử dụng các hàm
hoặc kiểu dữ liệu có dùng closures. Trong phần tiếp theo, chúng ta sẽ
thảo luận về iterators. Nhiều phương thức của iterator nhận đối số
là closure, nên hãy nhớ các chi tiết về closure này khi tiếp tục!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
