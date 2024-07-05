---
title: "Lab P-7: Code"
date: 2024-07-05T09:55:34+07:00
draft: false
weight: 8
chapter : false
---

In this lab, we will learn how to write prompts to help with software development. We can use Amazon Bedrock's large language models to write and analyze code. Use cases include:

- Writing a first draft of new functions and classes
- Generating unit tests for existing code
- Re-writing code
- Understanding and documenting existing code

## Prompt application configuration
Be sure to select the **Code** radio button under the **Lab context** section of the [prompt application you set up earlier](AppSetup.md).

![app](/images/2-Bedrock/prompteng/P-1/context-code.png)

{{% notice note %}}
**A note on {context} placeholders**\
Some of the prompt exercises below will include a `{context}` placeholder. This allows us to reuse content and keep the examples below from getting too large. The **See context** expander in the prompt application will show you the text that will be inserted into the `{context}` placeholders when those examples are run.
{{% /notice %}}

## Prompt exercises

### Write a function

We can have the models write a function:

```txt
Write a Python function that calculates the sum of all numbers in a list:
```

{{% notice note %}}
**Challenge**\
Can you update the prompt to ensure that a list with mixed types will be handled properly?
**Hint**
Write a Python function that calculates the sum of all numbers in a list. Assume the list might contain both numeric and non-numeric data types.
{{% /notice %}}

### Rewrite a function
If we see an issue with a function, we can also have the large language model rewrite the function in a way that addresses the issue:

```txt
def sum_list(numbers):
    total = 0
    for num in numbers:
        total += num
    return total

Rewrite the above function to check the type of each item before adding it to the total:
```

### Explain a function
We can also have the models help us understand and document code by writing an explanation:

```txt
def func_1(numbers):
    total = 0
    for num in numbers:
        if isinstance(num, (int, float)):
            total += num
    return total
    
What does the above function do?
```

### Write a unit test
You can ask the model to write a unit test for a function:

```txt
def func_1(numbers):
    total = 0
    for num in numbers:
        if isinstance(num, (int, float)):
            total += num
    return total
    
Write a unit test for the above function:
```

### Write a class
Large language models can also generate classes:

```txt
Write a "Person" class in Python that stores the person's name, date of birth. Include a method to display the person in the format "Name (Date of Birth)":
```

{{% notice tip %}}
**Congratulations!**\
You have successfully demonstrated prompt engineering techniques for writing and analyzing code!
{{% /notice %}}