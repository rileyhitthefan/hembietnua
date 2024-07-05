---
title: "Lab P-3: Summarization"
date: 2024-07-05T09:55:22+07:00
draft: false
weight: 4
chapter : false
---

In this lab, we will learn prompt engineering techniques for summarization. Summarization can be useful when users don't have time to read large documents, have too much content to consume, or want to better understand what's in a document before investing the time to read it.

Use cases include:

- Abbreviated content
- Quick descriptions of content
- Structured summaries
-Personalized summaries based on user interests

## Prompt application configuration
Be sure to select the **Summarization** radio button under the **Lab context** section of the [prompt application you set up earlier](AppSetup.md).

![app](/images/2-Bedrock/prompteng/P-1/context-summarization.png)

{{% notice note %}}
**A note on {context} placeholders**\
Some of the prompt exercises below will include a `{context}` placeholder. This allows us to reuse content and keep the examples below from getting too large. The **See context** expander in the prompt application will show you the text that will be inserted into the `{context}` placeholders when those examples are run.
{{% /notice %}}

## Prompt excercises


### Basic summary

You can ask the model to generate a summary for you:

```txt
{context}

Summarize the above article:
```

{{% notice note %}}
The above prompt is generic and can lead to a variety of formats, content lengths, and random selection of content to include. We’ll explorer various ways to guide the model to more specific results below.
{{% /notice %}}

### One-sentence summary
What if we want a one sentence summary so someone can quickly decide if they want to take the time to read more?

```txt
{context}

Write a one-sentence summary of the above content:
```

### Short summary
Or maybe you want a paragraph or a few sentences version, to accomodate a little more detail, but without requiring a significant time investment to consume:

```txt
{context}

In a couple of sentences, briefly summarize the above article:
```

{{% notice note %}}
**Challenge**\
Can you get the model to return a summary that focuses on a specific division of Amazon?\
**Hint**:\
In a couple of sentences, briefly summarize any information about AWS in the article:\
In a couple of sentences, briefly summarize information about Amazon's e-commerce businesses:
{{% /notice %}}

### Structured summary
Maybe you want to have a more structured format for the summary. This helps group related items together and provides the model with topics to focus on.

```txt
{context}

Summarize the above article under the following headings: Pain Points, Positive Results, Growth Opportunities
```

{{% notice note %}}
**Challenge**\
Can you come up with different sections for the structured summary?\
**Hint**:\
You could try something like: Cloud Computing, Retail Businesses, Other Items of Note.
{{% /notice %}}

### Summarize for reading level
Large language models can also be used to rewrite content for younger audiences.

```txt
{context}

Summarize the above article so that a third-grader can understand it:
```

### Personalized summarization
You might want to generate summaries for distinct roles or individuals, so that they only need to focus on the areas that most concern them. Below, we have a prompt geared towards a financial analyst audience.

```txt
{context}

Concisely summarize the above article from a financial analyst's perspective. Focus on areas that could impact the overall profitability or growth of the company.
```

{{% notice note %}}
**Challenge**\
Can you rewrite the prompt to get a response for a different role?\
**Hint**:\
Concisely summarize the above article for a Chief Technology Officer. Include any interesting technology developments or innovations that might appeal to information technology leaders:
{{% /notice %}}

{{% notice tip %}}
**Congratulations!**\
You have successfully demonstrated prompt engineering techniques for summarization!
{{% /notice %}}