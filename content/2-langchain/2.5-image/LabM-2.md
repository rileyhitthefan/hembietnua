---
title: "Lab M-2: Image prompting"
date: 2024-07-03T09:02:10+07:00
draft: false
weight: 2
chapter : false
---

In this lab, we will build a basic image generator using Amazon Titan Image Generator, Amazon Bedrock, and Streamlit. We'll be using the Boto3 library to interact with Titan Image Generator, since LangChain primarily supports text generation models.

Titan Image Generator creates images from a text prompt. It starts with random noise and gradually forms an image over a series of steps. It can also be used to transform existing images based on a prompt, which we will try in a later lab.

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The image generation pattern is good for the following use cases:
- Creation of custom imagery for websites, emails, etc.
- Creation of concept art for various media forms

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/image_prompts** folder, and open the file **image_prompts_lib.py**

![lib](/images/2-Bedrock/image/M-2/open.png)

2. Add the import statements.

   - These statements allow us to use the Boto3 library to call Bedrock, read environment variables, and process image data.

```python
import boto3
import json
import base64
from io import BytesIO
from random import randint
```

3. Add the request body builder function.

   - This function prepares the request payload for submission to Bedrock:

```python
#get the stringified request body for the InvokeModel API call
def get_titan_image_generation_request_body(prompt, negative_prompt=None):
    
    body = { #create the JSON payload to pass to the InvokeModel API
        "taskType": "TEXT_IMAGE",
        "textToImageParams": {
            "text": prompt,
        },
        "imageGenerationConfig": {
            "numberOfImages": 1,  # Number of images to generate
            "quality": "premium",
            "height": 512,
            "width": 512,
            "cfgScale": 8.0,
            "seed": randint(0, 100000),  # Use a random seed
        },
    }
    
    if negative_prompt:
        body['textToImageParams']['negativeText'] = negative_prompt
    
    return json.dumps(body)
```

4. Add the image conversion function.

   - This function extracts the image data from the returned payload and converts it to a format that Streamlit can use.

```python
#get a BytesIO object from the Titan Image Generator response
def get_titan_response_image(response):

    response = json.loads(response.get('body').read())
    
    images = response.get('images')
    
    image_data = base64.b64decode(images[0])

    return BytesIO(image_data)
```

5. Add this function to call Bedrock.

   - We're creating a function we can call from the Streamlit front end application. This function passes the input content to Bedrock and returns the image.

```python
#generate an image using Amazon Titan Image Generator
def get_image_from_model(prompt_content, negative_prompt=None):
    session = boto3.Session()

    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    body = get_titan_image_generation_request_body(prompt_content, negative_prompt=negative_prompt)
    
    response = bedrock.invoke_model(body=body, modelId="amazon.titan-image-generator-v1", contentType="application/json", accept="application/json")
    
    output = get_titan_response_image(response)
    
    return output
```

6. Save the file.

## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **image_prompts_app.py**

2. Add the import statements.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st
import image_prompts_lib as glib
```

3. Add the page title, configuration, and column layout.

   - Here we are setting the page title on the actual page and the title shown in the browser tab. We're creating two columns to collect the input on the left and display the output on the right.

```python
st.set_page_config(layout="wide", page_title="Image Generation")

st.title("Image Generation")

col1, col2 = st.columns(2)
```

4. Add the input elements.

   - In the first column, we are creating a multiline text box and button to get the user's prompt and send it to Bedrock.

```python
with col1:
    st.subheader("Image parameters")
    
    prompt_text = st.text_area("What you want to see in the image:", height=100, help="The prompt text")
    negative_prompt = st.text_input("What shoud not be in the image:", help="The negative prompt")

    generate_button = st.button("Generate", type="primary")
```

5. Add the output elements.

   - In the second column, we use the `if` block to handle the button click. We display a spinner while the backing function is called, then write the output to the web page.

```python
with col2:
    st.subheader("Result")

    if generate_button:
        with st.spinner("Drawing..."):
            generated_image = glib.get_image_from_model(
                prompt_content=prompt_text, 
                negative_prompt=negative_prompt,
            )
        
        st.image(generated_image)
```

6. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/image_prompts
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/image_prompts
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run image_prompts_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-2/app.png)

4. Under the Image search tab, try some search terms:

   - `daguerreotype of robot and cowboy standing side-by-side, directly facing the camera, steampunk, western town in the background, long shot, sepia tone`
   - `photograph of a calico cat, cyberpunk, futuristic cityscape in the background, low angle, long shot, neon sign on building "CALICO CORP", Epic, photorealistic, 4K`

![app](/images/2-Bedrock/image/M-2/app-in-use.png)

5. See the table below for some example prompts built using different elements.

| Element added   | Prompt                                                                                                                                   | Image                                                 | Note                                                                                                          |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Subject         | *A doctor*                                                                                                                          | ![subject](/images/2-Bedrock/image/M-2/subject.png)   | Usually will default to a photograph-like image                                                               |
| Medium          | *Painting of a doctor*                                                                                                              | ![medium](/images/2-Bedrock/image/M-2/medium.png)     | The type of art to generate (other examples: daguerreotype, pencil sketch, watercolor, 3d cartoon, many more) |
| Style           | *Painting of a doctor, Impressionist style*                                                                                          | ![style](/images/2-Bedrock/image/M-2/style.png)       | The style or theme of the art to generate (other examples: Dadaist style, Renaissance style, many more)       |
| Shot type/angle | *Painting of a doctor, Impressionist style, low-angle shot*                                                                          | ![angle](/images/2-Bedrock/image/M-2/angle.png)       | The angle or distance for the image (other examples: wide shot, close-up, many more)                          |
| Light           | *Painting of a doctor, Impressionist style, low-angle shot, dim lighting*                                                            | ![lighting](/images/2-Bedrock/image/M-2/lighting.png) | The lighting scheme for the image (other examples: golden hour, studio lighting, many more)                   |
| Color scheme    | *Painting of a doctor, Impressionist style, low-angle shot, dim lighting, blue and purple color scheme*                                  | ![color](/images/2-Bedrock/image/M-2/color.png)       | The color scheme for the image (other examples: pastel colors, neon colors, grayscale, many more)             |
| Negative prompt | (Use the **What shoud not be in the image** field) *Stethoscope*                                                                         | ![negative](/images/2-Bedrock/image/M-2/negative.png) | What should not be included in the image                                                                      |
| Text            | *Painting of a doctor, Impressionist style, low-angle shot, dim lighting, blue and purple color scheme, sign reading "The Doctor is in"* | ![text](/images/2-Bedrock/image/M-2/text.png)         | For Titan Image Generator, text should be wrapped in double quotes. The generated image may have typos.       |



5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built an image generation app with Bedrock and Streamlit!
{{% /notice %}}