---
title: "Lab M-1: Image search"
date: 2024-07-03T09:02:06+07:00
draft: false
weight: 1
chapter : false
---

In this lab, we will build a simple image search application with **Titan Multimodal Embeddings**, **LangChain**, and **Streamlit**.

[Titan Multimodal Embeddings](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-multiemb-models.html) is a multimodal embeddings model for use cases like searching images by text, image, or a combination of text and image.

This example is similar to the [Embeddings Search](../2.4-text/LabI-4.md) lab. In this example, we create a searchable index of images by doing the following:

1. Read each image file in a local directory.
2. Pass the image bytes to Titan Multimodal Embeddings to get a numerical representation (AKA vector or embedding) of the image.
3. Save the embedding and metadata (including the path to the original image file) to an in-memory [FAISS](https://github.com/facebookresearch/faiss)  database.

We can then search for images in the FAISS database using the following approach:

1. Convert a user’s search expression or uploaded image to a numerical representation using Amazon Titan Multimodal Embeddings.
1. Search the FAISS database for the closest matches.
1. Return and display the closest matching images.

We use FAISS in this example for simplicity’s sake, at the expense of scalability or durability. In a real-world scenario, you will most likely want to use a persistent data store like the [vector engine for Amazon OpenSearch Serverless](https://aws.amazon.com/opensearch-service/serverless-vector-engine/) or [Amazon MemoryDB](https://docs.aws.amazon.com/memorydb/latest/devguide/vector-search-overview.html)

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image search pattern is good for the following use cases:

- Image collection search
- Finding similar / identical images in a collection
- Classifying images based on similarity to existing examples

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/image_search** folder, and open the file **image_search_lib.py**

![lib](/images/2-Bedrock/image/M-1/1.png)

2. Add the import statements.

   - These statements allow us to use LangChain to manage our FAISS database, and use Boto3 to call Bedrock.

```python
import os
import boto3
import json
import base64
from langchain_community.vectorstores import FAISS
from io import BytesIO
```

3. Add this function to call Bedrock.

```python
#calls Bedrock to get a vector from either an image, text, or both
def get_multimodal_vector(input_image_base64=None, input_text=None):
    
    session = boto3.Session()

    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    request_body = {}
    
    if input_text:
        request_body["inputText"] = input_text
        
    if input_image_base64:
        request_body["inputImage"] = input_image_base64
    
    body = json.dumps(request_body)
    
    response = bedrock.invoke_model(
    	body=body, 
    	modelId="amazon.titan-embed-image-v1", 
    	accept="application/json", 
    	contentType="application/json"
    )
    
    response_body = json.loads(response.get('body').read())
    
    embedding = response_body.get("embedding")
    
    return embedding
```

4. Add the functions to create the in-memory vector store from the image files on disk.

   - See the comments in the code below for more details.

```python
#creates a vector from a file
def get_vector_from_file(file_path):
    with open(file_path, "rb") as image_file:
        input_image_base64 = base64.b64encode(image_file.read()).decode('utf8')
    
    vector = get_multimodal_vector(input_image_base64 = input_image_base64)
    
    return vector


#creates a list of (path, vector) tuples from a directory
def get_image_vectors_from_directory(path):
    items = []
    
    for file in os.listdir("images"):
        file_path = os.path.join(path,file)
        
        vector = get_vector_from_file(file_path)
        
        items.append((file_path, vector))
        
    return items


#creates and returns an in-memory vector store to be used in the application
def get_index(): 

    image_vectors = get_image_vectors_from_directory("images")
    
    text_embeddings = [("", item[1]) for item in image_vectors]
    metadatas = [{"image_path": item[0]} for item in image_vectors]
    
    index = FAISS.from_embeddings(
        text_embeddings=text_embeddings,
        embedding = None,
        metadatas = metadatas
    )
    
    return index
```

5. Add these functions to get search results for the image or text query.

   - This code passes the user's search expression or image to Bedrock and returns its embeddings vector.

```python
#get a base64-encoded string from file bytes
def get_base64_from_bytes(image_bytes):
    
    image_io = BytesIO(image_bytes)
    
    image_base64 = base64.b64encode(image_io.getvalue()).decode("utf-8")
    
    return image_base64


#get a list of images based on the provided search term and/or search image
def get_similarity_search_results(index, search_term=None, search_image=None):
    
    search_image_base64 = (get_base64_from_bytes(search_image) if search_image else None)

    search_vector = get_multimodal_vector(input_text=search_term, input_image_base64=search_image_base64)
    
    results = index.similarity_search_by_vector(embedding=search_vector)
    
    results_images = []
    
    for res in results: #load images into list
        
        with open(res.metadata['image_path'], "rb") as f:
            img = BytesIO(f.read())
        
        results_images.append(img)
    
    
    return results_images
```

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_search_app.py**

2. Add the import statements and page configuration.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.
   - We are setting the page title on the actual page and the title shown in the browser tab.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import image_search_lib as glib #reference to local lib script


st.set_page_config(page_title="Image Search", layout="wide") #HTML title
st.title("Image Search") #page title
```

3. Add the vector index to the session cache, and initialize the tabs layout.

   - This allows us to maintain an in-memory vector database per user session.
   - The tabs will contain two input modes: image search or finding similar images.

```python
if 'vector_index' not in st.session_state: #see if the vector index hasn't been created yet
    with st.spinner("Indexing images..."): #show a spinner while the code in this with block runs
        st.session_state.vector_index = glib.get_index() #retrieve the index through the supporting library and store in the app's session cache


search_images_tab, find_similar_images_tab = st.tabs(["Image search", "Find similar images"])
```

4. Add the elements for image search mode.

```python
with search_images_tab:

    search_col_1, search_col_2 = st.columns(2)

    with search_col_1:
        input_text = st.text_input("Search for:") #display a multiline text box with no label
        search_button = st.button("Search", type="primary") #display a primary button

    with search_col_2:
        if search_button: #code in this if block will be run when the button is clicked
            st.subheader("Results")
            with st.spinner("Searching..."): #show a spinner while the code in this with block runs
                response_content = glib.get_similarity_search_results(index=st.session_state.vector_index, search_term=input_text)
                
                #st.write(response_content) #using table so text will wrap
                
                for res in response_content:
                    st.image(res, width=250)
```

5. Add the elements for find similar images mode.

```python
with find_similar_images_tab:
    
    find_col_1, find_col_2 = st.columns(2)

    with find_col_1:
    
        uploaded_file = st.file_uploader("Select an image", type=['png', 'jpg'])
        
        if uploaded_file:
            uploaded_image_preview = uploaded_file.getvalue()
            st.image(uploaded_image_preview)
    
        find_button = st.button("Find", type="primary") #display a primary button

    with find_col_2:
        if find_button: #code in this if block will be run when the button is clicked
            st.subheader("Results")
            with st.spinner("Finding..."): #show a spinner while the code in this with block runs
                response_content = glib.get_similarity_search_results(index=st.session_state.vector_index, search_image=uploaded_file.getvalue())
                
                for res in response_content:
                    st.image(res, width=250)
```

6. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image_search
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image_search
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run image_search_app.py --server.port 8080 --server.enableXsrfProtection=false
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-1/2.png)

4. Under the Image search tab, try some search terms:

   - `Cat`
   - `Two cats`
   - `People`
   - `House`
   - `Car and house`
   - `Tacos`
   - `Italian food`
   - `Bedroom`
   - `Living room`
   - `Miniature`
   - `Christmas`

![app](/images/2-Bedrock/image/M-1/3.png)

5. Under the **Find similar images** tab, try uploading an image:

You can download sample images from the **/workshop/sample_images** folder in your Cloud9 environment.

![app](/images/2-Bedrock/image/M-1/4.png)

6. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image search app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}