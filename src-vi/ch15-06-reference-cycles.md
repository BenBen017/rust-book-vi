## Chu kỳ tham chiếu có thể gây rò rỉ bộ nhớ

Các đảm bảo về an toàn bộ nhớ của Rust khiến việc vô tình tạo ra bộ nhớ
không bao giờ được giải phóng (gọi là *rò rỉ bộ nhớ*) trở nên khó khăn,
nhưng không phải là không thể. Ngăn chặn hoàn toàn rò rỉ bộ nhớ không phải
là một trong những đảm bảo của Rust, có nghĩa là rò rỉ bộ nhớ vẫn an toàn
về mặt bộ nhớ trong Rust. Chúng ta có thể thấy Rust cho phép rò rỉ bộ nhớ
khi sử dụng `Rc<T>` và `RefCell<T>`: có thể tạo ra các tham chiếu mà các
phần tử tham chiếu lẫn nhau theo một chu kỳ. Điều này gây rò rỉ bộ nhớ
bởi vì số tham chiếu của mỗi phần tử trong chu kỳ sẽ không bao giờ
đạt 0, và các giá trị sẽ không bao giờ được drop.

### Tạo một chu kỳ tham chiếu

Hãy xem cách một chu kỳ tham chiếu có thể xảy ra và cách ngăn chặn nó,
bắt đầu với định nghĩa của enum `List` và phương thức `tail` trong Listing
15-25:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs}}
```

<span class="caption">Listing 15-25: Định nghĩa cons list giữ một
`RefCell<T>` để chúng ta có thể sửa đổi phần mà một biến thể `Cons` đang tham chiếu</span>

Chúng ta đang sử dụng một biến thể khác của định nghĩa `List` từ Listing 15-5. 
Phần tử thứ hai trong biến thể `Cons` giờ là `RefCell<Rc<List>>`, có nghĩa là 
thay vì chỉ sửa đổi giá trị `i32` như chúng ta đã làm trong Listing 15-24, 
chúng ta muốn sửa đổi giá trị `List` mà một biến thể `Cons` đang trỏ tới. 
Chúng ta cũng thêm phương thức `tail` để thuận tiện khi truy cập phần tử 
thứ hai nếu chúng ta có một biến thể `Cons`.

Trong Listing 15-26, chúng ta thêm một hàm `main` sử dụng các định nghĩa trong 
Listing 15-25. Đoạn code này tạo một list trong `a` và một list trong `b` trỏ 
tới list trong `a`. Sau đó nó sửa đổi list trong `a` để trỏ tới `b`, tạo ra 
một chu kỳ tham chiếu. Có các câu lệnh `println!` dọc theo quá trình để hiển 
thị số tham chiếu tại các điểm khác nhau.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

<span class="caption">Listing 15-26: Tạo một chu kỳ tham chiếu của hai giá trị `List` trỏ lẫn nhau</span>

Chúng ta tạo một instance `Rc<List>` giữ một giá trị `List` trong biến `a` 
với danh sách ban đầu là `5, Nil`. Sau đó chúng ta tạo một instance `Rc<List>` 
giữ một giá trị `List` khác trong biến `b` chứa giá trị 10 và trỏ tới list trong `a`.

Chúng ta sửa đổi `a` để nó trỏ tới `b` thay vì `Nil`, tạo ra một chu kỳ. 
Chúng ta làm điều này bằng cách sử dụng phương thức `tail` để lấy tham chiếu tới 
`RefCell<Rc<List>>` trong `a`, và lưu vào biến `link`. Sau đó chúng ta dùng phương thức 
`borrow_mut` trên `RefCell<Rc<List>>` để thay đổi giá trị bên trong từ một `Rc<List>` 
giữ giá trị `Nil` thành `Rc<List>` trong `b`.

Khi chạy đoạn code này, giữ câu lệnh `println!` cuối cùng đang bị comment lại, 
chúng ta sẽ nhận được output sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

Số lượng tham chiếu của các instance `Rc<List>` trong cả `a` và `b` là 2 sau khi 
chúng ta thay đổi danh sách trong `a` để trỏ tới `b`. Ở cuối `main`, Rust sẽ 
drop biến `b`, điều này làm giảm số lượng tham chiếu của instance `Rc<List>` trong `b` 
từ 2 xuống còn 1. Bộ nhớ mà `Rc<List>` chiếm trên heap sẽ không bị giải phóng tại thời điểm này, 
bởi vì số lượng tham chiếu vẫn là 1, không phải 0. Sau đó, Rust drop `a`, làm giảm số lượng 
tham chiếu của instance `Rc<List>` trong `a` từ 2 xuống còn 1. Bộ nhớ của instance này cũng 
không thể bị giải phóng, vì instance `Rc<List>` khác vẫn trỏ tới nó. Bộ nhớ được cấp phát 
cho danh sách sẽ mãi không được thu hồi. Để hình dung chu kỳ tham chiếu này, chúng tôi tạo một sơ đồ trong Hình 15-4.

<img alt="Reference cycle of lists" src="img/trpl15-04.svg" class="center" />

<span class="caption">Hình 15-4: Một chu kỳ tham chiếu của các danh sách `a` và `b` trỏ lẫn nhau</span>

Nếu bạn bỏ comment câu lệnh `println!` cuối cùng và chạy chương trình, Rust sẽ cố gắng 
in chu kỳ này với `a` trỏ tới `b` trỏ tới `a` và cứ thế cho đến khi tràn stack.

So với một chương trình thực tế, hậu quả của việc tạo chu kỳ tham chiếu trong ví dụ này 
không quá nghiêm trọng: ngay sau khi chúng ta tạo chu kỳ tham chiếu, chương trình kết thúc. 
Tuy nhiên, nếu một chương trình phức tạp hơn cấp phát nhiều bộ nhớ trong một chu kỳ và giữ nó lâu, 
chương trình sẽ sử dụng nhiều bộ nhớ hơn cần thiết và có thể làm hệ thống bị quá tải, dẫn đến hết bộ nhớ khả dụng.

Việc tạo chu kỳ tham chiếu không dễ, nhưng cũng không phải là không thể. Nếu bạn có các giá trị 
`RefCell<T>` chứa các giá trị `Rc<T>` hoặc các kết hợp lồng nhau tương tự của các kiểu với khả năng 
biến đổi bên trong và đếm tham chiếu, bạn phải đảm bảo rằng mình không tạo ra các chu kỳ; 
bạn không thể trông chờ Rust bắt lỗi chúng. Tạo một chu kỳ tham chiếu sẽ là một lỗi logic trong chương trình của bạn, 
và bạn nên dùng kiểm thử tự động, review code và các phương pháp phát triển phần mềm khác để giảm thiểu.

Một giải pháp khác để tránh chu kỳ tham chiếu là tổ chức lại cấu trúc dữ liệu sao cho 
một số tham chiếu biểu thị quyền sở hữu và một số tham chiếu không biểu thị quyền sở hữu. 
Kết quả là, bạn có thể có các chu kỳ được tạo từ một số quan hệ sở hữu và một số quan hệ 
không sở hữu, và chỉ các quan hệ sở hữu mới ảnh hưởng tới việc một giá trị có thể bị drop hay không. 
Trong Listing 15-25, chúng ta luôn muốn các variant `Cons` sở hữu danh sách của chúng, 
vì vậy việc tổ chức lại cấu trúc dữ liệu là không khả thi. Hãy xem một ví dụ sử dụng các đồ thị 
gồm các nút cha và nút con để thấy khi nào các quan hệ không sở hữu là cách phù hợp để ngăn 
chu kỳ tham chiếu.

### Ngăn Ngừa Chu Kỳ Tham Chiếu: Chuyển `Rc<T>` thành `Weak<T>`

Cho đến nay, chúng ta đã minh họa rằng việc gọi `Rc::clone` sẽ tăng `strong_count` 
của một instance `Rc<T>`, và một instance `Rc<T>` chỉ được dọn dẹp nếu `strong_count` 
của nó là 0. Bạn cũng có thể tạo một *tham chiếu yếu* tới giá trị bên trong một 
instance `Rc<T>` bằng cách gọi `Rc::downgrade` và truyền một tham chiếu tới `Rc<T>`. 
Các tham chiếu mạnh (strong) là cách bạn chia sẻ quyền sở hữu của một instance `Rc<T>`. 
Các tham chiếu yếu (weak) không biểu thị mối quan hệ sở hữu, và số lượng của chúng 
không ảnh hưởng đến thời điểm instance `Rc<T>` được dọn dẹp. Chúng sẽ không gây ra chu kỳ 
tham chiếu vì bất kỳ chu kỳ nào có chứa tham chiếu yếu sẽ bị phá vỡ khi số lượng 
tham chiếu mạnh của các giá trị liên quan về 0.

Khi bạn gọi `Rc::downgrade`, bạn nhận được một smart pointer kiểu `Weak<T>`. 
Thay vì tăng `strong_count` trong instance `Rc<T>` lên 1, việc gọi `Rc::downgrade` 
sẽ tăng `weak_count` lên 1. Kiểu `Rc<T>` dùng `weak_count` để theo dõi có bao nhiêu 
tham chiếu `Weak<T>` tồn tại, tương tự như `strong_count`. Sự khác biệt là `weak_count` 
không cần phải về 0 để instance `Rc<T>` được dọn dẹp.

Vì giá trị mà `Weak<T>` tham chiếu có thể đã bị drop, để thao tác với giá trị mà một 
`Weak<T>` trỏ tới, bạn phải chắc chắn giá trị vẫn tồn tại. Làm điều này bằng cách 
gọi phương thức `upgrade` trên instance `Weak<T>`, phương thức này sẽ trả về 
`Option<Rc<T>>`. Bạn sẽ nhận được `Some` nếu giá trị `Rc<T>` chưa bị drop và `None` 
nếu giá trị `Rc<T>` đã bị drop. Vì `upgrade` trả về `Option<Rc<T>>`, Rust sẽ đảm bảo 
rằng cả trường hợp `Some` và `None` đều được xử lý, và sẽ không có con trỏ không hợp lệ.

Ví dụ, thay vì sử dụng một danh sách mà các phần tử chỉ biết về phần tử tiếp theo, 
chúng ta sẽ tạo một cây mà các phần tử biết về các phần tử con *và* các phần tử cha của chúng.

#### Tạo Cấu Trúc Dữ Liệu Cây: một `Node` với các Node Con

Để bắt đầu, chúng ta sẽ xây dựng một cây với các node biết về các node con của chúng. 
Chúng ta sẽ tạo một struct có tên `Node` giữ giá trị `i32` của chính nó cũng như các 
tham chiếu tới các giá trị `Node` con:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Chúng ta muốn một `Node` sở hữu các node con của nó, và chúng ta muốn chia sẻ quyền sở hữu đó với các biến 
để có thể truy cập trực tiếp từng `Node` trong cây. Để làm điều này, chúng ta định nghĩa các phần tử trong `Vec<T>` 
là các giá trị kiểu `Rc<Node>`. Chúng ta cũng muốn thay đổi node nào là con của node khác, 
vì vậy chúng ta đặt một `RefCell<T>` trong `children` bao quanh `Vec<Rc<Node>>`.

Tiếp theo, chúng ta sẽ sử dụng định nghĩa struct và tạo một instance `Node` có tên `leaf` với giá trị 3 và không có node con, 
và một instance khác có tên `branch` với giá trị 5 và `leaf` là một trong các node con của nó, 
như được minh họa trong Listing 15-27:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

Chúng ta clone `Rc<Node>` trong `leaf` và lưu vào `branch`, nghĩa là `Node` trong `leaf` bây giờ có hai chủ sở hữu: `leaf` và `branch`. 
Chúng ta có thể đi từ `branch` đến `leaf` thông qua `branch.children`, nhưng không có cách nào đi từ `leaf` đến `branch`. 
Lý do là `leaf` không có tham chiếu đến `branch` và không biết chúng có liên quan. Chúng ta muốn `leaf` biết rằng `branch` là cha của nó. 
Chúng ta sẽ làm điều đó tiếp theo.

#### Thêm Tham Chiếu từ Con tới Cha

Để cho node con nhận biết cha của nó, chúng ta cần thêm một trường `parent` vào định nghĩa struct `Node`. 
Vấn đề là xác định kiểu dữ liệu của `parent` nên là gì. Chúng ta biết nó không thể chứa `Rc<T>`, vì điều đó sẽ tạo ra một vòng tham chiếu 
với `leaf.parent` trỏ tới `branch` và `branch.children` trỏ tới `leaf`, dẫn đến giá trị `strong_count` của chúng sẽ không bao giờ về 0.

Xem xét các mối quan hệ theo cách khác, một node cha nên sở hữu các node con của nó: nếu một node cha bị drop, các node con cũng nên bị drop theo. 
Tuy nhiên, một node con không nên sở hữu cha của nó: nếu chúng ta drop một node con, node cha vẫn tồn tại. Đây là trường hợp sử dụng tham chiếu yếu!

Vì vậy, thay vì dùng `Rc<T>`, chúng ta sẽ làm kiểu của `parent` sử dụng `Weak<T>`, cụ thể là `RefCell<Weak<Node>>`. 
Bây giờ định nghĩa struct `Node` của chúng ta trông như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Một node sẽ có thể tham chiếu tới node cha của nó nhưng không sở hữu node cha. 
Trong Listing 15-28, chúng ta cập nhật `main` để sử dụng định nghĩa mới này, 
vì vậy node `leaf` sẽ có cách để tham chiếu tới cha của nó là `branch`:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

<span class="caption">Listing 15-28: Node `leaf` với tham chiếu yếu tới node cha `branch`</span>

Việc tạo node `leaf` trông giống như trong Listing 15-27 ngoại trừ trường `parent`: 
`leaf` bắt đầu mà không có cha, vì vậy chúng ta tạo một instance `Weak<Node>` mới, rỗng.

Tại thời điểm này, khi chúng ta cố lấy tham chiếu tới cha của `leaf` bằng phương thức 
`upgrade`, chúng ta nhận được giá trị `None`. Điều này được thể hiện trong kết quả 
từ câu lệnh `println!` đầu tiên:

```text
leaf parent = None
```

Khi chúng ta tạo node `branch`, nó cũng sẽ có một tham chiếu `Weak<Node>` mới trong 
trường `parent`, vì `branch` không có node cha. Chúng ta vẫn giữ `leaf` là một trong 
các con của `branch`. Khi đã có instance `Node` trong `branch`, chúng ta có thể sửa 
`leaf` để cung cấp cho nó một tham chiếu `Weak<Node>` tới cha của nó. Chúng ta dùng 
phương thức `borrow_mut` trên `RefCell<Weak<Node>>` trong trường `parent` của `leaf`, 
sau đó dùng hàm `Rc::downgrade` để tạo một tham chiếu `Weak<Node>` tới `branch` từ 
`Rc<Node>` trong `branch`.

Khi in lại cha của `leaf`, lần này chúng ta sẽ nhận được biến thể `Some` chứa `branch`: 
bây giờ `leaf` có thể truy cập cha của nó! Khi in `leaf`, chúng ta cũng tránh được 
vòng lặp tham chiếu gây tràn stack như trong Listing 15-26; các tham chiếu `Weak<Node>` 
được in ra dưới dạng `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

Việc không có đầu ra vô hạn cho thấy mã này không tạo ra vòng lặp tham chiếu. Chúng ta 
cũng có thể nhận thấy điều này bằng cách xem các giá trị nhận được từ việc gọi 
`Rc::strong_count` và `Rc::weak_count`.

#### Trực quan hóa các thay đổi của `strong_count` và `weak_count`

Hãy xem cách các giá trị `strong_count` và `weak_count` của các instance `Rc<Node>` 
thay đổi bằng cách tạo một phạm vi con mới và chuyển việc tạo `branch` vào phạm vi đó. 
Bằng cách này, chúng ta có thể thấy điều gì xảy ra khi `branch` được tạo và sau đó bị 
hủy khi nó ra khỏi phạm vi. Các thay đổi được thể hiện trong Listing 15-29:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

<span class="caption">Listing 15-29: Tạo `branch` trong một phạm vi con và 
kiểm tra số lượng tham chiếu mạnh và yếu</span>

Sau khi `leaf` được tạo, `Rc<Node>` của nó có `strong count` là 1 và `weak count` là 0. 
Trong phạm vi con, chúng ta tạo `branch` và liên kết nó với `leaf`, vào thời điểm đó khi 
in ra các số lượng, `Rc<Node>` trong `branch` sẽ có `strong count` là 1 và `weak count` là 1 
(cho `leaf.parent` trỏ tới `branch` với một `Weak<Node>`). Khi in các số lượng của `leaf`, 
chúng ta sẽ thấy nó có `strong count` là 2, vì `branch` bây giờ có một bản clone của 
`Rc<Node>` của `leaf` lưu trong `branch.children`, nhưng vẫn có `weak count` là 0.

Khi phạm vi con kết thúc, `branch` ra khỏi phạm vi và `strong count` của `Rc<Node>` giảm xuống 0, 
vì vậy `Node` của nó bị drop. `weak count` là 1 từ `leaf.parent` không ảnh hưởng đến việc `Node` 
có bị drop hay không, vì vậy chúng ta không gặp rò rỉ bộ nhớ!

Nếu chúng ta thử truy cập parent của `leaf` sau khi kết thúc phạm vi, chúng ta sẽ nhận được 
`None` một lần nữa. Cuối chương trình, `Rc<Node>` trong `leaf` có `strong count` là 1 và 
`weak count` là 0, vì biến `leaf` bây giờ là tham chiếu duy nhất đến `Rc<Node>`.

Tất cả logic quản lý số lượng tham chiếu và việc drop giá trị đều được tích hợp trong 
`Rc<T>` và `Weak<T>` cùng với các implement của trait `Drop`. Bằng cách xác định rằng 
mối quan hệ từ con đến cha nên là một tham chiếu `Weak<T>` trong định nghĩa của `Node`, 
bạn có thể cho các node cha trỏ tới node con và ngược lại mà không tạo ra vòng tham chiếu 
và rò rỉ bộ nhớ.

## Tóm tắt

Chương này trình bày cách sử dụng smart pointer để thực hiện các đảm bảo và đánh đổi khác 
với những gì Rust mặc định thực hiện với các reference thông thường. Kiểu `Box<T>` có kích thước 
cố định và trỏ đến dữ liệu được cấp phát trên heap. Kiểu `Rc<T>` theo dõi số lượng tham 
chiếu đến dữ liệu trên heap để dữ liệu có thể có nhiều chủ sở hữu. Kiểu `RefCell<T>` với 
khả năng mutability nội tại cung cấp cho chúng ta một kiểu có thể sử dụng khi cần một kiểu 
immutable nhưng vẫn cần thay đổi giá trị bên trong; nó cũng thực thi các quy tắc mượn tại 
runtime thay vì compile time.

Chúng ta cũng đã bàn về các trait `Deref` và `Drop`, cho phép nhiều chức năng của smart pointer. 
Chúng ta khám phá các vòng tham chiếu có thể gây rò rỉ bộ nhớ và cách ngăn chặn chúng bằng 
`Weak<T>`.

Nếu chương này khiến bạn hứng thú và bạn muốn triển khai smart pointer của riêng mình, hãy xem 
thêm tại [“The Rustonomicon”][nomicon] để có thêm thông tin hữu ích.

Tiếp theo, chúng ta sẽ bàn về đồng thời trong Rust. Bạn thậm chí sẽ được tìm hiểu về vài smart 
pointer mới.

[nomicon]: ../nomicon/index.html
