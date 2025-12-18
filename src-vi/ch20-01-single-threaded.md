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

The first line is a *status line* that contains the HTTP version used in the
response, a numeric status code that summarizes the result of the request, and
a reason phrase that provides a text description of the status code. After the
CRLF sequence are any headers, another CRLF sequence, and the body of the
response.

Here is an example response that uses HTTP version 1.1, has a status code of
200, an OK reason phrase, no headers, and no body:

```text
HTTP/1.1 200 OK\r\n\r\n
```

The status code 200 is the standard success response. The text is a tiny
successful HTTP response. Let’s write this to the stream as our response to a
successful request! From the `handle_connection` function, remove the
`println!` that was printing the request data and replace it with the code in
Listing 20-3.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-03/src/main.rs:here}}
```

<span class="caption">Listing 20-3: Writing a tiny successful HTTP response to
the stream</span>

The first new line defines the `response` variable that holds the success
message’s data. Then we call `as_bytes` on our `response` to convert the string
data to bytes. The `write_all` method on `stream` takes a `&[u8]` and sends
those bytes directly down the connection. Because the `write_all` operation
could fail, we use `unwrap` on any error result as before. Again, in a real
application you would add error handling here.

With these changes, let’s run our code and make a request. We’re no longer
printing any data to the terminal, so we won’t see any output other than the
output from Cargo. When you load *127.0.0.1:7878* in a web browser, you should
get a blank page instead of an error. You’ve just hand-coded receiving an HTTP
request and sending a response!

### Returning Real HTML

Let’s implement the functionality for returning more than a blank page. Create
the new file *hello.html* in the root of your project directory, not in the
*src* directory. You can input any HTML you want; Listing 20-4 shows one
possibility.

<span class="filename">Filename: hello.html</span>

```html
{{#include ../listings/ch20-web-server/listing-20-05/hello.html}}
```

<span class="caption">Listing 20-4: A sample HTML file to return in a
response</span>

This is a minimal HTML5 document with a heading and some text. To return this
from the server when a request is received, we’ll modify `handle_connection` as
shown in Listing 20-5 to read the HTML file, add it to the response as a body,
and send it.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-05/src/main.rs:here}}
```

<span class="caption">Listing 20-5: Sending the contents of *hello.html* as the
body of the response</span>

We’ve added `fs` to the `use` statement to bring the standard library’s
filesystem module into scope. The code for reading the contents of a file to a
string should look familiar; we used it in Chapter 12 when we read the contents
of a file for our I/O project in Listing 12-4.

Next, we use `format!` to add the file’s contents as the body of the success
response. To ensure a valid HTTP response, we add the `Content-Length` header
which is set to the size of our response body, in this case the size of
`hello.html`.

Run this code with `cargo run` and load *127.0.0.1:7878* in your browser; you
should see your HTML rendered!

Currently, we’re ignoring the request data in `http_request` and just sending
back the contents of the HTML file unconditionally. That means if you try
requesting *127.0.0.1:7878/something-else* in your browser, you’ll still get
back this same HTML response. At the moment, our server is very limited and
does not do what most web servers do. We want to customize our responses
depending on the request and only send back the HTML file for a well-formed
request to */*.

### Validating the Request and Selectively Responding

Right now, our web server will return the HTML in the file no matter what the
client requested. Let’s add functionality to check that the browser is
requesting */* before returning the HTML file and return an error if the
browser requests anything else. For this we need to modify `handle_connection`,
as shown in Listing 20-6. This new code checks the content of the request
received against what we know a request for */* looks like and adds `if` and
`else` blocks to treat requests differently.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-06/src/main.rs:here}}
```

<span class="caption">Listing 20-6: Handling requests to */* differently from
other requests</span>

We’re only going to be looking at the first line of the HTTP request, so rather
than reading the entire request into a vector, we’re calling `next` to get the
first item from the iterator. The first `unwrap` takes care of the `Option` and
stops the program if the iterator has no items. The second `unwrap` handles the
`Result` and has the same effect as the `unwrap` that was in the `map` added in
Listing 20-2.

Next, we check the `request_line` to see if it equals the request line of a GET
request to the */* path. If it does, the `if` block returns the contents of our
HTML file.

If the `request_line` does *not* equal the GET request to the */* path, it
means we’ve received some other request. We’ll add code to the `else` block in
a moment to respond to all other requests.

Run this code now and request *127.0.0.1:7878*; you should get the HTML in
*hello.html*. If you make any other request, such as
*127.0.0.1:7878/something-else*, you’ll get a connection error like those you
saw when running the code in Listing 20-1 and Listing 20-2.

Now let’s add the code in Listing 20-7 to the `else` block to return a response
with the status code 404, which signals that the content for the request was
not found. We’ll also return some HTML for a page to render in the browser
indicating the response to the end user.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-07/src/main.rs:here}}
```

<span class="caption">Listing 20-7: Responding with status code 404 and an
error page if anything other than */* was requested</span>

Here, our response has a status line with status code 404 and the reason phrase
`NOT FOUND`. The body of the response will be the HTML in the file *404.html*.
You’ll need to create a *404.html* file next to *hello.html* for the error
page; again feel free to use any HTML you want or use the example HTML in
Listing 20-8.

<span class="filename">Filename: 404.html</span>

```html
{{#include ../listings/ch20-web-server/listing-20-07/404.html}}
```

<span class="caption">Listing 20-8: Sample content for the page to send back
with any 404 response</span>

With these changes, run your server again. Requesting *127.0.0.1:7878* should
return the contents of *hello.html*, and any other request, like
*127.0.0.1:7878/foo*, should return the error HTML from *404.html*.

### A Touch of Refactoring

At the moment the `if` and `else` blocks have a lot of repetition: they’re both
reading files and writing the contents of the files to the stream. The only
differences are the status line and the filename. Let’s make the code more
concise by pulling out those differences into separate `if` and `else` lines
that will assign the values of the status line and the filename to variables;
we can then use those variables unconditionally in the code to read the file
and write the response. Listing 20-9 shows the resulting code after replacing
the large `if` and `else` blocks.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
{{#rustdoc_include ../listings/ch20-web-server/listing-20-09/src/main.rs:here}}
```

<span class="caption">Listing 20-9: Refactoring the `if` and `else` blocks to
contain only the code that differs between the two cases</span>

Now the `if` and `else` blocks only return the appropriate values for the
status line and filename in a tuple; we then use destructuring to assign these
two values to `status_line` and `filename` using a pattern in the `let`
statement, as discussed in Chapter 18.

The previously duplicated code is now outside the `if` and `else` blocks and
uses the `status_line` and `filename` variables. This makes it easier to see
the difference between the two cases, and it means we have only one place to
update the code if we want to change how the file reading and response writing
work. The behavior of the code in Listing 20-9 will be the same as that in
Listing 20-8.

Awesome! We now have a simple web server in approximately 40 lines of Rust code
that responds to one request with a page of content and responds to all other
requests with a 404 response.

Currently, our server runs in a single thread, meaning it can only serve one
request at a time. Let’s examine how that can be a problem by simulating some
slow requests. Then we’ll fix it so our server can handle multiple requests at
once.
