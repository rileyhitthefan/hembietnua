---
title: "Ứng dụng chatbot"
date: 2024-06-27T14:39:49+07:00
draft: true
weight : 1
chapter: false
pre : " <b> 1.4 </b> "
---

Chạy thử ứng dụng tại đây: https://partyrock.aws/u/js2222/G9KTPc8Ue/Geek-Chatbot 

#### Giới thiệu

Chúng ta sẽ tạo một ứng dụng chatbot bằng PartyRock, kết hợp các tiện ích **User Input, Chatbot, Text Generation và Image Generation** bẳng **prompt chaining**.

**Prompt chaining** dùng output của một tiện ích AI làm thành input của một tiện ích AI khác. Trong trường hợp này, chúng ta sẽ tạo hình ảnh dựa trên nội dung hội thoại với chatbot

---

#### Tạo từ ứng dụng trống
1. Truy cập https://partyrock.aws và đăng nhập.
2. Chọn nút **Build your own app**, sau đó chọn **Create empty app**.

![pr](/images/1-PartyRock/012-PartyRock.png)

3. Xem lại ứng dụng vừa tạo và tên ứng dụng ngẫu nhiên.

![pr](/images/1-PartyRock/013-PartyRock.png)

4. Đổi tên ứng dụng thành `Geek Chatbot`

5. Chọn nút **+ Add widget**, chọn **User Input**

![pr](/images/1-PartyRock/024-PartyRock.png)

6. Sửa các đặc tính của User Input
   - Đặt **Widget title** là `Geeky Topic`
   - Đặt **Placeholder** là `Provide a geeky topic that your chatbot is obsessed with!`
   - Chọn **Save**.

![pr](/images/1-PartyRock/025-PartyRock.png)

7. Chọn nút **+ Add widget**, chọn **Chatbot**

8. Sửa các đặc tính của Chatbot
   - Đặt **Widget title** là `Talk to your Geek Chatbot!`
   - Đặt **Placeholder** là `Geek out here!`
   - Đặt **Model** là `Claude 3 Sonnet`
   - Đặt **Initial message** là `Let’s geek out!`
   - Đặt **Prompt** là `Pretend you are obsessed with TOPIC. You only want to talk about TOPIC. The user should only want to talk about TOPIC. - The user will now have a conversation with you.`
   - Trong ô Prompt, dùng ký tự **@** để thay những chữ TOPIC bằng mục **Geeky Topic**
   - Chọn **Save**.

![pr](/images/1-PartyRock/026-PartyRock.png)

9. Điều chỉnh kích cỡ các tiện ích **Geeky Topic** và **Talk to your Geek Chatbot** bằng các kéo từ góc phải bên dưới các ô tiện ích.

10. Chọn nút **+ Add widget** hoặc ô **Create widget** ở các vùng chấm đen.

11. Thêm tiện ích **Text Generation**.

![pr](/images/1-PartyRock/027-PartyRock.png)

12. Sửa các đặc tính của Text Generation
    - Đặt **Widget title** là `Summary`
    - Đặt **Model** là `Claude 3 Haiku`
    - Đặt **Prompt** là `Write a one-line summary of the most recent message in the conversation:`
    - Nhập ký tự **@** và chọn mục `Talk to your Geek chatbot!` để thiết lập nội dung gốc
    - Chọn **Save**

![pr](/images/1-PartyRock/028-PartyRock.png)

13. Thêm tiện ích **Image Generation**.

14. Sửa các đặc tính của Image Generation
    - Đặt **Widget title** là `Image`
    - Đặt **Model** là `Stable Diffusion XL`
    - Đặt **Image description** là `An image of `
    - Nhập ký tự **@** và chọn mục `Summary` để thiết lập nội dung gốc
    - Select the **Save** button
    
![pr](/images/1-PartyRock/029-PartyRock.png)

15. Chọn chủ đề!
    - Một số chủ đề: `Aerospace engineering`, `Plumbing`, `Atlantic`

![pr](/images/1-PartyRock/030-PartyRock.png)

16. (Tuỳ chọn) Phản biện

![pr](/images/1-PartyRock/031-PartyRock.png)