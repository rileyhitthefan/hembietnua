---
title: "Lab F-6: Điều chỉnh biến thiên của phản hồi"
date: 2024-06-28T15:18:06+07:00
draft: false
chapter : false
weight: 6
---

**Temperature** là tham số ảnh hướng mức độ biến thiên của kết quả khởi tạo bời model nền.

Tham số **Temperature** cho phép model "sáng tạo" hơn khi trả lời. Temperater = 0 sẽ đưa ra kết quả không ngẫu nhiên (câu trả lời không thay đổi nhiều qua mỗi lượt chạy). Temperature càng cao, sẽ có nhiều biến thể của kết quả trả về. Trong các ngữ cảnh sáng tạo, temperature cao sẽ thích hợp hơn. Trong các quá trình kinh doanh hay lập trình, temperature bằng 0 có thể sẽ tốt nhất.

_* Nếu model cập nhật phiên bản mới, có thể có một số khác biệt so với phiên bản trước đó. Luôn có biến thiên qua thời gian, kể cả khí temperature là 0._

## Tạo script python

1. Vào thư mục **workshop/labs/temperature**, mở file  **temperature.py**

![Temperature](/images/2-Bedrock/F-6/1.png)

2. Thêm câu lệnh import.

   - Sử dụng thư viện của LangChain để gọi Bedrock, truy cập biến của command line, và đọc biến môi trường.

```python
import sys
from langchain_community.llms import Bedrock
```

3. Tạo hàm gọi Bedrock với tham số phù hợp theo model.

   - Tạo Bedrock client bằng LangChain, đặt câu lệnh, và đặt temperature

```python
def get_text_response(input_content, temperature): #text-to-text client function
    
    model_kwargs = { #AI21
        "maxTokens": 1024, 
        "temperature": temperature, 
        "topP": 0.5, 
        "stopSequences": [], 
        "countPenalty": {"scale": 0 }, 
        "presencePenalty": {"scale": 0 }, 
        "frequencyPenalty": {"scale": 0 } 
    }
    
    llm = Bedrock( #create a Bedrock llm client
        model_id="ai21.j2-ultra-v1",
        model_kwargs = model_kwargs
    )
    
    return llm.invoke(input_content) #return a response to the prompt
```

4. Gửi biến khai báo sử dụng command line vào hàm get_text_response.

   - Chúng ta sẽ dùng biến (prompt) và biến (temperature) từ command line.
   - Gọi hàm này trong một vòng lập để quan sát sự thay đổi của câu trả lời.

```python
for i in range(3):
    response = get_text_response(sys.argv[1], float(sys.argv[2]))
    print(response)
```

5. Lưu file.

## Chạy script python
1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.
```bash
cd ~/environment/workshop/labs/temperature
```

2. Chạy script python bằng terminal, đặt temperature là 0.0.
```bash
python temperature.py "Write a haiku about a long journey:" 0.0
```

Câu trả lời sẽ được in trong terminal:
![Temperature](/images/2-Bedrock/F-6/2.png)

3. Chạy script python bằng terminal, đặt temperature là 1.0.
```bash
python temperature.py "Write a haiku about a long journey:" 1.0
```

Câu trả lời sẽ được in trong terminal:
![Temperature](/images/2-Bedrock/F-6/3.png)