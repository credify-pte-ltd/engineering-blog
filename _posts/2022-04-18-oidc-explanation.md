---
layout: post
id: 2022-04-18-oidc-explanation
title: OpenID Connect là gì?
date: 2022-06-21 00:20:00
authors: [ngo275,danh.nguyen]
categories: []
tags: [OpenID Connect, Authentication]
comments: true
cover_photo: /img/oidc-explanation/oidc.jpg
excerpt: "Nếu bạn đọc phần này, bạn sẽ có thể hiểu OpenID Connect ngay cả khi bạn không phải là nhà phát triển!"
---

## Đặt vấn đề

Làm thế nào để triển khai tính năng xác thực danh tính? Các nhà phát triển ứng dụng và dịch vụ web luôn phải đối mặt với vấn đề lớn này khi họ phát triển các dịch vụ mới. Ngay cả khi bạn cài đặt cho người dùng đăng nhập bằng địa chỉ email/mật khẩu, sẽ rắc rối về sau nếu bạn không cài đặt chúng một cách chuẩn xác và bạn cũng nên cân nhắc việc đăng nhập bằng mạng xã hội có thể sẽ được thêm vào trong tương lai. Các tính năng xoay quanh việc xác thực rất quan trọng vì nó ảnh hưởng đáng kể đến cả việc bảo mật lẫn trải nghiệm của người dùng. Thật vậy, các công cụ dễ dùng như Xác thực Firebase Authentication ngày càng hỗ trợ người sử dụng tốt hơn trong việc triển khai các chức năng liên quan đến xác thực một cách hoàn chỉnh với chi phí phát triển ngày càng thấp. Tuy nhiên, mặc dù chúng ta sử dụng OAuth 2.0 và OpenID Connect thông qua các công cụ ấy, có thể có nhiều trường hợp chúng ta chưa hiểu đúng về cách hoạt động của OAuth 2.0 và OpenID Connect. Trong bài viết này, chúng tôi sẽ giải thích cơ chế của OAuth 2.0 và OpenID Connect, là những hệ thống xác thực được sử dụng rộng rãi nhất, cho những ai muốn hiểu rõ hơn về các cơ chế xoay quanh việc xác thực.

## Đầu tiên, OAuth 2.0 là gì?

Trước khi đi vào OpenID Connect, trước tiên chúng ta hãy nói một chút về OAuth 2.0: Khi ngày càng có nhiều dịch vụ như Twitter và Facebook cho phép người dùng thêm thông tin của riêng họ, cũng như khả năng chia sẻ thông tin danh tính và yêu cầu tích hợp giữa các dịch vụ với nhau. Ví dụ, một dịch vụ có thể muốn tra cứu thông tin của bạn bè bạn trên Facebook. Trong trường hợp này, người dùng không nên chuyển thông tin đăng nhập Facebook của họ (ví dụ: địa chỉ email và mật khẩu) vào dịch vụ đó mà chỉ nên chia sẻ quyền truy cập chỉ những thông tin cần thiết trên Facebook mà thôi . OAuth được phát triển vào năm 2007, nhưng đã được tìm thấy một lỗ hổng vào năm 2009 và nó đã phát triển thành OAuth 2.0. OAuth không tương thích với OAuth 2.0 và hầu như không được sử dụng nữa. Trao quyền  truy cập dữ liệu cho bên thứ 3 được gọi là “phân quyền” và OAuth 2.0 có thể xem là một giao thức phân quyền bảo mật phổ biến hiện nay.

Bây giờ chúng ta sẽ xem xét đến luồng OAuth 2.0.

Công ty A muốn tham khảo các tài liệu của Alice do Công ty X lưu giữ.

<div class="post-image-section"><figure>
  <img src="/img/oidc-explanation/oauth2.jpg" alt="OAuth 2.0 Authorization code flow" style="width:100%">
  </figure>
</div>

1. Công ty A hỏi khách hàng Alice của mình nếu cô ấy có tài khoản với Công ty X hay không - Alice trả lời "CÓ". Công ty A hỏi Alice, "Vậy, chúng tôi muốn xem các tài liệu của bạn được lưu giữ tại Công ty X. Bạn có thể cho chúng tôi chìa khóa tủ tài liệu được không?"

2. Alice đến Công ty X để xin cấp chìa khóa cho công ty A. Tại Quầy dịch vụ khách hàng của Công ty X, Alice trình bày là muốn chia sẻ thông tin danh tính cho Công ty A và muốn đưa cho họ chìa khóa tủ tài liệu của cô.

3. Quầy dịch vụ khách hàng của Công ty X sau đó sẽ gửi một bức thư đến Công ty A với nội dung “Chìa khóa đã sẵn sàng, hãy đến lấy.

4. Một nhân viên của Công ty A mang bức thư đến Công ty X và nói với họ rằng anh ta là nhân viên của Công ty A và đưa lá thư đó để nhận thông tin  danh tính và chìa khoá tủ tài liệu của Alice. Nói một cách chính xác, bản thân chìa khóa không phải là một chiếc đơn lẻ, mà là một chùm chìa khóa với nhiều chìa khóa con dẫn đến "những tủ tài liệu khác nhau của Alice".

5. Một nhân viên của Công ty A lấy chìa khóa đến kho lưu trữ của Công ty X và sử dụng chìa khóa để lấy các tài liệu liên quan đến Alice từ tủ tài liệu. Nhân viên của Công ty A sao chép tài liệu tại chỗ và rời đi. Công ty A lấy thành công các tài liệu của Alice từ Công ty X.


Tại thời điểm này, bạn có thể tự hỏi, "Tại sao Alice lại không chia sẻ một bản sao cho chính tài liệu ngay từ lần đầu mà lại phải tạo ra một chìa khóa và sau đó có thể đưa cho Công ty A bản sao đó của tài liệu thay vì phải đưa chìa khóa?" Đó chính xác là lý do OpenID Connect xuất hiện, sẽ được mô tả bên dưới.

Bằng cách tạo một chìa khóa khi một tài liệu được cập nhật, Alice không phải làm gì đặc biệt và Công ty A có thể dễ dàng tham khảo tài liệu mới nhất (nhân viên chỉ cần đến Công ty X với chiếc chìa khoá đó). Ngoài ra, nếu Công ty A muốn cập nhật tài liệu của Alice, khóa này cho phép họ cập nhật tài liệu gốc tại Công ty X. Chìa khóa này hoàn toàn giống như một chùm chìa khóa, chứa các chìa khóa cho phép truy cập vào các tài liệu được phân quyền. Nếu Alice giữ tài liệu của mình ở năm vị trí khác nhau, thì mỗi chùm chìa khóa sẽ chứa năm chìa khóa dẫn đến năm vị trí lưu trữ khác nhau.

Chúng ta hãy tìm hiểu thêm các nhân vật trong câu chuyện trên đã đại diện cho những khái niệm nào trong bảng dưới đây.

| Trong ví dụ | Thuật ngữ OIDC |
|:---|:---:|
| Alice | Resource owner|
| Công ty A |Client |
| Quầy dịch vụ khách hàng của Công ty X | Authorization server |
| Kho lưu trữ tài liệu của Công ty X | Resource server |
| Thư gửi qua Quầy dịch vụ khách hàng của Công ty X | Mã xác thực |
| Thông tin nhận dạng được Quầy dịch vụ khách hàng của Công ty X sử dụng để xác minh rằng nhân viên đó thực sự là nhân viên của Công ty A. | Client secret |
| Chùm chìa khóa (chuỗi khóa) mà Công ty A chấp nhận tạo ra. | Access token |
| Các chìa khóa riêng lẻ trong chuỗi khóa | Scope |
| Tài liệu | Resource |
| Luồng trong đó bước 3 được thực hiện một cách rõ ràng và mã khóa chỉ được trao tận tay. | Authorization code flow |
| Luồng để bỏ qua bước 3 và gửi ngay mã khóa qua thư. | Implicit flow |
| Địa chỉ của Công ty X | Redirect URL |


## Ứng dụng OAuth 2.0

OAuth 2.0 là một giao thức rất hữu ích, tuy nhiên việc bàn giao chìa khóa cho bên thứ ba nhiều khi sẽ nguy hiểm. Nếu Công ty A là một tổ chức xấu và yêu cầu Alice truy cập vào tài khoản ngân hàng và Alice chuyển giao chìa khóa mà không cần suy nghĩ, cô ấy có thể bị mất hết tiền trong ngân hàng. Hơn nữa, trong nhiều trường hợp chìa khóa không cần thiết được tạo ra để truy cập vào hồ sơ của Alice, mà thông tin của Alice thì luôn được các bên khao khát và mặc dù OAuth 2.0 là giao thức ủy quyền (ví dụ: quản lý quyền của Alice), nó cũng thường được sử dụng sai cho việc xác thực (ví dụ: Xác minh danh tính của Alice) trong nhiều trường hợp.

<div class="post-image-section"><figure>
  <img src="/img/oidc-explanation/oauth-risk.jpg" alt="OAuth 2.0 risk" style="width:100%">
  </figure>
</div>

## OpenID Connect là gì?

Để giải quyết vấn đề này, OpenID Connect (sau đây gọi tắt là OIDC) là một sự bổ sung cho OAuth 2.0 nhằm kết hợp việc xác thực như một tiêu chuẩn.

```
1. Công ty A hỏi Alice, "Vậy thì, chúng tôi muốn có những thông tin của bạn được lưu giữ tại Công ty X. Bạn có thể cho chúng tôi chìa khóa nơi lưu giữ tài liệu không?"
```

OpenID Connect cũng đã thêm "bản sao tài liệu" như một tùy chọn trong bước 1 của OAuth 2.0, thay vì chìa khoá, như đã đề cập trước đó. Ở đây, chỉ định "bản sao của tài liệu" sẽ được mô tả ở bước 3


```
3. Quầy dịch vụ khách hàng của Công ty X sau đó sẽ gửi một bức thư đến Công ty A với nội dung “Chìa khóa đã sẵn sàng, hãy đến lấy.
```

<div class="post-image-section"><figure>
  <img src="/img/oidc-explanation/oidc-flow.jpg" alt="OIDC Authorization code flow" style="width:100%">
  </figure>
</div>


Nếu tất cả những gì Công ty A muốn là thông tin để xác minh danh tính của Alice, thì khi họ đã có bản sao của tài liệu này, họ không cần thực hiện các bước còn lại. Họ thậm chí sẽ không phải bận tâm phát hành một chìa khóa nào cả. Tất nhiên, bản sao của tài liệu này được ký bởi Công ty X để công ty A có thể xác nhận lại về sau. Bản sao của tài liệu này được gọi là "ID Token" và OIDC đã tiêu chuẩn hóa quy trình xác thực bằng cách đưa ID Token vào OAuth 2.0. Tất nhiên, bản sao của tài liệu này và quá trình tạo khóa có thể được thực hiện cùng một lúc và quy trình tạo khóa giống như trong OAuth 2.0.

Yêu cầu từ Công ty A trong Bước 1 được gọi là "Authorization request", thực tế được chuyển trực tiếp đến "Authorization server" (Quầy dịch vụ khách hàng của Công ty X), chứ không gửi cho Alice. Trong yêu cầu ủy quyền, "Client" (Công ty A) định cấu hình các cài đặt khác nhau, chẳng hạn như sử dụng "Implicit flow" hay "Authorization code flow", lựa chọn "scope" (phạm vi truy cập hồ sơ được gắn với người giữ khóa trong ví dụ trên ) được sử dụng và cách thông tin được gửi đến khách hàng (phương pháp gửi  sẽ được khai báo tường minh), v.v.

## Tóm lược

OIDC là giao thức dựa trên OAuth 2.0, đã tiêu chuẩn hóa  việc tạo mã truy cập và phân quyền thông qua mã truy cập  bao gồm cả ID Token. Đây là giao thức được sử dụng rộng rãi nhất để xác thực và phân quyền ở năm 2022. Trong phần tiếp theo, chúng ta sẽ xem xét việc triển khai các chức năng đăng nhập bằng OIDC.

