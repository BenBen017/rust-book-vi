## Concurrency Có Thể Mở Rộng với các Trait `Sync` và `Send`

Thật thú vị là ngôn ngữ Rust có *rất* ít tính năng concurrency được tích hợp
trực tiếp. Gần như mọi khái niệm concurrency mà chúng ta đã nói tới trong
chương này đều thuộc về thư viện chuẩn, chứ không phải bản thân ngôn ngữ. Các
lựa chọn để xử lý concurrency của bạn không bị giới hạn trong ngôn ngữ hay thư
viện chuẩn; bạn hoàn toàn có thể tự viết các cơ chế concurrency của riêng mình
hoặc sử dụng những cơ chế do người khác viết.

Tuy nhiên, có hai khái niệm concurrency được nhúng trực tiếp vào ngôn ngữ: các
marker trait `Sync` và `Send` trong module `std::marker`.

### Cho Phép Chuyển Quyền Sở Hữu Giữa Các Thread với `Send`

Marker trait `Send` cho biết rằng quyền sở hữu của các giá trị thuộc kiểu triển
khai `Send` có thể được chuyển giữa các thread. Hầu như mọi kiểu dữ liệu trong
Rust đều là `Send`, nhưng vẫn có một số ngoại lệ, trong đó có `Rc<T>`: kiểu này
không thể là `Send` bởi vì nếu bạn clone một giá trị `Rc<T>` và cố gắng chuyển
quyền sở hữu của bản clone đó sang một thread khác, thì cả hai thread có thể
cập nhật bộ đếm tham chiếu cùng lúc. Vì lý do này, `Rc<T>` được triển khai để
sử dụng trong các tình huống single-thread, nơi bạn không muốn trả chi phí hiệu
năng cho tính an toàn với thread.

Do đó, hệ thống kiểu dữ liệu và các trait bound của Rust đảm bảo rằng bạn không
thể vô tình gửi một giá trị `Rc<T>` qua các thread theo cách không an toàn. Khi
chúng ta cố gắng làm điều này trong Listing 16-14, chúng ta đã nhận được lỗi
`the trait Send is not implemented for Rc<Mutex<i32>>`. Khi chúng ta chuyển sang
dùng `Arc<T>`, vốn là `Send`, thì đoạn mã đã biên dịch thành công.

Bất kỳ kiểu dữ liệu nào được cấu thành hoàn toàn từ các kiểu `Send` cũng sẽ tự
động được đánh dấu là `Send`. Hầu hết các kiểu primitive đều là `Send`, ngoại
trừ raw pointer, thứ mà chúng ta sẽ bàn tới trong Chương 19.

### Cho Phép Truy Cập Từ Nhiều Thread với `Sync`

Marker trait `Sync` cho biết rằng kiểu dữ liệu triển khai `Sync` là an toàn khi
được tham chiếu từ nhiều thread. Nói cách khác, một kiểu `T` là `Sync` nếu `&T`
(một immutable reference tới `T`) là `Send`, tức là reference đó có thể được gửi
sang một thread khác một cách an toàn. Tương tự như `Send`, các kiểu primitive
là `Sync`, và các kiểu được cấu thành hoàn toàn từ các kiểu `Sync` cũng sẽ là
`Sync`.

Smart pointer `Rc<T>` cũng không phải là `Sync` vì những lý do giống như việc nó
không phải là `Send`. Kiểu `RefCell<T>` (mà chúng ta đã nói tới trong Chương 15)
và họ các kiểu liên quan `Cell<T>` không phải là `Sync`. Cơ chế kiểm tra borrow
mà `RefCell<T>` thực hiện tại runtime là không an toàn với thread. Smart pointer
`Mutex<T>` thì là `Sync` và có thể được dùng để chia sẻ quyền truy cập giữa
nhiều thread, như bạn đã thấy trong phần
[“Chia Sẻ `Mutex<T>` Giữa Nhiều Thread”][sharing-a-mutext-between-multiple-threads]<!-- ignore -->.

### Việc Tự Triển Khai `Send` và `Sync` Là Không An Toàn

Bởi vì các kiểu dữ liệu được tạo thành từ những thành phần `Send` và `Sync` sẽ
tự động cũng là `Send` và `Sync`, nên chúng ta không cần phải tự tay triển khai
những trait này. Là các marker trait, chúng thậm chí còn không có phương thức
nào để triển khai. Chúng chỉ đơn thuần hữu ích trong việc đảm bảo các bất biến
(invariant) liên quan đến concurrency.

Việc triển khai thủ công các trait này đồng nghĩa với việc viết mã Rust không
an toàn (unsafe). Chúng ta sẽ nói về việc sử dụng unsafe Rust trong Chương 19;
còn tại thời điểm này, điều quan trọng cần biết là việc xây dựng các kiểu dữ
liệu đồng thời mới, không được cấu thành từ các thành phần `Send` và `Sync`, đòi
hỏi phải suy nghĩ rất cẩn trọng để duy trì các đảm bảo an toàn. [“The
Rustonomicon”][nomicon] có thêm thông tin về những đảm bảo này và cách duy trì
chúng.

## Tóm Tắt

Đây chưa phải là lần cuối cùng bạn gặp concurrency trong cuốn sách này: dự án ở
Chương 20 sẽ sử dụng các khái niệm trong chương này trong một bối cảnh thực tế
hơn so với những ví dụ nhỏ đã được thảo luận ở đây.

Như đã đề cập trước đó, bởi vì rất ít cách Rust xử lý concurrency là một phần
của chính ngôn ngữ, nên nhiều giải pháp concurrency được triển khai dưới dạng
crate. Những crate này phát triển nhanh hơn so với thư viện chuẩn, vì vậy hãy
nhớ tìm kiếm trên mạng để cập nhật những crate hiện đại và tốt nhất hiện nay
để sử dụng trong các tình huống đa luồng.

Thư viện chuẩn của Rust cung cấp các channel để truyền thông điệp và các kiểu
smart pointer, chẳng hạn như `Mutex<T>` và `Arc<T>`, an toàn khi sử dụng trong
bối cảnh đồng thời. Hệ thống kiểu dữ liệu và borrow checker đảm bảo rằng đoạn mã
sử dụng các giải pháp này sẽ không rơi vào tình trạng data race hay reference
không hợp lệ. Một khi bạn đã làm cho chương trình biên dịch thành công, bạn có
thể yên tâm rằng nó sẽ chạy ổn định trên nhiều thread mà không gặp phải những
lỗi khó truy vết thường thấy ở các ngôn ngữ khác. Lập trình đồng thời không còn
là một khái niệm đáng sợ nữa: hãy tiến lên và làm cho chương trình của bạn trở
nên đồng thời, một cách không sợ hãi!

Tiếp theo, chúng ta sẽ nói về những cách làm mang tính idiomatic để mô hình hóa
vấn đề và cấu trúc lời giải khi chương trình Rust của bạn ngày càng lớn. Ngoài
ra, chúng ta cũng sẽ thảo luận về cách các idiom của Rust liên hệ với những gì
bạn có thể đã quen thuộc trong lập trình hướng đối tượng.

[sharing-a-mutext-between-multiple-threads]:
ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
