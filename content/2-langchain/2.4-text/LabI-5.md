---
title: "Lab I-5: Personalized recommendations"
date: 2024-07-01T16:12:01+07:00
draft: false
weight: 5
chapter : false
---
In this lab, we will build a personalized recommendations application with Amazon Bedrock, LangChain, and Streamlit.

This example is similar to the [Retrieval-Augmented Generation](../2.3-basic/LabB-3.md) lab. In this lab, we also match a query to the closest entries in a vector database. But in this case, we will pass each matched result to the large language model to create a personalized summary about that match.

In this lab, we will use an in-memory [FAISS](https://github.com/facebookresearch/faiss) database to store and search for embeddings vectors. In a real-world scenario, you will most likely want to use a persistent data store like Amazon Kendra or the [vector engine for Amazon OpenSearch Serverless](https://aws.amazon.com/opensearch-service/serverless-vector-engine/).

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The personalized recommendations pattern is good for the following use cases:
- Creating personalized product recommendations, including a justification for each recommendation.
- Creating custom "10 best" articles for vacation destinations, colleges, vehicles, or anything else that lends itself to that type of article.

### Architecture

![architecture](/images/2-Bedrock/text/I-5/architecture.png)

1. A document is broken up into chunks of text. The chunks are passed to Titan Embeddings to be converted to vectors. The vectors are then saved to the vector database.
2. The user submits a request.
3. The question is converted to a vector using Amazon Titan Embeddings, then matched to the closest vectors in the vector database.
4. The combined content and request are then used to generate a personalized recommendation to return to the user.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/recommendations** folder, and open the file 

![lib](/images/2-Bedrock/text/I-5/1.png)

2. Add the import statements.

   - These statements allow us to use LangChain to load a JSON file, index the document, and call Bedrock.

```python
from langchain_community.llms import Bedrock
from langchain_community.embeddings import BedrockEmbeddings
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import JSONLoader
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

4. Add a function to capture the metadata for each vector record.

   - This allows us to store the product name and URL along with the embeddings data, so we can include them when displaying results.

```python
#function to identify the metadata to capture in the vectorstore and return along with the matched content
def item_metadata_func(record: dict, metadata: dict) -> dict: 

    metadata["name"] = record.get("name")
    metadata["url"] = record.get("url")

    return metadata
```

5. Add the function to create the in-memory vector store.

   - See the in-line comments in the code below for more details.

```python
def get_index(): #creates and returns an in-memory vector store to be used in the application
    
    embeddings = BedrockEmbeddings() #create a Titan Embeddings client
    
    loader = JSONLoader(
        file_path="services.json",
        jq_schema='.[]',
        content_key='description',
        metadata_func=item_metadata_func)

    text_splitter = RecursiveCharacterTextSplitter( #create a text splitter
        separators=["\n\n", "\n", ".", " "], #split chunks at (1) paragraph, (2) line, (3) sentence, or (4) word, in that order
        chunk_size=8000, #based on this content, we just want the whole item so no chunking - this could lead to an error if the content is too long
        chunk_overlap=0 #number of characters that can overlap with previous chunk
    )
    
    index_creator = VectorstoreIndexCreator( #create a vector store factory
        vectorstore_cls=FAISS, #use an in-memory vector store for demo purposes
        embedding=embeddings, #use Titan embeddings
        text_splitter=text_splitter, #use the recursive text splitter
    )
    
    index_from_loader = index_creator.from_loaders([loader]) #create an vector store index from the loaded PDF
    
    return index_from_loader #return the index to be cached by the client app
```

6. Add this function to call Bedrock.

   - This code searches the previously created index based on the user's input, creates a personalized summary based on each match, and returns the best results in a flattened form.

```python
def get_similarity_search_results(index, question):
    raw_results = index.vectorstore.similarity_search_with_score(question)
    
    llm = get_llm()
    
    results = []
    
    for res in raw_results:
        content = res[0].page_content
        prompt = f"{content}\n\nSummarize how the above service addresses the following needs : {question}"
        
        summary = llm.invoke(prompt)
        
        results.append({"name": res[0].metadata["name"], "url": res[0].metadata["url"], "summary": summary, "original": content})
    
    return results
```

7. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **recommendations_app.py**

2. Add the import statements.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import recommendations_lib as glib #reference to local lib script
```

3. Add the page title and configuration.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(page_title="Personalized Recommendations", layout="wide") #HTML title
st.title("Personalized Recommendations") #page title
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
input_text = st.text_input("Name some key features you need from a cloud service:") #display a multiline text box with no label
go_button = st.button("Go", type="primary") #display a primary button
```

6. Add the output elements.

   - We use the `if` block below to handle the button click. We display a spinner while the backing functions are called.
   - We display each result with the product name linked to its URL, its personalized recommendation, and an expander with the source content.

```python
if go_button: #code in this if block will be run when the button is clicked
    
    with st.spinner("Working..."): #show a spinner while the code in this with block runs
        response_content = glib.get_similarity_search_results(index=st.session_state.vector_index, question=input_text)
        
        for result in response_content:
            st.markdown(f"### [{result['name']}]({result['url']})")
            st.write(result['summary'])
            with st.expander("Original"):
                st.write(result['original'])
```

7. Save the file.

## Run the Streamlit app

1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/recommendations
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/recommendations
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run recommendations_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/text/I-5/2.png)

4. Try out some prompts and see the results:

   - `desktop application hosting`
   - `serverless messaging`
   - `NoSQL database`

![app](/images/2-Bedrock/text/I-5/3.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a personalized recommendation app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}