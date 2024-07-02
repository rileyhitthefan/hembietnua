---
title: "Lab I-7: Extracting CSV data from text"
date: 2024-07-01T16:12:12+07:00
draft: false
weight: 7
chapter : false
---

In this lab, we will build a CSV generator using Bedrock, LangChain, and Streamlit.

Text to CSV allows us to extract tabular data from unstructured content. For example, we can extract information from a customer's email, including any referenced products, employees mentioned, sentiment, account numbers, and anything else the model can detect based on our prompt.

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The text to CSV pattern is good for the following use cases:

- Email data extraction
- Call transcript data extraction
- Document data extraction
- Sample dataset generation

### Architecture

![architecture](/images/2-Bedrock/text/I-7/architecture.png)

From an architectural perspective, text-to-CSV is identical to text-to-text. We just need to safely attempt to convert the text returned from the LLM to CSV output.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/csv** folder, and open the file **csv_lib.py**

![lib](/images/2-Bedrock/text/I-7/1.png)

2. Add the import statements.

   - These statements allow us to use LangChain to call Bedrock, and safely attempt to convert the output to a CSV.

```python
import pandas as pd
from io import StringIO
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

4. Add the function to attempt converting the text result to a Pandas dataframe.

   - This will allow us to gracefully handle a situation where the LLM doesn't generate valid CSV-formatted text.

```python
def validate_and_return_csv(response_text):
    #returns has_error, response_content, err 
    try:
        csv_io = StringIO(response_text)
        return False, pd.read_csv(csv_io), None #attempt to load response CSV into a dataframe
    
    except Exception as err:
        return True, response_text, err
```

5. Add this function to call Bedrock.
   - This code calls Bedrock and passes the response to the CSV converter.

```python
def get_csv_response(input_content): #text-to-text client function
    
    llm = get_llm()

    response = llm.invoke(input_content) #the text response for the prompt
    
    return validate_and_return_csv(response)
```

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **csv_app.py**

2. Add the import statements.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import csv_lib as glib #reference to local lib script
```

3. Add the page title, configuration, and two-column layout.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(page_title="Text to CSV", layout="wide")  #set the page width wider to accommodate columns

st.title("Text to CSV")  #page title

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

   - We use the `if` block below to handle the button click. We display a spinner while the backing function is called, then write the output to the web page. We display the dataframe as a table, along with the raw CSV content.

```python
with col2: #everything in this with block will be placed in column 2
    st.subheader("Result") #subhead for this column
    
    if process_button: #code in this if block will be run when the button is clicked
        with st.spinner("Running..."): #show a spinner while the code in this with block runs
            has_error, response_content, err = glib.get_csv_response(input_content=input_text) #call the model through the supporting library
        
        if not has_error:
            st.dataframe(response_content)
            
            csv_content = response_content.to_csv(index = False)
            
            st.markdown("#### Raw CSV")
            st.text(csv_content)
            
        else:
            st.error(err)
            st.write(response_content)
```

6. Save the file.

## Run the Streamlit app

1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/csv
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/csv
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run csv_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/text/I-7/2.png)

4. Try out some prompts and see the results.

   - `Return the first five presidents including their name, party, and term in CSV format:`

    ```txt
    Robert needs to produce a TPS report for Sherry by Thursday.

    Jane, can you please build the initial proof of concept? I'd like to see it by October 1st.

    I know we have other things to do, but those two things should be top priority.

    Oh yeah - Nilesh, can you please send Dagmar the latest summary by next week? That's not as urgent. Thank you!



    Based on the above text, please return a list of action items in CSV format, with columns for "Action Item", "Owner", "Priority", and "Due Date". Do not add any note, just return the CSV content with one row per action item:

    ```

![app](/images/2-Bedrock/text/I-7/3.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a CSV generator app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}
