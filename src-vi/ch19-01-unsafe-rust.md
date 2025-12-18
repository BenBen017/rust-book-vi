## Unsafe Rust

Tất cả code mà chúng ta đã thảo luận cho đến giờ đều được Rust 
đảm bảo an toàn bộ nhớ tại thời điểm biên dịch. Tuy nhiên, 
Rust còn có một ngôn ngữ “ẩn” bên trong, không thực thi các đảm bảo về an toàn bộ nhớ này: 
nó được gọi là *unsafe Rust* và hoạt động giống như Rust bình thường, 
nhưng cung cấp cho chúng ta những “siêu quyền” bổ sung.

Unsafe Rust tồn tại vì bản chất, phân tích tĩnh là bảo thủ. 
Khi trình biên dịch cố gắng xác định xem code có tuân thủ các đảm bảo hay không, 
nó thà từ chối một số chương trình hợp lệ còn hơn chấp nhận một 
số chương trình không hợp lệ. Mặc dù code *có thể* ổn, nếu trình biên dịch 
Rust không có đủ thông tin để tự tin, nó sẽ từ chối code. Trong những trường hợp này, 
bạn có thể dùng code unsafe để nói với trình biên dịch: “Tin tôi đi, tôi biết mình đang làm gì.” 
Tuy nhiên, hãy cẩn thận, vì sử dụng unsafe Rust không đúng cách có thể gây ra 
các vấn đề liên quan đến an toàn bộ nhớ, như truy cập con trỏ null.

Một lý do khác khiến Rust có “bản sao unsafe” là vì phần cứng máy tính vốn không an toàn. 
Nếu Rust không cho phép các thao tác unsafe, bạn sẽ không thể thực hiện một số tác vụ nhất định. Rust cần cho phép bạn lập trình hệ thống mức thấp, như tương tác trực tiếp với hệ điều hành hoặc thậm chí viết hệ điều hành riêng. Làm việc với lập trình hệ thống mức thấp là một trong những mục tiêu của ngôn ngữ. Hãy khám phá những gì chúng ta có thể làm với unsafe Rust và cách thực hiện.

### Siêu Quyền Unsafe

Để chuyển sang unsafe Rust, dùng từ khóa `unsafe` và bắt đầu một khối mới chứa code unsafe. Trong unsafe Rust, bạn có thể thực hiện năm hành động mà trong Rust an toàn không được phép, gọi là *siêu quyền unsafe*. Những siêu quyền này bao gồm khả năng:

* Giải tham chiếu một con trỏ thô (raw pointer)
* Gọi một hàm hoặc phương thức unsafe
* Truy cập hoặc sửa đổi biến static có thể thay đổi (mutable static)
* Triển khai một trait unsafe
* Truy cập các trường của `union`

Điều quan trọng cần hiểu là `unsafe` không tắt bộ kiểm tra mượn (borrow checker) hay vô hiệu hóa bất kỳ kiểm tra an toàn nào khác của Rust: nếu bạn dùng một reference trong unsafe code, nó vẫn được kiểm tra. Từ khóa `unsafe` chỉ cho phép bạn truy cập năm tính năng trên mà compiler không kiểm tra an toàn bộ nhớ. Bạn vẫn có một mức độ an toàn nhất định bên trong khối unsafe.

Ngoài ra, `unsafe` không có nghĩa là code bên trong khối này chắc chắn nguy hiểm hay sẽ gây ra vấn đề an toàn bộ nhớ: ý định là người lập trình phải đảm bảo code trong khối `unsafe` truy cập bộ nhớ một cách hợp lệ.

Con người có thể mắc sai lầm, và lỗi sẽ xảy ra, nhưng việc yêu cầu năm thao tác unsafe phải nằm trong các khối đánh dấu `unsafe` giúp bạn biết rằng mọi lỗi liên quan đến an toàn bộ nhớ đều phải nằm trong khối `unsafe`. Giữ các khối `unsafe` càng nhỏ càng tốt; bạn sẽ biết ơn khi sau này phải điều tra lỗi bộ nhớ.

Để cô lập code unsafe càng nhiều càng tốt, tốt nhất là bao bọc code unsafe bên trong một abstraction an toàn và cung cấp một API an toàn, mà chúng ta sẽ thảo luận sau trong chương khi xem xét các hàm và phương thức unsafe. Một số phần của thư viện chuẩn được triển khai như các abstraction an toàn trên code unsafe đã được kiểm tra. Việc bọc code unsafe trong một abstraction an toàn ngăn việc sử dụng `unsafe` lan rộng ra tất cả các nơi mà bạn hoặc người dùng có thể muốn dùng chức năng được triển khai bằng code unsafe, bởi vì việc sử dụng abstraction an toàn là an toàn.

Bây giờ, hãy cùng xem từng siêu quyền unsafe. Chúng ta cũng sẽ xem một số abstraction cung cấp giao diện an toàn cho code unsafe.

### Giải Tham Chiếu Một Con Trỏ Thô

Trong Chương 4, trong phần [“Dangling References”][dangling-references]<!-- ignore -->, chúng ta đã đề cập rằng compiler đảm bảo các reference luôn hợp lệ. Unsafe Rust có hai loại mới gọi là *raw pointers*, tương tự như reference. Giống như reference, raw pointer có thể là immutable hoặc mutable, được viết là `*const T` và `*mut T`. Dấu sao ở đây không phải là toán tử dereference; nó là một phần của tên kiểu. Trong ngữ cảnh raw pointer, *immutable* có nghĩa là con trỏ không thể được gán trực tiếp sau khi được dereference.

Khác với reference và smart pointer, raw pointer:

* Cho phép bỏ qua các quy tắc mượn bằng cách có cả immutable và mutable pointer hoặc nhiều mutable pointer tới cùng một địa chỉ
* Không được đảm bảo trỏ tới bộ nhớ hợp lệ
* Có thể là null
* Không thực hiện bất kỳ việc dọn dẹp tự động nào

Bằng cách từ chối các đảm bảo mà Rust áp dụng, bạn có thể đánh đổi an toàn đã được đảm bảo để lấy hiệu năng cao hơn hoặc khả năng tương tác với ngôn ngữ khác hoặc phần cứng nơi các đảm bảo của Rust không áp dụng.

Listing 19-1 minh họa cách tạo một immutable và một mutable raw pointer từ các reference.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-01/src/main.rs:here}}
```

### Tạo Con Trỏ Thô từ Reference

Lưu ý rằng trong ví dụ này chúng ta **không dùng từ khóa `unsafe`**. Chúng ta có thể tạo raw pointer trong code an toàn; chỉ có điều không thể dereference raw pointer ngoài một khối `unsafe`, như bạn sẽ thấy ngay sau đây.

Chúng ta đã tạo raw pointer bằng cách dùng `as` để ép kiểu một immutable và một mutable reference thành các loại raw pointer tương ứng. Vì chúng được tạo trực tiếp từ các reference đã được đảm bảo hợp lệ, chúng ta biết những raw pointer này là hợp lệ, nhưng không thể giả định điều đó cho bất kỳ raw pointer nào khác.

Để minh họa, tiếp theo chúng ta sẽ tạo một raw pointer mà tính hợp lệ của nó không chắc chắn. Listing 19-2 sẽ cho thấy cách tạo raw pointer tới một địa chỉ bộ nhớ tùy ý. Việc sử dụng bộ nhớ tùy ý là undefined behavior: có thể có dữ liệu tại địa chỉ đó hoặc không, compiler có thể tối ưu code khiến không có truy cập bộ nhớ nào, hoặc chương trình có thể lỗi với segmentation fault. Thường thì không có lý do chính đáng để viết code như vậy, nhưng về mặt kỹ thuật là có thể.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-02/src/main.rs:here}}
```

### Ghi Nhớ về Raw Pointer

Nhớ rằng chúng ta có thể tạo raw pointer trong code an toàn, nhưng không thể dereference raw pointer và đọc dữ liệu mà nó trỏ tới. Trong Listing 19-3, chúng ta sẽ dùng toán tử dereference `*` trên raw pointer, và điều này yêu cầu phải nằm trong một khối `unsafe`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-03/src/main.rs:here}}
```

### Lý do sử dụng Raw Pointer

Việc tạo con trỏ không gây hại; chỉ khi chúng ta cố truy cập giá trị mà nó trỏ tới mới có thể gặp giá trị không hợp lệ.

Lưu ý rằng trong Listing 19-1 và 19-3, chúng ta đã tạo `*const i32` và `*mut i32` trỏ tới cùng một vị trí bộ nhớ chứa `num`. Nếu thay vào đó ta tạo tham chiếu immutable và mutable tới `num`, code sẽ không biên dịch vì luật sở hữu của Rust không cho phép một tham chiếu mutable cùng lúc với bất kỳ tham chiếu immutable nào. Với raw pointer, ta có thể tạo một con trỏ mutable và một con trỏ immutable tới cùng vị trí và thay đổi dữ liệu qua con trỏ mutable, có thể gây ra **data race**. Hãy cẩn thận!

Một trường hợp chính sử dụng raw pointer là khi giao tiếp với code C, hoặc khi xây dựng các abstraction an toàn mà borrow checker không hiểu được.

---

### Gọi Unsafe Function hoặc Method

Một loại thao tác khác trong khối unsafe là gọi **unsafe function**. Chúng trông giống như hàm thông thường, nhưng có thêm từ khóa `unsafe` trước phần định nghĩa. Từ khóa này chỉ ra rằng hàm có những yêu cầu mà ta cần đảm bảo khi gọi, vì Rust không thể chắc rằng ta đã đáp ứng các yêu cầu đó. Khi gọi một unsafe function trong khối `unsafe`, ta đồng nghĩa rằng đã đọc tài liệu hàm và chịu trách nhiệm đảm bảo các hợp đồng của hàm.

Ví dụ một hàm unsafe tên `dangerous` mà body không làm gì:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Chúng ta phải gọi hàm `dangerous` bên trong một khối `unsafe` riêng. Nếu chúng ta cố gắng gọi `dangerous` mà không có khối `unsafe`, chúng ta sẽ nhận được lỗi:

```console
{{#include ../listings/ch19-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Với khối `unsafe`, chúng ta đang khẳng định với Rust rằng chúng ta đã đọc tài liệu của hàm, hiểu cách sử dụng đúng, và đã xác minh rằng chúng ta đang tuân thủ hợp đồng của hàm đó.

Các thân hàm của các hàm `unsafe` về cơ bản là các khối `unsafe`, vì vậy để thực hiện các thao tác unsafe khác bên trong một hàm unsafe, chúng ta không cần phải thêm một khối `unsafe` khác.

#### Tạo một Safe Abstraction trên Unsafe Code

Chỉ vì một hàm chứa mã unsafe không có nghĩa là chúng ta phải đánh dấu toàn bộ hàm là `unsafe`. Thực tế, việc bao bọc mã `unsafe` trong một hàm an toàn là một trừu tượng phổ biến.

Ví dụ, hãy nghiên cứu hàm `split_at_mut` từ thư viện chuẩn, hàm này yêu cầu một số mã `unsafe`. Chúng ta sẽ khám phá cách triển khai nó. Phương thức an toàn này được định nghĩa trên các slice có thể thay đổi: nó nhận một slice và chia nó thành hai bằng cách tách slice tại chỉ số được truyền làm tham số. Danh sách 19-4 cho thấy cách sử dụng `split_at_mut`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-04/src/main.rs:here}}
```

<span class="caption">Listing 19-4: Sử dụng hàm an toàn `split_at_mut`</span>

Chúng ta không thể triển khai hàm này chỉ bằng Rust an toàn. Một thử nghiệm có thể trông giống như Listing 19-5, nhưng sẽ không biên dịch được. Để đơn giản, chúng ta sẽ triển khai `split_at_mut` như một hàm thay vì một phương thức và chỉ dành cho các slice của giá trị `i32` thay vì một kiểu tổng quát `T`.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-05/src/main.rs:here}}
```

<span class="caption">Listing 19-5: Một triển khai thử nghiệm của `split_at_mut` chỉ sử dụng Rust an toàn</span>

Hàm này trước tiên lấy tổng độ dài của slice. Sau đó, nó khẳng định rằng chỉ số được truyền làm tham số nằm trong slice bằng cách kiểm tra xem nó có nhỏ hơn hoặc bằng độ dài hay không. Việc khẳng định này có nghĩa là nếu chúng ta truyền một chỉ số lớn hơn độ dài để tách slice tại đó, hàm sẽ panic trước khi cố gắng sử dụng chỉ số đó.

Tiếp theo, chúng ta trả về hai slice có thể thay đổi trong một tuple: một từ đầu slice gốc đến chỉ số `mid` và một từ `mid` đến cuối slice.

Khi chúng ta cố gắng biên dịch mã trong Listing 19-5, chúng ta sẽ nhận được lỗi.

```console
{{#include ../listings/ch19-advanced-features/listing-19-05/output.txt}}
```

Trình kiểm tra mượn (borrow checker) của Rust không thể hiểu rằng chúng ta đang mượn các phần khác nhau của slice; nó chỉ biết rằng chúng ta đang mượn cùng một slice hai lần. Việc mượn các phần khác nhau của slice về cơ bản là hợp lệ vì hai slice này không chồng lấn, nhưng Rust không đủ thông minh để nhận ra điều này. Khi chúng ta biết mã là an toàn, nhưng Rust thì không, đó là lúc cần sử dụng mã `unsafe`.

Listing 19-6 cho thấy cách sử dụng một khối `unsafe`, một con trỏ thô (raw pointer), và một số lời gọi tới các hàm unsafe để thực hiện triển khai `split_at_mut`.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-06/src/main.rs:here}}
```

<span class="caption">Listing 19-6: Sử dụng mã unsafe trong triển khai hàm `split_at_mut`</span>

Nhớ lại từ phần [“The Slice Type”][the-slice-type]<!-- ignore --> trong Chương 4 rằng slice là một con trỏ tới dữ liệu và độ dài của slice. Chúng ta sử dụng phương thức `len` để lấy độ dài của slice và phương thức `as_mut_ptr` để truy cập con trỏ thô (raw pointer) của slice. Trong trường hợp này, vì chúng ta có một slice có thể thay đổi của các giá trị `i32`, `as_mut_ptr` trả về một con trỏ thô với kiểu `*mut i32`, mà chúng ta đã lưu vào biến `ptr`.

Chúng ta giữ lại khẳng định rằng chỉ số `mid` nằm trong slice. Sau đó, chúng ta đến phần mã unsafe: hàm `slice::from_raw_parts_mut` nhận một con trỏ thô và một độ dài, và nó tạo ra một slice. Chúng ta sử dụng hàm này để tạo một slice bắt đầu từ `ptr` với độ dài `mid`. Sau đó, chúng ta gọi phương thức `add` trên `ptr` với `mid` làm tham số để lấy một con trỏ thô bắt đầu từ `mid`, và chúng ta tạo một slice sử dụng con trỏ đó với số phần tử còn lại sau `mid` làm độ dài.

Hàm `slice::from_raw_parts_mut` là unsafe vì nó nhận một con trỏ thô và phải tin rằng con trỏ này là hợp lệ. Phương thức `add` trên con trỏ thô cũng là unsafe, vì nó phải tin rằng vị trí offset cũng là một con trỏ hợp lệ. Do đó, chúng ta phải đặt một khối `unsafe` xung quanh các lời gọi tới `slice::from_raw_parts_mut` và `add` để có thể gọi chúng. Bằng cách nhìn vào mã và thêm khẳng định rằng `mid` phải nhỏ hơn hoặc bằng `len`, chúng ta có thể chắc chắn rằng tất cả các con trỏ thô được sử dụng bên trong khối `unsafe` sẽ là con trỏ hợp lệ tới dữ liệu trong slice. Đây là cách sử dụng `unsafe` chấp nhận được và phù hợp.

Lưu ý rằng chúng ta không cần đánh dấu hàm `split_at_mut` kết quả là `unsafe`, và chúng ta có thể gọi hàm này từ Rust an toàn. Chúng ta đã tạo một lớp trừu tượng an toàn cho mã unsafe với triển khai của hàm sử dụng mã `unsafe` một cách an toàn, vì nó chỉ tạo ra các con trỏ hợp lệ từ dữ liệu mà hàm này có quyền truy cập.

Ngược lại, việc sử dụng `slice::from_raw_parts_mut` trong Listing 19-7 có thể gây crash khi slice được sử dụng. Mã này lấy một vị trí bộ nhớ tùy ý và tạo một slice dài 10.000 phần tử.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-07/src/main.rs:here}}
```

<span class="caption">Listing 19-7: Tạo một slice từ vị trí bộ nhớ tùy ý</span>

Chúng ta không sở hữu bộ nhớ tại vị trí tùy ý này, và không có đảm bảo rằng slice mà mã này tạo ra chứa các giá trị `i32` hợp lệ. Việc cố gắng sử dụng `values` như thể nó là một slice hợp lệ sẽ dẫn đến hành vi không xác định (undefined behavior).

#### Sử dụng các hàm `extern` để gọi mã bên ngoài

Đôi khi, mã Rust của bạn có thể cần tương tác với mã được viết bằng ngôn ngữ khác. Để làm điều này, Rust có từ khóa `extern` giúp tạo và sử dụng *Foreign Function Interface (FFI)*. FFI là một cách để một ngôn ngữ lập trình định nghĩa các hàm và cho phép một ngôn ngữ lập trình khác (foreign) gọi các hàm đó.

Listing 19-8 minh họa cách thiết lập tích hợp với hàm `abs` từ thư viện chuẩn C. Các hàm được khai báo trong các khối `extern` luôn là unsafe khi gọi từ mã Rust. Lý do là các ngôn ngữ khác không tuân thủ các quy tắc và đảm bảo của Rust, và Rust không thể kiểm tra chúng, vì vậy trách nhiệm đảm bảo an toàn thuộc về lập trình viên.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-08/src/main.rs}}
```

<span class="caption">Listing 19-8: Khai báo và gọi một hàm `extern` được định nghĩa trong ngôn ngữ khác</span>

Trong khối `extern "C"`, chúng ta liệt kê tên và chữ ký của các hàm bên ngoài từ ngôn ngữ khác mà chúng ta muốn gọi. Phần `"C"` định nghĩa *application binary interface (ABI)* mà hàm bên ngoài sử dụng: ABI xác định cách gọi hàm ở cấp độ assembly. ABI `"C"` là phổ biến nhất và theo chuẩn ABI của ngôn ngữ lập trình C.

> #### Gọi hàm Rust từ các ngôn ngữ khác
>
> Chúng ta cũng có thể sử dụng `extern` để tạo một giao diện cho phép các ngôn ngữ khác gọi các hàm Rust. Thay vì tạo toàn bộ khối `extern`, chúng ta thêm từ khóa `extern` và chỉ định ABI sẽ sử dụng ngay trước từ khóa `fn` cho hàm tương ứng. Chúng ta cũng cần thêm chú thích `#[no_mangle]` để thông báo cho trình biên dịch Rust không thay đổi tên hàm này. *Mangling* là khi một trình biên dịch thay đổi tên mà chúng ta đã đặt cho hàm thành một tên khác chứa thêm thông tin cho các phần khác của quá trình biên dịch sử dụng, nhưng ít dễ đọc hơn với con người. Mỗi trình biên dịch ngôn ngữ lập trình thay đổi tên theo cách hơi khác nhau, vì vậy để một hàm Rust có thể được gọi từ các ngôn ngữ khác, chúng ta phải tắt việc thay đổi tên của trình biên dịch Rust.
>
> Trong ví dụ sau, chúng ta làm cho hàm `call_from_c` có thể truy cập từ mã C, sau khi nó được biên dịch thành thư viện chia sẻ và liên kết từ C:
>
> ```rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> This usage of `extern` does not require `unsafe`.

### Truy cập hoặc sửa đổi một biến static có thể thay đổi

Trong cuốn sách này, chúng ta chưa nói về *biến toàn cục* (global variables), mà Rust hỗ trợ nhưng có thể gây ra vấn đề với các quy tắc ownership của Rust. Nếu hai luồng truy cập cùng một biến toàn cục có thể thay đổi, điều này có thể gây ra *data race*.

Trong Rust, các biến toàn cục được gọi là biến *static*. Listing 19-9 cho thấy một ví dụ về khai báo và sử dụng biến static với một slice chuỗi làm giá trị.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-09/src/main.rs}}
```

<span class="caption">Listing 19-9: Khai báo và sử dụng một biến static không thể thay đổi</span>

Các biến static tương tự như các hằng số (constants), mà chúng ta đã thảo luận trong phần [“Differences Between Variables and Constants”][differences-between-variables-and-constants]<!-- ignore --> ở Chương 3. Tên của các biến static theo quy ước viết `SCREAMING_SNAKE_CASE`. Các biến static chỉ có thể lưu trữ các tham chiếu với lifetime `'static`, nghĩa là trình biên dịch Rust có thể xác định được lifetime và chúng ta không cần phải chú thích rõ ràng. Truy cập một biến static không thể thay đổi là an toàn.

Một khác biệt tinh tế giữa constants và biến static không thể thay đổi là các giá trị trong một biến static có một địa chỉ cố định trong bộ nhớ. Việc sử dụng giá trị này sẽ luôn truy cập cùng một dữ liệu. Ngược lại, constants được phép sao chép dữ liệu mỗi khi chúng được sử dụng. Một khác biệt nữa là các biến static có thể thay đổi được (mutable). Truy cập và sửa đổi các biến static mutable là *unsafe*. Listing 19-10 cho thấy cách khai báo, truy cập và sửa đổi một biến static mutable có tên là `COUNTER`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-10/src/main.rs}}
```

<span class="caption">Listing 19-10: Đọc hoặc ghi vào một biến static mutable là unsafe</span>

Như với các biến thông thường, chúng ta xác định khả năng thay đổi bằng từ khóa `mut`. Bất kỳ mã nào đọc hoặc ghi từ `COUNTER` phải nằm trong một khối `unsafe`. Mã này biên dịch và in ra `COUNTER: 3` như chúng ta mong đợi vì nó chạy đơn luồng. Nếu có nhiều luồng truy cập `COUNTER`, rất có khả năng xảy ra *data races*.

Với dữ liệu mutable có thể truy cập toàn cục, rất khó đảm bảo không có *data race*, đó là lý do Rust coi các biến static mutable là unsafe. Khi có thể, nên sử dụng các kỹ thuật đồng thời (concurrency) và các con trỏ thông minh an toàn với luồng (thread-safe smart pointers) mà chúng ta đã thảo luận ở Chương 16 để trình biên dịch kiểm tra rằng dữ liệu được truy cập từ các luồng khác nhau một cách an toàn.

### Triển khai một Unsafe Trait

Chúng ta có thể sử dụng `unsafe` để triển khai một unsafe trait. Một trait là unsafe khi ít nhất một trong các phương thức của nó có một bất biến (invariant) mà trình biên dịch không thể xác minh. Chúng ta khai báo một trait là `unsafe` bằng cách thêm từ khóa `unsafe` trước `trait` và đánh dấu việc triển khai trait đó cũng là `unsafe`, như được minh họa trong Listing 19-11.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-11/src/main.rs}}
```

<span class="caption">Listing 19-11: Khai báo và triển khai một unsafe trait</span>

Bằng cách sử dụng `unsafe impl`, chúng ta hứa rằng sẽ tuân thủ các bất biến mà trình biên dịch không thể xác minh.

Ví dụ, hãy nhớ lại các marker trait `Sync` và `Send` mà chúng ta đã thảo luận trong phần [“Extensible Concurrency with the `Sync` and `Send` Traits”][extensible-concurrency-with-the-sync-and-send-traits]<!-- ignore --> ở Chương 16: trình biên dịch sẽ triển khai các trait này tự động nếu các kiểu của chúng ta được cấu thành hoàn toàn từ các kiểu `Send` và `Sync`. Nếu chúng ta triển khai một kiểu chứa một kiểu không phải `Send` hoặc `Sync`, chẳng hạn như raw pointer, và muốn đánh dấu kiểu đó là `Send` hoặc `Sync`, chúng ta phải sử dụng `unsafe`. Rust không thể xác minh rằng kiểu của chúng ta tuân thủ các đảm bảo có thể gửi an toàn qua các luồng hoặc truy cập từ nhiều luồng; do đó, chúng ta cần thực hiện các kiểm tra này thủ công và chỉ ra điều đó bằng `unsafe`.

### Truy cập các trường của một Union

Hành động cuối cùng chỉ hoạt động với `unsafe` là truy cập các trường của một *union*. Một `union` tương tự như một `struct`, nhưng tại một thời điểm chỉ một trường được khai báo được sử dụng trong một thể hiện cụ thể. Union chủ yếu được dùng để tương tác với union trong mã C. Việc truy cập các trường của union là unsafe vì Rust không thể đảm bảo kiểu dữ liệu đang được lưu trữ trong thể hiện union là gì. Bạn có thể tìm hiểu thêm về union trong [the Rust Reference][reference].

### Khi nào nên sử dụng mã Unsafe

Việc sử dụng `unsafe` để thực hiện một trong năm hành động (siêu năng lực) vừa thảo luận không phải là sai hoặc bị coi thường. Nhưng việc viết mã `unsafe` đúng là khó hơn vì trình biên dịch không thể đảm bảo an toàn bộ nhớ. Khi bạn có lý do để sử dụng mã `unsafe`, bạn có thể làm như vậy, và việc có chú thích `unsafe` rõ ràng giúp dễ dàng theo dõi nguồn gốc của vấn đề khi chúng xảy ra.

[dangling-references]:
ch04-02-references-and-borrowing.html#dangling-references
[differences-between-variables-and-constants]:
ch03-01-variables-and-mutability.html#constants
[extensible-concurrency-with-the-sync-and-send-traits]:
ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits
[the-slice-type]: ch04-03-slices.html#the-slice-type
[reference]: ../reference/items/unions.html
