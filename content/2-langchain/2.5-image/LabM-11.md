---
title: "Lab M-11: Multimodal chatbot"
date: 2024-07-03T09:02:38+07:00
draft: false
weight: 11
chapter : false
---

In this lab, we will build a multimodal chatbot with Amazon Bedrock, Anthropic Claude 3, and Streamlit.

A **multimodal chatbot** is a chatbot that can understand text, images, and other formats. We will keep this example simple and stick to just text and images. We can upload or add some pre-created images, and then discuss them with the Claude 3 large language model.

LLMs don't have any concept of state or memory. Any chat history has to be tracked externally and then passed into the model with each new message. We are using a list of custom objects to track chat history. Since there is a limit on the amount of content that can be processed by the model, we need to prune the chat history so there is enough space left to handle the user's message and the model's responses. Our code will delete older messages.

In this example, we're building the chatbot directly using Boto3, and not using LangChain. You can try the [Basic Chatbot](../2.3-basic/LabB-4.md) or [RAG Chatbot](../2.4-text/LabI-1.md) labs for examples of chatbots built using LangChain. We'll be using the [Anthropic Claude Messages API](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-anthropic-claude-messages.html).

{{% notice note %}}
**Just want to run the app?**\
You can [jump ahead to run a pre-made application.](#run-the-streamlit-app)
{{% /notice %}}

### Use cases
The multimodal chatbot pattern is good for the following use cases:

- Simple interactive user conversation with image analysis
- Image comparisons

This application consists of two files: one for the Streamlit front end, and one for the supporting library to make calls to Bedrock.

## Create the library script
First we will create the supporting library to connect the Streamlit front end to the Bedrock back end.

1. Navigate to the **workshop/labs/multimodal_chatbot** folder, and open the file **multimodal_chatbot_lib.py**

![open](/images/2-Bedrock/image/M-11/open.png)

2. Add the import statements and the file and image processing helper functions.

   - These statements allow us to use the Boto3 library to call Bedrock, and process image data.

   - `MAX_MESSAGES` sets the upper limit for previous chat messages kept in memory.

   - The ChatMessage class is used to store image and text messages.

   - The helper functions are used to convert data between files, images, and base64-encoded bytes.

```python
import boto3
import json
import base64
from io import BytesIO

MAX_MESSAGES = 20

class ChatMessage(): #create a class that can store image and text messages
    def __init__(self, role, message_type, text, bytesio=None):
        self.role = role
        self.message_type = message_type
        self.text = text
        self.bytesio = bytesio

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

3. Add a function to convert ChatMessages to the Claude 3 Messages API format.

   - This format allows us to send a list of current and past messages to Claude 3 for processing.

```python
def convert_chat_messages_to_messages_api(chat_messages):
    
    messages = []
    
    for chat_msg in chat_messages:
        if (chat_msg.message_type == 'image'):
            messages.append({
                "role": "user",
                "content": [
                    {
                        "type": "image",
                        "source": {
                            "type": "base64",
                            "media_type": "image/jpeg",
                            "data": chat_msg.text,
                        },
                    }
                ]
            })
        else:
            messages.append({
                "role": chat_msg.role,
                "content": [
                    {
                        "type": "text",
                        "text": chat_msg.text
                    }
                ]
            })
            
    return messages
```

4. Add the request body builder function.

   - This function prepares the request payload for submission to Bedrock

```python
#get the stringified request body for the InvokeModel API call
def get_multimodal_chat_request_body(chat_messages):
    
    messages = convert_chat_messages_to_messages_api(chat_messages)
    
    body = {
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 8000,
        "temperature": 0,
        "messages": messages,
    }
    
    return json.dumps(body)
```

5. Add a function to call Bedrock and return the response.

   - We're creating a function we can call from the Streamlit front-end application. This function passes the input content to Bedrock and returns the text.

   - Well add either a new text message or image message to the message history, depending on what the user submitted.

   - If we have more than 20 messages, we'll delete older messages. This keeps costs down and avoids breaching token limits by preventing the conversation from getting too long.

   - We'll add Claude 3's response to the message history, so it can be displayed in the user interface.

```python
#generate a response using Anthropic Claude
def chat_with_model(message_history, new_text=None, new_image_bytes=None):
    session = boto3.Session()
    
    bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
    
    if new_text:
        new_text_message = ChatMessage('user', 'text', text=new_text)
        message_history.append(new_text_message)
        
    elif new_image_bytes:
        image_bytesio = get_bytesio_from_bytes(new_image_bytes)
        image_base64 = get_base64_from_bytes(new_image_bytes)
        new_image_message = ChatMessage('user', 'image', text=image_base64, bytesio=image_bytesio)
        message_history.append(new_image_message)
    
    
    number_of_messages = len(message_history)
    
    if number_of_messages > MAX_MESSAGES:
        del message_history[0 : (number_of_messages - MAX_MESSAGES) * 2] #make sure we remove both the user and assistant responses
    
    
    body = get_multimodal_chat_request_body(message_history)
    
    response = bedrock.invoke_model(body=body, modelId="anthropic.claude-3-sonnet-20240229-v1:0", contentType="application/json", accept="application/json")
    
    response_body = json.loads(response.get('body').read()) # read the response
    
    output = response_body['content'][0]['text']
    
    response_message = ChatMessage('assistant', 'text', output)
    
    message_history.append(response_message)
    
    return
```

6. Save the file.


## Create the Streamlit front-end app
1. In the same folder as your lib file, open the file **multimodal_chatbot_app.py**

2. Add the import statements.

   - These statements allow us to use Streamlit elements and call functions in the backing library script.

```python
import streamlit as st #all streamlit commands will be available through the "st" alias
import multimodal_chatbot_lib as glib #reference to local lib script
```

3. Add the page title and configuration.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```python
st.set_page_config(page_title="Multimodal Chatbot") #HTML title
st.title("Multimodal Chatbot") #page title
```

4. Add the chat history to the session cache.

   - This allows us to maintain a unique chat memory per user session. Otherwise, the chatbot won't be able to remember past messages with the user.
   - In Streamlit, session state is tracked server-side. If the browser tab is closed, or the application is stopped, the session and its chat history will be lost. In a real-world application, you would want to track the chat history with a service like [Amazon ElastiCache](https://aws.amazon.com/blogs/database/solutions-for-building-modern-applications-with-amazon-elasticache-and-amazon-memorydb-for-redis/) or [Amazon DynamoDB](https://aws.amazon.com/dynamodb/).

```python
if 'chat_history' not in st.session_state: #see if the chat history hasn't been created yet
    st.session_state.chat_history = [] #initialize the chat history
```

5. Add the chat input controls

   - These controls allow us to send text, pre-defined images, or uploaded images to the Claude 3 for processing.

```python
chat_container = st.container()

input_text = st.chat_input("Chat with your bot here") #display a chat input box

uploaded_file = st.file_uploader("Select an image", type=['png', 'jpg'], label_visibility="collapsed")


col1, col2, col3 = st.columns(3)
with col1:
    upload_image_1 = st.button("Add miniature house image")

with col2:
    upload_image_2 = st.button("Add house and car image")

with col3:
    upload_image_3 = st.button("Add miniature car image")
```

6. Add the event handler logic.

   - We use the if blocks below to handle the user input.

```python
if upload_image_1:
    image_bytes = glib.get_bytes_from_file("images/minihouse.jpg")
    glib.chat_with_model(message_history=st.session_state.chat_history, new_text=None, new_image_bytes=image_bytes)
    
elif upload_image_2:
    image_bytes = glib.get_bytes_from_file("images/house_and_car.jpg")
    glib.chat_with_model(message_history=st.session_state.chat_history, new_text=None, new_image_bytes=image_bytes)

elif upload_image_3:
    image_bytes = glib.get_bytes_from_file("images/minicar.jpg")
    glib.chat_with_model(message_history=st.session_state.chat_history, new_text=None, new_image_bytes=image_bytes)

elif input_text: #run the code in this if block after the user submits a chat message
    glib.chat_with_model(message_history=st.session_state.chat_history, new_text=input_text, new_image_bytes=None)

elif uploaded_file:
    image_bytes = uploaded_file.getvalue()
    
    glib.chat_with_model(message_history=st.session_state.chat_history, new_text=None, new_image_bytes=image_bytes)
```

7. Add the for loop to render previous chat messages.
   - Re-render previous messages based on the chat_history session state object.

```python
#Re-render the chat history (Streamlit re-runs this script, so need this to preserve previous chat messages)
for message in st.session_state.chat_history: #loop through the chat history
    with chat_container.chat_message(message.role): #renders a chat line for the given role, containing everything in the with block
        if (message.message_type == 'image'):
            st.image(message.bytesio)
        else:
            st.markdown(message.text) #display the chat content
```

8. Save the file.

## Run the Streamlit app
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/multimodal_chatbot
```

**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/multimodal_chatbot
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

   - We set `server.enableXsrfProtection=false` so that we can upload a file from the AWS Cloud9 preview.

```bash
streamlit run multimodal_chatbot_app.py --server.port 8080 --server.enableXsrfProtection=false
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In AWS Cloud9, select **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![app](/images/2-Bedrock/image/M-11/app.png)

4. Try out some images and prompts and see the results. You can use the **Add image** buttons to bring some predefined images into the conversation, or upload your own images (between 200-1000 pixels each for width and height, max 5MB per image, max 20 images). [Review limits here](https://docs.anthropic.com/claude/docs/vision#faq) 

   - `Compare the two images:`
   - `Which one is larger?`
   - `Which one looks more fun?`
   - `What color is the car?`

![app](/images/2-Bedrock/image/M-11/app-in-use.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a multimodal chatbot with Bedrock and Streamlit!
{{% /notice %}}