---
title: "Lab B-3: Retrieval-Augmented Generation"
date: 2024-07-01T13:20:46+07:00
draft: false
weight: 3
chapter : false
---

In this lab, we will build a simple question & answer application with **AI21 Jurassic 2**, **Titan Embeddings**, **LangChain**, and **Streamlit**.

Large language models are prone to **hallucination**, which is just a fancy word for making up a response. To correctly and consistently answer questions, we need to ensure that the model has real information available to support its responses. We use the **Retrieval-Augmented Generation (RAG)** pattern to make this happen.

With Retrieval-Augmented Generation, we first pass a user's prompt to a data store. This might be in the form of a query to [Amazon Kendra](https://aws.amazon.com/kendra/). We could also create a numerical representation of the prompt using Amazon Titan Embeddings to pass to a vector database. We then retrieve the most relevant content from the data store to support the large language model's response.

In this lab, we will use an in-memory [FAISS](https://github.com/facebookresearch/faiss) database to demonstrate the RAG pattern. In a real-world scenario, you will most likely want to use a persistent data store like Amazon Kendra or the [vector engine for Amazon OpenSearch Serverless](https://aws.amazon.com/opensearch-service/serverless-vector-engine/).

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The Retrieval-Augmented Generation pattern is good for the following use cases:

- Question & answer, supported by specialized knowledge or data
- Intelligent search

### Architecture
![Architecture](/images/2-Bedrock/basic/B-3/architecture.png)

1. A document is broken up into chunks of text. The chunks are passed to Titan Embeddings to be converted to vectors. The vectors are then saved to the vector database.
2. The user submits a question.
3. The question is converted to a vector using Amazon Titan Embeddings, then matched to the closest vectors in the vector database.
4. The combined content from the matching vectors + the original question are then passed to the large language model to get the best answer.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/rag** folder, and open the file **rag_lib.py**

![Architecture](/images/2-Bedrock/basic/B-3/1.png)

2. Add the import statements.

   - These statements allow us to use LangChain to load a PDF file, index the document, and call Bedrock.

```python
from langchain_community.embeddings import BedrockEmbeddings
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader
from langchain_community.llms import Bedrock
```

3. Add a function to create a Bedrock LangChain client.
   - This includes the inference parameters we want to use.

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

4. Add the function to create the in-memory vector store.

   - See the in-line comments in the code below for more details.

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

5. Add this function to call Bedrock.

   - This code searches the previously created index based on the user's input, adds the best matches to a prompt along with the user's text, and then sends the combined prompt to the model.

```py
def get_rag_response(index, question): #rag client function
    
    llm = get_llm()
    
    response_text = index.query(question=question, llm=llm) #search against the in-memory index, stuff results into a prompt and send to the llm
    
    return response_text
```

6. Save the file.

## Create the Streamlit front-end app

1. In the same folder as your lib file, open the file **rag_app.py**

2. Add the import statements.
   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import rag_lib as glib #reference to local lib script
```

3. Add the page title and configuration.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```py
st.set_page_config(page_title="Retrieval-Augmented Generation") #HTML title
st.title("Retrieval-Augmented Generation") #page title
```

4. Add the vector index to the session cache.

   - This allows us to maintain an in-memory vector database per user session.

```py
if 'vector_index' not in st.session_state: #see if the vector index hasn't been created yet
    with st.spinner("Indexing document..."): #show a spinner while the code in this with block runs
        st.session_state.vector_index = glib.get_index() #retrieve the index through the supporting library and store in the app's session cache
```

5. Add the input elements.

   - We are creating a multiline text box and button to get the user's prompt and send it to Bedrock.

```py
input_text = st.text_area("Input text", label_visibility="collapsed") #display a multiline text box with no label
go_button = st.button("Go", type="primary") #display a primary button
```

6. Add the output elements.

   - We use the if block below to handle the button click. We display a spinner while the backing function is called, then write the output to the web page.

```py
if go_button: #code in this if block will be run when the button is clicked
    
    with st.spinner("Working..."): #show a spinner while the code in this with block runs
        response_content = glib.get_rag_response(index=st.session_state.vector_index, question=input_text) #call the model through the supporting library
        
        st.write(response_content) #display the response content

```

7. Save the file

## Run the Streamlit app

1. Select the bash terminal in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/rag
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/rag
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run rag_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![App](/images/2-Bedrock/basic/B-3/3.png)

4. Try out some prompts and see the results:

   - `What are some of the current strategic initiatives for the company?`
   - `What is the company's strategy for generative AI?`
   - `What are the key growth drivers for the company?`
   - `¿Cuáles son algunas de las iniciativas estratégicas actuales de la empresa?`
   - `Quelle est la stratégie de l'entreprise en matière d'IA générative ?`
   - `Was sind die wichtigsten Wachstumstreiber für das Unternehmen?`

![App](/images/2-Bedrock/basic/B-3/4.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a question & answer app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}