---
title: "Lab F-8: Embeddings"
date: 2024-06-28T15:20:59+07:00
draft: false
chapter : false
weight: 8
---

In this lab, we will learn the basics of embeddings. We will use [Amazon Titan Embeddings](https://aws.amazon.com/bedrock/titan/) to help find the relative similarity of text content.

**Embeddings** capture the meaning of a piece of text in a series of numbers called a **vector**. We can then use these vectors to determine how similar pieces of text are to each other.

We can use a **vector database** to store these embeddings and perform fast similarity searches. Embeddings paired with a vector database are a core component of **retrieval-augmented generation**, a key pattern that we will learn more about in later labs.

In the lab below, we will compare the similarity of meaning between various pieces of text, including some translations.

## Create the python script

1. Navigate to the **workshop/labs/embedding** folder, and open the file **bedrock_embedding.py**

![embedding](/images/2-Bedrock/F-8/1.png)

2. Add the import statements.

   - These statements allow us to use LangChain library to call Amazon Bedrock, and make the necessary calculations to compare vectors.

```python
from langchain_community.embeddings import BedrockEmbeddings
from numpy import dot
from numpy.linalg import norm
```

3. Initialize the Bedrock Embeddings LangChain client.

```python
#create an Amazon Titan Embeddings client
belc = BedrockEmbeddings()
```

4. Define the classes to store embeddings and the comparison results.

```py
class EmbedItem:
    def __init__(self, text):
        self.text = text
        self.embedding = belc.embed_query(text)

class ComparisonResult:
    def __init__(self, text, similarity):
        self.text = text
        self.similarity = similarity
```

5. Define the function to compare the similarity of two vectors.

   - This implements the [Cosine Similarity](https://en.wikipedia.org/wiki/Cosine_similarity) equation.

```py
def calculate_similarity(a, b): #See Cosine Similarity: https://en.wikipedia.org/wiki/Cosine_similarity
    return dot(a, b) / (norm(a) * norm(b))
```

6. Build a list of embeddings from the items.txt file.

```py
#Build the list of embeddings to compare
items = []

with open("items.txt", "r") as f:
    text_items = f.read().splitlines()

for text in text_items:
    items.append(EmbedItem(text))
```

7. Compare embeddings and display lists to show how similar or different the various texts are.

   - A similarity value of 1 means exactly the same.
   - The smaller the similarity, the less similar are the embeddings.

```py
for e1 in items:
    print(f"Closest matches for '{e1.text}'")
    print ("----------------")
    cosine_comparisons = []
    
    for e2 in items:
        similarity_score = calculate_similarity(e1.embedding, e2.embedding)
        
        cosine_comparisons.append(ComparisonResult(e2.text, similarity_score)) #save the comparisons to a list
        
    cosine_comparisons.sort(key=lambda x: x.similarity, reverse=True) # list the closest matches first
    
    for c in cosine_comparisons:
        print("%.6f" % c.similarity, "\t", c.text)
    
    print()
```

8. Save the file.

## Run the script
1. Select the **bash terminal** in AWS Cloud9 and change directory.

```bash
cd ~/environment/workshop/labs/embedding
```

2. Run the script from the terminal.

```bash
python bedrock_embedding.py
```

3. The results should be displayed in the terminal. Note the relative similarities.

![embedding](/images/2-Bedrock/F-8/2.png)

{{% notice tip %}}
**Congratulations!**\
You have successfully demonstrated Amazon Titan Embeddings and vector similarities!
{{% /notice %}}

