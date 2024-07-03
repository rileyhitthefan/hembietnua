---
title: "Lab M-10: Image understanding"
date: 2024-07-03T09:02:34+07:00
draft: false
weight: 10
chapter : false
---

In this lab, we will build an image understanding application using Anthropic Claude 3, Amazon Bedrock, and Streamlit. We'll be using the Boto3 library to interact with Anthropic Claude.

The image understanding pattern requires two inputs:

1. One or more images to analyze.
2. A prompt to describe what you want to know about the images.

We'll build the image understanding example using Anthropic Claude's [vision capabilities](https://docs.anthropic.com/claude/docs/use-cases-and-capabilities#vision-capabilities).

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image understanding pattern is good for the following use cases:

- Generate accessible alternate text and image captions
- Use in an image processing pipeline to detect if an image is inappropriate or a mismatch for its related content
- Use in an image processing pipeline to automate mask prompting and inpainting/outpainting actions
- Categorize images
- Extract text from images
- [Many more](https://docs.anthropic.com/claude/docs/use-cases-and-capabilities#vision-capabilities)

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. In AWS Cloud9, navigate to the **workshop/labs/image_understanding** folder, and open the file **image_understanding_lib.py**

![open](/images/2-Bedrock/image/M-10/open.png)

2. Add the import statements and the file and image processing helper functions.

   - These statements allow us to use the Boto3 library to call Bedrock, and process image data.

   - The helper functions are used to convert data between files, images, and base64-encoded bytes.

```python
import boto3
import json
import base64
from io import BytesIO


#get a BytesIO object from file bytes
def get_bytesio_from_bytes(image_bytes):
    image_io = BytesIO(image_bytes)
    return image_io


#get a base64-encoded string from file bytes
def get_base64_from_bytes(image_bytes):
    resized_io = get_bytesio_from_bytes(image_bytes)
    img_str = base64.b64encode(resized_io.getvalue()).decode("utf-8")
    return img_str


#load the bytes from a file on disk
def get_bytes_from_file(file_path):
    with open(file_path, "rb") as image_file:
        file_bytes = image_file.read()
    return file_bytes
```

3. Add the request body builder function.

   - This function prepares the request payload for submission to Bedrock
   - We create two messages: one for the encoded image, the other for the prompt about the image

```python
#get the stringified request body for the InvokeModel API call
def get_image_understanding_request_body(prompt, image_bytes=None, mask_prompt=None, negative_prompt=None):
    input_image_base64 = get_base64_from_bytes(image_bytes)
    
    body = {
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 2000,
        "temperature": 0,
        "messages": [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image",
                        "source": {
                            "type": "base64",
                            "media_type": "image/jpeg", #this doesn't seem to matter?
                            "data": input_image_base64,
                        },
                    },
                    {
                        "type": "text",
                        "text": prompt
                    }
                ],
            }
        ],
    }
    
    return json.dumps(body)
```

4. Add a function to call Bedrock and return the response.

   - We're creating a function we can call from the Streamlit front end application. This function passes the input content to Bedrock and returns the text.

```python
#generate a response using Anthropic Claude
def get_response_from_model(prompt_content, image_bytes, mask_prompt=None):
    session = boto3.Session()
    
    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    body = get_image_understanding_request_body(prompt_content, image_bytes, mask_prompt=mask_prompt)
    
    response = bedrock.invoke_model(body=body, modelId="anthropic.claude-3-sonnet-20240229-v1:0", contentType="application/json", accept="application/json")
    
    response_body = json.loads(response.get('body').read()) # read the response
    
    output = response_body['content'][0]['text']
    
    return output
```

5. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_understanding_app.py**

2. Copy and paste the Streamlit app code.

   - This sets up the needed elements to collect user input and pass the request to the corresponding library script.

```python
import streamlit as st
import image_understanding_lib as glib

st.set_page_config(layout="wide", page_title="Image Understanding")

st.title("Image Understanding")

col1, col2, col3 = st.columns(3)

prompt_options_dict = {
    "Image caption": "Please provide a brief caption for this image.",
    "Detailed description": "Please provide a thoroughly detailed description of this image.",
    "Image classification": "Please categorize this image into one of the following categories: People, Food, Other. Only return the category name.",
    "Object recognition": "Please create a comma-separated list of the items found in this image. Only return the list of items.",
    "Subject identification": "Please name the primary object in the image. Only return the name of the object in <object> tags.",
    "Writing a story": "Please write a fictional short story based on this image.",
    "Answering questions": "What emotion are the people in this image displaying?",
    "Transcribing text": "Please transcribe any text found in this image.",
    "Translating text": "Please translate the text in this image to French.",
    "Other": "",
}

prompt_options = list(prompt_options_dict)

image_options_dict = {
    "Food": "images/food.jpg",
    "People": "images/people.jpg",
    "Person and cat": "images/person_and_cat.jpg",
    "Room": "images/room.jpg",
    "Text in image": "images/text2.png",
    "Toy": "images/toy_car.jpg",
    "Other": "images/house.jpg",
}

image_options = list(image_options_dict)


with col1:
    st.subheader("Select an Image")
    
    image_selection = st.radio("Image example:", image_options)
    
    if image_selection == 'Other':
        uploaded_file = st.file_uploader("Select an image", type=['png', 'jpg'], label_visibility="collapsed")
    else:
        uploaded_file = None
    
    if uploaded_file and image_selection == 'Other':
        uploaded_image_preview = glib.get_bytesio_from_bytes(uploaded_file.getvalue())
        st.image(uploaded_image_preview)
    else:
        st.image(image_options_dict[image_selection])
    
    
with col2:
    st.subheader("Prompt")
    
    prompt_selection = st.radio("Prompt example:", prompt_options)
    
    prompt_example = prompt_options_dict[prompt_selection]
    
    prompt_text = st.text_area("Prompt",
        #value=,
        value=prompt_example,
        height=100,
        help="What you want to know about the image.",
        label_visibility="collapsed")
    
    go_button = st.button("Go", type="primary")
    
    
with col3:
    st.subheader("Result")

    if go_button:
        with st.spinner("Processing..."):
            
            if uploaded_file:
                image_bytes = uploaded_file.getvalue()
            else:
                image_bytes = glib.get_bytes_from_file(image_options_dict[image_selection])
            
            response = glib.get_response_from_model(
                prompt_content=prompt_text, 
                image_bytes=image_bytes,
            )
        
        st.write(response)
```

3. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image_understanding
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image_understanding
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

   - We set `server.enableXsrfProtection=false` so that we can upload a file from the AWS Cloud9 preview.

```bash
streamlit run image_understanding_app.py --server.port 8080 --server.enableXsrfProtection=false
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-10/app.png)

4. Set the **Image example** and **Prompt example** to test different capabilities. Click the **Go** button to see the generated response.

![app](/images/2-Bedrock/image/M-10/app-in-use.png)

5. Optionally, select **Image example: Other** and upload an image. You can also try a custom prompt. Click the **Go** button to see the response.

6. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image understanding app with Amazon Bedrock and Streamlit!
{{% /notice %}}