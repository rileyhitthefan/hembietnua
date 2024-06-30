---
title: "Content summarizer app"
date: 2024-06-27T14:34:27+07:00
draft: true
weight : 3
chapter: false
pre : " <b> 1.3 </b> "
---
You can try this app here: https://partyrock.aws/u/js2222/aw3_JHzyG/Content-Summarizer 

#### Introduction

In this lab, we will build a basic content summarizer app from scratch. In addition to the prompt-based app builder feature, PartyRock allows you to create an app through a web-based app editor.

In the steps below, we will use User Input and Text Generation widgets to automatically generate detailed & one-line summaries from longer content.

---

#### Build from scratch
1. Go to https://partyrock.aws and sign in if needed.
2. Select the **Build your own app** button, then select **Create empty app**.

![pr](/images/1-PartyRock/012-PartyRock.png)

3. Review your empty app, including its placeholder title

![pr](/images/1-PartyRock/013-PartyRock.png)

4. Replace the placeholder title with `Content Summarizer`

![pr](/images/1-PartyRock/014-PartyRock.png)

#### Add User Input and Static Text widgets
5. Select the **+ Add widget** button, then select **Static Text**

![pr](/images/1-PartyRock/015-PartyRock.png)

6. Edit the Static Text widget’s properties.
   - Set Widget title to `Welcome`
   - Set Content to:
   
        ```command
        # Welcome to the Content Summarizer application!

        Instructions:

        1. Paste your content
        2. Get your summaries!

        ## WARNING: Never share sensitive information when building or using a PartyRock app. This includes personal information and internal company information.
        ```
   - Adjust the height of the widget by dragging the resize handle in its lower right corner (make sure all content is visible on screen by default)
   - Select the **Save** button

![pr](/images/1-PartyRock/016-PartyRock.png)

{{% notice note %}}
The number signs in the text above are used to represent headings. The Static Text widget uses **Markdown** to indicate text styles. Please see the included Markdown basics guide to learn more.
{{% /notice %}}

7. Select the **+ Add widget** button, then select **User Input**

![pr](/images/1-PartyRock/017-PartyRock.png)

8. Set the User Input widget’s properties.

   - Set **Widget title** to `Paste your content here`
   - Set **Placeholder** to `Content to summarize (NO SENSITIVE CONTENT)`
   - Select the **Save** button

![pr](/images/1-PartyRock/018-PartyRock.png)

#### Add AI-powered widgets
9. Select the **+ Add widget** button, then select **Text Generation**

10. Set the Text Generation widget’s properties.
    - Set **Widget title** to `Detailed summary`
    - Set **Model** to `Command`
    - In the **Prompt** field, enter ` Write a detailed summary based on this content:`, then enter the @ symbol to select the `Paste your content here` box as a source of content.
    - Select the **Save** button

![pr](/images/1-PartyRock/019-PartyRock.png)

11. Adjust the width of the **Detailed summary** panel by dragging the resize handle in its lower right corner.

12. Add a new **Text Generation** widget.

![pr](/images/1-PartyRock/020-PartyRock.png)

13. Set the Text Generation widget’s properties.

    - Set **Widget title** to `One-line summary`
    - Set **Model** to `Claude 3 Sonnet`
    - In the **Prompt** field, enter `Write a one-line summary based on this content:`, then enter the @ symbol to select the `Paste your content here` box as a source of content.
    - Expand the **Advanced settings** section and set the **Temperature** field to *0.5*. This will let the model generate a random response each time it is run.
    - Select the **Save** button

![pr](/images/1-PartyRock/021-PartyRock.png)

![pr](/images/1-PartyRock/022-PartyRock.png)


#### Test the app
14. Copy the following text and paste it into the **Paste your content here** box.

    ```command
    PartyRock, an Amazon Bedrock playground, is a shareable generative AI app building playground. Experiment with prompt engineering in a hands-on and fun way. In a short time, you can build, share, and remix apps, to get inspired while playing with generative AI. For example, you can:

        Build an app to generate dad jokes on the topic of your choice.
        Create the perfect playlist based on your musical tastes.
        Recommend what to serve based on ingredients in your pantry.
        Create and play a virtual trivia game online with friends from around the world.
        Create an AI storyteller to guide your next fantasy -playing campaign.

    By building and playing with PartyRock apps, you learn about the fundamental techniques and capabilities needed to get started with generative AI, including understanding how a foundation model responds to a given prompt, experimenting with different text-based prompts, and chaining prompts together. Anyone can access PartyRock through its intuitive web-based UI. For anyone interested in learning the fundamental skills needed to harness the power of generative AI, PartyRock uses Amazon Bedrock to create an accessible environment for experimentation with powerful foundational models (FMs). The tool builds confidence by providing new PartyRock users with free trial use for a limited time, and the PartyRock Guide helps learners extend their knowledge. Builders who are ready to move into production can proceed to the Bedrock console using the basic prompts and skills developed in PartyRock.

    Any builder can experiment with PartyRock by creating a profile using a social login from amazon.com, Apple, or Google. PartyRock is separate from the AWS console and does not require an AWS account to get started.
    PartyRock is for EVERYONE. It has been designed in a way that is friendly and accessible to builders of all skill levels, particularly those who have limited experience with generative AI. Even if you have no coding experience, you can use text-based prompts to experiment with foundation models and create your own generative AI-powered apps. Developers can explore model capabilities and refine prompt engineering techniques in PartyRock without managing API calls.

    ```

![pr](/images/1-PartyRock/023-PartyRock.png)

{{% notice warning %}}
Never include sensitive information in PartyRock!
{{% /notice %}}