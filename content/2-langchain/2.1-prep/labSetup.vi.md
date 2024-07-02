---
title: "Setup mội trường"
date: 2024-06-27T14:35:46+07:00
draft: false
weight: 3
---

## Tảo các thiết lập và tài nguyên cần thiết
1. Trên AWS Cloud9 IDE, chọn **bash terminal**.

![Bash](/images/2-Bedrock/prep/Prep-16.png)

2. Dán và chạy câu lệnh sau vào terminal để tải và giái nén code.

```
cd ~/environment/
curl 'https://static.us-east-1.prod.workshops.aws/public/4aaab724-21a5-4155-80ee-db443f910e6c/assets/workshop.zip' --output workshop.zip
unzip workshop.zip
```

Sau khi thành công, kết quả giải nén sẽ hiện lên terminal.

![Bash](/images/2-Bedrock/prep/Prep-17.png)

3. Tải các tài nguyên phụ thuộc
```
pip3 install -r ~/environment/workshop/setup/requirements.txt -U
```

Thông báo tải thành công (bỏ qua warning):

![Bash](/images/2-Bedrock/prep/Prep-18.png)

4. Kiểm tra thiết lập bằng câu lệnh:

```
cd ~/environment/
python ~/environment/workshop/completed/api/bedrock_api.py
```

Nếu thiết lập hoàn chỉnh, terminal sẽ output câu trả lời về Manchester, New Hampshire

![Bash](/images/2-Bedrock/prep/Prep-19.png)

## Các bài thực hành!
Bạn có thể chọn bài thực hành bất kỳ để bắt đầu hoàn thiện workshop, không quan trọng kinh nghiệm.

- [🧱Foundational concepts Labs](../2.2-foundational/)
- [📙Basic patterns Labs](../2.3-basic/)
- [📚Text patterns Labs](../2.4-text/)
- [🌃Multimodal & Image Labs](../2.5-image/)
- [🔐Security & Safeguard Labs](../2.6-security/)
- [💬Prompt Engineering Labs](../2.7-prompteng/)
- [🎁Bonus Labs](../2.8-bonus/)

