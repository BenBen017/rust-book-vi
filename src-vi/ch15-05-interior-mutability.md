## `RefCell<T>` và Mẫu Interior Mutability

*Interior mutability* là một mẫu thiết kế trong Rust cho phép bạn thay đổi dữ liệu ngay cả khi có các tham chiếu không thay đổi đến dữ liệu đó; thông thường, hành động này bị cấm bởi các quy tắc mượn. Để thay đổi dữ liệu, mẫu này sử dụng mã `unsafe` bên trong một cấu trúc dữ liệu để uốn nắn các quy tắc thông thường của Rust về mutation và borrowing. Mã `unsafe` báo cho trình biên dịch rằng chúng ta sẽ kiểm tra các quy tắc thủ công thay vì dựa vào trình biên dịch; chúng ta sẽ thảo luận thêm về mã `unsafe` trong Chương 19.

Chúng ta chỉ có thể sử dụng các kiểu dữ liệu áp dụng mẫu interior mutability khi có thể đảm bảo rằng các quy tắc mượn sẽ được tuân thủ tại thời điểm chạy, mặc dù trình biên dịch không thể đảm bảo điều đó. Mã `unsafe` liên quan sẽ được bao bọc trong một API an toàn, và kiểu bên ngoài vẫn không thay đổi.

Hãy khám phá khái niệm này bằng cách xem kiểu `RefCell<T>` áp dụng mẫu interior mutability.

### Thực thi các quy tắc mượn tại thời điểm chạy với `RefCell<T>`

Khác với `Rc<T>`, kiểu `RefCell<T>` đại diện cho quyền sở hữu duy nhất đối với dữ liệu mà nó giữ. Vậy điều gì làm `RefCell<T>` khác với một kiểu như `Box<T>`? Hãy nhớ lại các quy tắc mượn mà bạn đã học trong Chương 4:

* Tại bất kỳ thời điểm nào, bạn có thể có *hoặc* (nhưng không phải cả hai) một tham chiếu có thể thay đổi hoặc bất kỳ số lượng tham chiếu không thay đổi nào.
* Các tham chiếu phải luôn hợp lệ.

Với các tham chiếu và `Box<T>`, các quy tắc mượn được thực thi tại thời điểm biên dịch. Với `RefCell<T>`, những bất biến này được thực thi *tại thời điểm chạy*. Với các tham chiếu, nếu bạn vi phạm các quy tắc này, bạn sẽ nhận được lỗi biên dịch. Với `RefCell<T>`, nếu bạn vi phạm các quy tắc này, chương trình của bạn sẽ panic và kết thúc.

Ưu điểm của việc kiểm tra các quy tắc mượn tại thời điểm biên dịch là các lỗi sẽ được phát hiện sớm hơn trong quá trình phát triển, và không ảnh hưởng đến hiệu năng tại thời điểm chạy vì tất cả phân tích đã hoàn tất trước đó. Vì những lý do này, kiểm tra các quy tắc mượn tại thời điểm biên dịch là lựa chọn tốt nhất trong hầu hết các trường hợp, đó là lý do tại sao đây là mặc định của Rust.

Ưu điểm của việc kiểm tra các quy tắc mượn tại thời điểm chạy là cho phép một số tình huống an toàn về bộ nhớ mà nếu kiểm tra tại thời điểm biên dịch sẽ bị cấm. Phân tích tĩnh, như trình biên dịch Rust, vốn mang tính thận trọng. Một số đặc tính của mã là không thể phát hiện bằng việc phân tích mã: ví dụ nổi tiếng nhất là Vấn đề Halting, vượt ngoài phạm vi cuốn sách này nhưng là một chủ đề thú vị để nghiên cứu.

Bởi vì một số phân tích là không thể, nếu trình biên dịch Rust không chắc chắn mã tuân thủ các quy tắc sở hữu, nó có thể từ chối một chương trình hợp lệ; theo cách này, nó mang tính thận trọng. Nếu Rust chấp nhận một chương trình không đúng, người dùng sẽ không thể tin tưởng vào các đảm bảo mà Rust đưa ra. Tuy nhiên, nếu Rust từ chối một chương trình hợp lệ, lập trình viên sẽ gặp bất tiện, nhưng không có gì thảm khốc xảy ra. Kiểu `RefCell<T>` hữu ích khi bạn chắc chắn mã của mình tuân thủ các quy tắc mượn nhưng trình biên dịch không thể hiểu và đảm bảo điều đó.

Tương tự như `Rc<T>`, `RefCell<T>` chỉ dùng trong các kịch bản đơn luồng và sẽ báo lỗi biên dịch nếu bạn cố gắng sử dụng trong bối cảnh đa luồng. Chúng ta sẽ nói về cách có được chức năng của `RefCell<T>` trong chương trình đa luồng ở Chương 16.

Dưới đây là tóm tắt lý do chọn `Box<T>`, `Rc<T>` hoặc `RefCell<T>`:

* `Rc<T>` cho phép nhiều chủ sở hữu cùng dữ liệu; `Box<T>` và `RefCell<T>` chỉ có một chủ sở hữu.
* `Box<T>` cho phép mượn có thể hoặc không thể thay đổi được kiểm tra tại thời điểm biên dịch; `Rc<T>` chỉ cho phép mượn không thay đổi được kiểm tra tại thời điểm biên dịch; `RefCell<T>` cho phép mượn có thể hoặc không thể thay đổi được kiểm tra tại thời điểm chạy.
* Vì `RefCell<T>` cho phép mượn có thể thay đổi được kiểm tra tại thời điểm chạy, bạn có thể thay đổi giá trị bên trong `RefCell<T>` ngay cả khi `RefCell<T>` là bất biến.

Việc thay đổi giá trị bên trong một giá trị bất biến là mẫu *interior mutability*. Hãy xem xét một tình huống mà interior mutability hữu ích và khám phá cách mà nó có thể thực hiện được.

### Interior Mutability: Mượn có thể thay đổi trên một giá trị bất biến

Một hệ quả của các quy tắc mượn là khi bạn có một giá trị bất biến, bạn không thể mượn nó theo cách có thể thay đổi. Ví dụ, đoạn mã này sẽ không biên dịch:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

Nếu bạn thử biên dịch đoạn mã này, bạn sẽ nhận được lỗi sau:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

Tuy nhiên, có những tình huống mà một giá trị có thể tự thay đổi trong 
các phương thức của nó nhưng vẫn xuất hiện như bất biến đối với các đoạn mã khác. 
Các đoạn mã bên ngoài các phương thức của giá trị sẽ không thể thay đổi giá trị đó. 
Sử dụng `RefCell<T>` là một cách để có thể thực hiện *interior mutability*, 
nhưng `RefCell<T>` không bỏ qua hoàn toàn các quy tắc mượn: trình kiểm tra mượn 
trong trình biên dịch cho phép nội bộ thay đổi này và các quy tắc mượn được kiểm tra tại thời gian chạy. 
Nếu bạn vi phạm các quy tắc này, bạn sẽ nhận được một `panic!` thay vì lỗi biên dịch.

Hãy cùng xem một ví dụ thực tế nơi chúng ta có thể sử dụng `RefCell<T>` để thay đổi 
một giá trị bất biến và hiểu tại sao điều đó lại hữu ích.

#### Trường hợp sử dụng Interior Mutability: Mock Objects

Đôi khi trong quá trình kiểm thử, lập trình viên sẽ sử dụng một kiểu thay cho một 
kiểu khác để quan sát hành vi cụ thể và xác nhận rằng nó được thực hiện đúng. 
Kiểu thay thế này được gọi là *test double*. Hãy tưởng tượng nó giống như một “stunt double” trong điện ảnh, 
nơi một người đóng thế cho diễn viên trong một cảnh khó. Test doubles đứng thay cho 
các kiểu khác khi chúng ta chạy kiểm thử. *Mock objects* là những loại test doubles 
cụ thể ghi lại những gì xảy ra trong quá trình kiểm thử để bạn có thể xác nhận rằng các hành động đúng đã diễn ra.

Rust không có các object theo cùng cách các ngôn ngữ khác có object, 
và Rust không có chức năng mock object tích hợp trong thư viện chuẩn như một số ngôn ngữ khác. 
Tuy nhiên, bạn hoàn toàn có thể tạo một struct phục vụ cùng mục đích như một mock object.

Dưới đây là kịch bản mà chúng ta sẽ kiểm thử: chúng ta sẽ tạo một thư viện theo dõi 
giá trị so với giá trị tối đa và gửi thông điệp dựa trên mức gần với giá trị tối đa 
mà giá trị hiện tại đạt được. Thư viện này có thể được dùng để theo dõi hạn mức API mà người dùng được phép gọi, ví dụ.

Thư viện của chúng ta chỉ cung cấp chức năng theo dõi mức gần với giá trị 
tối đa và các thông điệp nên được gửi khi nào. Ứng dụng sử dụng thư viện này 
sẽ chịu trách nhiệm cung cấp cơ chế gửi thông điệp: ứng dụng có thể lưu thông điệp, 
gửi email, gửi tin nhắn, hoặc thực hiện việc khác. Thư viện không cần biết chi tiết đó. 
Tất cả những gì nó cần là một thứ gì đó thực thi một trait mà chúng ta sẽ cung cấp gọi là `Messenger`. 
Listing 15-20 hiển thị mã thư viện:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

<span class="caption">Listing 15-20: Một thư viện để theo dõi mức gần với giá trị tối đa và cảnh báo khi giá trị đạt các mức nhất định</span>

Một phần quan trọng của mã này là trait `Messenger` có một phương thức gọi là `send` nhận một tham chiếu bất biến tới `self` và nội dung của thông điệp. Trait này là giao diện mà mock object của chúng ta cần thực thi để mock có thể được sử dụng giống như một object thật. Phần quan trọng khác là chúng ta muốn kiểm thử hành vi của phương thức `set_value` trên `LimitTracker`. Chúng ta có thể thay đổi giá trị truyền vào tham số `value`, nhưng `set_value` không trả về gì để chúng ta thực hiện các khẳng định. Chúng ta muốn có thể nói rằng nếu chúng ta tạo một `LimitTracker` với một thứ thực thi trait `Messenger` và một giá trị cụ thể cho `max`, khi truyền các số khác nhau cho `value`, messenger sẽ được yêu cầu gửi các thông điệp thích hợp.

Chúng ta cần một mock object mà thay vì gửi email hoặc tin nhắn khi gọi `send`, chỉ giữ lại các thông điệp mà nó được yêu cầu gửi. Chúng ta có thể tạo một instance mới của mock object, tạo một `LimitTracker` sử dụng mock object đó, gọi phương thức `set_value` trên `LimitTracker`, và sau đó kiểm tra xem mock object có các thông điệp mà chúng ta mong đợi không. Listing 15-21 trình bày một thử nghiệm triển khai mock object làm việc này, nhưng trình kiểm tra mượn sẽ không cho phép:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

<span class="caption">Listing 15-21: Một nỗ lực triển khai `MockMessenger` nhưng không được trình kiểm tra mượn cho phép</span>

Mã kiểm thử này định nghĩa một struct `MockMessenger` có trường `sent_messages` với một `Vec` chứa các giá trị `String` để theo dõi các thông điệp mà nó được yêu cầu gửi. Chúng ta cũng định nghĩa một hàm liên kết `new` để thuận tiện tạo các giá trị `MockMessenger` mới bắt đầu với danh sách thông điệp trống. Sau đó, chúng ta triển khai trait `Messenger` cho `MockMessenger` để có thể cung cấp một `MockMessenger` cho `LimitTracker`. Trong định nghĩa phương thức `send`, chúng ta lấy thông điệp truyền vào dưới dạng tham số và lưu nó vào danh sách `sent_messages` của `MockMessenger`.

Trong kiểm thử, chúng ta kiểm tra điều gì xảy ra khi `LimitTracker` được yêu cầu đặt `value` thành một giá trị lớn hơn 75% giá trị `max`. Trước tiên, chúng ta tạo một `MockMessenger` mới, bắt đầu với danh sách thông điệp trống. Sau đó, chúng ta tạo một `LimitTracker` mới và truyền cho nó một tham chiếu tới `MockMessenger` mới và một giá trị `max` bằng 100. Chúng ta gọi phương thức `set_value` trên `LimitTracker` với giá trị 80, lớn hơn 75% của 100. Sau đó, chúng ta khẳng định rằng danh sách thông điệp mà `MockMessenger` theo dõi bây giờ phải có một thông điệp.

Tuy nhiên, có một vấn đề với kiểm thử này, như được trình bày ở đây:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Chúng ta không thể sửa `MockMessenger` để theo dõi các thông điệp, vì phương thức `send` nhận một tham chiếu không thay đổi tới `self`. Chúng ta cũng không thể áp dụng gợi ý từ thông báo lỗi để dùng `&mut self`, vì khi đó chữ ký của `send` sẽ không khớp với chữ ký trong định nghĩa trait `Messenger` (bạn có thể thử và xem thông báo lỗi nhận được).

Đây là một tình huống mà *interior mutability* có thể giúp ích! Chúng ta sẽ lưu `sent_messages` bên trong một `RefCell<T>`, và sau đó phương thức `send` sẽ có thể sửa đổi `sent_messages` để lưu các thông điệp mà chúng ta đã nhận thấy. Listing 15-22 minh họa điều đó:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

<span class="caption">Listing 15-22: Sử dụng `RefCell<T>` để thay đổi giá trị bên trong trong khi giá trị bên ngoài được coi là bất biến</span>

Trường `sent_messages` bây giờ có kiểu `RefCell<Vec<String>>` thay vì `Vec<String>`. Trong hàm `new`, chúng ta tạo một thể hiện `RefCell<Vec<String>>` xung quanh vector rỗng.

Trong phần triển khai phương thức `send`, tham số đầu tiên vẫn là một borrow bất biến của `self`, khớp với định nghĩa trait. Chúng ta gọi `borrow_mut` trên `RefCell<Vec<String>>` trong `self.sent_messages` để lấy một tham chiếu mutable tới giá trị bên trong `RefCell<Vec<String>>`, chính là vector. Sau đó, chúng ta có thể gọi `push` trên tham chiếu mutable của vector để lưu lại các thông điệp được gửi trong quá trình test.

Thay đổi cuối cùng là trong assertion: để xem có bao nhiêu mục trong vector bên trong, chúng ta gọi `borrow` trên `RefCell<Vec<String>>` để lấy một tham chiếu bất biến tới vector.

Bây giờ bạn đã thấy cách sử dụng `RefCell<T>`, hãy tìm hiểu cách nó hoạt động!

#### Theo dõi borrow tại runtime với `RefCell<T>`

Khi tạo các tham chiếu bất biến và mutable, chúng ta dùng cú pháp `&` và `&mut`. Với `RefCell<T>`, chúng ta dùng các phương thức `borrow` và `borrow_mut`, là một phần của API an toàn thuộc về `RefCell<T>`. Phương thức `borrow` trả về smart pointer kiểu `Ref<T>`, và `borrow_mut` trả về smart pointer kiểu `RefMut<T>`. Cả hai kiểu đều implement `Deref`, vì vậy chúng ta có thể sử dụng chúng như các tham chiếu thông thường.

`RefCell<T>` theo dõi có bao nhiêu smart pointer `Ref<T>` và `RefMut<T>` đang hoạt động. Mỗi lần gọi `borrow`, `RefCell<T>` tăng số lượng immutable borrows đang hoạt động. Khi một giá trị `Ref<T>` ra khỏi scope, số lượng immutable borrows giảm đi một. Giống như các quy tắc borrow tại compile-time, `RefCell<T>` cho phép nhiều immutable borrows hoặc một mutable borrow tại bất kỳ thời điểm nào.

Nếu chúng ta cố vi phạm các quy tắc này, thay vì nhận lỗi biên dịch như với references, việc implement của `RefCell<T>` sẽ gây panic tại runtime. Listing 15-23 minh họa một sửa đổi của phương thức `send` trong Listing 15-22, chúng ta cố tình tạo hai mutable borrows cùng lúc để chứng minh rằng `RefCell<T>` ngăn chặn điều này tại runtime.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

<span class="caption">Listing 15-23: Tạo hai tham chiếu mutable trong cùng một scope để thấy rằng `RefCell<T>` sẽ panic</span>

Chúng ta tạo một biến `one_borrow` cho smart pointer `RefMut<T>` được trả về từ `borrow_mut`. 
Sau đó, chúng ta tạo một borrow mutable khác theo cách tương tự trong biến `two_borrow`. 
Điều này tạo ra hai tham chiếu mutable trong cùng một scope, điều mà không được phép. 
Khi chạy các test cho thư viện của chúng ta, mã trong Listing 15-23 sẽ biên dịch mà không gặp lỗi, nhưng test sẽ thất bại:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Lưu ý rằng mã đã panic với thông báo `already borrowed: BorrowMutError`. Đây là cách mà `RefCell<T>` xử lý việc vi phạm các quy tắc mượn (borrowing rules) tại thời điểm chạy.

Việc chọn kiểm tra lỗi mượn tại thời điểm chạy thay vì thời điểm biên dịch, như chúng ta đã làm ở đây, có nghĩa là bạn có thể sẽ phát hiện ra lỗi trong mã muộn hơn trong quá trình phát triển: có thể là cho đến khi mã của bạn được triển khai vào môi trường sản xuất. Ngoài ra, mã của bạn sẽ chịu một chút chi phí hiệu năng tại thời điểm chạy do phải theo dõi các lần mượn tại runtime thay vì compile time. Tuy nhiên, việc sử dụng `RefCell<T>` cho phép bạn viết một đối tượng mock có thể tự sửa đổi để theo dõi các thông điệp mà nó đã nhận trong khi bạn đang sử dụng nó trong một ngữ cảnh mà chỉ cho phép các giá trị bất biến (immutable). Bạn có thể sử dụng `RefCell<T>` bất chấp những đánh đổi này để có thêm chức năng hơn so với các tham chiếu thông thường.

### Sở hữu nhiều chủ sở hữu cho dữ liệu có thể thay đổi bằng cách kết hợp `Rc<T>` và `RefCell<T>`

Một cách phổ biến để sử dụng `RefCell<T>` là kết hợp với `Rc<T>`. Nhớ rằng `Rc<T>` cho phép bạn có nhiều chủ sở hữu cho cùng một dữ liệu, nhưng nó chỉ cung cấp quyền truy cập bất biến (immutable) tới dữ liệu đó. Nếu bạn có một `Rc<T>` chứa `RefCell<T>`, bạn sẽ có một giá trị vừa có thể có nhiều chủ sở hữu *vừa* có thể thay đổi được!

Ví dụ, hãy nhớ ví dụ danh sách cons trong Listing 15-18, nơi chúng ta sử dụng `Rc<T>` để cho phép nhiều danh sách chia sẻ quyền sở hữu của một danh sách khác. Vì `Rc<T>` chỉ chứa các giá trị bất biến, chúng ta không thể thay đổi bất kỳ giá trị nào trong danh sách sau khi đã tạo chúng. Hãy thêm `RefCell<T>` để có khả năng thay đổi các giá trị trong danh sách. Listing 15-24 cho thấy rằng bằng cách sử dụng `RefCell<T>` trong định nghĩa `Cons`, chúng ta có thể sửa đổi giá trị được lưu trữ trong tất cả các danh sách:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

<span class="caption">Listing 15-24: Sử dụng `Rc<RefCell<i32>>` để tạo một
`List` mà chúng ta có thể thay đổi</span>

Chúng ta tạo một giá trị là một thể hiện của `Rc<RefCell<i32>>` và lưu nó
vào biến tên là `value` để sau này có thể truy cập trực tiếp. Sau đó chúng
ta tạo một `List` trong `a` với biến thể `Cons` chứa `value`. Chúng ta cần
clone `value` để cả `a` và `value` đều sở hữu giá trị bên trong là `5`, thay
vì chuyển quyền sở hữu từ `value` sang `a` hoặc để `a` mượn từ `value`.

Chúng ta bao `List` `a` trong một `Rc<T>` để khi tạo các danh sách `b` và `c`,
cả hai đều có thể tham chiếu tới `a`, giống như những gì chúng ta đã làm
trong Listing 15-18.

Sau khi tạo các danh sách `a`, `b`, và `c`, chúng ta muốn cộng thêm 10
vào giá trị trong `value`. Chúng ta làm điều này bằng cách gọi `borrow_mut`
trên `value`, sử dụng tính năng dereferencing tự động mà chúng ta đã thảo
luận trong Chương 5 (xem phần [“Where’s the `->` Operator?”][wheres-the---operator]<!-- ignore -->)
để giải tham chiếu `Rc<T>` tới giá trị bên trong `RefCell<T>`. Phương thức
`borrow_mut` trả về một con trỏ thông minh `RefMut<T>`, và chúng ta sử dụng
toán tử dereference trên nó và thay đổi giá trị bên trong.

Khi in `a`, `b`, và `c`, chúng ta có thể thấy rằng tất cả đều có giá trị
đã được sửa thành 15 thay vì 5:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Kỹ thuật này khá hay ho! Bằng cách sử dụng `RefCell<T>`, chúng ta có một
giá trị `List` bên ngoài là bất biến. Nhưng chúng ta có thể dùng các phương
thức trên `RefCell<T>` cung cấp quyền truy cập vào khả năng thay đổi bên trong
nó để sửa đổi dữ liệu khi cần. Các kiểm tra tại thời gian chạy về các quy
tắc mượn bảo vệ chúng ta khỏi các race condition, và đôi khi đáng để đánh
đổi một chút tốc độ để có sự linh hoạt này trong cấu trúc dữ liệu. Lưu ý
rằng `RefCell<T>` không hoạt động cho mã đa luồng! `Mutex<T>` là phiên bản
an toàn cho luồng của `RefCell<T>` và chúng ta sẽ thảo luận về `Mutex<T>`
trong Chương 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
