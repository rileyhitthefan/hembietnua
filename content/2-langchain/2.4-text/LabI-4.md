---
title: "Lab I-4: Embeddings search"
date: 2024-07-01T16:11:58+07:00
draft: false
weight: 4
chapter : false
---

In this lab, we will build a simple embeddings search application with Titan Embeddings, LangChain, and Streamlit.

This example is similar to the [Retrieval-Augmented Generation](../2.3-basic/LabB-3.md) lab. In this lab, we also match a query to the closest entries in a vector database. But in this case, we do not pass those matches to a large language model. Instead, we will just display those matches directly in the user interface. This can be useful if you want to troubleshoot a RAG application, or directly evaluate an embeddings model.

In this lab, we will use an in-memory [FAISS](https://github.com/facebookresearch/faiss)  database to store and search for embeddings vectors. In a real-world scenario, you will most likely want to use a persistent data store like Amazon Kendra or the [vector engine for Amazon OpenSearch Serverless](https://aws.amazon.com/opensearch-service/serverless-vector-engine/).

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The embeddings search pattern is good for the following use cases:
- Identifying related items based on text descriptions
- Application portfolio rationalization - particularly in cases where data between the companies or divisions is inconsistent, matching applications based on their descriptions can accelerate the process of finding potential overlap.

### Architecture

![architecture](/images/2-Bedrock/text/I-4/architecture.png)

1. A document is broken up into chunks of text. The chunks are passed to Titan Embeddings to be converted to vectors. The vectors are then saved to the vector database.
2. The user submits a question.
3. The question is converted to a vector using Amazon Titan Embeddings, then matched to the closest vectors in the vector database.
4. The combined content from the matching vectors are then returned to the user

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/embeddings_search** folder, and open the file **embeddings_search_lib.py**

![lib](/images/2-Bedrock/text/I-4/1.png)

2. Add the import statements.
   - These statements allow us to use LangChain to load a CSV file, index the document, and call Bedrock.

```python
from langchain_community.embeddings import BedrockEmbeddings
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import CharacterTextSplitter
from langchain_community.document_loaders.csv_loader import CSVLoader
```

3. Add the function to create the in-memory vector store.
   - See the in-line comments in the code below for more details.

```python
def get_index(): #creates and returns an in-memory vector store to be used in the application
    
    embeddings = BedrockEmbeddings() #create a Titan Embeddings client
    
    loader = CSVLoader(file_path="sagemaker_answers.csv")

    index_creator = VectorstoreIndexCreator(
        vectorstore_cls=FAISS,
        embedding=embeddings,
        text_splitter=CharacterTextSplitter(chunk_size=300, chunk_overlap=0),
    )

    index_from_loader = index_creator.from_loaders([loader])
    
    return index_from_loader
```

4. Add this function to call Bedrock.
   - This code searches the previously created index based on the user's input, and returns the best results in a flattened form.

```python
def get_similarity_search_results(index, question):
    results = index.vectorstore.similarity_search_with_score(question)
    
    flattened_results = [{"content":res[0].page_content, "score":res[1]} for res in results] #flatten results for easier display and handling
    
    return flattened_results
```

5. Add this function to call get the raw embeddings for the query.
   - This code passes the user's question to Bedrock and returns its embeddings vector.

```python
def get_embedding(text):
    embeddings = BedrockEmbeddings() #create a Titan Embeddings client
    
    return embeddings.embed_query(text)
```

6. Save the file.

## Create the Streamlit front-end app

1. In the same folder as your lib file, open the file **embeddings_search_app.py**

2. Add the import statements.
   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import embeddings_search_lib as glib #reference to local lib script
```

3. Add the page title and configuration.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(page_title="Embeddings Search", layout="wide") #HTML title
st.title("Embeddings Search") #page title
```

4. Add the vector index to the session cache.
   - This allows us to maintain an in-memory vector database per user session.

```python
if 'vector_index' not in st.session_state: #see if the vector index hasn't been created yet
    with st.spinner("Indexing document..."): #show a spinner while the code in this with block runs
        st.session_state.vector_index = glib.get_index() #retrieve the index through the supporting library and store in the app's session cache
```

5. Add the input elements.
   - We are creating a multiline text box and button to get the user's prompt and send it to Bedrock.

```python
input_text = st.text_input("Ask a question about Amazon SageMaker:") #display a multiline text box with no label
go_button = st.button("Go", type="primary") #display a primary button
```

6. Add the output elements.
   - We use the `if` block below to handle the button click. We display a spinner while the backing functions are called.
   - Streamlit's `table` function displays the search results.
   - We use Streamlit's `json` function to display the embeddings array, and conditionally hide it with a Streamlit `expander`.

```python
if go_button: #code in this if block will be run when the button is clicked
    
    with st.spinner("Working..."): #show a spinner while the code in this with block runs
        response_content = glib.get_similarity_search_results(index=st.session_state.vector_index, question=input_text)
        
        st.table(response_content) #using table so text will wrap
        
        
        raw_embedding = glib.get_embedding(input_text)
        
        with st.expander("View question embedding"):
            st.json(raw_embedding)
```

7. Save the file.

## Run the Streamlit app

1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/embeddings_search
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/embeddings_search
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run embeddings_search_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/text/I-4/2.png)

4. Try out some prompts and see the results. The lower the **score** value, the closer the match.
   - `What can you do with SageMaker Canvas?`
   - `How does SageMaker support MLOps?`
   - `How does a feature store work?`

![app](/images/2-Bedrock/text/I-4/3.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an embeddings search app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}