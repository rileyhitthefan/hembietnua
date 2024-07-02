---
title: "Lab F-1: Bedrock Console"
date: 2024-06-28T15:17:27+07:00
draft: false
chapter : false
weight: 1
---
Trong bài lab này, chúng ta sẽ 

1. Truy cập console **Amazon Bedrock**.

![bedrock-console](/images/2-Bedrock/F-1/1-bedrock-welcome.png)

2. Mở menu **Amazon Bedrock**, chọn **Examples**.

   - Mỗi ví dụ bao gồm câu lệnh hướng dẫn, điểu chỉnh dự đoán, mẫu trả lời và chi tiết yêu cầu API.

![bedrock-console](/images/2-Bedrock/F-1/2-examples.png)

3. Chọn một ví dụ, chọn **Open in Playground** để xem thử.

![bedrock-console](/images/2-Bedrock/F-1/3-playground.png)

4. Chọn **Run** và xem thử câu trả lời.

   - Tham số **Temperature** cho phép model "sáng tạo" hơn khi trả lời. Temperater = 0 sẽ đưa ra kết quả không ngẫu nhiên (câu trả lời không thay đổi nhiều qua mỗi lượt chạy). 
   - Tham số **Response length** xác định số lượng tokens được trả về ở câu trả lời. Thay đổi tham số này để rút ngắn hoặc tăng thêm độ dài nội dung câu trả lời của model. Nếu độ dài quá ngắn, câu trả lời có thể bị cắt giữa chừng.
   - Xem thêm link **Info** để xem giải thích thêm về các tham số.

5. Quay lại trang các ví dụ, xem qua JSON yêu cầu API cho câu lệnh được sử dụng.

![bedrock-console](/images/2-Bedrock/F-1/4-reqAPI.png)


