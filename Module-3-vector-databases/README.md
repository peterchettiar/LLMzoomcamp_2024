# Week 3 notes

### Table of contents

- [3.1 Introduction to Vector Databases](#31-introduction-to-vector-databases)
  - [Vector Search](#vector-search)
  - [Vector Embeddings and Indexing](#vector-embeddings-and-indexing)
  - [Approximate Nearest Neighbour (ANN)](#approximate-nearest-neighbour-ann)
  - [Vector Search Data and Workflow](#vector-search-data-workflow)
- [3.2 Semantic Search Engine with ElasticSearch](#32-semantic-search-engine-with-elasticsearch)
  - [Introduction](#introduction)
  - [Understanding Documents and Indexes in Elasticsearch](#understanding-documents-and-indexes-in-elasticsearch)
  - [Steps to run Semantic Search using ElasticSearch](#steps-to-run-semantic-search-using-elasticsearch)
- [3.3 Evaluating Retrieval](#33-evaluating-retrieval)
  - [Introduction](#introduction-1)
  - [Generating The GroundTruth Datasets](#generating-the-ground-truth-datasets)
  - [Ranking Evaluation: Text Search](#ranking-evaluation-text-search)
  - [Ranking Evaluation: Vector Search](#ranking-evaluation-vector-search)

## 3.1 Introduction to Vector Databases

In the evolving landscape of data management, vector databases have emerged as a critical solution for handling vast and diverse datasets (examples of unstructured data that make up to more than 80% of the data being generated today - social media posts, images, videos, audio). Unlike traditional databases, which are limited to structured data, vector databases excel in managing unstructured data and providing relevant results based on context.

> Note: A vector database indexes and stores vector embeddings for fast retrieval and similarity search.

Let's take an image of a cat as an example of handling unstructured data. Based on pixel values alone we cannot search for similar images. And since we cannot store unstructured data in relational databases, the only way to find similar cat images in said database is to assign tags or attributes to the image, often manually, to perform such searches. Again this is not ideal.

![image](https://github.com/user-attachments/assets/a060dbf3-4e8f-47d3-8cf0-5cc4595e6aa6)

Therefore, there was a need to come up with a more viable solution to represent unstructured data, the solution being vector search and vector embeddings!

> Please note that the terms vector search and vector database are related concepts in the field of data management and information retrieval, but they have distinct meanings.

### Vector Search

Vector search is a method of finding similar items in a dataset by comparing their vector representations (a.k.a `vector embeddings`, which will be discussed in the next section). Unlike traditional keyword-based search, which relies on exact matches, vector search uses mathematical representations of data to find items that are similar in meaning or context. This is a high-level summary and we will look a little deeper into this topic but at this stage, I think it would be prudent to make a comparison between `vector search` and `vector database`. Essentially they refer to the same thing, a process for converting unstructured data into `vector embeddings` and storing them as well as indexing the numeric representations for fast retrieval, but I guess the context in which the terms are used could be different. Hence, please find the following differences:

| Aspect           | Vector Search                                                     | Vector Database                                                               |
| ---------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Definition       | A technique to find similar items based on vector representations | A specialized database system for storing, managing, and querying vector data |
| Primary Function | Searching for similar vectors                                     | Storing and managing vector data, including search capabilities               |
| Storage          | Does not inherently involve storage                               | Provides persistent storage for vector data                                   |
| Implementation   | Can be implemented on various data structures                     | Purpose-built system for vector data                                          |
| Scope            | A method or operation                                             | A complete data management system                                             |
| Optimization     | Focuses on search algorithms                                      | Optimized for vector operations, indexing, and scaling                        |
| Features         | Primarily search functionality                                    | Includes data management, indexing, and querying capabilities                 |
| Use Cases        | Can be part of larger systems                                     | Standalone system for vector-based applications                               |
| Examples         | Cosine similarity, Euclidean distance, ANN algorithms             | Pinecone, Milvus, Faiss, Weaviate                                             |
| Scalability      | Depends on implementation                                         | Often designed for large-scale operations                                     |
| Performance      | Varies based on implementation                                    | Generally optimized for high-performance vector operations                    |

The idea behind the `vector search` concept is to basically convert our unstructured data like text documents or images into a numerical representation (your vector embedding) and subsequently be stored in a multi-dimensional vector space. This way it's easy for the machine to learn and understand, as well as yield more relevant results when performing semantic searches.

Using the same cat example as before, if you provide a cat image, this would be converted to a vector embedding, and `vector search` would return the vector embedding closest to our query vector embedding based on the Euclidean distance (i.e. straight line distance between two vectors in a multidimensional space) or cosine similarity (i.e. cosine of the angle between two vectors - range from -1 to 1 with 1 being an identical vector) in our vector database. And because we have an `index` structure that often includes a distance metric, the execution time is much shorter for the search process as opposed to having to calculate the distance for each vector embedding in our vector database.

So you may be wondering what the purpose of all this, is simply to enable the following use cases:

1. Long-term memory for LLMs
2. Semantic search; search based on the meaning or context
3. Similarity search for text, images, audio, or video data
4. Recommendation engine

### Vector Embeddings and Indexing

At this point we should already have a working knowledge of `vector embeddings` but the official definition by [elastic](https://www.elastic.co/what-is/vector-embedding) is:

_They are a way to convert words, sentences and other data into numbers that capture their meanings and relationships. They represent different data types as points in a multidimensional space, where similar data points are clustered closer together. These numerical representations help machines understand and process this data more effectively._

So the way to convert unstructured data to a `vector embedding` is through the use of ML Models, depending on the type of data you are working with. Following are a few examples of the type of embeddings:

| Type of Embedding   | Description                                                                                                                                                  | Examples/Techniques                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| Word embeddings     | Represent individual words as vectors, capturing semantic relationships and contextual information from large text corpora.                                  | Word2Vec, GloVe, FastText                                                        |
| Sentence embeddings | Represent entire sentences as vectors, capturing the overall meaning and context of the sentences.                                                           | Universal Sentence Encoder (USE), SkipThought                                    |
| Document embeddings | Represent documents (anything from newspaper articles to academic papers) as vectors, capturing the semantic information and context of the entire document. | Doc2Vec, Paragraph Vectors                                                       |
| Image embeddings    | Represent images as vectors by capturing different visual features.                                                                                          | Convolutional neural networks (CNNs), ResNet, VGG                                |
| User embeddings     | Represent users in a system or platform as vectors, capturing user preferences, behaviors, and characteristics.                                              | Used in recommendation systems, personalized marketing, user segmentation        |
| Product embeddings  | Represent products in e-commerce or recommendation systems as vectors, capturing a product's attributes, features, and other semantic information.           | Used to compare, recommend, and analyze products based on vector representations |

The following is a simple guideline for creating a `vector embedding`:

| Step                  | Description                                                                                                                                                                                                                |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Data Collection    | Gather a large dataset representing the type of data for which you want to create embeddings (e.g., text or images).                                                                                                       |
| 2. Preprocessing      | Clean and prepare the data by removing noise, normalizing text, resizing images, or other tasks depending on the data type.                                                                                                |
| 3. Model Selection    | Choose a neural network model that fits your data and goals.                                                                                                                                                               |
| 4. Training           | Feed the preprocessed data into the model. The model learns patterns and relationships by adjusting its internal parameters (e.g., associating words that often appear together or recognizing visual features in images). |
| 5. Vector Generation  | As the model learns, it generates numerical vectors (embeddings) representing the meaning or characteristics of each data point.                                                                                           |
| 6. Quality Assessment | Evaluate the quality and effectiveness of the embeddings by measuring their performance on specific tasks or using human evaluation.                                                                                       |
| 7. Implementation     | Once satisfied with the embeddings' performance, use them for analyzing and processing your data sets.                                                                                                                     |

Now moving on to the concept of `indexing`. As mentioned previously, this is another important process to enable fast retrieval for `vector search`. There are a few types of indexing methods such as:

1. Flat index (brute force) - compares the query with every single vector stored in the database
2. Approximate Nearest Neighbour (ANN) Methods - as the name suggests, using algorithms to find the close vectors that are similar or approximate to the query vector
3. Tree-Based indexing - use a tree-like structure to partition the vector database thereby eliminating large portions of data during the search
4. Graph-based indexing - constructs a graph-like structure where each node represents a vector, and edges connect nodes based on proximity (similarity)

### Approximate Nearest Neighbour (ANN)

Since we are using `elasticsearch` as our choice of search engine, we can take a deeper look into their method for indexing - ANN algorithms.

Approximate Nearest Neighbour (ANN) is an algorithm that finds a data point in a data set that is very close to the query point, but not necessarily the absolute closest one. This is an upgrade from traditional NN algorithms that search through all the data to find the perfect match, which can be time-consuming as well as computationally expensive given that data sources get larger each year. Hence, ANNs are game changers as they use intelligent shortcuts and data structures to efficiently navigate the search space. So instead of taking up huge amounts of time and resources, it can identify data points with much less effort that are close enough to be useful in most practical scenarios.

Now that we know what ANNs are as well as their purpose of building vector indexes, we can proceed to understand how they work. Generally how these algorithms work is firstly a **dimensionality reduction** technique being deployed followed by a **defined metric** to calculate the similarity between the query vector and all other vectors in the table.

There are types of ANNs, to name a few:

1. KD-trees
2. Local-sensitivity hashing (LSH)
3. Annoy
4. Linear scan algorithm
5. Inverted file (IVF) indexes
6. Hierarchical Navigational Small Worlds (HNSW)

Let's take a closer look into LSH to get a deeper understanding of how ANNs work. LSH builds the index in the vector database by using a hashing function. Vector embeddings that are near each other are hashed to the same bucket. We can then store all these similar vectors in a single table or bucket. When a query vector is provided, its nearest neighbours can be found by hashing the query vector, and then computing the similarity metric for all the vectors in the table for all other vectors that hashed to the same value. This indexing strategy optimizes for speed and finding.

### Vector Search Data Workflow

To summarise what we have discussed, the below diagram visually describes the end-to-end workflow of `vector search`

![image](https://github.com/user-attachments/assets/5ec81fcd-8361-4db0-a4f7-6103ffca15fc)

So starting from the left-hand side of the image, we have the unstructured data sources where data is being pulled and converted into `vector embeddings` using ML models. Again, the data type determines the ML model being deployed for this transformation. For example, to convert word-to-word embeddings we use Word2Vec.

After the transformation, these `vector embeddings` undergo an indexing process using Approximate Nearest Neighbours (ANNs) such as Local-Sensitivity Hashing (LSH) where `vector embedding` is grouped with other `vector embeddings` with high similarity scores.

On the other side, the query goes through a similar process where the query is converted into an embedding as well as undergoing an indexing process. Naturally, the query index will enable the search of similar vector embedding indices based on the similarity score with the query index and finally provide the results.

## 3.2 Semantic Search Engine with ElasticSearch

### Introduction

In this chapter, we will explore how to build a semantic search engine using Elasticsearch and Python. Semantic search using Elasticsearch is a specific implementation of vector search that leverages Elasticsearch's capabilities to perform semantic search. It enhances traditional search by understanding the context and meaning behind the search terms, going beyond keyword matching to deliver more relevant results.
![image](https://github.com/user-attachments/assets/59f079f5-aa30-460d-8fe0-1a939fa7ded5)

**Why use Elasticsearch for Semantic Search?**

- _Scalability_: Elasticsearch can handle large volumes of data and high query loads.
- _Flexibility_: It supports various types of data, including text, numbers, and geospatial data.
- _Advanced Features_: Elasticsearch offers advanced search features like full-text search, filtering, and aggregations.

### Understanding Documents and Indexes in Elasticsearch

Elasticsearch is a distributed search engine that stores data in the form of documents. Two very important concepts in Elasticsearch are documents and indexes.

**Documents**

A document is a collection of fields with their associated values. Each document is a JSON object that contains data in a structured format. For example, a document representing a book might contain fields like title, author, and publication date.

**Indexes**

An index is a collection of documents that is stored in a highly optimized format designed to perform efficient searches. Indexes are similar to tables in a relational database, but they are more flexible and can store complex data structures.

To work with Elasticsearch, you need to organize your data into documents and then add all your documents into an index. This allows Elasticsearch to efficiently search and retrieve relevant documents based on the search queries.

### Steps to run Semantic Search using ElasticSearch

**Step 1: Setting up the environment**

The process involves setting up a Docker container, preparing data, generating embeddings with a pre-trained model, and indexing these embeddings into Elasticsearch.

First, check if Docker is running. If not, use a command from a previous module to start a Docker container for Elasticsearch:

```bash
docker run -it \
    --rm \
    --name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```

I used the [docker-compose](https://github.com/peterchettiar/LLMzoomcamp_2024/blob/main/Module-2-open-source-llm/docker-compose.yaml) file from Week 2 but made a slight tweak by adding a `volume` flag. Instead of using the default directory in codespaces, I remounted onto a `/tmps` folder on the host machine so that there is more disk space available. The additional item is as follows:

```yaml
volumes:
  - /tmp/elasticsearch_data:/usr/share/elasticsearch/data
```

So to break it down, `/tmp/elasticsearch_data` is the data directory in the host machine and `/usr/share/elasticsearch/data` is the default directory inside the Elasticsearch container. Since, we made these changes we need to re-build the image using `docker-compose up --build -d` to make sure that the new changes are applied.

There may be also a possibility that the `elasticsearch` container exits unexpectedly. It's good practice to check the logs by running `docker logs elasticsearch` to see what the errors are. Chances are, it may have exited unexpectedly due to the changes we made when mounting volumes. The reason for this is that we do not have the permissions to access these folders by default. Hence, we need to change this by running the command `sudo chown -R 1000:1000 /tmp/elasticsearch_data`. Basically what we are doing here is that we are changing the ownership to all the files and subdirecotries recusrsicely to a new user and group which in our case is both 1000, this is referring to the user in the `elasticsearch` container.

> Note: Please make sure to have the container running before proceeding.

**Step 2: Data Loading and Preprocessing**

In this step, we will load the `documents.json` file and preprocess it to flatten the hierarchy, making it suitable for Elasticsearch. The `documents.json` file contains a list of courses, each with a list of documents. We will extract each document and add a `course` field, indicating which course it belongs to.

**Step 3: Embeddings - Sentence Transformers**

To perform a semantic search, we need to convert our documents into dense vectors (embeddings) that capture the semantic meaning of the text. We will use a pre-trained model from the Sentence Transformers library to generate these embeddings. These embeddings are then indexed into Elasticsearch. These embeddings enable us to perform semantic search, where the goal is to find text that is contextually similar to a given query.

The `text` and `question` fields are the actual data fields containing the primary information, whereas other fields like `section` and `course` are more categorical and less informative for the purpose of creating meaningful embeddings.

- Install the` sentence_transformers` library.
- Load the pre-trained model and use it to generate embeddings for our documents.

```python
# Load a pretrained sentence transformer model

model = SentenceTransformer("all-mpnet-base-v2") # best pretrained model in their library

documents = [doc.update({'text_vector': model.encode(doc['text']).tolist()}) or doc for doc in documents]
```

Pretty much what we did here was to convert the `text` field into an embedding and creating a new key called `text_vector` for each `doc`

**Step 4: Connecting to ElasticSearch**

In this step, we will set up a connection to an Elasticsearch instance. Make sure you have Elasticsearch running.

```python
# establishing the connection with ElasticSearch

es_client = Elasticsearch("http://localhost:9200")

es_client.info()
```

**Step 5: Create Mappings and Index**

We will define the mappings and create the index in Elasticsearch, where the generated embeddings will also be stored.

Mapping is specifying how documents and their fields are structured and indexed in Elasticsearch. Each document is composed of various fields, each assigned a specific data type.

Similar to a database schema, mapping outlines the structure of documents, detailing the fields, their data types (e.g., string, integer, or date), and how these fields should be indexed and stored.

By defining documents and indices, we ensure that an index acts like a table of contents in a book, facilitating efficient searches.

```python
index_settings = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"},
            "text_vector": {"type": "dense_vector", "dims": 768, "index": True, "similarity": "cosine"},
        }
    }
}

index_name = "course-questions"

# Delete the index if it exists
es_client.indices.delete(index=index_name, ignore_unavailable=True)

# Create the index
es_client.indices.create(index=index_name, body=index_settings)
```

**Step 6: Adding the documents to the Index**

We then add the preprocessed documents along with their embeddings to the Elasticsearch index. This allows Elasticsearch to store and manage the documents efficiently, enabling fast and accurate search queries.

I used the `bulk` method instead of the conventional `index` method, just for exploratory purposes:

```python
# lastly to populate the index with our documents list using the bulk method instead of the conventional create method

index = {"index": {
    "_index":index_name
}}

operations =  [item for doc in documents for item in (index, doc)]

resp = es_client.bulk(operations = operations, timeout="120s")
```

**Step 7: Performing Semantic Search with Filter**

Based on our workflow diagram at the start of the section, the other side of the coin is the user query. This too needs to undergo a transformation process to be converted into a vector embedding, followed by defining the parameters of the query before running the `search` method.

1. Let's transform our query into an embedding.

```python
# Here we will use the search term that was used in the course - again we need to convert our search term into an embedding

search_term = 'Windows or Mac?'

vector_search_term = model.encode(search_term)
```

2. Define our query parameters.

```python
# we need to define the parameters of our query, that includes our search term vector as well

knn_query = {
    "field" : "text_vector",  # the field in which the search term should be queried
    "query_vector" : vector_search_term,  # the embedding of our search term
    "k" : 5,  # the number of nearest documents to be retrieved that matches the search term
    "num_candidates" : 10000 # group of documents the search is going to look into
}
```

3. Running a search query using a filter.

```python
# running our semantic search with a filter in place - `match` is used as a filter field

response = es_client.search(
    index=index_name,
    query={
        "match" : {"section": "General course-related questions"},
    },
    knn=knn_query,
    size=5
)
```

For the full notebook of the example we looked through, please click [here](https://github.com/peterchettiar/LLMzoomcamp_2024/blob/main/Module-3-vector-databases/semantic_search_example.ipynb).

## 3.3 Evaluating Retrieval

### Introduction

Retrieval-Augmented Generation (RAG) frameworks rely on retrieval systems to fetch relevant documents or passages from a knowledge base, playing a critical role in the overall performance of the model. Evaluating the quality of these retrieved results is crucial, as they directly influence the effectiveness of the generated responses. While many metrics exist to assess retrieval performance, it's important to recognize that no single metric fits all scenarios. The choice of evaluation metric should align with the specific goals and approach of the retrieval system in question. Below are some widely used metrics, but this list is by no means exhaustive:

| **Metric**                                       | **Description**                                                                                                | **Formula**                                                                                                                                 |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Precision at k (P@k)**                         | Measures the number of relevant documents in the top k results.                                                | `P@k = (Relevant documents in top k results) ÷ k`                                                                                           |
| **Recall**                                       | Measures the number of relevant documents retrieved out of the total number of relevant documents.             | `Recall = (Relevant documents retrieved) ÷ (Total relevant documents)`                                                                      |
| **Mean Average Precision (MAP)**                 | Computes the average precision for each query and then averages these values over all queries.                 | `MAP = (1 ÷ ABS(Q)) × Σ(Average Precision(q)) for q in Q`                                                                                   |
| **Normalized Discounted Cumulative Gain (NDCG)** | Measures the usefulness of a document based on its position in the result list.                                | `NDCG = DCG ÷ IDCG`<br>DCG = Σ((2^rel_i − 1) ÷ log₂(i + 1)) for i = 1 to p <br>IDCG is the ideal DCG, where documents are perfectly ranked. |
| **Mean Reciprocal Rank (MRR)**                   | Evaluates the rank position of the first relevant document.                                                    | `MRR = (1 ÷ ABS(Q)) × Σ(1 ÷ rank_i) for i = 1 to ABS(Q)`                                                                                    |
| **F1 Score**                                     | Harmonic mean of precision and recall.                                                                         | `F1 = 2 × (Precision × Recall) ÷ (Precision + Recall) `                                                                                     |
| **Area Under the ROC Curve (AUC-ROC)**           | Measures the ability of the model to distinguish between relevant and non-relevant documents.                  | AUC = Area under the ROC curve, which plots true positive rate (TPR) vs. false positive rate (FPR).                                         |
| **Mean Rank (MR)**                               | The average rank of the first relevant document across all queries.                                            | Lower values indicate better performance.                                                                                                   |
| **Hit Rate (HR) or Recall at k**                 | Measures the proportion of queries for which at least one relevant document is retrieved in the top k results. | `HR@k = (Queries with at least one relevant document in top k) ÷ ABS(Q)`                                                                    |
| **Expected Reciprocal Rank (ERR)**               | Measures the probability that a user finds a relevant document at each position in the ranked list.            | `ERR = Σ(1 ÷ i) × Π(1 − r_j) × r_i for j = 1 to i−1`<br> where, `r_i` = relevance probability of the document at position i.                |

**Evaluation Process:**

- Create a dataset with queries and corresponding ground truth answers (e.g., relevant documents).
- For each query, retrieve the top-k documents and compare them against the ground truth.
- Compute the relevant metrics (e.g., Precision@k, Recall@k, MRR, etc.).

### Generating the Ground Truth Datasets

A critical aspect of evaluation is the use of gold-standard data sets. These are ground truth data sets where the relevant documents for each query are known. For instance, the relevant documents would be pre-identified given a query like "Can I still join the course?". This allows for a clear benchmark to assess the performance of different retrieval methods. An oversimplified illustration of how such a predetermined "Ground-Truth" dataset might look is as follows:

```text
Query: I just discovered the course. Can I still join?

Relevant documents: doc1, doc10, doc 11

Query: Windows or Linux?

Relevant documents: doc4
```

> Note: Typically, for each query, there are multiple documents but for the sake of simplicity above is a query-document pair.

**Preparing the documents**

For generating the `Ground Truth Datasets`, there are a few approaches:

1. Human Annotation

In production systems, human annotators and domain experts review documents and queries to identify relevant document-query pairs. Although this method is time-consuming, it results in high-quality data, often referred to as "gold standard" data. Observing user interactions and evaluating system responses also contribute to refining the data set.

2. Using Large Language Models (LLMs)

LLMs can be used to generate user queries based on FAQ records. By creating multiple questions for each FAQ record, we ensure that the generated questions are varied and relevant. This automated approach speeds up the process of creating a ground truth data set and is suitable for initial experiments before deploying a production system.

Let us proceed with generating the ground truth datasets using `gpt-3.5-turbo`. As usual, after loading and processing our data, we need to generate stable IDs for the documents (i.e. for each record)

To accurately track relevant documents, each document is assigned a unique ID. Maintaining consistent IDs allows us to manage changes and updates to the document set without affecting the evaluation process. This helps know which answer goes with which question.

Here are the steps to generate stable IDs:

Step 1: Concatenate Document Attributes: Combine key attributes of the document (e.g., course name, question, and a portion of the text) into a single string. This ensures that the ID is unique to the specific content of the document.

Step 2: Generate MD5 Hash: Use the MD5 hashing algorithm to create a hash from the concatenated string. MD5 is chosen for its balance of speed and uniqueness.

Step 3: Extract a Substring of the Hash: To keep the ID concise, extract a substring (e.g., the first 8 characters) of the MD5 hash. This substring serves as the document's unique ID.

Step 4: Assign the ID to the Document: Attach the generated ID to the document. This ID will be used to reference the document in the evaluation process.

A simple function that describe the steps above as follows:

```python
# a simple function to generate hash ids based on the concatenation of all our dictionary values
def generate_doc_id(doc:dict) -> dict:

    # first let's Concatenate the different fields together
    combined = "-".join(doc.values())

    # now to hash our combined unique id
    hash_object = hashlib.md5(combined.encode()) # converts string to bytes

    # generates the MD5 hash of the encoded string and converts it to a hexidecimal string
    hash_hex = hash_object.hexdigest()

    return hash_hex[:8]  # only returning the first 8 characters of the hexidecimal string
```

Next, we want to use a LLM Model to generate questions for each record ID. But before we do that, we need to define our prompt template as well as a function that helps generate the questions based on the prompt.

Our prompt template would look something like this:

```python
# now to create out prompt template - we will use the template provided in the course

prompt_template = """
You emulate a student who's taking our course.
Formulate 5 questions this student might ask based on a FAQ record. The record
should contain the answer to the questions, and the questions should be complete and not too short.
If possible, use as fewer words as possible from the record.

The record:

section: {section}
question: {question}
answer: {text}

Provide the output in parsable JSON without using code blocks:

["question1", "question2", ..., "question5"]
""".strip()
```

You would notice that the template takes in the keys for each document dictionary as the argument to be put into the LLM model as context.

Next, the function to generate the question.

```python
# next we want to write a simple function that generates the question for each record:
def generate_questions(doc_dict):
    # each key in doc_dict corresponds to a placeholder in prompt_template, and the associated value will be inserted into the template
    prompt = prompt_template.format(**doc_dict)

    responses = openai_client.chat.completions.create(
        model = 'gpt-3.5-turbo',
        messages = [{'role':'user', 'content': prompt}]
    )

    return responses.choices[0].message.content
```

We now have to run the following to get the function to generate 5 questions for each record.

```python
# now to generate the questions for each record id
results = [{'Course': doc['course'], 'document_ID': doc['id'], 'Questions': generate_questions(doc)} for doc in tqdm(documents_updated)]
```

This would return a list of dictionaries, and all you need to do is to convert it into a dataframe before throwing out a `.csv` file in your local directory.

> Note: The questions generated are a JSON response and can be converted into a python object using json.load() to do so. However, there are bound to be parsing issues in the conversion.

Please refer to notebook that includes some parsing steps that was not covered in this README.md file [here](https://github.com/peterchettiar/LLMzoomcamp_2024/blob/main/Module-3-vector-databases/generate_gt_data_example.ipynb).

### Ranking Evaluation: Text Search

In this section, we will be evaluating the two search engines we had created in the previous lecturs for text search; `elasticsearch` and `minsearch`. As a quick refresh, we would start off by intialising each search engine with their respective index settings as well as populating them with our [documents_with_ids.json](https://github.com/peterchettiar/LLMzoomcamp_2024/blob/main/Module-3-vector-databases/documents_with_ids.json) file.

So once that is done, we have to evalute our search engine through the use of the following two metrics:

1. `Hit Rate` - Hit rate is the percentage of successful results (or "hits") where the system returns at least one relevant document or result for a given query.
2. `Mean Reciprocal Rank` - MRR is the average of the reciprocal ranks of the first relevant result for each query. The reciprocal rank for a query is the inverse of the rank (position) at which the first relevant result appears.

But to do so, we need to execute each query that we had generated in our [ground_truth_dataset](https://github.com/peterchettiar/LLMzoomcamp_2024/blob/main/Module-3-vector-databases/ground-truth-data.csv) in the respective search engines to retrieve the `document_id` and compare it to the `document_id` and see if the result is the same or at least ranked closely.

Hence, based on what was discussed the steps is as follows:

1. Make sure to connect to `elasticsearch`, test the connection if needed.
```python
# remember to run `sudo chown -R 1000:1000 /tmp/elasticsearch_data` after docker-compose up
es_client = Elasticsearch("http://localhost:9200")

if es_client.ping():
    print("Connected to ElasticSearch!")
else:
    print("Connection Failed.")
```
2. Next to create the index as well as defining its setttings.
```python
# now to create a new index as well as defining the index settings

index_settings = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"},
            "id": {"type": "keyword"},
        }
    }
}

index_name = "course-questions"

# lets delete and create a new index if it exists for ease of re-runs
if es_client.indices.exists(index="course-questions"):
    es_client.indices.delete(index="course-questions")

es_client.indices.create(index=index_name, body=index_settings)
```
3. Followed by populating the index, then defining a search funtion - this had been done so in previous lectures but if need be feel free to follow the notebook for the current lecture [here](https://github.com/peterchettiar/LLMzoomcamp_2024/blob/main/Module-3-vector-databases/evaluating_text_search.ipynb).
4. After loading our golden truth dataset, we need to define a function to return the hit rate (i.e. 1 if any of the docs returned is relevant or otherwise 0) for each query from GTD.
```python
def hit_rate(course:str, doc_id:str, query:str):
    query_res = elasticsearch_query(query=query, course=course)

    hit_rate = [True if res["_source"]['id'] == doc_id else False for res in query_res]

    if any(hit_rate):
        return 1

    return 0
```
5. Similaryly, we need to define a function for our Mean Reciprocal Rank.
```python
def mrr(course:str, doc_id:str, query:str) -> int:
    query_res = elasticsearch_query(query=query, course=course)

    id_res = [res['_source']['id'] for res in query_res]

    for index, id in enumerate(id_res):
        if id == doc_id:
            return 1/(index+1)
    return 0
```
Similar steps were performed for `minsearch` search engine as well, and conclusively `elasticsearch` had a better performance and accuracy for retrieval of relevant documents based on the two metrics discussed in detail so far. Hence, I will not delve into the details for the `minsearch` evaluation functions. Instead I would encourage for you to go through the [evaluation-text-search](https://github.com/peterchettiar/LLMzoomcamp_2024/blob/main/Module-3-vector-databases/evaluating_text_search.ipynb) instead.

### Ranking Evaluation: Vector Search

This process is very similar to our previous section on text search, with small minor changes.

1. Index settings - we need to define our three dense vectors (i.e. `question_vector`,`text_vector`,`question_text_vector`) as these would be the fields that our query vector would be performing the search on in our index.
```python
index_settings = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"},
            "id": {"type": "keyword"},
            "question_vector" :{
                "type": "dense_vector",
                "dims": 384,
                "index": True,
                "similarity": "cosine"
            },
            "text_vector":{
                "type": "dense_vector",
                "dims": 384,
                "index": True,
                "similarity": "cosine", # there is also another option of using dotp product which should yield the same result
            },
            "question_text_vector":{
                "type": "dense_vector",
                "dims": 384,
                "index": True,
                "similarity": "cosine"
            }
        }
    }
}
```
2. Next, we want to intialise as well as transform our above mentioned fields into their respective dense vectors / vector embeddings.
```python
# Now that we have defined the schema of our index, we can proceed create the vector embeddings before populating it

# but first we need to load a pretrained sentence transformer model that would convert our questions and text to vector embeddings
model = SentenceTransformer("all-MiniLM-L12-v2")

# next we want to update our documents variable to include our vector embeddings while retaining the original data
documents = [doc.update({"question_vector": model.encode(doc["question"]),
                         "text_vector": model.encode(doc["text"]),
                         "question_text_vector": model.encode(doc["question"] + '-' + doc["text"])}) or doc for doc in documents]
```
3. Now to define our search query function - you would notice there are 3 arguments for the function (field, vector and course), field refers to the dense vector that our query vector would be compared with and relevant documents retrieved.
```python
# last step for this function would be to define the search query function for elasticsearch
def elastic_search_knn(field, vector, course):
    knn = {
        "field": field,
        "query_vector": vector,
        "k": 5,
        "num_candidates": 10000,
        "filter": {
            "term": {
                "course": course
            }
        }
    }

    search_query = {
        "knn": knn,
        "_source": ["text", "section", "question", "course", "id"]
    }

    es_results = es_client.search(
        index=index_name,
        body=search_query
    )

    result_docs = [hit['_source'] for hit in es_results['hits']['hits']]

    return result_docs
```
4. Lastly, let us run the following function to evaluate the performance of `elasticsearch` based on `hit rate` and `mrr`. The functions for the metric called within the evaluation function is very similar to the ones defined in text search. Only difference is the `index_field` argument that we mentioned in the previous point.
```python
def evaluate(index_field, df):
    # calculating hit rate
    df[f'hit_rate_{index_field}'] = df.apply(lambda x : hit_rate_elasticserach(index_field,x['Course'],x['document_ID'],x['Question']), axis=1)
    hit_rate = df[f'hit_rate_{index_field}'].sum() / df[f'hit_rate_{index_field}'].count()

    # calculating mrr
    df[f'{index_field}_recip_rank'] = df.apply(lambda x : mrr_elasticsearch(index_field,x['Course'],x['document_ID'],x['Question']), axis=1)
    mrr = df[f'{index_field}_recip_rank'].sum() / df[f'{index_field}_recip_rank'].count() 

    # we want to drop these columns that we created
    df.drop(columns=[f"hit_rate_{index_field}", f"{index_field}_recip_rank"], inplace=True)

    return {
        f'hit rate for {index_field}' : hit_rate,
        f'mrr for {index_field}' : mrr
    }
```
5. As an extra step in order to improve the performance of `elasticsearch` we can combine our 3 vector embeddings in our index into 1 vector and compare our query vector. Following is an updated function to do just that using the `script_score` feature.
```python
# updated elasticsearch search function - combining all three dense vectors
def elastic_search_knn_combined(vector, course):
    search_query = {
        "size": 5,
        "query": {
            "bool": {
                "must": [
                    {
                        "script_score": {
                            "query": {
                                "term": {
                                    "course": course
                                }
                            },
                            "script": {
                                "source": """
                                // Calculate cosine similarity for all vectors
                                double sim1 = cosineSimilarity(params.query_vector, 'question_vector');
                                double sim2 = cosineSimilarity(params.query_vector, 'text_vector');
                                double sim3 = cosineSimilarity(params.query_vector, 'question_text_vector');

                                // Map cosine similarity from range [-1,1] to [0,1]
                                sim1 = (sim1 + 1) / 2;
                                sim2 = (sim2 + 1) / 2;
                                sim3 = (sim3 + 1) / 2;

                                // Combine the scores (e.g., sum of average)
                                return sim1 + sim2 + sim3;
                                """,
                                "params": {
                                    "query_vector": vector
                                }
                            }
                        }
                    }
                ],
                "filter": {
                    "term": {
                        "course": course
                    }
                }
            }
        },
        "_source" : ["text", "section", "question", "course", "id"]
    }

    es_results = es_client.search(
        index=index_name,
        body=search_query
    )

    result_docs = [hit['_source'] for hit in es_results['hits']['hits']]

    return result_docs
```

> Note: In the script section of the function, we have re-map the scores for cosine similarity as it does return negative scores which is not accepted by elastic search.
