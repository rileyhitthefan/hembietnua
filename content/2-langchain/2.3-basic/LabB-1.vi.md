---
title: "Lab B-1: Tạo văn bản"
date: 2024-07-01T13:20:41+07:00
draft: false
chapter : false
weight: 1
---

Xây dựng ứng dụng tạo văn bản cơ bản bằng [Amazon Bedrock](https://aws.amazon.com/bedrock/), **LangChain**, và **Streamlit**. Chúng ta sẽ nhận dữ liệu người dùng nhập vào, đưa đến Bedrock và trả phản hồi từ model nền. Đây là một ví dụ khá căn bản, những sẽ giúp hiểu rõ hơn về cách tạo một nguyên mãu gen AI căn bản sử dụng ít code.

{{% notice note %}}
**Chỉ muốn chạy ứng dúng?**\
Đến mục [Chạy ứng dụng tạo sẵn.](#run-the-streamlit-app)
{{% /notice %}}

### Trường hợp sử dụng 
Mẫu tạo văn bản căn bản có thể dùng cho:  
- Sáng tạo nội dung chung chung, không yêu cầu tính thực tế cao
- Hỏi đáp cơ bản cho những kiến thức tổng quát được tìm nhiều trên internet

### Kiến trúc
![Architecture](/images/2-Bedrock/basic/B-1/architecture.png)
Ứng dụng gồm 2 file: 1 file front-end Streamlist, 1 file cho các thự viện hỗ trợ để gọi API Bedrock.

## Tạo script thư viện
Tạo thư viện hỗ trỡ để kết nối front-end Streamlit đến back-end Bedrock.

1. Vào thư mục **workshop/labs/text**, mở file **text_lib.py**

![textlib.py](/images/2-Bedrock/basic/B-1/1.png)

2. Thêm câu lệnh import.   
   - Dùng LangChain để gọi Bedrock và đọc biến môi trường.

```py
from langchain_community.llms import Bedrock
```

3. Thêm hàm gọi Bedrock.

   - Hàm này tạo client Bedrock bằng LangChain, sau đó đưa dữ liệu nhập từ Streamlit vào Bedrock.

```py 
def get_text_response(input_content): #text-to-text client function

    llm = Bedrock( #create a Bedrock llm client
        model_id="cohere.command-text-v14", #set the foundation model
        model_kwargs={
            "max_tokens": 512,
            "temperature": 0,
            "p": 0.01,
            "k": 0,
            "stop_sequences": [],
            "return_likelihoods": "NONE"
        }
    )
    
    return llm.invoke(input_content) #return a response to the prompt
```

4. Lưu file.

## Tạo giao diện người dùng của app Streamlit

1. Trong cùng thư mục với script thư viện, mở file **text_app.py**

2. Thêm câu lệnh import.
   - Dùng các phần tử Streamlit và gọi các hàm từ script thư viện back-end.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
import text_lib as glib #reference to local lib script
```

3. Thêm tiêu đề trang và file config.

```py
st.set_page_config(page_title="Text to Text") #HTML title
st.title("Text to Text") #page title
```

4. Thêm phần tử input.
   - Tạo một hộp nhập đoạn văn bản và nút để gửi câu lệnh từ người dùng đến Bedrock.

```py
input_text = st.text_area("Input text", label_visibility="collapsed") #display a multiline text box with no label
go_button = st.button("Go", type="primary") #display a primary button
```

5. Thêm phần tử output.  
   - Dùng lệnh if để xử lý hoạt động nút chọn. Hiển thị con xoay trong lúc chờ, và viết lại output lên trang web sau khi trả kết quả.

```py
if go_button: #code in this if block will be run when the button is clicked
    
    with st.spinner("Working..."): #show a spinner while the code in this with block runs
        response_content = glib.get_text_response(input_content=input_text) #call the model through the supporting library
        
        st.write(response_content) #display the response content
```

6. Lưu file.

## Chạy ứng dụng Streamlit

1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.
```bash
cd ~/environment/workshop/labs/text
```
**Chỉ muốn chạy thử ứng dụng?**
{{%expand "Chạy lệnh này:" %}}
```bash
cd ~/environment/workshop/completed/text
```
Đến bước 2.
{{% /expand%}}

2. Chạy lệnh streamlit từ terminal.
```bash
streamlit run text_app.py --server.port 8080
```

Dùng tình năng xem trước của Cloud9, bỏ qua các URL của lệnh trước.

3. Trong AWS Cloud9, chọn **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

Trang web chạy thành công:

![App](/images/2-Bedrock/basic/B-1/3.png)

4. Chạy thử các lệnh khác nhau và xem kết quả.

   - `What is a good name for a product that provides large language models?`
   - `Why is the sky blue?`
   - `What is the capital of New Hampshire?`
   - `I am pleased to meet you. What is the sentiment of the previous statement?`

![App Results](/images/2-Bedrock/basic/B-1/4.png)

5. Tắt trang web trước của Cloud9. Vào terminal, bấm Control-C để thoát ứng dụng.