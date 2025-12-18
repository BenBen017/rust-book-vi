# Các Đặc Điểm Lập Trình Hướng Đối Tượng của Rust

Lập trình hướng đối tượng (Object-Oriented Programming – OOP) là một cách để mô
hình hóa chương trình. Object như một khái niệm trong lập trình được giới thiệu
lần đầu trong ngôn ngữ lập trình Simula vào những năm 1960. Những object này đã
ảnh hưởng tới kiến trúc lập trình của Alan Kay, trong đó các object gửi thông
điệp cho nhau. Để mô tả kiến trúc này, ông đã đặt ra thuật ngữ *object-oriented
programming* vào năm 1967. Có rất nhiều định nghĩa khác nhau, thậm chí cạnh
tranh nhau, về việc OOP là gì; theo một số định nghĩa thì Rust là hướng đối
tượng, nhưng theo những định nghĩa khác thì Rust lại không phải.

Trong chương này, chúng ta sẽ khám phá một số đặc điểm thường được xem là thuộc
về lập trình hướng đối tượng, và cách những đặc điểm đó được thể hiện trong
Rust theo phong cách idiomatic. Sau đó, chúng ta sẽ chỉ ra cách triển khai một
design pattern hướng đối tượng trong Rust và thảo luận về những đánh đổi khi
làm như vậy so với việc triển khai một lời giải tận dụng các thế mạnh vốn có
của Rust.
