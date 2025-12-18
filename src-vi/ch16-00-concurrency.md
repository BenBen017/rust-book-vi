# Đồng Thời Không Sợ Hãi

Xử lý lập trình đồng thời một cách an toàn và hiệu quả là một trong những mục tiêu lớn của Rust. *Lập trình đồng thời* (concurrent programming), nơi các phần khác nhau của chương trình thực thi độc lập, và *lập trình song song* (parallel programming), nơi các phần khác nhau của chương trình thực thi cùng một lúc, đang trở nên ngày càng quan trọng khi nhiều máy tính tận dụng nhiều bộ xử lý của chúng. Trong lịch sử, lập trình trong những bối cảnh này rất khó khăn và dễ xảy ra lỗi: Rust hy vọng sẽ thay đổi điều đó.

Ban đầu, nhóm Rust nghĩ rằng đảm bảo an toàn bộ nhớ và ngăn ngừa các vấn đề đồng thời là hai thách thức riêng biệt cần được giải quyết bằng các phương pháp khác nhau. Theo thời gian, nhóm phát hiện rằng hệ thống ownership và type là một tập hợp công cụ mạnh mẽ giúp quản lý cả vấn đề an toàn bộ nhớ *và* đồng thời! Bằng cách tận dụng ownership và kiểm tra kiểu, nhiều lỗi đồng thời trở thành lỗi thời gian biên dịch trong Rust thay vì lỗi thời gian chạy. Do đó, thay vì bạn phải dành nhiều thời gian để tái hiện chính xác hoàn cảnh mà lỗi đồng thời xảy ra trong thời gian chạy, mã không đúng sẽ từ chối biên dịch và đưa ra thông báo lỗi giải thích vấn đề. Kết quả là bạn có thể sửa mã trong khi đang làm việc thay vì có thể sau khi nó đã được triển khai lên production. Chúng tôi đã đặt biệt danh cho khía cạnh này của Rust là *fearless concurrency* (đồng thời không sợ hãi). Đồng thời không sợ hãi cho phép bạn viết mã không có lỗi tinh vi và dễ dàng tái cấu trúc mà không gây ra lỗi mới.

> Lưu ý: Vì đơn giản, chúng tôi sẽ gọi nhiều vấn đề là *concurrent* thay vì chính xác hơn bằng cách nói *concurrent và/hoặc parallel*. Nếu cuốn sách này về concurrency và/hoặc parallelism, chúng tôi sẽ cụ thể hơn. Trong chương này, hãy tạm thời thay thế *concurrent và/hoặc parallel* bất cứ khi nào chúng tôi dùng *concurrent*.

Nhiều ngôn ngữ rất bảo thủ về các giải pháp mà chúng cung cấp để xử lý các vấn đề đồng thời. Ví dụ, Erlang có chức năng tinh tế cho đồng thời dựa trên truyền thông điệp nhưng chỉ có những cách mơ hồ để chia sẻ trạng thái giữa các luồng. Chỉ hỗ trợ một phần các giải pháp có thể là chiến lược hợp lý cho các ngôn ngữ cấp cao, vì ngôn ngữ cấp cao hứa hẹn lợi ích từ việc từ bỏ một số kiểm soát để có được các trừu tượng. Tuy nhiên, các ngôn ngữ cấp thấp được kỳ vọng cung cấp giải pháp có hiệu suất tốt nhất trong bất kỳ tình huống nào và có ít trừu tượng hơn trên phần cứng. Do đó, Rust cung cấp nhiều công cụ để mô hình hóa vấn đề theo bất kỳ cách nào phù hợp với tình huống và yêu cầu của bạn.

Các chủ đề chúng ta sẽ đề cập trong chương này:

* Cách tạo các luồng để chạy nhiều đoạn mã cùng một lúc
* Đồng thời dựa trên *truyền thông điệp* (message-passing concurrency), nơi các channel gửi thông điệp giữa các luồng
* Đồng thời dựa trên *trạng thái chia sẻ* (shared-state concurrency), nơi nhiều luồng có quyền truy cập vào một mảnh dữ liệu
* Các trait `Sync` và `Send`, mở rộng đảm bảo đồng thời của Rust cho các kiểu do người dùng định nghĩa cũng như các kiểu do thư viện chuẩn cung cấp
