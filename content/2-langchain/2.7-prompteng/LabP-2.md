---
title: "Lab P-2: Content creation"
date: 2024-07-05T09:55:20+07:00
draft: false
weight: 3
chapter : false
---

In this lab, we will learn how to write prompts to generate content. Use cases include:

- Brainstorming
- Outlining
- First draft
- Rewriting


## Prompt application configuration
Be sure to select the **Content creation** radio button under the **Lab context** section of the [prompt application you set up earlier](AppSetup.md).

![app](/images/2-Bedrock/prompteng/P-1/context-content.png)

{{% notice note %}}
**A note on {context} placeholders**\
Some of the prompt exercises below will include a `{context}` placeholder. This allows us to reuse content and keep the examples below from getting too large. The **See context** expander in the prompt application will show you the text that will be inserted into the `{context}` placeholders when those examples are run.
{{% /notice %}}


## Prompt excercises


### Brainstorming
Blocked on where to start? Have the model generate a list of ideas for you!

```txt
Please create a list of startup ideas for a new generative AI service for the food industry:
```


### Brainstorming with more details
You can also influence the format of responses by letting the model know what you want to see:

```txt
Please create a list of startup ideas for a new generative AI service for the food industry. For each startup idea, provide a name and a one-sentence elevator pitch for what the startup will do.
```

{{% notice note %}}
**Challenge**\
Can you alter the prompt to be more specific, or add more details?\
**Hint**:\
Could you have it focus on restaurants, food manufacturers, or consumers?\
Can you have it indicate the target market for each startup idea?
{{% /notice %}}


### Outlining
You can also have the model create an outline for an article, based on some existing content:

```txt
{context}

Write an outline for a blog article about the service described above:
```


### Write a first draft
You can also have the model create a first draft of content in a new form, based on existing material:

```txt
{context}

Write a marketing plan for the service described above:
```


### Draft with questions to be answered
You can help influence the output by asking the model to answer specific questions in its response:

```txt
{context}

Write a marketing plan for the service described above. Be sure to answer the following in the marketing plan:

Who is the person using the product?
What is the unmet need that will be satisfied by the product?
Who is the buyer of the product?
What is the go-to-market strategy for the product?
```


### Slogan
You can also use the model to generate short content like a marketing slogan, based on existing material:

```txt
Amazon Bedrock is a fully managed service that offers a choice of high-performing foundation models (FMs) from leading AI companies like AI21 Labs, Anthropic, Cohere, Meta, Stability AI, and Amazon with a single API, along with a broad set of capabilities you need to build generative AI applications, simplifying development while maintaining privacy and security. With the comprehensive capabilities of Amazon Bedrock, you can easily experiment with a variety of top FMs, privately customize them with your data using techniques such as fine-tuning and retrieval augmented generation (RAG), and create managed agents that execute complex business tasks—from booking travel and processing insurance claims to creating ad campaigns and managing inventory—all without writing any code. Since Amazon Bedrock is serverless, you don't have to manage any infrastructure, and you can securely integrate and deploy generative AI capabilities into your applications using the AWS services you are already familiar with.

Create a slogan for the above product:
```


### Rewriting content
Large language models can be used to rewrite existing content for a different purpose:

```txt
{context}

Rewrite the above content for a marketing brochure:
```


### Rewriting content with constraints
You can set constraints on the output by telling the model who the content is for, and what not to do:

```txt
{context}

Write a marketing brochure based on the above content. The marketing brochure should be targeted to an executive business audience. Avoid the use of technical jargon and acronyms.
```

{{% notice note %}}
**Challenge**\
Can you change the prompt to focus more on the security aspects of the service?
{{% /notice %}}


### Rewriting from bullet points
The models can be used to take your raw thoughts in bullet point forms and generate content from them:

```txt
Use cases for generative AI:
* Summarizing large documents
* Answering questions based on a body of knowledge
* Analyzing content and providing recommendations
* Creating new content

Please write an introductory paragraph to a blog article highlighting the above use cases:
```


### FAQ from content
You can also have the model anticipate frequently asked questions based on content you provide:

```txt
{context}

Write an FAQ based on the above content:
```

{{% notice tip %}}
**Congratulations!**\
You have successfully demonstrated prompt engineering techniques for content creation!
{{% /notice %}}