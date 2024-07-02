---
title: "Lab F-4: Giới thiệu Langchain"
date: 2024-06-28T15:18:01+07:00
draft: false
chapter : false
weight: 4
---

Chúng ta sẽ sử dụng LangChain để gọi đến Bedrock, gần tương tự với cách gọi API Bedrock bằng Boto3. LangChain giản lược nhiều chi tiết hơn so với Boto3, đặc biệt nếu bạn muốn tập trung vào văn bản nhập và xuất. Boto3 yêu cầu sử dụng nhiều code hơn, nhưng sẽ cho phép bạn toàn quyền truy cập đến các yêu cầu và kết quả trả lời từ file JSON.

Bạn có thể chạy thử code ứng dụng bằng các sao chép các đoạn code sau đây và dán vào file Python tương ứng.

## Tạo script Python

1. Vào thư mục **workshop/labs/LangChain**, mở file **bedrock_api.py**
![langchain](/images/2-Bedrock/F-4/1.png)

2. Thêm câu lệnh import.

   - Sử dụng thư viện của LangChain để gọi Amazon Bedrock.

```py
from langchain_community.llms import Bedrock
```

3. Tạo client Bedrock bằng LangChain.

   - Client này cho phép sử dụng và tự động thay đổi phép gọi của LangChain để gọi các model từ Bedrock.

```py
llm = Bedrock( #create a Bedrock llm client
    model_id="ai21.j2-ultra-v1" #set the foundation model
)
```

4. Đặt câu lệnh để gọi Bedrock.

   - Câu lệnh là payload tối thiểu để gửi đến Bedrock. Sau này chúng ta sẽ tìm hiểu cách thay đổi tham số suy đoán.

```py
prompt = "What is the largest city in Vermont?"
```

5. Gọi API Bedrock.

   - Dùng hàm invoke của LnagChain. Clien của Bedrock cũng tự động tách văn bản trả lời từ payload trả lời, kết quả trả lời là dạng chuỗi.
```py
response_text = llm.invoke(prompt) #return a response to the prompt
```

6. In câu trả lời.

   - In chuỗi được trả về từ client của LangChain.

```py
print(response_text)
```

7. Lưu file.

## Chạy script

1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.
```bash
cd ~/environment/workshop/labs/langchain
```

2. Chạy script python bằng terminal.
```bash
python bedrock_langchain.py
```

3. Câu trả lời sẽ được in trong terminal.

![langchain](/images/2-Bedrock/F-4/2.png)

