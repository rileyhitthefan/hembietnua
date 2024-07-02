---
title: "Chuẩn bị Amazon Bedrock"
date: 2024-06-27T14:35:46+07:00
draft: false
weight: 1
---
Chúng ta sử dụng [Amazon Bedrock](https://aws.amazon.com/bedrock/) để truy cập các LLM model nền tảng trong workshop này. 

Sau đây chúng ta sẽ chỉnh sửa quyền truy cấp các model trong Amazon Bedrock để tạo và chạy các ứng dụng gen AI. Amazon Bedrock cung cấp nhiều model nền tảng từ nhiều nhà cung cấp.

## Hướng dẫn thiết lập Amazon Bedrock
1. Truy cập **Amazon Bedrock** bằng AWS Console

![Bedrock](/images/2-Bedrock/prep/Prep-2.png)

2. Mở rộng menu.

![Bedrock](/images/2-Bedrock/prep/Prep-3.png)


![Bedrock](/images/2-Bedrock/prep/Prep-4.png)

4. Chọn nút **Enable specific models button**.

![Bedrock](/images/2-Bedrock/prep/Prep-4'.png)

5. Đánh dấu để chọn các mô hình sau đây. Nếu sử dụng tài khoản AWS cá nhân, bạn sẽ không bị tính phí kích hoạt truy cập, chỉ bị tính phí sử dụng model khi thực hành. 

- AI21 > Jurassic-2 Ultra
- Amazon (select Amazon to automatically select all Amazon Titan models)
- Anthropic > Claude 3 Sonnet
- Cohere > Command
- Meta > Llama 2 Chat 70B
- Mistral AI > Mixtral 8x7B Instruct

Chọn **Next**.

![Bedrock](/images/2-Bedrock/prep/Prep-5.png)

6. Ở trang **Review and submit**, chọn **Submit**.

![Bedrock](/images/2-Bedrock/prep/Prep-6.png)

7. Quan sát trạng thái truy cập, có thể chọn **Refresh** để kiểm tra quá trình cập nhật từ **In Progress** thành **Access granted**.

8. Xác minh trạng thái **Access granted** cho những model đã chọn ở trên.

![Bedrock](/images/2-Bedrock/prep/Prep-8.png)

{{% notice tip %}}
**Congratulations!**\
Chúc mừng bạn đã thành công thiết lập Amazon Bedrock!
{{% /notice %}}