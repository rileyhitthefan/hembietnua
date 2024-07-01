---
title: "Lab B-2: Image generation"
date: 2024-07-01T13:20:44+07:00
draft: false
weight: 2
chapter : false
---

In this lab, we will build a basic image generator using **Stable Diffusion**, **Amazon Bedrock**, and **Streamlit**. We'll be using the **Boto3** library to interact with Stable Diffusion, since LangChain primarily supports text generation models.

**Stable Diffusion** creates images from a text prompt. It starts with random noise and gradually forms an image over a series of steps. It can also be used to transform existing images based on a prompt, which we will try in a later lab.

{{% notice info %}}
Your account will need to be subscribed to Stable Diffusion models in order to run this lab. Enable **Stability AI** > **SDXL 1.0** in the **Model access** page in the Amazon Bedrock console (see [these instructions](../2.1-prep/bedrockSetup.md) again)
{{% /notice %}}

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image generation pattern is good for the following use cases:

- Creation of custom imagery for websites, emails, etc.
- Creation of concept art for various media forms

### Architecture
![Architecture](/images/2-Bedrock/basic/B-2/architecture.png)

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/image** folder, and open the file **image_lib.py**

![lib](/images/2-Bedrock/basic/B-2/1.png)

2. Add the import statements.
   - These statements allow us to use the Boto3 library to call Bedrock, read environment variables, and process image data.

```py
import boto3 #import aws sdk and supporting libraries
import json
import base64
from io import BytesIO
```

3. Initialize the Boto3 client and Bedrock model id.

```py
session = boto3.Session()

bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client

bedrock_model_id = "stability.stable-diffusion-xl-v1" #use the Stable Diffusion model
```

4. Add the image conversion function.
   - This function extracts the image data from the returned payload and converts it to a format that Streamlit can use.

```py
def get_response_image_from_payload(response): #returns the image bytes from the model response payload

    payload = json.loads(response.get('body').read()) #load the response body into a json object
    images = payload.get('artifacts') #extract the image artifacts
    image_data = base64.b64decode(images[0].get('base64')) #decode image

    return BytesIO(image_data) #return a BytesIO object for client app consumption
```

5. Add this function to call Bedrock.
   - We're creating a function we can call from the Streamlit front end application. This function passes the input content to Bedrock and returns the image.

```python
def get_image_response(prompt_content): #text-to-text client function
    
    request_body = json.dumps({"text_prompts": 
                               [ {"text": prompt_content } ], #prompts to use
                               "cfg_scale": 9, #how closely the model tries to match the prompt
                               "steps": 50, }) #number of diffusion steps to perform
    
    response = bedrock.invoke_model(body=request_body, modelId=bedrock_model_id) #call the Bedrock endpoint
    
    output = get_response_image_from_payload(response) #convert the response payload to a BytesIO object for the client to consume
    
    return output
```

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_app.py**

2. Add the import statements.
   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
import image_lib as glib #reference to local lib script
```

3. Add the page title, configuration, and column layout.
   - Here we are setting the page title on the actual page and the title shown in the browser tab. We're creating two columns to collect the input on the left and display the output on the right.

```python
st.set_page_config(layout="wide", page_title="Image Generation") #set the page width wider to accommodate columns

st.title("Image Generation") #page title

col1, col2 = st.columns(2) #create 2 columns
```

4. Add the input elements.
   - In the first column, we are creating a multiline text box and button to get the user's prompt and send it to Bedrock.

```py
with col1: #everything in this with block will be placed in column 1
    st.subheader("Image generation prompt") #subhead for this column
    
    prompt_text = st.text_area("Prompt text", height=200, label_visibility="collapsed") #display a multiline text box with no label
    
    process_button = st.button("Run", type="primary") #display a primary button
```

5. Add the output elements.
   - In the second column, we use the if block to handle the button click. We display a spinner while the backing function is called, then write the output to the web page.

```python
with col2: #everything in this with block will be placed in column 2
    st.subheader("Result") #subhead for this column
    
    if process_button: #code in this if block will be run when the button is clicked
        with st.spinner("Drawing..."): #show a spinner while the code in this with block runs
            generated_image = glib.get_image_response(prompt_content=prompt_text) #call the model through the supporting library
        
        st.image(generated_image) #display the generated image
```

6. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run image_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![App](/images/2-Bedrock/basic/B-2/3.png)

4. Try out some prompts and see the results.

   - `A painting of a person holding a cat, in the style of Rembrandt`
   - `A painting of a person holding a cat, in the style of Picasso`

![App Results](/images/2-Bedrock/basic/B-2/4.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image generation app with Bedrock, LangChain, and Streamlit!
{{% /notice %}}