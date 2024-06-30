---
title: "Chatbot app"
date: 2024-06-27T14:39:49+07:00
draft: true
weight : 4
chapter: false
pre : " <b> 1.4 </b> "
---

#### Introduction

In this lab, we will build a chatbot app from scratch using PartyRock. We will connect **User Input, Chatbot, Text Generation, and Image Generation widgets** together for maximum geeky effect.

We will use **prompt chaining** to wire the various widgets together. **Prompt chaining** is the practice of using the output of an AI-powered widget as input to another AI-powered widget. In this case, we will generate an image based on whatever the geek chatbot is talking about.

You can try this app here: https://partyrock.aws/u/js2222/G9KTPc8Ue/Geek-Chatbot 

---

#### Build from scratch
1. Go to https://partyrock.aws and sign in if needed.
2. Select the **Build your own app** button, then select **Create empty app**.

![pr](/images/1-PartyRock/012-PartyRock.png)

3. Review your empty app, including its placeholder title

![pr](/images/1-PartyRock/013-PartyRock.png)

4. Set the page title to `Geek Chatbot`

5. Select the **+ Add widget** button, then select **User Input**

![pr](/images/1-PartyRock/024-PartyRock.png)

6. Edit the User Input widget’s properties
   - Set **Widget title** to `Geeky Topic`
   - Set **Placeholder** to `Provide a geeky topic that your chatbot is obsessed with!`
   - Select the **Save** button

![pr](/images/1-PartyRock/025-PartyRock.png)

7. Select the **+ Add widget** button, then select **Chatbot**

8. Edit the Chatbot widget’s properties
   - Set **Widget title** to `Talk to your Geek Chatbot!`
   - Set **Placeholder** to `Geek out here!`
   - Set **Model** to `Claude 3 Sonnet`
   - Set **Initial message** to `Let’s geek out!`
   - Set **Prompt** to `Pretend you are obsessed with TOPIC. You only want to talk about TOPIC. The user should only want to talk about TOPIC. - The user will now have a conversation with you.`
   - In the Prompt box, replace all occurrences of the word TOPIC using the @ symbol to select the **Geeky Topic** field
   - Select the **Save** button

![pr](/images/1-PartyRock/026-PartyRock.png)

9. Adjust the width of the **Geeky Topic** and **Talk to your Geek Chatbot** panels by dragging the resize handle in their lower right corners.

10. Hover over the area to the right of the Geeky Topic widget to see the Create Widget target area. You can click in that area to place a widget there.
    - Alternatively, you could use the + Add widget button and move a widget into that area later.

11. Add a **Text Generation** widget.

![pr](/images/1-PartyRock/027-PartyRock.png)

12. Edit the Text Generation widget’s properties
    - Set **Widget title** to `Summary`
    - Set **Model** to `Claude 3 Haiku`
    - Set **Prompt** to `Write a one-line summary of the most recent message in the conversation:`
    - Then enter the @ symbol to select the `Talk to your Geek chatbot!` box as a source of content.
    - Select the **Save** button

![pr](/images/1-PartyRock/028-PartyRock.png)

13. Add an **Image Generation** widget.

14. Edit the Image Generation widget’s properties
    - Set **Widget title** to `Image`
    - Set **Model** to `Stable Diffusion XL`
    - Set **Image description** to `An image of `
    - Then enter the **@** symbol to select the `Summary` box as a source of content.
    - Select the **Save** button
    
![pr](/images/1-PartyRock/029-PartyRock.png)

15. Enter your geeky topic!
    - Some possible topics: `Aerospace engineering`, `Plumbing`, `Atlantic`

![pr](/images/1-PartyRock/030-PartyRock.png)

16. (Optional) Disappoint your chatbot

![pr](/images/1-PartyRock/031-PartyRock.png)