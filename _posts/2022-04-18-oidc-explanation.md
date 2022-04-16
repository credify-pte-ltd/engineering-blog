---
layout: post
id: 2022-04-18-oidc-explanation
title: OpenID Connect là gì?
date: 2022-04-15 00:20:00
authors: [ngo275]
categories: []
tags: [OpenID Connect, Authentication]
comments: true
cover_photo: /img/oidc-explanation/oidc.png
excerpt: "Nếu bạn đọc phần này, bạn sẽ có thể hiểu OpenID Connect ngay cả khi bạn không phải là nhà phát triển!"
---

## Lai lịch

Làm thế nào để thực hiện chức năng xác thực? Các nhà phát triển ứng dụng và dịch vụ web hầu như luôn phải đối mặt với những thách thức này khi họ bắt đầu phát triển các dịch vụ mới. Ngay cả khi bạn triển khai đăng nhập bằng địa chỉ email / mật khẩu của riêng mình, có thể là một thảm họa nếu bạn không quản lý mật khẩu một cách chính xác và các yêu cầu về đăng nhập mạng xã hội (ví dụ: đăng nhập Facebook) có thể được thêm vào sau đó. Chức năng xung quanh xác thực rất quan trọng vì nó ảnh hưởng đáng kể đến cả bảo mật và UX. Thật vậy, các công cụ như Xác thực Firebase ngày càng trở nên sẵn có để dễ dàng triển khai một bộ hoàn chỉnh các chức năng liên quan đến xác thực và chi phí phát triển ngày càng giảm. Tuy nhiên, ngay cả khi bạn đang sử dụng OAuth 2.0 và OpenID Connect thông qua các công cụ như vậy, có lẽ nhiều trường hợp bạn chưa hiểu đúng về OAuth 2.0 và OpenID Connect. Trong bài viết này, chúng tôi sẽ giải thích cơ chế của OAuth 2.0 và OpenID Connect, là những hệ thống xác thực được sử dụng rộng rãi nhất, cho những ai muốn hiểu rõ hơn về các cơ chế xung quanh xác thực.

## OAuth 2.0 ở vị trí đầu tiên là gì?

Trước khi chúng ta chạm vào OpenID Connect, trước tiên chúng ta hãy xem xét OAuth 2.0: Khi ngày càng có nhiều dịch vụ như Twitter và Facebook cho phép người dùng thêm thông tin của riêng họ, cần phải có chia sẻ ID và tích hợp phiên giữa các dịch vụ khác nhau. Ví dụ, một dịch vụ có thể muốn tham khảo thông tin của bạn bè trên Facebook. Trong trường hợp này, người dùng không nên chuyển chính thông tin đăng nhập Facebook (ví dụ: địa chỉ email và mật khẩu) vào dịch vụ đó mà chỉ nên chia sẻ quyền truy cập đối với thông tin Facebook cần thiết. OAuth được phát triển vào năm 2007, nhưng một lỗ hổng đã được tìm thấy vào năm 2009 và nó đã phát triển thành OAuth 2.0. Điều này cũng đúng với danh mục "khác". Về mặt kỹ thuật, loại ủy quyền này được gọi là "ủy quyền"; OAuth 2.0 là một giao thức để ủy quyền an toàn.

Bây giờ chúng ta sẽ xem xét luồng OAuth 2.0.

Công ty ABC muốn tham khảo các tài liệu Alice do Công ty XYZ lưu giữ.

1. Công ty ABC hỏi khách hàng Alice của mình nếu cô ấy có tài khoản với Công ty XYZ.
2. Alice trả lời "CÓ".
3. Công ty ABC hỏi Alice, "Sau đó, chúng tôi muốn tài liệu của bạn, được lưu giữ tại Công ty XYZ. Bạn có thể cho chúng tôi chìa khóa nơi lưu giữ tài liệu không?"
4. Alice đến Công ty XYZ để lấy chìa khóa.
5. Tại bàn khách hàng của Công ty XYZ, Alice nộp đơn đến Công ty ABC để xác nhận danh tính của mình và làm chìa khóa nơi lưu trữ tài liệu của cô.
6. Bàn khách hàng của Công ty XYZ sau đó sẽ gửi một bức thư đến Công ty ABC với nội dung “Chìa khóa đã sẵn sàng, hãy đến lấy.
7. Một nhân viên của Công ty ABC mang bức thư đến Công ty XYZ và nói với họ rằng anh ta là nhân viên của Công ty ABC và để đổi lấy lá thư, anh ta sẽ nhận được một chìa khóa thông tin của Alice. Nói một cách chính xác, bản thân chiếc chìa khóa không phải là một chiếc chìa khóa, mà là một chùm chìa khóa với nhiều chìa khóa dẫn đến "vị trí lưu trữ tài liệu của Alice".
8. Một nhân viên của Công ty ABC lấy chìa khóa đến vị trí lưu trữ của Công ty XYZ và sử dụng chìa khóa để lấy các tài liệu liên quan đến Alice từ vị trí lưu trữ.
9. Nhân viên của Công ty ABC in tài liệu ngay tại chỗ và rời đi. Công ty ABC lấy thành công các tài liệu của Alice từ Công ty XYZ.

<div class="post-image-section"><figure>
  <img src="/img/oidc-explanation/keychain.png" alt="Flow" style="width:60%">
  </figure>
</div>


Tại thời điểm này, bạn có thể tự hỏi, "Tại sao Alice không đăng ký bản sao của chính tài liệu mà không làm chìa khóa ngay từ đầu và đưa cho Công ty ABC một bản sao của tài liệu thay vì chìa khóa?" Đó chính xác là nơi OpenID Connect xuất hiện. Điều này được mô tả bên dưới.

Bằng cách tạo một khóa, khi một tài liệu được cập nhật, Alice không làm gì đặc biệt và Công ty ABC có thể dễ dàng tham khảo tài liệu mới nhất (nhân viên chỉ cần đến Công ty XYZ với khóa lại). Ngoài ra, nếu Công ty ABC muốn cập nhật tài liệu của Alice, khóa này cho phép họ cập nhật tài liệu gốc tại Công ty XYZ. Chìa khóa này hoàn toàn giống như một vòng chìa khóa, chứa các chìa khóa cho phép truy cập vào các vị trí bị hạn chế. Nếu Alice giữ tài liệu của mình ở năm vị trí khác nhau, thì mỗi vòng chìa khóa sẽ chứa năm chìa khóa dẫn đến năm vị trí lưu trữ khác nhau.

Chúng tôi sẽ xem xét các ký tự và điều khoản thực tế xuất hiện ở đây.

| Trong ví dụ | Thuật ngữ OIDC |
|:---|:---:|
| Alice | Resource owner |
| Công ty ABC | Client |
| Bàn khách hàng của Công ty XYZ | Authorization server |
| Lưu trữ tài liệu của Công ty XYZ | Resource server |
| Thư gửi qua bàn khách hàng của Công ty XYZ | Authorization code |
| Thông tin nhận dạng được bàn khách hàng của Công ty XYZ sử dụng để xác minh rằng một nhân viên của Công ty ABC thực sự là nhân viên của Công ty ABC. | Client secret |
| Bộ sưu tập chìa khóa (móc khóa) mà Công ty ABC nhận làm. | Access token |
| Các khóa riêng lẻ trong chuỗi khóa | Scope |
| Tài liệu | Resource |
| Luồng trong đó bước 6 được thực hiện một cách rõ ràng và các khóa chỉ được trao tận tay. | Authorization code flow |
| Luồng để bỏ qua bước 6 và đột ngột gửi khóa qua thư. | Implicit flow |
| Thư từ Công ty XYZ. | Redirect URL |


## Lạm dụng OAuth 2.0

OAuth 2.0 là một giao thức rất hữu ích, nhưng hành động bình tĩnh bàn giao khóa cũng rất nguy hiểm. Nếu Công ty ABC là một tổ chức độc hại và yêu cầu Alice truy cập vào tài khoản ngân hàng của cô ấy và Alice đăng ký khóa mà không cần suy nghĩ, thậm chí có khả năng tiền có thể bị lấy từ tài khoản ngân hàng của Alice. Hơn nữa, có nhiều trường hợp khóa không cần thiết để sử dụng tài nguyên của Alice, mà thông tin của Alice chỉ là mong muốn và mặc dù OAuth 2.0 là giao thức ủy quyền (ví dụ: quản lý quyền của Alice), nó cũng được sử dụng sai để xác thực (ví dụ: Xác minh danh tính của Alice) trong nhiều trường hợp.

## OpenID Connect là gì?

Để đối phó với tình huống này, OpenID Connect (sau đây gọi là OIDC) là một phần mở rộng của OAuth 2.0 kết hợp xác thực làm tiêu chuẩn.

```
3. Công ty ABC hỏi Alice, "Sau đó, chúng tôi muốn tài liệu của bạn, được lưu giữ tại Công ty XYZ. Bạn có thể cho chúng tôi chìa khóa nơi lưu giữ tài liệu không?"
```

OpenID Connect cũng đã thêm "bản sao tài liệu" như một tùy chọn trong bước 3 của OAuth 2.0, thay vì các khóa, như đã đề cập trước đó. Ở đây, chỉ định "bản sao của tài liệu" sẽ là,

```
6. Bàn khách hàng của Công ty XYZ sau đó sẽ gửi một bức thư đến Công ty ABC với nội dung “Chìa khóa đã sẵn sàng, hãy đến lấy.
```

Trong bước 6 này, bạn sẽ nhận được một bản sao của tài liệu. Nếu tất cả những gì Công ty ABC muốn là thông tin để xác minh danh tính của Alice, thì khi họ đã có bản sao của tài liệu này, họ không cần thực hiện các bước còn lại. Họ thậm chí sẽ không phải bận tâm phát hành một chìa khóa. Tất nhiên, bản sao của tài liệu này được ký bởi Công ty XYZ để người phát hành có thể được xác nhận. Bản sao của tài liệu này được gọi là "Mã thông báo ID" và OIDC đã tiêu chuẩn hóa quy trình xác thực bằng cách đưa Mã thông báo ID vào OAuth 2.0. Tất nhiên, bản sao của tài liệu này và quá trình tạo khóa có thể được thực hiện cùng một lúc và quy trình tạo khóa giống như trong OAuth 2.0.

Yêu cầu từ Công ty ABC trong Bước 3 được gọi là "Authorization request", thực tế chuyển trực tiếp đến "Authorization server" (bàn khách hàng của Công ty XYZ), không phải cho Alice. Trong yêu cầu ủy quyền, "Client" (Công ty ABC) định cấu hình các cài đặt khác nhau, chẳng hạn như sử dụng "Implicit flow" hay "Authorization code flow", lựa chọn "scope" (các khóa được gắn với người giữ khóa trong ví dụ trên ) được sử dụng và cách thông tin được gửi đến khách hàng (phương pháp gửi thư cụ thể), v.v.

## Tóm lược


OIDC dựa trên OAuth 2.0, ban đầu đã tiêu chuẩn hóa việc tạo mã thông báo truy cập và chuyển giao quyền thông qua mã thông báo truy cập và hiện cũng có thể xử lý ID Token. Đây là giao thức được sử dụng rộng rãi nhất để xác thực và ủy quyền vào năm 2022. Trong phần tiếp theo, chúng ta sẽ xem xét việc triển khai các chức năng đăng nhập bằng OIDC.
