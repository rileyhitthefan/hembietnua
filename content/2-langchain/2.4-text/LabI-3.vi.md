---
title: "Lab I-3: Dòng phản hồi"
date: 2024-07-01T16:11:56+07:00
draft: false
weight: 3
chapter : false
---

Tạo một ứng dụng với dòng phản hồi (response streaming) bằng Bedrock, LangChain và Streamlit.

Dòng phản hồi hữu ích nếu bạn muốn trả về nội dung phản hồi lập tức cho người dùng cuối thông qua việc biểu diễn output theo một khoảng thời gian, thay vì phải chờ cho đến khi output hoàn tất.

{{% notice note %}}
**Chỉ muốn chạy ứng dúng?**\
Đến mục [Chạy ứng dụng tạo sẵn.](#run-the-streamlit-app)
{{% /notice %}}

### Ứng dụng
Mẫu dòng phản hồi phù hợp cho việc: 
- Giúp người dùng nhận được (1 phần) câu trả lời mong muốn ngay lập tức mà không phải chờ phản hồi hoàn tất.

### Kiến trúc
![architecture](/images/2-Bedrock/text/I-3/architecture.png)

Từ góc nhìn kiến trúc, dòng phản hồi tương đương với dạng chuyển văn bản thành văn bản. Chúng ta chỉ cần thêm một hàm có thể lập tức xủ lý phản hồi từ model.

Phản hồi dòng được trả về dưới dạng các mảnh JSON, từ đó bạn có thể tách văn bản ra khỏi các JSON để gửi đến người dùng cuối.

Ứng dụng gồm 2 file: 1 file front-end Streamlist, 1 file cho các thự viện hỗ trợ để gọi API Bedrock.

## Tạo script thư viện
Tạo thư viện hỗ trỡ để kết nối front-end Streamlit đến back-end Bedrock.

1. Vào thư mục **workshop/labs/streaming**, mở file **streaming_lib.py**

![lib](/images/2-Bedrock/text/I-3/1.png)

2. Thêm câu lệnh import.    
   - Dùng LangChain để gọi Bedrock và xử lý dòng output.

```py
from langchain.chains import ConversationChain
from langchain_community.llms import Bedrock
```

3. Thêm hàm gọi client Bedrock bằng LangChain, cho phép trả về theo dòng

```python
def get_llm(streaming_callback):
    model_kwargs = {
        "max_tokens": 4000,
        "temperature": 0,
        "p": 0.01,
        "k": 0,
        "stop_sequences": [],
        "return_likelihoods": "NONE",
        "stream": True
    }
    
    llm = Bedrock(
        model_id="cohere.command-text-v14",
        model_kwargs=model_kwargs,
        streaming=True,
        callbacks=[streaming_callback],
    )
    
    return llm
```

4. Thêm hàm xử lý dòng phản hồi.
   - Hàm này gọi Bedrock theo phương thức streaming. Các đoạn phản hồi được đưa qua phương thức gọi về được cung cấp.
   
```python
def get_streaming_response(prompt, streaming_callback):
    conversation_with_summary = ConversationChain(
        llm=get_llm(streaming_callback)
    )
    return conversation_with_summary.predict(input=prompt)
```

5. Lưu file.

## Tạo giao diện người dùng của app Streamlit

1. Trong cùng thư mục với script thư viện, mở file **streaming_app.py**

2. Thêm câu lệnh import.

   - Dùng các phần tử Streamlit và gọi các hàm từ script thư viện back-end.

```python
import streaming_lib as glib  # reference to local lib script
import streamlit as st
from langchain_community.callbacks.streamlit import StreamlitCallbackHandler
```

3. Thêm tiêu đề trang và file config.

```python
st.set_page_config(page_title="Response Streaming")  # HTML title
st.title("Response Streaming")  # page title
```

4. Thêm phần tử input
   - Tạo một hộp nhập đoạn văn bản và nút để gửi câu lệnh từ người dùng đến Bedrock.

```python
input_text = st.text_area("Input text", label_visibility="collapsed")
go_button = st.button("Go", type="primary")  # display a primary button
```

5. Thêm phần tử output.
   - Dùng lệnh if để xử lý hoạt động nút chọn.
   - Tạo một container streamlit trống và đưa vào đối tượng `StreamlitCallbackHandler` để có thể biểu diễn output trong lúc xuất kết quả.
   - Đưa `StreamlitCallbackHandler` vào hàm back-end để xử lý phản hồi ngay khi các mảnh streaming được model trả về.

```python
if go_button:  # code in this if block will be run when the button is clicked
    #use an empty container for streaming output
    st_callback = StreamlitCallbackHandler(st.container())
    streaming_response = glib.get_streaming_response(prompt=input_text, streaming_callback=st_callback)
```

6. Lưu file

## Chạy ứng dụng Streamlit

1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/streaming
```

**Chỉ muốn chạy thử ứng dụng?**
{{%expand "Chạy lệnh này:" %}}
```bash
cd ~/environment/workshop/completed/streaming
```
Đến bước 2.
{{% /expand%}}

2. Chạy lệnh streamlit từ terminal.

```bash
streamlit run streaming_app.py --server.port 8080
```

Dùng tình năng xem trước của Cloud9, bỏ qua các URL của lệnh trước.

3. Trong AWS Cloud9, chọn **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

Trang web chạy thành công:

![app](/images/2-Bedrock/text/I-3/2.png)

4. Chạy thử các lệnh khác nhau và xem kết quả.
   - `Write a story about two cats that go on an adventure:`

![app](/images/2-Bedrock/text/I-3/3.gif)

5. Tắt trang xem trước của Cloud9. Vào terminal, bấm Control-C để thoát ứng dụng.