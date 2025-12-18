## Các Đặc Điểm của Ngôn Ngữ Lập Trình Hướng Đối Tượng

Trong cộng đồng lập trình, không có sự đồng thuận tuyệt đối về việc một ngôn ngữ
cần phải có những đặc điểm nào thì mới được xem là hướng đối tượng. Rust chịu
ảnh hưởng từ nhiều mô hình lập trình khác nhau, bao gồm cả OOP; ví dụ, chúng ta
đã khám phá những đặc điểm xuất phát từ lập trình hàm trong Chương 13. Có thể
lập luận rằng các ngôn ngữ OOP chia sẻ một số đặc điểm chung, cụ thể là object,
encapsulation và inheritance. Hãy cùng xem từng đặc điểm này có ý nghĩa gì và
Rust có hỗ trợ chúng hay không.

### Object Chứa Dữ Liệu và Hành Vi

Cuốn sách *Design Patterns: Elements of Reusable Object-Oriented Software* của
Erich Gamma, Richard Helm, Ralph Johnson và John Vlissides (Addison-Wesley
Professional, 1994), thường được gọi một cách thân mật là cuốn sách của *Gang of
Four*, là một tuyển tập các design pattern hướng đối tượng. Cuốn sách này định
nghĩa OOP như sau:

> Các chương trình hướng đối tượng được tạo thành từ các object. Một *object*
> đóng gói cả dữ liệu lẫn các thủ tục thao tác trên dữ liệu đó. Các thủ tục này
> thường được gọi là *method* hoặc *operation*.

Theo định nghĩa này, Rust là một ngôn ngữ hướng đối tượng: struct và enum có dữ
liệu, và các khối `impl` cung cấp method cho struct và enum. Mặc dù struct và
enum có method không được *gọi* là object, nhưng theo định nghĩa về object của
Gang of Four, chúng cung cấp cùng một chức năng.

### Encapsulation Che Giấu Chi Tiết Triển Khai

Một khía cạnh khác thường gắn liền với OOP là khái niệm *encapsulation*, nghĩa
là các chi tiết triển khai của một object không thể bị truy cập trực tiếp bởi
đoạn mã sử dụng object đó. Do đó, cách duy nhất để tương tác với một object là
thông qua public API của nó; mã sử dụng object không nên có khả năng đi sâu vào
bên trong object và thay đổi trực tiếp dữ liệu hoặc hành vi. Điều này cho phép
lập trình viên thay đổi và refactor phần nội bộ của object mà không cần phải
thay đổi mã đang sử dụng object đó.

Chúng ta đã thảo luận về cách kiểm soát encapsulation trong Chương 7: chúng ta
có thể dùng từ khóa `pub` để quyết định module, type, function và method nào
trong mã của mình là public, và theo mặc định thì mọi thứ còn lại đều là
private. Ví dụ, chúng ta có thể định nghĩa một struct `AveragedCollection` có
một field chứa một vector các giá trị `i32`. Struct này cũng có thể có một field
khác chứa giá trị trung bình của các phần tử trong vector, nghĩa là giá trị
trung bình không cần phải được tính toán mỗi khi ai đó cần tới. Nói cách khác,
`AveragedCollection` sẽ cache giá trị trung bình đã được tính sẵn cho chúng ta.
Listing 17-1 trình bày định nghĩa của struct `AveragedCollection`:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-01/src/lib.rs}}
```

<span class="caption">Listing 17-1: Một struct `AveragedCollection` duy trì
một danh sách các số nguyên và giá trị trung bình của các phần tử trong
collection</span>

Struct này được đánh dấu là `pub` để các đoạn mã khác có thể sử dụng nó, nhưng
các field bên trong struct vẫn được giữ ở mức private. Điều này rất quan trọng
trong trường hợp này, bởi vì chúng ta muốn đảm bảo rằng mỗi khi có một giá trị
được thêm vào hoặc bị loại bỏ khỏi danh sách, thì giá trị trung bình cũng sẽ
được cập nhật tương ứng.

Chúng ta thực hiện điều đó bằng cách triển khai các method `add`, `remove` và
`average` cho struct này, như được minh họa trong Listing 17-2:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-02/src/lib.rs:here}}
```

<span class="caption">Listing 17-2: Triển khai các phương thức public
`add`, `remove` và `average` cho `AveragedCollection`</span>

Các phương thức public `add`, `remove` và `average` là những cách duy nhất
để truy cập hoặc thay đổi dữ liệu trong một instance của
`AveragedCollection`. Khi một phần tử được thêm vào `list` thông qua phương
thức `add`, hoặc bị loại bỏ thông qua phương thức `remove`, thì phần triển khai
của mỗi phương thức này sẽ gọi phương thức private `update_average` để xử lý
việc cập nhật field `average`.

Chúng ta giữ các field `list` và `average` ở trạng thái private để đảm bảo rằng
không có cách nào để code bên ngoài có thể trực tiếp thêm hoặc xóa phần tử khỏi
field `list`. Nếu cho phép làm như vậy, field `average` có thể bị lệch và không
còn đồng bộ với `list` khi dữ liệu thay đổi. Phương thức `average` chỉ trả về
giá trị của field `average`, cho phép code bên ngoài đọc giá trị trung bình
nhưng không thể chỉnh sửa nó.

Bởi vì chúng ta đã đóng gói (encapsulate) các chi tiết cài đặt của struct
`AveragedCollection`, nên trong tương lai chúng ta có thể dễ dàng thay đổi một
số khía cạnh, chẳng hạn như cấu trúc dữ liệu được sử dụng. Ví dụ, ta có thể
dùng `HashSet<i32>` thay cho `Vec<i32>` cho field `list`. Miễn là chữ ký
(signatures) của các phương thức public `add`, `remove` và `average` vẫn giữ
nguyên, thì code sử dụng `AveragedCollection` sẽ không cần phải thay đổi.

Nếu chúng ta để `list` là public, thì điều này sẽ không còn đúng nữa:
`HashSet<i32>` và `Vec<i32>` có các phương thức khác nhau để thêm và xóa phần
tử, nên code bên ngoài rất có thể sẽ phải thay đổi nếu nó thao tác trực tiếp
lên `list`.

Nếu đóng gói (encapsulation) là một yêu cầu bắt buộc để một ngôn ngữ được xem là
hướng đối tượng, thì Rust hoàn toàn đáp ứng được yêu cầu đó. Việc có thể lựa
chọn dùng hay không dùng từ khóa `pub` cho từng phần của code cho phép chúng ta
che giấu chi tiết cài đặt một cách hiệu quả.


### Inheritance as a Type System and as Code Sharing

*Inheritance* là một cơ chế cho phép một đối tượng kế thừa các thành phần từ
định nghĩa của một đối tượng khác, nhờ đó nó có được dữ liệu và hành vi của đối tượng cha
mà không cần bạn phải định nghĩa lại chúng.

Nếu một ngôn ngữ bắt buộc phải có inheritance thì mới được coi là một ngôn ngữ hướng đối tượng,
thì Rust không phải là một ngôn ngữ như vậy. Không có cách nào để định nghĩa một struct
kế thừa các field và phần cài đặt method của struct cha mà không sử dụng macro.

Tuy nhiên, nếu bạn đã quen với việc có inheritance trong “hộp công cụ” lập trình của mình,
bạn có thể sử dụng các giải pháp khác trong Rust, tùy thuộc vào lý do ban đầu khiến bạn
muốn dùng inheritance.

Bạn thường chọn inheritance vì hai lý do chính. Lý do thứ nhất là để tái sử dụng mã nguồn:
bạn có thể cài đặt một hành vi cụ thể cho một kiểu, và inheritance cho phép bạn
tái sử dụng phần cài đặt đó cho một kiểu khác. Trong Rust, bạn có thể làm điều này
theo cách hạn chế bằng cách sử dụng các phương thức mặc định (default method implementations)
trong trait, như bạn đã thấy ở Listing 10-14 khi chúng ta thêm một phần cài đặt mặc định
cho phương thức `summarize` trong trait `Summary`. Bất kỳ kiểu nào triển khai trait
`Summary` đều sẽ có sẵn phương thức `summarize` mà không cần thêm bất kỳ dòng mã nào.
Điều này tương tự như việc một lớp cha có sẵn phần cài đặt của một method và một lớp con
kế thừa cũng có phần cài đặt method đó. Chúng ta cũng có thể ghi đè phần cài đặt mặc định
của phương thức `summarize` khi triển khai trait `Summary`, tương tự như cách một lớp con
ghi đè method được kế thừa từ lớp cha.

Lý do thứ hai để sử dụng inheritance liên quan đến hệ thống kiểu (type system):
cho phép một kiểu con có thể được sử dụng ở những nơi mà kiểu cha được sử dụng.
Điều này còn được gọi là *polymorphism*, nghĩa là bạn có thể thay thế nhiều đối tượng
cho nhau tại thời điểm chạy nếu chúng chia sẻ một số đặc điểm nhất định.

> ### Polymorphism
>
> Với nhiều người, polymorphism đồng nghĩa với inheritance. Nhưng thực tế đây là
> một khái niệm tổng quát hơn, dùng để chỉ những đoạn mã có thể làm việc với dữ liệu
> của nhiều kiểu khác nhau. Trong trường hợp inheritance, các kiểu đó thường là
> các lớp con.
>
> Thay vào đó, Rust sử dụng generics để trừu tượng hóa trên nhiều kiểu khả dĩ khác nhau
> và trait bounds để áp đặt các ràng buộc về những gì các kiểu đó phải cung cấp.
> Cách tiếp cận này đôi khi được gọi là *bounded parametric polymorphism*.

Inheritance gần đây đã dần mất đi sự ưa chuộng như một giải pháp thiết kế trong nhiều
ngôn ngữ lập trình, bởi vì nó thường có nguy cơ chia sẻ nhiều mã hơn mức cần thiết.
Các lớp con không phải lúc nào cũng nên chia sẻ toàn bộ đặc điểm của lớp cha,
nhưng inheritance lại buộc chúng phải làm vậy. Điều này có thể khiến thiết kế của
chương trình kém linh hoạt hơn. Nó cũng tạo ra khả năng gọi các method trên lớp con
mà không hợp lý, hoặc gây lỗi vì các method đó không áp dụng cho lớp con.
Ngoài ra, một số ngôn ngữ chỉ cho phép đơn kế thừa (single inheritance),
nghĩa là một lớp con chỉ có thể kế thừa từ một lớp duy nhất, điều này càng
hạn chế thêm tính linh hoạt của thiết kế chương trình.

Vì những lý do đó, Rust chọn một hướng tiếp cận khác là sử dụng *trait objects*
thay vì inheritance. Hãy cùng xem trait objects cho phép polymorphism trong Rust như thế nào.
