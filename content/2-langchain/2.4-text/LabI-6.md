---
title: "Lab I-6: Extracting JSON data from text"
date: 2024-07-01T16:12:03+07:00
draft: false
weight: 6
chapter : false
---

In this lab, we will build a JSON generator using Bedrock, LangChain, and Streamlit.

Text to JSON allows us to extract hierarchical data from unstructured content. For example, we can extract information from a customer's email, including any referenced products, employees mentioned, sentiment, account numbers, and anything else the model can detect based on our prompt.

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The text to JSON pattern is good for the following use cases:

- Email data extraction
- Call transcript data extraction
- Document data extraction

### Architecture

![architecture](/images/2-Bedrock/text/I-6/architecture.png)

From an architectural perspective, text-to-JSON is identical to text-to-text. We just need to safely attempt to convert the text returned from the LLM to JSON output.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/json** folder, and open the file **json_lib.py**

![lib](/images/2-Bedrock/text/I-6/1.png)

2. Add the import statements.

   - These statements allow us to use LangChain to call Bedrock, and safely attempt to convert the output to JSON.

```python
import json
from json import JSONDecodeError
from langchain_community.llms import Bedrock
```

3. Add a function to create a Bedrock LangChain client.

   - This includes the inference parameters we want to use.

```python
def get_llm():

    llm = Bedrock( #create a Bedrock llm client
        model_id="ai21.j2-ultra-v1", #use the AI21 Jurassic-2 Ultra model
        model_kwargs = {"maxTokens": 1024, "temperature": 0.0 } #for data extraction, minimum temperature is best
    )

    return llm
```

4. Add the function to attempt converting the text result to a JSON object.

   - This will allow us to gracefully handle a situation where the LLM doesn't generate valid JSON-formatted text.

```python
def validate_and_return_json(response_text):
    try:
        response_json = json.loads(response_text) #attempt to load text into JSON
        return False, response_json, None #returns has_error, response_content, err 
    
    except JSONDecodeError as err:
        return True, response_text, err #returns has_error, response_content, err 
```

5. Add this function to call Bedrock.

   - This code calls Bedrock and passes the response to the JSON converter.

```python
def get_json_response(input_content): #text-to-text client function
    
    llm = get_llm()

    response = llm.invoke(input_content) #the text response for the prompt
    
    return validate_and_return_json(response)
```

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **json_app.py**

2. Add the import statements.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import json_lib as glib #reference to local lib script
```

3. Add the page title, configuration, and two-column layout.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(page_title="Text to JSON", layout="wide")  #set the page width wider to accommodate columns

st.title("Text to JSON")  #page title

col1, col2 = st.columns(2)  #create 2 columns
```

4. Add the input elements.

   - We are creating a multiline text box and button to get the user's prompt and send it to Bedrock.

```python
with col1: #everything in this with block will be placed in column 1
    st.subheader("Prompt") #subhead for this column
    
    input_text = st.text_area("Input text", height=500, label_visibility="collapsed")

    process_button = st.button("Run", type="primary") #display a primary button
```

5. Add the output elements.

   - We use the if block below to handle the button click. We display a spinner while the backing function is called, then write the output to the web page. We display the formatted JSON.

```python
with col2: #everything in this with block will be placed in column 2
    st.subheader("Result") #subhead for this column
    
    if process_button: #code in this if block will be run when the button is clicked
        with st.spinner("Running..."): #show a spinner while the code in this with block runs
            has_error, response_content, err = glib.get_json_response(input_content=input_text) #call the model through the supporting library

        if not has_error:
            st.json(response_content) #render JSON if there was no error
        else:
            st.error(err) #otherwise render the error
            st.write(response_content) #and render the raw response from the model
```

6. Save the file.

## Run the Streamlit app

1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/json
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/json
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run json_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/text/I-6/2.png)

4. Try out some prompts and see the results.

   - `Return the first five presidents including their name, party, and term as an array in JSON format:`
    ```txt
    Dear Acme Investments,
    I am writing to bring to your attention a situation that I believe to be unethical on the part of one of your account managers, Roger Longbottom.
    I recently met with Roger to discuss my investment portfolio and was deeply concerned to hear that he suggested I invest in a certain stock. When I asked him why he thought this was a good investment, he stated that the stock was currently undervalued and was likely to increase in value in the near future.
    However, upon further research, I have discovered that the stock in question has a questionable reputation. It has been the subject of multiple lawsuits and has been found to have engaged in questionable business practices.
    I believe Roger was aware of these facts, but failed to disclose them to me. As a result, I feel I was misled into making an unwise investment decision.
    I therefore urge you to investigate whether Roger has acted unethically and take appropriate action if necessary.
    Yours sincerely,
    Carson Bradford

    Based on the above text, please return the following values in JSON format:
    Sentiment: Positive, Neutral, or Negative
    Category: Sales, Operations, Customer Service, or Fund Management
    Summary: (1-2 sentences summarizing the text)
    Concern: (Rate the level of concern for the above content on a scale from 1-10)
    Customer_Name: (Name of the customer)
    Employee_Name: (Name of the first employee mentioned in the text)
    ```

![app](/images/2-Bedrock/text/I-6/3.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a JSON generator app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}