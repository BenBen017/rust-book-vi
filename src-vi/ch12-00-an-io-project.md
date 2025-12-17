# Một Dự Án I/O: Xây Dựng Chương Trình Dòng Lệnh

Chương này tổng kết lại nhiều kỹ năng mà bạn đã học cho đến nay và khám phá một vài tính năng khác của thư viện chuẩn. Chúng ta sẽ xây dựng một công cụ dòng lệnh tương tác với tệp tin và nhập/xuất dòng lệnh để thực hành một số khái niệm Rust mà bạn đã nắm.

Tốc độ, an toàn, xuất ra một nhị phân duy nhất và hỗ trợ đa nền tảng của Rust khiến nó trở thành ngôn ngữ lý tưởng để tạo công cụ dòng lệnh. Vì vậy, trong dự án này, chúng ta sẽ tạo phiên bản riêng của công cụ tìm kiếm dòng lệnh cổ điển `grep` (**g**lobally search a **r**egular **e**xpression and **p**rint). Trong trường hợp sử dụng đơn giản nhất, `grep` tìm kiếm một tệp cụ thể với một chuỗi cụ thể. Để làm điều này, `grep` nhận hai đối số: đường dẫn tệp và chuỗi tìm kiếm. Sau đó nó đọc tệp, tìm các dòng chứa chuỗi đã cho, và in ra những dòng đó.

Trong quá trình đó, chúng ta sẽ minh họa cách làm cho công cụ dòng lệnh của mình sử dụng các tính năng terminal mà nhiều công cụ dòng lệnh khác cũng dùng. Chúng ta sẽ đọc giá trị của một biến môi trường để cho phép người dùng cấu hình hành vi của công cụ. Chúng ta cũng sẽ in thông báo lỗi ra luồng lỗi chuẩn (`stderr`) thay vì luồng xuất chuẩn (`stdout`), để ví dụ, người dùng có thể chuyển hướng output thành công vào một tệp trong khi vẫn thấy thông báo lỗi trên màn hình.

Một thành viên cộng đồng Rust, Andrew Gallant, đã tạo một phiên bản `grep` hoàn chỉnh, cực nhanh, gọi là `ripgrep`. So sánh với nó, phiên bản của chúng ta sẽ khá đơn giản, nhưng chương này sẽ cung cấp cho bạn kiến thức nền tảng cần thiết để hiểu một dự án thực tế như `ripgrep`.

Dự án `grep` của chúng ta sẽ kết hợp nhiều khái niệm mà bạn đã học:

* Tổ chức mã (sử dụng kiến thức về modules trong [Chương 7][ch7]<!-- ignore -->)
* Sử dụng vectors và strings (collections, [Chương 8][ch8]<!-- ignore -->)
* Xử lý lỗi ([Chương 9][ch9]<!-- ignore -->)
* Sử dụng traits và lifetimes khi phù hợp ([Chương 10][ch10]<!-- ignore -->)
* Viết các bài kiểm tra ([Chương 11][ch11]<!-- ignore -->)

Chúng ta cũng sẽ giới thiệu ngắn gọn về closures, iterators, và trait objects, mà các Chương [13][ch13]<!-- ignore --> và [17][ch17]<!-- ignore --> sẽ trình bày chi tiết.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch17]: ch17-00-oop.html
