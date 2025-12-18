## Hàm và Closures Nâng Cao

Phần này khám phá một số tính năng nâng cao liên quan đến hàm và closures, bao gồm con trỏ hàm và việc trả về closures.

### Con Trỏ Hàm

Chúng ta đã nói về cách truyền closures vào hàm; bạn cũng có thể truyền các hàm thông thường vào hàm! Kỹ thuật này hữu ích khi bạn muốn truyền một hàm đã định nghĩa thay vì định nghĩa một closure mới. Các hàm tự động chuyển sang kiểu `fn` (với chữ f thường), không nhầm lẫn với trait closure `Fn`. Kiểu `fn` được gọi là *con trỏ hàm*. Việc truyền các hàm bằng con trỏ hàm cho phép bạn sử dụng các hàm như các đối số cho các hàm khác.

Cú pháp để chỉ định rằng một tham số là con trỏ hàm tương tự như cú pháp của closures, như được minh họa trong Listing 19-27, nơi chúng ta đã định nghĩa một hàm `add_one` cộng một vào tham số của nó. Hàm `do_twice` nhận hai tham số: một con trỏ hàm tới bất kỳ hàm nào nhận một tham số `i32` và trả về `i32`, và một giá trị `i32`. Hàm `do_twice` gọi hàm `f` hai lần, truyền cho nó giá trị `arg`, sau đó cộng kết quả của hai lần gọi hàm lại với nhau. Hàm `main` gọi `do_twice` với các đối số `add_one` và `5`.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-27/src/main.rs}}
```

<span class="caption">Listing 19-27: Sử dụng kiểu `fn` để nhận con trỏ hàm làm đối số</span>

Đoạn mã này in ra `The answer is: 12`. Chúng ta chỉ định rằng tham số `f` trong `do_twice` là một `fn` nhận một tham số kiểu `i32` và trả về `i32`. Sau đó, chúng ta có thể gọi `f` trong thân hàm `do_twice`. Trong `main`, chúng ta có thể truyền tên hàm `add_one` làm đối số đầu tiên cho `do_twice`.

Khác với closures, `fn` là một kiểu chứ không phải trait, vì vậy chúng ta chỉ định `fn` làm kiểu tham số trực tiếp thay vì khai báo một tham số generic với một trong các trait `Fn` làm ràng buộc trait.

Con trỏ hàm triển khai cả ba trait của closure (`Fn`, `FnMut` và `FnOnce`), nghĩa là bạn luôn có thể truyền một con trỏ hàm làm đối số cho một hàm yêu cầu một closure. Tốt nhất là viết các hàm sử dụng kiểu generic và một trong các trait của closure để hàm của bạn có thể chấp nhận cả hàm và closures.

Tuy nhiên, một ví dụ về trường hợp bạn chỉ muốn nhận `fn` mà không phải closures là khi tương tác với mã bên ngoài không có closures: các hàm C có thể nhận các hàm làm đối số, nhưng C không có closures.

Lấy ví dụ về nơi bạn có thể sử dụng một closure được định nghĩa inline hoặc một hàm có tên, hãy xem cách sử dụng phương thức `map` do trait `Iterator` cung cấp trong thư viện chuẩn. Để sử dụng hàm `map` để biến một vector các số thành một vector các chuỗi, chúng ta có thể sử dụng một closure, như sau:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-15-map-closure/src/main.rs:here}}
```

Hoặc chúng ta có thể đặt tên một hàm làm đối số cho `map` thay vì sử dụng closure, như sau:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-16-map-function/src/main.rs:here}}
```

Lưu ý rằng chúng ta phải sử dụng cú pháp đầy đủ mà chúng ta đã nói trước đó trong phần [“Advanced Traits”][advanced-traits]<!-- ignore --> vì có nhiều hàm cùng tên `to_string` có sẵn. Ở đây, chúng ta đang sử dụng hàm `to_string` được định nghĩa trong trait `ToString`, mà thư viện chuẩn đã triển khai cho bất kỳ kiểu nào thực thi trait `Display`.

Nhớ lại từ phần [“Enum values”][enum-values]<!-- ignore --> trong Chương 6 rằng tên của mỗi biến thể enum mà chúng ta định nghĩa cũng trở thành một hàm khởi tạo. Chúng ta có thể sử dụng các hàm khởi tạo này như các con trỏ hàm triển khai các trait của closure, nghĩa là chúng ta có thể chỉ định các hàm khởi tạo làm đối số cho các phương thức nhận closure, như sau:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-17-map-initializer/src/main.rs:here}}
```

Ở đây chúng ta tạo các instance `Status::Value` sử dụng từng giá trị `u32` trong phạm vi mà `map` được gọi bằng cách sử dụng hàm khởi tạo của `Status::Value`. Một số người thích phong cách này, trong khi một số người thích dùng closures. Chúng biên dịch ra cùng một mã, vì vậy hãy sử dụng phong cách nào rõ ràng hơn với bạn.

### Trả về Closures

Closures được biểu diễn bằng các trait, nghĩa là bạn không thể trả về closures trực tiếp. Trong hầu hết các trường hợp mà bạn muốn trả về một trait, bạn có thể thay vào đó sử dụng kiểu cụ thể triển khai trait làm giá trị trả về của hàm. Tuy nhiên, bạn không thể làm điều đó với closures vì chúng không có kiểu cụ thể có thể trả về; ví dụ, bạn không được phép sử dụng con trỏ hàm `fn` làm kiểu trả về.

Đoạn mã sau cố gắng trả về một closure trực tiếp, nhưng sẽ không biên dịch:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-18-returns-closure/src/lib.rs}}
```

Lỗi biên dịch sẽ như sau:

```console
{{#include ../listings/ch19-advanced-features/no-listing-18-returns-closure/output.txt}}
```

Lỗi này lại liên quan đến trait `Sized`! Rust không biết cần bao nhiêu bộ nhớ để lưu closure. Chúng ta đã thấy một giải pháp cho vấn đề này trước đó. Chúng ta có thể sử dụng một trait object:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-19-returns-closure-trait-object/src/lib.rs}}
```

Đoạn mã này sẽ biên dịch bình thường. Để tìm hiểu thêm về trait objects, tham khảo phần [“Using Trait Objects That Allow for Values of Different Types”][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore --> trong Chương 17.

Tiếp theo, hãy cùng xem về macros!

[advanced-traits]:
ch19-03-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[using-trait-objects-that-allow-for-values-of-different-types]:
ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
