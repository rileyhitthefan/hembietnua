---
title: "Lab I-1: Chatbot với RAG"
date: 2024-07-01T16:11:51+07:00
draft: false
weight: 1
chapter : false
---

**Amazon Bedrock** (**LLMs** nói chung) không có khái niệm về trạng thái hoặc bộ nhớ. Lịch sử trò chuyện phải được theo dõi bên ngoài và trả lại vào model trong tin nhắn mới. Dùng lớp **ConversationSummaryBufferMemory** của LangChain có thể giúp theo dõi lịch sử. Lượng token sẽ bị hạn chế, vì thế cần cắt ngắn lịch sử để chừa đủ dung lượng nhận và trả lời lệnh từ người dùng.

**ConversationSummaryBufferMemory** hỗ trợ theo dõi tin nhắn gần đây và tóm tắt các tin cũ hơn.

Chúng ta cũng có thể củng cố dữ liền nền của model với các kiến thức bên ngoài qua **Retrieval-Augmented Generation (RAG)**. Dùng lớp **ConversationlRetrievalChain** của LangChain để tích hợp tính năng chatbot và RAG trong một tín hiệu gọi đến LangChain.

Trong lab này, chúng ta sẽ tạo chatbot hỗ trợ bởi RAG bằng model **Anthropic Claude, Amazon Titan Embeddings, LangChain và Streamlit**. Trong lab này, chúng ta dùng CSDL trên bộ nhớ [FAISS](https://github.com/facebookresearch/faiss) để miêu tả phương pháp RAG. Trên thức tế, một data store như Amazon Kendra hoặc [vector engine sử dụng cho Amazon OpenSearch Serverless](https://aws.amazon.com/opensearch-service/serverless-vector-engine/) sẽ tốt hơn.

{{% notice note %}}
**Chỉ muốn chạy ứng dúng?**\
Đến mục [Chạy ứng dụng tạo sẵn.](#run-the-streamlit-app)
{{% /notice %}}

### Trường hợp sử dụng 
Chatbot với RAG dùng cho:

- Hỏi đáp tương tác đơn giản với người dùng cần thêm kiến thức hoặc dữ liệu chuyên môn

### Kiến trúc
![architecture](/images/2-Bedrock/text/I-1/architecture.png)

1. Tương tác cũ được lưu trong lịch sử đoạn hội thoại.
2. Người dùng nhập tin nhắn mới.
3. Lấy lịch sử hội thoại từ vật thể nhớ trước khi nhận tin nhắn mới.
4. Câu hỏi được nhúng thành vector bằng Titan Embeddings, và được kết nối với câu trả lời gần giống nhất trong vector database.
5. Nội dung lịch sử trò chuyện, kiến thức từ RAG và tin nhắn mới được gửi đến model.
6. Model gửi phản hồi đến người dùng.

Ứng dụng gồm 2 file: 1 file front-end Streamlist, 1 file cho các thự viện hỗ trợ để gọi API Bedrock.

## Tạo script thư viện
Tạo thư viện hỗ trỡ để kết nối front-end Streamlit đến back-end Bedrock.

1. Vào thư mục **workshop/labs/rag_chatbot**, mở file **rag_chatbot_lib.py**

![lib](/images/2-Bedrock/text/I-1/1.png)

2. Thêm câu lệnh import.    
   - Dùng LangChain để gọi Bedrock và đọc biến môi trường.

```py
from langchain.memory import ConversationBufferWindowMemory
from langchain_community.chat_models import BedrockChat
from langchain.chains import ConversationalRetrievalChain

from langchain_community.embeddings import BedrockEmbeddings
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader
```

3. Thêm hàm gọi client Bedrock bằng LangChain.  
   - Bao gồm các tham số suy đoán sử dụng cho chatbot.

```py
def get_llm():
        
    model_kwargs = { #anthropic
        "max_tokens": 512,
        "temperature": 0, 
        "top_k": 250, 
        "top_p": 1, 
        "stop_sequences": ["\n\nHuman:"] 
    }
    
    llm = BedrockChat(
        model_id="anthropic.claude-3-sonnet-20240229-v1:0", #set the foundation model
        model_kwargs=model_kwargs) #configure the inference parameters
    
    return llm
```

4. Thêm hàm tạo CSDL trên bộ nhớ

```py
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

5. Thêm hàm tạo vật thể nhớ bằng LangChain.

   - Dùng lớp **ConversationSummaryBufferMemory** để lưu trữ các tin nhắn gần nhất và tóm tắt các tin nhắn cũ hơn để duy trì hội thoại.

```py
def get_memory(): #create memory for this chat session
    
    memory = ConversationBufferWindowMemory(memory_key="chat_history", return_messages=True) #Maintains a history of previous messages
    
    return memory
```

6. Thêm hàm gọi Bedrock.

   - Tìm trong các chỉ mục tạo ra từ input người dùng, thêm nội dung tốt nhất được nối với lệnh từ người dùng để gửi đến model.

```py
def get_rag_chat_response(input_text, memory, index): #chat client function
    
    llm = get_llm()
    
    conversation_with_retrieval = ConversationalRetrievalChain.from_llm(llm, index.vectorstore.as_retriever(), memory=memory, verbose=True)
    
    chat_response = conversation_with_retrieval.invoke({"question": input_text}) #pass the user message and summary to the model
    
    return chat_response['answer']
```

7. Lưu file.

## Tạo giao diện người dùng của app Streamlit

1. Trong cùng thư mục với script thư viện, mở file **rag_chatbot_app.py**

2. Thêm câu lệnh import.

   - Dùng các phần tử Streamlit và gọi các hàm từ script thư viện back-end.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
import rag_chatbot_lib as glib #reference to local lib script
```

3. Thêm tiêu đề trang và file config.

```py
st.set_page_config(page_title="RAG Chatbot") #HTML title
st.title("RAG Chatbot") #page title
```

4. Thêm bộ nhớ LangChain vào bộ nhớ đệm của phiên chạy.

   - Dùng để duy trì hội thoại cho từng phiên riêng biệt.
   - Trong Streamlit, trạng thái phiên được theo dõi từ máy chủ. Nếu tab trình duyệt bị đóng hoặc ứng dụng bị dừng, phiên và lịch sử trò chuyện của sẽ mất. Trong ứng dụng thực tế, bạn sẽ muốn theo dõi lịch sử trò chuyện bằng dịch vụ như [Amazon ElastiCache](https://aws.amazon.com/blogs/database/solutions-for-building-modern-applications-with-amazon-elasticache-and-amazon-memorydb-for-redis/)  hoặc [Amazon DynamoDB](https://aws.amazon.com/dynamodb/).

```py
if 'memory' not in st.session_state: #see if the memory hasn't been created yet
    st.session_state.memory = glib.get_memory() #initialize the memory
```

5. Thêm UI lịch sử trò chuyện vào phiên.

```py
if 'chat_history' not in st.session_state: #see if the chat history hasn't been created yet
    st.session_state.chat_history = [] #initialize the chat history
```

6. Thêm chỉ mục vector vào bộ nhớ đệm.

```py
if 'vector_index' not in st.session_state: #see if the vector index hasn't been created yet
    with st.spinner("Indexing document..."): #show a spinner while the code in this with block runs
        st.session_state.vector_index = glib.get_index() #retrieve the index through the supporting library and store in the app's session cache
```

7. Thêm vòng lập for để kết xuất lịch sử hội thoại dựa trên vật thể trạng thái phiên chat_history.

```py
#Re-render the chat history (Streamlit re-runs this script, so need this to preserve previous chat messages)
for message in st.session_state.chat_history: #loop through the chat history
    with st.chat_message(message["role"]): #renders a chat line for the given role, containing everything in the with block
        st.markdown(message["text"]) #display the chat content
```

8. Thêm phần tử input.  
   - Dùng lệnh if để xử lý input người dùng.

```py
input_text = st.chat_input("Chat with your bot here") #display a chat input box

if input_text: #run the code in this if block after the user submits a chat message
    
    with st.chat_message("user"): #display a user chat message
        st.markdown(input_text) #renders the user's latest message
    
    st.session_state.chat_history.append({"role":"user", "text":input_text}) #append the user's latest message to the chat history
    
    chat_response = glib.get_rag_chat_response(input_text=input_text, memory=st.session_state.memory, index=st.session_state.vector_index,) #call the model through the supporting library
    
    with st.chat_message("assistant"): #display a bot chat message
        st.markdown(chat_response) #display bot's latest response
    
    st.session_state.chat_history.append({"role":"assistant", "text":chat_response}) #append the bot's latest message to the chat history
```

9. Lưu file.

# Chạy ứng dụng Streamlit

1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/rag_chatbot
```

**Chỉ muốn chạy thử ứng dụng?**
{{%expand "Chạy lệnh này:" %}}
```bash
cd ~/environment/workshop/completed/rag_chatbot
```
Đến bước 2.
{{% /expand%}}

2. Chạy lệnh streamlit từ terminal.

```bash
streamlit run rag_chatbot_app.py --server.port 8080
```

Dùng tình năng xem trước của Cloud9, bỏ qua các URL của lệnh trước.

3. Trong AWS Cloud9, chọn **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

Trang web chạy thành công:

![App](/images/2-Bedrock/text/I-1/2.png)

4. Chạy thử các lệnh khác nhau và xem kết quả.

   - `Who is the CEO?`
   - `When did he start?`
   - `What is the company's generative AI strategy?`

![App Results](/images/2-Bedrock/text/I-1/3.png)

5. Tắt trang xem trước của Cloud9. Vào terminal, bấm Control-C để thoát ứng dụng.