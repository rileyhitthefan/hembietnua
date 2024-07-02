---
title: "Lab B-3: Retrieval-Augmented Generation"
date: 2024-07-01T13:20:46+07:00
draft: false
weight: 3
chapter : false
---

Xâu dựng ứng dụng hỏi đáp đơn giản bằng model **AI21 Jurassic 2**, **Titan Embeddings**, **LangChain**, and **Streamlit**.

Large language models có xu hướng **ảo giác**/**hallucination**, hay tự thêm chữ vào câu trả lời. Để có câu trả lời chính xác và nhất quán, cần đảm bảo model có các thông tin có sẵn để hỗ trợ phản hồi. Dùng phương pháp **Retrieval-Augmented Generation (RAG)** để thực hiện việc này.

Trong Retrieval-Augmented Generation, chúng ta đầu tiên đưa lệnh của người dùng vào data store, ví dụ như một truy vấn đến [Amazon Kendra](https://aws.amazon.com/kendra/). Cũng có thể dùng Amazon Titan Embeddings để tạo các biểu diễn số học cho câu lệnh và đưa vào vector database. Sau đó, nội dung liên quan nhất sẽ được lấy từ data store để hỗ trợ phản hồi của model ngôn ngữ.

Trong lab này, chúng ta dùng CSDL trên bộ nhớ [FAISS](https://github.com/facebookresearch/faiss) để miêu tả phương pháp RAG. Trên thức tế, một data store như Amazon Kendra hoặc [vector engine sử dụng cho Amazon OpenSearch Serverless](https://aws.amazon.com/opensearch-service/serverless-vector-engine/) sẽ tốt hơn.

{{% notice note %}}
**Chỉ muốn chạy ứng dúng?**\
Đến mục [Chạy ứng dụng tạo sẵn.](#run-the-streamlit-app)
{{% /notice %}}

### Trường hợp sử dụng 
Retrieval-Augmented Generation dùng cho:

- Hỏi và đáp, hỗ trợ bằng kiến thức hoặc dữ liệu chuyên môn
- Tìm kiếm thông minh

### Kiến trúc
![Architecture](/images/2-Bedrock/basic/B-3/architecture.png)

1. Tài liệu được phân ra thành các đoạn văn bản, đưa qua TItan Embeddings để nhúng thành vector lưu vào vector database.
2. Người dùng nhập câu hỏi
3. Câu hỏi được nhúng thành vector bằng Titan Embeddings, và được kết nối với câu trả lời gần giống nhất trong vector database.
4. Nội dung kết nối từ vector và câu hỏi gốc được đưa qua model ngôn ngữ để trả lại phản hồi tốt nhất.

Ứng dụng gồm 2 file: 1 file front-end Streamlist, 1 file cho các thự viện hỗ trợ để gọi API Bedrock.

## Tạo script thư viện
Tạo thư viện hỗ trỡ để kết nối front-end Streamlit đến back-end Bedrock.

1. Vào thư mục **workshop/labs/rag**, mở file **rag_lib.py**

![Architecture](/images/2-Bedrock/basic/B-3/1.png)

2. Thêm câu lệnh import.  
   - Dùng LangChain để gọi Bedrock tải file PDF, chỉ mục tài liệu.

```python
from langchain_community.embeddings import BedrockEmbeddings
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader
from langchain_community.llms import Bedrock
```

3. Thêm hàm gọi Bedrock.
   - Bao gồm các tham số suy đoán.

```python
def get_llm():
    
    model_kwargs = { #AI21
        "maxTokens": 1024, 
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

4. Thêm hàm tạo CSDL trên bộ nhớ

```python
def get_index(): #creates and returns an in-memory vector store to be used in the application
    
    embeddings = BedrockEmbeddings() #create a Titan Embeddings client
    
    pdf_path = "2022-Shareholder-Letter.pdf" #assumes local PDF file with this name

    loader = PyPDFLoader(file_path=pdf_path) #load the pdf file
    
    text_splitter = RecursiveCharacterTextSplitter( #create a text splitter
        separators=["\n\n", "\n", ".", " "], #split chunks at (1) paragraph, (2) line, (3) sentence, or (4) word, in that order
        chunk_size=1000, #divide into 1000-character chunks using the separators above
        chunk_overlap=100 #number of characters that can overlap with previous chunk
    )
    
    index_creator = VectorstoreIndexCreator( #create a vector store factory
        vectorstore_cls=FAISS, #use an in-memory vector store for demo purposes
        embedding=embeddings, #use Titan embeddings
        text_splitter=text_splitter, #use the recursive text splitter
    )
    
    index_from_loader = index_creator.from_loaders([loader]) #create an vector store index from the loaded PDF
    
    return index_from_loader #return the index to be cached by the client app
```

5. Thêm hàm gọi Bedrock.

   - Tìm trong các chỉ mục tạo ra từ input người dùng, thêm nội dung tốt nhất được nối với lệnh từ người dùng để gửi đến model.

```py
def get_rag_response(index, question): #rag client function
    
    llm = get_llm()
    
    response_text = index.query(question=question, llm=llm) #search against the in-memory index, stuff results into a prompt and send to the llm
    
    return response_text
```

6. Lưu file.

## Tạo giao diện người dùng của app Streamlit

1. Trong cùng thư mục với script thư viện, mở file **rag_app.py**

2. Thêm câu lệnh import.

   - Dùng các phần tử Streamlit và gọi các hàm từ script thư viện back-end.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import rag_lib as glib #reference to local lib script
```

3. Thêm tiêu đề trang và file config.

```py
st.set_page_config(page_title="Retrieval-Augmented Generation") #HTML title
st.title("Retrieval-Augmented Generation") #page title
```

4. Thêm chỉ mục vector vào bộ nhớ đệm của phiên chạy

   - Tạo CSDL trên bộ nhớ riêng cho từng phiên chạy khác nhau

```py
if 'vector_index' not in st.session_state: #see if the vector index hasn't been created yet
    with st.spinner("Indexing document..."): #show a spinner while the code in this with block runs
        st.session_state.vector_index = glib.get_index() #retrieve the index through the supporting library and store in the app's session cache
```

5. Thêm phần tử input.  
   - Tạo một hộp nhập đoạn văn bản và nút để gửi câu lệnh từ người dùng đến Bedrock.

```py
input_text = st.text_area("Input text", label_visibility="collapsed") #display a multiline text box with no label
go_button = st.button("Go", type="primary") #display a primary button
```

6. Thêm phần tử output.   
   - Dùng lệnh if để xử lý hoạt động nút chọn. Hiển thị con xoay trong lúc chờ, và viết lại output lên trang web sau khi trả kết quả.

```py
if go_button: #code in this if block will be run when the button is clicked
    
    with st.spinner("Working..."): #show a spinner while the code in this with block runs
        response_content = glib.get_rag_response(index=st.session_state.vector_index, question=input_text) #call the model through the supporting library
        
        st.write(response_content) #display the response content

```

7. Lưu file

## Chạy ứng dụng Streamlit

1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/rag
```

**Chỉ muốn chạy thử ứng dụng?**
{{%expand "Chạy lệnh này:" %}}
```bash
cd ~/environment/workshop/completed/rag
```
Đến bước 2.
{{% /expand%}}

2. Chạy lệnh streamlit từ terminal.

```bash
streamlit run rag_app.py --server.port 8080
```

Dùng tình năng xem trước của Cloud9, bỏ qua các URL của lệnh trước.

3. Trong AWS Cloud9, chọn **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

Trang web chạy thành công:

![App](/images/2-Bedrock/basic/B-3/3.png)

4. Chạy thử các lệnh khác nhau và xem kết quả.

   - `What are some of the current strategic initiatives for the company?`
   - `What is the company's strategy for generative AI?`
   - `What are the key growth drivers for the company?`
   - `¿Cuáles son algunas de las iniciativas estratégicas actuales de la empresa?`
   - `Quelle est la stratégie de l'entreprise en matière d'IA générative ?`
   - `Was sind die wichtigsten Wachstumstreiber für das Unternehmen?`

![App](/images/2-Bedrock/basic/B-3/4.png)

5. Tắt trang xem trước của Cloud9. Vào terminal, bấm Control-C để thoát ứng dụng.