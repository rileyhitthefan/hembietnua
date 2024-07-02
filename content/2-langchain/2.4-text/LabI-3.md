---
title: "Lab I-3: Response streaming"
date: 2024-07-01T16:11:56+07:00
draft: false
weight: 3
chapter : false
---

In this lab, we will build a response streaming application using Bedrock, LangChain, and Streamlit.

Streaming responses are useful when you want to start returning content immediately to the end user. You can display the output a few words at a time, instead of waiting for the entire response to be created.

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The response streaming pattern is good for the following use cases:
- Situations where longer text will be generated, but you want to keep the user engaged by beginning to return a response immediately.

### Architecture
![architecture](/images/2-Bedrock/text/I-3/architecture.png)

From an architectural perspective, response streaming is similar to text-to-text. We just need to add a special handler to immediately process the streaming output as it is created.

The streamed response is returned in chunks of JSON. You can then extract the returned text from each chunk to be displayed to the end user.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/streaming** folder, and open the file **streaming_lib.py**

![lib](/images/2-Bedrock/text/I-3/1.png)

2. Add the import statements.
- These statements allow us to use LangChain to call Bedrock and process the streaming output.

```py
from langchain.chains import ConversationChain
from langchain_community.llms import Bedrock
```

3. Add the code to create a LangChain Bedrock client with streaming enabled.

```python
def get_llm(streaming_callback):
    model_kwargs = {
        "max_tokens": 4000,
        "temperature": 0,
        "p": 0.01,
        "k": 0,
        "stop_sequences": [],
        "return_likelihoods": "NONE",
        "stream": True
    }
    
    llm = Bedrock(
        model_id="cohere.command-text-v14",
        model_kwargs=model_kwargs,
        streaming=True,
        callbacks=[streaming_callback],
    )
    
    return llm
```

4. Add this function to call Bedrock and handle the streaming response.
   - This function calls Bedrock with the streaming invocation method. As response chunks are returned, it passes the chunk's text to the provided callback method.

```python
def get_streaming_response(prompt, streaming_callback):
    conversation_with_summary = ConversationChain(
        llm=get_llm(streaming_callback)
    )
    return conversation_with_summary.predict(input=prompt)
```

5. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **streaming_app.py**

2. Add the import statements.
   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streaming_lib as glib  # reference to local lib script
import streamlit as st
from langchain_community.callbacks.streamlit import StreamlitCallbackHandler
```

3. Add the page title, configuration, and two-column layout.
   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(page_title="Response Streaming")  # HTML title
st.title("Response Streaming")  # page title
```

4. Add the input elements.
   - We are creating a multiline text box and button to get the user's prompt and send it to Bedrock.

```python
input_text = st.text_area("Input text", label_visibility="collapsed")
go_button = st.button("Go", type="primary")  # display a primary button
```

5. Add the output elements.
   - We use the `if` block below to handle the button click.
   - We create an empty streamlit container and pass it to the `StreamlitCallbackHandler` object so it can display output as it is generated.
   - We pass the `StreamlitCallbackHandler` to the backing function so it can handle responses as streaming chunks are returned.

```python
if go_button:  # code in this if block will be run when the button is clicked
    #use an empty container for streaming output
    st_callback = StreamlitCallbackHandler(st.container())
    streaming_response = glib.get_streaming_response(prompt=input_text, streaming_callback=st_callback)
```

6. Save the file

## Run the Streamlit app

1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/streaming
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/streaming
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run streaming_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/text/I-3/2.png)

4. Try out some prompts and see the results.
   - `Write a story about two cats that go on an adventure:`

![app](/images/2-Bedrock/text/I-3/3.gif)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a response streaming app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}