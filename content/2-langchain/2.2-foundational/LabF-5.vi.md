---
title: "Lab F-5: Tham số suy đoán"
date: 2024-06-28T15:18:04+07:00
draft: false
chapter : false
weight: 5
---

[Tham số suy đoán](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters.html) dùng để cấu hình cách trả kết quả của model nền. Các tham số thay đổi tuỳ model:

- Ở mức tối thiểu, có các tham số thay đổi mức độ biến thiên (**Temperature, Top P**) and **token length** của câu trả lời.
  - Chúng ta sẽ tìm hiểu thêm về temperature trong phần [Điều chỉnh biến thiên dự đoán](./LabF-6.md)
  - **Tokens** có thể được xem như một từ hoặc 1 phần của một từ. Tỉ lệ số lượng token trên từ thay đổi giữa các model. Những từ thông dụng có thể được biểu diễn bằng 1 token, còn những từ hiếm gpặ hơn có thể chứa nhiều token.
- **Stop Sequences** dùng để tách các ví dụ khi few-shot prompting, hoặc ngăn cản model tự nói chuyện với bản thân.

Trong lab này, chúng ta sẽ điều chỉnh các tham số suy đoán gửi đến Amazon Bedrock thông qua client của LangChain. Client Bedrock nhận biến `model_kwargs` cho phép đặt ra các tham số suy đoán cho model.

{{% notice note%}}
`model_kwargs` objects không tương thích giữa các model Bedrock. Nếu thay đổi model, bạn cũng phải chỉnh sửa tham số cho phù hợp.
{{% /notice %}}

## Tạo script python

1. Vào thư mục **workshop/labs/params**, mở file **bedrock_api.py**
![Params](/images/2-Bedrock/F-5/1.png)

2. Thêm câu lệnh import.

   - Sử dụng thư viện của LangChain để gọi Bedrock, truy cập biến của command line, và đọc biến môi trường.
```py
import sys
from langchain_community.llms import Bedrock
```

3. Tạo hàm helper để lấy tên tham số phù hợp model.

   - Mỗi model có một bộ tham số suy đoán riêng. Hàm helper này trả về bộ tham số phù hợp dựa trên phần tên từ id của model.
   - Bạn có thể dùng nút **View APU request** để xem lại các tham số được sử dụng.

```py
def get_inference_parameters(model): #return a default set of parameters based on the model's provider
    bedrock_model_provider = model.split('.')[0] #grab the model provider from the first part of the model id
    
    if (bedrock_model_provider == 'anthropic'): #Anthropic model
        return { #anthropic
            "max_tokens": 512,
            "temperature": 0, 
            "top_k": 250, 
            "top_p": 1, 
            "stop_sequences": ["\n\nHuman:"] 
           }
    
    elif (bedrock_model_provider == 'ai21'): #AI21
        return { #AI21
            "maxTokens": 512, 
            "temperature": 0, 
            "topP": 0.5, 
            "stopSequences": [], 
            "countPenalty": {"scale": 0 }, 
            "presencePenalty": {"scale": 0 }, 
            "frequencyPenalty": {"scale": 0 } 
           }
    
    elif (bedrock_model_provider == 'cohere'): #COHERE
        return {
            "max_tokens": 512,
            "temperature": 0,
            "p": 0.01,
            "k": 0,
            "stop_sequences": [],
            "return_likelihoods": "NONE"
        }
    
    elif (bedrock_model_provider == 'meta'): #META
        return {
            "temperature": 0,
            "top_p": 0.9,
            "max_gen_len": 512
        }
    
    elif (bedrock_model_provider == 'mistral'): #MISTRAL
        return {
            "max_tokens" : 512,
            "stop" : [],    
            "temperature": 0,
            "top_p": 0.9,
            "top_k": 50
        } 

    else: #Amazon
        #For the LangChain Bedrock implementation, these parameters will be added to the 
        #textGenerationConfig item that LangChain creates for us
        return { 
            "maxTokenCount": 512, 
            "stopSequences": [], 
            "temperature": 0, 
            "topP": 0.9 
        }
```

4. Tạo hàm gọi Bedrock với các tham số thích hợp cho model.

   - Khởi tạo client Bedrock từ LangChain, chọn model, áp dụng các tham số suy đoán cho model.

```py
def get_text_response(model, input_content): #text-to-text client function
    
    model_kwargs = get_inference_parameters(model) #get the default parameters based on the selected model
    
    llm = Bedrock( #create a Bedrock llm client
        model_id=model, #use the requested model
        model_kwargs = model_kwargs
    )
    
    return llm.invoke(input_content) #return a response to the prompt
```

5. Gửi biến khai báo sử dụng command line vào hàm get_text_response.

   - Chúng ta sẽ dùng biến (Bedrock Model ID) và (prompt) từ command line.
```py
response = get_text_response(sys.argv[1], sys.argv[2])
```

6. In câu trả lời.
   - In văn bản trả lời từ model
```python
print(response)
```

7. Lưu file.

## Chạy script python
1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.
```bash
cd ~/environment/workshop/labs/params
```

2. Chạy script python bằng terminal.
```bash
python params.py "ai21.j2-ultra-v1" "Write a haiku:"
```

Câu trả lời sẽ được in trong terminal:
![Params](/images/2-Bedrock/F-5/2.png)

3. Dùng model Mistral
```bash
python params.py "mistral.mixtral-8x7b-instruct-v0:1" "[INST]Write a haiku:[/INST]" 
```

4. Dùng model Cohere Command.
```bash
python params.py "cohere.command-text-v14" "Write a haiku:"
```

5. Dùng model Meta Llama.
```bash
python params.py "meta.llama2-70b-chat-v1" "Write a haiku:"
```

6. Dùng model Amazon Titan.
```bash
python params.py "amazon.titan-text-express-v1" "Please write a haiku:"
```

Đọc thêm về tham số suy đoán: [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters.html).
