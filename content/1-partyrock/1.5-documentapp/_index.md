---
title: "Document chat app"
date: 2024-06-27T14:34:42+07:00
draft: true
weight : 1
chapter: false
pre : " <b> 1.5 </b> "
---

#### Introduction

In this lab, we will build a document chatbot app from scratch using PartyRock. We will connect **Document** and **Chatbot** widgets together to allow you to have a conversation about an uploaded document.

{{% notice warning %}}
Never post sensitive information (either personal information or sensitive business information) to PartyRock!
{{% /notice %}}

You can provide your own document in one of the following formats:
- PDF
- Text (.txt)
- Word (.docx)
- Markdown (.md)
- HTML
- CSV

There is a limit of 120,000 characters in the uploaded file.

{{% notice note %}}
Use a smaller uploaded file to minimize your PartyRock credit consumption.
{{% /notice %}}

{{%attachments title="If you want a simple, small PDF, you can download the PDF version of Amazon's leadership principles here:" pattern=".*(pdf)"/%}}

---

#### Build from scratch
1. Go to https://partyrock.aws and sign in if needed.
2. Select the **Build your own app** button, then select **Create empty app**.

![pr](/images/1-PartyRock/012-PartyRock.png)

3. Review your empty app, including its placeholder title

![pr](/images/1-PartyRock/013-PartyRock.png)

4. Set the page title to `Document Chat`

5. Select the **+ Add widget** button, then select **Document**

6. Edit the Document widget’s properties
   - Set **Widget title** to `Upload a Document`
   - Select the **Save** button

![pr](/images/1-PartyRock/032-PartyRock.png)

7. Select the **+ Add widget** button, then select **Chatbot**

8. Edit the Chatbot widget’s properties
   - Set **Widget title** to `Chat About the Document`
   - Set **Placeholder** to `Chat here!`
   - Set **Model** to `Claude 3 Sonnet`
   - Set **Initial message** to `Ready to chat!`
   - Set **Prompt** to: `Support the conversation using the information within the document tags. <document>[Upload a Document]</document> The user will now have a conversation with you.`
   - Select the **Save** button

![pr](/images/1-PartyRock/033-PartyRock.png)

9. Upload a document and chat!

{{%attachments title="You can download the PDF version of Amazon's leadership principles here: " pattern=".*(pdf)"/%}}

{{% notice warning %}}
Never post sensitive information (either personal information or sensitive business information) to PartyRock!
{{% /notice %}}

![pr](/images/1-PartyRock/034-PartyRock.png)