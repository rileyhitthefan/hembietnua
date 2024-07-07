---
title: "Lab B-4: Chatbot"
date: 2024-07-01T13:20:48+07:00
draft: false
weight: 4
chapter : false
---

Tạp chatbot đơn giản bằng **Amazon Bedrock, LangChain, và Streamlit**.

**Amazon Bedrock** (**LLMs** nói chung) không có khái niệm về trạng thái hoặc bộ nhớ. Lịch sử trò chuyện phải được theo dõi bên ngoài và trả lại vào model trong tin nhắn mới. Dùng lớp **ConversationSummaryBufferMemory** của LangChain có thể giúp theo dõi lịch sử. Lượng token sẽ bị hạn chế, vì thế cần cắt ngắn lịch sử để chừa đủ dung lượng nhận và trả lời lệnh từ người dùng.

**ConversationSummaryBufferMemory** hỗ trợ theo dõi tin nhắn gần đây và tóm tắt các tin cũ hơn.

So với phần [retrieval-augmented generation lab](LabB-3.md), các câu trả lời trong phần này không sử dụng bất kỳ tài liệu hỗ trợ nào. Vì thế, model có thể ảo giác và tự tạo câu trả lời không chính xác.

{{% notice note %}}
**Chỉ muốn chạy ứng dúng?**\
Đến mục [Chạy ứng dụng tạo sẵn.](#run-the-streamlit-app)
{{% /notice %}}

### Ứng dụng
Mẫu chatbot có thể dùng cho:
- Hội thoại tương tác đơn giản với người dùng, không yêu cầu kiến thức hoặc dữ liệu chuyên môn

### Kiến trúc

![Architecture](/images/2-Bedrock/basic/B-4/architecture.png)

1. Tương tác cũ được lưu trong lịch sử đoạn hội thoại.
2. Người dùng nhập tin nhắn mới.
3. Lấy lịch sử hội thoại từ vật thể nhớ trước khi nhận tin nhắn mới.
4. Gửi lịch sử hội thoại kèm với tin nhắn đến model.
5. Model gửi phản hồi đến người dùng.

Ứng dụng gồm 2 file: 1 file front-end Streamlist, 1 file cho các thự viện hỗ trợ để gọi API Bedrock.

## Create the library script
Tạo thư viện hỗ trỡ để kết nối front-end Streamlit đến back-end Bedrock.

1. Vào thư mục **workshop/labs/chatbot**, mở file **chatbot_lib.py**

![lib](/images/2-Bedrock/basic/B-4/1.png)

2. Thêm câu lệnh import.    
   - Dùng LangChain để gọi Bedrock và đọc biến môi trường.

```py
from langchain.memory import ConversationSummaryBufferMemory
from langchain_community.chat_models import BedrockChat
from langchain.chains import ConversationChain
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

4. Thêm hàm tạo vật thể nhớ bằng LangChain.

   - Dùng lớp **ConversationSummaryBufferMemory** để lưu trữ các tin nhắn gần nhất và tóm tắt các tin nhắn cũ hơn để duy trì hội thoại.

```py
def get_memory(): #create memory for this chat session
    
    #ConversationSummaryBufferMemory requires an LLM for summarizing older messages
    #this allows us to maintain the "big picture" of a long-running conversation
    llm = get_llm()
    
    memory = ConversationSummaryBufferMemory(llm=llm, max_token_limit=1024) #Maintains a summary of previous messages
    
    return memory
```

5. Thêm hàm gọi Bedrock.

   - Tìm trong các chỉ mục tạo ra từ input người dùng, thêm nội dung tốt nhất được nối với lệnh từ người dùng để gửi đến model.

```py
def get_chat_response(input_text, memory): #chat client function
    
    llm = get_llm()
    
    conversation_with_summary = ConversationChain( #create a chat client
        llm = llm, #using the Bedrock LLM
        memory = memory, #with the summarization memory
        verbose = True #print out some of the internal states of the chain while running
    )
    
    chat_response = conversation_with_summary.invoke(input_text) #pass the user message and summary to the model
    
    return chat_response['response']
```

6. Lưu file.

## Tạo giao diện người dùng của app Streamlit

1. Trong cùng thư mục với script thư viện, mở file **chatbot_app.py**

2. Thêm câu lệnh import.

   - Dùng các phần tử Streamlit và gọi các hàm từ script thư viện back-end.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
import chatbot_lib as glib #reference to local lib script
```

3. Thêm tiêu đề trang và file config.

```py
st.set_page_config(page_title="Chatbot") #HTML title
st.title("Chatbot") #page title
```

4. Thêm bộ nhớ LangChain vào bộ nhớ đệm của phiên chạy.

   - Dùng để duy trì hội thoại cho từng phiên riêng biệt.
   - Trong Streamlit, trạng thái phiên được theo dõi từ máy chủ. Nếu tab trình duyệt bị đóng hoặc ứng dụng bị dừng, phiên và lịch sử trò chuyện của sẽ mất. Trong ứng dụng thực tế, bạn sẽ muốn theo dõi lịch sử trò chuyện bằng dịch vụ như [Amazon ElastiCache](https://aws.amazon.com/blogs/database/solutions-for-building-modern-applications-with-amazon-elasticache-and-amazon-memorydb-for-redis/)  hoặc [Amazon DynamoDB](https://aws.amazon.com/dynamodb/).

```py
if 'memory' not in st.session_state: #see if the memory hasn't been created yet
    st.session_state.memory = glib.get_memory() #initialize the memory
```

5. Thêm UI lịch sử trò chuyện vào phiên.

```python
if 'chat_history' not in st.session_state: #see if the chat history hasn't been created yet
    st.session_state.chat_history = [] #initialize the chat history
```

6. Thêm vòng lập for để kết xuất lịch sử hội thoại dựa trên vật thể trạng thái phiên chat_history.

```py
#Re-render the chat history (Streamlit re-runs this script, so need this to preserve previous chat messages)
for message in st.session_state.chat_history: #loop through the chat history
    with st.chat_message(message["role"]): #renders a chat line for the given role, containing everything in the with block
        st.markdown(message["text"]) #display the chat content
```

7. Thêm phần tử input.  
   - Dùng lệnh if để xử lý input người dùng.

```python
input_text = st.chat_input("Chat with your bot here") #display a chat input box

if input_text: #run the code in this if block after the user submits a chat message
    
    with st.chat_message("user"): #display a user chat message
        st.markdown(input_text) #renders the user's latest message
    
    st.session_state.chat_history.append({"role":"user", "text":input_text}) #append the user's latest message to the chat history
    
    chat_response = glib.get_chat_response(input_text=input_text, memory=st.session_state.memory) #call the model through the supporting library
    
    with st.chat_message("assistant"): #display a bot chat message
        st.markdown(chat_response) #display bot's latest response
    
    st.session_state.chat_history.append({"role":"assistant", "text":chat_response}) #append the bot's latest message to the chat history
```
8. Lưu file.

## Chạy ứng dụng Streamlit

1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/chatbot
```

**Chỉ muốn chạy thử ứng dụng?**
{{%expand "Chạy lệnh này:" %}}
```bash
cd ~/environment/workshop/completed/chatbot
```
Đến bước 2.
{{% /expand%}}

2. Chạy lệnh streamlit từ terminal.

```bash
streamlit run chatbot_app.py --server.port 8080
```

Dùng tình năng xem trước của Cloud9, bỏ qua các URL của lệnh trước.

3. Trong AWS Cloud9, chọn **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

Trang web chạy thành công:

![App](/images/2-Bedrock/basic/B-4/3.png)

4. Chạy thử các lệnh khác nhau và xem kết quả.

   - `What is the first color of the rainbow?`
   - `What is the next one?`
   - `What is the first planet from the sun?`
   - `What is the next one?`

![App Result](/images/2-Bedrock/basic/B-4/4.png)

5. Tắt trang xem trước của Cloud9. Vào terminal, bấm Control-C để thoát ứng dụng.