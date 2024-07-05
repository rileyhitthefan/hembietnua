---
title: "Lab O-2: Text generation playground"
date: 2024-07-05T15:36:30+07:00
draft: false
weight: 2
chapter : false
---

In this lab, we will build a text playground with Amazon Titan Text, LangChain, and Streamlit. We will collect user input and inference parameters, pass it to Bedrock, and return Titan’s response. This application gives the end user direct control of the inference parameters.

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The text generation playground is good for the following use cases:

- Developers cannot access the AWS console, but want to experiment with different prompts and inference parameters.

### Architecture

![architecture](/images/2-Bedrock/bonus/O-2/architecture.png)

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/text_playground** folder, and open the file **text_playground_lib.py**

![architecture](/images/2-Bedrock/bonus/O-2/open.png)

2. Add the import statements.

   - These statements allow us to use LangChain to call Bedrock.

```py
from langchain_community.llms import Bedrock
```

3. Add this function to call Bedrock.

   - We're creating a function we can call from the Streamlit front end application. This function creates a Bedrock client with LangChain, then passes the input content and inference parameters to Bedrock.

```python
def get_titan_response(model, input_content, temperature, top_p, max_token_count, stop_sequence): #text-to-text client function
    
    model_kwargs = { #For the LangChain Bedrock implementation, these parameters will be added to the textGenerationConfig item that LangChain creates for us
        "maxTokenCount": max_token_count, 
        "stopSequences": [stop_sequence],
        "temperature": temperature, 
        "topP": top_p
    }
    
    llm = Bedrock( #create a Bedrock llm client
        model_id=model, #use the requested model
        model_kwargs = model_kwargs
    )
    
    return llm.invoke(input_content) #return a response to the prompt
```

4. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **text_playground_app.py**

2. Add the import statements.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import text_playground_lib as glib #reference to local lib script
```

3. Add the page title and configuration.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(layout="wide", page_title="Text Playground") #set the page width wider to accommodate columns

st.title("Text Playground") #page title

col1, col2 = st.columns(2) #create 2 columns
```

4. Add the input elements.

   - We are creating a multiline text box, sliders, and button to get the user's prompt and inference parameters and send them to Bedrock.

```python
with col1:
    input_text = st.text_area("Input text", height=400) #display a multiline text box
    go_button = st.button("Go", type="primary") #display a primary button
            
with col2:
    model = "amazon.titan-text-express-v1"
    
    help_temperature = "Modulates the probability density function for the next tokens, implementing the temperature sampling technique. This parameter can be used to deepen or flatten the density function curve. A lower value results in a steeper curve and more deterministic responses, whereas a higher value results in a flatter curve and more random responses. (float, defaults to 0, max value is 1.5)"
    help_top_p = "Top P controls token choices, based on the probability of the potential choices. If you set Top P below 1.0, the model considers only the most probable options and ignores less probable options. The result is more stable and repetitive completions."
    help_response_tokens = "Configures the max number of tokens to use in the generated response. (int, defaults to 512)"
    help_stop_sequences = "Number of pipe symbols to use for a stop sequence."
    
    titan_temperature = st.slider("Temperature", min_value=0.0, max_value=1.0, value=0.0, step=0.1, help=help_temperature, format='%.1f')
    titan_top_p = st.slider("Top P", min_value=0.1, max_value=1.0, value=0.9, step=0.1, help=help_top_p, format='%.1f')
    titan_max_token_count = st.slider("Response tokens", min_value=1, max_value=4096, value=512, step=1, help=help_response_tokens)
    titan_stop_sequence = st.text_input("Stop Sequence", value="User:", help=help_stop_sequences)
```

5. Add the output elements.

   - We use the `if` block below to handle the button click. We display a spinner while the backing function is called, then write the output to the web page.

```python
if go_button: #code in this if block will be run when the button is clicked

    with st.spinner("Working..."): #show a spinner while the code in this with block runs
        response_content = glib.get_titan_response( #call the model through the supporting library
            model=model,
            input_content=input_text, 
            temperature=titan_temperature, 
            top_p=titan_top_p, 
            max_token_count=titan_max_token_count, 
            stop_sequence=titan_stop_sequence) 

        st.write(response_content) #display the response content
```

6. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/text_playground
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/text_playground
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run text_playground_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/bonus/O-2/app.png)

4. Try out some prompts and see the results:

   - `Why is the sky blue?`
   - `What is the capital of New Hampshire?`
   - `I am pleased to meet you. What is the sentiment of the previous statement?`

![app](/images/2-Bedrock/bonus/O-2/app-in-use.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a text playground app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}