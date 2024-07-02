---
title: "Lab F-2: InvokeModel API"
date: 2024-06-28T15:17:55+07:00
draft: false
chapter : false
weight: 2
---
Trong lab này, chúng ta thực hành cách gọi API cơ bản đến Bedrock.

Bạn có thể chạy thử code ứng dụng bằng các sao chép các đoạn code sau đây và dán vào file Python tương ứng.

## Tạo script Python

1. Vào thư mục **workshop/labs/api**, mở file **bedrock_api.py**

![bedrockAPI](/images/2-Bedrock/F-2/1.png)

2. Thêm câu lệnh import.

   - Chúng ta sử dụng thư viện AWS Boto3 để gọi Amazon Bedrock

```py
import json
import boto3
```

3. Khởi tạo phiên chạy Bedrock.

   - Tạo client Bedrock.

```py
session = boto3.Session()

bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
```

4. Tạo payload để gọi API.

   - Chọn model, câu lệnh và các tham số để suy đoán cho model đã chọn.

```py
bedrock_model_id = "ai21.j2-ultra-v1" #set the foundation model

prompt = "What is the largest city in New Hampshire?" #the prompt to send to the model

body = json.dumps({
    "prompt": prompt, #AI21
    "maxTokens": 1024, 
    "temperature": 0, 
    "topP": 0.5, 
    "stopSequences": [], 
    "countPenalty": {"scale": 0 }, 
    "presencePenalty": {"scale": 0 }, 
    "frequencyPenalty": {"scale": 0 }
}) #build the request payload
```

5. Gọi API Bedrock.

   - Dùng hàm invoke_model của Bedrock.

```py
response = bedrock.invoke_model(body=body, modelId=bedrock_model_id, accept='application/json', contentType='application/json') #send the payload to Bedrock
```

6. In câu trả lời.

   - Tách và in đoạn văn bản trả lời của model từ JSON.

```py
response_body = json.loads(response.get('body').read()) # read the response

response_text = response_body.get("completions")[0].get("data").get("text") #extract the text from the JSON response

print(response_text)
```

7. Lưu file.

## Chạy script

1. Chọn bash terminal trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/api
```

2. Chạy script python bằng terminal.

```bash
python bedrock_api.py
```

3. Câu trả lời sẽ được in trong terminal.

![bedrockAPI](/images/2-Bedrock/F-2/2.png)