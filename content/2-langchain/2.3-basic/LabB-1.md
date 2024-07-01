---
title: "Lab B-1: Text Generation"
date: 2024-07-01T13:20:41+07:00
draft: false
chapter : false
weight: 1
---

In this lab, we will build a simple text generator with [Amazon Bedrock](https://aws.amazon.com/bedrock/), **LangChain**, and **Streamlit**. We will collect user input, pass it to Bedrock, and return the foundation model’s response. While this is a fairly trivial example, it allows us to understand how to build a basic generative AI prototype with very little code.

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The basic text generation pattern is good for the following use cases:
- General content creation, where factuality isn't critical
- Basic question & answer for well-known facts that are repeated frequently on the internet

### Architecture
![Architecture](/images/2-Bedrock/basic/B-1/architecture.png)
This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/text** folder, and open the file **text_lib.py**

![textlib.py](/images/2-Bedrock/basic/B-1/1.png)

2. Add the import statements.
   - These statements allow us to use LangChain to call Bedrock and read environment variables.

```py
from langchain_community.llms import Bedrock
```

3. Add this function to call Bedrock.

   - We're creating a function we can call from the Streamlit front end application. This function creates a Bedrock client with LangChain, then passes the input content to Bedrock.

```py 
def get_text_response(input_content): #text-to-text client function

    llm = Bedrock( #create a Bedrock llm client
        model_id="cohere.command-text-v14", #set the foundation model
        model_kwargs={
            "max_tokens": 512,
            "temperature": 0,
            "p": 0.01,
            "k": 0,
            "stop_sequences": [],
            "return_likelihoods": "NONE"
        }
    )
    
    return llm.invoke(input_content) #return a response to the prompt
```

4. Save the file.

## Create the Streamlit front-end app

1. In the same folder as your lib file, open the file **text_app.py**

2. Add the import statements.
   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
import text_lib as glib #reference to local lib script
```

3. Add the page title and configuration.
   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```py
st.set_page_config(page_title="Text to Text") #HTML title
st.title("Text to Text") #page title
```

4. Add the input elements.
   - We are creating a multiline text box and button to get the user's prompt and send it to Bedrock.

```py
input_text = st.text_area("Input text", label_visibility="collapsed") #display a multiline text box with no label
go_button = st.button("Go", type="primary") #display a primary button
```

5. Add the output elements.
   - We use the if block below to handle the button click. We display a spinner while the backing function is called, then write the output to the web page.

```py
if go_button: #code in this if block will be run when the button is clicked
    
    with st.spinner("Working..."): #show a spinner while the code in this with block runs
        response_content = glib.get_text_response(input_content=input_text) #call the model through the supporting library
        
        st.write(response_content) #display the response content
```

6. Save the file.

## Run the Streamlit app 
1. Select the bash terminal in AWS Cloud9 and change directory.
```bash
cd ~/environment/workshop/labs/text
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/text
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.
```bash
streamlit run text_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![App](/images/2-Bedrock/basic/B-1/3.png)

4. Try out some prompts and see the results.
   - `What is a good name for a product that provides large language models?`
   - `Why is the sky blue?`
   - `What is the capital of New Hampshire?`
   - `I am pleased to meet you. What is the sentiment of the previous statement?`

![App Results](/images/2-Bedrock/basic/B-1/4.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a text-to-text app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}