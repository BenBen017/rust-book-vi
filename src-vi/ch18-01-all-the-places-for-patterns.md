## Tất cả các nơi patterns có thể được sử dụng

Patterns xuất hiện ở nhiều nơi trong Rust, và bạn đã sử dụng chúng khá nhiều
mà có thể không nhận ra! Phần này sẽ thảo luận tất cả các nơi mà patterns
hợp lệ.

### Nhánh (`arm`) của `match`

Như đã thảo luận trong Chương 6, chúng ta sử dụng patterns trong các nhánh
của biểu thức `match`. Về mặt hình thức, biểu thức `match` được định nghĩa
bằng từ khóa `match`, một giá trị để so khớp, và một hoặc nhiều nhánh (`match
arm`) bao gồm một pattern và một biểu thức sẽ chạy nếu giá trị khớp với
pattern của nhánh đó, ví dụ:

```text
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

Ví dụ, đây là biểu thức `match` từ Listing 6-5, khớp với một giá trị
`Option<i32>` trong biến `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Các pattern trong biểu thức `match` này là `None` và `Some(value)` đứng phía
trái của mỗi mũi tên.

Một yêu cầu đối với biểu thức `match` là chúng phải *toàn diện* (exhaustive),
tức là phải bao quát mọi khả năng của giá trị trong biểu thức `match`. Một
cách để đảm bảo rằng bạn đã bao quát tất cả khả năng là sử dụng một pattern
bắt tất cả (catchall) ở arm cuối cùng: ví dụ, một tên biến khớp với bất kỳ
giá trị nào sẽ luôn thành công và bao quát tất cả các trường hợp còn lại.

Pattern `_` sẽ khớp với bất cứ giá trị nào, nhưng không gán giá trị đó cho
biến, nên thường được dùng ở arm cuối cùng. Pattern `_` hữu ích khi bạn muốn
bỏ qua bất kỳ giá trị nào không được chỉ định. Chúng ta sẽ đi sâu hơn về
pattern `_` trong phần [“Ignoring Values in a Pattern”][ignoring-values-in-a-pattern]<!-- ignore --> ở cuối chương này.

### Biểu thức điều kiện `if let`

Trong Chương 6, chúng ta đã thảo luận cách dùng `if let` chủ yếu như một
cách viết ngắn gọn cho `match` khi chỉ quan tâm đến một trường hợp. Tùy chọn,
`if let` có thể đi kèm `else` chứa code sẽ chạy nếu pattern trong `if let` không
khớp.

Listing 18-1 cho thấy bạn có thể kết hợp `if let`, `else if` và `else if let`.
Cách làm này linh hoạt hơn `match` vì bạn có thể so sánh nhiều giá trị khác
nhau, và Rust không yêu cầu các điều kiện trong chuỗi `if let`, `else if`,
`else if let` phải liên quan đến nhau.

Ví dụ này xác định màu nền dựa trên một chuỗi kiểm tra nhiều điều kiện khác
nhau. Chúng ta sử dụng các biến với giá trị cố định, mà trong thực tế chương
trình có thể nhận từ input của người dùng.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-01/src/main.rs}}
```

<span class="caption">Listing 18-1: Kết hợp `if let`, `else if`, `else if let` và `else`</span>

Nếu người dùng chỉ định một màu yêu thích, màu đó sẽ được sử dụng làm màu nền.
Nếu không có màu yêu thích nào được chỉ định và hôm nay là thứ Ba, màu nền sẽ
là xanh lá. Ngược lại, nếu người dùng chỉ định tuổi của họ dưới dạng chuỗi và
chúng ta có thể parse thành số thành công, màu nền sẽ là tím hoặc cam tùy
thuộc vào giá trị số đó. Nếu không có điều kiện nào trong số này đúng, màu
nền sẽ là xanh dương.

Cấu trúc điều kiện này cho phép chúng ta hỗ trợ các yêu cầu phức tạp. Với các
giá trị cố định trong ví dụ này, chương trình sẽ in ra
`Using purple as the background color`.

Bạn có thể thấy rằng `if let` cũng có thể tạo ra các biến shadowed (biến che
phủ) tương tự như các nhánh trong `match`: dòng `if let Ok(age) = age` tạo
ra một biến `age` mới shadowed, chứa giá trị bên trong variant `Ok`. Điều này
có nghĩa là chúng ta cần đặt điều kiện `if age > 30` bên trong khối đó: chúng
ta không thể kết hợp hai điều kiện này thành `if let Ok(age) = age && age > 30`.
Biến `age` shadowed mà chúng ta muốn so sánh với 30 chỉ hợp lệ khi phạm vi
mới bắt đầu với dấu ngoặc nhọn.

Nhược điểm của việc sử dụng các biểu thức `if let` là trình biên dịch không kiểm tra
tính bao quát (exhaustiveness), trong khi với các biểu thức `match`, trình biên dịch
sẽ làm điều này. Nếu chúng ta bỏ qua khối `else` cuối cùng và do đó bỏ lỡ việc xử lý
một số trường hợp, trình biên dịch sẽ không cảnh báo chúng ta về lỗi logic tiềm ẩn.

### Vòng lặp điều kiện `while let`

Tương tự như `if let`, vòng lặp điều kiện `while let` cho phép một vòng lặp `while`
chạy miễn là pattern tiếp tục khớp. Trong Listing 18-2, chúng ta viết một vòng lặp
`while let` sử dụng vector như một stack và in các giá trị trong vector theo thứ tự
ngược lại so với thứ tự chúng được đẩy vào.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-02/src/main.rs:here}}
```

<span class="caption">Listing 18-2: Sử dụng vòng lặp `while let` để in các giá trị
miễn là `stack.pop()` trả về `Some`</span>

Ví dụ này sẽ in ra 3, 2, rồi 1. Phương thức `pop` lấy phần tử cuối cùng ra khỏi vector
và trả về `Some(value)`. Nếu vector trống, `pop` trả về `None`. Vòng lặp `while`
tiếp tục chạy mã trong khối của nó miễn là `pop` trả về `Some`. Khi `pop` trả về
`None`, vòng lặp dừng lại. Chúng ta có thể dùng `while let` để lấy từng phần tử
ra khỏi stack.

### Vòng lặp `for`

Trong một vòng lặp `for`, giá trị ngay sau từ khóa `for` là một pattern. Ví dụ, trong
`for x in y`, `x` là pattern. Listing 18-3 minh họa cách sử dụng pattern trong vòng lặp
`for` để phá vỡ (destructure) một tuple như một phần của vòng lặp.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-03/src/main.rs:here}}
```

<span class="caption">Listing 18-3: Sử dụng pattern trong vòng lặp `for` để
phá vỡ (destructure) một tuple</span>

Mã trong Listing 18-3 sẽ in ra kết quả sau:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-03/output.txt}}
```

Chúng ta điều chỉnh một iterator bằng cách sử dụng phương thức `enumerate` 
để nó trả về một giá trị và chỉ số của giá trị đó, đặt vào trong một tuple. 
Giá trị đầu tiên được tạo ra là tuple `(0, 'a')`. Khi giá trị này được so khớp 
với pattern `(index, value)`, `index` sẽ là `0` và `value` sẽ là `'a'`, 
in ra dòng đầu tiên của output.

### Câu lệnh `let`

Trước chương này, chúng ta chỉ mới thảo luận rõ ràng về việc sử dụng pattern với 
`match` và `if let`, nhưng thực ra, chúng ta đã sử dụng pattern ở nhiều nơi khác 
nữa, bao gồm cả trong các câu lệnh `let`. Ví dụ, xem xét một phép gán biến đơn giản 
với `let`:

```rust
let x = 5;
```

Mỗi khi bạn sử dụng một câu lệnh `let` như thế này, bạn đã đang sử dụng pattern, 
mặc dù có thể bạn chưa nhận ra! Nói một cách chính thức hơn, một câu lệnh `let` 
trông như sau:

```text
let PATTERN = EXPRESSION;
```

Trong các câu lệnh như `let x = 5;` với tên biến ở vị trí `PATTERN`, 
tên biến chỉ là một dạng đơn giản đặc biệt của pattern. Rust so sánh biểu thức với pattern và gán các tên mà nó tìm thấy. 
Vì vậy, trong ví dụ `let x = 5;`, `x` là một pattern có nghĩa là “ràng buộc giá trị khớp ở đây vào biến `x`.” 
Bởi vì tên `x` là toàn bộ pattern, pattern này về cơ bản có nghĩa là “ràng buộc mọi giá trị vào biến `x`, bất kể giá trị đó là gì.”

Để thấy rõ hơn khía cạnh pattern matching của `let`, hãy xem Listing 18-4, 
trong đó sử dụng một pattern với `let` để phân rã (destructure) một tuple.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-04/src/main.rs:here}}
```

<span class="caption">Listing 18-4: Sử dụng pattern để phân rã một tuple và
tạo ba biến cùng lúc</span>

Ở đây, chúng ta so khớp một tuple với một pattern. Rust so sánh giá trị `(1, 2, 3)`
với pattern `(x, y, z)` và nhận thấy giá trị khớp với pattern, vì vậy Rust
ràng buộc `1` cho `x`, `2` cho `y`, và `3` cho `z`. Bạn có thể coi pattern tuple này
như là việc lồng ba pattern biến riêng lẻ bên trong nó.

Nếu số lượng phần tử trong pattern không khớp với số lượng phần tử
trong tuple, kiểu tổng thể sẽ không khớp và chúng ta sẽ nhận được lỗi biên dịch. 
Ví dụ, Listing 18-5 cho thấy một thử nghiệm phân rã một tuple có ba phần tử 
thành hai biến, điều này sẽ không hoạt động.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-05/src/main.rs:here}}
```

<span class="caption">Listing 18-5: Tạo pattern không đúng khi số biến
không khớp với số phần tử trong tuple</span>

Việc cố gắng biên dịch đoạn code này sẽ dẫn đến lỗi kiểu sau:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-05/output.txt}}
```

Để sửa lỗi, chúng ta có thể bỏ qua một hoặc nhiều giá trị trong tuple bằng cách sử dụng `_` hoặc `..`, như bạn sẽ thấy trong phần [“Ignoring Values in a Pattern”][ignoring-values-in-a-pattern]<!-- ignore -->. Nếu vấn đề là có quá nhiều biến trong pattern, giải pháp là điều chỉnh cho khớp kiểu bằng cách loại bỏ bớt các biến sao cho số biến bằng số phần tử trong tuple.

### Tham số Hàm

Tham số hàm cũng có thể là pattern. Đoạn code trong Listing 18-6, khai báo một hàm tên là `foo` với một tham số tên là `x` kiểu `i32`, chắc hẳn giờ đây đã trở nên quen thuộc.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-06/src/main.rs:here}}
```

<span class="caption">Listing 18-6: Chữ ký hàm sử dụng pattern trong các tham số</span>

Phần `x` chính là một pattern! Giống như với `let`, chúng ta có thể so khớp một tuple trong các tham số của hàm với pattern. Listing 18-7 tách các giá trị trong một tuple khi truyền nó vào hàm.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-07/src/main.rs}}
```

<span class="caption">Listing 18-7: Một hàm với các tham số tách một tuple</span>

Đoạn mã này sẽ in ra `Current location: (3, 5)`. Các giá trị `&(3, 5)` khớp với pattern `&(x, y)`, vì vậy `x` là giá trị `3` và `y` là giá trị `5`.

Chúng ta cũng có thể sử dụng pattern trong danh sách tham số của closure giống như trong danh sách tham số của hàm, bởi vì closures tương tự như các hàm, như đã thảo luận ở Chương 13.

Tại thời điểm này, bạn đã thấy nhiều cách sử dụng pattern, nhưng pattern không hoạt động giống nhau ở mọi nơi mà chúng ta có thể sử dụng chúng. Ở một số nơi, pattern phải là không thể thất bại (irrefutable); trong các trường hợp khác, chúng có thể thất bại (refutable). Chúng ta sẽ thảo luận hai khái niệm này tiếp theo.

[ignoring-values-in-a-pattern]:
ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern
