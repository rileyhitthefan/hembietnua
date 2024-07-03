---
title: "Lab M-5: Masking introduction"
date: 2024-07-03T09:02:18+07:00
draft: false
weight: 5
chapter : false
---

In this lab, we will build an image masking application using Amazon Titan Image Generator, Amazon Bedrock, and Streamlit. We'll be using the Boto3 library to interact with Titan Image Generator.

There are two key concepts to understand when editing images using Amazon Titan Image Generator: **masking** and **painting**.

### Masking
**Masking** is the process of marking a portion of an image to be redrawn or preserved. Titan Image Generator supports two masking methods:

An **image mask** is a special image file that indicates the pixels that should be masked in the original image. Black pixels indicate what is masked, and white pixels mark what is not masked. The image mask should have the following characteristics:

- The same dimensions and resolution as the original image file
- PNG file format
- Either RGB or grayscale
- No alpha channel

A **mask prompt** is a way to dynamically mask an image by using a prompt. The prompt can be one or more words that describes what to mask. Behind the scenes, Titan Image Generator will automatically create a mask based on that prompt.

### Painting
Painting is the creation of new imagery using an image generation model. There are two modes of painting available with Titan Image Generator:

**Inpainting** is the process of repainting all of the pixels within the masked area of an image. When using an image mask, all of the black pixels will be repainted. When using a mask prompt, the items indicated in the mask prompt will be repainted.

**Outpainting** is the process of painting all the pixels outside of the masked area of an image. When using an image mask, all of the white pixels will be repainted. When using a mask prompt, the items indicated in the mask prompt will be preserved, and everything else will be repainted.

See the illustrations below to understand the different masking and painting modes:

### Image mask examples
| Mode                   | 1. Start with the original image                | 2. Specify the image mask                             | 3. Specify what should be painted | 4. Generate the edited image                                   |
| ---------------------- | ----------------------------------------------- | ----------------------------------------------------- | --------------------------------- | -------------------------------------------------------------- |
| Image mask inpainting  | ![dinos](/images/2-Bedrock/image/M-5/dinos.png) | ![dinomask](/images/2-Bedrock/image/M-5/dinomask.png) | "Easter Basket"                   | ![easter inpaint](/images/2-Bedrock/image/M-5/easter-in.jpg)   |
| Image mask outpainting | ![dinos](/images/2-Bedrock/image/M-5/dinos.png) | ![dinos](/images/2-Bedrock/image/M-5/dinos.png)       | "Easter Basket"                   | ![easter outpaint](/images/2-Bedrock/image/M-5/easter-out.jpg) |
|                        | *Uploaded file*                                 | *Image mask*                                          | *Prompt text*                     | *Image created with Titan Image Generator*                     |

### Mask prompt examples
| Mode                    | 1. Start with the original image                               | 2. Specify the mask prompt | 3. Specify what should be painted | 4. Generate the edited image                                      |
| ----------------------- | -------------------------------------------------------------- | -------------------------- | --------------------------------- | ----------------------------------------------------------------- |
| Mask prompt inpainting  | ![food](/images/2-Bedrock/image/M-5/food.png)                  | "Bowl of salsa"            | "Bowl of black beans"             | ![food with beans](/images/2-Bedrock/image/M-5/with-beans.jpg)    |
| Mask prompt outpainting | ![socks on table](/images/2-Bedrock/image/M-5/socks-table.png) | "Socks"                    | "Socks on a granite countertop"   | ![easter outpaint](/images/2-Bedrock/image/M-5/socks-granite.jpg) |
|                         | *Uploaded file*                                                | *Image mask*               | *Prompt text*                     | *Image created with Titan Image Generator*                        |

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image masking pattern is good for the following use cases:

- Adding items to an image
- Removing undesired items from an image
- Changing the background of an image
- Extending an image

We'll go into each of these use cases more deeply over the next few labs.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. In AWS Cloud9, navigate to the **workshop/labs/image_masking** folder, and open the file **image_masking_lib.py**

![open](/images/2-Bedrock/image/M-5/open.png)

2. Add the import statements and the file and image processing helper functions.

   - These statements allow us to use the Boto3 library to call Bedrock, and process image data.

   - The helper functions are used to convert data between files, images, and base64-encoded bytes.

```python
import boto3
import json
import base64
from PIL import Image
from io import BytesIO
from random import randint


def get_bytesio_from_bytes(image_bytes):
    image_io = BytesIO(image_bytes)
    return image_io


def get_base64_from_bytes(image_bytes):
    resized_io = get_bytesio_from_bytes(image_bytes)
    img_str = base64.b64encode(resized_io.getvalue()).decode("utf-8")
    return img_str


def get_image_from_bytes(image_bytes):
    image_io = BytesIO(image_bytes)
    image = Image.open(image_io)
    return image
    

def get_png_base64(image):
    png_io = BytesIO()
    image.save(png_io, format="PNG")
    img_str = base64.b64encode(png_io.getvalue()).decode("utf-8")
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
def get_titan_image_masking_request_body(prompt_content, image_bytes, painting_mode, masking_mode, mask_bytes, mask_prompt):
    
    original_image = get_image_from_bytes(image_bytes)
    
    target_width, target_height = original_image.size
    
    image_base64 = get_base64_from_bytes(image_bytes)
    
    mask_base64 = get_base64_from_bytes(mask_bytes)
    
    body = {
        "taskType": painting_mode,
        "imageGenerationConfig": {
            "numberOfImages": 1,  # Number of variations to generate
            "quality": "premium",  # Allowed values are "standard" and "premium"
            "height": target_height,
            "width": target_width,
            "cfgScale": 8.0,
            "seed": randint(0, 100000),  # Use a random seed
        },
    }
    
    params = {
        "image": image_base64,
        "text": prompt_content,  # What should be displayed in the final image
    }
    
    if masking_mode == 'Image':
        params['maskImage'] = mask_base64
    else:
        params['maskPrompt'] = mask_prompt
        
    
    if painting_mode == 'OUTPAINTING':
        params['outPaintingMode'] = 'DEFAULT'
        body['outPaintingParams'] = params
    else:
        body['inPaintingParams'] = params
    
    return json.dumps(body)
```

4. Add the functions to call Bedrock and extract the image from the response.

   - The `get_titan_response_image` function extracts the image data from the returned payload and converts it to a format that Streamlit can use.
   - We're creating a function we can call from the Streamlit front end application. This function passes the input content to Bedrock and returns the image.

```python
def get_titan_response_image(response):

    response = json.loads(response.get('body').read())
    
    images = response.get('images')
    
    image_data = base64.b64decode(images[0])

    return BytesIO(image_data)


def get_image_from_model(prompt_content, image_bytes, painting_mode, masking_mode, mask_bytes=None, mask_prompt=None):
    session = boto3.Session()
    
    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    body = get_titan_image_masking_request_body(prompt_content, image_bytes, painting_mode, masking_mode, mask_bytes, mask_prompt)
    
    response = bedrock.invoke_model(body=body, modelId="amazon.titan-image-generator-v1", contentType="application/json", accept="application/json")
    
    output = get_titan_response_image(response)
    
    return output
```

5. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_masking_app.py**

2. Copy and paste the Streamlit app code.

   - This sets up the needed elements to collect user input and pass the request to the corresponding library script.

```python
import streamlit as st
import image_masking_lib as glib


st.set_page_config(layout="wide", page_title="Image Masking")

st.title("Image Masking")


col1, col2, col3 = st.columns(3)

with col1:
    st.subheader("Image")
    uploaded_image_file = st.file_uploader("Select an image", type=['png', 'jpg'])
    
    if uploaded_image_file:
        uploaded_image_preview = glib.get_bytesio_from_bytes(uploaded_image_file.getvalue())
        st.image(uploaded_image_preview)
    else:
        st.image("images/desk1.jpg")
    
    
with col2:
    st.subheader("Mask")
    
    masking_mode = st.radio("Masking mode:", ["Image", "Prompt"], horizontal=True)
    
    if masking_mode == 'Image':
    
        uploaded_mask_file = st.file_uploader("Select an image mask", type=['png', 'jpg'])
        
        if uploaded_mask_file:
            uploaded_mask_preview = glib.get_bytesio_from_bytes(uploaded_mask_file.getvalue())
            st.image(uploaded_mask_preview)
        else:
            st.image("images/mask1.png")
    else:
        mask_prompt = st.text_input("Item to mask:", help="The item to replace (if inpainting), or keep (if outpainting).")
    
        
with col3:
    st.subheader("Result")
    
    prompt_text = st.text_area("Prompt text:", height=100, help="The prompt text")

    painting_mode = st.radio("Painting mode:", ["INPAINTING", "OUTPAINTING"])
    
    generate_button = st.button("Generate", type="primary")

    if generate_button:
        with st.spinner("Drawing..."):
            
            if uploaded_image_file:
                image_bytes = uploaded_image_file.getvalue()
            else:
                image_bytes = glib.get_bytes_from_file("images/desk1.jpg")
            
            if masking_mode == 'Image':
                if uploaded_mask_file:
                    mask_bytes = uploaded_mask_file.getvalue()
                else:
                    mask_bytes = glib.get_bytes_from_file("images/mask1.png")
                
                generated_image = glib.get_image_from_model(
                    prompt_content=prompt_text, 
                    image_bytes=image_bytes,
                    masking_mode=masking_mode,
                    mask_bytes=mask_bytes,
                    painting_mode=painting_mode
                )
            else:
                generated_image = glib.get_image_from_model(
                    prompt_content=prompt_text, 
                    image_bytes=image_bytes,
                    masking_mode=masking_mode,
                    mask_prompt=mask_prompt,
                    painting_mode=painting_mode
                )
            
        
        st.image(generated_image)
```

3. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image_masking
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image_masking
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

   - We set `server.enableXsrfProtection=false` so that we can upload a file from the AWS Cloud9 preview.

```bash
streamlit run image_masking_app.py --server.port 8080 --server.enableXsrfProtection=false
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-5/app.png)

4. Set a value in the **Prompt text** field, and click the **Generate** button to see a repainted version of the pre-loaded image.

   - **Prompt text** `Framed photographs over a desk, with a stool under the desk`

![app](/images/2-Bedrock/image/M-5/app-in-use-image-in.png)

5. Set **Painting mode** to **OUTPAINTING**. Set a value in the **Prompt text** field, and click the **Generate** button to see a repainted version of the pre-loaded image.
   - **Prompt text** `A cozy living room`

![app](/images/2-Bedrock/image/M-5/app-in-use-image-out.png)

6. Set **Masking mode** to **Prompt**. Set a value for **Item to mask**. Set the painting mode to **INPAINTING**. Set a value in the **Prompt text** field, and click the **Generate** button to see a repainted version of the pre-loaded image.
   - **Item to mask** `painting`
   - **Prompt text** `embedded fireplace`

![app](/images/2-Bedrock/image/M-5/app-in-use-prompt-in.png)

7. Set **Masking mode** to **Prompt**. Set a value for **Item to mask**. Set the painting mode to **OUTPAINTING**. Set a value in the **Prompt text** field, and click the **Generate** button to see a repainted version of the pre-loaded image.
   - **Item to mask** `painting`
   - **Prompt text** `living room`

![app](/images/2-Bedrock/image/M-5/app-in-use-prompt-out.png)

8. Optionally, upload an image and image mask with [supported dimensions](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-titan-image.html)  and a resolution of 72 pixels per inch.

9. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image masking app with Bedrock and Streamlit!
{{% /notice %}}