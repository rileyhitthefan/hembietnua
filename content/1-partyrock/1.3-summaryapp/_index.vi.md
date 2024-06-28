---
title: "Ứng dụng tóm tắt nội dung"
date: 2024-06-27T14:34:27+07:00
draft: true
weight : 1
chapter: false
pre : " <b> 1.3 </b> "
---
Chạy thử ứng dụng tại đây: https://partyrock.aws/u/js2222/aw3_JHzyG/Content-Summarizer 

#### Giới thiệu

Chúng ta sẽ tự tạo một ứng dụng tóm tắt nội dung đơn giản. Ngoài tính năng tạo app bằng câu lệnh miêu tả, PartyRock cũng cho phép người dùng tạo ứng dụng bằng **app editor** trên website.

Các bước sau đây hướng dẫn sử dụng tiện ích **User input** và **Text generation** để tự động tạo ra tóm tắt chi tiết và tóm tắt một dòng từ các nội dung dài.

---

#### Tạo ứng dụng trống
1. Go to https://partyrock.aws và đăng nhập.
2. Chọn nút **Build your own app**, chọn **Create empty app**.

![pr](/images/1-PartyRock/012-PartyRock.png)

3. Xem lại ứng dụng vừa tạo và tên ứng dụng ngẫu nhiên.

![pr](/images/1-PartyRock/013-PartyRock.png)

4. Đổi tên ứng dụng thành `Content Summarizer`

![pr](/images/1-PartyRock/014-PartyRock.png)

#### Thêm tiện tính User input và Text generation
5. Chọn nút **+ Add widget**, chọn mục **Static Text**

![pr](/images/1-PartyRock/015-PartyRock.png)

6. Sửa các đặc tính của Static Text.
   - Đặt tên tiện ích `Welcome`
   - Đổi Content:
   
        ```command
        # Welcome to the Content Summarizer application!

        Instructions:

        1. Paste your content
        2. Get your summaries!

        ## WARNING: Never share sensitive information when building or using a PartyRock app. This includes personal information and internal company information.
        ```
   - Có thể điều chỉnh kích cỡ của các tiện ích bằng cách kéo từ góc phải bên dưới ô tiện ích.
   - Bấm **Save**

![pr](/images/1-PartyRock/016-PartyRock.png)

{{% notice note %}}
Tiện ích Static Text dùng ngôn ngữ **Markdown** để hiển thị nội dung.
{{% /notice %}}

7. Chọn nút **+ Add widget**, chọn **User Input**

![pr](/images/1-PartyRock/017-PartyRock.png)

8. Sửa các đặc tính của tiện ích User Input.

   - Đặt **Widget title** là `Paste your content here`
   - Đặt **Placeholder** là `Content to summarize (NO SENSITIVE CONTENT)`
   - Chọn **Save**.

![pr](/images/1-PartyRock/018-PartyRock.png)

#### Thêm các tiện ích AI
9. Chọn nút **+ Add widget**, sau đó chọn **Text Generation**

10. Sửa các đặc tính của tiện ích Text Generation.
    - Đặt **Widget title** là `Detailed summary`
    - Đặt **Model** là `Command`
    - Trong **Prompt**, nhập ` Write a detailed summary based on this content:`, nhập @ và chọn `Paste your content here` để đặt input người dùng làm nội dung gốc.
    - Chọn **Save**.

![pr](/images/1-PartyRock/019-PartyRock.png)

11. Điều chỉnh kích cỡ của tiện ích **Detailed summary** bằng cách kéo từ góc phải bên dưới ô tiện ích.

12. Thêm tiện ích **Text Generation**.

![pr](/images/1-PartyRock/020-PartyRock.png)

13. Sửa các đặc tính của tiện ích Text Generation.

    - Đặt **Widget title** là `One-line summary`
    - Đặt **Model** là `Claude 3 Sonnet`
    - Trong **Prompt**, nhập `Write a one-line summary based on this content:`, sau đó nhập **@** và chọn `Paste your content here` để đặt input người dùng làm nội dung gốc.
    - Chọn vào mục **Advanced settings** và chỉnh **Temperature** thành **0.5** để tạo sự ngẫu nhiên cho mỗi câu trả lời của model.
    - Chọn **Save**.

![pr](/images/1-PartyRock/021-PartyRock.png)

![pr](/images/1-PartyRock/022-PartyRock.png)


#### Test the app
14. Sao chép đoạn sau vào ô **Paste your content here**.

    ```command
    PartyRock, an Amazon Bedrock playground, is a shareable generative AI app building playground. Experiment with prompt engineering in a hands-on and fun way. In a short time, you can build, share, and remix apps, to get inspired while playing with generative AI. For example, you can:

        Build an app to generate dad jokes on the topic of your choice.
        Create the perfect playlist based on your musical tastes.
        Recommend what to serve based on ingredients in your pantry.
        Create and play a virtual trivia game online with friends from around the world.
        Create an AI storyteller to guide your next fantasy -playing campaign.

    By building and playing with PartyRock apps, you learn about the fundamental techniques and capabilities needed to get started with generative AI, including understanding how a foundation model responds to a given prompt, experimenting with different text-based prompts, and chaining prompts together. Anyone can access PartyRock through its intuitive web-based UI. For anyone interested in learning the fundamental skills needed to harness the power of generative AI, PartyRock uses Amazon Bedrock to create an accessible environment for experimentation with powerful foundational models (FMs). The tool builds confidence by providing new PartyRock users with free trial use for a limited time, and the PartyRock Guide helps learners extend their knowledge. Builders who are ready to move into production can proceed to the Bedrock console using the basic prompts and skills developed in PartyRock.

    Any builder can experiment with PartyRock by creating a profile using a social login from amazon.com, Apple, or Google. PartyRock is separate from the AWS console and does not require an AWS account to get started.
    PartyRock is for EVERYONE. It has been designed in a way that is friendly and accessible to builders of all skill levels, particularly those who have limited experience with generative AI. Even if you have no coding experience, you can use text-based prompts to experiment with foundation models and create your own generative AI-powered apps. Developers can explore model capabilities and refine prompt engineering techniques in PartyRock without managing API calls.

    ```

![pr](/images/1-PartyRock/023-PartyRock.png)

{{% notice warning %}}
Đừng bao giờ chia sẻ các thông tin cá nhân nhạy cảm trong snapshot hoặc PartyRock.
{{% /notice %}}