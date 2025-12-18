## Chạy Code Khi Dọn Dẹp với Trait `Drop`

Trait thứ hai quan trọng với mẫu thiết kế smart pointer là `Drop`, cho phép
bạn tùy chỉnh những gì xảy ra khi một giá trị sắp ra khỏi phạm vi. Bạn có
thể cung cấp một triển khai cho trait `Drop` trên bất kỳ kiểu nào, và mã đó
có thể được dùng để giải phóng tài nguyên như file hoặc kết nối mạng.

Chúng ta giới thiệu `Drop` trong bối cảnh smart pointer vì chức năng của
trait `Drop` hầu như luôn được sử dụng khi triển khai smart pointer. Ví
dụ, khi một `Box<T>` bị drop, nó sẽ giải phóng không gian trên heap mà box
trỏ tới.

Trong một số ngôn ngữ, với một số kiểu, lập trình viên phải gọi mã để
giải phóng bộ nhớ hoặc tài nguyên mỗi khi họ sử dụng xong một thể hiện
của các kiểu đó. Ví dụ bao gồm file handles, sockets, hoặc locks. Nếu họ
quên, hệ thống có thể bị quá tải và crash. Trong Rust, bạn có thể chỉ định
rằng một đoạn mã cụ thể sẽ chạy bất cứ khi nào một giá trị ra khỏi phạm
vi, và compiler sẽ tự động chèn đoạn mã đó. Kết quả là bạn không cần lo
lắng về việc đặt mã dọn dẹp ở khắp nơi trong chương trình khi một thể
hiện của một kiểu cụ thể kết thúc — bạn vẫn sẽ không bị rò rỉ tài
nguyên!

Bạn chỉ định đoạn mã chạy khi một giá trị ra khỏi phạm vi bằng cách triển
khai trait `Drop`. Trait `Drop` yêu cầu bạn triển khai một phương thức tên
là `drop` nhận một mutable reference tới `self`. Để thấy khi nào Rust
gọi `drop`, chúng ta sẽ triển khai `drop` với các lệnh `println!` trước
để quan sát.

Listing 15-14 hiển thị một struct `CustomSmartPointer` mà chức năng tùy
chỉnh duy nhất là in ra `Dropping CustomSmartPointer!` khi thể hiện ra
khỏi phạm vi, để minh họa khi Rust chạy hàm `drop`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

<span class="caption">Listing 15-14: Một struct `CustomSmartPointer` triển khai trait `Drop`, nơi chúng ta sẽ đặt code dọn dẹp</span>

Trait `Drop` đã được bao gồm trong prelude, nên chúng ta không cần phải đưa
nó vào phạm vi. Chúng ta triển khai trait `Drop` trên `CustomSmartPointer`
và cung cấp một triển khai cho phương thức `drop` gọi `println!`. Thân
hàm `drop` là nơi bạn sẽ đặt bất kỳ logic nào mà bạn muốn chạy khi một
thể hiện của kiểu của bạn ra khỏi phạm vi. Ở đây chúng ta in ra một số
văn bản để trực quan minh họa khi Rust sẽ gọi `drop`.

Trong hàm `main`, chúng ta tạo hai thể hiện của `CustomSmartPointer` và
sau đó in ra `CustomSmartPointers created`. Ở cuối `main`, các thể hiện
của `CustomSmartPointer` sẽ ra khỏi phạm vi, và Rust sẽ gọi đoạn code
chúng ta đặt trong phương thức `drop`, in ra thông điệp cuối cùng. Lưu
ý rằng chúng ta không cần phải gọi phương thức `drop` một cách rõ ràng.

Khi chạy chương trình này, chúng ta sẽ thấy đầu ra như sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust tự động gọi `drop` cho chúng ta khi các thể hiện ra khỏi phạm vi,
thực thi đoạn code mà chúng ta đã chỉ định. Các biến được drop theo thứ tự
ngược lại với thứ tự tạo ra, nên `d` được drop trước `c`. Mục đích của
ví dụ này là để cung cấp một hướng dẫn trực quan về cách hoạt động của
phương thức `drop`; thường thì bạn sẽ đặt code dọn dẹp mà kiểu của bạn
cần chạy thay vì một thông điệp in ra.

### Drop một giá trị sớm với `std::mem::drop`

Thật không may, việc tắt chức năng `drop` tự động không đơn giản. Thông
thường, việc tắt `drop` không cần thiết; toàn bộ mục đích của trait
`Drop` là nó được thực hiện tự động. Tuy nhiên, đôi khi bạn có thể muốn
dọn dẹp một giá trị sớm. Một ví dụ là khi sử dụng smart pointer quản lý
lock: bạn có thể muốn buộc phương thức `drop` giải phóng lock để các
đoạn code khác trong cùng phạm vi có thể lấy lock. Rust không cho phép
bạn gọi trực tiếp phương thức `drop` của trait `Drop`; thay vào đó, bạn
phải gọi hàm `std::mem::drop` do thư viện chuẩn cung cấp nếu muốn buộc
một giá trị bị drop trước khi kết thúc phạm vi của nó.

Nếu chúng ta thử gọi phương thức `drop` của trait `Drop` một cách thủ
công bằng cách sửa hàm `main` từ Listing 15-14, như trong Listing 15-15,
chúng ta sẽ nhận được lỗi biên dịch:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

<span class="caption">Listing 15-15: Thử gọi phương thức `drop` từ
trait `Drop` một cách thủ công để dọn dẹp sớm</span>

Khi chúng ta cố gắng biên dịch đoạn code này, chúng ta sẽ nhận được lỗi sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Thông báo lỗi này cho biết rằng chúng ta không được phép gọi `drop` một cách rõ ràng. Thông báo lỗi sử dụng thuật ngữ *destructor*, đây là thuật ngữ lập trình chung để chỉ một hàm dọn dẹp một thể hiện. Một *destructor* tương tự như một *constructor*, nhưng thay vì tạo thể hiện thì nó dọn dẹp thể hiện đó. Hàm `drop` trong Rust là một destructor cụ thể.

Rust không cho phép chúng ta gọi `drop` trực tiếp vì Rust vẫn sẽ tự động gọi `drop` trên giá trị vào cuối `main`. Điều này sẽ gây ra lỗi *double free* vì Rust sẽ cố gắng dọn dẹp cùng một giá trị hai lần.

Chúng ta không thể vô hiệu hóa việc chèn tự động `drop` khi một giá trị ra khỏi phạm vi, và cũng không thể gọi phương thức `drop` một cách rõ ràng. Vì vậy, nếu cần ép một giá trị được dọn dẹp sớm, chúng ta sử dụng hàm `std::mem::drop`.

Hàm `std::mem::drop` khác với phương thức `drop` trong trait `Drop`. Chúng ta gọi nó bằng cách truyền giá trị muốn ép dọn dẹp làm đối số. Hàm này có sẵn trong prelude, vì vậy chúng ta có thể chỉnh sửa `main` trong Listing 15-15 để gọi hàm `drop`, như minh họa trong Listing 15-16:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

<span class="caption">Listing 15-16: Gọi `std::mem::drop` để dọn dẹp một giá trị một cách rõ ràng trước khi nó ra khỏi phạm vi</span>

Chạy đoạn mã này sẽ in ra kết quả sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

Văn bản ```Dropping CustomSmartPointer with data `some data`!``` được in ra 
giữa các thông báo `CustomSmartPointer created.` và `CustomSmartPointer dropped before the end of main.`, 
cho thấy mã trong phương thức `drop` được gọi để hủy `c` tại thời điểm đó.

Bạn có thể sử dụng mã được xác định trong phần triển khai trait `Drop` theo nhiều cách 
để làm cho việc dọn dẹp thuận tiện và an toàn: ví dụ, bạn có thể dùng nó để tạo bộ 
cấp phát bộ nhớ riêng! Với trait `Drop` và hệ thống ownership của Rust, bạn không cần phải nhớ để dọn dẹp vì Rust sẽ tự động làm điều đó.

Bạn cũng không cần phải lo lắng về các vấn đề phát sinh từ việc vô tình dọn dẹp các 
giá trị vẫn đang được sử dụng: hệ thống ownership đảm bảo các tham chiếu luôn hợp lệ 
cũng đảm bảo rằng `drop` chỉ được gọi một lần khi giá trị không còn được sử dụng.

Bây giờ, sau khi đã xem xét `Box<T>` và một số đặc điểm của smart pointer, hãy cùng 
tìm hiểu một vài smart pointer khác được định nghĩa trong thư viện chuẩn.
