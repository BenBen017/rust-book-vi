# Dự án cuối cùng: Xây dựng Web Server Đa Luồng

Chúng ta đã đi một chặng đường dài, và giờ đã đến cuối cuốn sách. Trong chương này, chúng ta sẽ xây dựng thêm một dự án nữa cùng nhau để minh họa một số khái niệm đã học trong các chương cuối, cũng như ôn lại một số bài học trước đó.

Trong dự án cuối cùng này, chúng ta sẽ tạo một web server trả về “hello” và hiển thị giống như Hình 20-1 trong trình duyệt web.

![hello from rust](img/trpl20-01.png)

<span class="caption">Hình 20-1: Dự án chia sẻ cuối cùng của chúng ta</span>

Kế hoạch xây dựng web server của chúng ta như sau:

1. Tìm hiểu một chút về TCP và HTTP.
2. Lắng nghe các kết nối TCP trên một socket.
3. Phân tích một số yêu cầu HTTP.
4. Tạo phản hồi HTTP đúng chuẩn.
5. Cải thiện throughput của server bằng một thread pool.

Trước khi bắt đầu, chúng ta nên nhắc một chi tiết: phương pháp chúng ta sử dụng sẽ không phải là cách tốt nhất để xây dựng web server với Rust. Thành viên cộng đồng đã xuất bản nhiều crate sẵn sàng sản xuất trên [crates.io](https://crates.io/) cung cấp các triển khai web server và thread pool đầy đủ hơn so với những gì chúng ta sẽ xây dựng. Tuy nhiên, mục tiêu của chúng ta trong chương này là để học, không phải chọn con đường dễ dàng. Vì Rust là một ngôn ngữ lập trình hệ thống, chúng ta có thể lựa chọn mức độ trừu tượng mà mình muốn làm việc và có thể đi xuống mức thấp hơn so với các ngôn ngữ khác. Do đó, chúng ta sẽ viết server HTTP cơ bản và thread pool một cách thủ công để bạn có thể học các ý tưởng và kỹ thuật chung đằng sau các crate mà bạn có thể sử dụng trong tương lai.
