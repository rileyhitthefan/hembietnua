---
title: "Lab M-9: Image extension"
date: 2024-07-03T09:02:31+07:00
draft: false
weight: 9
chapter : false
---

In this lab, we will build an image extension application using Amazon Titan Image Generator, Amazon Bedrock, and Streamlit. We'll be using the Boto3 library to interact with Titan Image Generator.

The image extension pattern is a special case of outpainting. By default, Titan Image Generator outpainting works by taking an existing image, identifying a masked area to preserve, and repainting everything outside that masked area. To extend an image, we will need to programmatically resize the image and generate an image mask.

See the illustration below to understand the steps:

| 1. Start with the original image                             | 2. Place the original image within a larger blank image | 3. Generate a mask for the original image area | 4. Specify what should be outpainted in the extended area | 5. Outpaint the blank area of the larger image         |
| ------------------------------------------------------------ | ------------------------------------------------------- | ---------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------ |
| ![origninal](/images/2-Bedrock/image/M-9/image-original.png) | ![canvas](/images/2-Bedrock/image/M-9/image-canvas.png)    | ![app](/images/2-Bedrock/image/M-9/mask.png)   | "A laptop in front of Christmas decorations"              | ![app](/images/2-Bedrock/image/M-9/image-extended.png) |
| Uploaded file                                                | Programmaticaly created larger image                    | Programmatically created image mask            | Prompt text                                               | Extended image created with Titan Image Generator      |

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image extension pattern is good for the following use cases:

- Extending a product or marketing image to fit different dimension requirements

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. In AWS Cloud9, navigate to the **workshop/labs/image_extension** folder, and open the file **image_extension_lib.py**

![open](/images/2-Bedrock/image/M-9/open.png)

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


#get a BytesIO object from file bytes
def get_bytesio_from_bytes(image_bytes):
    image_io = BytesIO(image_bytes)
    return image_io


#get a base64-encoded PNG image from an Image object
def get_png_base64(image):
    png_io = BytesIO()
    image.save(png_io, format="PNG")
    img_str = base64.b64encode(png_io.getvalue()).decode("utf-8")
    return img_str


#get an Image object from file bytes
def get_image_from_bytes(image_bytes):
    
    image_io = BytesIO(image_bytes)
    
    image = Image.open(image_io)
    
    return image


#load the bytes from a file on disk
def get_bytes_from_file(file_path):
    with open(file_path, "rb") as image_file:
        file_bytes = image_file.read()
    return file_bytes
```

3. Add the image mask creation function.

   - This function creates a custom image mask based on the desired dimensions for inpainting and outpainting.
   - In this example, we are creating rectangular mask areas, but in your use case the mask can be made from any combination of black or white pixels based on the original image.

```python
#Get an inpainting or outpainting mask for the provided dimensions
def get_mask_image_base64(target_width, target_height, position, inside_width, inside_height):
    
    inside_color_value = (0, 0, 0) #inside is black - this is the masked area
    outside_color_value = (255, 255, 255)
    
    mask_image = Image.new("RGB", (target_width, target_height), outside_color_value)
    original_image_shape = Image.new(
        "RGB", (inside_width, inside_height), inside_color_value
    )
    mask_image.paste(original_image_shape, position)
    mask_image_base64 = get_png_base64(mask_image)
    
    #mask_image.save("mask.png") #uncomment this to see what the mask looks like

    return mask_image_base64
```

4. Add the request body builder function.

   - This function prepares the request payload for submission to Bedrock:

```python
#get the stringified request body for the InvokeModel API call
def get_titan_image_extension_request_body(prompt, input_image_bytes, negative_prompt=None, vertical_alignment=0.5, horizontal_alignment=0.5):

    # Create an input image which contains the original image with an extended canvas
    original_image = get_image_from_bytes(input_image_bytes)
    
    original_width, original_height = original_image.size
    
    target_width = 1024 #extended canvas size
    target_height = 1024
    
    position = ( #position the existing image within the larger canvas, based on the selected alignment
        int((target_width - original_width) * horizontal_alignment), 
        int((target_height - original_height) * vertical_alignment),
    )
    
    #######
    extended_image = Image.new("RGB", (target_width, target_height), (235, 235, 235))
    extended_image.paste(original_image, position)
    extended_image_base64 = get_png_base64(extended_image)
    
    #######
    mask_image_base64 = get_mask_image_base64(target_width, target_height, position, original_width, original_height)
    
    body = { #create the JSON payload to pass to the InvokeModel API
        "taskType": "OUTPAINTING",
        "outPaintingParams": {
            "image": extended_image_base64,
            "maskImage": mask_image_base64,
            "text": prompt,  # Description of the background to generate
            "outPaintingMode": "DEFAULT",  # "DEFAULT" softens the mask. "PRECISE" keeps it sharp.
        },
        "imageGenerationConfig": {
            "numberOfImages": 1,  # Number of variations to generate
            "quality": "premium",  # Allowed values are "standard" or "premium"
            "width": target_width,
            "height": target_height,
            "cfgScale": 8,
            "seed": randint(0, 100000),  # Use a random seed
        },
    }
    
    if negative_prompt:
        body['outPaintingParams']['negativeText'] = negative_prompt
    
    return json.dumps(body)
```

5. Add the functions to call Bedrock and extract the image from the response.
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
def get_image_from_model(prompt_content, image_bytes, negative_prompt=None, vertical_alignment=0.5, horizontal_alignment=0.5):
    session = boto3.Session()

    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    body = get_titan_image_extension_request_body(prompt_content, image_bytes, negative_prompt=negative_prompt, vertical_alignment=vertical_alignment, horizontal_alignment=horizontal_alignment)
    
    response = bedrock.invoke_model(body=body, modelId="amazon.titan-image-generator-v1", contentType="application/json", accept="application/json")
    
    output = get_titan_response_image(response)
    
    return output
```

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_extension_app.py**

2. Copy and paste the Streamlit app code.
   - This sets up the needed elements to collect user input and pass the request to the corresponding library script.

```python
import streamlit as st
import image_extension_lib as glib

st.set_page_config(layout="wide", page_title="Image Extension")

st.title("Image Extension")

col1, col2, col3 = st.columns(3)


horizontal_alignment_dict = { #horizontal alignment options for original image placement within the extended area
    "Left": 0.0,
    "Center": 0.5,
    "Right": 1.0,
}

vertical_alignment_dict = {  #vertical alignment options for original image placement within the extended area
    "Top": 0.0,
    "Middle": 0.5,
    "Bottom": 1.0,
}

horizontal_alignment_options = list(horizontal_alignment_dict)
vertical_alignment_options = list(vertical_alignment_dict)

with col1:
    st.subheader("Initial image")
    
    uploaded_file = st.file_uploader("Select an image (smaller than 1024x1024)", type=['png', 'jpg'])
    
    if uploaded_file:
        uploaded_image_preview = glib.get_bytesio_from_bytes(uploaded_file.getvalue())
        st.image(uploaded_image_preview)
    else:
        st.image("images/example.jpg")


with col2:
    st.subheader("Extension parameters")
    prompt_text = st.text_area("What should be seen in the extended image:", height=100, help="The prompt text")
    negative_prompt = st.text_input("What should not be in the extended area:", help="The negative prompt")
    
    horizontal_alignment_selection = st.select_slider("Original image horizontal placement:", options=horizontal_alignment_options, value="Center")
    vertical_alignment_selection = st.select_slider("Original image vertical placement:", options=vertical_alignment_options, value="Middle")
    
    generate_button = st.button("Generate", type="primary")
    

with col3:
    st.subheader("Result")

    if generate_button:
        with st.spinner("Drawing..."):
            
            if uploaded_file:
                image_bytes = uploaded_file.getvalue()
            else:
                image_bytes = glib.get_bytes_from_file("images/example.jpg")
             
            generated_image = glib.get_image_from_model(
                prompt_content=prompt_text, 
                image_bytes=image_bytes,
                negative_prompt=negative_prompt,
                vertical_alignment=vertical_alignment_dict[vertical_alignment_selection],
                horizontal_alignment=horizontal_alignment_dict[horizontal_alignment_selection],
            )
        
        st.image(generated_image)
```

3. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image_extension
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image_extension
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

   - We set `server.enableXsrfProtection=false` so that we can upload a file from the AWS Cloud9 preview.

```bash
streamlit run image_extension_app.py --server.port 8080 --server.enableXsrfProtection=false
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-9/app.png)

4. Set a value in the **What should be seen in the extended image** field, and click the **Generate** button to see an extension of the pre-loaded image.

   - `Christmas tree and toy car`

![app](/images/2-Bedrock/image/M-9/app-in-use.png)

5. Optionally, upload an image with dimensions between 256x256 and 1022x1022 and a resolution of 72 pixels per inch. Click the **Generate** button to see an extension of your image.

   - You can download sample images from the **/workshop/sample_images** folder in your Cloud9 environment.

6. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image extension app with Bedrock and Streamlit!
{{% /notice %}}