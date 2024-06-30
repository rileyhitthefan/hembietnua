---
title: "AWS Cloud9 Setup"
date: 2024-06-27T14:35:46+07:00
draft: false
weight: 2
---
Chúng ta sẽ dùng [AWS Cloud9](https://aws.amazon.com/cloud9/) làm IDE. AWS Cloud9 có thể tạo ứng dụng với Amazon Bedrock - giống như các công cụ khác (VS Code, PyCharm, v.v.), [Amazon SageMaker Studio](https://aws.amazon.com/sagemaker/studio/ ) ,hoặc Jupyter Notebook.

Sau đây chúng ta sẽ tạo [môi trường AWS Cloud9](https://docs.aws.amazon.com/cloud9/latest/user-guide/environments.html) để tạo và chạy các ứng dụng gen AI.

{{% notice info %}}
**Chú Ý**\
    - AWS Cloud9 sẽ được chạy từ cùng một tài khoản và region của các model Bedrock đã được kích hoạt.\
    - Tài khoản và region có cấu hình VPC mặc định (mặc định của AWS).\
\
Nếu có vấn đề xảy ra, bạn có thể phải sử dụng Bedrock từ máy tính cá nhân hoặc tạo môi trường ảo thay thế.
{{% /notice %}}

## Cloud9 Setup instructions

1. Đảm bảo bạn đang ở cùng region từ bước thiết lập Amazon Bedrock.

2. Từ console AWS, tìm và chọn **Cloud9** từ thanh tìm kiếm để truy cập.

![Cloud9](/images/2-Bedrock/prep/Prep-9.png?classes=border)

3. Chọn **Create environment**

![Cloud9](/images/2-Bedrock/prep/Prep-10.png)

4. Chỉnh sửa thông tin môi trường.
   - Name: `bedrock-environment`.

![Cloud9](/images/2-Bedrock/prep/Prep-11.png)

5. Chỉnh sửa thông tin EC2.

   - **Instance type** là **t3.small**
   - **Platform** là **Ubuntu Server 22.04 LTS**
   - **Timeout** là **4 hours**

![Cloud9](/images/2-Bedrock/prep/Prep-12.png)

{{% notice warning %}}
**Chọn Ubuntu làm ngôn ngữ điều hành**\
Đảm bảo môi trường Cloud9 sẽ chạy Ubuntu Server 22.04 LTS để có phiên bản Python có thể dụng LangChain và các thư viện quan trọng khác.
{{% /notice %}}

6. Chọn **Create**.

![Cloud9](/images/2-Bedrock/prep/Prep-13.png)

7. Chờ đợi có đáng sợ.

   - Khi môi trường khởi tạo thành công, sẽ có thông báo **"Successfully created bedrock-environment"**
   - Chọn Environments list, chọn **Open** để chạy Cloud9 IDE trong một tab mới.

{{% notice note %}}
**Xử lý lỗi khi tạo môi trường**\
    - Nếu có thông báo lỗi về instance type không tồn tại trong AZ, xoá môi trường vừa tạo và khởi tạo lại bằng isntance type khác.
{{% /notice %}}

![Cloud9](/images/2-Bedrock/prep/Prep-14.png)

8. Kiểm tra môi trường Cloud9 đang chạy.

![Cloud9](/images/2-Bedrock/prep/Prep-15.png)

   - Có thể thay đổi vị trí các cửa sổ tuỳ ý

![Cloud9](/images/2-Bedrock/prep/Prep-16.png)

{{% notice tip %}}
**Congratulations!**\
Bạn đã tạo thành công IDE AWS Cloud9!
{{% /notice %}}
