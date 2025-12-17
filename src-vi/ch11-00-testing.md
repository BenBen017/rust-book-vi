# Viết Các Bài Kiểm Tra Tự Động

Trong bài luận năm 1972 “The Humble Programmer,” Edsger W. Dijkstra đã nói rằng “Kiểm thử chương trình có thể là một cách rất hiệu quả để phát hiện lỗi, nhưng hoàn toàn không đủ để chứng minh rằng không có lỗi.” Điều đó không có nghĩa là chúng ta không nên cố gắng kiểm thử càng nhiều càng tốt!

Độ chính xác trong chương trình là mức độ mà code của chúng ta thực hiện đúng những gì chúng ta dự định. Rust được thiết kế với sự quan tâm cao về độ chính xác của chương trình, nhưng độ chính xác là một vấn đề phức tạp và khó chứng minh. Hệ thống kiểu của Rust đảm nhận phần lớn gánh nặng này, nhưng hệ thống kiểu không thể phát hiện tất cả. Do đó, Rust hỗ trợ viết các bài kiểm tra phần mềm tự động.

Giả sử chúng ta viết một hàm `add_two` để cộng thêm 2 vào bất kỳ số nào được truyền vào. Chữ ký của hàm này nhận một số nguyên làm tham số và trả về một số nguyên. Khi chúng ta triển khai và biên dịch hàm đó, Rust thực hiện tất cả kiểm tra kiểu và kiểm tra mượn mà bạn đã học để đảm bảo rằng, ví dụ, chúng ta không truyền một giá trị `String` hoặc một reference không hợp lệ vào hàm này. Nhưng Rust *không thể* kiểm tra rằng hàm này sẽ thực sự thực hiện đúng điều chúng ta muốn, chẳng hạn trả về tham số cộng 2 thay vì tham số cộng 10 hoặc trừ 50! Đây là lúc các bài kiểm tra phát huy tác dụng.

Chúng ta có thể viết các bài kiểm tra để khẳng định, ví dụ, khi truyền `3` vào hàm `add_two`, giá trị trả về là `5`. Chúng ta có thể chạy các bài kiểm tra này bất cứ khi nào thay đổi code để đảm bảo rằng các hành vi đúng trước đó không bị thay đổi.

Kiểm thử là một kỹ năng phức tạp: mặc dù không thể bao quát tất cả chi tiết về cách viết các bài kiểm tra tốt trong một chương, chúng ta sẽ thảo luận về cơ chế của các tiện ích kiểm thử trong Rust. Chúng ta sẽ nói về các annotation và macro có sẵn khi viết kiểm tra, hành vi mặc định và các tùy chọn khi chạy kiểm tra, cũng như cách tổ chức kiểm tra thành unit tests và integration tests.
