# Patterns and Matching

*Patterns* là một cú pháp đặc biệt trong Rust dùng để so khớp với cấu trúc
của các kiểu dữ liệu, cả phức tạp lẫn đơn giản. Sử dụng patterns kết hợp với
các biểu thức `match` và các cấu trúc khác sẽ cho bạn nhiều quyền kiểm soát
hơn đối với luồng điều khiển của chương trình. Một pattern bao gồm sự
kết hợp của một số thành phần sau:

* Literals (hằng số)
* Các mảng, enum, struct, hoặc tuple được destructure
* Biến
* Wildcards (`_`)
* Placeholders

Một số ví dụ về pattern gồm `x`, `(a, 3)`, và `Some(Color::Red)`. Trong
các ngữ cảnh mà pattern hợp lệ, các thành phần này mô tả “hình dạng” của
dữ liệu. Chương trình sau đó sẽ so khớp các giá trị với pattern để xác định
xem giá trị có đúng “hình dạng” dữ liệu để chạy đoạn code tương ứng hay không.

Để sử dụng một pattern, chúng ta so sánh nó với một giá trị. Nếu pattern
khớp với giá trị, chúng ta có thể sử dụng các phần của giá trị đó trong code.
Hãy nhớ lại các biểu thức `match` trong Chương 6 mà đã sử dụng patterns,
chẳng hạn như ví dụ máy phân loại đồng xu. Nếu giá trị phù hợp với hình dạng
của pattern, chúng ta có thể dùng các phần được đặt tên. Nếu không, đoạn
code liên quan đến pattern đó sẽ không chạy.

Chương này là một tài liệu tham khảo về tất cả các vấn đề liên quan đến
patterns. Chúng ta sẽ đề cập đến các vị trí hợp lệ để sử dụng patterns,
sự khác nhau giữa refutable và irrefutable patterns, cũng như các loại cú
pháp pattern mà bạn có thể gặp. Kết thúc chương này, bạn sẽ biết cách sử
dụng patterns để biểu diễn nhiều khái niệm một cách rõ ràng.
