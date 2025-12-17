## Xử Lý Một Chuỗi Phần Tử với Iterators

Mẫu iterator cho phép bạn thực hiện một tác vụ nào đó trên một chuỗi
các phần tử lần lượt. Một iterator chịu trách nhiệm về logic lặp qua
từng phần tử và xác định khi nào chuỗi kết thúc. Khi sử dụng iterators,
bạn không cần tự triển khai lại logic đó.

Trong Rust, iterators là *lazy*, nghĩa là chúng không có tác dụng gì
cho đến khi bạn gọi các phương thức tiêu thụ iterator để dùng hết nó.
Ví dụ, đoạn code trong Listing 13-10 tạo một iterator trên các phần
tử trong vector `v1` bằng cách gọi phương thức `iter` định nghĩa
trên `Vec<T>`. Đoạn code này tự nó không làm gì hữu ích.

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

<span class="caption">Listing 13-10: Tạo một iterator</span>

Iterator được lưu trong biến `v1_iter`. Khi đã tạo một iterator, chúng
ta có thể sử dụng nó theo nhiều cách khác nhau. Trong Listing 3-5 ở
Chương 3, chúng ta đã lặp qua một mảng bằng vòng lặp `for` để thực
hiện một số code trên từng phần tử. Ngầm định, điều này đã tạo và
sau đó tiêu thụ một iterator, nhưng chúng ta đã bỏ qua cách thức
hoạt động chính xác của nó cho đến bây giờ.

Trong ví dụ ở Listing 13-11, chúng ta tách việc tạo iterator ra khỏi
việc sử dụng iterator trong vòng lặp `for`. Khi vòng lặp `for` được
gọi với iterator trong `v1_iter`, mỗi phần tử trong iterator được
sử dụng trong một lần lặp của vòng lặp, in ra từng giá trị.

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

<span class="caption">Listing 13-11: Sử dụng iterator trong vòng lặp `for`</span>

Trong các ngôn ngữ không có iterator được cung cấp bởi thư viện chuẩn,
bạn có thể viết chức năng tương tự bằng cách khởi tạo một biến ở
chỉ số 0, dùng biến đó để truy cập vào vector lấy giá trị, và tăng
giá trị biến trong một vòng lặp cho đến khi đạt tổng số phần tử
trong vector.

Iterators xử lý tất cả logic đó cho bạn, giảm bớt code lặp đi lặp
lại mà bạn có thể vô tình làm sai. Iterators cung cấp nhiều sự linh
hoạt hơn để sử dụng cùng một logic với nhiều loại chuỗi khác nhau,
không chỉ những cấu trúc dữ liệu có thể truy cập theo chỉ số như
vector. Hãy xem cách iterators thực hiện điều đó.

### Trait `Iterator` và Phương Thức `next`

Tất cả iterators triển khai một trait có tên `Iterator` được định
nghĩa trong thư viện chuẩn. Định nghĩa của trait trông như sau:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

Lưu ý rằng định nghĩa này sử dụng một số cú pháp mới: `type Item` và
`Self::Item`, định nghĩa một *associated type* với trait này. Chúng ta
sẽ bàn kỹ về associated types trong Chương 19. Hiện tại, bạn chỉ cần
biết rằng đoạn code này nói rằng việc triển khai trait `Iterator` yêu
cầu bạn cũng phải định nghĩa một kiểu `Item`, và kiểu `Item` này được
sử dụng trong kiểu trả về của phương thức `next`. Nói cách khác, kiểu
`Item` sẽ là kiểu được trả về từ iterator.

Trait `Iterator` chỉ yêu cầu người triển khai định nghĩa một phương
thức: phương thức `next`, trả về từng phần tử của iterator một lần,
bao bọc trong `Some` và, khi kết thúc iteration, trả về `None`.

Chúng ta có thể gọi trực tiếp phương thức `next` trên iterators; Listing
13-12 minh họa các giá trị được trả về từ các lần gọi lặp đi lặp lại
`next` trên iterator được tạo từ vector.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

<span class="caption">Listing 13-12: Gọi phương thức `next` trên một iterator</span>

Lưu ý rằng chúng ta cần làm cho `v1_iter` có thể thay đổi (mutable):
gọi phương thức `next` trên một iterator thay đổi trạng thái nội bộ mà
iterator sử dụng để theo dõi vị trí hiện tại trong chuỗi. Nói cách
khác, đoạn code này *tiêu thụ*, hay dùng hết, iterator. Mỗi lần gọi
`next` lấy đi một phần tử từ iterator. Chúng ta không cần làm cho
`v1_iter` mutable khi sử dụng vòng lặp `for` vì vòng lặp lấy quyền
sở hữu của `v1_iter` và làm cho nó mutable ngầm.

Cũng lưu ý rằng các giá trị nhận được từ các lần gọi `next` là các
tham chiếu bất biến (immutable references) đến các giá trị trong
vector. Phương thức `iter` tạo ra một iterator trên các tham chiếu
bất biến. Nếu muốn tạo một iterator lấy quyền sở hữu của `v1` và
trả về các giá trị sở hữu, ta có thể gọi `into_iter` thay vì `iter`.
Tương tự, nếu muốn lặp qua các tham chiếu có thể thay đổi, ta có thể
gọi `iter_mut` thay vì `iter`.

### Các phương thức tiêu thụ iterator

Trait `Iterator` có một số phương thức khác nhau với triển khai mặc
định do thư viện chuẩn cung cấp; bạn có thể tìm hiểu về các phương
thức này bằng cách xem tài liệu API của trait `Iterator`. Một số phương
thức này gọi phương thức `next` trong định nghĩa của chúng, đó là lý
do bạn phải triển khai `next` khi implement trait `Iterator`.

Các phương thức gọi `next` được gọi là *consuming adaptors*, vì gọi
chúng sẽ tiêu thụ iterator. Một ví dụ là phương thức `sum`, nó
lấy quyền sở hữu iterator và lặp qua các phần tử bằng cách gọi
`next` liên tục, do đó tiêu thụ iterator. Khi lặp qua, nó cộng
mỗi phần tử vào tổng dồn và trả về tổng khi kết thúc iteration.
Listing 13-13 có một ví dụ minh họa việc sử dụng phương thức `sum`:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

<span class="caption">Listing 13-13: Gọi phương thức `sum` để lấy tổng
tất cả các phần tử trong iterator</span>

Chúng ta không được phép sử dụng `v1_iter` sau khi gọi `sum` vì `sum`
lấy quyền sở hữu của iterator mà chúng ta gọi nó trên đó.

### Các phương thức tạo ra iterator khác

*Iterator adaptors* là các phương thức định nghĩa trên trait `Iterator`
không tiêu thụ iterator. Thay vào đó, chúng tạo ra các iterator khác
bằng cách thay đổi một số khía cạnh của iterator gốc.

Listing 13-14 minh họa việc gọi phương thức iterator adaptor `map`,
nó nhận một closure để gọi trên từng phần tử khi các phần tử được
lặp qua. Phương thức `map` trả về một iterator mới tạo ra các phần
tử đã được sửa đổi. Closure ở đây tạo ra một iterator mới mà mỗi
phần tử từ vector sẽ được tăng thêm 1:

<span class="filename">Filename: src/main.rs</span>

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

<span class="caption">Listing 13-14: Gọi iterator adaptor `map` để
tạo một iterator mới</span>

Tuy nhiên, đoạn code này sinh ra một cảnh báo:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Đoạn code trong Listing 13-14 không làm gì cả; closure mà chúng
ta đã chỉ định chưa bao giờ được gọi. Cảnh báo nhắc nhở chúng
ta lý do: các iterator adaptors là lazy, và chúng ta cần tiêu
thụ iterator ở đây.

Để sửa cảnh báo này và tiêu thụ iterator, chúng ta sẽ sử dụng
phương thức `collect`, mà chúng ta đã dùng trong Chương 12 với
`env::args` trong Listing 12-1. Phương thức này tiêu thụ iterator
và thu thập các giá trị kết quả vào một kiểu dữ liệu collection.

Trong Listing 13-15, chúng ta thu thập kết quả của việc lặp qua
iterator được trả về từ cuộc gọi `map` vào một vector. Vector
này sẽ chứa từng phần tử từ vector gốc tăng thêm 1.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

<span class="caption">Listing 13-15: Gọi phương thức `map` để tạo một
iterator mới và sau đó gọi phương thức `collect` để tiêu thụ iterator
mới và tạo ra một vector</span>

Vì `map` nhận một closure, chúng ta có thể chỉ định bất kỳ thao tác
nào muốn thực hiện trên từng phần tử. Đây là một ví dụ tuyệt vời
về cách closures cho phép bạn tùy chỉnh hành vi trong khi tái sử dụng
hành vi lặp mà trait `Iterator` cung cấp.

Bạn có thể xâu chuỗi nhiều lần gọi các iterator adaptor để thực
hiện các hành động phức tạp một cách dễ đọc. Nhưng vì tất cả các
iterator là lazy, bạn phải gọi một trong các phương thức consuming
adaptor để nhận kết quả từ các cuộc gọi đến iterator adaptor.

### Sử dụng Closures capture environment

Nhiều iterator adaptor nhận closures làm tham số, và thường
các closures chúng ta chỉ định làm tham số cho iterator adaptor
sẽ là các closures capture environment của chúng.

Trong ví dụ này, chúng ta sẽ sử dụng phương thức `filter` nhận
một closure. Closure nhận một phần tử từ iterator và trả về
`bool`. Nếu closure trả về `true`, giá trị sẽ được bao gồm
trong iteration do `filter` tạo ra. Nếu closure trả về `false`,
giá trị sẽ không được bao gồm.

Trong Listing 13-16, chúng ta sử dụng `filter` với một closure
capture biến `shoe_size` từ môi trường của nó để lặp qua
một tập hợp các instance của struct `Shoe`. Nó chỉ trả về
những đôi giày có kích thước đã chỉ định.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

<span class="caption">Listing 13-16: Sử dụng phương thức `filter` với một
closure capture `shoe_size`</span>

Hàm `shoes_in_size` nhận quyền sở hữu một vector các đôi giày và một
kích thước giày làm tham số. Nó trả về một vector chỉ chứa các đôi
giày có kích thước đã chỉ định.

Trong thân hàm `shoes_in_size`, chúng ta gọi `into_iter` để tạo một
iterator lấy quyền sở hữu của vector. Sau đó, chúng ta gọi `filter`
để biến đổi iterator đó thành một iterator mới chỉ chứa các phần tử
mà closure trả về `true`.

Closure capture tham số `shoe_size` từ môi trường và so sánh giá trị
với kích thước của từng đôi giày, chỉ giữ lại những đôi có kích thước
được chỉ định. Cuối cùng, gọi `collect` thu thập các giá trị trả về
từ iterator đã được biến đổi vào một vector được hàm trả về.

Bài test cho thấy khi gọi `shoes_in_size`, chúng ta chỉ nhận lại
những đôi giày có cùng kích thước với giá trị đã chỉ định.
