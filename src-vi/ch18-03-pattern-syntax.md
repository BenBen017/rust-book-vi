## Cú pháp Pattern

Trong phần này, chúng ta sẽ tập hợp tất cả các cú pháp hợp lệ trong patterns 
và thảo luận lý do cũng như khi nào bạn muốn sử dụng từng loại.

### Khớp với Literal

Như bạn đã thấy trong Chương 6, bạn có thể so khớp patterns trực tiếp với các literal. Ví dụ code sau minh họa:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Code này sẽ in ra `one` vì giá trị trong `x` là 1. Cú pháp này hữu ích khi bạn muốn chương trình thực hiện một hành động nào đó nếu nhận được một giá trị cụ thể.

### Khớp với Biến Đặt Tên

Các biến đặt tên là các pattern không thể thất bại (irrefutable patterns) 
và khớp với bất kỳ giá trị nào; chúng ta đã sử dụng chúng nhiều lần trong sách. 
Tuy nhiên, có một vấn đề nhỏ khi sử dụng các biến đặt tên trong các biểu thức `match`. 
Vì `match` tạo ra một scope mới, các biến được khai báo như một phần của pattern 
bên trong `match` sẽ che khuất (shadow) các biến cùng tên bên ngoài `match`, 
giống như cách tất cả các biến khác hoạt động. Trong Listing 18-11, 
chúng ta khai báo một biến tên là `x` với giá trị `Some(5)` và một biến `y` với giá trị `10`. 
Sau đó, chúng ta tạo một biểu thức `match` trên giá trị `x`. 
Hãy quan sát các pattern trong các nhánh của `match` và lệnh `println!` ở cuối, 
và thử đoán xem code sẽ in gì trước khi chạy hoặc đọc tiếp.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-11/src/main.rs:here}}
```

<span class="caption">Listing 18-11: Một biểu thức `match` với một nhánh
giới thiệu biến bị shadow `y`</span>

Hãy cùng xem xét điều gì xảy ra khi biểu thức `match` chạy. 
Pattern trong nhánh `match` đầu tiên không khớp với giá trị đã định nghĩa của `x`, 
vì vậy chương trình tiếp tục.

Pattern trong nhánh `match` thứ hai giới thiệu một biến mới tên là `y` 
sẽ khớp với bất kỳ giá trị nào bên trong `Some`. Vì chúng ta đang ở 
trong một scope mới bên trong biểu thức `match`, đây là một biến `y` mới, 
không phải `y` mà chúng ta đã khai báo lúc đầu với giá trị 10. Binding `y` 
mới này sẽ khớp với bất kỳ giá trị nào bên trong `Some`, chính xác là giá trị trong `x`. 
Do đó, `y` mới này được gán giá trị bên trong `Some` trong `x`. Giá trị đó là `5`, 
vì vậy biểu thức của nhánh này sẽ chạy và in ra `Matched, y = 5`.

Nếu `x` là `None` thay vì `Some(5)`, các pattern trong hai nhánh đầu tiên sẽ không khớp, 
vì vậy giá trị sẽ khớp với dấu gạch dưới `_`. Chúng ta không giới thiệu biến `x` 
trong pattern của nhánh `_`, vì vậy `x` trong biểu thức vẫn là `x` bên ngoài, chưa bị shadow. 
Trong trường hợp giả định này, `match` sẽ in ra `Default case, x = None`.

Khi biểu thức `match` kết thúc, scope của nó kết thúc, và scope của `y` bên trong cũng kết thúc. 
Lệnh `println!` cuối cùng sẽ in ra `at the end: x = Some(5), y = 10`.

Để tạo một biểu thức `match` so sánh giá trị của `x` và `y` bên ngoài, 
thay vì giới thiệu một biến bị shadow, chúng ta sẽ cần sử dụng một match guard điều kiện. 
Chúng ta sẽ nói về match guards sau trong phần [“Extra Conditionals with Match Guards”](#extra-conditionals-with-match-guards).

### Nhiều Pattern

Trong biểu thức `match`, bạn có thể khớp nhiều pattern bằng cú pháp `|`, 
là toán tử *hoặc* cho pattern. Ví dụ, trong đoạn code sau chúng ta khớp giá trị của `x` 
với các nhánh `match`, trong đó nhánh đầu tiên có một tùy chọn *hoặc*, nghĩa là nếu giá trị 
của `x` khớp với bất kỳ giá trị nào trong nhánh đó, code của nhánh đó sẽ chạy:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Đoạn code này sẽ in ra `one or two`.

### Khớp với một phạm vi giá trị bằng `..=`

Cú pháp `..=` cho phép chúng ta khớp với một phạm vi giá trị bao gồm cả hai đầu. 
Trong đoạn code sau, khi một pattern khớp với bất kỳ giá trị nào trong phạm vi đã cho, nhánh đó sẽ được thực thi:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Nếu `x` là 1, 2, 3, 4 hoặc 5, nhánh đầu tiên sẽ khớp. Cú pháp này tiện lợi 
hơn khi muốn khớp nhiều giá trị so với việc dùng toán tử `|` để diễn tả cùng một ý tưởng;
 nếu dùng `|` thì phải viết `1 | 2 | 3 | 4 | 5`. Việc chỉ định một phạm vi ngắn gọn hơn nhiều,
  đặc biệt nếu muốn khớp, ví dụ, bất kỳ số nào từ 1 đến 1.000!

Trình biên dịch kiểm tra rằng phạm vi không rỗng tại thời điểm biên dịch, 
và vì chỉ có các kiểu `char` và số mà Rust có thể xác định phạm vi có rỗng hay không, 
nên phạm vi chỉ được phép với các giá trị số hoặc `char`.

Dưới đây là một ví dụ sử dụng phạm vi của các giá trị `char`:


```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust có thể xác định rằng `'c'` nằm trong phạm vi của nhánh đầu tiên và in ra `early ASCII letter`.

### Phá cấu trúc để tách các giá trị

Chúng ta cũng có thể dùng pattern để phá cấu trúc các struct, 
enum và tuple để sử dụng các phần khác nhau của các giá trị này. Hãy xem xét từng loại giá trị.

#### Phá cấu trúc Struct

Listing 18-12 minh họa một struct `Point` với hai trường `x` và `y`, mà chúng ta có thể tách ra bằng pattern trong một câu lệnh `let`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-12/src/main.rs}}
```

<span class="caption">Listing 18-12: Phá cấu trúc các trường của struct thành các biến riêng biệt</span>

Đoạn code này tạo ra các biến `a` và `b` tương ứng với giá trị của các trường 
`x` và `y` trong struct `p`. Ví dụ này cho thấy tên của các biến trong pattern
 không nhất thiết phải trùng với tên trường của struct. Tuy nhiên, thường thì 
 chúng ta đặt tên biến giống tên trường để dễ nhớ biến nào đến từ trường nào.

Do thói quen phổ biến này và vì viết `let Point { x: x, y: y } = p;` gây trùng lặp nhiều, 
Rust có cú pháp rút gọn cho các pattern khớp với các trường struct: 
bạn chỉ cần liệt kê tên trường, và các biến được tạo ra từ pattern sẽ có cùng tên. 
Listing 18-13 hoạt động giống như Listing 18-12, nhưng các biến được tạo ra 
trong pattern `let` là `x` và `y` thay vì `a` và `b`.


<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-13/src/main.rs}}
```

<span class="caption">Listing 18-13: Phá cấu trúc các trường struct sử dụng cú pháp rút gọn cho trường</span>

Đoạn code này tạo ra các biến `x` và `y` tương ứng với các trường `x` và `y` của biến `p`. 
Kết quả là các biến `x` và `y` chứa giá trị từ struct `p`.

Chúng ta cũng có thể phá cấu trúc với các giá trị literal như một phần của 
pattern struct thay vì tạo biến cho tất cả các trường. Cách này cho phép kiểm tra
 một số trường có giá trị cụ thể hay không trong khi tạo biến để phá cấu trúc các trường còn lại.

Trong Listing 18-14, chúng ta có một biểu thức `match` phân loại các 
giá trị `Point` thành ba trường hợp: các điểm nằm trực tiếp trên trục `x` (đúng khi `y = 0`), trên trục `y` (`x = 0`), 
hoặc không nằm trên cả hai trục.


<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-14/src/main.rs:here}}
```

<span class="caption">Listing 18-14: Phá cấu trúc và so khớp giá trị literal trong một pattern</span>

Arm đầu tiên sẽ khớp với bất kỳ điểm nào nằm trên trục `x` 
bằng cách chỉ định rằng trường `y` khớp nếu giá trị của nó bằng literal `0`. 
Pattern vẫn tạo ra một biến `x` mà chúng ta có thể sử dụng trong code của arm này.

Tương tự, arm thứ hai khớp với bất kỳ điểm nào nằm trên trục `y` bằng cách 
chỉ định rằng trường `x` khớp nếu giá trị của nó bằng `0` và tạo biến `y` 
cho giá trị của trường `y`. Arm thứ ba không chỉ định bất kỳ literal nào, 
nên nó khớp với mọi `Point` khác và tạo biến cho cả trường `x` và `y`.

Trong ví dụ này, giá trị `p` khớp với arm thứ hai vì `x` chứa giá trị 0,
 do đó code sẽ in ra `On the y axis at 7`.

Hãy nhớ rằng một biểu thức `match` sẽ dừng việc kiểm tra các arm khi 
đã tìm thấy pattern khớp đầu tiên, nên mặc dù `Point { x: 0, y: 0 }` 
nằm trên cả trục `x` và trục `y`, code này chỉ in ra `On the x axis at 0`.

#### Phá cấu trúc Enum

Chúng ta đã phá cấu trúc các enum trong sách này (ví dụ, Listing 6-5 trong Chương 6), 
nhưng chưa bàn rõ ràng rằng pattern để phá cấu trúc một enum tương ứng với cách dữ liệu 
bên trong enum được định nghĩa. Ví dụ, trong Listing 18-15 chúng ta dùng enum `Message` 
từ Listing 6-2 và viết một `match` với các pattern để phá cấu trúc từng giá trị bên trong.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-15/src/main.rs}}
```

<span class="caption">Listing 18-15: Phá cấu trúc các biến thể enum chứa các loại giá trị khác nhau</span>

Code này sẽ in ra `Change the color to red 0, green 160, and blue 255`. 
Hãy thử thay đổi giá trị của `msg` để xem code từ các arm khác chạy.

Đối với các biến thể enum không chứa dữ liệu, như `Message::Quit`, 
chúng ta không thể phá cấu trúc thêm. Chúng ta chỉ có thể so khớp 
với giá trị literal `Message::Quit`, và không có biến nào trong pattern này.

Đối với các biến thể enum giống struct, như `Message::Move`, 
chúng ta có thể dùng pattern tương tự như pattern dùng để so khớp struct. 
Sau tên biến thể, đặt dấu ngoặc nhọn và liệt kê các trường với biến 
để tách các phần ra dùng trong code của arm này. 
Ở đây chúng ta dùng dạng shorthand như trong Listing 18-13.

Đối với các biến thể enum giống tuple, như `Message::Write` chứa tuple 
với một phần tử và `Message::ChangeColor` chứa tuple với ba phần tử, 
pattern cũng tương tự như pattern dùng để so khớp tuple. Số biến 
trong pattern phải khớp với số phần tử trong biến thể mà chúng ta đang so khớp.

#### Phá cấu trúc Struct và Enum lồng nhau

Cho đến nay, các ví dụ của chúng ta đều so khớp struct hoặc enum một cấp, 
nhưng matching cũng có thể làm việc với các mục lồng nhau! Ví dụ, chúng ta 
có thể tái cấu trúc code trong Listing 18-15 để hỗ trợ màu RGB và HSV 
trong thông điệp `ChangeColor`, như được trình bày trong Listing 18-16.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-16/src/main.rs}}
```

<span class="caption">Listing 18-16: So khớp với enum lồng nhau</span>

Pattern của arm đầu tiên trong biểu thức `match` so khớp với biến thể
 enum `Message::ChangeColor` chứa một biến thể `Color::Rgb`; 
 sau đó pattern liên kết với ba giá trị `i32` bên trong. 
 Pattern của arm thứ hai cũng so khớp với biến thể enum `Message::ChangeColor`, 
 nhưng enum bên trong so khớp với `Color::Hsv` thay vào đó. 
 Chúng ta có thể chỉ định các điều kiện phức tạp này trong một biểu thức `match`, mặc dù liên quan đến hai enum.

#### Phá cấu trúc Struct và Tuple

Chúng ta có thể kết hợp, so khớp, và lồng các pattern destructuring 
theo những cách phức tạp hơn. Ví dụ sau đây minh họa một destructure phức tạp, 
nơi chúng ta lồng struct và tuple bên trong một tuple 
và phá cấu trúc tất cả các giá trị nguyên thủy ra:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Mã này cho phép chúng ta tách các kiểu dữ liệu phức tạp thành các phần thành phần để có thể sử dụng các giá trị mà chúng ta quan tâm một cách riêng biệt.

Phá cấu trúc với các pattern là một cách tiện lợi để sử dụng từng phần của giá trị, chẳng hạn như giá trị từ mỗi trường trong một struct, một cách tách biệt với nhau.

### Bỏ qua giá trị trong một pattern

Bạn đã thấy rằng đôi khi rất hữu ích khi bỏ qua các giá trị trong một pattern, ví dụ như trong arm cuối cùng của `match`, để có một catchall không thực sự làm gì nhưng vẫn bao quát tất cả các giá trị còn lại. Có vài cách để bỏ qua toàn bộ giá trị hoặc một phần giá trị trong một pattern: sử dụng pattern `_` (như bạn đã thấy), sử dụng pattern `_` bên trong một pattern khác, sử dụng tên bắt đầu bằng dấu gạch dưới, hoặc dùng `..` để bỏ qua các phần còn lại của giá trị. Hãy cùng tìm hiểu cách và lý do sử dụng từng pattern này.

#### Bỏ qua toàn bộ giá trị với `_`

Chúng ta đã sử dụng dấu gạch dưới như một wildcard pattern sẽ so khớp với bất kỳ giá trị nào nhưng không liên kết với giá trị đó. Điều này đặc biệt hữu ích khi là arm cuối cùng trong biểu thức `match`, nhưng chúng ta cũng có thể sử dụng nó trong bất kỳ pattern nào, bao gồm cả tham số của hàm, như được minh họa trong Listing 18-17.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-17/src/main.rs}}
```

<span class="caption">Listing 18-17: Sử dụng `_` trong chữ ký hàm</span>

Mã này sẽ hoàn toàn bỏ qua giá trị `3` được truyền vào làm đối số đầu tiên, và sẽ in ra `This code only uses the y parameter: 4`.

Trong hầu hết các trường hợp khi bạn không còn cần một tham số hàm cụ thể nào đó, bạn sẽ thay đổi chữ ký hàm để không bao gồm tham số không sử dụng. Việc bỏ qua một tham số hàm đặc biệt hữu ích trong các trường hợp, ví dụ, khi bạn đang triển khai một trait và cần một chữ ký kiểu nhất định nhưng phần thân hàm trong triển khai của bạn không cần một trong các tham số. Khi đó, bạn tránh được cảnh báo từ trình biên dịch về tham số hàm không được sử dụng, như sẽ xảy ra nếu bạn đặt tên cho tham số đó.

#### Bỏ qua một phần giá trị với `_` lồng nhau

Chúng ta cũng có thể sử dụng `_` bên trong một pattern khác để chỉ bỏ qua một phần của giá trị, ví dụ, khi chúng ta muốn kiểm tra chỉ một phần của giá trị nhưng không cần sử dụng các phần khác trong mã tương ứng mà chúng ta muốn chạy. Listing 18-18 cho thấy mã chịu trách nhiệm quản lý giá trị của một cài đặt. Yêu cầu nghiệp vụ là người dùng không được phép ghi đè một tùy chỉnh cài đặt đã tồn tại nhưng có thể hủy cài đặt đó và đặt giá trị nếu cài đặt hiện đang chưa được thiết lập.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-18/src/main.rs:here}}
```

<span class="caption">Listing 18-18: Sử dụng dấu gạch dưới `_` trong các pattern
khớp với các biến thể `Some` khi chúng ta không cần dùng giá trị bên trong `Some`</span>

Mã này sẽ in ra `Can't overwrite an existing customized value` và sau đó là `setting is Some(5)`. Trong nhánh match đầu tiên, chúng ta không cần khớp hoặc sử dụng các giá trị bên trong bất kỳ biến thể `Some` nào, nhưng chúng ta cần kiểm tra trường hợp khi `setting_value` và `new_setting_value` là biến thể `Some`. Trong trường hợp đó, chúng ta in lý do không thay đổi `setting_value`, và giá trị này sẽ không bị thay đổi.

Trong tất cả các trường hợp khác (nếu một trong `setting_value` hoặc `new_setting_value` là `None`) được biểu diễn bằng pattern `_` trong nhánh thứ hai, chúng ta cho phép `new_setting_value` trở thành `setting_value`.

Chúng ta cũng có thể sử dụng dấu gạch dưới `_` ở nhiều vị trí trong một pattern để bỏ qua các giá trị cụ thể. Listing 18-19 đưa ra ví dụ bỏ qua giá trị thứ hai và thứ tư trong một tuple gồm năm phần tử.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-19/src/main.rs:here}}
```

<span class="caption">Listing 18-19: Bỏ qua nhiều phần của một tuple</span>

Mã này sẽ in ra `Some numbers: 2, 8, 32`, và các giá trị 4 và 16 sẽ bị bỏ qua.

#### Bỏ qua biến không sử dụng bằng cách đặt tên biến bắt đầu bằng `_`

Nếu bạn tạo một biến nhưng không sử dụng nó ở bất kỳ đâu, Rust thường sẽ đưa ra cảnh báo vì một biến không sử dụng có thể là một lỗi. Tuy nhiên, đôi khi việc tạo một biến mà bạn chưa dùng đến cũng hữu ích, ví dụ khi bạn đang thử nghiệm hoặc mới bắt đầu một dự án. Trong trường hợp này, bạn có thể bảo Rust không cảnh báo về biến không sử dụng bằng cách đặt tên biến bắt đầu bằng dấu gạch dưới `_`. Trong Listing 18-20, chúng ta tạo hai biến không sử dụng, nhưng khi biên dịch mã này, chúng ta chỉ nhận cảnh báo về một trong số chúng.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-20/src/main.rs}}
```

<span class="caption">Listing 18-20: Đặt tên biến bắt đầu bằng dấu gạch dưới để tránh cảnh báo biến không sử dụng</span>

Ở đây, chúng ta nhận được cảnh báo về việc không sử dụng biến `y`, nhưng không có cảnh báo về việc không sử dụng `_x`.

Lưu ý rằng có một sự khác biệt tinh tế giữa việc chỉ dùng `_` và dùng một tên bắt đầu bằng dấu gạch dưới. Cú pháp `_x` vẫn gán giá trị cho biến `_x`, trong khi `_` không gán gì cả. Để minh họa trường hợp mà sự khác biệt này quan trọng, Listing 18-21 sẽ cung cấp cho chúng ta một lỗi.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-21/src/main.rs:here}}
```

<span class="caption">Listing 18-21: Một biến không sử dụng bắt đầu bằng dấu gạch dưới vẫn gán giá trị, điều này có thể chiếm quyền sở hữu giá trị</span>

Chúng ta sẽ nhận được lỗi vì giá trị `s` vẫn bị chuyển vào `_s`, điều này ngăn chúng ta sử dụng lại `s`. Tuy nhiên, việc chỉ dùng dấu gạch dưới `_` sẽ không bao giờ gán giá trị. Listing 18-22 sẽ biên dịch mà không có lỗi nào vì `s` không bị chuyển vào `_`.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-22/src/main.rs:here}}
```

<span class="caption">Listing 18-22: Sử dụng dấu gạch dưới không gán giá trị</span>

Đoạn mã này hoạt động bình thường vì chúng ta không bao giờ gán `s` cho bất cứ thứ gì; nó không bị di chuyển.

#### Bỏ qua các phần còn lại của giá trị với `..`

Với các giá trị có nhiều phần, chúng ta có thể sử dụng cú pháp `..` để dùng một số phần cụ thể và bỏ qua phần còn lại, tránh phải liệt kê nhiều dấu gạch dưới cho từng giá trị bị bỏ qua. Mẫu `..` bỏ qua bất kỳ phần nào của giá trị mà chúng ta chưa khớp rõ ràng trong phần còn lại của mẫu. Trong Listing 18-23, chúng ta có một struct `Point` chứa tọa độ trong không gian ba chiều. Trong biểu thức `match`, chúng ta chỉ muốn thao tác trên tọa độ `x` và bỏ qua các giá trị trong các trường `y` và `z`.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-23/src/main.rs:here}}
```

<span class="caption">Listing 18-23: Bỏ qua tất cả các trường của `Point` ngoại trừ `x` bằng cách sử dụng `..`</span>

Chúng ta liệt kê giá trị `x` và sau đó chỉ bao gồm mẫu `..`. Cách này nhanh hơn so với việc phải liệt kê `y: _` và `z: _`, đặc biệt khi làm việc với các struct có nhiều trường trong những trường hợp chỉ một hoặc hai trường là cần thiết.

Cú pháp `..` sẽ mở rộng ra đủ số lượng giá trị cần thiết. Listing 18-24 cho thấy cách sử dụng `..` với một tuple.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-24/src/main.rs}}
```

<span class="caption">Listing 18-24: Chỉ khớp giá trị đầu và cuối trong một tuple và bỏ qua tất cả các giá trị khác</span>

Trong đoạn code này, giá trị đầu và cuối được khớp với `first` và `last`. Mẫu `..` sẽ khớp và bỏ qua tất cả các giá trị ở giữa.

Tuy nhiên, việc sử dụng `..` phải rõ ràng. Nếu không rõ giá trị nào cần khớp và giá trị nào cần bỏ qua, Rust sẽ báo lỗi. Listing 18-25 cho thấy một ví dụ sử dụng `..` một cách mơ hồ, vì vậy sẽ không biên dịch được.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-25/src/main.rs}}
```

<span class="caption">Listing 18-25: Cố gắng sử dụng `..` một cách mơ hồ</span>

Khi biên dịch ví dụ này, chúng ta nhận được lỗi sau:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-25/output.txt}}
```

Việc Rust không thể xác định được bao nhiêu giá trị trong tuple sẽ bị 
bỏ qua trước khi so khớp với giá trị `second`, và sau đó bao nhiêu giá trị tiếp theo bị bỏ qua. 
Mã này có thể có nghĩa là chúng ta muốn bỏ qua `2`, gán `second` là `4`, 
và sau đó bỏ qua `8`, `16`, và `32`; hoặc bỏ qua `2` và `4`, gán `second` là `8`, 
và sau đó bỏ qua `16` và `32`; v.v. Tên biến `second` không có ý nghĩa đặc biệt với Rust, 
vì vậy chúng ta nhận được lỗi biên dịch vì việc sử dụng `..` ở hai vị trí như vậy là mơ hồ.

### Điều kiện bổ sung với Match Guards

*Match guard* là một điều kiện `if` bổ sung, được chỉ định sau pattern 
trong một nhánh `match`, và điều kiện này cũng phải đúng để nhánh đó được chọn. 
Match guards hữu ích để biểu diễn các ý tưởng phức tạp hơn so với việc chỉ dùng pattern.

Điều kiện có thể sử dụng các biến được tạo ra trong pattern. 
Listing 18-26 minh họa một `match` mà nhánh đầu tiên có pattern `Some(x)` 
và đồng thời có match guard `if x % 2 == 0` (điều này sẽ đúng nếu số là chẵn).

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-26/src/main.rs:here}}
```

<span class="caption">Listing 18-26: Thêm match guard vào một pattern</span>

Ví dụ này sẽ in ra `The number 4 is even`. Khi `num` 
được so khớp với pattern ở nhánh đầu tiên, nó khớp vì `Some(4)` khớp với `Some(x)`. 
Sau đó, match guard kiểm tra xem phần dư khi chia `x` 
cho 2 có bằng 0 không, và vì bằng 0, nhánh đầu tiên được chọn.

Nếu `num` là `Some(5)`, match guard ở nhánh đầu tiên sẽ trả về false 
vì phần dư của 5 chia 2 là 1, không bằng 0. Rust sẽ chuyển sang nhánh thứ hai, 
nhánh này khớp vì nhánh thứ hai không có match guard và do đó khớp với bất kỳ biến thể `Some` nào.

Không có cách nào để biểu diễn điều kiện `if x % 2 == 0` trong một pattern, 
vì vậy match guard cho phép chúng ta diễn đạt logic này. 
Nhược điểm của khả năng biểu diễn bổ sung này là trình biên dịch 
sẽ không kiểm tra tính đầy đủ (exhaustiveness) khi có các biểu thức match guard.

Trong Listing 18-11, chúng ta đã đề cập rằng có thể dùng match guards để giải quyết vấn đề shadowing biến trong pattern. Hãy nhớ rằng chúng ta đã tạo một biến mới bên trong pattern trong biểu thức `match` thay vì dùng biến bên ngoài `match`. Biến mới này khiến chúng ta không thể kiểm tra giá trị của biến bên ngoài. Listing 18-27 minh họa cách dùng match guard để khắc phục vấn đề này.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-27/src/main.rs}}
```

<span class="caption">Listing 18-27: Sử dụng match guard để kiểm tra bằng với biến bên ngoài</span>

Mã này giờ sẽ in ra `Default case, x = Some(5)`. Pattern trong nhánh thứ hai không tạo ra một biến mới `y` để shadow biến `y` bên ngoài, có nghĩa là chúng ta có thể sử dụng biến `y` bên ngoài trong match guard. Thay vì chỉ định pattern là `Some(y)`, điều này sẽ shadow biến `y` bên ngoài, chúng ta chỉ định `Some(n)`. Điều này tạo ra một biến mới `n` không shadow bất cứ gì vì bên ngoài `match` không có biến `n`.

Match guard `if n == y` không phải là một pattern và do đó không tạo biến mới. Biến `y` này *là* biến `y` bên ngoài, không phải một `y` mới bị shadow, và chúng ta có thể tìm giá trị có cùng giá trị với biến `y` bên ngoài bằng cách so sánh `n` với `y`.

Bạn cũng có thể sử dụng toán tử *or* `|` trong match guard để chỉ định nhiều pattern; điều kiện match guard sẽ áp dụng cho tất cả các pattern. Listing 18-28 minh họa thứ tự ưu tiên khi kết hợp pattern dùng `|` với match guard. Phần quan trọng của ví dụ này là match guard `if y` áp dụng cho `4`, `5`, *và* `6`, mặc dù có vẻ như `if y` chỉ áp dụng cho `6`.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-28/src/main.rs:here}}
```

<span class="caption">Listing 18-28: Kết hợp nhiều pattern với match guard</span>

Điều kiện match chỉ ra rằng nhánh này chỉ được chọn nếu giá trị của `x` bằng `4`, `5` hoặc `6` *và* nếu `y` là `true`. Khi đoạn mã này chạy, pattern của nhánh đầu tiên khớp vì `x` là `4`, nhưng match guard `if y` là false, nên nhánh đầu tiên không được chọn. Chương trình tiếp tục sang nhánh thứ hai, nhánh này khớp, và chương trình in ra `no`. Nguyên nhân là điều kiện `if` áp dụng cho toàn bộ pattern `4 | 5 | 6`, không chỉ giá trị cuối cùng `6`. Nói cách khác, thứ tự ưu tiên của match guard so với pattern được hiểu như sau:

```text
(4 | 5 | 6) if y => ...
```

rather than this:

```text
4 | 5 | (6 if y) => ...
```

Sau khi chạy mã, hành vi ưu tiên trở nên rõ ràng: nếu match guard chỉ áp dụng cho giá trị cuối cùng trong danh sách các giá trị được chỉ định bằng toán tử `|`, nhánh đó sẽ khớp và chương trình sẽ in ra `yes`.

### Gán với `@`

Toán tử *at* `@` cho phép chúng ta tạo một biến giữ giá trị đồng thời với việc kiểm tra giá trị đó để khớp với pattern. Trong Listing 18-29, chúng ta muốn kiểm tra rằng trường `id` của `Message::Hello` nằm trong phạm vi `3..=7`. Đồng thời, chúng ta muốn gán giá trị này cho biến `id_variable` để sử dụng trong mã của nhánh. Chúng ta có thể đặt tên biến này là `id`, giống như tên trường, nhưng trong ví dụ này sẽ dùng một tên khác.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-29/src/main.rs:here}}
```

<span class="caption">Listing 18-29: Sử dụng `@` để gán giá trị trong một pattern đồng thời kiểm tra nó</span>

Ví dụ này sẽ in ra `Found an id in range: 5`. Bằng cách chỉ định `id_variable @` trước phạm vi `3..=7`, chúng ta đang nắm bắt bất kỳ giá trị nào khớp với phạm vi đồng thời kiểm tra rằng giá trị đó khớp với pattern phạm vi.

Trong nhánh thứ hai, nơi chỉ có một phạm vi được chỉ định trong pattern, mã liên quan đến nhánh không có biến nào chứa giá trị thực tế của trường `id`. Giá trị trường `id` có thể là 10, 11, hoặc 12, nhưng mã đi kèm với pattern đó không biết giá trị nào. Mã pattern không thể sử dụng giá trị từ trường `id`, vì chúng ta chưa lưu giá trị `id` vào một biến.

Trong nhánh cuối cùng, nơi chúng ta đã chỉ định một biến mà không có phạm vi, chúng ta có giá trị sẵn để sử dụng trong mã nhánh thông qua biến có tên `id`. Lý do là chúng ta đã sử dụng cú pháp viết tắt cho trường struct. Nhưng chúng ta không áp dụng bất kỳ kiểm tra nào cho giá trị trong trường `id` trong nhánh này, như đã làm với hai nhánh đầu: bất kỳ giá trị nào cũng sẽ khớp với pattern này.

Sử dụng `@` cho phép chúng ta vừa kiểm tra giá trị vừa lưu nó vào một biến trong cùng một pattern.

## Tóm tắt

Các pattern trong Rust rất hữu ích để phân biệt các loại dữ liệu khác nhau. Khi sử dụng trong biểu thức `match`, Rust đảm bảo các pattern của bạn bao phủ mọi giá trị có thể, nếu không chương trình sẽ không biên dịch. Các pattern trong các câu lệnh `let` và tham số hàm làm cho các cấu trúc này hữu ích hơn, cho phép phá cấu trúc các giá trị thành các phần nhỏ hơn đồng thời gán cho các biến. Chúng ta có thể tạo các pattern đơn giản hoặc phức tạp tùy theo nhu cầu.

Tiếp theo, trong chương áp chót của cuốn sách, chúng ta sẽ xem xét một số khía cạnh nâng cao của nhiều tính năng trong Rust.
