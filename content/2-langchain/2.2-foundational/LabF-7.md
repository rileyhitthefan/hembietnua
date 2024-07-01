---
title: "Lab F-7: Streaming API"
date: 2024-06-28T15:18:11+07:00
draft: false
chapter : false
weight: 7
---

In this lab, we will show how to make a **streaming API** call directly to Bedrock.

Streaming responses are useful when you want to start returning content immediately to the end user. You can display the output a few words at a time, instead of waiting for the entire response to be created.

We'll use the **Boto3** library directly for this lab.

## Create the Python script
1. Navigate to the **workshop/labs/intro_streaming** folder, and open the file **intro_streaming.py**

![streaming](/images/2-Bedrock/F-7/1.png)

2. Add the import statements.

   - These statements allow us to use the AWS Boto3 library to call Amazon Bedrock.
```python
import json
import boto3
```

1. Initialize the Bedrock client library.

   - This creates a Bedrock client.

```python
session = boto3.Session()

bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client
```

4. Define the callback handler for streaming results.

   - This function allows us to print response chunks as they are returned from the streaming api.

```python
def chunk_handler(chunk):
    print(chunk, end='')
```

5. Define the function to call the Bedrock streaming API.

   - We use Bedrock's `invoke_model_with_response_stream` function to make the call to the streaming API endpoint.
   - As response chunks are returned, this code extracts the chunk's text from the returned JSON and passes it to the provided callback method.

```python
def get_streaming_response(prompt, streaming_callback):

    bedrock_model_id = "anthropic.claude-3-sonnet-20240229-v1:0" #set the foundation model
    
    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 8000,
        "temperature": 0,
        "messages": [
            {
                "role": "user",
                "content": [{ "type": "text", "text": prompt } ]
            }
        ],
    })
    
    response = bedrock.invoke_model_with_response_stream(modelId=bedrock_model_id, body=body) #invoke the streaming method
    
    for event in response.get('body'):
        chunk = json.loads(event['chunk']['bytes'])

        if chunk['type'] == 'content_block_delta':
            if chunk['delta']['type'] == 'text_delta':
                streaming_callback(chunk['delta']['text'])
```

6. Display the response.
   - This defines the prompt and passes it to the `get_streaming_response` function along with the callback handler.

```python
prompt = "Tell me a story about two puppies and two kittens who became best friends:"
                
get_streaming_response(prompt, chunk_handler)
```

7. Save the file.

## Run the script

1. Select the **bash terminal** in AWS Cloud9 and change directory.
```bash
cd ~/environment/workshop/labs/intro_streaming
```

2. Run the script from the terminal.
```bash
python intro_streaming.py
```

3. The results should be displayed in the terminal.

![streaming](/images/2-Bedrock/F-7/2.png)

{{% notice tip %}}
**Congratulations!**\
You have successfully called the Bedrock streaming API!
{{% /notice %}}