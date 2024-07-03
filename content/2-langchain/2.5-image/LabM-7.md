---
title: "Lab M-7: Background replacement"
date: 2024-07-03T09:02:25+07:00
draft: false
weight: 7
chapter : false
---

In this lab, we will build an image background replacement application using Amazon Titan Image Generator, Amazon Bedrock, and Streamlit. We'll be using the Boto3 library to interact with Titan Image Generator.

Titan Image Generator supports mask prompting. This allows us to specify items to preserve from an image without having to know their exact dimensions. Titan Image Generator will automatically mask the specified items so that something new can be painted in the background.

See the illustrations below to understand the steps:

| 1. Start with the original image                  | 2. Specify the object to mask | 3. Specify what should be painted in the unmasked area | 4. Outpaint the unmasked area of the original image          |
| ------------------------------------------------- | ----------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| ![laptop](/images/2-Bedrock/image/M-7/laptop.png) | "Laptop"                      | "Laptop on a table at the beach"                       | ![latop beach](/images/2-Bedrock/image/M-7/laptop-beach.jpg) |
| ![bed](/images/2-Bedrock/image/M-7/br.jpg)        | "Bed"                         | "A stately bedroom"                                    | ![bedroom2](/images/2-Bedrock/image/M-7/br2.jpg)             |
| ![robots](/images/2-Bedrock/image/M-7/robots.png) | "Robots"                      | "A bright neon robot arena"                            | ![robots arena](/images/2-Bedrock/image/M-7/arena.jpg)       |
| *Uploaded file*                                   | *Mask prompt*                 | *Prompt text*                                          | *Outpainted image created with Titan Image Generator*        |

### Outpainting mode: Default vs Precise
Titan Image Generator supports two **outpainting modes**:

- **Default** mode allows the masked image to potentially extend into the new background. This is helpful when you have existing background elements within your mask that should be extended seamlessly, or when the masked item should blend together smoothly with the background.
- **Precise** mode prevents the masked image from extending into the new background. This is important when you have a product that cannot be extended - it just needs to be placed on a new background.

See the illustrations below to see the difference between default and precise outpainting modes:

| Outpainting Mode | Input image                                         | Prompt                                     | Result                                                              | Note                                                                                                                 |
| ---------------- | --------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Default          | ![flowers](/images/2-Bedrock/image/M-7/flowers.png) | "Flowers in the kitchen"                   | ![default flowers](/images/2-Bedrock/image/M-7/default-flowers.jpg) | Additional flowers expand into the background (good if this is for an article about placing flowers in your kitchen) |
| Precise          | ![flowers](/images/2-Bedrock/image/M-7/flowers.png) | "Flowers in the kitchen"                   | ![precise flowers](/images/2-Bedrock/image/M-7/precise-flowers.jpg) | No additional flowers created (good if product is a specific number of flowers)                                      |
| Default          | ![dress](/images/2-Bedrock/image/M-7/dress.png)     | "Woman in a long flowing dress, in a city" | ![default dress](/images/2-Bedrock/image/M-7/dress-default.jpg)     | Dress extends into the new background (good if this is for an article about city living)                             |
| Precise          | ![dress](/images/2-Bedrock/image/M-7/dress.png)     | "Woman in a long flowing dress, in a city" | ![precise dress](/images/2-Bedrock/image/M-7/dress-precise.jpg)     | Dress is kept as-is (good if this is a product photo of a specific dress)                                            |

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image background replacement pattern is good for the following use cases:

- Creating alternate scenery for product photos
- Removing distractions and clutter from the background of product photos
- Creating new marketing or article images using a person or object from another image

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. In AWS Cloud9, navigate to the **workshop/labs/image_background** folder, and open the file **image_background_lib.py**

![open](/images/2-Bedrock/image/M-7/open.png)

2. Add the import statements and the file and image processing helper functions.

   - These statements allow us to use the Boto3 library to call Bedrock, and process image data.
   - The helper functions are used to convert data between files, images, and base64-encoded bytes.

```python
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
def get_titan_image_background_replacement_request_body(prompt, image_bytes, mask_prompt, negative_prompt=None, outpainting_mode="DEFAULT"):
    
    input_image_base64 = get_base64_from_bytes(image_bytes)

    body = { #create the JSON payload to pass to the InvokeModel API
        "taskType": "OUTPAINTING",
        "outPaintingParams": {
            "image": input_image_base64,
            "text": prompt,  # Description of the background to generate
            "maskPrompt": mask_prompt,  # The element(s) to keep
            "outPaintingMode": outpainting_mode,  # "DEFAULT" softens the mask. "PRECISE" keeps it sharp.
        },
        "imageGenerationConfig": {
            "numberOfImages": 1,  # Number of variations to generate
            "quality": "premium",  # Allowed values are "standard" and "premium"
            "height": 512,
            "width": 512,
            "cfgScale": 8.0,
            "seed": randint(0, 100000),  # Use a random seed
        },
    }
    
    if negative_prompt:
        body['outPaintingParams']['negativeText'] = negative_prompt
    
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
def get_image_from_model(prompt_content, image_bytes, mask_prompt=None, negative_prompt=None, outpainting_mode="DEFAULT"):
    session = boto3.Session()

    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    body = get_titan_image_background_replacement_request_body(prompt_content, image_bytes, mask_prompt=mask_prompt, negative_prompt=negative_prompt, outpainting_mode=outpainting_mode) #mask prompt "objects to keep" prompt text "description of background to add"
    
    response = bedrock.invoke_model(body=body, modelId="amazon.titan-image-generator-v1", contentType="application/json", accept="application/json")
    
    output = get_titan_response_image(response)
    
    return output
```

5. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_background_app.py**

2. Copy and paste the Streamlit app code.

   - This sets up the needed elements to collect user input and pass the request to the corresponding library script.

```python
import streamlit as st
import image_background_lib as glib

st.set_page_config(layout="wide", page_title="Image Background")

st.title("Image Background")

col1, col2, col3 = st.columns(3)


with col1:
    uploaded_file = st.file_uploader("Select an image", type=['png', 'jpg'])
    
    if uploaded_file:
        uploaded_image_preview = glib.get_bytesio_from_bytes(uploaded_file.getvalue())
        st.image(uploaded_image_preview)
    else:
        st.image("images/example.jpg")

with col2:
    st.subheader("Image parameters")
    
    mask_prompt = st.text_input("Object to keep:", value="A car", help="The mask text")
    
    prompt_text = st.text_area("Description including the object to keep and background to add:", value="Car at the beach", height=100, help="The prompt text")
    
    negative_prompt = st.text_input("What should not be in the background:", help="The negative prompt")

    outpainting_mode = st.radio("Outpainting mode:", ["DEFAULT", "PRECISE"], horizontal=True)
    
    generate_button = st.button("Generate", type="primary")


with col3:
    st.subheader("Result")

    if generate_button:
        if uploaded_file:
            image_bytes = uploaded_file.getvalue()
        else:
            image_bytes = glib.get_bytes_from_file("images/example.jpg")
        
        with st.spinner("Drawing..."):
            generated_image = glib.get_image_from_model(
                prompt_content=prompt_text, 
                image_bytes=image_bytes,
                mask_prompt=mask_prompt,
                negative_prompt=negative_prompt,
                outpainting_mode=outpainting_mode,
            )
        
        st.image(generated_image)
```

3. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image_background
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image_background
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

   - We set `server.enableXsrfProtection=false` so that we can upload a file from the AWS Cloud9 preview.

```bash
streamlit run image_background_app.py --server.port 8080 --server.enableXsrfProtection=false
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-7/app.png)

4. Click the **Generate** button to see a variation of the pre-loaded image.

![app](/images/2-Bedrock/image/M-7/app-in-use.png)

5. Optionally, upload an image with dimensions between 256x256 and 1024x1024 and a resolution of 72 pixels per inch. Update the values in the **Object to keep** and **Description including the object to keep and background to add** fields as needed. Click the **Generate** button to see a variation of your image.

   - You can download sample images from the **/workshop/sample_images** folder in your Cloud9 environment.

6. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image background replacement app with Bedrock and Streamlit!
{{% /notice %}}