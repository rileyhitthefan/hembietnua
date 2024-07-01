---
title: "Lab F-9: Intro to Streamlit"
date: 2024-06-28T15:18:14+07:00
draft: false
chapter : false
weight: 9
---

**Streamlit** is an open-source Python framework for building front-end applications to demo machine learning applications. Streamlit provides a library of commands for displaying web elements. You can see the list of controls in the [Streamlit API reference](https://docs.streamlit.io/library/api-reference).

Streamlit allows you to build simple and attractive user interfaces with a relatively small amount of Python code. For back-end developers, this means you can create demo applications for your code, without having to learn the various programming languages, frameworks, and hosting platforms for front-end development. Even for front-end developers, it allows you to quickly build a proof of concept to validate your approach.

Streamlit is well suited for creating generative AI prototypes. Throughout this workshop, you will have the opportunity to try a wide variety of Streamlit's capabilities that greatly simplify the demo-building process.

You run your streamlit applications from the command line using the streamlit run command. You can find a more in-depth introduction to Streamlit at its [Get started page](https://docs.streamlit.io/library/get-started).

## Create the Streamlit app
1. Navigate to the **workshop/labs/simple_streamlit** folder, and open the file **simple_streamlit_app.py**

![Steamlit](/images/2-Bedrock/F-9/1.png)

2. Add the import statement.
   - This statement allows us to use Streamlit elements and functions.

```py
import streamlit as st #all streamlit commands will be available through the "st" alias
```

3. Add the page title and configuration.

   - Here we are setting the page title on the actual page and the title shown in the browser tab.

```py
st.set_page_config(page_title="Streamlit Demo") #HTML title
st.title("Streamlit Demo") #page title
```

4. Add the input elements.
   - We are creating an input text box and button to get a color from the user.

```python
color_text = st.text_input("What's your favorite color?") #display a text box
go_button = st.button("Go", type="primary") #display a primary button
```

5. Add the output elements.
   - We use the if block below to handle the button click. We then format the submitted color and display it using Streamlit's write function.

```py
if go_button: #code in this if block will be run when the button is clicked

    st.write(f"I like {color_text} too!") #display the response content
```

6. Save the file.

## Run the Streamlit app

1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/simple_streamlit
```
**Just want to run the app?**
{{%expand "Expand here and run this command instead" %}}
```bash
cd ~/environment/workshop/completed/simple_streamlit
```
You can now proceed with step 2.
{{% /expand%}}

2. Run the streamlit command from the terminal.

```bash
streamlit run simple_streamlit_app.py --server.port 8080
```

Ignore the Network URL and External URL links displayed by the Streamlit command. Instead, we will use AWS Cloud9's preview feature.

3. In **AWS Cloud9**, select **Preview -> Preview Running Application**.

![Steamlit](/images/2-Bedrock/F-9/2.png)

You should see a web page like below:

![Steamlit](/images/2-Bedrock/F-9/3.png)

4. Enter a color and see the results.

![Steamlit](/images/2-Bedrock/F-9/4.png)

5. Close the preview tab in AWS Cloud9. Return to the terminal and press Control-C to exit the application.

{{% notice tip %}}
**Congratulations!**\
You have successfully built a simple Streamlit app!
{{% /notice %}}