## Sử dụng `Box<T>` để trỏ đến dữ liệu trên Heap

Smart pointer đơn giản nhất là *box*, với kiểu được viết là `Box<T>`. Boxes
cho phép bạn lưu trữ dữ liệu trên heap thay vì trên stack. Trên stack chỉ còn
pointer trỏ đến dữ liệu trên heap. Tham khảo Chương 4 để ôn lại sự khác biệt
giữa stack và heap.

Boxes không có chi phí hiệu năng nào, ngoài việc lưu dữ liệu trên heap thay vì
trên stack. Tuy nhiên, chúng cũng không có nhiều khả năng bổ sung. Bạn sẽ
sử dụng chúng thường xuyên nhất trong các trường hợp sau:

* Khi bạn có một kiểu mà kích thước không thể biết tại thời điểm biên dịch và
  bạn muốn sử dụng giá trị của kiểu đó trong một ngữ cảnh yêu cầu kích thước
  chính xác
* Khi bạn có một lượng dữ liệu lớn và muốn chuyển quyền sở hữu nhưng muốn
  đảm bảo dữ liệu không bị sao chép khi làm vậy
* Khi bạn muốn sở hữu một giá trị và bạn chỉ quan tâm rằng nó là một kiểu
  triển khai một trait cụ thể, thay vì phải là một kiểu cụ thể

Chúng ta sẽ minh họa tình huống đầu tiên trong phần [“Enabling Recursive Types
with Boxes”](#enabling-recursive-types-with-boxes). Trong trường hợp thứ hai,
việc chuyển quyền sở hữu một lượng lớn dữ liệu có thể mất nhiều thời gian
bởi vì dữ liệu bị sao chép trên stack. Để cải thiện hiệu năng trong trường
hợp này, chúng ta có thể lưu lượng dữ liệu lớn trên heap trong một box.
Khi đó, chỉ một lượng nhỏ dữ liệu pointer được sao chép trên stack, trong khi
dữ liệu mà nó tham chiếu vẫn nằm nguyên vị trí trên heap. Trường hợp thứ ba
được gọi là *trait object*, và Chương 17 dành hẳn một phần, [“Using Trait
Objects That Allow for Values of Different Types,”][trait-objects], để thảo luận
về chủ đề này. Vì vậy những gì bạn học ở đây sẽ được áp dụng lại ở Chương 17!

### Sử dụng `Box<T>` để lưu dữ liệu trên Heap

Trước khi thảo luận về trường hợp lưu trữ trên heap cho `Box<T>`, chúng ta sẽ
xem cú pháp và cách tương tác với các giá trị được lưu trữ bên trong `Box<T>`.

Listing 15-1 minh họa cách sử dụng box để lưu một giá trị `i32` trên heap:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

<span class="caption">Listing 15-1: Lưu một giá trị `i32` trên heap bằng box</span>

Chúng ta định nghĩa biến `b` là một `Box` trỏ đến giá trị `5`, được cấp phát
trên heap. Chương trình này sẽ in ra `b = 5`; trong trường hợp này, chúng ta
có thể truy cập dữ liệu trong box tương tự như khi dữ liệu nằm trên stack. Giống
như bất kỳ giá trị nào mà chúng ta sở hữu, khi một box ra khỏi phạm vi, như
`b` ở cuối `main`, nó sẽ được giải phóng bộ nhớ. Việc giải phóng xảy ra cả với
box (lưu trên stack) và dữ liệu mà nó trỏ đến (lưu trên heap).

Việc đặt một giá trị đơn lẻ trên heap không thực sự hữu ích, vì vậy bạn sẽ ít
khi dùng boxes một mình theo cách này. Việc lưu giá trị như một `i32` trên stack,
nơi chúng được lưu mặc định, là phù hợp hơn trong hầu hết các trường hợp. Hãy
xem một ví dụ mà boxes cho phép chúng ta định nghĩa các kiểu mà nếu không có
boxes thì sẽ không làm được.

### Cho phép kiểu đệ quy với Boxes

Một giá trị của *kiểu đệ quy* có thể chứa một giá trị khác cùng kiểu bên trong
nó. Kiểu đệ quy gặp vấn đề vì tại thời điểm biên dịch Rust cần biết một kiểu
chiếm bao nhiêu không gian. Tuy nhiên, việc lồng các giá trị kiểu đệ quy về lý
thuyết có thể tiếp tục vô hạn, vì vậy Rust không thể biết chính xác kích thước
cần thiết. Vì boxes có kích thước xác định, chúng ta có thể cho phép kiểu đệ
quy bằng cách chèn một box trong định nghĩa kiểu đệ quy.

Lấy ví dụ về một kiểu đệ quy, hãy xem xét *cons list*. Đây là một kiểu dữ liệu
thường thấy trong các ngôn ngữ lập trình hàm. Kiểu cons list chúng ta sẽ định
nghĩa khá đơn giản, ngoại trừ phần đệ quy; do đó, các khái niệm trong ví dụ này
sẽ hữu ích mỗi khi bạn gặp các tình huống phức tạp hơn liên quan đến kiểu đệ
quy.

#### Thông tin thêm về Cons List

*Cons list* là một cấu trúc dữ liệu xuất phát từ ngôn ngữ lập trình Lisp và các
biến thể của nó, được tạo từ các cặp lồng nhau, và là phiên bản Lisp của
linked list. Tên của nó xuất phát từ hàm `cons` (viết tắt của “construct
function”) trong Lisp, dùng để tạo một cặp mới từ hai đối số. Bằng cách gọi
`cons` trên một cặp gồm một giá trị và một cặp khác, chúng ta có thể tạo
cons lists gồm các cặp đệ quy.

Ví dụ, dưới đây là biểu diễn giả mã của một cons list chứa danh sách 1, 2, 3
với mỗi cặp được đặt trong dấu ngoặc:

```text
(1, (2, (3, Nil)))
```

Mỗi phần tử trong cons list chứa hai thành phần: giá trị của phần tử hiện tại
và phần tử tiếp theo. Phần tử cuối cùng trong danh sách chỉ chứa một giá trị
gọi là `Nil` mà không có phần tử tiếp theo. Một cons list được tạo ra bằng cách
gọi đệ quy hàm `cons`. Tên chuẩn để chỉ trường hợp cơ sở của đệ quy là `Nil`.
Lưu ý rằng đây không phải là cùng khái niệm “null” hay “nil” trong Chương 6,
mà là một giá trị không hợp lệ hoặc vắng mặt.

Cons list không phải là một cấu trúc dữ liệu thường được sử dụng trong Rust.
Hầu hết thời gian khi bạn cần một danh sách các phần tử trong Rust, `Vec<T>`
là lựa chọn tốt hơn. Các kiểu dữ liệu đệ quy phức tạp khác *có* thể hữu ích
trong nhiều tình huống, nhưng bắt đầu với cons list trong chương này giúp
chúng ta khám phá cách boxes cho phép định nghĩa kiểu đệ quy mà không bị
xao nhãng quá nhiều.

Listing 15-2 chứa định nghĩa enum cho một cons list. Lưu ý rằng mã này sẽ
chưa biên dịch được vì kiểu `List` chưa có kích thước xác định, điều này
chúng ta sẽ minh họa tiếp theo.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

<span class="caption">Listing 15-2: Lần thử đầu tiên định nghĩa enum để
biểu diễn cấu trúc dữ liệu cons list chứa các giá trị `i32`</span>

> Lưu ý: Chúng ta đang triển khai một cons list chỉ chứa các giá trị `i32`
> cho ví dụ này. Chúng ta hoàn toàn có thể dùng generics, như đã thảo luận
> trong Chương 10, để định nghĩa một cons list có thể lưu trữ giá trị của
> bất kỳ kiểu nào.

Sử dụng kiểu `List` để lưu danh sách `1, 2, 3` sẽ trông như mã trong
Listing 15-3:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

<span class="caption">Listing 15-3: Sử dụng enum `List` để lưu danh sách `1,
2, 3`</span>

Giá trị `Cons` đầu tiên chứa `1` và một giá trị `List` khác. Giá trị `List`
này là một giá trị `Cons` khác chứa `2` và một giá trị `List` khác nữa. Giá
trị `List` này là một `Cons` nữa chứa `3` và một giá trị `List`, cuối cùng là
`Nil`, biến thể không đệ quy đánh dấu kết thúc của danh sách.

Nếu chúng ta cố gắng biên dịch mã trong Listing 15-3, chúng ta sẽ nhận được
lỗi như trong Listing 15-4:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

<span class="caption">Listing 15-4: Lỗi nhận được khi cố gắng định nghĩa
một enum đệ quy</span>

Lỗi này cho thấy kiểu này “có kích thước vô hạn”. Nguyên nhân là chúng ta đã
định nghĩa `List` với một biến thể đệ quy: nó trực tiếp chứa một giá trị của
chính nó. Do đó, Rust không thể xác định cần bao nhiêu không gian để lưu một
giá trị `List`. Hãy phân tích lý do chúng ta nhận được lỗi này. Trước tiên, ta
xem cách Rust tính toán kích thước cần thiết để lưu một giá trị của kiểu không
đệ quy.

#### Tính Kích Thước của Kiểu Không Đệ Quy

Nhớ lại enum `Message` mà chúng ta đã định nghĩa trong Listing 6-2 khi bàn
về định nghĩa enum trong Chương 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Để xác định bao nhiêu không gian cần cấp phát cho một giá trị `Message`, Rust
sẽ xem qua từng biến thể để xem biến thể nào cần nhiều không gian nhất. Rust
thấy rằng `Message::Quit` không cần không gian, `Message::Move` cần đủ không
gian để lưu hai giá trị `i32`, v.v. Vì chỉ có một biến thể được sử dụng tại
một thời điểm, nên không gian tối đa mà một giá trị `Message` cần chính là
không gian cần để lưu biến thể lớn nhất.

So sánh với trường hợp khi Rust cố xác định kích thước cần thiết cho một
kiểu đệ quy như enum `List` trong Listing 15-2. Trình biên dịch bắt đầu bằng
việc xem biến thể `Cons`, chứa một giá trị kiểu `i32` và một giá trị kiểu
`List`. Do đó, `Cons` cần một lượng không gian bằng kích thước của `i32` cộng
với kích thước của `List`. Để tính kích thước của kiểu `List`, trình biên
dịch nhìn vào các biến thể, bắt đầu với `Cons`. Biến thể `Cons` chứa một
giá trị kiểu `i32` và một giá trị kiểu `List`, và quá trình này tiếp tục
vô hạn, như minh họa trong Hình 15-1.

<img alt="Danh sách Cons vô hạn" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 15-1: Một `List` vô hạn gồm các biến thể `Cons`
vô hạn</span>

#### Sử dụng `Box<T>` để có kiểu đệ quy với kích thước đã biết

Vì Rust không thể xác định bao nhiêu không gian cần cấp phát cho các kiểu
được định nghĩa đệ quy, trình biên dịch sẽ đưa ra lỗi kèm theo gợi ý hữu ích:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to make `List` representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

Trong gợi ý này, “điều hướng gián tiếp” (indirection) có nghĩa là thay vì lưu
một giá trị trực tiếp, chúng ta nên thay đổi cấu trúc dữ liệu để lưu giá trị
một cách gián tiếp bằng cách lưu một con trỏ tới giá trị đó.

Vì `Box<T>` là một con trỏ, Rust luôn biết kích thước của một `Box<T>` cần
bao nhiêu: kích thước của con trỏ không thay đổi dựa trên lượng dữ liệu mà nó
trỏ tới. Điều này có nghĩa là chúng ta có thể đặt một `Box<T>` bên trong
biến thể `Cons` thay vì một giá trị `List` khác trực tiếp. `Box<T>` sẽ trỏ
tới giá trị `List` tiếp theo, giá trị này sẽ nằm trên heap thay vì bên trong
biến thể `Cons`. Về mặt khái niệm, chúng ta vẫn có một danh sách, được tạo ra
bằng cách các danh sách chứa các danh sách khác, nhưng việc triển khai này
bây giờ giống như đặt các phần tử cạnh nhau thay vì bên trong nhau.

Chúng ta có thể thay đổi định nghĩa của enum `List` trong Listing 15-2 và
cách sử dụng `List` trong Listing 15-3 sang mã trong Listing 15-5, mã này sẽ
biên dịch được:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

<span class="caption">Listing 15-5: Định nghĩa `List` sử dụng `Box<T>` để có kích thước xác định</span>

Biến thể `Cons` cần kích thước của một `i32` cộng với không gian để lưu con trỏ của box. Biến thể `Nil` không lưu trữ giá trị nào, vì vậy nó cần ít không gian hơn biến thể `Cons`. Bây giờ chúng ta biết rằng bất kỳ giá trị `List` nào sẽ chiếm kích thước của một `i32` cộng với kích thước của con trỏ trong box. Bằng cách sử dụng box, chúng ta đã phá vỡ chuỗi đệ quy vô tận, vì vậy compiler có thể tính được kích thước cần thiết để lưu một giá trị `List`. Hình 15-2 minh họa biến thể `Cons` hiện nay trông như thế nào.

<img alt="A finite Cons list" src="img/trpl15-02.svg" class="center" />

<span class="caption">Hình 15-2: Một `List` không vô tận vì `Cons` chứa một `Box`</span>

Box chỉ cung cấp cơ chế gián tiếp và cấp phát trên heap; nó không có bất kỳ khả năng đặc biệt nào khác, như những gì chúng ta sẽ thấy với các loại smart pointer khác. Nó cũng không gây overhead về hiệu năng mà các khả năng đặc biệt đó mang lại, vì vậy nó có thể hữu ích trong các trường hợp như cons list, nơi cơ chế gián tiếp là tính năng duy nhất mà chúng ta cần. Chúng ta cũng sẽ xem thêm các trường hợp sử dụng box trong Chương 17.

Kiểu `Box<T>` là một smart pointer vì nó triển khai trait `Deref`, cho phép giá trị `Box<T>` được sử dụng như tham chiếu. Khi một giá trị `Box<T>` ra khỏi phạm vi, dữ liệu trên heap mà box trỏ tới cũng được giải phóng nhờ triển khai trait `Drop`. Hai trait này sẽ còn quan trọng hơn nữa đối với chức năng mà các loại smart pointer khác cung cấp mà chúng ta sẽ thảo luận trong phần còn lại của chương. Hãy cùng khám phá hai trait này chi tiết hơn.

[trait-objects]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
