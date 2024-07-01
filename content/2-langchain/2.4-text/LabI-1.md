---
title: "Lab I-1: Chatbot with RAG"
date: 2024-07-01T16:11:51+07:00
draft: false
weight: 1
chapter : false
---

Amazon Bedrock (and LLMs in general) don’t have any concept of state or memory. Any chat history has to be tracked externally and then passed into the model with each new message. We are using LangChain's **ConversationBufferWindowMemory** class to track chat history. Since there is a limit on the number of tokens that can be processed by the model, we need to prune the chat history so there is enough space left to handle the user's message and the model's responses. ConversationBufferWindowMemory supports this by tracking the most recent messages.

We also want to supplement the model's underlying data with external knowledge through **Retrieval-Augmented Generation (RAG)**. We'll use LangChain's **ConversationalRetrievalChain** class to combine chatbot and RAG functionality in a single call to LangChain.

In this lab, we will build a chatbot supported by Retrieval-Augmented Generation (RAG). We'll use **Anthropic Claude**, **Amazon Titan Embeddings**, **LangChain**, and **Streamlit**. We will use an in-memory **FAISS** database to demonstrate the RAG pattern. In a real-world scenario, you will most likely want to use a persistent data store like Amazon Kendra or the [vector engine for Amazon OpenSearch Serverless](https://aws.amazon.com/opensearch-service/serverless-vector-engine/).

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The chatbot with RAG pattern is good for the following use cases:
- Simple interactive user conversation, supported by specialized knowledge or data

### Architecture
![architecture](/images/2-Bedrock/text/I-1/architecture.png)

1. Past interactions are tracked in the chat memory object.
2. The user enters a new message.
3. The chat history is retrieved from the memory object and added before the new message.
4. The question is converted to a vector using Amazon Titan Embeddings, then matched to the closest vectors in the vector database.
5. The combined history, knowledge, and new message are sent to the model.
6. The model's response is displayed to the user.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/rag_chatbot** folder, and open the file **rag_chatbot_lib.py**

![lib](/images/2-Bedrock/text/I-1/1.png)

2. Add the import statements.
   - These statements allow us to use LangChain to call Bedrock and read environment variables.

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

3. Add a function to create a Bedrock LangChain client.
   - This includes the inference parameters we want to use for the chatbot.

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

4. Add the function to create the in-memory vector store.
   - See the in-line comments in the code below for more details.

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

5. Add a function to initialize a LangChain memory object.
   - In this case, we are using the ConversationBufferWindowMemory class. This allows us to track the most recent messages and summarize older messages so that the chat context is maintained over a long conversation.

```py
def get_memory(): #create memory for this chat session
    
    memory = ConversationBufferWindowMemory(memory_key="chat_history", return_messages=True) #Maintains a history of previous messages
    
    return memory
```

6. Add this function to call Bedrock.
   - We're creating a function we can call from the Streamlit front end application. This function creates a Bedrock client with LangChain, then passes the input content to Bedrock.

```py
def get_rag_chat_response(input_text, memory, index): #chat client function
    
    llm = get_llm()
    
    conversation_with_retrieval = ConversationalRetrievalChain.from_llm(llm, index.vectorstore.as_retriever(), memory=memory, verbose=True)
    
    chat_response = conversation_with_retrieval.invoke({"question": input_text}) #pass the user message and summary to the model
    
    return chat_response['answer']
```

7. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **rag_chatbot_app.py**

2. Add the import statements.
   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
import rag_chatbot_lib as glib #reference to local lib script
```

3. Add the page title and configuration.
   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```py
st.set_page_config(page_title="RAG Chatbot") #HTML title
st.title("RAG Chatbot") #page title
```

4. Add the LangChain memory to the session cache.
   - This allows us to maintain a unique chat memory per user session. Otherwise, the chatbot won't be able to remember past messages with the user.
   - In Streamlit, session state is tracked server-side. If the browser tab is closed, or the application is stopped, the session and its chat history will be lost. In a real-world application, you would want to track the chat history with a service like [Amazon ElastiCache](https://aws.amazon.com/blogs/database/solutions-for-building-modern-applications-with-amazon-elasticache-and-amazon-memorydb-for-redis/) or [Amazon DynamoDB](https://aws.amazon.com/dynamodb/).

```py
if 'memory' not in st.session_state: #see if the memory hasn't been created yet
    st.session_state.memory = glib.get_memory() #initialize the memory
```

5. Add the UI chat history to the session cache.
   - This allows us to re-render the chat history to the UI as the Streamlit app is re-run with each user interaction. Otherwise, the old messages will disappear from the user interface with each new chat message.

```py
if 'chat_history' not in st.session_state: #see if the chat history hasn't been created yet
    st.session_state.chat_history = [] #initialize the chat history
```

6. Add the vector index to the session cache.
   - This allows us to maintain an in-memory vector database per user session.

```py
if 'vector_index' not in st.session_state: #see if the vector index hasn't been created yet
    with st.spinner("Indexing document..."): #show a spinner while the code in this with block runs
        st.session_state.vector_index = glib.get_index() #retrieve the index through the supporting library and store in the app's session cache
```

7. Add the for loop to render previous chat messages.
   - Re-render previous messages based on the chat_history session state object.

```py
#Re-render the chat history (Streamlit re-runs this script, so need this to preserve previous chat messages)
for message in st.session_state.chat_history: #loop through the chat history
    with st.chat_message(message["role"]): #renders a chat line for the given role, containing everything in the with block
        st.markdown(message["text"]) #display the chat content
```

8. Add the input elements.
   - We use the `if` block below to handle the user input. See the in-line comments below for more details.

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

9. Save the file.

## Run the Streamlit app
1. Select the bash terminal in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/rag_chatbot
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/rag_chatbot
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run rag_chatbot_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![App](/images/2-Bedrock/text/I-1/2.png)

4. Try out some prompts and see the results.

   - `Who is the CEO?`
   - `When did he start?`
   - `What is the company's generative AI strategy?`

![App Results](/images/2-Bedrock/text/I-1/3.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Ctrl-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a RAG chatbot with Bedrock, LangChain, and Streamlit!
{{% /notice %}}