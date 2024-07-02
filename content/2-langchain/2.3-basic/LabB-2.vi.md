---
title: "Lab B-2: Tạo hình ảnh"
date: 2024-07-01T13:20:44+07:00
draft: false
weight: 2
chapter : false
---

Xây dựng ứng dụng tạo hình ảnh cơ bản bằng **Stable Diffusion**, **Amazon Bedrock** và **Streamlit**. Chúng ta sẽ dùng **Boto3** để tương tác với Stable Diffusion (LangChain chủ yếu hỗ trợ model tạo văn bản).

**Stable Diffusion** tạo hình ảnh từ lệnh văn bản. Bắt đầu từ các điểm nhiễu ngẫu nhiên và tiếp tục tạo hình ảnh qua nhiều bước. Ngoài ra, có thể dùng Stable Diffusion để biến đổi các hình ảnh có sẵn.

{{% notice info %}}
Chọn **Stability AI** > **SDXL 1.0** trong phần **Model access** trong console Amazon Bedrock (xem [hướng dẫn](../2.1-prep/bedrockSetup.md)) để truy cập Stable Diffusion
{{% /notice %}}

{{% notice note %}}
**Chỉ muốn chạy ứng dúng?**\
Đến mục [Chạy ứng dụng tạo sẵn.](#run-the-streamlit-app)
{{% /notice %}}

### Trường hợp sử dụng 
Mẫu tạo hình ảnh căn bản có thể dùng cho:  
- Tạo ảnh tuỳ chọn cho website, email, ...
- Tạo minh hoạ ý tưởng cho các loại truyền thông

### Kiến trúc
![Architecture](/images/2-Bedrock/basic/B-2/architecture.png)

Ứng dụng gồm 2 file: 1 file front-end Streamlist, 1 file cho các thự viện hỗ trợ để gọi API Bedrock.

## Tạo script thư viện
Tạo thư viện hỗ trỡ để kết nối front-end Streamlit đến back-end Bedrock.

1. Vào thư mục **workshop/labs/image**, mở file **image_lib.py**

![lib](/images/2-Bedrock/basic/B-2/1.png)

2. Thêm câu lệnh import.
   - Dùng Boto3 để gọi Bedrock, đọc biến môi trường và xử lý thông tin hình ảnh.

```py
import boto3 #import aws sdk and supporting libraries
import json
import base64
from io import BytesIO
```

3. Tạo client Boto3 và id của model Bedrock

```py
session = boto3.Session()

bedrock = session.client(service_name='bedrock-runtime') #creates a Bedrock client

bedrock_model_id = "stability.stable-diffusion-xl-v1" #use the Stable Diffusion model
```

4. Thêm hàm chuyến đổi hình ảnh
   - Hàm tách thông tin hình ảnh từ payload được trả về và chuyển thành định dạng phù hợp với Streamlit.

```py
def get_response_image_from_payload(response): #returns the image bytes from the model response payload

    payload = json.loads(response.get('body').read()) #load the response body into a json object
    images = payload.get('artifacts') #extract the image artifacts
    image_data = base64.b64decode(images[0].get('base64')) #decode image

    return BytesIO(image_data) #return a BytesIO object for client app consumption
```

5. Thêm hàm gọi Bedrock.
   - Tạo hàm gọi Bedrock từ ứng dụng Streamlit để đưa nội dung input vào Bedrock và trả lại hình ảnh.

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

6. Lưu file.

## Tạo giao diện người dùng của app Streamlit

1. Trong cùng thư mục với script thư viện, mở file **image_app.py**

2. Thêm câu lệnh import.
   - Dùng các phần tử Streamlit và gọi các hàm từ script thư viện back-end.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
import image_lib as glib #reference to local lib script
```

3. Thêm tiêu đề trang và file config.

```python
st.set_page_config(layout="wide", page_title="Image Generation") #set the page width wider to accommodate columns

st.title("Image Generation") #page title

col1, col2 = st.columns(2) #create 2 columns
```

4. Thêm phần tử input.
   - Tạo một hộp nhập đoạn văn bản và nút để gửi câu lệnh từ người dùng đến Bedrock.

```py
with col1: #everything in this with block will be placed in column 1
    st.subheader("Image generation prompt") #subhead for this column
    
    prompt_text = st.text_area("Prompt text", height=200, label_visibility="collapsed") #display a multiline text box with no label
    
    process_button = st.button("Run", type="primary") #display a primary button
```

5. Thêm phần tử output.
   - Trong cột số 2, dùng lệnh if để xử lý hoạt động nút chọn. Hiển thị con xoay trong lúc chờ, và viết lại output lên trang web sau khi trả kết quả.

```python
with col2: #everything in this with block will be placed in column 2
    st.subheader("Result") #subhead for this column
    
    if process_button: #code in this if block will be run when the button is clicked
        with st.spinner("Drawing..."): #show a spinner while the code in this with block runs
            generated_image = glib.get_image_response(prompt_content=prompt_text) #call the model through the supporting library
        
        st.image(generated_image) #display the generated image
```

6. Lưu file.

## Chạy ứng dụng Streamlit
1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/image
```

**Chỉ muốn chạy thử ứng dụng?**
{{%expand "Chạy lệnh này:" %}}
```bash
cd ~/environment/workshop/completed/image
```
Đến bước 2.
{{% /expand%}}

2. Chạy lệnh streamlit từ terminal.

```bash
streamlit run image_app.py --server.port 8080
```

Dùng tình năng xem trước của Cloud9, bỏ qua các URL của lệnh trước.

3. Trong AWS Cloud9, chọn **Preview** -> **Preview Running Application**.

![Preview App](/images/2-Bedrock/F-9/2.png)

Trang web chạy thành công:

![App](/images/2-Bedrock/basic/B-2/3.png)

4. Chạy thử các lệnh khác nhau và xem kết quả.

   - `A painting of a person holding a cat, in the style of Rembrandt`
   - `A painting of a person holding a cat, in the style of Picasso`

![App Results](/images/2-Bedrock/basic/B-2/4.png)

5. Tắt trang web trước của Cloud9. Vào terminal, bấm Control-C để thoát ứng dụng.