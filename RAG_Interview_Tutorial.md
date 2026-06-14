# RAG (Retrieval-Augmented Generation) Interview Tutorial - Quick Recap Guide

## Table of Contents
1. [RAG Fundamentals](#rag-fundamentals)
2. [RAG Architecture](#rag-architecture)
3. [Document Processing](#document-processing)
4. [Embeddings & Vector Stores](#embeddings--vector-stores)
5. [Retrieval Strategies](#retrieval-strategies)
6. [Generation & Prompting](#generation--prompting)
7. [RAG Optimization](#rag-optimization)
8. [Evaluation Metrics](#evaluation-metrics)
9. [Advanced RAG Techniques](#advanced-rag-techniques)
10. [Production Considerations](#production-considerations)
11. [Common Interview Questions](#common-interview-questions)
---

## RAG Fundamentals

### What is RAG?
**Retrieval-Augmented Generation (RAG)** is a technique that enhances Large Language Models (LLMs) by retrieving relevant information from external knowledge sources before generating responses.

### Why RAG?
1. **Reduces Hallucinations**: Grounds responses in factual data
2. **Current Information**: Access to up-to-date knowledge
3. **Domain-Specific Knowledge**: Incorporate specialized information
4. **Cost-Effective**: Cheaper than fine-tuning for knowledge updates
5. **Transparency**: Can cite sources
6. **Dynamic Updates**: Easy to update knowledge base

### RAG vs Fine-Tuning

| Aspect | RAG | Fine-Tuning |
|--------|-----|-------------|
| **Knowledge Update** | Easy (add documents) | Requires retraining |
| **Cost** | Lower | Higher |
| **Latency** | Slightly higher | Lower |
| **Transparency** | High (can cite sources) | Low |
| **Use Case** | Factual Q&A, documents | Behavior, style, format |
| **Maintenance** | Easy | Complex |

### Basic RAG Flow
```
User Query → Embedding → Vector Search → Retrieve Docs → 
Augment Prompt → LLM Generation → Response
```

### Interview Question 1: What is RAG and when should you use it?

**Answer:**
**RAG (Retrieval-Augmented Generation)** combines information retrieval with text generation to provide accurate, grounded responses.

**When to use RAG:**

1. **Factual Q&A Systems**: Customer support, documentation
2. **Domain-Specific Applications**: Legal, medical, technical
3. **Dynamic Knowledge**: Frequently updated information
4. **Source Attribution**: Need to cite sources
5. **Large Knowledge Bases**: Too much to fit in context

**When NOT to use RAG:**

1. **Creative Writing**: No factual grounding needed
2. **General Conversation**: No specific knowledge required
3. **Behavior Changes**: Use fine-tuning instead
4. **Real-time Data**: Use APIs or function calling

**Example:**
```python
# Good use case: Document Q&A
query = "What is the refund policy?"
# RAG retrieves relevant policy documents
# LLM generates answer based on retrieved docs

# Bad use case: Creative story
query = "Write a fantasy story about dragons"
# No need for retrieval, pure generation works better
```

---

## RAG Architecture

### Basic RAG Architecture
```
┌─────────────┐
│   Documents │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Chunking   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Embedding  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Vector Store│
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌──────────┐
│ User Query  │────▶│ Embedding│
└─────────────┘     └────┬─────┘
                         │
                         ▼
                  ┌──────────────┐
                  │   Retrieval  │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │   Augment    │
                  │    Prompt    │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │     LLM      │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │   Response   │
                  └──────────────┘
```

### Components Breakdown

**1. Indexing Phase (Offline):**
- Load documents
- Split into chunks
- Generate embeddings
- Store in vector database

**2. Retrieval Phase (Online):**
- Embed user query
- Search vector database
- Retrieve top-k relevant chunks
- Rank and filter results

**3. Generation Phase (Online):**
- Construct prompt with context
- Send to LLM
- Generate response
- Post-process output

### Simple RAG Implementation
```python
from langchain.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chat_models import ChatOpenAI
from langchain.chains import RetrievalQA

# 1. Load documents
loader = TextLoader("documents.txt")
documents = loader.load()

# 2. Split into chunks
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
chunks = text_splitter.split_documents(documents)

# 3. Create embeddings and vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)

# 4. Create retrieval chain
llm = ChatOpenAI(model="gpt-3.5-turbo")
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectorstore.as_retriever(search_kwargs={"k": 3})
)

# 5. Query
query = "What is the company's refund policy?"
response = qa_chain.run(query)
print(response)
```

### Interview Question 2: Explain the RAG pipeline and its components.

**Answer:**
The RAG pipeline consists of three main phases:

**1. Indexing Phase (Offline):**
```python
# Load documents
documents = load_documents("data/")

# Split into chunks
chunks = split_documents(documents, chunk_size=1000)

# Generate embeddings
embeddings = embed_chunks(chunks)

# Store in vector database
vector_db.add(embeddings, chunks)
```

**2. Retrieval Phase (Online):**
```python
# Embed query
query_embedding = embed_query(user_query)

# Search for similar chunks
relevant_chunks = vector_db.search(
    query_embedding, 
    top_k=5
)

# Re-rank results (optional)
ranked_chunks = rerank(relevant_chunks, user_query)
```

**3. Generation Phase (Online):**
```python
# Construct prompt
prompt = f"""
Context: {relevant_chunks}

Question: {user_query}

Answer based on the context above:
"""

# Generate response
response = llm.generate(prompt)
```

**Key Components:**
- **Document Loader**: Reads various file formats
- **Text Splitter**: Breaks documents into chunks
- **Embedding Model**: Converts text to vectors
- **Vector Store**: Stores and searches embeddings
- **Retriever**: Finds relevant documents
- **LLM**: Generates final response

---

## Document Processing

### Document Loading
```python
from langchain.document_loaders import (
    TextLoader,
    PDFLoader,
    CSVLoader,
    UnstructuredMarkdownLoader,
    WebBaseLoader
)

# Text files
loader = TextLoader("document.txt")

# PDF files
loader = PDFLoader("document.pdf")

# CSV files
loader = CSVLoader("data.csv")

# Markdown files
loader = UnstructuredMarkdownLoader("readme.md")

# Web pages
loader = WebBaseLoader("https://example.com")

# Load multiple files
from langchain.document_loaders import DirectoryLoader

loader = DirectoryLoader(
    "data/",
    glob="**/*.pdf",
    loader_cls=PDFLoader
)
documents = loader.load()
```

### Text Chunking Strategies

**1. Fixed-Size Chunking:**
```python
from langchain.text_splitter import CharacterTextSplitter

splitter = CharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separator="\n"
)
chunks = splitter.split_documents(documents)
```

**2. Recursive Chunking (Recommended):**
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", " ", ""]
)
chunks = splitter.split_documents(documents)
```

**3. Semantic Chunking:**
```python
from langchain.text_splitter import SemanticChunker

splitter = SemanticChunker(
    embeddings=OpenAIEmbeddings(),
    breakpoint_threshold_type="percentile"
)
chunks = splitter.split_documents(documents)
```

**4. Markdown-Aware Chunking:**
```python
from langchain.text_splitter import MarkdownTextSplitter

splitter = MarkdownTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
chunks = splitter.split_documents(documents)
```

### Chunk Size Considerations

| Chunk Size | Pros | Cons | Use Case |
|------------|------|------|----------|
| **Small (200-500)** | Precise retrieval | May lack context | Specific facts |
| **Medium (500-1000)** | Balanced | General purpose | Most applications |
| **Large (1000-2000)** | More context | Less precise | Complex topics |

### Metadata Enrichment
```python
from langchain.schema import Document

# Add metadata to chunks
for i, chunk in enumerate(chunks):
    chunk.metadata = {
        "source": "document.pdf",
        "page": chunk.metadata.get("page", 0),
        "chunk_id": i,
        "created_at": "2024-01-01",
        "category": "technical",
        "author": "John Doe"
    }
```

### Interview Question 3: How do you choose the right chunk size for RAG?

**Answer:**
Chunk size selection depends on several factors:

**Factors to Consider:**

1. **Content Type:**
   - **Short Q&A**: 200-500 tokens
   - **Technical docs**: 500-1000 tokens
   - **Long-form content**: 1000-2000 tokens

2. **LLM Context Window:**
   - Must fit: query + retrieved chunks + prompt
   - Leave room for generation

3. **Retrieval Precision:**
   - Smaller chunks: More precise, less context
   - Larger chunks: More context, less precise

4. **Domain Complexity:**
   - Simple topics: Smaller chunks
   - Complex topics: Larger chunks

**Best Practices:**

```python
# Experiment with different sizes
chunk_sizes = [500, 1000, 1500]
overlaps = [100, 200, 300]

for size in chunk_sizes:
    for overlap in overlaps:
        splitter = RecursiveCharacterTextSplitter(
            chunk_size=size,
            chunk_overlap=overlap
        )
        # Test and evaluate
        evaluate_rag_performance(splitter)
```

**Recommended Starting Point:**
- **Chunk size**: 1000 tokens
- **Overlap**: 200 tokens (20%)
- **Then optimize** based on evaluation metrics

**Overlap Importance:**
```python
# Without overlap - may lose context
chunk1 = "...end of sentence"
chunk2 = "Start of next..."

# With overlap - maintains context
chunk1 = "...end of sentence. Start of next..."
chunk2 = "end of sentence. Start of next..."
```

---

## Embeddings & Vector Stores

### Embedding Models

**1. OpenAI Embeddings:**
```python
from langchain.embeddings import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small"  # or text-embedding-3-large
)

# Generate embedding
text = "This is a sample text"
vector = embeddings.embed_query(text)
print(f"Dimension: {len(vector)}")  # 1536 for small, 3072 for large
```

**2. Hugging Face Embeddings:**
```python
from langchain.embeddings import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)
```

**3. Cohere Embeddings:**
```python
from langchain.embeddings import CohereEmbeddings

embeddings = CohereEmbeddings(
    model="embed-english-v3.0"
)
```

### Popular Embedding Models

| Model | Dimension | Performance | Cost | Use Case |
|-------|-----------|-------------|------|----------|
| **text-embedding-3-small** | 1536 | Good | Low | General purpose |
| **text-embedding-3-large** | 3072 | Excellent | Medium | High accuracy |
| **all-MiniLM-L6-v2** | 384 | Good | Free | Budget-friendly |
| **all-mpnet-base-v2** | 768 | Very Good | Free | Open-source |
| **embed-english-v3.0** | 1024 | Excellent | Medium | Cohere users |

### Vector Databases

**1. Chroma (Local, Simple):**
```python
from langchain.vectorstores import Chroma

vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# Query
results = vectorstore.similarity_search("query", k=5)
```

**2. Pinecone (Cloud, Scalable):**
```python
import pinecone
from langchain.vectorstores import Pinecone

pinecone.init(api_key="your-api-key", environment="us-west1-gcp")

vectorstore = Pinecone.from_documents(
    documents=chunks,
    embedding=embeddings,
    index_name="my-index"
)
```

**3. Weaviate (Open-Source, Feature-Rich):**
```python
from langchain.vectorstores import Weaviate
import weaviate

client = weaviate.Client("http://localhost:8080")

vectorstore = Weaviate.from_documents(
    documents=chunks,
    embedding=embeddings,
    client=client,
    index_name="Document"
)
```

**4. FAISS (Local, Fast):**
```python
from langchain.vectorstores import FAISS

vectorstore = FAISS.from_documents(
    documents=chunks,
    embedding=embeddings
)

# Save locally
vectorstore.save_local("faiss_index")

# Load
vectorstore = FAISS.load_local("faiss_index", embeddings)
```

### Similarity Search Methods

**1. Cosine Similarity (Most Common):**
```python
results = vectorstore.similarity_search(
    query="What is RAG?",
    k=5
)
```

**2. Maximum Marginal Relevance (MMR):**
```python
# Balances relevance and diversity
results = vectorstore.max_marginal_relevance_search(
    query="What is RAG?",
    k=5,
    fetch_k=20,  # Fetch more candidates
    lambda_mult=0.5  # 0=diversity, 1=relevance
)
```

**3. Similarity Search with Score:**
```python
results = vectorstore.similarity_search_with_score(
    query="What is RAG?",
    k=5
)

for doc, score in results:
    print(f"Score: {score}, Content: {doc.page_content[:100]}")
```

**4. Metadata Filtering:**
```python
results = vectorstore.similarity_search(
    query="What is RAG?",
    k=5,
    filter={"category": "technical", "year": 2024}
)
```

### Interview Question 4: Compare different vector databases and when to use each.

**Answer:**

**Vector Database Comparison:**

**1. Chroma:**
- **Pros**: Easy setup, local, good for development
- **Cons**: Limited scalability
- **Use Case**: Prototyping, small projects
```python
vectorstore = Chroma.from_documents(chunks, embeddings)
```

**2. Pinecone:**
- **Pros**: Fully managed, scalable, fast
- **Cons**: Paid service, vendor lock-in
- **Use Case**: Production, large-scale
```python
vectorstore = Pinecone.from_documents(chunks, embeddings, index_name="prod")
```

**3. Weaviate:**
- **Pros**: Open-source, feature-rich, hybrid search
- **Cons**: Requires setup and maintenance
- **Use Case**: Self-hosted production
```python
vectorstore = Weaviate.from_documents(chunks, embeddings, client=client)
```

**4. FAISS:**
- **Pros**: Very fast, local, free
- **Cons**: In-memory, no persistence by default
- **Use Case**: Research, experiments
```python
vectorstore = FAISS.from_documents(chunks, embeddings)
```

**5. Qdrant:**
- **Pros**: Fast, open-source, good filtering
- **Cons**: Newer, smaller community
- **Use Case**: High-performance needs

**Decision Matrix:**

| Need | Recommendation |
|------|----------------|
| **Quick prototype** | Chroma, FAISS |
| **Production (managed)** | Pinecone, Weaviate Cloud |
| **Production (self-hosted)** | Weaviate, Qdrant |
| **Budget-friendly** | Chroma, FAISS, Weaviate |
| **Hybrid search** | Weaviate, Qdrant |
| **Maximum speed** | FAISS, Qdrant |

---

## Retrieval Strategies

### Basic Retrieval
```python
from langchain.vectorstores import Chroma

# Create retriever
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 5}
)

# Retrieve documents
docs = retriever.get_relevant_documents("What is RAG?")
```

### Advanced Retrieval Techniques

**1. Multi-Query Retrieval:**
```python
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(temperature=0)

retriever = MultiQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(),
    llm=llm
)

# Generates multiple query variations
# "What is RAG?" → 
#   - "Explain retrieval augmented generation"
#   - "How does RAG work?"
#   - "What are the benefits of RAG?"

docs = retriever.get_relevant_documents("What is RAG?")
```

**2. Contextual Compression:**
```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor

llm = ChatOpenAI(temperature=0)
compressor = LLMChainExtractor.from_llm(llm)

compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=vectorstore.as_retriever()
)

# Compresses retrieved documents to only relevant parts
docs = compression_retriever.get_relevant_documents("What is RAG?")
```

**3. Parent Document Retriever:**
```python
from langchain.retrievers import ParentDocumentRetriever
from langchain.storage import InMemoryStore

# Store full documents
store = InMemoryStore()

# Retrieve small chunks, return full parent documents
retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter
)
```

**4. Ensemble Retriever:**
```python
from langchain.retrievers import EnsembleRetriever
from langchain.retrievers import BM25Retriever

# Combine vector search with keyword search
bm25_retriever = BM25Retriever.from_documents(documents)
vector_retriever = vectorstore.as_retriever()

ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, vector_retriever],
    weights=[0.5, 0.5]
)

docs = ensemble_retriever.get_relevant_documents("What is RAG?")
```

### Hybrid Search
```python
# Combine semantic and keyword search
results = vectorstore.similarity_search(
    query="machine learning",
    k=10,
    search_type="hybrid",  # Weaviate, Qdrant support this
    alpha=0.5  # 0=keyword, 1=semantic
)
```

### Re-ranking
```python
from langchain.retrievers.document_compressors import CohereRerank

# Initial retrieval
docs = vectorstore.similarity_search(query, k=20)

# Re-rank with Cohere
reranker = CohereRerank(model="rerank-english-v2.0")
reranked_docs = reranker.compress_documents(docs, query)[:5]
```


### Interview Question 5: What are advanced retrieval techniques and when to use them?

**Answer:**

**Advanced Retrieval Techniques:**

**1. Multi-Query Retrieval:**
- **What**: Generates multiple query variations
- **When**: User queries are ambiguous or short
- **Benefit**: Better recall, handles query variations

```python
# Single query: "RAG"
# Generated queries:
# - "What is Retrieval Augmented Generation?"
# - "How does RAG work?"
# - "RAG architecture explained"
```

**2. Hybrid Search:**
- **What**: Combines semantic + keyword search
- **When**: Need both meaning and exact matches
- **Benefit**: Better precision and recall

```python
# Semantic: "ML algorithms" matches "machine learning"
# Keyword: "ML" matches exact term "ML"
# Hybrid: Gets both
```

**3. Re-ranking:**
- **What**: Re-scores retrieved documents
- **When**: Initial retrieval returns many results
- **Benefit**: Improves relevance of top results

```python
# Retrieve 20 candidates
# Re-rank to get best 5
# More accurate than retrieving 5 directly
```

**4. Contextual Compression:**
- **What**: Extracts only relevant parts
- **When**: Documents are long, context window limited
- **Benefit**: Reduces noise, fits more context

**5. Parent Document Retrieval:**
- **What**: Search small chunks, return full documents
- **When**: Need context around matched text
- **Benefit**: Precise search, complete context

---

## Generation & Prompting

### Basic RAG Prompt
```python
from langchain.prompts import PromptTemplate

template = """
Use the following context to answer the question.
If you don't know the answer, say "I don't know".

Context: {context}

Question: {question}

Answer:
"""

prompt = PromptTemplate(
    template=template,
    input_variables=["context", "question"]
)
```

### Advanced RAG Prompts

**1. With Source Citation:**
```python
template = """
Answer the question based on the context below.
Include citations using [Source X] format.

Context:
{context}

Question: {question}

Answer with citations:
"""
```

**2. With Confidence Scoring:**
```python
template = """
Answer the question based on the context.
Rate your confidence (Low/Medium/High).

Context: {context}
Question: {question}

Answer:
Confidence:
Reasoning:
"""
```

**3. With Step-by-Step Reasoning:**
```python
template = """
Answer the question using the context provided.
Think step by step.

Context: {context}
Question: {question}

Let's think step by step:
1.
2.
3.

Final Answer:
"""
```

### RAG Chain Types

**1. Stuff Chain (Simple):**
```python
from langchain.chains import RetrievalQA

qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",  # Stuffs all docs into prompt
    retriever=retriever
)
```

**2. Map-Reduce Chain:**
```python
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="map_reduce",  # Process each doc separately, then combine
    retriever=retriever
)
```

**3. Refine Chain:**
```python
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="refine",  # Iteratively refine answer with each doc
    retriever=retriever
)
```

**4. Map-Rerank Chain:**
```python
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="map_rerank",  # Score each doc's answer, return best
    retriever=retriever
)
```

### Interview Question 6: How do you design effective prompts for RAG systems?

**Answer:**

**Effective RAG Prompt Design:**

**1. Clear Instructions:**
```python
# Bad
template = "Context: {context}\nQuestion: {question}"

# Good
template = """
Answer the question using ONLY the context provided.
If the answer is not in the context, say "I don't have enough information."

Context: {context}
Question: {question}

Answer:
"""
```

**2. Source Attribution:**
```python
template = """
Answer based on the context and cite sources.

Context:
[1] {doc1}
[2] {doc2}

Question: {question}

Answer with citations [1], [2]:
"""
```

**3. Output Format:**
```python
template = """
Answer in the following format:

Answer: [Your answer]
Confidence: [Low/Medium/High]
Sources: [List sources]

Context: {context}
Question: {question}
"""
```

**Best Practices:**
- Be explicit about using only provided context
- Request source citations
- Handle cases where answer isn't available
- Specify output format
- Use examples for complex tasks

---

## RAG Optimization

### Chunk Optimization
```python
# Experiment with different chunk sizes
def optimize_chunk_size(documents, test_queries):
    results = {}
    
    for chunk_size in [500, 1000, 1500, 2000]:
        for overlap in [100, 200, 300]:
            splitter = RecursiveCharacterTextSplitter(
                chunk_size=chunk_size,
                chunk_overlap=overlap
            )
            chunks = splitter.split_documents(documents)
            
            # Create vectorstore
            vectorstore = Chroma.from_documents(chunks, embeddings)
            
            # Evaluate
            score = evaluate_retrieval(vectorstore, test_queries)
            results[(chunk_size, overlap)] = score
    
    return results
```

### Query Optimization

**1. Query Expansion:**
```python
def expand_query(query, llm):
    prompt = f"""
    Generate 3 variations of this query:
    Original: {query}
    
    Variations:
    1.
    2.
    3.
    """
    variations = llm.predict(prompt)
    return [query] + parse_variations(variations)
```

**2. Query Rewriting:**
```python
def rewrite_query(query, llm):
    prompt = f"""
    Rewrite this query to be more specific and searchable:
    Original: {query}
    
    Rewritten:
    """
    return llm.predict(prompt)
```

### Retrieval Optimization

**1. Adjust Top-K:**
```python
# Retrieve more, then filter
docs = retriever.get_relevant_documents(query, k=20)

# Filter by score threshold
filtered_docs = [doc for doc, score in docs if score > 0.7]

# Use top-N after filtering
top_docs = filtered_docs[:5]
```

**2. Metadata Filtering:**
```python
# Pre-filter by metadata
retriever = vectorstore.as_retriever(
    search_kwargs={
        "k": 5,
        "filter": {
            "category": "technical",
            "date": {"$gte": "2024-01-01"}
        }
    }
)
```

### Interview Question 7: How do you optimize RAG performance?

**Answer:**

**RAG Optimization Strategies:**

**1. Chunk Size Optimization:**
- Test different sizes (500, 1000, 1500)
- Measure retrieval accuracy
- Balance precision vs context

**2. Embedding Model Selection:**
- Better embeddings = better retrieval
- Trade-off: accuracy vs cost/speed
- Consider domain-specific models

**3. Retrieval Strategy:**
- Use hybrid search (semantic + keyword)
- Implement re-ranking
- Adjust top-k parameter

**4. Query Enhancement:**
- Query expansion for better recall
- Query rewriting for clarity
- Multi-query retrieval

**5. Prompt Engineering:**
- Clear instructions
- Source attribution
- Handle edge cases

**6. Caching:**
- Cache embeddings
- Cache frequent queries
- Cache LLM responses

**Performance Metrics to Track:**
- Retrieval precision/recall
- Answer relevance
- Latency
- Cost per query

---

## Evaluation Metrics

### Retrieval Metrics

**1. Precision:**
```python
# Precision = Relevant Retrieved / Total Retrieved
def precision(retrieved_docs, relevant_docs):
    relevant_retrieved = set(retrieved_docs) & set(relevant_docs)
    return len(relevant_retrieved) / len(retrieved_docs)
```

**2. Recall:**
```python
# Recall = Relevant Retrieved / Total Relevant
def recall(retrieved_docs, relevant_docs):
    relevant_retrieved = set(retrieved_docs) & set(relevant_docs)
    return len(relevant_retrieved) / len(relevant_docs)
```

**3. F1 Score:**
```python
def f1_score(precision, recall):
    return 2 * (precision * recall) / (precision + recall)
```

**4. Mean Reciprocal Rank (MRR):**
```python
def mrr(retrieved_docs, relevant_doc):
    for i, doc in enumerate(retrieved_docs):
        if doc == relevant_doc:
            return 1 / (i + 1)
    return 0
```

### Generation Metrics

**1. Faithfulness:**
```python
# Answer is grounded in retrieved context
from ragas import faithfulness

score = faithfulness.score(
    question=question,
    answer=answer,
    contexts=retrieved_docs
)
```

**2. Answer Relevance:**
```python
# Answer addresses the question
from ragas import answer_relevancy

score = answer_relevancy.score(
    question=question,
    answer=answer
)
```

**3. Context Relevance:**
```python
# Retrieved context is relevant to question
from ragas import context_relevancy

score = context_relevancy.score(
    question=question,
    contexts=retrieved_docs
)
```

### End-to-End Evaluation
```python
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_relevancy,
    context_recall,
    context_precision
)

# Prepare dataset
dataset = {
    "question": ["What is RAG?", "How does it work?"],
    "answer": [answer1, answer2],
    "contexts": [contexts1, contexts2],
    "ground_truth": [truth1, truth2]
}

# Evaluate
results = evaluate(
    dataset,
    metrics=[
        faithfulness,
        answer_relevancy,
        context_relevancy,
        context_recall,
        context_precision
    ]
)

print(results)
```

### Interview Question 8: How do you evaluate RAG system performance?

**Answer:**

**RAG Evaluation Framework:**

**1. Retrieval Quality:**
- **Precision**: % of retrieved docs that are relevant
- **Recall**: % of relevant docs that are retrieved
- **MRR**: Position of first relevant document

```python
# Example
retrieved = ["doc1", "doc2", "doc3"]
relevant = ["doc1", "doc4"]

precision = 1/3  # Only doc1 is relevant
recall = 1/2     # Only doc1 out of 2 relevant docs retrieved
```

**2. Generation Quality:**
- **Faithfulness**: Answer grounded in context
- **Answer Relevance**: Addresses the question
- **Correctness**: Factually accurate

**3. End-to-End Metrics:**
- **Context Precision**: Relevant chunks in top-k
- **Context Recall**: All relevant info retrieved
- **Answer Similarity**: Compared to ground truth

**4. User-Centric Metrics:**
- **User satisfaction**: Thumbs up/down
- **Task completion**: Did it solve the problem?
- **Time to answer**: Latency

**Evaluation Process:**
```python
# 1. Create test dataset
test_cases = [
    {
        "question": "What is RAG?",
        "ground_truth": "RAG is...",
        "relevant_docs": ["doc1", "doc2"]
    }
]

# 2. Run RAG system
for case in test_cases:
    retrieved = retriever.get_docs(case["question"])
    answer = llm.generate(retrieved, case["question"])
    
    # 3. Evaluate
    metrics = {
        "precision": calculate_precision(retrieved, case["relevant_docs"]),
        "faithfulness": check_faithfulness(answer, retrieved),
        "relevance": check_relevance(answer, case["question"])
    }
```

---

## Advanced RAG Techniques

### 1. Agentic RAG
```python
from langchain.agents import create_react_agent

# Agent can decide when to retrieve
agent = create_react_agent(
    llm=llm,
    tools=[retriever_tool, calculator_tool],
    prompt=agent_prompt
)

# Agent reasoning:
# "User asks about company revenue in 2023"
# Thought: I need to retrieve financial documents
# Action: Use retriever tool
# Observation: Found revenue data
# Thought: Now I can answer
# Final Answer: The revenue was $X million
```

### 2. Self-RAG
```python
# Model decides when retrieval is needed
def self_rag(query, llm, retriever):
    # Step 1: Decide if retrieval needed
    need_retrieval = llm.predict(
        f"Does this query need external information? {query}"
    )
    
    if need_retrieval:
        # Step 2: Retrieve
        docs = retriever.get_relevant_documents(query)
        
        # Step 3: Generate with reflection
        answer = llm.predict(f"Context: {docs}\nQuestion: {query}")
        
        # Step 4: Self-critique
        is_supported = llm.predict(
            f"Is this answer supported by context? {answer}"
        )
        
        if not is_supported:
            # Retrieve more or refine
            return self_rag(query, llm, retriever)
    else:
        answer = llm.predict(query)
    
    return answer
```

### 3. Corrective RAG (CRAG)
```python
def corrective_rag(query, llm, retriever):
    # Initial retrieval
    docs = retriever.get_relevant_documents(query)
    
    # Evaluate relevance
    relevance_scores = evaluate_relevance(docs, query)
    
    if all(score > threshold for score in relevance_scores):
        # All docs relevant, proceed
        return generate_answer(docs, query)
    elif any(score > threshold for score in relevance_scores):
        # Some relevant, filter and proceed
        relevant_docs = filter_by_score(docs, relevance_scores)
        return generate_answer(relevant_docs, query)
    else:
        # No relevant docs, use web search
        web_results = web_search(query)
        return generate_answer(web_results, query)
```

### 4. HyDE (Hypothetical Document Embeddings)
```python
def hyde_retrieval(query, llm, vectorstore):
    # Generate hypothetical answer
    hypothetical_doc = llm.predict(
        f"Write a detailed answer to: {query}"
    )
    
    # Use hypothetical answer for retrieval
    docs = vectorstore.similarity_search(hypothetical_doc, k=5)
    
    # Generate final answer with retrieved docs
    answer = llm.predict(
        f"Context: {docs}\nQuestion: {query}\nAnswer:"
    )
    
    return answer
```

### 5. Multi-Modal RAG
```python
from langchain.document_loaders import ImageLoader

# Load images and text
text_docs = load_text_documents()
image_docs = load_images()

# Create multi-modal embeddings
text_embeddings = text_embedding_model.embed(text_docs)
image_embeddings = image_embedding_model.embed(image_docs)

# Store in vector database
vectorstore.add(text_embeddings + image_embeddings)

# Query with text or image
query = "Show me product images similar to this description"
results = vectorstore.similarity_search(query)
```

### Interview Question 9: What are advanced RAG architectures?

**Answer:**

**Advanced RAG Architectures:**

**1. Agentic RAG:**
- **What**: Agent decides when/what to retrieve
- **Benefit**: More intelligent retrieval
- **Use Case**: Complex multi-step queries

**2. Self-RAG:**
- **What**: Model self-reflects on retrieval need
- **Benefit**: Reduces unnecessary retrieval
- **Use Case**: Mixed factual/creative queries

**3. Corrective RAG (CRAG):**
- **What**: Evaluates and corrects retrieval
- **Benefit**: Handles poor retrieval gracefully
- **Use Case**: Unreliable knowledge bases

**4. HyDE:**
- **What**: Generates hypothetical answer first
- **Benefit**: Better semantic matching
- **Use Case**: Complex technical queries

**5. Multi-Modal RAG:**
- **What**: Retrieves text, images, audio
- **Benefit**: Richer context
- **Use Case**: Product search, visual Q&A

**Comparison:**

| Architecture | Complexity | Performance | Use Case |
|--------------|------------|-------------|----------|
| **Basic RAG** | Low | Good | Simple Q&A |
| **Agentic RAG** | High | Excellent | Complex tasks |
| **Self-RAG** | Medium | Very Good | Mixed queries |
| **CRAG** | Medium | Very Good | Noisy data |
| **HyDE** | Low | Very Good | Technical docs |

---

## Production Considerations

### Scalability
```python
# 1. Batch Processing
def batch_embed_documents(documents, batch_size=100):
    embeddings = []
    for i in range(0, len(documents), batch_size):
        batch = documents[i:i+batch_size]
        batch_embeddings = embedding_model.embed(batch)
        embeddings.extend(batch_embeddings)
    return embeddings

# 2. Async Processing
import asyncio

async def async_retrieve(query):
    docs = await vectorstore.asimilarity_search(query)
    return docs

# 3. Caching
from functools import lru_cache

@lru_cache(maxsize=1000)
def cached_embed(text):
    return embedding_model.embed(text)
```

### Monitoring
```python
import time
from prometheus_client import Counter, Histogram

# Metrics
query_counter = Counter('rag_queries_total', 'Total queries')
latency_histogram = Histogram('rag_latency_seconds', 'Query latency')

def monitored_rag_query(query):
    query_counter.inc()
    
    start_time = time.time()
    
    try:
        # Retrieve
        docs = retriever.get_relevant_documents(query)
        
        # Generate
        answer = llm.generate(docs, query)
        
        # Log success
        latency = time.time() - start_time
        latency_histogram.observe(latency)
        
        return answer
    except Exception as e:
        # Log error
        error_counter.inc()
        raise
```

### Cost Optimization
```python
# 1. Use smaller embedding models
embeddings = HuggingFaceEmbeddings(
    model_name="all-MiniLM-L6-v2"  # Free, smaller
)

# 2. Cache LLM responses
from langchain.cache import InMemoryCache
import langchain
langchain.llm_cache = InMemoryCache()

# 3. Use cheaper LLMs for simple queries
def smart_llm_selection(query_complexity):
    if query_complexity == "simple":
        return ChatOpenAI(model="gpt-3.5-turbo")
    else:
        return ChatOpenAI(model="gpt-4")
```

### Security
```python
# 1. Input validation
def validate_query(query):
    if len(query) > 1000:
        raise ValueError("Query too long")
    if contains_injection(query):
        raise ValueError("Potential injection detected")
    return query

# 2. Access control
def check_permissions(user, document):
    if document.metadata["access_level"] > user.clearance:
        return False
    return True

# 3. PII filtering
def filter_pii(text):
    # Remove emails, phone numbers, SSN, etc.
    filtered = remove_emails(text)
    filtered = remove_phone_numbers(filtered)
    return filtered
```

### Interview Question 10: What are key considerations for production RAG systems?

**Answer:**

**Production RAG Considerations:**

**1. Performance:**
- **Latency**: Target <2s end-to-end
- **Throughput**: Handle concurrent requests
- **Caching**: Cache embeddings and responses

**2. Scalability:**
- **Horizontal scaling**: Multiple instances
- **Vector DB**: Choose scalable solution (Pinecone, Weaviate)
- **Batch processing**: Process documents in batches

**3. Cost:**
- **Embedding costs**: Use efficient models
- **LLM costs**: Cache responses, use cheaper models
- **Storage costs**: Optimize vector storage

**4. Monitoring:**
- **Metrics**: Latency, error rate, cost
- **Logging**: Query logs, error logs
- **Alerts**: Set up alerts for failures

**5. Security:**
- **Input validation**: Prevent injection attacks
- **Access control**: Document-level permissions
- **PII protection**: Filter sensitive data

**6. Reliability:**
- **Error handling**: Graceful degradation
- **Fallbacks**: Backup retrieval strategies
- **Testing**: Comprehensive test suite

**7. Maintenance:**
- **Document updates**: Incremental indexing
- **Model updates**: Version management
- **Monitoring**: Track performance over time

---

## Common Interview Questions

### Question 11: Explain the difference between RAG and fine-tuning.

**Answer:**

| Aspect | RAG | Fine-Tuning |
|--------|-----|-------------|
| **Purpose** | Add external knowledge | Change model behavior |
| **Cost** | Lower (no training) | Higher (requires training) |
| **Update** | Easy (add documents) | Hard (retrain model) |
| **Latency** | Slightly higher | Lower |
| **Transparency** | High (cite sources) | Low (black box) |
| **Use Case** | Factual Q&A | Style, format, behavior |

**When to use RAG:**
- Need current information
- Frequently changing knowledge
- Want source attribution
- Limited budget

**When to use Fine-tuning:**
- Need specific behavior/style
- Static knowledge
- Latency critical
- Have training data

**Best: Combine both!**
- Fine-tune for behavior
- RAG for knowledge

### Question 12: How do you handle multi-hop questions in RAG?

**Answer:**

**Multi-hop questions** require information from multiple sources.

**Example:**
"Who is the CEO of the company that makes iPhone?"
- Hop 1: Apple makes iPhone
- Hop 2: Tim Cook is CEO of Apple

**Strategies:**

**1. Iterative Retrieval:**
```python
def multi_hop_rag(query):
    # First retrieval
    docs1 = retriever.get_relevant_documents(query)
    
    # Generate intermediate answer
    intermediate = llm.generate(docs1, query)
    
    # Second retrieval based on intermediate
    docs2 = retriever.get_relevant_documents(intermediate)
    
    # Final answer
    return llm.generate(docs1 + docs2, query)
```

**2. Decomposition:**
```python
def decompose_query(query):
    sub_queries = llm.predict(
        f"Break this into sub-questions: {query}"
    )
    
    answers = []
    for sub_q in sub_queries:
        docs = retriever.get_relevant_documents(sub_q)
        answer = llm.generate(docs, sub_q)
        answers.append(answer)
    
    # Combine answers
    return llm.generate(answers, query)
```

### Question 13: How do you handle long documents in RAG?

**Answer:**

**Strategies for Long Documents:**

**1. Hierarchical Chunking:**
```python
# Create parent and child chunks
parent_splitter = RecursiveCharacterTextSplitter(chunk_size=2000)
child_splitter = RecursiveCharacterTextSplitter(chunk_size=500)

# Search children, return parents
```

**2. Summarization:**
```python
# Summarize long documents before embedding
summary = llm.summarize(long_document)
embed_and_store(summary)
```

**3. Map-Reduce:**
```python
# Process each chunk separately, then combine
chunk_answers = [llm.generate(chunk, query) for chunk in chunks]
final_answer = llm.combine(chunk_answers)
```

**4. Sliding Window:**
```python
# Overlapping chunks to maintain context
chunks = create_chunks(document, size=1000, overlap=200)
```

### Question 14: How do you ensure RAG answers are factually correct?

**Answer:**

**Ensuring Factual Correctness:**

**1. Source Quality:**
- Curate high-quality documents
- Verify document accuracy
- Regular updates

**2. Retrieval Quality:**
- Use good embedding models
- Implement re-ranking
- Set relevance thresholds

**3. Prompt Engineering:**
```python
template = """
Answer ONLY based on the context.
If unsure, say "I don't have enough information."
Cite sources for each claim.

Context: {context}
Question: {question}
"""
```

**4. Post-Generation Validation:**
```python
def validate_answer(answer, context):
    # Check if answer is supported by context
    is_supported = llm.predict(
        f"Is this answer supported by context?\nAnswer: {answer}\nContext: {context}"
    )
    return is_supported
```

**5. Human-in-the-Loop:**
- Review flagged answers
- Collect feedback
- Continuous improvement

### Question 15: What are common RAG failure modes and how to fix them?

**Answer:**

**Common RAG Failures:**

**1. Poor Retrieval:**
- **Problem**: Relevant docs not retrieved
- **Fix**: Better embeddings, query expansion, hybrid search

**2. Irrelevant Context:**
- **Problem**: Retrieved docs not relevant
- **Fix**: Re-ranking, metadata filtering, better chunking

**3. Lost in the Middle:**
- **Problem**: LLM ignores middle documents
- **Fix**: Reorder docs, use compression, reduce context

**4. Hallucination:**
- **Problem**: LLM generates unsupported info
- **Fix**: Strict prompts, validation, source citation

**5. Outdated Information:**
- **Problem**: Documents are old
- **Fix**: Regular updates, timestamp filtering

**6. Contradictory Information:**
- **Problem**: Multiple conflicting sources
- **Fix**: Source ranking, recency weighting, explicit handling

---

## Quick Tips for RAG Interviews

1. **Understand the pipeline**: Indexing → Retrieval → Generation
2. **Know chunk size trade-offs**: Precision vs context
3. **Compare vector databases**: When to use each
4. **Master retrieval strategies**: Hybrid search, re-ranking
5. **Prompt engineering**: Clear instructions, source citation
6. **Evaluation metrics**: Precision, recall, faithfulness
7. **Advanced techniques**: Agentic RAG, Self-RAG, CRAG
8. **Production concerns**: Scalability, cost, monitoring
9. **Common failures**: How to debug and fix
10. **Hands-on experience**: Build a RAG system!

---

## Conclusion

RAG is a powerful technique for building accurate, grounded AI applications. Understanding the pipeline, optimization strategies, and production considerations is crucial for interviews and real-world applications.

**Key Takeaways:**
- RAG combines retrieval with generation
- Chunk size and overlap are critical
- Vector databases enable semantic search
- Advanced retrieval improves accuracy
- Evaluation is essential for optimization
- Production requires monitoring and scaling

Good luck with your RAG interview! 🚀
