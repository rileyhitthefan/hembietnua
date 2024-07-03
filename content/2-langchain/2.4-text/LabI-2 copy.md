---
title: "Lab I-2: Tóm tắt tài liệu"
date: 2024-07-01T16:11:54+07:00
draft: false
weight: 2
chapter : false
---
In this lab, we will build a simple document summarizer with **Amazon Bedrock, LangChain, and Streamlit**.

LangChain includes a **map-reduce summarization** function that allows us to process content that exceeds the token limit for the model. The map-reduce function works by breaking up a document into smaller pieces, summarize those pieces, then summarize the pieces' summaries.

{{% notice info %}}
This lab might fail if it exceeds your throttling rate limit.
{{% /notice %}}

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The map-reduce summarization pattern is good for the following use cases:
- Summarizing long documents
- Summarizing call transcripts
- Summarizing customer activity history

### Architecture

![architecture](/images/2-Bedrock/text/I-2/architecture.png)

The Map-Reduce pattern involves the following steps:

1. Break up the large document into small chunks
2. Generate intermediate summaries based on those small chunks
3. Summarize the intermediate summaries into a combined summary

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/summarization** folder, and open the file **summarization_lib.py**

![architecture](/images/2-Bedrock/text/I-2/lib.png)

2. Add the import statements.
   - These statements allow us to use LangChain to load a PDF file, split the document, and call Bedrock.

```python
from langchain.prompts import PromptTemplate
from langchain_community.llms import Bedrock
from langchain.chains.summarize import load_summarize_chain
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader
```

3. Add a function to create a Bedrock LangChain client.
   - This includes the inference parameters we want to use.

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

4. Add the function to create the document chunks.
   - This code attempts to split the document by paragraph, line, sentence, or word.

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

5. Add this function to call Bedrock.
   - This code creates prompts for the map and reduce steps. It then passes the documents to the map-reduce summarizer chain to produce the combined summary.

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

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **summarization_app.py**

2. Add the import statements.
   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```py
import streamlit as st
import summarization_lib as glib
```

3. Add the page title and configuration.
   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(page_title="Document Summarization")
st.title("Document Summarization")
```

4. Add the summarization elements.
   - This section will not be displayed until the session state's `has_document` property has been set.
   - We are creating a checkbox and button to get the user's prompt and send it to Bedrock. The "Return intermediate steps" checkbox determines if we will display the summaries from the map stage.
   - We use the `if` block below to handle the button click. We display a spinner while the backing function is called, then write the output to the web page.

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

5. Save the file.

## Run the Streamlit app
1. Select the bash terminal in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/summarization
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/summarization
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```python
streamlit run summarization_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select Preview -> Preview Running Application.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![App](/images/2-Bedrock/text/I-2/1.png)

4. Click the summarize button.
   - The summarization may take about 90 seconds to run.
   - Once complete, the combined summary for the document will be displayed, along with the individual section summaries if you chose to return the intermediate steps.

{{% notice info %}}
This lab might fail if it exceeds your throttling rate limit.
{{% /notice %}}

![App Result](/images/2-Bedrock/text/I-2/2.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a document summarizer with Bedrock, LangChain, and Streamlit!
{{% /notice %}}