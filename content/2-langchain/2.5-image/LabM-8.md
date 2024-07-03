---
title: "Lab M-8: Image inpainting"
date: 2024-07-03T09:02:29+07:00
draft: false
weight: 8
chapter : false
---

In this lab, we will build an image inpainting application using Amazon Titan Image Generator, Amazon Bedrock, and Streamlit. We'll be using the Boto3 library to interact with Titan Image Generator.

The image inpainting pattern uses Titan Image Generator’s image mask inpainting capability. It requires two image files:

1. The original image.
2. An image mask consisting of black and white pixels. The black pixels indicate the mask portion of the image. Since we’re inpainting, the black pixels will be repainted.

In this lab, we'll programmatically generate an image mask based on a set of predetermined values, or allow the user to specify rectangular coordinates for a custom-generated image mask. For an example of using a pre-created image mask, please see the [Masking introduction lab](./LabM-5.md).

See the illustration below to understand the steps:

| 1. Start with the original image                      | 2. Generate or upload an image mask file for the area to be inpainted | 3. Specify what should be inpainted in the masked area | 4. Inpaint the masked area of the original image        |
| ----------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------- |
| ![original](/images/2-Bedrock/image/M-8/original.png) | ![mask](/images/2-Bedrock/image/M-8/mask.png)                         | "Two framed portraits of cats"                         | ![inpainted](/images/2-Bedrock/image/M-8/inpainted.png) |
| *Uploaded file*                                       | *Image mask*                                                          | *Prompt text*                                          | *Inpainted image created with Titan Image Generator*    |

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image inpainting pattern is good for the following use cases:

- Brainstorm decoration ideas by swapping out specific areas of a room photo.
- Change a portion of the image that is too bare, or has distractions that should be removed.
- Allowing a user to select areas of an image that should be repainted.

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. In AWS Cloud9, navigate to the **workshop/labs/image_insertion** folder, and open the file **image_insertion_lib.py**

![open](/images/2-Bedrock/image/M-8/open.png)

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
```

3. Add the image mask creation function.

   - This function creates a custom image mask based on the desired dimensions for inpainting and outpainting.
   - In this example, we are creating rectangular mask areas, but in your use case the mask can be made from any combination of black or white pixels based on the original image.

4. Add the request body builder function.

   - This function prepares the request payload for submission to Bedrock:

```python
#get the stringified request body for the InvokeModel API call
def get_titan_image_insertion_request_body(prompt_content, input_image_bytes, insertion_position, insertion_dimensions):

    original_image = get_image_from_bytes(input_image_bytes)
    
    input_image_base64 = get_png_base64(original_image)
    
    target_width, target_height = original_image.size
    
    inside_width, inside_height = insertion_dimensions
    
    mask_image_base64 = get_mask_image_base64(target_width, target_height, insertion_position, inside_width, inside_height)
    
    body = { #create the JSON payload to pass to the InvokeModel API
        "taskType": "INPAINTING",
        "inPaintingParams": {
            "image": input_image_base64,
            "maskImage": mask_image_base64,
            "text": prompt_content,  # What to add to the image
        },
        "imageGenerationConfig": {
            "numberOfImages": 1,  # Number of variations to generate
            "quality": "premium",  # Allowed values are "standard" and "premium"
            "height": target_height,
            "width": target_width,
            "cfgScale": 8.0,
            "seed": randint(0, 100000),  # Change the seed to generate different content
        },
    }
    
    return json.dumps(body)
```

5. Add the functions to call Bedrock and extract the image from the response.

   - The get_titan_response_image function extracts the image data from the returned payload and converts it to a format that Streamlit can use.
   - We're creating a function we can call from the Streamlit front end application. This function passes the input content to Bedrock and returns the image.

```python
#get a BytesIO object from the Titan Image Generator response
def get_titan_response_image(response):

    response = json.loads(response.get('body').read())
    
    images = response.get('images')
    
    image_data = base64.b64decode(images[0])

    return BytesIO(image_data)


#load the bytes from a file on disk
def get_bytes_from_file(file_path):
    with open(file_path, "rb") as image_file:
        file_bytes = image_file.read()
    return file_bytes


#generate an image using Amazon Titan Image Generator
def get_image_from_model(prompt_content, image_bytes, mask_prompt=None, negative_prompt=None, insertion_position=None, insertion_dimensions=None):
    session = boto3.Session()

    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    if image_bytes == None:
        image_bytes = get_bytes_from_file("images/desk.jpg") #use desk.jpg if no file uploaded
    
   
    body = get_titan_image_insertion_request_body(prompt_content, image_bytes, insertion_position, insertion_dimensions) #preload desk.jpg
    
    response = bedrock.invoke_model(body=body, modelId="amazon.titan-image-generator-v1", contentType="application/json", accept="application/json")
    
    output = get_titan_response_image(response)
    
    return output
```

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_insertion_app.py**

2. Copy and paste the Streamlit app code.

   - This sets up the needed elements to collect user input and pass the request to the corresponding library script.

```python
import streamlit as st
import image_insertion_lib as glib


st.set_page_config(layout="wide", page_title="Image Insertion")

st.title("Image Insertion")

col1, col2, col3 = st.columns(3)

placement_options_dict = { #Configure mask areas for image insertion
    "Wall behind desk": (3, 3, 506, 137), #x, y, width, height
    "On top of desk": (78, 60, 359, 115),
    "Beneath desk": (108, 237, 295, 239),
    "Custom": (0, 0, 200, 100), 
}

placement_options = list(placement_options_dict)


with col1:
    st.subheader("Initial image")
    
    uploaded_file = st.file_uploader("Select an image (must be 512x512)", type=['png', 'jpg'])
    
    if uploaded_file:
        uploaded_image_preview = glib.get_bytesio_from_bytes(uploaded_file.getvalue())
        st.image(uploaded_image_preview)
    else:
        st.image("images/desk.jpg")
    

with col2:
    st.subheader("Insertion parameters")
    
    placement_area = st.radio("Placement area:", 
        placement_options,
    )
    
    with st.expander("Custom:", expanded=False):
        
        mask_dimensions = placement_options_dict[placement_area]
    
        mask_x = st.number_input("Mask x position", value=mask_dimensions[0])
        mask_y = st.number_input("Mask y position", value=mask_dimensions[1])
        mask_width = st.number_input("Mask width", value=mask_dimensions[2])
        mask_height = st.number_input("Mask height", value=mask_dimensions[3])
    
    prompt_text = st.text_area("Object to add:", height=100, help="The prompt text")
    
    generate_button = st.button("Generate", type="primary")
    

with col3:
    st.subheader("Result")

    if generate_button:
        with st.spinner("Drawing..."):
            if uploaded_file:
                image_bytes = uploaded_file.getvalue()
            else:
                image_bytes = None
            
            generated_image = glib.get_image_from_model(
                prompt_content=prompt_text, 
                image_bytes=image_bytes, 
                insertion_position=(mask_x, mask_y),
                insertion_dimensions=(mask_width, mask_height),
            )
        
        st.image(generated_image)
```

3. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image_insertion
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image_insertion
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

   - We set `server.enableXsrfProtection=false` so that we can upload a file from the AWS Cloud9 preview.

```bash
streamlit run image_insertion_app.py --server.port 8080 --server.enableXsrfProtection=false
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-8/app.png)

4. Set the **Placement area** and **Object to add** field values for what you want to inpaint and where. Click the **Generate** button to see a variation of the pre-loaded image.

**Placement area**: Wall behind desk
   - `A painting of a Spanish galleon in rough seas`
   - `Framed family photographs`

**Placement area**: On top of desk
   - `writing instruments`
   - `A hutch`
   - `A pile of papers`

**Placement area**: Beneath desk
   - `a stool`
   - `A sleeping cat`

![app](/images/2-Bedrock/image/M-8/app-in-use.png)

5. Optionally, upload an image with [supported dimensions](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-titan-image.html) and a resolution of 72 pixels per inch. Specify a custom masking area that would work with your image. Click the **Generate** button to see the inpainted version of your image.

6. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image inpainting app with Bedrock and Streamlit!
{{% /notice %}}