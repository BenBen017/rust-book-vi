## Implementing an Object-Oriented Design Pattern

*State pattern* là một mẫu thiết kế hướng đối tượng (object-oriented design
pattern). Cốt lõi của mẫu thiết kế này là chúng ta định nghĩa một tập hợp
các trạng thái mà một giá trị có thể có bên trong. Các trạng thái được
biểu diễn bằng một tập hợp các *state objects*, và hành vi của giá trị thay
đổi dựa trên trạng thái của nó. Chúng ta sẽ làm ví dụ với một struct
blog post có một field để giữ trạng thái, đây sẽ là một state object từ
tập hợp "draft", "review" hoặc "published".

Các state object chia sẻ chức năng: trong Rust, tất nhiên, chúng ta sử dụng
struct và trait thay vì objects và inheritance. Mỗi state object chịu trách
nhiệm cho hành vi riêng của nó và quyết định khi nào nó nên chuyển sang
trạng thái khác. Giá trị giữ state object không biết gì về các hành vi
khác nhau của các trạng thái hoặc khi nào cần chuyển đổi giữa các trạng thái.

Ưu điểm của việc sử dụng state pattern là, khi yêu cầu nghiệp vụ của chương
trình thay đổi, chúng ta sẽ không cần thay đổi mã của giá trị giữ trạng
thái hoặc mã sử dụng giá trị đó. Chúng ta chỉ cần cập nhật mã bên trong
một state object để thay đổi quy tắc của nó hoặc thêm nhiều state object
mới.

Đầu tiên, chúng ta sẽ triển khai state pattern theo cách hướng đối tượng
truyền thống hơn, sau đó sẽ dùng cách tiếp cận tự nhiên hơn trong Rust.
Hãy cùng thực hiện dần dần workflow cho blog post sử dụng state pattern.

Chức năng cuối cùng sẽ trông như sau:

1. Một blog post bắt đầu như một draft rỗng.
2. Khi draft hoàn tất, yêu cầu review bài post.
3. Khi bài post được phê duyệt, nó sẽ được xuất bản (published).
4. Chỉ các blog post đã published mới trả về nội dung để in, nên các post
   chưa được phê duyệt không thể vô tình được xuất bản.

Bất kỳ thay đổi nào khác trên một post cũng không có hiệu lực. Ví dụ, nếu
chúng ta cố gắng approve một draft blog post trước khi yêu cầu review, post
sẽ vẫn là một draft chưa được published.

Listing 17-11 minh họa workflow này dưới dạng code: đây là ví dụ cách sử
dụng API mà chúng ta sẽ triển khai trong một library crate tên là `blog`.
Mã này sẽ chưa compile vì chúng ta chưa triển khai crate `blog`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:all}}
```

<span class="caption">Listing 17-11: Code that demonstrates the desired
behavior we want our `blog` crate to have</span>

Chúng ta muốn cho phép người dùng tạo một draft blog post mới bằng
`Post::new`. Chúng ta muốn cho phép thêm văn bản vào blog post. Nếu cố
gắng lấy nội dung của post ngay lập tức, trước khi được approval, chúng
ta sẽ không nhận được bất kỳ văn bản nào vì post vẫn còn là draft. Chúng
ta đã thêm `assert_eq!` trong code nhằm mục đích minh họa. Một unit test
tuyệt vời cho trường hợp này là assert rằng một draft blog post trả về
chuỗi rỗng từ phương thức `content`, nhưng chúng ta sẽ không viết test
cho ví dụ này.

Tiếp theo, chúng ta muốn cho phép yêu cầu review cho post, và muốn
`content` trả về chuỗi rỗng trong khi đang chờ review. Khi post được
approval, nó sẽ được published, nghĩa là văn bản của post sẽ được trả
về khi gọi `content`.

Chú ý rằng kiểu duy nhất chúng ta tương tác từ crate là `Post`. Kiểu
này sẽ sử dụng state pattern và sẽ giữ một giá trị là một trong ba state
object đại diện cho các trạng thái khác nhau mà post có thể ở — draft,
waiting for review hoặc published. Việc thay đổi từ trạng thái này sang
trạng thái khác sẽ được quản lý nội bộ trong kiểu `Post`. Các state thay
đổi dựa trên các phương thức mà người dùng thư viện gọi trên instance
của `Post`, nhưng họ không cần quản lý trực tiếp việc thay đổi state.
Ngoài ra, người dùng cũng không thể mắc lỗi với các state, ví dụ như
xuất bản post trước khi được review.

### Defining `Post` and Creating a New Instance in the Draft State

Hãy bắt đầu với việc triển khai thư viện! Chúng ta biết cần một struct
`Post` public để giữ nội dung, nên sẽ bắt đầu với định nghĩa struct và
một hàm public `new` liên kết để tạo một instance của `Post`, như được
minh họa trong Listing 17-12. Chúng ta cũng sẽ tạo một trait `State`
private để định nghĩa hành vi mà tất cả state object của `Post` phải có.

Sau đó, `Post` sẽ giữ một trait object `Bo

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-12/src/lib.rs}}
```

<span class="caption">Listing 17-12: Definition of a `Post` struct and a `new`
function that creates a new `Post` instance, a `State` trait, and a `Draft`
struct</span>

Trait `State` định nghĩa hành vi được chia sẻ bởi các trạng thái khác nhau của post.
Các state object là `Draft`, `PendingReview` và `Published`, tất cả sẽ triển khai
trait `State`. Hiện tại, trait chưa có phương thức nào, và chúng ta sẽ bắt đầu
bằng việc định nghĩa trạng thái `Draft` vì đây là trạng thái mà post sẽ bắt đầu.

Khi tạo một `Post` mới, chúng ta gán field `state` với giá trị `Some` chứa một
`Box`. `Box` này trỏ tới một instance mới của struct `Draft`. Điều này đảm bảo
mỗi khi tạo một `Post` mới, nó sẽ bắt đầu ở trạng thái draft. Vì field `state`
của `Post` là private, không có cách nào tạo một `Post` ở trạng thái khác! Trong
hàm `Post::new`, chúng ta gán field `content` là một `String` mới rỗng.

### Storing the Text of the Post Content

Chúng ta đã thấy trong Listing 17-11 rằng chúng ta muốn gọi một phương thức
có tên `add_text` và truyền vào một `&str`, nội dung này sẽ được thêm làm
text content của blog post. Chúng ta triển khai phương thức này thay vì để
field `content` là `pub`, để sau này có thể cài đặt một phương thức kiểm soát
cách dữ liệu của field `content` được đọc. Phương thức `add_text` khá đơn
giản, nên chúng ta sẽ thêm phần cài đặt trong Listing 17-13 vào khối `impl
Post`:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-13/src/lib.rs:here}}
```

<span class="caption">Listing 17-13: Implementing the `add_text` method to add
text to a post’s `content`</span>

Phương thức `add_text` nhận một tham chiếu mutable tới `self`, vì chúng ta
thay đổi instance của `Post` mà chúng ta gọi `add_text` trên đó. Sau đó,
chúng ta gọi `push_str` trên `String` trong `content` và truyền đối số
`text` để thêm vào `content` đã lưu. Hành vi này không phụ thuộc vào trạng
thái hiện tại của post, nên nó không thuộc về state pattern. Phương thức
`add_text` không tương tác với field `state` chút nào, nhưng nó là một
phần của hành vi mà chúng ta muốn hỗ trợ.

### Ensuring the Content of a Draft Post Is Empty

Ngay cả khi chúng ta đã gọi `add_text` và thêm một số nội dung vào post,
chúng ta vẫn muốn phương thức `content` trả về một chuỗi rỗng vì post
vẫn đang ở trạng thái draft, như được minh họa ở dòng 7 của Listing 17-11.
Hiện tại, chúng ta sẽ triển khai phương thức `content` với cách đơn giản
nhất để đáp ứng yêu cầu này: luôn trả về một chuỗi rỗng. Chúng ta sẽ
thay đổi sau khi triển khai khả năng thay đổi trạng thái của post để nó
có thể được published. Cho đến nay, post chỉ có thể ở trạng thái draft,
nên nội dung post luôn phải trống. Listing 17-14 minh họa phần cài đặt
placeholder này:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-14/src/lib.rs:here}}
```

<span class="caption">Listing 17-14: Adding a placeholder implementation for
the `content` method on `Post` that always returns an empty string slice</span>

Với phương thức `content` được thêm này, mọi thứ trong Listing 17-11 đến
dòng 7 hoạt động như mong muốn.

### Requesting a Review of the Post Changes Its State

Tiếp theo, chúng ta cần thêm chức năng để yêu cầu review cho một post,
điều này sẽ thay đổi trạng thái của nó từ `Draft` sang `PendingReview`.
Listing 17-15 minh họa đoạn code này:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-15/src/lib.rs:here}}
```

<span class="caption">Listing 17-15: Implementing `request_review` methods on
`Post` and the `State` trait</span>

Chúng ta tạo cho `Post` một phương thức public tên là `request_review` nhận
một tham chiếu mutable tới `self`. Sau đó chúng ta gọi phương thức
`request_review` nội bộ trên trạng thái hiện tại của `Post`, và phương thức
`request_review` thứ hai này sẽ tiêu thụ trạng thái hiện tại và trả về trạng
thái mới.

Chúng ta thêm phương thức `request_review` vào trait `State`; tất cả các
kiểu triển khai trait này giờ đây phải triển khai phương thức
`request_review`. Lưu ý rằng thay vì có `self`, `&self` hoặc `&mut self`
làm tham số đầu tiên, chúng ta có `self: Box<Self>`. Cú pháp này nghĩa là
phương thức chỉ hợp lệ khi được gọi trên một `Box` chứa kiểu đó. Cú pháp
này nhận quyền sở hữu của `Box<Self>`, làm vô hiệu hóa trạng thái cũ để
giá trị trạng thái của `Post` có thể biến đổi thành trạng thái mới.

Để tiêu thụ trạng thái cũ, phương thức `request_review` cần nhận quyền sở
hữu của giá trị trạng thái. Đây là lý do `Option` trong field `state` của
`Post` xuất hiện: chúng ta gọi phương thức `take` để lấy giá trị `Some`
ra khỏi field `state` và để lại `None` tại chỗ, vì Rust không cho phép
có các field chưa được gán trong struct. Điều này cho phép chúng ta di
chuyển giá trị `state` ra khỏi `Post` thay vì chỉ mượn. Sau đó, chúng ta
gán giá trị `state` của post bằng kết quả của thao tác này.

Chúng ta cần gán tạm `state` là `None` thay vì gán trực tiếp với code như
`self.state = self.state.request_review();` để có quyền sở hữu giá trị
`state`. Điều này đảm bảo `Post` không thể dùng giá trị `state` cũ sau
khi chúng ta đã biến đổi nó thành trạng thái mới.

Phương thức `request_review` trên `Draft` trả về một instance mới, được
box của struct `PendingReview`, đại diện cho trạng thái khi một post đang
chờ review. Struct `PendingReview` cũng triển khai phương thức
`request_review` nhưng không thực hiện biến đổi nào. Thay vào đó, nó trả
về chính nó, vì khi yêu cầu review trên một post đã ở trạng thái
`PendingReview`, nó vẫn giữ trạng thái `PendingReview`.

Giờ đây chúng ta bắt đầu thấy được lợi ích của state pattern: phương thức
`request_review` trên `Post` giống nhau bất kể giá trị `state` là gì.
Mỗi state chịu trách nhiệm cho các quy tắc riêng của nó.

Chúng ta sẽ giữ nguyên phương thức `content` trên `Post`, trả về chuỗi
rỗng. Giờ đây chúng ta có thể có một `Post` ở trạng thái `PendingReview`
cũng như `Draft`, nhưng chúng ta muốn hành vi giống nhau trong trạng
thái `PendingReview`. Listing 17-11 giờ hoạt động đến dòng 10!

<!-- Old headings. Do not remove or links may break. -->
<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

### Adding `approve` to Change the Behavior of `content`

Phương thức `approve` sẽ tương tự như phương thức `request_review`: nó sẽ
gán field `state` thành giá trị mà trạng thái hiện tại xác định khi trạng
thái đó được approve, như minh họa trong Listing 17-16:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-16/src/lib.rs:here}}
```

<span class="caption">Listing 17-16: Implementing the `approve` method on
`Post` and the `State` trait</span>

Chúng ta thêm phương thức `approve` vào trait `State` và thêm một struct mới
triển khai `State`, đó là trạng thái `Published`.

Tương tự như cách `request_review` trên `PendingReview` hoạt động, nếu chúng
ta gọi phương thức `approve` trên `Draft`, sẽ không có hiệu lực vì `approve`
trả về chính `self`. Khi gọi `approve` trên `PendingReview`, nó trả về một
instance mới được box của struct `Published`. Struct `Published` triển khai
trait `State`, và cả phương thức `request_review` lẫn `approve` đều trả về
chính nó, vì trong các trường hợp này post nên giữ trạng thái
`Published`.

Bây giờ chúng ta cần cập nhật phương thức `content` trên `Post`. Chúng ta
muốn giá trị trả về từ `content` phụ thuộc vào trạng thái hiện tại của
`Post`, nên chúng ta sẽ để `Post` ủy quyền cho một phương thức `content`
được định nghĩa trên field `state` của nó, như minh họa trong Listing 17-17:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-17/src/lib.rs:here}}
```

<span class="caption">Listing 17-17: Updating the `content` method on `Post` to
delegate to a `content` method on `State`</span>

Vì mục tiêu là giữ tất cả các quy tắc này bên trong các struct triển khai
`State`, chúng ta gọi phương thức `content` trên giá trị trong `state` và
truyền instance của post (tức là `self`) làm đối số. Sau đó, chúng ta trả
về giá trị được trả từ việc sử dụng phương thức `content` trên giá trị
`state`.

Chúng ta gọi phương thức `as_ref` trên `Option` vì muốn một tham chiếu tới
giá trị bên trong `Option` thay vì quyền sở hữu giá trị. Vì `state` là
`Option<Box<dyn State>>`, khi gọi `as_ref`, chúng ta nhận được
`Option<&Box<dyn State>>`. Nếu không gọi `as_ref`, chúng ta sẽ gặp lỗi
vì không thể di chuyển `state` ra khỏi `&self` được mượn của tham số
hàm.

Sau đó, chúng ta gọi phương thức `unwrap`, mà chúng ta biết sẽ không bao
giờ panic, vì các phương thức trên `Post` đảm bảo rằng `state` luôn chứa
giá trị `Some` khi các phương thức đó hoàn thành. Đây là một trong những
trường hợp đã bàn trong phần [“Cases In Which You Have More Information Than the
Compiler”][more-info-than-rustc]<!-- ignore --> của Chương 9, khi chúng ta biết
giá trị `None` là không thể xảy ra, mặc dù compiler không hiểu được điều đó.

Tại thời điểm này, khi gọi `content` trên `&Box<dyn State>`, deref coercion
sẽ có hiệu lực trên `&` và `Box` nên phương thức `content` cuối cùng sẽ
được gọi trên kiểu triển khai trait `State`. Điều đó có nghĩa là chúng ta
cần thêm `content` vào định nghĩa trait `State`, và đó là nơi chúng ta sẽ
đặt logic cho nội dung trả về tùy thuộc vào trạng thái hiện tại, như minh
họa trong Listing 17-18:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-18/src/lib.rs:here}}
```

<span class="caption">Listing 17-18: Adding the `content` method to the `State`
trait</span>

Chúng ta thêm một triển khai mặc định cho phương thức `content` trả về một
chuỗi rỗng. Điều này có nghĩa là chúng ta không cần triển khai `content`
trên các struct `Draft` và `PendingReview`. Struct `Published` sẽ override
phương thức `content` và trả về giá trị trong `post.content`.

Lưu ý rằng chúng ta cần chú thích lifetime cho phương thức này, như đã
bàn trong Chương 10. Chúng ta đang lấy một tham chiếu tới `post` làm
đối số và trả về một tham chiếu tới một phần của `post`, nên lifetime của
tham chiếu trả về liên quan tới lifetime của tham số `post`.

Và xong—tất cả Listing 17-11 giờ hoạt động! Chúng ta đã triển khai state
pattern với các quy tắc của workflow blog post. Logic liên quan tới các
quy tắc nằm trong các state object thay vì rải rác trong `Post`.

> #### Why Not An Enum?
>
> Bạn có thể thắc mắc tại sao chúng ta không dùng một `enum` với các trạng thái
> post khác nhau làm các variant. Đây chắc chắn là một giải pháp khả thi, hãy
> thử và so sánh kết quả cuối cùng để xem bạn thích cách nào! Một nhược điểm
> khi dùng enum là mọi nơi kiểm tra giá trị enum sẽ cần một biểu thức `match`
> hoặc tương tự để xử lý tất cả các variant có thể. Điều này có thể trở nên
> lặp đi lặp lại nhiều hơn so với giải pháp trait object này.

### Trade-offs of the State Pattern

Chúng ta đã thấy rằng Rust có khả năng triển khai state pattern theo hướng
object-oriented để đóng gói các kiểu hành vi khác nhau mà một post có thể
có trong từng trạng thái. Các phương thức trên `Post` không biết gì về
các hành vi khác nhau. Với cách chúng ta tổ chức code, chỉ cần nhìn vào
một nơi để biết các cách mà một post đã được publish có thể hành xử: đó
là phần triển khai trait `State` trên struct `Published`.

Nếu tạo một triển khai thay thế không dùng state pattern, chúng ta có thể
dùng biểu thức `match` trong các phương thức của `Post` hoặc thậm chí trong
code `main` để kiểm tra trạng thái của post và thay đổi hành vi ở những nơi
đó. Điều này có nghĩa là chúng ta phải xem ở nhiều nơi để hiểu tất cả
các tác động khi post ở trạng thái published! Số lượng chỗ phải xem sẽ
tăng lên khi thêm nhiều trạng thái: mỗi biểu thức `match` đó cần thêm
một nhánh mới.

Với state pattern, các phương thức `Post` và nơi sử dụng `Post` không cần
biểu thức `match`, và để thêm trạng thái mới, chỉ cần thêm một struct mới
và triển khai các phương thức trait trên struct đó.

Triển khai theo state pattern dễ dàng mở rộng để thêm nhiều chức năng hơn.
Để thấy sự đơn giản khi bảo trì code dùng state pattern, thử một số gợi ý:

* Thêm phương thức `reject` thay đổi trạng thái post từ `PendingReview`
  về `Draft`.
* Yêu cầu hai lần gọi `approve` trước khi trạng thái được chuyển sang
  `Published`.
* Cho phép người dùng thêm nội dung text chỉ khi post ở trạng thái
  `Draft`. Gợi ý: để state object chịu trách nhiệm với những gì có thể
  thay đổi về content nhưng không chịu trách nhiệm sửa đổi `Post`.

Một nhược điểm của state pattern là, vì các state triển khai các chuyển
đổi giữa các state, một số state bị coupling với nhau. Nếu thêm một state
giữa `PendingReview` và `Published`, như `Scheduled`, chúng ta sẽ phải
thay đổi code trong `PendingReview` để chuyển sang `Scheduled`. Sẽ bớt
công sức hơn nếu `PendingReview` không cần thay đổi khi thêm trạng thái
mới, nhưng điều đó đồng nghĩa phải chuyển sang một design pattern khác.

Một nhược điểm khác là chúng ta đã lặp lại một số logic. Để loại bỏ một
số trùng lặp, có thể thử tạo triển khai mặc định cho các phương thức
`request_review` và `approve` trên trait `State` trả về `self`; tuy nhiên,
điều này vi phạm object safety, vì trait không biết chính xác `self` cụ thể
sẽ là gì. Chúng ta muốn dùng `State` như một trait object, nên các phương
thức cần object safe.

Các trùng lặp khác gồm các triển khai tương tự của `request_review` và
`approve` trên `Post`. Cả hai phương thức đều ủy quyền cho triển khai
của cùng phương thức trên giá trị trong field `state` của `Option` và
gán giá trị mới của field `state` bằng kết quả trả về. Nếu có nhiều
phương thức trên `Post` theo mẫu này, có thể cân nhắc định nghĩa một macro
để loại bỏ sự lặp lại (xem phần [“Macros”][macros]<!-- ignore --> trong
Chương 19).

Bằng việc triển khai state pattern chính xác như định nghĩa cho các ngôn
ngữ hướng đối tượng, chúng ta chưa tận dụng hết sức mạnh của Rust. Hãy xem
một số thay đổi có thể thực hiện trên crate `blog` để biến các trạng thái
và chuyển đổi không hợp lệ thành lỗi biên dịch.

#### Encoding States and Behavior as Types

Chúng ta sẽ minh họa cách suy nghĩ lại state pattern để nhận được một bộ
trade-off khác. Thay vì đóng gói hoàn toàn các state và chuyển đổi sao cho
code bên ngoài không biết gì về chúng, chúng ta sẽ mã hóa các state thành
các kiểu khác nhau. Do đó, hệ thống kiểm tra kiểu của Rust sẽ ngăn việc
sử dụng các post đang ở trạng thái draft ở những nơi chỉ cho phép post
đã publish bằng cách phát sinh lỗi biên dịch.

Hãy xem xét phần đầu của `main` trong Listing 17-11:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:here}}
```

Chúng ta vẫn cho phép tạo các post mới ở trạng thái draft bằng `Post::new`
và thêm text vào nội dung của post. Nhưng thay vì có một phương thức `content`
trên post draft trả về chuỗi rỗng, chúng ta sẽ làm sao để post draft hoàn
toàn không có phương thức `content`. Như vậy, nếu cố gắng lấy nội dung
của post draft, trình biên dịch sẽ báo lỗi rằng phương thức không tồn tại.
Kết quả là, sẽ không thể vô tình hiển thị nội dung post draft trong
môi trường production, vì code đó thậm chí sẽ không compile được. Listing
17-19 trình bày định nghĩa của struct `Post` và struct `DraftPost`, cũng
như các phương thức trên mỗi struct:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-19/src/lib.rs}}
```

<span class="caption">Listing 17-19: A `Post` with a `content` method and a
`DraftPost` without a `content` method</span>

Cả hai struct `Post` và `DraftPost` đều có một field `content` private
lưu trữ nội dung blog post. Các struct không còn field `state` vì chúng
ta đang chuyển việc mã hóa trạng thái sang các kiểu của struct. Struct
`Post` sẽ đại diện cho một post đã publish, và nó có phương thức `content`
trả về giá trị của `content`.

Chúng ta vẫn có một hàm `Post::new`, nhưng thay vì trả về một instance
của `Post`, nó trả về một instance của `DraftPost`. Vì `content` là private
và không có hàm nào trả về `Post`, hiện tại không thể tạo một instance
của `Post`.

Struct `DraftPost` có phương thức `add_text`, nên chúng ta có thể thêm text
vào `content` như trước, nhưng lưu ý rằng `DraftPost` không có phương thức
`content` được định nghĩa! Vì vậy chương trình đảm bảo tất cả các post
bắt đầu là draft, và post draft không có nội dung sẵn sàng để hiển thị.
Bất kỳ cố gắng vượt qua các giới hạn này sẽ dẫn đến lỗi biên dịch.

#### Implementing Transitions as Transformations into Different Types

Vậy làm sao để có một post đã publish? Chúng ta muốn đảm bảo quy tắc
rằng một post draft phải được review và approve trước khi publish. Một
post ở trạng thái pending review vẫn không được hiển thị nội dung. Hãy
triển khai các giới hạn này bằng cách thêm một struct khác là
`PendingReviewPost`, định nghĩa phương thức `request_review` trên
`DraftPost` trả về `PendingReviewPost`, và định nghĩa phương thức `approve`
trên `PendingReviewPost` trả về một `Post`, như minh họa trong Listing 17-20:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-20/src/lib.rs:here}}
```

<span class="caption">Listing 17-20: A `PendingReviewPost` that gets created by
calling `request_review` on `DraftPost` and an `approve` method that turns a
`PendingReviewPost` into a published `Post`</span>

Các phương thức `request_review` và `approve` nhận quyền sở hữu của `self`,
do đó tiêu thụ các instance `DraftPost` và `PendingReviewPost` và biến
chúng thành `PendingReviewPost` và `Post` đã publish, tương ứng. Như vậy,
sau khi gọi `request_review` trên một post draft, sẽ không còn tồn tại
instance `DraftPost` nào, và tương tự cho các trạng thái khác. Struct
`PendingReviewPost` không có phương thức `content` được định nghĩa, vì
vậy cố gắng đọc nội dung của nó sẽ dẫn đến lỗi biên dịch, tương tự như
`DraftPost`. Vì cách duy nhất để có một instance `Post` đã publish có
phương thức `content` là gọi `approve` trên `PendingReviewPost`, và cách
duy nhất để có một `PendingReviewPost` là gọi `request_review` trên
`DraftPost`, chúng ta đã mã hóa workflow của blog post vào hệ thống kiểu.

Nhưng chúng ta cũng phải thực hiện một số thay đổi nhỏ cho `main`. Các
phương thức `request_review` và `approve` trả về các instance mới thay vì
sửa đổi struct hiện tại, vì vậy chúng ta cần thêm các gán `let post =`
shadowing để lưu các instance trả về. Chúng ta cũng không thể giữ các
assertion về nội dung của draft và pending review posts là chuỗi rỗng,
và cũng không cần chúng nữa: code cố gắng sử dụng content của các post
trong những trạng thái đó sẽ không compile được. Code cập nhật trong `main`
được trình bày trong Listing 17-21:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-21/src/main.rs}}
```

<span class="caption">Listing 17-21: Modifications to `main` to use the new
implementation of the blog post workflow</span>

Những thay đổi cần thiết để gán lại `post` trong `main` nghĩa là
implementation này không hoàn toàn tuân theo state pattern hướng đối
tượng nữa: các chuyển đổi giữa các trạng thái không còn được
encapsulate hoàn toàn bên trong `Post`. Tuy nhiên, điểm lợi là các
trạng thái không hợp lệ giờ đã không thể xảy ra nhờ hệ thống kiểu và
việc kiểm tra kiểu diễn ra tại thời điểm biên dịch! Điều này đảm bảo
một số lỗi, chẳng hạn như hiển thị nội dung của post chưa publish, sẽ
được phát hiện trước khi vào production.

Hãy thử thực hiện các tác vụ gợi ý ở đầu phần này trên crate `blog`
sau Listing 17-21 để xem bạn đánh giá thế nào về thiết kế phiên bản
mã nguồn này. Lưu ý rằng một số tác vụ có thể đã được hoàn thành
trong thiết kế này.

Chúng ta thấy rằng mặc dù Rust có thể triển khai các pattern hướng
đối tượng, các pattern khác, chẳng hạn như mã hóa trạng thái vào
hệ thống kiểu, cũng có sẵn trong Rust. Các pattern này có các
trade-offs khác nhau. Mặc dù bạn có thể rất quen thuộc với các
pattern hướng đối tượng, suy nghĩ lại vấn đề để tận dụng các
tính năng của Rust có thể mang lại lợi ích, chẳng hạn như ngăn
một số lỗi ngay từ thời điểm biên dịch. Pattern hướng đối tượng
không phải lúc nào cũng là giải pháp tốt nhất trong Rust do một
số đặc điểm, như ownership, mà các ngôn ngữ hướng đối tượng không
có.

## Tóm tắt

Dù bạn có coi Rust là ngôn ngữ hướng đối tượng hay không sau khi
đọc chương này, bạn đã biết rằng có thể sử dụng trait object để
có một số tính năng hướng đối tượng trong Rust. Dynamic dispatch
có thể mang lại cho code của bạn một số tính linh hoạt đổi lấy
một chút chi phí runtime. Bạn có thể dùng linh hoạt này để triển
khai các pattern hướng đối tượng giúp maintainability của code tốt
hơn. Rust cũng có các tính năng khác, như ownership, mà các ngôn
ngữ hướng đối tượng không có. Một pattern hướng đối tượng sẽ
không phải lúc nào cũng là cách tốt nhất để tận dụng sức mạnh
của Rust, nhưng là một lựa chọn khả thi.

Tiếp theo, chúng ta sẽ xem các pattern, là một trong những tính
năng của Rust cho phép nhiều linh hoạt. Chúng ta đã xem qua chúng
ngắn gọn trong suốt cuốn sách nhưng chưa thấy đầy đủ khả năng của
chúng. Hãy tiếp tục!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch19-06-macros.html#macros
