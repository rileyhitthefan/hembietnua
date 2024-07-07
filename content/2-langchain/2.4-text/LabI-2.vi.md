---
title: "Lab I-2: Tóm tắt tài liệu"
date: 2024-07-01T16:11:54+07:00
draft: false
weight: 2
chapter : false
---
Tạo ứng dụng tóm tắt tài liệu đơn giản bằng **Amazon Bedrock, LangChain, và Streamlit**.

LangChain sử dụng hàm **map-reduce summarization** cho phép người dùng xử lý các nội dung vượt quá số lượng token được định sẵn. Hàm này chia nhỏ tài liệu thành nhiều mảnh, và tóm tắt nội dung của từng mảnh nhỏ đó.

{{% notice info %}}
Bài lab có thể không chạy được nếu vượt quá giới hạn throttling.
{{% /notice %}}

{{% notice note %}}
**Chỉ muốn chạy ứng dúng?**\
Đến mục [Chạy ứng dụng tạo sẵn.](#run-the-streamlit-app)
{{% /notice %}}

### Ứng dụng
Mẫu tóm tắt dùng map-reduce phù hợp cho việc:
- Tóm tắt các tài liệu dài
- Tóm tắt bản ghi cuộc gọi
- Tóm tắt lịch sử người dùng

### Kiến trúc

![architecture](/images/2-Bedrock/text/I-2/architecture.png)

The Map-Reduce pattern involves the following steps:

1. Chia tài liệu thành các đoạn nhỏ
2. Tạo tóm tắt trung gian bằng các tóm tắt các đoạn đã được chia nhỏ
3. Tóm tắt các đoạn trung gian để tạo đoạn tóm tắt hoàn chỉnh

Ứng dụng gồm 2 file: 1 file front-end Streamlist, 1 file cho các thự viện hỗ trợ để gọi API Bedrock.

## Tạo script thư viện 
Tạo thư viện hỗ trỡ để kết nối front-end Streamlit đến back-end Bedrock.

1. Vào thư mục **workshop/labs/summarization**, mở file **summarization_lib.py**

![architecture](/images/2-Bedrock/text/I-2/lib.png)

2. Thêm câu lệnh import.    
   - Dùng LangChain để gọi Bedrock, tải file PDF và chia nhỏ tài liệu

```python
from langchain.prompts import PromptTemplate
from langchain_community.llms import Bedrock
from langchain.chains.summarize import load_summarize_chain
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader
```

3. Thêm hàm gọi client Bedrock bằng LangChain.  
   - Bao gồm các tham số suy đoán muốn sử dụng.

```python
def get_llm():
    
    model_kwargs = { #AI21
        "maxTokens": 8000, 
        "temperature": 0, 
        "topP": 0.5, 
        "stopSequences": [], 
        "countPenalty": {"scale": 0 }, 
        "presencePenalty": {"scale": 0 }, 
        "frequencyPenalty": {"scale": 0 } 
    }
    
    llm = Bedrock(
        model_id="ai21.j2-ultra-v1", #set the foundation model
        model_kwargs=model_kwargs) #configure the inference parameters
    
    return llm
```

4. Thêm hàm để tạo các mảnh tài liệu.

   - Tài liệu có thể được chia theo các đoạn, dòng, câu hoặc các chữ.

```python
pdf_path = "2022-Shareholder-Letter.pdf"

def get_docs():
    
    loader = PyPDFLoader(file_path=pdf_path)
    documents = loader.load()
    text_splitter = RecursiveCharacterTextSplitter(
        separators=["\n\n", "\n", ".", " "], chunk_size=4000, chunk_overlap=100 
    )
    docs = text_splitter.split_documents(documents=documents)
    
    return docs
```

5. Thêm hàm gọi Bedrock.
   
   - Tạo lệnh để map và reduce để tạo các khoá, sau đó đưa tài liệu qua hàm này để tạo bản tóm tắt hoàn thiện.

```python
def get_summary(return_intermediate_steps=False):
    
    map_prompt_template = "{text}\n\nWrite a few sentences summarizing the above:"
    map_prompt = PromptTemplate(template=map_prompt_template, input_variables=["text"])
    
    combine_prompt_template = "{text}\n\nWrite a detailed analysis of the above:"
    combine_prompt = PromptTemplate(template=combine_prompt_template, input_variables=["text"])
    
    
    llm = get_llm()
    docs = get_docs()
    
    chain = load_summarize_chain(llm, chain_type="map_reduce", map_prompt=map_prompt, combine_prompt=combine_prompt, return_intermediate_steps=return_intermediate_steps)
    
    if return_intermediate_steps:
        return chain.invoke({"input_documents": docs}, return_only_outputs=True)
    else:
        return chain.invoke(docs, return_only_outputs=True)
```

6. Lưu file.

## Tạo giao diện người dùng của app Streamlit

1. Trong cùng thư mục với script thư viện, mở file **summarization_app.py**

2. Thêm câu lệnh import.

   - Dùng các phần tử Streamlit và gọi các hàm từ script thư viện back-end.

```py
import streamlit as st
import summarization_lib as glib
```

3. Thêm tiêu đề trang và file config.

```python
st.set_page_config(page_title="Document Summarization")
st.title("Document Summarization")
```

4. Thêm các thành phần tóm tắt  
   - Phần này sẽ không được biểu diễn cho đến khi tình trạng của phiên -- `has_document` được thiết lập.
   - Chúng ta sẽ tạo checkbox và nút để nhận lệnh người dùng để gứi đến Bedrock. Checkbox "Return intermediate steps" cho người dùng xem các bản tóm tắt trung gian.
   - Dùng lệnh if để xử lý hoạt động nút chọn. Hiển thị con xoay trong lúc chờ, và viết lại output lên trang web sau khi trả kết quả.

```python
return_intermediate_steps = st.checkbox("Return intermediate steps", value=True)
summarize_button = st.button("Summarize", type="primary")


if summarize_button:
    st.subheader("Combined summary")

    with st.spinner("Running..."):
        response_content = glib.get_summary(return_intermediate_steps=return_intermediate_steps)


    if return_intermediate_steps:

        st.write(response_content["output_text"])

        st.subheader("Section summaries")

        for step in response_content["intermediate_steps"]:
            st.write(step)
            st.markdown("---")

    else:
        st.write(response_content["output_text"])
```

5. Lưu file.

## Chạy ứng dụng Streamlit

1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/summarization
```

**Chỉ muốn chạy thử ứng dụng?**
{{%expand "Chạy lệnh này:" %}}
```bash
cd ~/environment/workshop/completed/summarization
```
Đến bước 2.
{{% /expand%}}

2. Chạy lệnh streamlit từ terminal.

```python
streamlit run summarization_app.py --server.port 8080
```

Dùng tình năng xem trước của Cloud9, bỏ qua các URL của lệnh trước.

3. Trong AWS Cloud9, chọn **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

Trang web chạy thành công:

![App](/images/2-Bedrock/text/I-2/1.png)

4. Chọn nút tóm tắt
   - Có thể tốn khoảng 90s.
   - Sau khi chạy thành công, bản tóm tắt hoàn thiện sẽ được biểu diễn, cùng với các bản tóm tắt trung gian nếu bạn chọn checkbox tương ứng.

![App Result](/images/2-Bedrock/text/I-2/2.png)

5. Tắt trang xem trước của Cloud9. Vào terminal, bấm Control-C để thoát ứng dụng.