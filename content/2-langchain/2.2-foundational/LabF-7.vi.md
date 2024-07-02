---
---
title: "Lab F-7: Streaming API"
date: 2024-06-28T15:18:11+07:00
draft: false
chapter : false
weight: 7
---

Chúng ta sẽ dùng **streaming API** để gọi trực tiếp Amazon Bedrock.

Phản hồi theo dòng phù hợp cho trường hợp muốn đưa câu trả lời ngay lập tức đến người dùng cuối. Bạn có thể in câu trả lời từng ít chữ một, thay vì phải đợi đến khi toàn bộ phản hồi thành công.

Chúng ta sẽ dùng thư viện **Boto3** cho lab này.

## Tạo script python

1. Vào thư mục **workshop/labs/intro_streaming**, mở file  **intro_streaming.py**

![streaming](/images/2-Bedrock/F-7/1.png)

2. Thêm câu lệnh import.

   - Chúng ta sử dụng thư viện AWS Boto3 để gọi Amazon Bedrock
```python
import json
import boto3
```

3. Khởi tạo phiên chạy Bedrock.

   - Tạo client Bedrock.

```python
session = boto3.Session()

bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
```

4. Tạo phương thức xử lý phản hồi từ dòng phản hồi.

   - Phương thức này cho phép in cụm phản hồi ngay khi kết quả được trả về từ streaming api.

```python
def chunk_handler(chunk):
    print(chunk, end='')
```

5. Tạo hàm để gọi the Bedrock streaming API.

   - Dùng hàm `invoke_model_with_response_stream` của Bedrock để gọi tới điểm cuối của streaming API.
   - Khi cụm phản hồi được trả về, lệnh này tách văn bản phản hồi từ file JSON và đưa vào phương thức xử lý phản hồi.

```python
def get_streaming_response(prompt, streaming_callback):

    bedrock_model_id = "anthropic.claude-3-sonnet-20240229-v1:0" #set the foundation model
    
    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 8000,
        "temperature": 0,
        "messages": [
            {
                "role": "user",
                "content": [{ "type": "text", "text": prompt } ]
            }
        ],
    })
    
    response = bedrock.invoke_model_with_response_stream(modelId=bedrock_model_id, body=body) #invoke the streaming method
    
    for event in response.get('body'):
        chunk = json.loads(event['chunk']['bytes'])

        if chunk['type'] == 'content_block_delta':
            if chunk['delta']['type'] == 'text_delta':
                streaming_callback(chunk['delta']['text'])
```

6. In câu trả lời.
   - Đặt câu lệnh và đưa vào hàm `get_streaming_response`, cùng với phương thức nhận phản hồi.

```python
prompt = "Tell me a story about two puppies and two kittens who became best friends:"
                
get_streaming_response(prompt, chunk_handler)
```

7. Lưu file.

## Run the script

1. Chọn bash terminal trong AWS Cloud9, di chuyến đến thư mục sau.
```bash
cd ~/environment/workshop/labs/intro_streaming
```

2. Chạy script python bằng terminal.

```bash
python intro_streaming.py
```

3. Câu trả lời sẽ được in trong terminal.

![streaming](/images/2-Bedrock/F-7/2.png)