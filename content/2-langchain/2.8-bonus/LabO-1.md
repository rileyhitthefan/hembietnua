---
title: "Lab O-1: Prompt showcase"
date: 2024-07-05T15:36:28+07:00
draft: false
weight: 1
chapter : false
---

In this lab, we will build a prompt showcase app with Amazon Bedrock, LangChain, and Streamlit. We will allow the user to select from a list of pre-made prompts and input content, pass them to Bedrock, and return Titan’s response. This application gives the end user direct control of the prompt template and input content.

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The prompt showcase is good for the following scenarios:

- Demonstrating multiple text-to-text use cases in a single application.

### Architecture

![architecture](/images/2-Bedrock/bonus/O-1/architecture.png)

This application consists of three files: one for the Streamlit front end, one for the supporting library to make calls to Bedrock, and one for storing the prompt templates and input examples.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/showcase** folder, and open the file **showcase_lib.py**

![open](/images/2-Bedrock/bonus/O-1/open.png)

2. Add the import statements.

   - These statements allow us to use LangChain to call Bedrock.

```python
from langchain_community.llms import Bedrock
from langchain.prompts import PromptTemplate
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

4. Add the function to create the custom prompt.

   - This code builds a custom prompt from the template and user input parameters.

```python
def get_prompt(user_input, template):
    
    prompt_template = PromptTemplate.from_template(template) #this will automatically identify the input variables for the template

    prompt = prompt_template.format(user_input=user_input)
    
    return prompt
```

5. Add this function to call Bedrock.

   - This function passes the custom prompt to Bedrock.

```python
def get_text_response(user_input, template): #text-to-text client function
    llm = get_llm()
    
    prompt = get_prompt(user_input, template)
    
    return llm.invoke(prompt) #return a response to the prompt
```

6. Save the file.

## Create the examples script
We use the examples script to hold both our example prompts and user inputs so they can be viewed in the user interface. We capture each example as a dictionary entry, and use Python triple quotes for the values so we can easily make changes to multiline examples.

1. In the same folder as your lib file, open the file **showcase_examples.py**

2. Create the example dictionaries

   - These dictionaries are used to hold the examples to be displayed in the user interface.

```python
#################################################################################################################################

prompts = {} #pre-defined prompt templatess, include "{user_input}" to merge input content
inputs = {} #used to merge into prompt templates, merged into the "{user_input}" placeholder
defaults = {} #used for default values in simple examples
```

3. Define example prompts.

   - The prompts dictionary key will be used to select example prompts in the user interface.
   - When the prompt dictionary key is selected in the UI, its value will be loaded for either direct use or to be modified by the user.

```python
#################################################################################################################################
# PROMPTS
#################################################################################################################################

prompts["Reply Template"] = """
{user_input}

Please write a reply to the above text:
"""

#################################################################################################################################

prompts["Summarize"] = """
{user_input}

Please summarize the above content:
"""

#################################################################################################################################

prompts["Sentiment"] = """
{user_input}

Sentiment of the above content (Positive or negative):
"""

#################################################################################################################################

prompts["Recommendation"] = """
{user_input}

Recommended next step based on the above content:
"""
```

4. Define example inputs.

   - The inputs dictionary key will be used to select example inputs in the user interface.
   - When the inputs dictionary key is selected in the UI, its value will be loaded for either direct use or to be modified by the user.

```python
#################################################################################################################################
# INPUTS
#################################################################################################################################

inputs["Complementary Customer Email"] = """
Dear Acme Investments,
I am writing to compliment one of your customer service representatives, Shirley Scarry. I recently had the pleasure of speaking with Shirley regarding my loan. Shirley was extremely helpful and knowledgeable, and went above and beyond to ensure that all of my questions were answered. Shirley also had Robert Herbford join the call, who wasn't quite as helpful. My wife, Clara Bradford, didn't like him at all.
Shirley's professionalism and expertise were greatly appreciated, and I would be happy to recommend Acme Investments to others based on my experience.
Sincerely,

Carson Bradford
"""

#################################################################################################################################

inputs["Ethics Complaint Email"] = """
Dear Acme Investments,
I am writing to bring to your attention a situation that I believe to be unethical on the part of one of your account managers, Roger Longbottom.
I recently met with Roger to discuss my investment portfolio and was deeply concerned to hear that he suggested I invest in a certain stock. When I asked him why he thought this was a good investment, he stated that the stock was currently undervalued and was likely to increase in value in the near future.
However, upon further research, I have discovered that the stock in question has a questionable reputation. It has been the subject of multiple lawsuits and has been found to have engaged in questionable business practices.
I believe Roger was aware of these facts, but failed to disclose them to me. As a result, I feel I was misled into making an unwise investment decision.
I therefore urge you to investigate whether Roger has acted unethically and take appropriate action if necessary.
Yours sincerely,
Carson Bradford
"""
```

5. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **showcase_app.py**

2. Add the import statements.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st
import showcase_lib as glib
import showcase_examples as examples
```

3. Add the page title and column layout.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.
   - We are creating three columns, for the prompt templates, user inputs, and results.

```python
st.set_page_config(page_title="Demo Showcase", layout="wide")

st.title("Demo Showcase")

col1, col2, col3 = st.columns(3)
```

4. Add the prompt elements.

   - We are creating a `selectbox` control to allow the user to choose from various prompt examples.
   - We use the `expander` control to optionally hide the prompt details in the demo application.
   - The user can customize the prompt in the user interface (changes won't be saved, though).

```python
with col1:
    st.subheader("Prompt template")
    
    prompts_keys = list(examples.prompts)

    prompt_selection = st.selectbox("Select a prompt template:", prompts_keys)
    
    with st.expander("View prompt"):

        selected_prompt_template_text = examples.prompts[prompt_selection]

        prompt_text = st.text_area("Prompt template text:", value=selected_prompt_template_text, height=350)
```

5. Add the input elements.

   - Similar to above, we use a `selectbox` control to allow the user to select various input examples.
   - The user can customize the input in the user interface (changes won't be saved, though).

```python
with col2:
    st.subheader("User input")
    inputs_keys = list(examples.inputs)
    
    input_selection = st.selectbox("Select an input example:", inputs_keys)
    
    selected_input_template_text = examples.inputs[input_selection]

    input_text = st.text_area("Input text:", value=selected_input_template_text, height=350)
    
    process_button = st.button("Run", type="primary")
```

6. Add the output elements.

   - We use the `if` block below to handle the button click. We display a spinner while the backing function is called, then write the output to the web page.

```python
with col3:
    st.subheader("Result")
    
    if process_button:
        with st.spinner("Running..."):
            response_content = glib.get_text_response(user_input=input_text, template=prompt_text)

            st.write(response_content)
```

7. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/showcase
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/showcase
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run showcase_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/bonus/O-1/app.png)

4. Try selecting different prompts and inputs and see the results.

![app](/images/2-Bedrock/bonus/O-1/app-in-use.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a prompt showcase app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}