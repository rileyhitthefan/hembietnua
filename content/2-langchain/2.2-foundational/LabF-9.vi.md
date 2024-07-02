---
title: "Lab F-9: Giới thiệu Streamlit"
date: 2024-06-28T15:18:14+07:00
draft: false
chapter : false
weight: 9
---

**Streamlit** là một khung ứng dụng Python mã nguồn mở dùng trong tạo ra giao diện người dùng để thử nghiệm các ứng dụng ML. Streamlit bao gồm một thư viện có các [câu lệnh](https://docs.streamlit.io/library/api-reference) biểu diễn các thành phần trang web. 

Streamlit rất phù hợp để tạo các nguyên mẫu gen AI. Ứng dụng của bạn có thể chạy từ CLI bằng lệnh streamlit run. Xem thêm về Streamlit tại [đây](https://docs.streamlit.io/library/get-started).

## Tạo app Streamlit
1. Vào thư mục **workshop/labs/simple_streamlit**, mở file **simple_streamlit_app.py**

![Steamlit](/images/2-Bedrock/F-9/1.png)

2. Thêm câu lệnh import.
   - Dùng các phần tử và hàm của Streamlit.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
```

3. Thêm tiêu đề trang và file config.

```py
st.set_page_config(page_title="Streamlit Demo") #HTML title
st.title("Streamlit Demo") #page title
```

4. Thêm phần tử input.
   - Tạo một hộp nhập văn bản và một nút chọn màu sắc cho người dùng.

```python
color_text = st.text_input("What's your favorite color?") #display a text box
go_button = st.button("Go", type="primary") #display a primary button
```

5. Thêm phần tử output.
   - Dùng lệnh if để xử lý hoạt động nút chọn, in màu sắc được chọn bằng hàm write của Streamlit.

```py
if go_button: #code in this if block will be run when the button is clicked

    st.write(f"I like {color_text} too!") #display the response content
```

6. Lưu file.

## Chạy app Streamlit.

1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/simple_streamlit
```
**Chỉ muốn chạy thử ứng dụng?**
{{%expand "Chạy lệnh này:" %}}
```bash
cd ~/environment/workshop/completed/simple_streamlit
```
Đến bước 2.
{{% /expand%}}

2. Chạy lệnh streamlit từ terminal.

```bash
streamlit run simple_streamlit_app.py --server.port 8080
```

Dùng tình năng xem trước của Cloud9, bỏ qua các URL của lệnh trước.

3. Trong **AWS Cloud9**, chọn **Preview -> Preview Running Application**.

![Steamlit](/images/2-Bedrock/F-9/2.png)

Trang web chạy thành công:

![Steamlit](/images/2-Bedrock/F-9/3.png)

4. Nhập một màu sắc và xem kết quả:

![Steamlit](/images/2-Bedrock/F-9/4.png)

5. Tắt trang web trước của Cloud9. Vào terminal, bấm Control-C để thoát ứng dụng.

