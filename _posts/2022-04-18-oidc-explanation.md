---
layout: post
id: 2022-04-18-oidc-explanation
title: OpenID Connect là gì?
date: 2022-04-15 00:20:00
authors: [ngo275]
categories: []
tags: [OpenID Connect, Authentication]
comments: true
cover_photo: /img/introduction/team.png
excerpt: "Nếu bạn đọc phần này, bạn sẽ có thể hiểu OpenID Connect ngay cả khi bạn không phải là nhà phát triển!"
---

## Lai lịch

Làm thế nào để thực hiện chức năng xác thực? Làm thế nào để thực hiện chức năng đăng nhập? Các nhà phát triển ứng dụng và dịch vụ web hầu như luôn phải đối mặt với những thách thức này khi họ bắt đầu phát triển các dịch vụ mới. Ngay cả khi bạn thực hiện đăng nhập bằng địa chỉ email và mật khẩu của riêng mình, có thể rất thảm nếu bạn không quản lý mật khẩu một cách chính xác và các yêu cầu đăng nhập xã hội (đăng nhập Facebook, v.v.) có thể được thêm vào sau đó. Các chức năng xung quanh xác thực rất quan trọng vì chúng có tác động đáng kể đến cả bảo mật và UX. Đúng là số lượng các công cụ như Xác thực Firebase để dễ dàng triển khai một tập hợp các chức năng xung quanh xác thực ngày càng tăng, và chi phí phát triển cũng đang giảm. Tuy nhiên, ngay cả khi bạn sử dụng OAuth2 hoặc OpenID Connect thông qua các công cụ như vậy, vẫn có nhiều trường hợp bạn hiểu không đúng về OAuth2 hoặc OpenID Connect. Trong bài viết này, tôi sẽ giải thích cơ chế của OAuth2 và OpenID Connect, được sử dụng nhiều nhất để xác thực, cho những ai muốn hiểu rõ hơn về cơ chế xác thực.

## OAuth2 ở vị trí đầu tiên là gì?

Trước khi nói về OpenID Connect, chúng ta hãy xem xét OAuth2. Khi số lượng các dịch vụ như Twitter và Facebook nơi người dùng tự thêm thông tin đã tăng lên, việc chia sẻ ID và liên kết các phiên giữa các dịch vụ khác nhau trở nên cần thiết. Ví dụ: dịch vụ của bạn muốn xem thông tin bạn bè trên Facebook của bạn. Trong trường hợp này, người dùng không nên chuyển chính thông tin đăng nhập Facebook (địa chỉ email và mật khẩu, v.v.) vào dịch vụ mà chỉ chia sẻ quyền truy cập các thông tin cần thiết trên Facebook. Theo cách này, OAuth là một cơ chế (giao thức) chỉ chia sẻ thành công các quyền mà không cần chuyển thông tin đăng nhập. OAuth được xây dựng vào năm 2007, nhưng một lỗ hổng đã được phát hiện vào năm 2009 và nó đã phát triển thành OAuth 2. OAuth1 và OAuth2 không tương thích, và bây giờ OAuth2 về cơ bản đã được sử dụng. Về mặt kỹ thuật, việc chuyển quyền theo cách này được gọi là "ủy quyền". OAuth2 là một giao thức để ủy quyền an toàn.

Bây giờ chúng ta hãy xem xét luồng của OAuth2.

Công ty TNHH ABC muốn tham khảo các tài liệu của Alice do Công ty TNHH XYZ lưu giữ.

1. Công ty TNHH ABC hỏi khách hàng của mình, Alice, nếu họ có tài khoản với XYZ Co., Ltd.
2. Alice trả lời "CÓ".
3. Công ty TNHH ABC hỏi Alice, "Nếu vậy, tôi muốn tài liệu của bạn được lưu trữ tại Công ty TNHH XYZ, nhưng bạn có thể cho tôi chìa khóa nơi lưu trữ tài liệu không?"
4. Alice đến Công ty TNHH XYZ để lấy chìa khóa.
5. Tại cửa sổ của Công ty TNHH XYZ, Alice xác nhận danh tính của mình và xin vào Công ty TNHH ABC để làm chìa khóa lưu trữ tài liệu.
6. Vào một ngày sau đó, cửa sổ của Công ty TNHH XYZ sẽ gửi một bức thư đến Công ty TNHH ABC với nội dung “Vui lòng đến nhận chìa khóa vì nó đã hoàn thành”.
7. Công ty TNHH ABC Mang bức thư đến cửa sổ của Công ty TNHH XYZ, nói với họ rằng bạn là nhân viên của Công ty TNHH ABC và nhận chìa khóa của Alice để đổi lấy bức thư. Nói một cách chính xác, đó không phải là một chiếc chìa khóa đơn lẻ, mà là hình ảnh của một chiếc móc chìa khóa với chìa khóa là “nơi lưu trữ tài liệu của Alice”.
8. Một nhân viên của Công ty TNHH ABC lấy chìa khóa đến vị trí lưu trữ tài liệu của Công ty TNHH XYZ và sử dụng chìa khóa để lấy các tài liệu liên quan đến Alice từ vị trí lưu trữ.
9. Nhân viên Công ty TNHH ABC in hồ sơ tại chỗ và về nhà. Công ty TNHH ABC đã lấy được thành công tài liệu của Alice từ Công ty TNHH XYZ.

Tại thời điểm này, bạn có thể tự hỏi, "Alice nên đăng ký một bản sao của tài liệu mà không cần làm chìa khóa ngay từ đầu và cung cấp cho Công ty TNHH ABC một bản sao của tài liệu đó thay vì chìa khóa." chính xác nơi mà OpenID Connect xuất hiện. Điều này sẽ được mô tả sau.

Bằng cách cấp khóa, Alice có thể dễ dàng tham khảo các tài liệu mới nhất mà không cần thực hiện bất kỳ thao tác đặc biệt nào ngay cả khi tài liệu đã được cập nhật (nhân viên chỉ phải đến XYZ Co., Ltd. một lần nữa). Ngoài ra, nếu Công ty TNHH ABC muốn cập nhật tài liệu, bạn có thể cập nhật tài liệu gốc trong Công ty TNHH XYZ bằng cách có khóa. Nói một cách chính xác, chiếc chìa khóa này giống như một chiếc móc chìa khóa và chứa một chiếc chìa khóa để đặt vào nơi được chỉ định bởi Alice. Nếu Alice giữ các tài liệu ở năm vị trí riêng biệt, thì các chìa khóa của năm vị trí lưu trữ sẽ bị dính vào nhau.

Hãy kiểm tra các ký tự và thuật ngữ thực tế xuất hiện ở đây.

| Alice | Resource owner |
| Công ty TNHH ABC | Client |
| Tại cửa sổ của Công ty TNHH XYZ | Authorization server |
| Địa điểm lưu trữ tài liệu của Công ty TNHH XYZ | Resource server |
| Một lá thư được gửi qua cửa sổ XYZ của công ty | Authorization code |
| Thông tin xác minh danh tính được sử dụng khi chứng minh rằng nhân viên của Công ty TNHH ABC thực sự là nhân viên của ABC tại cửa sổ của Công ty TNHH XYZ. | Client secret |
| Bộ sưu tập chìa khóa mà Công ty TNHH ABC nhận được (móc chìa khóa) | Access token |
| Các khóa riêng lẻ trong chuỗi khóa | Scope |
| Tài liệu | Resource |
| Quy trình trong đó bước 6 được thực hiện rõ ràng và khóa chỉ được chuyển trực tiếp | Authorization code flow |
| Quy trình bỏ qua bước 6 và gửi khóa đột ngột qua thư | Implicit flow |
| Thư từ XYZ Co., Ltd. | Redirect URL |


## Lạm dụng OAuth2

OAuth2 là một giao thức rất hữu ích, nhưng hành động bình tĩnh chuyển khóa cũng rất nguy hiểm. Nếu Công ty TNHH ABC là một tổ chức độc hại và yêu cầu Alice truy cập vào tài khoản ngân hàng của Alice và Alice đăng ký chìa khóa mà không cần suy nghĩ, tiền từ tài khoản ngân hàng của Alice thậm chí có khả năng bị lấy đi. Hơn nữa, có nhiều trường hợp bạn chỉ muốn thông tin của Alice thay vì cần khóa để sử dụng tài nguyên của Alice và OAuth2 là một giao thức ủy quyền (ví dụ: quản lý thẩm quyền của Alice), nhưng nó xác thực. (Ví dụ: xác minh danh tính của Alice) cũng được sử dụng rất thường xuyên.

## OpenID Connect là gì?

Để đối phó với tình huống này, OpenID Connect (sau đây gọi là OIDC) là một phần mở rộng của OAuth2 kết hợp xác thực làm tiêu chuẩn.

```
3. Công ty TNHH ABC hỏi Alice, "Nếu vậy, tôi muốn tài liệu của bạn được lưu trữ tại Công ty TNHH XYZ, nhưng bạn có thể cho tôi chìa khóa nơi lưu trữ tài liệu không?"
```

Trong OpenID Connect, ở bước 3 của OAuth2 trước đó, thay vì khóa, "bản sao tài liệu" đã được thêm vào các tùy chọn. Nếu bạn chỉ định "bản sao của tài liệu" ở đây,

```
6. Vào một ngày sau đó, cửa sổ của Công ty TNHH XYZ sẽ gửi một bức thư đến Công ty TNHH ABC với nội dung "Vui lòng đến nhận chìa khóa vì đã hoàn thành."
```

Trong bước 6, bạn sẽ nhận được một bản sao của tài liệu. Nếu Công ty TNHH ABC chỉ muốn thông tin có thể xác nhận danh tính của Alice, thì khi bạn có bản sao của tài liệu này, bạn không phải thực hiện các bước còn lại. Bạn không cần phải phát hành chìa khóa. Tất nhiên, bản sao văn bản này có chữ ký của Công ty TNHH XYZ để nhà xuất bản xác nhận. Bản sao của tài liệu này được gọi là "Mã thông báo ID" và OIDC đã xác thực tiêu chuẩn hóa bằng cách đưa Mã thông báo ID vào OAuth2 theo cách này. Tất nhiên, bạn có thể tạo khóa cùng lúc khi sao chép tài liệu này và quy trình tạo khóa giống như OAuth2.

Yêu cầu từ Công ty TNHH ABC ở bước 3 được gọi là yêu cầu ủy quyền và yêu cầu thực sự chuyển trực tiếp đến máy chủ Ủy quyền (cửa sổ của Công ty TNHH XYZ) thay vì Alice. Khách hàng (ABC Co., Ltd.) thực hiện các cài đặt khác nhau trong yêu cầu ủy quyền. Cho dù đó là Luồng ngầm hay luồng mã Ủy quyền, lựa chọn phạm vi (khóa riêng lẻ được gắn với chuỗi khóa) bạn muốn sử dụng, phương pháp nào được sử dụng khi máy chủ Ủy quyền thực sự gửi thông tin đến Máy khách (phương pháp gửi thư cụ thể), Hãy cài đặt chi tiết chẳng hạn như.

## Tóm lược

Theo cách này, OIDC dựa trên OAuth2, vốn ban đầu đã tiêu chuẩn hóa việc tạo mã thông báo truy cập và chuyển giao quyền thông qua mã thông báo truy cập, đồng thời cũng có thể xử lý Mã thông báo ID. Nhiều tổ chức sử dụng OIDC an toàn và đơn giản hơn các giao thức xác thực và ủy quyền khác, và tính đến năm 2022, nó là giao thức được sử dụng nhiều nhất để xác thực và ủy quyền. Trong phần tiếp theo, chúng ta sẽ xem xét việc triển khai chức năng đăng nhập bằng OIDC.
