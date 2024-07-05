---
title: "Lab P-5: Translation"
date: 2024-07-05T09:55:29+07:00
draft: false
weight: 6
chapter : false
---

The foundation models available through Amazon Bedrock have varying degrees of multilingual capabilities. In this lab, you will be able to see the different outputs from each model. If you are proficient in another language, you can change the prompts to see how well the various models perform.

## Prompt application configuration
Be sure to select the **Translation** radio button under the **Lab context** section of the [prompt application you set up earlier](AppSetup.md).

![app](/images/2-Bedrock/prompteng/P-1/context-translation.png)

{{% notice note %}}
**A note on {context} placeholders**\
Some of the prompt exercises below will include a `{context}` placeholder. This allows us to reuse content and keep the examples below from getting too large. The **See context** expander in the prompt application will show you the text that will be inserted into the `{context}` placeholders when those examples are run.
{{% /notice %}}

## Prompt excercises

### Translating text
You can use large language models to translate text from one language to another:

```txt
Use cases for generative AI:
* Summarizing large documents
* Answering questions based on a body of knowledge
* Analyzing content and providing recommendations
* Creating new content

Please translate the above text into Spanish:
```

{{% notice note %}}
**Challenge**\
If you are proficient in another language, try comparing translation results across the different models.
{{% /notice %}}

### Translated question and answer
You can also ask a question and get an answer in a different language than the source content. Below we ask “What are the key security and privacy features of Amazon Bedrock?” in Spanish:

```txt
{context}

¿Cuáles son las principales funciones de seguridad y privacidad de Amazon Bedrock? Respuesta en español:
```

{{% notice tip %}}
**Congratulations!**\
You have successfully demonstrated prompt engineering techniques for translation!
{{% /notice %}}
