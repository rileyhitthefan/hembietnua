---
title: "Lab M-4: Image style mixing"
date: 2024-07-03T09:02:16+07:00
draft: false
weight: 4
chapter : false
---

In this lab, we will build an image style mixing application using Amazon Titan Image Generator, Amazon Bedrock, and Streamlit. We'll be using the Boto3 library to interact with Titan Image Generator.

Titan Image Generator can take several images and create variations based on those images. This can be helpful when you have images with the elements you want, but want to create some alternative options with combined styles or elements.

| 1. First image                                 | 2. Second image                              | 3. Briefly describe the variation to generate | 4. Generate the image variation                                       |
| ---------------------------------------------- | -------------------------------------------- | --------------------------------------------- | --------------------------------------------------------------------- |
| ![bedroom](/images/2-Bedrock/image/M-4/br.jpg) | ![cat](/images/2-Bedrock/image/M-4/cat.jpg)  | "Bedroom with cat"                            | ![bedroom with cat](/images/2-Bedrock/image/M-4/bedroom-with-cat.jpg) |
| ![img1](/images/2-Bedrock/image/M-4/im1.jpg)   | ![img2](/images/2-Bedrock/image/M-4/im2.jpg) | "Blend the images"                            | ![blend](/images/2-Bedrock/image/M-4/blend-the-images.jpg)            |
| *Uploaded file*                                | *Uploaded file*                              | *Variation prompt*                            | *Image variation created with Titan Image Generator*                  |

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image style mixing pattern is good for the following use cases:

- Creating alternate versions of images for marketing content, stock photography use cases, and concept art.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. In AWS Cloud9, navigate to the **workshop/labs/image_style_mixing** folder, and open the file **image_style_mixing_lib.py**

![open](/images/2-Bedrock/image/M-4/open.png)

2. Add the import statements and the file and image processing helper functions.

   - These statements allow us to use the Boto3 library to call Bedrock, and process image data.

   - The helper functions are used to convert data between files, images, and base64-encoded bytes.

```python
import os
import boto3
import json
import base64
from io import BytesIO
from random import randint


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

   - This function prepares the request payload for submission to Bedrock:

```python
#get the stringified request body for the InvokeModel API call
def get_titan_image_variation_request_body(prompt, similarity, image_bytes1 = None, image_bytes2 = None):
    
    input_image1_base64 = get_base64_from_bytes(image_bytes1)
    input_image2_base64 = get_base64_from_bytes(image_bytes2)
    
    body = { #create the JSON payload to pass to the InvokeModel API
        "taskType": "IMAGE_VARIATION",
        "imageVariationParams": {
            "images": [ input_image1_base64, input_image2_base64 ],  
            "text": prompt,  
            "similarityStrength": similarity,
        },
        "imageGenerationConfig": {
            "numberOfImages": 1,  # Number of variations to generate
            "quality": "premium",  # Allowed values are "standard" or "premium"
            "height": 512,
            "width": 512,
            "cfgScale": 8.0,
            "seed": randint(0, 100000),  # Use a random seed
        },
    }
    
    return json.dumps(body)
```

4. Add the functions to call Bedrock and extract the image from the response.

   - The `get_titan_response_image` function extracts the image data from the returned payload and converts it to a format that Streamlit can use.
   - We're creating a function we can call from the Streamlit front end application. This function passes the input content to Bedrock and returns the image.

```python
#get a BytesIO object from the Titan Image Generator response
def get_titan_response_image(response):

    response = json.loads(response.get('body').read())
    
    images = response.get('images')
    
    image_data = base64.b64decode(images[0])

    return BytesIO(image_data)


#generate an image using Amazon Titan Image Generator
def get_image_from_model(prompt_content, similarity_strength, image_bytes1, image_bytes2):
    session = boto3.Session()

    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    body = get_titan_image_variation_request_body(prompt_content, similarity_strength, image_bytes1, image_bytes2) #prompt text hardcode since it doesn't matter
    
    response = bedrock.invoke_model(body=body, modelId="amazon.titan-image-generator-v1", contentType="application/json", accept="application/json")
    
    output = get_titan_response_image(response)
    
    return output
```

5. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_style_mixing_app.py**
 

2. Copy and paste the Streamlit app code.

   - This sets up the needed elements to collect user input and pass the request to the corresponding library script.

```python
import streamlit as st
import image_style_mixing_lib as glib


st.set_page_config(layout="wide", page_title="Image Style Mixing")

st.title("Image Style Mixing")

col1, col2, col3, col4 = st.columns(4)


with col1:
    st.subheader("Image One")
    
    uploaded_file1 = st.file_uploader("Select image 1", type=['png', 'jpg'])
    
    if uploaded_file1:
        uploaded_image_preview = glib.get_bytesio_from_bytes(uploaded_file1.getvalue())
        st.image(uploaded_image_preview)
    else:
        st.image("images/art_example.png")

with col2:
    st.subheader("Image Two")
    
    uploaded_file2 = st.file_uploader("Select image 2", type=['png', 'jpg'])
    
    if uploaded_file2:
        uploaded_image_preview = glib.get_bytesio_from_bytes(uploaded_file2.getvalue())
        st.image(uploaded_image_preview)
    else:
        st.image("images/cat_example.png")

with col3:  
    st.subheader("Options")

    prompt_text = st.text_input("Brief description of the target image:", value="a cat using art style", help="The prompt text")
    similarity_strength = st.slider("Similarity strength", min_value=0.2, max_value=1.0, value=0.9, step=0.1, help="How similar the generated image should be to the original image", format='%.1f')

    generate_button = st.button("Generate", type="primary")    

with col4:
    st.subheader("Result")

    if generate_button:

        if uploaded_file1:
            image_bytes1 = uploaded_file1.getvalue()
        else:
            image_bytes1 = glib.get_bytes_from_file("images/art_example.png")
            
        if uploaded_file2:
            image_bytes2 = uploaded_file2.getvalue()
        else:
            image_bytes2 = glib.get_bytes_from_file("images/cat_example.png")
        
        
        with st.spinner("Drawing..."):
            generated_image = glib.get_image_from_model(
                prompt_content=prompt_text,
                similarity_strength=float(similarity_strength),
                image_bytes1=image_bytes1,
                image_bytes2=image_bytes2,
            )
        
        st.image(generated_image)
```

3. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image_style_mixing
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image_style_mixing
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

   - We set `server.enableXsrfProtection=false` so that we can upload a file from the AWS Cloud9 preview.

```bash
streamlit run image_style_mixing_app.py --server.port 8080 --server.enableXsrfProtection=false
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-4/app.png)

4. Click the **Generate** button to see a variation of the pre-loaded images.

![app](/images/2-Bedrock/image/M-4/app-in-use.png)

5. Optionally, upload images with dimensions between 256x256 and 1024x1024 and a resolution of 72 pixels per inch. The prompt should briefly describe the image variation to be generated. Click the **Generate** button to see a variation of your images.

   - You can download sample images from the **/workshop/sample_images** folder in your Cloud9 environment.

6. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image style mixing app with Bedrock and Streamlit!
{{% /notice %}}