## Xác thực References với Lifetimes

Lifetimes là một loại generic khác mà chúng ta đã sử dụng. Thay vì đảm bảo rằng một kiểu có hành vi như mong muốn, lifetimes đảm bảo rằng các reference hợp lệ trong suốt thời gian chúng ta cần.

Một chi tiết mà chúng ta chưa thảo luận trong phần [“References and Borrowing”][references-and-borrowing]<!-- ignore --> ở Chương 4 là mỗi reference trong Rust có một *lifetime*, tức là phạm vi mà reference đó hợp lệ. Phần lớn thời gian, lifetimes là ẩn và được suy luận, giống như hầu hết thời gian, các kiểu cũng được suy luận. Chúng ta chỉ cần chú thích kiểu khi có nhiều kiểu khả thi. Tương tự, chúng ta phải chú thích lifetimes khi các lifetimes của references có thể liên quan theo nhiều cách khác nhau. Rust yêu cầu chúng ta chú thích các mối quan hệ này bằng các tham số lifetime generic để đảm bảo các reference thực tế được sử dụng tại runtime chắc chắn hợp lệ.

Chú thích lifetimes thậm chí còn không phải là một khái niệm mà hầu hết các ngôn ngữ lập trình khác có, vì vậy điều này sẽ cảm thấy lạ. Mặc dù chúng ta sẽ không bao quát toàn bộ lifetimes trong chương này, chúng ta sẽ thảo luận các cách phổ biến mà bạn có thể gặp cú pháp lifetime để bạn quen với khái niệm này.

### Ngăn ngừa Dangling References với Lifetimes

Mục tiêu chính của lifetimes là ngăn ngừa *dangling references*, là những reference khiến chương trình tham chiếu đến dữ liệu khác dữ liệu mà nó dự định tham chiếu. Xem xét chương trình trong Listing 10-16, có một phạm vi bên ngoài và một phạm vi bên trong.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

<span class="caption">Listing 10-16: Một nỗ lực sử dụng reference mà giá trị của nó đã ra khỏi phạm vi</span>

> Lưu ý: Các ví dụ trong Listings 10-16, 10-17 và 10-23 khai báo biến mà không gán giá trị ban đầu, vì vậy tên biến tồn tại trong phạm vi bên ngoài. Nhìn sơ qua, điều này có vẻ mâu thuẫn với việc Rust không có giá trị null. Tuy nhiên, nếu chúng ta cố sử dụng một biến trước khi gán giá trị, chúng ta sẽ nhận lỗi tại thời điểm biên dịch, điều này chứng tỏ Rust thực sự không cho phép giá trị null.

Phạm vi bên ngoài khai báo một biến tên `r` mà không có giá trị ban đầu, và phạm vi bên trong khai báo một biến tên `x` với giá trị khởi tạo là 5. Bên trong phạm vi bên trong, chúng ta cố gắng gán giá trị của `r` làm reference tới `x`. Sau đó phạm vi bên trong kết thúc, và chúng t


```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Variable `x` không “sống đủ lâu” (live long enough). Lý do là `x` sẽ ra khỏi scope khi inner scope kết thúc ở line 7. Nhưng `r` vẫn hợp lệ trong outer scope; vì phạm vi của nó lớn hơn, ta nói rằng nó “sống lâu hơn” (lives longer). Nếu Rust cho phép code này chạy, `r` sẽ tham chiếu đến memory đã bị giải phóng khi `x` ra khỏi scope, và mọi thao tác với `r` sẽ không đúng. Vậy Rust làm sao biết code này không hợp lệ? Nó dùng borrow checker.

### The Borrow Checker

Compiler của Rust có một *borrow checker* dùng để so sánh các scope và xác định xem tất cả các borrow có hợp lệ hay không. Listing 10-17 hiển thị cùng đoạn code với Listing 10-16 nhưng có chú thích thể hiện lifetimes của các biến.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

<span class="caption">Listing 10-17: Chú thích lifetimes của `r` và `x`, lần lượt là `'a` và `'b`</span>

Ở đây, chúng ta đã chú thích lifetime của `r` với `'a` và lifetime của `x` với `'b`. Như bạn thấy, block `'b` bên trong nhỏ hơn nhiều so với block `'a` bên ngoài. Tại thời điểm compile, Rust so sánh kích thước của hai lifetime và nhận thấy `r` có lifetime `'a` nhưng tham chiếu tới memory có lifetime `'b`. Chương trình bị từ chối vì `'b` ngắn hơn `'a`: giá trị mà reference trỏ tới không sống lâu bằng reference.

Listing 10-18 sửa code để không còn dangling reference và compile mà không có lỗi.

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

<span class="caption">Listing 10-18: Một reference hợp lệ vì dữ liệu có lifetime dài hơn reference</span>

Ở đây, `x` có lifetime `'b`, trong trường hợp này lớn hơn `'a`. Điều này có nghĩa là `r` có thể tham chiếu tới `x` vì Rust biết reference trong `r` sẽ luôn hợp lệ khi `x` còn hợp lệ.

Bây giờ, khi bạn đã hiểu nơi lifetimes của các reference nằm và cách Rust phân tích lifetimes để đảm bảo reference luôn hợp lệ, hãy cùng khám phá generic lifetimes của tham số và giá trị trả về trong ngữ cảnh của các hàm.

### Generic Lifetimes trong Functions

Chúng ta sẽ viết một hàm trả về string slice dài hơn trong hai string slice. Hàm này sẽ nhận hai string slice và trả về một string slice duy nhất. Sau khi triển khai hàm `longest`, code trong Listing 10-19 sẽ in ra `The longest string is abcd`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

<span class="caption">Listing 10-19: Một hàm `main` gọi hàm `longest` để tìm string slice dài hơn trong hai string slice</span>

Lưu ý rằng chúng ta muốn hàm nhận string slices, là references, thay vì strings, vì chúng ta không muốn hàm `longest` chiếm quyền sở hữu các tham số. Tham khảo phần [“String Slices as Parameters”][string-slices-as-parameters]<!-- ignore --> trong Chương 4 để tìm hiểu thêm lý do tại sao các tham số trong Listing 10-19 là những tham số chúng ta muốn sử dụng.

Nếu cố gắng triển khai hàm `longest` như trong Listing 10-20, code sẽ không compile.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

<span class="caption">Listing 10-20: Một triển khai của hàm `longest` trả về string slice dài hơn trong hai string slice nhưng chưa compile</span>

Thay vào đó, chúng ta nhận được lỗi sau liên quan đến lifetimes:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Hướng dẫn lỗi cho thấy kiểu trả về cần một generic lifetime parameter vì Rust không biết reference được trả về trỏ tới `x` hay `y`. Thực ra, chúng ta cũng không biết, vì block `if` trong thân hàm trả về reference tới `x` và block `else` trả về reference tới `y`!

Khi định nghĩa hàm này, chúng ta không biết các giá trị cụ thể sẽ được truyền vào hàm, nên không biết block `if` hay `else` sẽ được thực thi. Chúng ta cũng không biết concrete lifetimes của các reference được truyền vào, nên không thể quan sát các scope như trong Listings 10-17 và 10-18 để xác định reference trả về có luôn hợp lệ hay không. Borrow checker cũng không thể xác định, vì nó không biết lifetimes của `x` và `y` liên quan thế nào tới lifetime của giá trị trả về. Để sửa lỗi này, chúng ta sẽ thêm generic lifetime parameters để định nghĩa mối quan hệ giữa các reference, giúp borrow checker thực hiện phân tích.

### Lifetime Annotation Syntax

Lifetime annotations không thay đổi thời gian sống (lifetime) của bất kỳ reference nào. Thay vào đó, chúng mô tả mối quan hệ giữa lifetimes của nhiều reference với nhau mà không ảnh hưởng tới lifetimes. Cũng như hàm có thể nhận bất kỳ kiểu nào khi chữ ký khai báo generic type parameter, hàm có thể nhận reference với bất kỳ lifetime nào bằng cách khai báo generic lifetime parameter.

Lifetime annotations có cú pháp hơi khác thường: tên các lifetime parameter phải bắt đầu bằng apostrophe (`'`) và thường viết thường, ngắn, giống như generic types. Hầu hết mọi người dùng `'a` cho lifetime annotation đầu tiên. Chúng ta đặt lifetime parameter annotation sau `&` của reference, dùng khoảng trắng để tách annotation khỏi type của reference.

Dưới đây là một số ví dụ: reference tới `i32` mà không có lifetime parameter, reference tới `i32` có lifetime parameter tên `'a`, và mutable reference tới `i32` cũng có lifetime `'a`.

```rust,ignore
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

Một lifetime annotation đơn lẻ không có nhiều ý nghĩa, vì các annotation này nhằm chỉ cho Rust cách các generic lifetime parameter của nhiều reference liên quan với nhau. Hãy xem cách các lifetime annotation liên quan trong ngữ cảnh của hàm `longest`.

### Lifetime Annotations trong Function Signatures

Để sử dụng lifetime annotations trong chữ ký hàm, chúng ta cần khai báo các generic *lifetime* parameters trong dấu ngoặc nhọn giữa tên hàm và danh sách tham số, giống như cách chúng ta làm với các generic *type* parameters.

Chúng ta muốn chữ ký biểu diễn ràng buộc sau: reference trả về sẽ hợp lệ chừng nào cả hai tham số còn hợp lệ. Đây là mối quan hệ giữa lifetimes của các tham số và giá trị trả về. Chúng ta sẽ đặt tên lifetime là `'a` và thêm nó vào từng reference, như trong Listing 10-21.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

<span class="caption">Listing 10-21: Định nghĩa hàm `longest` chỉ định rằng tất cả reference trong chữ ký phải có cùng lifetime `'a`</span>

Code này sẽ compile và cho kết quả như mong muốn khi sử dụng với hàm `main` trong Listing 10-19.

Chữ ký hàm bây giờ cho Rust biết rằng với một lifetime `'a`, hàm nhận hai tham số, cả hai đều là string slices sống ít nhất bằng lifetime `'a`. Chữ ký hàm cũng cho Rust biết rằng string slice được trả về từ hàm sẽ sống ít nhất bằng lifetime `'a`. Thực tế, điều này có nghĩa là lifetime của reference trả về bởi hàm `longest` sẽ bằng lifetime nhỏ hơn trong hai lifetimes của các giá trị được tham chiếu bởi các tham số của hàm. Đây là mối quan hệ mà chúng ta muốn Rust sử dụng khi phân tích code này.

Hãy nhớ rằng, khi chúng ta chỉ định các lifetime parameters trong chữ ký hàm này, chúng ta không thay đổi lifetime của bất kỳ giá trị nào được truyền vào hoặc trả về. Thay vào đó, chúng ta chỉ định rằng borrow checker sẽ từ chối bất kỳ giá trị nào không tuân thủ các ràng buộc này. Lưu ý rằng hàm `longest` không cần biết chính xác `x` và `y` sẽ sống bao lâu, chỉ cần biết rằng một scope nào đó có thể thay thế cho `'a` để thỏa mãn chữ ký này.

Khi chú thích lifetimes trong functions, các annotation đặt trong chữ ký hàm, không phải trong thân hàm. Các lifetime annotation trở thành một phần của hợp đồng của hàm, giống như các kiểu trong chữ ký. Việc chữ ký hàm chứa hợp đồng về lifetime giúp Rust compiler phân tích code dễ dàng hơn. Nếu có vấn đề với cách một hàm được annotation hoặc cách nó được gọi, lỗi của compiler sẽ chỉ ra chính xác phần code và các ràng buộc. Nếu Rust compiler tự suy luận nhiều hơn về mối quan hệ các lifetime, compiler có thể chỉ ra lỗi cách nguyên nhân nhiều bước, khó xác định.

Khi truyền các reference cụ thể vào `longest`, lifetime cụ thể thay thế cho `'a` là phần scope của `x` chồng lấn với scope của `y`. Nói cách khác, generic lifetime `'a` sẽ nhận lifetime cụ thể bằng lifetime nhỏ hơn giữa `x` và `y`. Vì chúng ta đã chú thích reference trả về với cùng lifetime `'a`, reference trả về cũng sẽ hợp lệ trong khoảng thời gian bằng lifetime nhỏ hơn giữa `x` và `y`.

Hãy xem cách các lifetime annotation giới hạn hàm `longest` khi truyền vào các reference có lifetimes khác nhau. Listing 10-22 là một ví dụ đơn giản.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

<span class="caption">Listing 10-22: Sử dụng hàm `longest` với các reference tới các giá trị `String` có lifetimes cụ thể khác nhau</span>

Trong ví dụ này, `string1` hợp lệ cho đến cuối outer scope, `string2` hợp lệ cho đến cuối inner scope, và `result` tham chiếu tới một giá trị hợp lệ cho đến cuối inner scope. Chạy code này, bạn sẽ thấy borrow checker chấp nhận; code sẽ compile và in ra `The longest string is long string is long`.

Tiếp theo, hãy thử một ví dụ cho thấy lifetime của reference trong `result` phải là lifetime nhỏ hơn trong hai tham số. Chúng ta sẽ chuyển khai báo biến `result` ra ngoài inner scope nhưng vẫn để việc gán giá trị cho `result` bên trong scope với `string2`. Sau đó, chúng ta sẽ di chuyển `println!` sử dụng `result` ra ngoài inner scope, sau khi inner scope kết thúc. Code trong Listing 10-23 sẽ không compile.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

<span class="caption">Listing 10-23: Cố gắng sử dụng `result` sau khi `string2` đã ra khỏi scope</span>

Khi cố compile code này, chúng ta nhận được lỗi sau:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Lỗi cho thấy để `result` hợp lệ cho câu lệnh `println!`, `string2` cần phải hợp lệ cho đến cuối outer scope. Rust biết điều này vì chúng ta đã chú thích lifetimes của các tham số và giá trị trả về của hàm bằng cùng lifetime parameter `'a`.

Là con người, chúng ta có thể nhìn vào code này và thấy rằng `string1` dài hơn `string2`, do đó `result` sẽ chứa reference tới `string1`. Vì `string1` vẫn chưa ra khỏi scope, reference tới `string1` sẽ hợp lệ cho câu lệnh `println!`. Tuy nhiên, compiler không thể thấy reference này hợp lệ trong trường hợp này. Chúng ta đã nói với Rust rằng lifetime của reference trả về từ hàm `longest` bằng lifetime nhỏ hơn trong hai reference được truyền vào. Do đó, borrow checker sẽ không cho phép code trong Listing 10-23 vì có khả năng chứa reference không hợp lệ.

Hãy thử thiết kế thêm các thí nghiệm thay đổi giá trị và lifetimes của các reference được truyền vào hàm `longest` và cách reference trả về được sử dụng. Đưa ra giả thuyết về việc thí nghiệm có vượt qua borrow checker hay không trước khi compile; sau đó kiểm tra xem bạn có đúng không!

### Nghĩ theo hướng Lifetimes

Cách bạn cần chỉ định lifetime parameters phụ thuộc vào việc hàm của bạn làm gì. Ví dụ, nếu chúng ta thay đổi implementation của hàm `longest` để luôn trả về tham số đầu tiên thay vì string slice dài nhất, chúng ta sẽ không cần chỉ định lifetime cho tham số `y`. Code

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

Chúng ta đã chỉ định lifetime parameter `'a` cho tham số `x` và kiểu trả về, nhưng không cho tham số `y`, vì lifetime của `y` không có mối quan hệ nào với lifetime của `x` hoặc giá trị trả về.

Khi trả về một reference từ một hàm, lifetime parameter của kiểu trả về cần khớp với lifetime parameter của một trong các tham số. Nếu reference trả về *không* tham chiếu tới một trong các tham số, nó phải tham chiếu tới một giá trị được tạo bên trong hàm này. Tuy nhiên, điều này sẽ tạo ra dangling reference vì giá trị sẽ ra khỏi scope khi hàm kết thúc. Xem xét triển khai hàm `longest` dưới đây sẽ không compile:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

Ở đây, mặc dù chúng ta đã chỉ định lifetime parameter `'a` cho kiểu trả về, implementation này vẫn không compile vì lifetime của giá trị trả về không liên quan gì đến lifetime của các tham số. Dưới đây là thông báo lỗi mà chúng ta nhận được:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Vấn đề là `result` ra khỏi scope và bị giải phóng khi hàm `longest` kết thúc. Chúng ta lại cố gắng trả về reference tới `result` từ hàm. Không có cách nào chỉ định lifetime parameters để thay đổi dangling reference này, và Rust sẽ không cho phép tạo dangling reference. Trong trường hợp này, cách sửa tốt nhất là trả về một kiểu dữ liệu owned thay vì reference, để hàm gọi chịu trách nhiệm giải phóng giá trị.

Cuối cùng, cú pháp lifetime là để kết nối các lifetimes của các tham số và giá trị trả về của hàm. Khi các lifetimes được kết nối, Rust có đủ thông tin để cho phép các thao tác an toàn với bộ nhớ và ngăn các thao tác tạo ra dangling pointer hoặc vi phạm an toàn bộ nhớ.

### Lifetime Annotations trong Struct Definitions

Cho đến nay, các struct chúng ta định nghĩa đều chứa các kiểu owned. Chúng ta có thể định nghĩa struct chứa references, nhưng trong trường hợp đó cần thêm lifetime annotation cho mỗi reference trong định nghĩa struct. Listing 10-24 có một struct tên là `ImportantExcerpt` chứa một string slice.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

<span class="caption">Listing 10-24: Một struct chứa reference, yêu cầu lifetime annotation</span>

Struct này có một field duy nhất `part` chứa một string slice, vốn là một reference. Giống như với generic data types, chúng ta khai báo tên của generic lifetime parameter trong dấu ngoặc nhọn sau tên struct để có thể sử dụng lifetime parameter trong thân định nghĩa struct. Annotation này có nghĩa là một instance của `ImportantExcerpt` không thể sống lâu hơn reference mà nó giữ trong field `part`.

Hàm `main` ở đây tạo một instance của struct `ImportantExcerpt` giữ reference tới câu đầu tiên của `String` do biến `novel` sở hữu. Dữ liệu trong `novel` tồn tại trước khi instance của `ImportantExcerpt` được tạo. Thêm vào đó, `novel` không ra khỏi scope cho đến sau khi `ImportantExcerpt` ra khỏi scope, nên reference trong instance của `ImportantExcerpt` là hợp lệ.

### Lifetime Elision

Bạn đã học rằng mỗi reference có một lifetime và cần chỉ định lifetime parameters cho các hàm hoặc struct sử dụng reference. Tuy nhiên, trong Chương 4, chúng ta có một hàm trong Listing 4-9, được nhắc lại trong Listing 10-25, mà compile mà không cần lifetime annotations.

<span class="filename">Filename: src/lib.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

<span class="caption">Listing 10-25: Một hàm chúng ta đã định nghĩa trong Listing 4-9 mà compile mà không cần lifetime annotations, mặc dù tham số và kiểu trả về là references</span>

Lý do hàm này compile mà không cần lifetime annotations là do lý do lịch sử: trong các phiên bản đầu của Rust (trước 1.0), code này sẽ không compile vì mỗi reference cần một lifetime rõ ràng. Khi đó, chữ ký hàm sẽ được viết như sau:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Sau khi viết nhiều code Rust, nhóm phát triển Rust nhận thấy các lập trình viên thường nhập cùng một lifetime annotation nhiều lần trong những tình huống nhất định. Những tình huống này có thể dự đoán được và theo một số mẫu xác định. Các nhà phát triển đã lập trình các mẫu này vào compiler để borrow checker có thể suy luận lifetimes trong những tình huống này mà không cần annotation rõ ràng.

Lịch sử Rust này có liên quan vì có khả năng trong tương lai sẽ xuất hiện nhiều mẫu xác định hơn và được thêm vào compiler. Khi đó, thậm chí ít lifetime annotation hơn sẽ cần thiết.

Các mẫu được lập trình vào phân tích reference của Rust được gọi là *lifetime elision rules*. Đây không phải là các quy tắc dành cho lập trình viên; chúng là các trường hợp cụ thể mà compiler sẽ xem xét, và nếu code của bạn phù hợp các trường hợp này, bạn không cần viết lifetime một cách rõ ràng.

Các elision rules không cung cấp suy luận đầy đủ. Nếu Rust áp dụng các quy tắc một cách xác định nhưng vẫn còn mơ hồ về lifetime của các reference, compiler sẽ không đoán lifetime của các reference còn lại. Thay vì đoán, compiler sẽ báo lỗi và bạn có thể sửa bằng cách thêm lifetime annotation.

Lifetime trên các tham số hàm hoặc method được gọi là *input lifetimes*, và lifetime trên giá trị trả về được gọi là *output lifetimes*.

Compiler sử dụng ba quy tắc để xác định lifetime của reference khi không có annotation rõ ràng. Quy tắc đầu áp dụng cho input lifetimes, còn quy tắc thứ hai và thứ ba áp dụng cho output lifetimes. Nếu compiler áp dụng hết ba quy tắc mà vẫn còn reference chưa xác định được lifetime, compiler sẽ báo lỗi. Các quy tắc này áp dụng cho định nghĩa `fn` cũng như các `impl` block.

Quy tắc đầu tiên là compiler gán một lifetime parameter cho mỗi tham số là reference. Nói cách khác, một hàm có một tham số sẽ nhận một lifetime parameter: `fn foo<'a>(x: &'a i32)`; một hàm có hai tham số sẽ nhận hai lifetime parameter riêng biệt: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; và tương tự.

Quy tắc thứ hai là, nếu có đúng một input lifetime parameter, lifetime này sẽ được gán cho tất cả output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.

Quy tắc thứ ba là, nếu có nhiều input lifetime parameters, nhưng một trong số đó là `&self` hoặc `&mut self` vì đây là một method, lifetime của `self` sẽ được gán cho tất cả output lifetime parameters. Quy tắc thứ ba này giúp các method dễ đọc và viết hơn vì cần ít ký hiệu hơn.

Hãy giả sử chúng ta là compiler. Chúng ta sẽ áp dụng các quy tắc này để xác định lifetime của các reference trong chữ ký hàm `first_word` trong Listing 10-25. Chữ ký bắt đầu mà không có lifetime nào gắn với các reference:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Sau đó compiler áp dụng quy tắc đầu tiên, quy định rằng mỗi tham số nhận một lifetime riêng. Chúng ta sẽ gọi nó là `'a` như thường lệ, nên chữ ký bây giờ là:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

Quy tắc thứ hai áp dụng vì chỉ có đúng một input lifetime. Quy tắc thứ hai quy định rằng lifetime của tham số input này sẽ được gán cho output lifetime, nên chữ ký bây giờ là:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Bây giờ tất cả các reference trong chữ ký hàm này đã có lifetimes, và compiler có thể tiếp tục phân tích mà không cần lập trình viên phải annotation lifetimes trong chữ ký hàm này.

Hãy xem một ví dụ khác, lần này sử dụng hàm `longest` mà trước đó không có lifetime parameters khi chúng ta bắt đầu làm việc với nó trong Listing 10-20:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Hãy áp dụng quy tắc đầu tiên: mỗi tham số nhận một lifetime riêng. Lần này chúng ta có hai tham số thay vì một, nên sẽ có hai lifetimes:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Bạn có thể thấy rằng quy tắc thứ hai không áp dụng vì có nhiều hơn một input lifetime. Quy tắc thứ ba cũng không áp dụng, vì `longest` là một function chứ không phải method, nên không có tham số nào là `self`. Sau khi áp dụng cả ba quy tắc, chúng ta vẫn chưa xác định được lifetime của kiểu trả về. Đây là lý do chúng ta nhận được lỗi khi cố compile code trong Listing 10-20: compiler đã áp dụng các lifetime elision rules nhưng vẫn không xác định được tất cả lifetimes của các reference trong chữ ký.

Vì quy tắc thứ ba thực sự chỉ áp dụng trong chữ ký method, chúng ta sẽ xem xét lifetimes trong ngữ cảnh này tiếp theo để hiểu tại sao quy tắc thứ ba giúp không cần phải annotation lifetimes trong chữ ký method thường xuyên.

### Lifetime Annotations trong Method Definitions

Khi triển khai các method trên một struct có lifetimes, chúng ta sử dụng cùng cú pháp như với generic type parameters đã trình bày trong Listing 10-11. Việc khai báo và sử dụng lifetime parameters phụ thuộc vào việc chúng liên quan đến fields của struct hay các tham số và giá trị trả về của method.

Tên lifetime cho các field của struct luôn cần được khai báo sau từ khóa `impl` và sau đó sử dụng sau tên struct, vì những lifetime này là một phần của kiểu struct.

Trong chữ ký method bên trong `impl` block, các reference có thể liên quan tới lifetime của các reference trong field của struct, hoặc có thể độc lập. Thêm vào đó, các lifetime elision rules thường làm cho việc annotation lifetime trong chữ ký method không cần thiết. Hãy xem một số ví dụ sử dụng struct tên là `ImportantExcerpt` mà chúng ta đã định nghĩa trong Listing 10-24.

Trước tiên, chúng ta sẽ dùng một method tên là `level` mà tham số duy nhất là reference tới `self` và giá trị trả về là một `i32`, không phải reference tới bất cứ thứ gì:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

Việc khai báo lifetime parameter sau `impl` và sử dụng nó sau tên kiểu là bắt buộc, nhưng chúng ta không cần annotation lifetime của reference tới `self` vì quy tắc elision đầu tiên.

Dưới đây là ví dụ mà quy tắc elision thứ ba áp dụng:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Có hai input lifetimes, vì vậy Rust áp dụng quy tắc elision đầu tiên và gán cho cả `&self` và `announcement` lifetime riêng của chúng. Sau đó, vì một trong các tham số là `&self`, kiểu trả về nhận lifetime của `&self`, và tất cả các lifetimes đã được xác định.

### The Static Lifetime

Một lifetime đặc biệt mà chúng ta cần thảo luận là `'static`, biểu thị rằng reference bị ảnh hưởng *có thể* tồn tại trong toàn bộ thời gian chạy của chương trình. Tất cả string literals đều có lifetime `'static`, và chúng ta có thể annotation như sau:

```rust
let s: &'static str = "I have a static lifetime.";
```

Nội dung của string này được lưu trực tiếp trong binary của chương trình, vốn luôn có sẵn. Do đó, lifetime của tất cả string literals là `'static`.

Bạn có thể thấy các gợi ý sử dụng lifetime `'static` trong các thông báo lỗi. Nhưng trước khi chỉ định `'static` làm lifetime cho một reference, hãy suy nghĩ xem reference bạn có thực sự sống trong toàn bộ thời gian chạy của chương trình hay không, và liệu bạn có muốn vậy không. Phần lớn thời gian, một thông báo lỗi gợi ý lifetime `'static` xuất phát từ việc cố tạo một dangling reference hoặc sự không khớp giữa các lifetimes sẵn có. Trong những trường hợp này, giải pháp là sửa các vấn đề đó, chứ không phải chỉ định lifetime `'static`.

## Generic Type Parameters, Trait Bounds, and Lifetimes Together

Hãy cùng xem nhanh cú pháp để chỉ định generic type parameters, trait bounds và lifetimes trong cùng một hàm!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Đây là hàm `longest` từ Listing 10-21 trả về chuỗi con dài hơn trong hai chuỗi con. Nhưng bây giờ nó có thêm một tham số tên là `ann` với kiểu generic `T`, có thể là bất kỳ kiểu nào triển khai trait `Display` như được chỉ định trong `where` clause. Tham số thêm này sẽ được in ra bằng `{}`, đó là lý do cần trait bound `Display`. Vì lifetimes cũng là một loại generic, nên khai báo lifetime parameter `'a` và generic type parameter `T` được đặt trong cùng một danh sách bên trong dấu ngoặc nhọn sau tên hàm.

## Tóm tắt

Chúng ta đã bao quát rất nhiều trong chương này! Giờ đây bạn đã hiểu về generic type parameters, traits và trait bounds, cùng generic lifetime parameters, bạn đã sẵn sàng viết code không lặp lại và hoạt động trong nhiều tình huống khác nhau. Generic type parameters cho phép áp dụng code cho nhiều kiểu khác nhau. Traits và trait bounds đảm bảo rằng mặc dù các kiểu là generic, chúng vẫn có hành vi mà code cần. Bạn đã học cách sử dụng lifetime annotations để đảm bảo rằng code linh hoạt này sẽ không có bất kỳ dangling reference nào. Và tất cả phân tích này xảy ra tại thời điểm compile, không ảnh hưởng đến hiệu năng runtime!

Tin hay không, vẫn còn nhiều điều để học về các chủ đề đã thảo luận trong chương này: Chương 17 sẽ nói về trait objects, một cách khác để sử dụng traits. Cũng có những tình huống phức tạp hơn liên quan đến lifetime annotations mà bạn chỉ cần trong các trường hợp nâng cao; với những trường hợp đó, bạn nên đọc [Rust Reference][reference]. Nhưng tiếp theo, bạn sẽ học cách viết tests trong Rust để đảm bảo code của bạn hoạt động như mong đợi.

[references-and-borrowing]:
ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]:
ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/index.html
