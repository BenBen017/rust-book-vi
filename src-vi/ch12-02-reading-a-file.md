## Đọc Tệp

Bây giờ chúng ta sẽ thêm chức năng để đọc tệp được chỉ định trong tham số `file_path`. Trước tiên, chúng ta cần một tệp mẫu để thử: chúng ta sẽ dùng một tệp có một lượng nhỏ văn bản trên nhiều dòng với một số từ lặp lại. Listing 12-3 có một bài thơ của Emily Dickinson rất phù hợp! Hãy tạo một tệp có tên *poem.txt* ở cấp gốc của dự án, và nhập bài thơ “I’m Nobody! Who are you?” vào.

<span class="filename">Filename: poem.txt</span>

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

<span class="caption">Listing 12-3: Một bài thơ của Emily Dickinson là ví dụ thử nghiệm tốt</span>

Khi văn bản đã sẵn sàng, hãy chỉnh sửa *src/main.rs* và thêm mã để đọc tệp, như được trình bày trong Listing 12-4.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

<span class="caption">Listing 12-4: Đọc nội dung của tệp được chỉ định bởi tham số thứ hai</span>

Trước tiên, chúng ta đưa vào phần liên quan của thư viện chuẩn bằng câu lệnh `use`: chúng ta cần `std::fs` để xử lý tệp.

Trong `main`, câu lệnh mới `fs::read_to_string` nhận `file_path`, mở tệp đó, và trả về một `std::io::Result<String>` chứa nội dung của tệp.

Sau đó, chúng ta lại thêm một câu lệnh tạm thời `println!` in giá trị của `contents` sau khi tệp được đọc, để kiểm tra xem chương trình hoạt động đến đâu.

Hãy chạy mã này với bất kỳ chuỗi nào làm tham số dòng lệnh đầu tiên (vì chúng ta chưa triển khai phần tìm kiếm) và tệp *poem.txt* làm tham số thứ hai:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Tuyệt vời! Mã đã đọc và in ra nội dung của tệp. Nhưng mã này còn một vài điểm chưa tốt. Hiện tại, hàm `main` đảm nhận nhiều trách nhiệm: nhìn chung, các hàm sẽ rõ ràng và dễ bảo trì hơn nếu mỗi hàm chỉ chịu trách nhiệm cho một ý tưởng duy nhất. Vấn đề khác là chúng ta chưa xử lý lỗi tốt nhất có thể. Chương trình vẫn còn nhỏ, nên những điểm này chưa phải là vấn đề lớn, nhưng khi chương trình phát triển, việc sửa chúng sẽ khó khăn hơn. Thực hành tốt là bắt đầu tái cấu trúc (refactor) sớm khi phát triển chương trình, vì việc refactor một lượng mã nhỏ dễ dàng hơn nhiều. Chúng ta sẽ làm việc đó ngay sau đây.
