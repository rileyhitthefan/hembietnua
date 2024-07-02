---
title: "Lab F-8: Phép nhúng"
date: 2024-06-28T15:20:59+07:00
draft: false
chapter : false
weight: 8
---

Dùng model [Amazon Titan Embeddings](https://aws.amazon.com/bedrock/titan/) để tìm điểm tương đồng tương đối của ngữ cảnh văn bản.

**Phép nhúng** đặt nghĩa của các mảnh văn bản trong dãy các số hay **vector**. Từ các vector, chúng ta có thể xác định dự tương đồng giữa các đoạn văn bản. 

Chúng ta dùng những **vector database** để chứa các vector và tìm kiếm độ tương đồng với tốc độ cao. Các từ nhúng được chứa trong vector database là yếu tố then chốt của **RAG** (retrieval-augmented generation), một phương pháp quan trọng chúng ta sẽ tìm hiểu trong các lab sau.

Trong lab này, chúng ta sẽ so sánh sự tương đồng về nghĩa giữa các đoạn văn bản, bao gồm bản dịch.

## Tạo script python

1. Vào thư mục **workshop/labs/embedding**, mở file **bedrock_embedding.py**

![embedding](/images/2-Bedrock/F-8/1.png)

2. Thêm câu lệnh import.

   -  Sử dụng thư viện của LangChain để gọi Bedrock và làm các phép toán cần thiết để so sánh vector. 

```python
from langchain_community.embeddings import BedrockEmbeddings
from numpy import dot
from numpy.linalg import norm
```

3. Tạo LangChain client của Bedrock Embeddings.

```python
#create an Amazon Titan Embeddings client
belc = BedrockEmbeddings()
```

4. Tạo lớp lưu trữ từ nhúng và kết quả so sánh

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

5. Tạo hàm để so sánh sự tương đồng giữa các vector.

   - This implements the [Cosine Similarity](https://en.wikipedia.org/wiki/Cosine_similarity) equation.

```py
def calculate_similarity(a, b): #See Cosine Similarity: https://en.wikipedia.org/wiki/Cosine_similarity
    return dot(a, b) / (norm(a) * norm(b))
```

6. Tạo danh sách nhúng từ file items.txt.

```py
#Build the list of embeddings to compare
items = []

with open("items.txt", "r") as f:
    text_items = f.read().splitlines()

for text in text_items:
    items.append(EmbedItem(text))
```

7. So sánh các vector nhúng và in danh sách để so sánh sự tương đồng và khác biệt giữa các đoạn văn bản. 

   - Các đoạn đồng nghĩa có giá trị tương đồng bằng 1. A similarity value of 1 means exactly the same.
   - Sự tương đồng các ít, các vector nhúng càng khác nhau.

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

8. Lưu file.

## Run the script
1. Chọn **bash terminal** trong AWS Cloud9, di chuyến đến thư mục sau.

```bash
cd ~/environment/workshop/labs/embedding
```

2. Chạy script python bằng terminal.

```bash
python bedrock_embedding.py
```

3. Câu trả lời sẽ được in trong terminal. Chú ý sự tương đồng tương đối giữa các đoạn.

![embedding](/images/2-Bedrock/F-8/2.png)


