---
title: "Ứng dụng chat bằng tập tin"
date: 2024-06-27T14:34:42+07:00
draft: true
weight : 1
chapter: false
pre : " <b> 1.5 </b> "
---

#### Giới thiệu

Chúng ta sẽ tạo một ứng dụng chatbot bằng tập tin bằng PartyRock, kết hợp tiện tích **Document** và **Chatbot** để trò chuyện về một tập tin được tải lên.

{{% notice warning %}}
Đừng bao giờ chia sẻ các thông tin cá nhân/doanh nghiệp nhạy cảm trên PartyRock.
{{% /notice %}}

Một số định dạng tập tin được hỗ trợ:
- PDF
- Text (.txt)
- Word (.docx)
- Markdown (.md)
- HTML
- CSV

Mỗi tập tin giới hạn số lượng 120,000 ký tự.

{{% notice note %}}
Có thể chọn tải lên file nhỏ hơn để giảm thiểu credits PartyRock.
{{% /notice %}}

{{%attachments title="PDF Amazon's leadership principles, file nhỏ:" pattern=".*(pdf)"/%}}

---

#### Tạo từ ứng dụng trống
1. Truy cập https://partyrock.aws và đăng nhập.
2. Chọn nút **Build your own app**, sau đó chọn **Create empty app**.

![pr](/images/1-PartyRock/012-PartyRock.png)

3. Xem lại ứng dụng vừa tạo và tên ứng dụng ngẫu nhiên.

![pr](/images/1-PartyRock/013-PartyRock.png)

4. Đặt the page title là `Document Chat`

5. Chọn nút **+ Add widget**, sau đó chọn **Document**

6. Sửa các đặc tính của tiện ích Document
   - Đặt **Widget title** là `Upload a Document`
   - Chọn **Save**.

![pr](/images/1-PartyRock/032-PartyRock.png)

7. Chọn nút **+ Add widget**, sau đó chọn **Chatbot**

8. Edit the Chatbot widget’s properties
   - Đặt **Widget title** là `Chat About the Document`
   - Đặt **Placeholder** là `Chat here!`
   - Đặt **Model** là `Claude 3 Sonnet`
   - Đặt **Initial message** là `Ready to chat!`
   - Đặt **Prompt** là: `Support the conversation using the information within the document tags. <document>[Upload a Document]</document> The user will now have a conversation with you.`
   - Chọn **Save**

![pr](/images/1-PartyRock/033-PartyRock.png)

9. Tải tập tin lên để bắt đầu trò chuyện!

{{%attachments title="PDF Amazon's leadership principles, file nhỏ:" pattern=".*(pdf)"/%}}

{{% notice warning %}}
Đừng bao giờ chia sẻ các thông tin cá nhân/doanh nghiệp nhạy cảm trên PartyRock.
{{% /notice %}}

![pr](/images/1-PartyRock/034-PartyRock.png)