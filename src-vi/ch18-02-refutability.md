## Tính có thể thất bại của Pattern: Liệu một Pattern có thể không khớp

Pattern có hai dạng: **có thể thất bại** (refutable) và **không thể thất bại** (irrefutable). Pattern sẽ khớp với mọi giá trị có thể được truyền vào gọi là **không thể thất bại**. Ví dụ, `x` trong câu lệnh `let x = 5;` vì `x` khớp với mọi giá trị và do đó không thể thất bại. 

Pattern có thể không khớp với một số giá trị cụ thể gọi là **có thể thất bại**. Ví dụ, `Some(x)` trong biểu thức `if let Some(x) = a_value` vì nếu giá trị trong biến `a_value` là `None` thay vì `Some`, pattern `Some(x)` sẽ không khớp.

Tham số của hàm, câu lệnh `let`, và vòng lặp `for` chỉ chấp nhận **pattern không thể thất bại**, vì chương trình không thể thực hiện việc gì có ý nghĩa nếu giá trị không khớp. Biểu thức `if let` và `while let` chấp nhận cả **pattern có thể thất bại** và **không thể thất bại**, nhưng trình biên dịch sẽ cảnh báo nếu dùng pattern không thể thất bại, bởi vì theo định nghĩa, các pattern này được thiết kế để xử lý khả năng thất bại: chức năng của một điều kiện phụ thuộc vào việc thực hiện khác nhau khi thành công hoặc thất bại.

Nhìn chung, bạn không cần quá lo lắng về sự khác biệt giữa **refutable** và **irrefutable**, nhưng bạn cần hiểu khái niệm này để phản ứng khi gặp thông báo lỗi liên quan. Trong các trường hợp đó, bạn sẽ cần thay đổi pattern hoặc cấu trúc sử dụng pattern, tùy thuộc vào hành vi mong muốn của mã.

Hãy xem ví dụ về những gì xảy ra khi chúng ta sử dụng **pattern có thể thất bại** ở nơi Rust yêu cầu **pattern không thể thất bại** và ngược lại. Listing 18-8 cho thấy một câu lệnh `let`, nhưng pattern được chỉ định là `Some(x)`, một pattern có thể thất bại. Như bạn có thể đoán, mã này sẽ không biên dịch được.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-08/src/main.rs:here}}
```

<span class="caption">Listing 18-8: Cố gắng sử dụng pattern có thể thất bại với `let`</span>

Nếu `some_option_value` là giá trị `None`, nó sẽ không khớp với pattern `Some(x)`, 
tức pattern này là có thể thất bại. Tuy nhiên, câu lệnh `let` chỉ chấp nhận pattern 
không thể thất bại, vì không có gì hợp lệ mà chương trình có thể thực hiện với giá trị `None`. 
Khi biên dịch, Rust sẽ báo lỗi vì chúng ta đã cố sử dụng pattern có thể thất bại ở nơi yêu cầu pattern không thể thất bại:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-08/output.txt}}
```

Vì chúng ta không bao phủ được (và cũng không thể bao phủ!) mọi giá trị hợp lệ với pattern `Some(x)`, Rust chính xác khi đưa ra lỗi biên dịch.

Nếu chúng ta có một pattern có thể thất bại ở nơi yêu cầu pattern không thể thất bại, ta có thể sửa bằng cách thay đổi đoạn mã sử dụng pattern: thay vì dùng `let`, ta dùng `if let`. Khi đó, nếu pattern không khớp, chương trình sẽ bỏ qua đoạn mã trong dấu ngoặc nhọn, cho phép tiếp tục thực thi hợp lệ. Listing 18-9 minh họa cách sửa mã trong Listing 18-8.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-09/src/main.rs:here}}
```

<span class="caption">Listing 18-9: Sử dụng `if let` và một khối với pattern có thể thất bại thay vì `let`</span>

Chúng ta đã tạo ra một “lối thoát” cho đoạn mã! Đoạn mã này hoàn toàn hợp lệ, mặc dù điều đó có nghĩa là chúng ta không thể dùng một pattern không thể thất bại mà không nhận được cảnh báo. Nếu chúng ta cung cấp cho `if let` một pattern luôn khớp, chẳng hạn như `x`, như trong Listing 18-10, trình biên dịch sẽ đưa ra cảnh báo.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-10/src/main.rs:here}}
```

<span class="caption">Listing 18-10: Cố gắng sử dụng một pattern không thể thất bại với `if let`</span>

Rust báo lỗi rằng việc dùng `if let` với một pattern không thể thất bại là vô nghĩa:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-10/output.txt}}
```

Vì lý do này, các nhánh (`arm`) trong `match` phải sử dụng các 
pattern có thể thất bại (refutable), ngoại trừ nhánh cuối cùng, 
vốn nên khớp với mọi giá trị còn lại bằng một pattern không thể thất bại (irrefutable). 
Rust cho phép chúng ta sử dụng một pattern không thể thất bại trong một `match` 
chỉ có một nhánh, nhưng cú pháp này không thực sự hữu ích và có thể được 
thay thế bằng một câu lệnh `let` đơn giản hơn.

Bây giờ, khi bạn đã biết được nơi có thể sử dụng patterns và sự khác 
nhau giữa pattern có thể thất bại và không thể thất bại, hãy 
cùng tìm hiểu tất cả cú pháp mà chúng ta có thể dùng để tạo patterns.
