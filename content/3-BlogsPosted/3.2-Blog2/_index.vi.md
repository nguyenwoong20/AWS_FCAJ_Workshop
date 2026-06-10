---
title: "Blog 2: Mô hình hóa dữ liệu DynamoDB cho app xem phim"
date: 2026-06-10
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

{{% notice note %}}
✏️ Bản nháp — đăng lên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) rồi dán link bài đăng vào đây.
{{% /notice %}}

**Đã đăng tại:** [LINK BÀI ĐĂNG FACEBOOK]

# "WHERE của tôi đâu rồi?" — Mô hình hóa dữ liệu DynamoDB cho app xem phim

Từ MongoDB chuyển qua, câu hỏi DynamoDB đầu tiên của tôi là: *làm sao query phim theo thể loại khi key là id phim?* Đây là những gì tôi học được khi thiết kế bảng cho app Cinemax.

**Quy tắc 1: thiết kế theo access pattern, không phải theo dữ liệu.** Tôi liệt kê những gì app thật sự làm: lấy phim theo id, duyệt phim mới nhất theo thể loại, tìm theo tên, liệt kê bookmark của người dùng. Mỗi pattern quyết định một key hoặc một index.

**Quy tắc 2: GSI là "mệnh đề WHERE thứ hai" của bạn.** Bảng chính dùng `id` làm partition key (lấy theo id là O(1)). Để "duyệt phim Action, mới nhất trước", tôi thêm Global Secondary Index: partition key `category`, sort key `createdAt`. Một lệnh `Query`, đã sắp xếp sẵn, không cần scan.

**Quy tắc 3: khóa tổ hợp mô hình hóa quan hệ.** Bookmark không cần id riêng. Bảng chỉ gồm partition key `userId` + sort key `movieId`. "Mọi bookmark của user X" là một Query; "X có bookmark phim Y không" là một GetItem; thêm trùng bookmark thì tự ghi đè chính nó. Ba hành vi miễn phí chỉ nhờ thiết kế khóa.

**Quy tắc 4: chấp nhận Scan là ngoại lệ.** Tìm kiếm theo tên dùng Scan với filter — ổn ở quy mô danh mục của tôi, sai ở quy mô một triệu item. Giải pháp đúng khi scale là OpenSearch hoặc search index chuyên dụng, và việc biết *thiết kế của mình ngừng scale ở đâu* cũng là một phần của thiết kế.

DynamoDB trừng phạt bạn nếu bỏ qua bước mô hình hóa, và thưởng hậu hĩnh khi bạn làm đúng: query mili-giây một chữ số trong free tier, không cần server nào.

#AWS #DynamoDB #NoSQL #DataModeling #FirstCloudJourney
