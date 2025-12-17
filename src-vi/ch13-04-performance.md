## So sánh hiệu năng: Vòng lặp vs. Iterator

Để quyết định nên dùng vòng lặp hay iterator, bạn cần biết phiên bản nào
nhanh hơn: phiên bản hàm `search` dùng vòng lặp `for` rõ ràng hay phiên bản
dùng iterator.

Chúng tôi đã chạy benchmark bằng cách tải toàn bộ nội dung *The Adventures of
Sherlock Holmes* của Sir Arthur Conan Doyle vào một `String` và tìm từ *the*
trong nội dung. Dưới đây là kết quả benchmark cho phiên bản `search` dùng
vòng lặp `for` và phiên bản dùng iterator:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

Phiên bản dùng iterator nhanh hơn một chút! Chúng tôi sẽ không giải thích chi tiết mã benchmark ở đây, vì mục đích không phải là chứng minh hai phiên bản hoàn toàn tương đương, mà chỉ để có cái nhìn tổng quan về hiệu năng của hai cách triển khai này.

Để benchmark toàn diện hơn, bạn nên thử với nhiều loại văn bản có kích thước khác nhau làm `contents`, các từ khác nhau với độ dài khác nhau làm `query`, và nhiều biến thể khác. Ý chính là: iterator, mặc dù là một trừu tượng cao cấp, được biên dịch xuống gần như cùng một mã máy như nếu bạn tự viết mã cấp thấp. Iterator là một trong những *zero-cost abstractions* của Rust, nghĩa là sử dụng trừu tượng này không gây thêm chi phí runtime. Điều này tương tự cách Bjarne Stroustrup, nhà thiết kế và hiện thực C++ gốc, định nghĩa *zero-overhead* trong “Foundations of C++” (2012):

> In general, C++ implementations obey the zero-overhead principle: What you
> don’t use, you don’t pay for. And further: What you do use, you couldn’t hand
> code any better.

Một ví dụ khác, đoạn mã sau lấy từ một bộ giải mã audio. Thuật toán giải mã sử dụng phép toán dự đoán tuyến tính để ước lượng các giá trị tương lai dựa trên hàm tuyến tính của các mẫu trước đó. Đoạn mã này sử dụng chuỗi iterator để tính toán trên ba biến trong phạm vi: một slice `buffer` chứa dữ liệu, một mảng 12 phần tử `coefficients`, và một lượng dịch dữ liệu trong `qlp_shift`. Chúng tôi đã khai báo các biến trong ví dụ này nhưng không gán giá trị; mặc dù mã này không có nhiều ý nghĩa ngoài ngữ cảnh của nó, nó vẫn là một ví dụ thực tế, cô đọng về cách Rust chuyển các ý tưởng cao cấp xuống mã cấp thấp.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

Để tính giá trị của `prediction`, đoạn mã này lặp qua từng giá trị trong 12 phần tử của `coefficients` và sử dụng phương thức `zip` để ghép các giá trị hệ số với 12 giá trị trước đó trong `buffer`. Sau đó, với mỗi cặp, chúng ta nhân các giá trị với nhau, cộng tất cả kết quả, và dịch các bit trong tổng lên phải `qlp_shift` bit.

Các phép tính trong các ứng dụng như bộ giải mã audio thường ưu tiên hiệu năng hàng đầu. Ở đây, chúng ta tạo một iterator, sử dụng hai adaptor, và sau đó tiêu thụ giá trị. Mã assembly mà Rust biên dịch ra từ đoạn mã này sẽ như thế nào? Tính đến thời điểm hiện tại, nó được biên dịch xuống cùng mã assembly mà bạn có thể viết tay. Không hề có vòng lặp tương ứng với việc lặp qua các giá trị trong `coefficients`: Rust biết có 12 lần lặp, nên nó *“unroll”* vòng lặp. *Unrolling* là một tối ưu loại bỏ chi phí overhead của mã điều khiển vòng lặp và thay vào đó tạo mã lặp lại cho từng lần lặp.

Tất cả các hệ số được lưu trong các thanh ghi, nghĩa là truy cập giá trị rất nhanh. Không có kiểm tra ranh giới mảng tại runtime. Tất cả các tối ưu hóa mà Rust áp dụng giúp mã kết quả cực kỳ hiệu quả. Bây giờ bạn đã hiểu điều này, bạn có thể sử dụng iterators và closures mà không lo lắng! Chúng khiến mã trông cao cấp hơn nhưng không gây ra bất kỳ phạt hiệu năng nào tại runtime.

## Tóm tắt

Closures và iterators là các tính năng của Rust được lấy cảm hứng từ các ngôn ngữ lập trình hàm. Chúng giúp Rust thể hiện rõ ràng các ý tưởng cao cấp với hiệu năng cấp thấp. Cách triển khai closures và iterators được thiết kế sao cho hiệu năng runtime không bị ảnh hưởng. Đây là một phần trong mục tiêu của Rust nhằm cung cấp các *zero-cost abstractions*.

Bây giờ, sau khi chúng ta đã cải thiện khả năng biểu đạt trong dự án I/O, hãy cùng xem một số tính năng khác của `cargo` sẽ giúp chúng ta chia sẻ dự án với thế giới.
