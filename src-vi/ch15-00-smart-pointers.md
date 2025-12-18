# Smart Pointers

Một *pointer* là một khái niệm chung cho một biến chứa một địa chỉ trong bộ nhớ.
Địa chỉ này tham chiếu đến, hay "chỉ vào," một dữ liệu khác. Loại pointer phổ
biến nhất trong Rust là reference, mà bạn đã học trong Chương 4. References được
biểu thị bằng ký hiệu `&` và mượn giá trị mà chúng trỏ tới. Chúng không có
khả năng đặc biệt nào ngoài việc tham chiếu dữ liệu và không có chi phí overhead.

*Smart pointers*, ngược lại, là các cấu trúc dữ liệu hoạt động như một pointer
nhưng cũng có metadata và khả năng bổ sung. Khái niệm smart pointer không chỉ
có ở Rust: smart pointer bắt nguồn từ C++ và tồn tại trong các ngôn ngữ khác.
Rust có nhiều smart pointer được định nghĩa trong thư viện chuẩn cung cấp chức
năng vượt trội hơn so với references. Để khám phá khái niệm chung, chúng ta sẽ
xem một vài ví dụ về smart pointers, bao gồm một loại *reference counting*.
Loại pointer này cho phép dữ liệu có nhiều chủ sở hữu bằng cách theo dõi số lượng
chủ sở hữu, và khi không còn chủ sở hữu nào, sẽ dọn dẹp dữ liệu.

Rust, với khái niệm ownership và borrowing, có thêm sự khác biệt giữa references
và smart pointers: trong khi references chỉ mượn dữ liệu, trong nhiều trường
hợp, smart pointers *sở hữu* dữ liệu mà chúng trỏ tới.

Mặc dù chúng ta chưa gọi chúng là smart pointers trước đây, nhưng chúng ta đã
gặp một vài smart pointers trong cuốn sách này, bao gồm `String` và `Vec<T>`
trong Chương 8. Cả hai loại này được coi là smart pointers vì chúng sở hữu một
số bộ nhớ và cho phép bạn thao tác với nó. Chúng cũng có metadata và khả năng
bổ sung hoặc đảm bảo thêm. Ví dụ, `String` lưu trữ dung lượng của nó như
metadata và có khả năng đảm bảo dữ liệu luôn hợp lệ UTF-8.

Smart pointers thường được triển khai bằng struct. Không giống như một struct
bình thường, smart pointers triển khai các trait `Deref` và `Drop`. Trait `Deref`
cho phép một instance của smart pointer struct hoạt động như một reference để
bạn có thể viết code làm việc với references hoặc smart pointers. Trait `Drop`
cho phép bạn tùy chỉnh code được chạy khi một instance của smart pointer ra
khỏi scope. Trong chương này, chúng ta sẽ thảo luận về cả hai trait và chứng
minh lý do chúng quan trọng với smart pointers.

Với việc pattern smart pointer là một pattern thiết kế phổ biến được sử dụng
thường xuyên trong Rust, chương này sẽ không đề cập đến tất cả các smart
pointer hiện có. Nhiều thư viện có smart pointer riêng, và bạn thậm chí có thể
tự viết smart pointer của mình. Chúng ta sẽ đề cập đến các smart pointer phổ
biến trong thư viện chuẩn:

* `Box<T>` để cấp phát giá trị trên heap
* `Rc<T>`, một kiểu reference counting cho phép nhiều chủ sở hữu
* `Ref<T>` và `RefMut<T>`, truy cập thông qua `RefCell<T>`, một kiểu
  thực thi các quy tắc borrowing lúc runtime thay vì compile time

Ngoài ra, chúng ta sẽ thảo luận về pattern *interior mutability* nơi một kiểu
immutable cung cấp API để thay đổi giá trị bên trong. Chúng ta cũng sẽ bàn về
*các vòng tham chiếu (reference cycles)*: cách chúng có thể gây rò rỉ bộ nhớ
và cách phòng tránh.

Hãy cùng khám phá!