---
title: "Lab O-3: Custom prompt templates"
date: 2024-07-05T15:36:32+07:00
draft: false
weight: 3
chapter : false
---

In this lab, we will build a simple custom prompt formatter with Amazon Bedrock, LangChain, and Streamlit.

Custom prompt templates allow us to merge various inputs into a single prompt. Prompt templates are needed when context must passed to the model along with the user's input. For example, with a chatbot LangChain uses a prompt template to merge the user's chat history with their latest message, so the model will understand what has been said before.

Another use case for prompt templates is combining data from a database to pass to a model. For example, we may need to pass the user's name and preferences to a model to have it generate a custom email for the user.

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The custom prompt template pattern is good for the following use cases:

- Building prompts to use with other patterns including document summarization, text generation, retrieval-augmented generation, and chatbots.
- Building custom content for a user based on their preferences.

### Architecture

![architecture](/images/2-Bedrock/bonus/O-3/architecture.png)

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/templates** folder, and open the file **templates_lib.py**

![open](/images/2-Bedrock/bonus/O-3/open.png)

2. Add the import statements.

   - These statements allow us to use LangChain to create custom prompt templates and call Bedrock.

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

   - This code builds a custom prompt from the adjective, noun, and verb parameters.

```python
def get_prompt(adjective, noun, verb):
    
    template = "Tell me a story about a {adjective} {noun} who loves to {verb}:"

    prompt_template = PromptTemplate.from_template(template) #this will automatically identify the input variables for the template

    prompt = prompt_template.format(noun=noun, adjective=adjective, verb=verb)
    
    return prompt
```

5. Add this function to call Bedrock.

   - This function passes the custom prompt to Bedrock.

```python
def get_text_response(adjective, noun, verb): #text-to-text client function
    llm = get_llm()
    
    prompt = get_prompt(adjective, noun, verb)
    
    return llm.invoke(prompt) #return a response to the prompt
```

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **templates_app.py**

2. Add the import statements.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import templates_lib as glib #reference to local lib script
```

3. Add the page title and configuration.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(page_title="Prompt Templates") #HTML title
st.title("Prompt Templates") #page title
```

4. Add the input elements.

   - We are creating the values to be used in the custom prompt.

```python
noun = st.text_input("Noun")
adjective = st.text_input("Adjective")
verb = st.text_input("Verb")
go_button = st.button("Go", type="primary") #display a primary button
```

5. Add the output elements.

   - We use the `if` block below to handle the button click. We display a spinner while the backing function is called, then write the output to the web page.

```python
if go_button: #code in this if block will be run when the button is clicked
    
    with st.spinner("Working..."): #show a spinner while the code in this with block runs
        response_content = glib.get_text_response(noun=noun, adjective=adjective, verb=verb) #call the model through the supporting library
        
        st.write(response_content) #display the response content
```

6. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/templates
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/templates
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run templates_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/bonus/O-3/app.png)

4. Try out some values and see the results.
   - Noun = `snail`;
   - Adjective = `fun`;
   - Verb = `surf`;

![app](/images/2-Bedrock/bonus/O-3/app-in-use.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a custom prompt formatter with Bedrock, LangChain, and Streamlit!
{{% /notice %}}