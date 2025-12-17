# Các Tính Năng Ngôn Ngữ Hàm: Iterators và Closures

Thiết kế của Rust đã lấy cảm hứng từ nhiều ngôn ngữ và kỹ thuật hiện
có, và một ảnh hưởng đáng kể là *lập trình hàm* (functional programming).
Lập trình theo phong cách hàm thường bao gồm việc sử dụng các hàm như
giá trị bằng cách truyền chúng vào các đối số, trả về chúng từ các hàm
khác, gán chúng cho biến để thực thi sau này, và nhiều thao tác khác.

Trong chương này, chúng ta sẽ không tranh luận về vấn đề lập trình hàm
là gì hay không phải là gì mà sẽ thay vào đó thảo luận một số tính năng
của Rust tương tự như các tính năng trong nhiều ngôn ngữ thường được
gọi là hàm.

Cụ thể hơn, chúng ta sẽ tìm hiểu:

* *Closures*, một cấu trúc giống hàm mà bạn có thể lưu vào biến
* *Iterators*, một cách xử lý một chuỗi các phần tử
* Cách sử dụng closures và iterators để cải thiện dự án I/O trong Chương 12
* Hiệu suất của closures và iterators (Cảnh báo: chúng nhanh hơn bạn
  tưởng!)

Chúng ta đã tìm hiểu một số tính năng khác của Rust, chẳng hạn như pattern
matching và enums, cũng chịu ảnh hưởng từ phong cách hàm. Vì việc nắm vững
closures và iterators là một phần quan trọng để viết code Rust theo phong
cách idiomatic và nhanh, chúng ta sẽ dành toàn bộ chương này cho chúng.
