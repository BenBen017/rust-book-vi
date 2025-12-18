## Xây dựng Web Server Đơn Luồng

Chúng ta sẽ bắt đầu bằng việc tạo một web server đơn luồng hoạt động. Trước khi bắt đầu, hãy xem qua tổng quan nhanh về các giao thức liên quan đến việc xây dựng web server. Chi tiết về các giao thức này vượt ngoài phạm vi của cuốn sách, nhưng một cái nhìn tổng quan sẽ cung cấp thông tin cần thiết.

Hai giao thức chính liên quan đến web server là *Hypertext Transfer Protocol* *(HTTP)* và *Transmission Control Protocol* *(TCP)*. Cả hai đều là các giao thức *yêu cầu-phản hồi*, nghĩa là *client* khởi tạo yêu cầu và *server* lắng nghe các yêu cầu và trả về phản hồi cho client. Nội dung của các yêu cầu và phản hồi được xác định bởi các giao thức này.

TCP là giao thức cấp thấp hơn, mô tả chi tiết cách thông tin di chuyển từ server này sang server khác nhưng không xác định thông tin đó là gì. HTTP xây dựng trên TCP bằng cách xác định nội dung của các yêu cầu và phản hồi. Về mặt kỹ thuật, có thể sử dụng HTTP với các giao thức khác, nhưng trong phần lớn các trường hợp, HTTP gửi dữ liệu qua TCP. Chúng ta sẽ làm việc với các byte thô của yêu cầu và phản hồi TCP và HTTP.

### Lắng nghe Kết nối TCP

Web server của chúng ta cần lắng nghe kết nối TCP, đây là phần đầu tiên chúng ta sẽ thực hiện. Thư viện chuẩn cung cấp module `std::net` cho phép chúng ta làm điều này. Hãy tạo một dự án mới theo cách thông thường:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Bây giờ, hãy nhập mã trong Listing 20-1 vào *src/main.rs* để bắt đầu. Mã này sẽ lắng nghe tại địa chỉ cục bộ `127.0.0.1:7878` cho các luồng TCP đến. Khi nhận được một luồng đến, nó sẽ in ra `Connection established!`.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-01/src/main.rs}}
```

<span class="caption">Listing 20-1: Lắng nghe các luồng đến và in thông báo khi nhận được luồng</span>

Sử dụng `TcpListener`, chúng ta có thể lắng nghe các kết nối TCP tại địa chỉ `127.0.0.1:7878`. Trong địa chỉ này, phần trước dấu hai chấm là địa chỉ IP đại diện cho máy tính của bạn (điều này giống nhau trên mọi máy và không đại diện cho máy của tác giả), và `7878` là cổng. Chúng ta chọn cổng này vì hai lý do: HTTP thường không được sử dụng trên cổng này nên server của chúng ta khó bị xung đột với bất kỳ web server nào đang chạy trên máy của bạn, và 7878 gợi nhớ đến từ *rust* khi bấm trên bàn phím số điện thoại.

Hàm `bind` trong trường hợp này hoạt động giống như hàm `new` vì nó trả về một instance `TcpListener` mới. Hàm được gọi là `bind` vì trong mạng, việc kết nối đến một cổng để lắng nghe được gọi là “binding to a port”.

Hàm `bind` trả về một `Result<T, E>`, điều này cho thấy việc binding có thể thất bại. Ví dụ, kết nối tới cổng 80 yêu cầu quyền quản trị (người dùng thường không có quyền có thể lắng nghe các cổng cao hơn 1023), nên nếu thử kết nối tới cổng 80 mà không có quyền quản trị, binding sẽ thất bại. Binding cũng sẽ không thành công nếu chạy hai instance của chương trình cùng lúc trên cùng một cổng. Vì chúng ta đang viết một server cơ bản cho mục đích học tập, chúng ta sẽ không xử lý những lỗi này; thay vào đó, sử dụng `unwrap` để dừng chương trình nếu có lỗi xảy ra.

Phương thức `incoming` trên `TcpListener` trả về một iterator cung cấp một chuỗi các luồng (cụ thể là các luồng kiểu `TcpStream`). Một *stream* đại diện cho một kết nối mở giữa client và server. Một *connection* là tên gọi cho toàn bộ quá trình yêu cầu và phản hồi trong đó client kết nối đến server, server sinh ra phản hồi, và server đóng kết nối. Vì vậy, chúng ta sẽ đọc từ `TcpStream` để xem client gửi gì và sau đó ghi phản hồi vào luồng để gửi dữ liệu trở lại client. Nhìn chung, vòng lặp `for` này sẽ xử lý từng kết nối theo thứ tự và tạo ra một chuỗi các luồng để chúng ta xử lý.

Hiện tại, cách xử lý luồng của chúng ta chỉ gồm việc gọi `unwrap` để kết thúc chương trình nếu luồng có lỗi; nếu không có lỗi, chương trình sẽ in ra thông báo. Chúng ta sẽ thêm nhiều chức năng hơn cho trường hợp thành công trong listing tiếp theo. Lý do có thể nhận được lỗi từ phương thức `incoming` khi client kết nối đến server là vì chúng ta không thực sự lặp qua các kết nối mà lặp qua *các nỗ lực kết nối*. Kết nối có thể không thành công vì nhiều lý do, nhiều lý do phụ thuộc vào hệ điều hành. Ví dụ, nhiều hệ điều hành giới hạn số lượng kết nối mở đồng thời mà chúng có thể hỗ trợ; các nỗ lực kết nối mới vượt quá số lượng đó sẽ gây lỗi cho đến khi một số kết nối mở được đóng.

Hãy thử chạy đoạn mã này! Thực thi `cargo run` trong terminal và mở *127.0.0.1:7878* trong trình duyệt web. Trình duyệt sẽ hiển thị thông báo lỗi như “Connection reset,” vì server hiện tại chưa gửi dữ liệu trả về. Nhưng khi nhìn vào terminal, bạn sẽ thấy nhiều thông báo được in ra khi trình duyệt kết nối tới server!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Đôi khi bạn sẽ thấy nhiều thông báo được in ra cho cùng một yêu cầu từ trình duyệt; lý do có thể là trình duyệt đang gửi yêu cầu cho trang chính cũng như cho các tài nguyên khác, chẳng hạn như biểu tượng *favicon.ico* xuất hiện trong tab trình duyệt.

Cũng có thể trình duyệt đang cố gắng kết nối tới server nhiều lần vì server không phản hồi dữ liệu. Khi `stream` ra khỏi phạm vi và bị drop ở cuối vòng lặp, kết nối sẽ được đóng theo cách `drop` được triển khai. Trình duyệt đôi khi xử lý các kết nối bị đóng bằng cách thử lại, vì vấn đề có thể chỉ là tạm thời. Yếu tố quan trọng là chúng ta đã thành công trong việc lấy được handle tới một kết nối TCP!

Hãy nhớ dừng chương trình bằng cách nhấn <span class="keystroke">ctrl-c</span> khi bạn hoàn tất việc chạy một phiên bản code. Sau đó khởi động lại chương trình bằng lệnh `cargo run` sau khi bạn thực hiện các thay đổi để đảm bảo bạn đang chạy phiên bản code mới nhất.

### Đọc Request

Hãy triển khai chức năng đọc request từ trình duyệt! Để tách biệt việc lấy kết nối và xử lý kết nối, chúng ta sẽ tạo một hàm mới để xử lý kết nối. Trong hàm `handle_connection` mới này, chúng ta sẽ đọc dữ liệu từ TCP stream và in ra để có thể quan sát dữ liệu được gửi từ trình duyệt. Thay đổi code để trông giống như Listing 20-2.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-02/src/main.rs}}
```

<span class="caption">Listing 20-2: Đọc từ `TcpStream` và in dữ liệu ra</span>

Chúng ta đưa `std::io::prelude` và `std::io::BufReader` vào phạm vi để có quyền truy cập vào các trait và kiểu giúp đọc và ghi từ stream. Trong vòng lặp `for` ở hàm `main`, thay vì in thông báo rằng chúng ta đã nhận được kết nối, giờ chúng ta gọi hàm `handle_connection` mới và truyền `stream` vào.

Trong hàm `handle_connection`, chúng ta tạo một instance `BufReader` bao quanh tham chiếu mutable tới `stream`. `BufReader` thêm bộ đệm bằng cách quản lý các gọi tới các phương thức của trait `std::io::Read` cho chúng ta.

Chúng ta tạo biến `http_request` để thu thập các dòng của request mà trình duyệt gửi đến server. Chúng ta chỉ định rằng muốn thu thập các dòng này trong một vector bằng cách thêm chú thích kiểu `Vec<_>`.

`BufReader` triển khai trait `std::io::BufRead`, trait này cung cấp phương thức `lines`. Phương thức `lines` trả về một iterator các giá trị `Result<String, std::io::Error>` bằng cách tách luồng dữ liệu mỗi khi gặp byte newline. Để lấy được từng `String`, chúng ta dùng `map` và `unwrap` mỗi `Result`. `Result` có thể là lỗi nếu dữ liệu không hợp lệ UTF-8 hoặc có vấn đề khi đọc từ stream. Trong chương trình sản xuất nên xử lý lỗi một cách mềm dẻo hơn, nhưng chúng ta chọn dừng chương trình trong trường hợp lỗi để đơn giản.

Trình duyệt báo hiệu kết thúc một HTTP request bằng việc gửi hai ký tự newline liên tiếp, vì vậy để lấy một request từ stream, chúng ta lấy các dòng cho tới khi gặp một dòng trống. Khi đã thu thập xong các dòng vào vector, chúng ta in chúng ra sử dụng định dạng debug đẹp để có thể xem các lệnh mà trình duyệt gửi tới server.

Hãy thử chạy code này! Khởi động chương trình và thực hiện một request từ trình duyệt lần nữa. Lưu ý rằng trình duyệt vẫn sẽ hiển thị trang lỗi, nhưng đầu ra của chương trình trong terminal sẽ trông giống như sau:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Tùy vào trình duyệt, bạn có thể thấy đầu ra hơi khác nhau. Giờ khi chúng ta đã in dữ liệu request ra, chúng ta có thể thấy lý do vì sao một request từ trình duyệt lại tạo ra nhiều kết nối bằng cách nhìn vào đường dẫn sau `GET` trong dòng đầu tiên của request. Nếu các kết nối lặp lại đều yêu cầu `/*`, chúng ta biết trình duyệt đang cố gắng lấy `/*` nhiều lần vì chưa nhận được phản hồi từ chương trình của chúng ta.

Hãy phân tích dữ liệu request này để hiểu trình duyệt đang yêu cầu gì từ chương trình của chúng ta.

### Xem kỹ một HTTP Request

HTTP là một giao thức dựa trên văn bản, và một request có định dạng như sau:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Dòng đầu tiên là *request line* chứa thông tin về yêu cầu mà client gửi tới. Phần đầu của request line chỉ ra *method* đang được sử dụng, chẳng hạn như `GET` hoặc `POST`, mô tả cách client thực hiện request này. Client của chúng ta sử dụng request `GET`, có nghĩa là nó đang yêu cầu lấy thông tin.

Phần tiếp theo của request line là `/*`, chỉ ra *Uniform Resource Identifier* *(URI)* mà client đang yêu cầu: URI gần giống, nhưng không hoàn toàn giống, với *Uniform Resource Locator* *(URL)*. Sự khác biệt giữa URI và URL không quan trọng trong chương này, nhưng chuẩn HTTP dùng thuật ngữ URI, nên ta có thể tạm thay thế URL bằng URI trong suy nghĩ.

Phần cuối cùng là phiên bản HTTP mà client đang sử dụng, và sau đó request line kết thúc bằng *CRLF sequence*. (CRLF là viết tắt của *carriage return* và *line feed*, các thuật ngữ từ thời máy đánh chữ!) Chuỗi CRLF cũng có thể viết là `\r\n`, trong đó `\r` là carriage return và `\n` là line feed. CRLF phân tách request line với phần dữ liệu request còn lại. Khi in ra, CRLF hiển thị như xuống dòng thay vì `\r\n`.

Nhìn vào request line từ kết quả chương trình, ta thấy `GET` là method, `/*` là request URI, và `HTTP/1.1` là version.

Sau request line, các dòng còn lại bắt đầu từ `Host:` trở đi là các header. Các request `GET` không có body.

Hãy thử gửi request từ một trình duyệt khác hoặc yêu cầu một địa chỉ khác, ví dụ *127.0.0.1:7878/test*, để thấy dữ liệu request thay đổi như thế nào.

Giờ khi đã biết trình duyệt đang yêu cầu gì, hãy gửi dữ liệu phản hồi lại!

### Viết Response

Chúng ta sẽ triển khai việc gửi dữ liệu phản hồi cho client. Response có định dạng như sau:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Dòng đầu tiên là một *dòng trạng thái* chứa phiên bản HTTP được sử dụng trong
phản hồi, một mã trạng thái dạng số tóm tắt kết quả của yêu cầu, và một
cụm từ lý do cung cấp mô tả văn bản về mã trạng thái. Sau chuỗi CRLF là
các header (nếu có), một chuỗi CRLF khác, và phần thân của phản hồi.

Dưới đây là một ví dụ về phản hồi sử dụng HTTP phiên bản 1.1, có mã trạng
thái 200, cụm từ lý do OK, không có header, và không có thân:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Mã trạng thái 200 là phản hồi thành công chuẩn. Văn bản là một phản hồi HTTP
nhỏ thành công. Hãy ghi điều này vào stream như phản hồi của chúng ta cho
một yêu cầu thành công! Từ hàm `handle_connection`, xóa `println!` đang
in dữ liệu yêu cầu và thay thế bằng mã trong Listing 20-3.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-03/src/main.rs:here}}
```

<span class="caption">Listing 20-3: Ghi một phản hồi HTTP nhỏ thành công vào
stream</span>

Dòng mới đầu tiên định nghĩa biến `response` chứa dữ liệu thông điệp thành
công. Sau đó chúng ta gọi `as_bytes` trên `response` để chuyển dữ liệu
chuỗi thành bytes. Phương thức `write_all` trên `stream` nhận một `&[u8]`
và gửi trực tiếp các byte đó xuống kết nối. Vì thao tác `write_all` có thể
thất bại, chúng ta dùng `unwrap` trên kết quả lỗi như trước. Một lần nữa, trong
ứng dụng thực tế bạn nên thêm xử lý lỗi ở đây.

Với những thay đổi này, hãy chạy code và thực hiện một yêu cầu. Chúng ta
không còn in dữ liệu ra terminal, nên sẽ không thấy bất kỳ output nào ngoài
output từ Cargo. Khi bạn load *127.0.0.1:7878* trong trình duyệt web, bạn
sẽ nhận được một trang trắng thay vì một lỗi. Bạn vừa tự tay viết code để
nhận một yêu cầu HTTP và gửi một phản hồi!

### Trả về HTML thực

Hãy triển khai chức năng để trả về nhiều hơn một trang trắng. Tạo file mới
*hello.html* ở thư mục gốc của dự án, không phải trong thư mục *src*. Bạn
có thể nhập bất kỳ HTML nào bạn muốn; Listing 20-4 cho thấy một ví dụ
khả thi.

<span class="filename">Filename: hello.html</span>

```html
{{#include ../listings/ch20-web-server/listing-20-05/hello.html}}
```

<span class="caption">Listing 20-4: Một file HTML mẫu để trả về trong phản hồi</span>

Đây là một tài liệu HTML5 tối giản với một tiêu đề và một vài đoạn văn bản.
Để trả về file này từ server khi nhận được một yêu cầu, chúng ta sẽ chỉnh
sửa `handle_connection` như trong Listing 20-5 để đọc file HTML, thêm nó vào
phản hồi dưới dạng phần thân, và gửi đi.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-05/src/main.rs:here}}
```

<span class="caption">Listing 20-5: Gửi nội dung của *hello.html* như phần
thân của phản hồi</span>

Chúng ta đã thêm `fs` vào câu lệnh `use` để đưa module filesystem của thư viện
chuẩn vào phạm vi sử dụng. Code để đọc nội dung của một file thành một
chuỗi có thể sẽ quen thuộc; chúng ta đã dùng nó ở Chương 12 khi đọc
nội dung file cho dự án I/O trong Listing 12-4.

Tiếp theo, chúng ta sử dụng `format!` để thêm nội dung file làm phần thân của
phản hồi thành công. Để đảm bảo một phản hồi HTTP hợp lệ, chúng ta thêm
header `Content-Length`, được đặt bằng kích thước của phần thân phản hồi, trong
trường hợp này là kích thước của `hello.html`.

Chạy code này với `cargo run` và load *127.0.0.1:7878* trong trình duyệt;
bạn sẽ thấy HTML của bạn được hiển thị!

Hiện tại, chúng ta đang bỏ qua dữ liệu yêu cầu trong `http_request` và chỉ
gửi lại nội dung của file HTML một cách vô điều kiện. Điều này có nghĩa là
nếu bạn thử yêu cầu *127.0.0.1:7878/something-else* trong trình duyệt, bạn
vẫn sẽ nhận được cùng phản hồi HTML này. Hiện tại, server của chúng ta rất
hạn chế và không thực hiện những gì hầu hết các web server làm. Chúng ta
muốn tùy chỉnh phản hồi dựa trên yêu cầu và chỉ gửi file HTML cho một yêu
cầu hợp lệ tới */*.

### Xác thực yêu cầu và phản hồi có chọn lọc

Hiện tại, web server của chúng ta sẽ trả về HTML trong file bất kể client
yêu cầu gì. Hãy thêm chức năng kiểm tra rằng trình duyệt đang yêu cầu */*
trước khi trả về file HTML và trả lỗi nếu trình duyệt yêu cầu bất cứ thứ gì
khác. Để làm điều này, chúng ta cần chỉnh sửa `handle_connection`, như
trong Listing 20-6. Code mới này kiểm tra nội dung yêu cầu nhận được với
cái mà chúng ta biết là một yêu cầu tới */*, và thêm các khối `if` và `else`
để xử lý các yêu cầu khác nhau.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-06/src/main.rs:here}}
```

<span class="caption">Listing 20-6: Xử lý các yêu cầu tới */* khác với các
yêu cầu khác</span>

Chúng ta chỉ xem dòng đầu tiên của yêu cầu HTTP, vì vậy thay vì đọc toàn bộ
yêu cầu vào một vector, chúng ta gọi `next` để lấy phần tử đầu tiên từ iterator.
`unwrap` đầu tiên xử lý `Option` và dừng chương trình nếu iterator không có
phần tử nào. `unwrap` thứ hai xử lý `Result` và có cùng hiệu ứng như `unwrap`
trong `map` được thêm ở Listing 20-2.

Tiếp theo, chúng ta kiểm tra `request_line` xem nó có bằng dòng yêu cầu của
một GET request tới đường dẫn */* hay không. Nếu có, khối `if` trả về
nội dung file HTML của chúng ta.

Nếu `request_line` *không* bằng GET request tới đường dẫn */*, điều đó
có nghĩa là chúng ta nhận được một yêu cầu khác. Chúng ta sẽ thêm code vào
khối `else` trong một lúc để phản hồi tất cả các yêu cầu khác.

Chạy code này ngay và yêu cầu *127.0.0.1:7878*; bạn sẽ nhận được HTML trong
*hello.html*. Nếu bạn thực hiện bất kỳ yêu cầu nào khác, chẳng hạn như
*127.0.0.1:7878/something-else*, bạn sẽ nhận được lỗi kết nối giống như khi
chạy code ở Listing 20-1 và Listing 20-2.

Bây giờ hãy thêm code trong Listing 20-7 vào khối `else` để trả về phản hồi
với mã trạng thái 404, báo hiệu rằng nội dung cho yêu cầu không tìm thấy.
Chúng ta cũng sẽ trả về một số HTML cho một trang hiển thị trong trình duyệt
để báo phản hồi tới người dùng cuối.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-07/src/main.rs:here}}
```

<span class="caption">Listing 20-7: Phản hồi với mã trạng thái 404 và trang
lỗi nếu có yêu cầu khác ngoài */*</span>

Ở đây, phản hồi của chúng ta có một dòng trạng thái với mã 404 và cụm từ lý
do `NOT FOUND`. Phần thân của phản hồi sẽ là HTML trong file *404.html*. Bạn
sẽ cần tạo một file *404.html* bên cạnh *hello.html* cho trang lỗi; một lần
nữa, bạn có thể sử dụng bất kỳ HTML nào bạn muốn hoặc dùng ví dụ HTML trong
Listing 20-8.

<span class="filename">Filename: 404.html</span>

```html
{{#include ../listings/ch20-web-server/listing-20-07/404.html}}
```

<span class="caption">Listing 20-8: Nội dung mẫu cho trang trả về với
bất kỳ phản hồi 404 nào</span>

Với những thay đổi này, hãy chạy lại server của bạn. Yêu cầu *127.0.0.1:7878*
sẽ trả về nội dung của *hello.html*, và bất kỳ yêu cầu nào khác, như
*127.0.0.1:7878/foo*, sẽ trả về HTML lỗi từ *404.html*.

### Một chút refactoring

Hiện tại, các khối `if` và `else` có nhiều đoạn lặp lại: cả hai đều đọc file
và ghi nội dung file vào stream. Sự khác biệt duy nhất là dòng trạng thái
và tên file. Hãy làm code ngắn gọn hơn bằng cách tách những khác biệt đó
ra các dòng `if` và `else` riêng, gán giá trị dòng trạng thái và tên file
vào các biến; sau đó chúng ta có thể sử dụng các biến này một cách
vô điều kiện trong code để đọc file và ghi phản hồi. Listing 20-9 cho thấy
code kết quả sau khi thay thế các khối `if` và `else` lớn.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-09/src/main.rs:here}}
```

<span class="caption">Listing 20-9: Refactoring các khối `if` và `else` chỉ
chứa code khác nhau giữa hai trường hợp</span>

Bây giờ các khối `if` và `else` chỉ trả về các giá trị thích hợp cho dòng
trạng thái và tên file dưới dạng một tuple; sau đó chúng ta dùng
destructuring để gán hai giá trị này vào `status_line` và `filename`
sử dụng pattern trong câu lệnh `let`, như đã thảo luận trong Chương 18.

Code bị lặp trước đó giờ nằm ngoài các khối `if` và `else` và sử dụng
các biến `status_line` và `filename`. Điều này giúp dễ nhìn thấy sự khác
biệt giữa hai trường hợp, và có nghĩa là chúng ta chỉ có một chỗ để
cập nhật code nếu muốn thay đổi cách đọc file và ghi phản hồi. Hành vi
của code trong Listing 20-9 sẽ giống với Listing 20-8.

Tuyệt vời! Bây giờ chúng ta có một web server đơn giản với khoảng 40 dòng
code Rust, phản hồi một yêu cầu với một trang nội dung và phản hồi tất cả
các yêu cầu khác với phản hồi 404.

Hiện tại, server của chúng ta chạy trong một thread duy nhất, nghĩa là
chỉ có thể phục vụ một yêu cầu tại một thời điểm. Hãy xem điều này có thể
gây ra vấn đề như thế nào bằng cách mô phỏng một số yêu cầu chậm. Sau đó
chúng ta sẽ sửa để server có thể xử lý nhiều yêu cầu cùng lúc.
