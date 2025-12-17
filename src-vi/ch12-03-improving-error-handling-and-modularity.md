## Tái Cấu Trúc Để Cải Thiện Tính Module và Xử Lý Lỗi

Để cải thiện chương trình, chúng ta sẽ khắc phục bốn vấn đề liên quan đến cấu trúc chương trình và cách xử lý các lỗi tiềm ẩn. Đầu tiên, hàm `main` hiện tại thực hiện hai nhiệm vụ: phân tích tham số và đọc tệp. Khi chương trình phát triển, số lượng nhiệm vụ riêng lẻ mà hàm `main` đảm nhận sẽ tăng lên. Khi một hàm càng nhiều trách nhiệm, nó càng khó lý giải, khó kiểm thử và khó thay đổi mà không làm hỏng một phần nào đó. Tốt nhất là tách chức năng để mỗi hàm chỉ chịu trách nhiệm cho một nhiệm vụ duy nhất.

Vấn đề này cũng liên quan đến vấn đề thứ hai: mặc dù `query` và `file_path` là các biến cấu hình cho chương trình, các biến như `contents` được dùng để thực hiện logic của chương trình. Khi `main` trở nên dài hơn, chúng ta sẽ cần nhiều biến hơn trong phạm vi; càng nhiều biến trong phạm vi, việc theo dõi mục đích của từng biến càng khó. Tốt nhất là nhóm các biến cấu hình thành một cấu trúc để làm rõ mục đích của chúng.

Vấn đề thứ ba là chúng ta đã dùng `expect` để in thông báo lỗi khi việc đọc tệp thất bại, nhưng thông báo lỗi chỉ in ra `Should have been able to read the file`. Việc đọc tệp có thể thất bại theo nhiều cách: ví dụ, tệp có thể bị mất, hoặc chúng ta có thể không có quyền mở tệp. Hiện tại, bất kể tình huống nào, chúng ta cũng in cùng một thông báo lỗi, điều này sẽ không cung cấp thông tin gì cho người dùng!

Thứ tư, chúng ta dùng `expect` lặp đi lặp lại để xử lý các lỗi khác nhau, và nếu người dùng chạy chương trình mà không cung cấp đủ tham số, họ sẽ nhận được lỗi `index out of bounds` từ Rust, mà không giải thích rõ vấn đề. Sẽ tốt hơn nếu tất cả mã xử lý lỗi được đặt ở một chỗ, để các người bảo trì trong tương lai chỉ cần tham khảo một nơi nếu logic xử lý lỗi cần thay đổi. Việc đặt tất cả mã xử lý lỗi ở một chỗ cũng đảm bảo rằng chúng ta in các thông điệp có ý nghĩa với người dùng cuối.

Hãy giải quyết bốn vấn đề này bằng cách tái cấu trúc dự án của chúng ta.

### Tách Biệt Trách Nhiệm Cho Các Dự Án Binary

Vấn đề tổ chức khi phân bổ trách nhiệm cho nhiều nhiệm vụ vào hàm `main` là phổ biến trong nhiều dự án binary. Do đó, cộng đồng Rust đã phát triển các hướng dẫn để tách các mối quan tâm riêng biệt của một chương trình binary khi `main` bắt đầu trở nên lớn. Quá trình này có các bước sau:

* Chia chương trình của bạn thành *main.rs* và *lib.rs* và chuyển logic của chương trình sang *lib.rs*.
* Miễn là logic phân tích tham số dòng lệnh còn nhỏ, nó có thể giữ lại trong *main.rs*.
* Khi logic phân tích tham số dòng lệnh bắt đầu phức tạp, tách nó ra khỏi *main.rs* và chuyển sang *lib.rs*.

Các trách nhiệm còn lại trong hàm `main` sau quá trình này nên giới hạn ở các mục sau:

* Gọi logic phân tích tham số dòng lệnh với các giá trị tham số
* Thiết lập bất kỳ cấu hình nào khác
* Gọi hàm `run` trong *lib.rs*
* Xử lý lỗi nếu `run` trả về lỗi

Mẫu này nhằm tách biệt các mối quan tâm: *main.rs* đảm nhận việc chạy chương trình, và *lib.rs* đảm nhận toàn bộ logic của nhiệm vụ. Vì bạn không thể kiểm thử trực tiếp hàm `main`, cấu trúc này cho phép bạn kiểm thử toàn bộ logic chương trình bằng cách chuyển nó vào các hàm trong *lib.rs*. Mã còn lại trong *main.rs* sẽ đủ nhỏ để xác minh độ chính xác chỉ bằng cách đọc. Hãy tái cấu trúc chương trình theo quá trình này.

#### Tách Hàm Phân Tích Tham Số

Chúng ta sẽ tách chức năng phân tích tham số vào một hàm mà `main` sẽ gọi, để chuẩn bị chuyển logic phân tích tham số dòng lệnh sang *src/lib.rs*. Listing 12-5 cho thấy phần đầu mới của `main` gọi một hàm mới `parse_config`, mà chúng ta sẽ định nghĩa trong *src/main.rs* tạm thời.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

<span class="caption">Listing 12-5: Tách hàm `parse_config` từ `main`</span>

Chúng ta vẫn đang thu thập các tham số dòng lệnh vào một vector, nhưng thay vì gán giá trị tham số tại chỉ số 1 vào biến `query` và giá trị tham số tại chỉ số 2 vào biến `file_path` trong hàm `main`, chúng ta truyền toàn bộ vector vào hàm `parse_config`. Hàm `parse_config` sau đó chứa logic xác định tham số nào sẽ vào biến nào và trả lại các giá trị cho `main`. Chúng ta vẫn tạo các biến `query` và `file_path` trong `main`, nhưng `main` không còn trách nhiệm xác định cách các tham số dòng lệnh và biến tương ứng với nhau.

Việc tái cấu trúc này có thể có vẻ quá mức đối với chương trình nhỏ của chúng ta, nhưng chúng ta đang refactor theo các bước nhỏ, từng phần. Sau khi thực hiện thay đổi này, hãy chạy lại chương trình để xác minh rằng việc phân tích tham số vẫn hoạt động. Việc kiểm tra tiến trình thường xuyên là tốt, giúp xác định nguyên nhân vấn đề khi chúng xảy ra.

#### Nhóm Các Giá Trị Cấu Hình

Chúng ta có thể thực hiện một bước nhỏ nữa để cải thiện hàm `parse_config`. Hiện tại, chúng ta đang trả về một tuple, nhưng ngay sau đó lại tách tuple đó thành các phần riêng lẻ. Đây là dấu hiệu rằng có lẽ chúng ta chưa có trừu tượng (abstraction) phù hợp.

Một chỉ báo khác cho thấy còn có thể cải thiện là phần `config` trong `parse_config`, cho thấy hai giá trị chúng ta trả về có liên quan và đều là một phần của một giá trị cấu hình. Hiện tại, chúng ta chỉ truyền đạt ý nghĩa này bằng cách nhóm hai giá trị vào một tuple; thay vào đó, chúng ta sẽ đưa hai giá trị vào một struct và đặt tên có ý nghĩa cho từng trường trong struct. Làm như vậy sẽ giúp những người bảo trì mã trong tương lai hiểu rõ cách các giá trị khác nhau liên quan với nhau và mục đích của chúng là gì.

Listing 12-6 trình bày các cải tiến cho hàm `parse_config`.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

<span class="caption">Listing 12-6: Tái cấu trúc `parse_config` để trả về một thể hiện của struct `Config`</span>

Chúng ta đã thêm một struct có tên `Config` với các trường `query` và `file_path`. Chữ ký của `parse_config` hiện nay cho biết nó trả về một giá trị `Config`. Trong thân của `parse_config`, nơi trước đây chúng ta trả về các string slice tham chiếu tới các giá trị `String` trong `args`, bây giờ chúng ta định nghĩa `Config` để chứa các giá trị `String` sở hữu (owned). Biến `args` trong `main` là chủ sở hữu của các giá trị tham số và chỉ cho phép hàm `parse_config` mượn chúng, điều này có nghĩa là chúng ta sẽ vi phạm quy tắc mượn của Rust nếu `Config` cố gắng chiếm quyền sở hữu các giá trị trong `args`.

Có nhiều cách để quản lý dữ liệu `String`; cách đơn giản nhất, mặc dù hơi kém hiệu quả, là gọi phương thức `clone` trên các giá trị. Điều này sẽ tạo một bản sao đầy đủ của dữ liệu để thể hiện `Config` sở hữu, mất nhiều thời gian và bộ nhớ hơn so với việc chỉ lưu tham chiếu tới dữ liệu chuỗi. Tuy nhiên, việc clone dữ liệu cũng làm cho mã của chúng ta rất trực quan vì chúng ta không phải quản lý thời gian sống của các tham chiếu; trong trường hợp này, đánh đổi một chút hiệu suất để đổi lấy sự đơn giản là xứng đáng.

> ### Những đánh đổi khi sử dụng `clone`
>
> Nhiều Rustacean có xu hướng tránh sử dụng `clone` để giải quyết vấn đề ownership vì chi phí thời gian chạy của nó. Trong [Chương 13][ch13]<!-- ignore -->, bạn sẽ học cách sử dụng các phương pháp hiệu quả hơn trong tình huống này. Nhưng hiện tại, sao chép một vài chuỗi để tiếp tục tiến triển là ổn vì bạn chỉ thực hiện các bản sao này một lần và đường dẫn tệp cùng chuỗi truy vấn rất nhỏ. Tốt hơn là có một chương trình hoạt động hơi kém hiệu quả hơn là cố gắng tối ưu hóa mã ngay lần đầu tiên. Khi bạn quen thuộc hơn với Rust, việc bắt đầu với giải pháp hiệu quả nhất sẽ dễ dàng hơn, nhưng hiện tại, việc gọi `clone` hoàn toàn chấp nhận được.

Chúng ta đã cập nhật `main` để đặt thể hiện `Config` được trả về từ `parse_config` vào một biến có tên `config`, và cập nhật mã trước đây sử dụng các biến riêng biệt `query` và `file_path` để bây giờ sử dụng các trường trong struct `Config`.

Bây giờ mã của chúng ta truyền đạt rõ ràng hơn rằng `query` và `file_path` có liên quan với nhau và mục đích của chúng là cấu hình cách chương trình hoạt động. Bất kỳ mã nào sử dụng các giá trị này biết rằng chúng nằm trong thể hiện `config` và được đặt trong các trường với tên phản ánh mục đích.

#### Tạo Constructor cho `Config`

Cho đến nay, chúng ta đã tách logic chịu trách nhiệm phân tích tham số dòng lệnh ra khỏi `main` và đặt nó trong hàm `parse_config`. Việc này giúp chúng ta thấy rằng các giá trị `query` và `file_path` có liên quan và mối quan hệ này nên được thể hiện trong mã. Chúng ta sau đó thêm một struct `Config` để đặt tên cho mục đích liên quan của `query` và `file_path` và có thể trả về các giá trị này với tên trường trong struct từ hàm `parse_config`.

Bây giờ, mục đích của hàm `parse_config` là tạo một thể hiện `Config`, vì vậy chúng ta có thể thay đổi `parse_config` từ một hàm thông thường thành một hàm `new` liên kết với struct `Config`. Việc này sẽ làm mã trở nên idiomatic hơn. Chúng ta có thể tạo các thể hiện của các kiểu trong thư viện chuẩn, như `String`, bằng cách gọi `String::new`. Tương tự, bằng cách đổi `parse_config` thành hàm `new` liên kết với `Config`, chúng ta sẽ có thể tạo các thể hiện của `Config` bằng cách gọi `Config::new`. Listing 12-7 trình bày các thay đổi cần thực hiện.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

<span class="caption">Listing 12-7: Chuyển `parse_config` thành `Config::new`</span>

Chúng ta đã cập nhật `main`, nơi trước đây gọi `parse_config`, để thay vào đó gọi `Config::new`. Chúng ta đã đổi tên `parse_config` thành `new` và chuyển nó vào trong một khối `impl`, liên kết hàm `new` với `Config`. Hãy thử biên dịch lại mã để đảm bảo nó hoạt động.

### Sửa Xử Lý Lỗi

Bây giờ chúng ta sẽ sửa cách xử lý lỗi. Hãy nhớ rằng việc truy cập các giá trị trong vector `args` tại chỉ số 1 hoặc 2 sẽ khiến chương trình panic nếu vector chứa ít hơn ba phần tử. Hãy thử chạy chương trình mà không có tham số nào; kết quả sẽ như sau:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

Dòng `index out of bounds: the len is 1 but the index is 1` là thông báo lỗi dành cho lập trình viên. Nó sẽ không giúp người dùng cuối hiểu họ nên làm gì. Hãy sửa điều đó ngay bây giờ.

#### Cải Thiện Thông Báo Lỗi

Trong Listing 12-8, chúng ta thêm một kiểm tra trong hàm `new` để xác minh rằng slice đủ dài trước khi truy cập chỉ số 1 và 2. Nếu slice không đủ dài, chương trình sẽ panic và hiển thị một thông báo lỗi tốt hơn.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

<span class="caption">Listing 12-8: Thêm kiểm tra số lượng tham số</span>

Mã này tương tự với [hàm `Guess::new` mà chúng ta đã viết trong Listing 9-13][ch9-custom-types]<!-- ignore -->, nơi chúng ta gọi `panic!` khi tham số `value` nằm ngoài phạm vi giá trị hợp lệ. Thay vì kiểm tra phạm vi giá trị ở đây, chúng ta kiểm tra rằng độ dài của `args` ít nhất là 3 và phần còn lại của hàm có thể hoạt động với giả định điều kiện này đã được thỏa mãn. Nếu `args` có ít hơn ba phần tử, điều kiện này sẽ đúng, và chúng ta gọi macro `panic!` để kết thúc chương trình ngay lập tức.

Với vài dòng mã bổ sung trong `new`, hãy chạy lại chương trình mà không có tham số nào để xem thông báo lỗi trông như thế nào bây giờ:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Kết quả này tốt hơn: chúng ta giờ có một thông báo lỗi hợp lý. Tuy nhiên, cũng có một số thông tin thừa mà chúng ta không muốn cung cấp cho người dùng. Có lẽ kỹ thuật mà chúng ta đã dùng trong Listing 9-13 không phải là cách tốt nhất ở đây: việc gọi `panic!` thích hợp hơn cho vấn đề lập trình hơn là vấn đề sử dụng, [như đã thảo luận trong Chương 9][ch9-error-guidelines]<!-- ignore -->. Thay vào đó, chúng ta sẽ dùng kỹ thuật khác mà bạn đã học trong Chương 9 — [trả về một `Result`][ch9-result]<!-- ignore -->, biểu thị thành công hoặc lỗi.

<!-- Old headings. Do not remove or links may break. -->
<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Trả Về `Result` Thay Vì Gọi `panic!`

Chúng ta có thể trả về một giá trị `Result`, trong đó sẽ chứa một thể hiện `Config` trong trường hợp thành công và mô tả vấn đề trong trường hợp lỗi. Chúng ta cũng sẽ đổi tên hàm từ `new` thành `build` vì nhiều lập trình viên kỳ vọng các hàm `new` sẽ không bao giờ thất bại. Khi `Config::build` truyền thông tin về `main`, chúng ta có thể dùng kiểu `Result` để báo hiệu có vấn đề xảy ra. Sau đó, chúng ta có thể thay đổi `main` để chuyển một biến thể `Err` thành lỗi thực tế hơn cho người dùng mà không kèm theo các thông tin về `thread 'main'` và `RUST_BACKTRACE` mà một lần gọi `panic!` gây ra.

Listing 12-9 trình bày các thay đổi cần thực hiện đối với giá trị trả về của hàm mà bây giờ chúng ta gọi là `Config::build` và phần thân hàm cần để trả về `Result`. Lưu ý rằng mã này sẽ không biên dịch cho đến khi chúng ta cập nhật `main`, điều này sẽ được thực hiện trong listing tiếp theo.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

<span class="caption">Listing 12-9: Trả về `Result` từ `Config::build`</span>

Hàm `build` của chúng ta trả về một `Result` với thể hiện `Config` trong trường hợp thành công và một `&'static str` trong trường hợp lỗi. Các giá trị lỗi của chúng ta sẽ luôn là literal string có thời gian sống `'static`.

Chúng ta đã thực hiện hai thay đổi trong thân hàm: thay vì gọi `panic!` khi người dùng không cung cấp đủ tham số, chúng ta bây giờ trả về một giá trị `Err`, và chúng ta đã bọc giá trị `Config` trong `Ok`. Những thay đổi này giúp hàm phù hợp với chữ ký kiểu mới của nó.

Việc trả về một giá trị `Err` từ `Config::build` cho phép hàm `main` xử lý giá trị `Result` trả về từ hàm `build` và kết thúc tiến trình một cách gọn gàng hơn trong trường hợp lỗi.

<!-- Old headings. Do not remove or links may break. -->
<a id="calling-confignew-and-handling-errors"></a>

#### Gọi `Config::build` và Xử Lý Lỗi

Để xử lý trường hợp lỗi và in thông báo thân thiện với người dùng, chúng ta cần cập nhật `main` để xử lý giá trị `Result` được trả về từ `Config::build`, như được trình bày trong Listing 12-10. Chúng ta cũng sẽ loại bỏ trách nhiệm kết thúc công cụ dòng lệnh với mã lỗi khác 0 khỏi `panic!` và thay vào đó tự triển khai thủ công. Mã thoát khác 0 là một quy ước để báo cho tiến trình gọi chương trình của chúng ta rằng chương trình đã kết thúc với trạng thái lỗi.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

<span class="caption">Listing 12-10: Thoát với mã lỗi nếu việc tạo `Config` thất bại</span>

Trong listing này, chúng ta đã sử dụng một phương thức mà chưa đề cập chi tiết: `unwrap_or_else`, được định nghĩa trên `Result<T, E>` bởi thư viện chuẩn. Việc sử dụng `unwrap_or_else` cho phép chúng ta định nghĩa một số xử lý lỗi tùy chỉnh, không dùng `panic!`. Nếu `Result` là một giá trị `Ok`, hành vi của phương thức này tương tự như `unwrap`: nó trả về giá trị bên trong mà `Ok` bọc. Tuy nhiên, nếu giá trị là `Err`, phương thức này sẽ gọi mã trong *closure*, là một hàm ẩn danh mà chúng ta định nghĩa và truyền như một tham số cho `unwrap_or_else`. Chúng ta sẽ tìm hiểu closures chi tiết hơn trong [Chương 13][ch13]<!-- ignore -->. Hiện tại, bạn chỉ cần biết rằng `unwrap_or_else` sẽ truyền giá trị bên trong của `Err`, trong trường hợp này là chuỗi tĩnh `"not enough arguments"` mà chúng ta đã thêm trong Listing 12-9, vào closure của chúng ta thông qua tham số `err` xuất hiện giữa hai dấu thẳng đứng. Mã trong closure có thể dùng giá trị `err` khi chạy.

Chúng ta đã thêm một dòng `use` mới để đưa `process` từ thư viện chuẩn vào phạm vi. Mã trong closure chạy khi xảy ra lỗi chỉ gồm hai dòng: in giá trị `err` và gọi `process::exit`. Hàm `process::exit` sẽ dừng chương trình ngay lập tức và trả về số được truyền làm mã trạng thái thoát. Điều này tương tự cách xử lý dựa trên `panic!` mà chúng ta đã dùng trong Listing 12-8, nhưng giờ đây chúng ta không nhận được tất cả thông tin thừa. Hãy thử nó:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Tuyệt vời! Kết quả này thân thiện hơn nhiều với người dùng của chúng ta.

### Tách Logic Ra Khỏi `main`

Bây giờ chúng ta đã hoàn tất việc refactor phân tích cấu hình, hãy chuyển sang logic của chương trình. Như đã nêu trong [“Tách Biệt Trách Nhiệm Cho Các Dự Án Binary”](#separation-of-concerns-for-binary-projects)<!-- ignore -->, chúng ta sẽ tách một hàm có tên `run` để chứa tất cả logic hiện đang nằm trong hàm `main` mà không liên quan đến việc thiết lập cấu hình hoặc xử lý lỗi. Khi hoàn tất, `main` sẽ ngắn gọn và dễ kiểm tra bằng cách đọc, và chúng ta sẽ có thể viết các bài kiểm tra cho toàn bộ logic còn lại.

Listing 12-11 trình bày hàm `run` đã được tách. Hiện tại, chúng ta chỉ thực hiện cải tiến nhỏ, từng bước bằng cách tách hàm. Chúng ta vẫn định nghĩa hàm này trong *src/main.rs*.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

<span class="caption">Listing 12-11: Tách hàm `run` chứa phần còn lại của logic chương trình</span>

Hàm `run` bây giờ chứa tất cả logic còn lại từ `main`, bắt đầu từ việc đọc file. Hàm `run` nhận thể hiện `Config` làm tham số.

#### Trả Lỗi từ Hàm `run`

Với logic chương trình còn lại đã được tách ra thành hàm `run`, chúng ta có thể cải thiện việc xử lý lỗi, giống như đã làm với `Config::build` trong Listing 12-9. Thay vì để chương trình panic bằng cách gọi `expect`, hàm `run` sẽ trả về một `Result<T, E>` khi có vấn đề xảy ra. Điều này cho phép chúng ta tiếp tục gom gọn logic xử lý lỗi vào `main` theo cách thân thiện với người dùng. Listing 12-12 trình bày các thay đổi cần thực hiện đối với chữ ký và thân hàm của `run`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

<span class="caption">Listing 12-12: Thay đổi hàm `run` để trả về `Result`</span>

Chúng ta đã thực hiện ba thay đổi quan trọng ở đây. Thứ nhất, chúng ta thay đổi kiểu trả về của hàm `run` thành `Result<(), Box<dyn Error>>`. Trước đây hàm này trả về kiểu unit `()`, và chúng ta giữ giá trị đó trong trường hợp `Ok`.

Đối với kiểu lỗi, chúng ta dùng *trait object* `Box<dyn Error>` (và đã đưa `std::error::Error` vào phạm vi với một câu lệnh `use` ở đầu). Chúng ta sẽ tìm hiểu trait object trong [Chương 17][ch17]<!-- ignore -->. Hiện tại, chỉ cần biết rằng `Box<dyn Error>` nghĩa là hàm sẽ trả về một kiểu triển khai trait `Error`, nhưng chúng ta không phải chỉ định loại cụ thể của giá trị trả về. Điều này cho phép chúng ta linh hoạt trả về các giá trị lỗi có thể khác nhau trong các trường hợp lỗi khác nhau. Từ khóa `dyn` là viết tắt của “dynamic”.

Thứ hai, chúng ta đã loại bỏ việc gọi `expect` và thay bằng toán tử `?`, như đã bàn trong [Chương 9][ch9-question-mark]<!-- ignore -->. Thay vì `panic!` khi có lỗi, `?` sẽ trả về giá trị lỗi từ hàm hiện tại cho hàm gọi xử lý.

Thứ ba, hàm `run` bây giờ trả về giá trị `Ok` trong trường hợp thành công. Chúng ta khai báo kiểu thành công của hàm `run` là `()` trong chữ ký, có nghĩa là chúng ta cần bọc giá trị unit trong `Ok`. Cú pháp `Ok(())` có vẻ hơi lạ lúc đầu, nhưng dùng `()` như thế này là cách idiomatic để biểu thị rằng chúng ta gọi `run` chỉ vì hiệu ứng phụ; nó không trả về giá trị mà chúng ta cần.

Khi chạy mã này, nó sẽ biên dịch nhưng sẽ hiển thị một cảnh báo:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust báo cho chúng ta rằng mã của chúng ta đã bỏ qua giá trị `Result` và giá trị `Result` có thể chỉ ra rằng đã xảy ra lỗi. Nhưng chúng ta không kiểm tra xem có lỗi hay không, và trình biên dịch nhắc nhở rằng có lẽ chúng ta muốn có một số mã xử lý lỗi ở đây! Hãy khắc phục vấn đề đó ngay bây giờ.

#### Xử Lý Lỗi Trả Về Từ `run` Trong `main`

Chúng ta sẽ kiểm tra lỗi và xử lý chúng bằng một kỹ thuật tương tự như đã dùng với `Config::build` trong Listing 12-10, nhưng có một chút khác biệt:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Chúng ta sử dụng `if let` thay vì `unwrap_or_else` để kiểm tra xem `run` có trả về giá trị `Err` không và gọi `process::exit(1)` nếu có. Hàm `run` không trả về một giá trị mà chúng ta muốn `unwrap` giống như `Config::build` trả về thể hiện `Config`. Vì `run` trả về `()` trong trường hợp thành công, chúng ta chỉ quan tâm đến việc phát hiện lỗi, nên không cần dùng `unwrap_or_else` để trả về giá trị đã được unwrap, vốn chỉ là `()`.

Thân hàm của `if let` và `unwrap_or_else` đều giống nhau trong cả hai trường hợp: chúng ta in lỗi và thoát chương trình.

### Tách Mã Thành Thư Viện (Library Crate)

Dự án `minigrep` của chúng ta trông khá ổn rồi! Bây giờ chúng ta sẽ tách file *src/main.rs* và chuyển một số mã vào file *src/lib.rs*. Bằng cách đó, chúng ta có thể viết test cho mã và có một file *src/main.rs* với ít trách nhiệm hơn.

Hãy chuyển tất cả mã không phải hàm `main` từ *src/main.rs* sang *src/lib.rs*:

* Định nghĩa hàm `run`
* Các câu lệnh `use` liên quan
* Định nghĩa của `Config`
* Định nghĩa hàm `Config::build`

Nội dung của *src/lib.rs* nên có chữ ký các hàm như trong Listing 12-13 (chúng tôi đã bỏ qua thân hàm để ngắn gọn). Lưu ý rằng mã này sẽ không biên dịch cho đến khi chúng ta chỉnh sửa *src/main.rs* trong Listing 12-14.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs:here}}
```

<span class="caption">Listing 12-13: Chuyển `Config` và `run` vào *src/lib.rs*</span>

Chúng ta đã sử dụng rộng rãi từ khóa `pub`: trên `Config`, trên các trường của nó và phương thức `build`, cũng như trên hàm `run`. Bây giờ chúng ta có một library crate với API công khai mà chúng ta có thể viết test!

Bây giờ chúng ta cần đưa mã đã chuyển vào *src/lib.rs* vào phạm vi của binary crate trong *src/main.rs*, như được trình bày trong Listing 12-14.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

<span class="caption">Listing 12-14: Sử dụng library crate `minigrep` trong *src/main.rs*</span>

Chúng ta thêm một dòng `use minigrep::Config` để đưa kiểu `Config` từ library crate vào phạm vi của binary crate, và tiền tố hàm `run` bằng tên crate của chúng ta. Bây giờ tất cả chức năng nên được kết nối và hoạt động. Chạy chương trình với `cargo run` và đảm bảo mọi thứ hoạt động đúng.

Hú! Đó là một khối lượng công việc khá lớn, nhưng chúng ta đã chuẩn bị tốt cho thành công trong tương lai. Bây giờ việc xử lý lỗi trở nên dễ dàng hơn, và chúng ta đã làm cho mã modular hơn. Hầu hết công việc của chúng ta từ đây về sau sẽ được thực hiện trong *src/lib.rs*.

Hãy tận dụng tính modular mới này bằng cách làm một việc mà trước đây sẽ khó với mã cũ nhưng giờ đây lại dễ dàng: chúng ta sẽ viết một số bài kiểm tra (tests)!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch17]: ch17-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
