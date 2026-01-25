# RAG Systems - Core Concepts

## 1. RAG Use Cases

**Definition**: Retrieval-Augmented Generation (RAG) enhances LLM responses by retrieving relevant context from external knowledge bases before generating answers.

**Common Use Cases:**

### **1. Enterprise Q&A / Document Search**
- **Problem**: Answer questions from internal documents (policies, manuals, reports)
- **Example**: "What is our vacation policy?" → Retrieve from HR docs → Generate answer
- **Value**: Employees get instant answers without searching docs manually

### **2. Customer Support Chatbots**
- **Problem**: Answer customer questions using product docs, FAQs, tickets
- **Example**: "How do I reset my password?" → Retrieve from support KB → Respond
- **Value**: 24/7 support, reduce ticket volume by 40-60%

### **3. Code Documentation Assistant**
- **Problem**: Help developers find API usage, code examples, troubleshooting
- **Example**: "How to authenticate with our API?" → Retrieve from docs → Show code example
- **Value**: Faster onboarding, reduce support requests

### **4. Legal / Compliance Research**
- **Problem**: Search case law, regulations, contracts
- **Example**: "Find precedents for data privacy violations" → Retrieve cases → Summarize
- **Value**: Faster legal research, comprehensive coverage

### **5. Medical / Scientific Research**
- **Problem**: Query research papers, clinical trials, medical guidelines
- **Example**: "Latest treatments for diabetes" → Retrieve papers → Summarize findings
- **Value**: Evidence-based answers, citation tracking

### **6. E-commerce Product Recommendations**
- **Problem**: Help customers find products based on natural language queries
- **Example**: "I need a laptop for gaming under $1500" → Retrieve products → Recommend with specs
- **Value**: Better search experience, higher conversion

### **7. Content Generation with Citations**
- **Problem**: Write blog posts, reports grounded in company/industry knowledge
- **Example**: "Write quarterly report on AI trends" → Retrieve industry reports → Generate with citations
- **Value**: Factual, verifiable content

**Interview Insight**: RAG is the #1 GenAI application in production because it's cheaper than fine-tuning, easier to update, and provides source attribution.

---

## 2. RAG vs Fine-Tuning

**Question**: "When should I use RAG vs fine-tuning?"

| Aspect | RAG | Fine-Tuning |
|--------|-----|-------------|
| **Purpose** | Add external knowledge | Change model behavior/style |
| **Cost** | Low ($10-$100 setup) | High ($1K-$10K per run) |
| **Speed** | Instant (add docs to DB) | Days/weeks (training time) |
| **Knowledge Updates** | Real-time (add new docs) | Requires retraining |
| **Use Case** | Q&A, search, grounding | Tone, format, domain expertise |
| **Accuracy on Facts** | High (retrieves exact docs) | Medium (memorizes training data) |
| **Hallucination Risk** | Lower (grounded in docs) | Higher (no source) |
| **Complexity** | Medium (vector DB, embeddings) | High (GPU, training data) |
| **When Knowledge Changes** | Easy (update docs) | Expensive (retrain) |

**Decision Framework:**

**Use RAG when:**
- ✅ Need to answer questions from documents
- ✅ Knowledge changes frequently (news, docs, data)
- ✅ Need source attribution / citations
- ✅ Want low cost and fast iteration
- ✅ Have unstructured knowledge (PDFs, docs, web pages)

**Use Fine-Tuning when:**
- ✅ Need consistent output format (JSON, code style)
- ✅ Teaching domain-specific language (legal, medical)
- ✅ Changing personality / tone
- ✅ Improving task-specific performance (classification, extraction)
- ✅ Reducing prompt length (behavior baked into model)

**Best Approach: Combine Both**
```
Base LLM → Fine-tune for tone/format → RAG for knowledge → Production
```
- Example: Fine-tune for medical terminology + RAG for latest research papers

**Interview Tip**: RAG is cheaper and faster for knowledge. Fine-tuning is for behavior. Most production systems use RAG.

---

## 3. Chunking

**Definition**: Breaking large documents into smaller segments (chunks) for embedding and retrieval.

**Why Chunking is Necessary:**
- LLM context limits (can't fit entire 1000-page manual)
- Embedding models have token limits (512-8192 tokens)
- Smaller chunks → more precise retrieval
- Better semantic search (focused content)

**Chunking Strategies:**

### **1. Fixed-Size Chunking**
- **Method**: Split every N tokens/characters
- **Parameters**: `chunk_size=512`, `overlap=50`
- **Pros**: Simple, predictable, fast
- **Cons**: May split sentences mid-way, lose context

```python
chunks = split_text(document, chunk_size=512, overlap=50)
# Overlap prevents losing context at boundaries
```

### **2. Semantic Chunking**
- **Method**: Split at natural boundaries (paragraphs, sections, sentences)
- **Pros**: Preserves meaning, coherent chunks
- **Cons**: Variable chunk sizes, slower

```python
# Split by paragraphs
chunks = document.split("\n\n")

# Or split by sentences
chunks = nltk.sent_tokenize(document)
```

### **3. Recursive Chunking (LangChain)**
- **Method**: Try splitting by delimiters in order: `\n\n` → `\n` → `.` → ` `
- **Pros**: Balances semantic meaning and size
- **Cons**: More complex

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", ".", " "]
)
chunks = splitter.split_text(document)
```

### **4. Document-Aware Chunking**
- **Markdown**: Split by headers (`#`, `##`, `###`)
- **Code**: Split by functions/classes
- **PDFs**: Respect page boundaries
- **Tables**: Keep tables intact

**Best Practices:**
- **Chunk size**: 512-1024 tokens (sweet spot for most use cases)
- **Overlap**: 10-20% (prevents losing context at boundaries)
- **Test**: Evaluate retrieval quality with your specific queries

**Metadata to Include:**
```python
chunk = {
    "text": "...",
    "metadata": {
        "source": "manual.pdf",
        "page": 42,
        "section": "Chapter 5: Installation",
        "chunk_id": "doc_42_chunk_3"
    }
}
```

**Interview Tip**: Chunking is critical for RAG quality. Too small = lose context. Too large = irrelevant content. Experiment and measure.

---

## 4. Embedding

**Definition**: Converting text into dense vector representations (arrays of numbers) that capture semantic meaning.

**How It Works:**
```
Text: "The cat sat on the mat"
↓ Embedding Model
Vector: [0.23, -0.15, 0.67, ..., 0.42]  # 1536 dimensions
```

**Key Properties:**
- **Semantic similarity**: Similar text → similar vectors
- **Cosine similarity**: Measure similarity between vectors (-1 to 1)
- **Dense representation**: Captures meaning in high-dimensional space

**Popular Embedding Models (2026):**

| Model | Provider | Dimensions | Max Tokens | Cost | Quality |
|-------|----------|-----------|-----------|------|---------|
| text-embedding-3-small | OpenAI | 1536 | 8191 | $0.02/1M | Good |
| text-embedding-3-large | OpenAI | 3072 | 8191 | $0.13/1M | Best |
| all-MiniLM-L6-v2 | Open-source | 384 | 512 | Free | Medium |
| BGE-large-en | BAAI | 1024 | 512 | Free | Good |
| Voyage-2 | Voyage AI | 1024 | 16000 | $0.10/1M | Excellent |

**Embedding Process:**

```python
import openai

# Embed documents (offline/indexing)
docs = ["Document 1 text", "Document 2 text", ...]
response = openai.embeddings.create(
    model="text-embedding-3-small",
    input=docs
)
embeddings = [item.embedding for item in response.data]

# Store in vector DB
vector_db.upsert(embeddings, metadata=doc_metadata)

# Embed query (at runtime)
query = "What is the refund policy?"
query_embedding = openai.embeddings.create(
    model="text-embedding-3-small",
    input=query
).data[0].embedding

# Search
results = vector_db.search(query_embedding, k=5)
```

**Best Practices:**
1. **Same model**: Use same embedding model for docs and queries
2. **Batch processing**: Embed 100-1000 docs at once (faster, cheaper)
3. **Caching**: Don't re-embed unchanged documents
4. **Normalization**: Normalize vectors for cosine similarity

**Cost Optimization:**
- 1M tokens ≈ 750K words
- text-embedding-3-small: $0.02/1M tokens = $0.000002 per doc (500 words)
- For 100K documents: ~$20 one-time cost

**Interview Insight**: Embeddings capture semantic meaning. "Laptop repair" and "Fix broken computer" have high similarity even with no word overlap.

---

## 5. Vector Database

**Definition**: Specialized databases for storing and querying high-dimensional vectors (embeddings) using similarity search.

**Why Not Regular Databases?**
- **Traditional DB**: O(n) linear scan for similarity (slow for millions of vectors)
- **Vector DB**: O(log n) with indexing (HNSW, IVF) → 1000x faster

**Popular Vector Databases:**

| Database | Type | Hosting | Features | Cost | Best For |
|----------|------|---------|----------|------|----------|
| **Pinecone** | Managed | Cloud | Serverless, easy setup | $0.096/GB/mo | Production, quick start |
| **Weaviate** | Open-source | Self-hosted | Hybrid search, GraphQL | Free | Full control |
| **Qdrant** | Open-source | Self/Cloud | Fast, Rust-based | Free/Paid | Performance |
| **Chroma** | Open-source | In-memory | Lightweight, simple | Free | Prototyping |
| **Milvus** | Open-source | Self/Cloud | Scalable, GPU support | Free/Paid | Large scale |
| **pgvector** | Extension | Postgres | SQL, familiar | Free | Small-scale, existing PG |

**Key Features:**

### **1. Indexing Algorithms**
- **HNSW**: High accuracy, slower inserts (best for most use cases)
- **IVF**: Faster inserts, lower accuracy (good for large scale)
- **FAISS**: Facebook's library, optimized for CPU/GPU

### **2. Similarity Metrics**
- **Cosine**: Measures angle (0-1), most common
- **Euclidean**: L2 distance
- **Dot product**: For non-normalized vectors

### **3. Hybrid Search**
- Combine vector similarity + keyword matching (BM25)
- Example: Find docs about "AI" (keyword) that are semantically similar to query

### **4. Metadata Filtering**
- Filter before vector search for efficiency
```python
results = vector_db.search(
    query_embedding,
    k=10,
    filter={"date": {"$gte": "2024-01-01"}, "category": "finance"}
)
```

### **5. Multi-Tenancy**
- Isolate data per user/customer
- Namespace or collection-based separation

**Performance Metrics:**
- **Latency**: 10-50ms for top-10 search (1M vectors)
- **Throughput**: 1000+ queries/sec
- **Scale**: 100M-1B+ vectors

**Code Example (Pinecone):**
```python
import pinecone

# Initialize
pinecone.init(api_key="xxx")
index = pinecone.Index("my-rag-index")

# Upsert vectors
index.upsert([
    ("doc1", embedding1, {"text": "...", "page": 1}),
    ("doc2", embedding2, {"text": "...", "page": 2}),
])

# Query
results = index.query(
    vector=query_embedding,
    top_k=5,
    include_metadata=True
)
```

**Interview Tip**: For production RAG, choose Pinecone (managed, easy) or Weaviate (self-hosted, full control). pgvector is good for small-scale (<1M vectors).

---

## 6. Retrieval Process

**Definition**: The step where relevant documents/chunks are fetched from the vector database based on the user query.

**Basic Retrieval Flow:**
```
User Query → Embed Query → Vector Search → Top-K Chunks → Return to LLM
```

**Retrieval Strategies:**

### **1. Basic Vector Search**
```python
# 1. Embed query
query_embedding = embed_model.encode("What is the refund policy?")

# 2. Search vector DB
results = vector_db.search(query_embedding, k=5)

# 3. Extract text
context = "\n\n".join([r.metadata['text'] for r in results])
```

### **2. Hybrid Search (Vector + Keyword)**
- Combine semantic search with exact keyword matching
- Best of both: Semantic understanding + exact term matches

```python
# Vector score + BM25 keyword score
results = vector_db.hybrid_search(
    query="refund policy",
    vector_weight=0.7,
    keyword_weight=0.3,
    k=5
)
```

### **3. Metadata Filtering**
- Filter by date, category, author before vector search
```python
results = vector_db.search(
    query_embedding,
    k=5,
    filter={"category": "billing", "date": {"$gte": "2024-01-01"}}
)
```

### **4. Reranking**
- Initial retrieval: Get top-20 candidates (fast)
- Reranking: Use cross-encoder to reorder top-20 → best 5 (slow but accurate)

```python
# Step 1: Fast retrieval (top-20)
candidates = vector_db.search(query_embedding, k=20)

# Step 2: Rerank with cross-encoder
from sentence_transformers import CrossEncoder
reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-12-v2')

scores = reranker.predict([(query, c.text) for c in candidates])
top_5 = sorted(zip(candidates, scores), key=lambda x: -x[1])[:5]
```

**Reranking improves accuracy by 10-30% but adds 100-200ms latency.**

### **5. Query Expansion**
- Rephrase query multiple ways, search for each, merge results

```python
# Original query
query = "How to reset password?"

# Expansions
expansions = [
    "password reset instructions",
    "change forgotten password",
    "account recovery password"
]

# Search all, merge results
all_results = []
for q in [query] + expansions:
    results = vector_db.search(embed(q), k=5)
    all_results.extend(results)

# Deduplicate and rank
final_results = deduplicate(all_results)[:5]
```

**Retrieval Metrics:**
- **Recall@K**: Of all relevant docs, what % are in top-K?
- **Precision@K**: Of top-K results, what % are relevant?
- **MRR (Mean Reciprocal Rank)**: Average rank of first relevant result

**Interview Tip**: Basic vector search gets 70% quality. Add hybrid search + reranking → 90% quality. Measure with real queries.

---

## 7. Generation

**Definition**: The final step where the LLM generates a response using the retrieved context.

**Generation Flow:**
```
Retrieved Chunks → Assemble Prompt → LLM API Call → Response → Post-processing
```

**Prompt Structure:**

```python
SYSTEM_PROMPT = """
You are a helpful assistant answering questions based on provided context.
Use ONLY the information in the context to answer.
If the answer is not in the context, say "I don't have enough information."
Always cite the source document.
"""

USER_PROMPT = """
Context:
{retrieved_chunks}

Question: {user_query}

Answer based on the context above:
"""
```

**Code Example:**

```python
def generate_rag_response(query, retrieved_chunks):
    # Assemble context from retrieved chunks
    context = "\n\n".join([
        f"[Source: {chunk.metadata['source']}, Page: {chunk.metadata['page']}]\n{chunk.text}"
        for chunk in retrieved_chunks
    ])
    
    # Build prompt
    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": USER_PROMPT.format(
            retrieved_chunks=context,
            user_query=query
        )}
    ]
    
    # Call LLM
    response = openai.chat.completions.create(
        model="gpt-4",
        messages=messages,
        temperature=0.1,  # Low for factual responses
        max_tokens=500
    )
    
    return response.choices[0].message.content
```

**Best Practices:**

### **1. Temperature Settings**
- **Factual Q&A**: 0.0-0.3 (deterministic, less creativity)
- **Creative writing**: 0.7-1.0 (more variation)

### **2. Context Truncation**
- LLM context limits: 4K-128K tokens
- If retrieved context too long, truncate or summarize

```python
MAX_CONTEXT_TOKENS = 3000

if token_count(context) > MAX_CONTEXT_TOKENS:
    # Option 1: Truncate
    context = truncate(context, MAX_CONTEXT_TOKENS)
    
    # Option 2: Summarize
    context = llm.summarize(context, max_tokens=MAX_CONTEXT_TOKENS)
```

### **3. Citation / Source Attribution**
- Include source metadata in response
- Helps users verify information

```python
Answer: According to the Employee Handbook (page 42), the vacation policy is...

Sources:
- Employee_Handbook.pdf, Page 42
- HR_Policy_2024.docx, Section 3.2
```

### **4. Fallback Handling**
- If no relevant chunks found, don't hallucinate

```python
if max_similarity_score < 0.7:
    return "I don't have enough information to answer this question."
```

### **5. Answer Verification (Optional)**
- Use LLM to verify answer is grounded in context

```python
verification_prompt = f"""
Context: {context}
Answer: {generated_answer}

Is the answer fully supported by the context? (Yes/No)
If No, explain what's unsupported.
"""
```

**Interview Tip**: Generation is where you control quality. Use low temperature, enforce citations, handle low-confidence gracefully.

---

## 8. Implementing RAG

**Complete RAG Pipeline:**

```python
import openai
import pinecone
from langchain.text_splitter import RecursiveCharacterTextSplitter

class RAGSystem:
    def __init__(self):
        # Initialize components
        self.embed_model = "text-embedding-3-small"
        self.llm_model = "gpt-4"
        
        # Vector DB
        pinecone.init(api_key="xxx")
        self.index = pinecone.Index("rag-index")
    
    def ingest_documents(self, documents):
        """Index documents into vector DB"""
        # Step 1: Chunk documents
        splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200
        )
        chunks = []
        for doc in documents:
            doc_chunks = splitter.split_text(doc['text'])
            for i, chunk in enumerate(doc_chunks):
                chunks.append({
                    'text': chunk,
                    'metadata': {
                        'source': doc['source'],
                        'page': doc.get('page', 0),
                        'chunk_id': f"{doc['source']}_{i}"
                    }
                })
        
        # Step 2: Generate embeddings (batch)
        texts = [c['text'] for c in chunks]
        response = openai.embeddings.create(
            model=self.embed_model,
            input=texts
        )
        embeddings = [item.embedding for item in response.data]
        
        # Step 3: Upsert to vector DB
        vectors = [
            (chunk['metadata']['chunk_id'], emb, chunk['metadata'])
            for chunk, emb in zip(chunks, embeddings)
        ]
        self.index.upsert(vectors)
        
        return f"Indexed {len(chunks)} chunks"
    
    def retrieve(self, query, k=5):
        """Retrieve relevant chunks"""
        # Embed query
        query_emb = openai.embeddings.create(
            model=self.embed_model,
            input=query
        ).data[0].embedding
        
        # Search vector DB
        results = self.index.query(
            vector=query_emb,
            top_k=k,
            include_metadata=True
        )
        
        return results['matches']
    
    def generate(self, query, retrieved_chunks):
        """Generate answer from retrieved context"""
        # Assemble context
        context = "\n\n".join([
            f"[{c['metadata']['source']}]\n{c['metadata']['text']}"
            for c in retrieved_chunks
        ])
        
        # Build prompt
        messages = [
            {"role": "system", "content": 
             "Answer based on context. Cite sources. Say 'I don't know' if unsure."},
            {"role": "user", "content": f"Context:\n{context}\n\nQuestion: {query}"}
        ]
        
        # Generate
        response = openai.chat.completions.create(
            model=self.llm_model,
            messages=messages,
            temperature=0.1
        )
        
        return response.choices[0].message.content
    
    def query(self, question, k=5):
        """End-to-end RAG query"""
        # 1. Retrieve
        chunks = self.retrieve(question, k=k)
        
        # 2. Generate
        answer = self.generate(question, chunks)
        
        return {
            'answer': answer,
            'sources': [c['metadata']['source'] for c in chunks]
        }

# Usage
rag = RAGSystem()

# Index documents
docs = [
    {'text': 'Document 1 content...', 'source': 'doc1.pdf', 'page': 1},
    {'text': 'Document 2 content...', 'source': 'doc2.pdf', 'page': 1},
]
rag.ingest_documents(docs)

# Query
result = rag.query("What is the refund policy?")
print(result['answer'])
print("Sources:", result['sources'])
```

**Key Steps:**
1. **Ingestion**: Chunk → Embed → Store in vector DB
2. **Retrieval**: Embed query → Search → Get top-K
3. **Generation**: Assemble prompt → LLM → Response

**Interview Tip**: Show you understand the full pipeline: ingestion (offline) and query (runtime) are separate processes.

---

## 9. Ways of Implementing RAG

**Three Approaches:**

### **1. Custom Implementation (DIY)**
- **Components**: OpenAI API + Pinecone/Weaviate + Python
- **Pros**: Full control, minimal dependencies, understand every step
- **Cons**: More code to maintain, reinvent patterns

```python
# Manual implementation (shown in section 8)
embedding = openai.embeddings.create(...)
results = pinecone.query(...)
response = openai.chat.completions.create(...)
```

**When to use**: Small projects, learning, custom requirements

### **2. Using SDKs Directly**
- **SDKs**: OpenAI SDK, Pinecone SDK, Weaviate client
- **Pros**: Official, well-documented, stable
- **Cons**: Still need to write orchestration logic

```python
import openai
import pinecone

# Direct SDK calls (more control than frameworks)
```

**When to use**: Production systems, need reliability, avoid framework overhead

### **3. Using RAG Frameworks**
- **Frameworks**: LangChain, LlamaIndex
- **Pros**: Pre-built patterns, abstractions, less boilerplate
- **Cons**: Learning curve, black box behavior, dependencies

**When to use**: Rapid prototyping, standard RAG patterns, complex orchestration

**Comparison:**

| Approach | Setup Time | Control | Flexibility | Maintenance | Best For |
|----------|-----------|---------|-------------|-------------|----------|
| **DIY** | Days | Full | Maximum | High | Learning, custom |
| **SDKs** | Hours | High | High | Medium | Production |
| **Frameworks** | Minutes | Medium | Medium | Low | Prototyping |

**Interview Tip**: Each approach has trade-offs. DIY for learning, SDKs for production control, frameworks for speed.

---

## 10. LangChain for RAG

**LangChain**: Framework for building LLM applications with pre-built chains, loaders, and integrations.

**Key Components:**

### **1. Document Loaders**
- Load from various sources (PDF, web, databases)

```python
from langchain.document_loaders import PyPDFLoader, WebBaseLoader

# Load PDF
loader = PyPDFLoader("document.pdf")
docs = loader.load()

# Load webpage
loader = WebBaseLoader("https://example.com/docs")
docs = loader.load()
```

### **2. Text Splitters**
- Chunk documents intelligently

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", ".", " "]
)
chunks = splitter.split_documents(docs)
```

### **3. Embeddings**
- Wrap embedding models

```python
from langchain.embeddings import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
```

### **4. Vector Stores**
- Integrate with vector databases

```python
from langchain.vectorstores import Pinecone, Chroma

# Pinecone
vectorstore = Pinecone.from_documents(chunks, embeddings, index_name="rag")

# Chroma (local)
vectorstore = Chroma.from_documents(chunks, embeddings, persist_directory="./db")
```

### **5. Retrievers**
- Query vector stores

```python
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})
docs = retriever.get_relevant_documents("What is the refund policy?")
```

### **6. Chains**
- Pre-built RAG patterns

```python
from langchain.chains import RetrievalQA
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)

qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",  # "stuff", "map_reduce", "refine"
    retriever=retriever,
    return_source_documents=True
)

result = qa_chain({"query": "What is the refund policy?"})
print(result['result'])
print(result['source_documents'])
```

**Complete LangChain RAG Example:**

```python
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import RetrievalQA
from langchain.chat_models import ChatOpenAI

# 1. Load documents
loader = PyPDFLoader("employee_handbook.pdf")
documents = loader.load()

# 2. Chunk
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(documents)

# 3. Embed and store
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_documents(chunks, embeddings)

# 4. Create retriever
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# 5. Create QA chain
llm = ChatOpenAI(model="gpt-4", temperature=0)
qa = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever,
    return_source_documents=True
)

# 6. Query
result = qa({"query": "What is the vacation policy?"})
print(result['result'])
```

**Pros:**
- ✅ Quick setup (minutes)
- ✅ Many integrations (100+ loaders, vector stores)
- ✅ Pre-built patterns (QA, summarization, chatbots)

**Cons:**
- ❌ Abstraction hides details
- ❌ Version updates break code
- ❌ Slower than direct SDK calls
- ❌ Debugging can be difficult

---

## 11. LlamaIndex for RAG

**LlamaIndex** (formerly GPT Index): Specialized framework for RAG and data indexing.

**Key Difference from LangChain:**
- **LangChain**: General LLM orchestration (chains, agents, memory)
- **LlamaIndex**: Focused on data loading, indexing, and querying

**Core Components:**

### **1. Data Loaders**
- 100+ connectors (Notion, Google Docs, Slack, databases)

```python
from llama_index import SimpleDirectoryReader

# Load all files in directory
documents = SimpleDirectoryReader("./data").load_data()
```

### **2. Indexes**
- Multiple index types for different use cases

```python
from llama_index import VectorStoreIndex, GPTListIndex

# Vector index (most common)
index = VectorStoreIndex.from_documents(documents)

# List index (for sequential data)
list_index = GPTListIndex.from_documents(documents)
```

### **3. Query Engine**
- Natural language querying

```python
query_engine = index.as_query_engine()
response = query_engine.query("What is the refund policy?")
print(response)
```

### **4. Response Modes**
- **Compact**: Fit as much context as possible
- **Tree Summarize**: Hierarchical summarization for large contexts
- **Refine**: Iteratively refine answer with each chunk

**Complete LlamaIndex RAG Example:**

```python
from llama_index import VectorStoreIndex, SimpleDirectoryReader, ServiceContext
from llama_index.llms import OpenAI
from llama_index.embeddings import OpenAIEmbedding

# 1. Load documents
documents = SimpleDirectoryReader("./docs").load_data()

# 2. Configure service context
llm = OpenAI(model="gpt-4", temperature=0)
embed_model = OpenAIEmbedding(model="text-embedding-3-small")

service_context = ServiceContext.from_defaults(
    llm=llm,
    embed_model=embed_model,
    chunk_size=1000,
    chunk_overlap=200
)

# 3. Build index
index = VectorStoreIndex.from_documents(
    documents,
    service_context=service_context
)

# 4. Query
query_engine = index.as_query_engine(
    similarity_top_k=5,
    response_mode="compact"
)

response = query_engine.query("What is the vacation policy?")
print(response)
print(response.source_nodes)  # See retrieved chunks
```

**Advanced Features:**

### **1. Chat Engine (Multi-turn)**
```python
chat_engine = index.as_chat_engine()
response = chat_engine.chat("What is the vacation policy?")
response = chat_engine.chat("How many days?")  # Remembers context
```

### **2. Custom Prompts**
```python
from llama_index.prompts import PromptTemplate

template = PromptTemplate(
    "Context: {context_str}\n"
    "Question: {query_str}\n"
    "Answer based on context only:"
)

query_engine = index.as_query_engine(text_qa_template=template)
```

### **3. Persist Index**
```python
# Save to disk
index.storage_context.persist(persist_dir="./storage")

# Load later
from llama_index import load_index_from_storage, StorageContext
storage_context = StorageContext.from_defaults(persist_dir="./storage")
index = load_index_from_storage(storage_context)
```

**LangChain vs LlamaIndex:**

| Aspect | LangChain | LlamaIndex |
|--------|-----------|------------|
| **Focus** | General orchestration | Data indexing & querying |
| **Use Case** | Agents, chains, memory | RAG, knowledge bases |
| **Learning Curve** | Steeper | Gentler |
| **Flexibility** | Very flexible | Optimized for RAG |
| **Best For** | Complex workflows | Simple RAG systems |

**Interview Tip**: 
- Use **LlamaIndex** for straightforward RAG (Q&A, document search)
- Use **LangChain** for complex agents, multi-step workflows
- Use **Direct SDKs** for production systems needing full control

---

## 12. RAG Architecture Components

```
User Query → Query Enhancement → Vector Search → Reranking → Context Assembly → LLM → Response
```

### Basic RAG
1. User asks question
2. Convert query to embedding
3. Search vector DB for similar chunks
4. Pass top-K chunks + query to LLM
5. LLM generates answer grounded in context

### Advanced RAG
- Query rewriting/expansion
- Hybrid search (vector + keyword)
- Reranking with cross-encoder
- Metadata filtering
- Multi-hop retrieval
- Answer verification

## 2. Document Processing Pipeline

### Chunking Strategies

**1. Fixed-size chunking**
```python
chunk_size = 512  # tokens
overlap = 50      # prevent context loss
chunks = split_text(document, chunk_size, overlap)
```
- **Pros**: Simple, predictable
- **Cons**: May break sentences, lose context

**2. Semantic chunking**
```python
# Split at paragraph/section boundaries
chunks = split_by_semantics(document)
# Or use LLM to identify logical breaks
```
- **Pros**: Preserves meaning
- **Cons**: Variable chunk sizes

**3. Recursive chunking** (LangChain approach)
```python
# Try to split by: \n\n, then \n, then sentences, then characters
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separators=["\n\n", "\n", ".", " "]
)
```

**4. Document-aware chunking**
- Markdown: Split by headers (#, ##, ###)
- Code: Split by functions/classes
- PDFs: Respect page boundaries
- Tables: Keep table intact

**Production recommendation**: 
- Chunk size: 512-1024 tokens (balance between context and precision)
- Overlap: 10-20% (prevent split concepts)
- Test with your query types

### Metadata Enrichment
```python
chunk = {
    "text": "...",
    "embedding": [...],
    "metadata": {
        "source": "doc.pdf",
        "page": 5,
        "section": "Introduction",
        "author": "John Doe",
        "date": "2024-01-15",
        "chunk_index": 10,
        "total_chunks": 50
    }
}
```

**Why metadata matters**: Filtering, attribution, debugging, reranking.

## 3. Embedding Models

### Model Comparison

| Model | Dimensions | Max Tokens | Cost | Use Case |
|-------|-----------|-----------|------|----------|
| text-embedding-3-small | 1536 | 8191 | $0.02/1M | Cost-sensitive |
| text-embedding-3-large | 3072 | 8191 | $0.13/1M | Max quality |
| text-embedding-ada-002 | 1536 | 8191 | $0.10/1M | Legacy |
| all-MiniLM-L6-v2 | 384 | 512 | Free | Self-hosted |
| BGE-large-en | 1024 | 512 | Free | Self-hosted |

**Production considerations:**
- OpenAI embeddings: High quality, low latency, cost scales
- Self-hosted: One-time cost, full control, lower quality
- Sentence Transformers: Good middle ground

### Embedding Best Practices

1. **Consistent model**: Same model for documents and queries
2. **Normalize vectors**: Cosine similarity assumes normalized
3. **Cache embeddings**: Don't re-embed same documents
4. **Batch embedding**: 100-1000 texts per API call (faster, cheaper)

```python
# Good: Batch embedding
texts = ["text1", "text2", ..., "text1000"]
embeddings = openai.embeddings.create(model="text-embedding-3-large", input=texts)

# Bad: One at a time (100x slower)
for text in texts:
    embedding = openai.embeddings.create(model="text-embedding-3-large", input=text)
```

## 4. Retrieval Strategies

### Vector Search
```python
query_embedding = embed(query)
results = vector_db.search(
    query_embedding,
    k=10,  # top-10 results
    metric="cosine"
)
```

**Limitations**: 
- Misses exact keyword matches
- Struggles with rare terms, acronyms
- No understanding of negation ("not about X")

### Keyword Search (BM25)
```python
results = bm25_search(query, k=10)
```

**Benefits**: 
- Exact matches (names, IDs, technical terms)
- No embedding needed
- Fast for specific terms

**Limitations**: 
- No semantic understanding
- Synonym problem

### Hybrid Search
```python
# Combine both approaches
vector_results = vector_search(query, k=20)
keyword_results = bm25_search(query, k=20)

# Merge with weights
final_results = merge(
    vector_results, weight=0.7,
    keyword_results, weight=0.3,
    k=10
)
```

**Production standard**: Hybrid search catches both semantic and keyword matches.

### Metadata Filtering
```python
# Pre-filter by metadata before semantic search
results = vector_db.search(
    query_embedding,
    k=10,
    filter={
        "date": {"$gte": "2024-01-01"},
        "department": "engineering",
        "access_level": {"$in": [user.level, "public"]}
    }
)
```

**Use cases**: Multi-tenancy, temporal queries, access control.

## 5. Reranking

### Why Rerank?
- Vector search returns ~similar~, not ~relevant~
- First-stage retrieval optimized for recall
- Reranking optimizes for precision

### Cross-Encoder Reranking
```python
# 1. Get candidates (fast, approximate)
candidates = vector_search(query, k=50)

# 2. Rerank with cross-encoder (slow, accurate)
reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')
pairs = [(query, doc.text) for doc in candidates]
scores = reranker.predict(pairs)

# 3. Return top-K after reranking
top_k = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)[:10]
```

**Cost-quality tradeoff**: 
- Retrieve 50, rerank to 10: +100ms, +20% relevance
- Retrieve 100, rerank to 10: +200ms, +25% relevance

### LLM-based Reranking
```python
rerank_prompt = f"""
Query: {query}

Rank these passages by relevance (1-5):
{passages}

Output JSON: [{{"passage_id": 1, "relevance": 4}}, ...]
"""
```

**When to use**: Complex relevance criteria, domain-specific ranking.

## 6. Query Enhancement

### Query Expansion
```python
# Add synonyms, related terms
original = "GPU out of memory"
expanded = llm.expand(original)
# Result: "GPU out of memory OR CUDA memory error OR OOM GPU"

# Search with expanded query
results = search(expanded)
```

### Query Rewriting
```python
# User: "What did the author say about that?"
# System: Rewrite using conversation history

rewritten = llm.rewrite(
    query=user_query,
    history=conversation_history
)
# Result: "What did John Smith say about AI safety in his paper?"
```

### HyDE (Hypothetical Document Embeddings)
```python
# Instead of embedding query, embed hypothetical answer
hypothetical_answer = llm.generate_hypothetical_answer(query)
query_embedding = embed(hypothetical_answer)
results = vector_search(query_embedding, k=10)
```

**Why it works**: Hypothetical answer is semantically closer to actual documents than short query.

## 7. Multi-Hop Retrieval

### Single-Hop (Standard RAG)
```
Query → Retrieve → Answer
```

### Multi-Hop (Iterative RAG)
```
Query → Retrieve → Generate sub-query → Retrieve → Answer
```

**Example:**
```
User: "Compare the CEO's stance on AI in Q1 vs Q4 earnings calls"

Step 1: Retrieve Q1 transcript → Extract CEO's AI comments
Step 2: Retrieve Q4 transcript → Extract CEO's AI comments  
Step 3: Compare both → Generate answer
```

**Implementation:**
```python
def multi_hop_rag(query, max_hops=3):
    context = []
    current_query = query
    
    for hop in range(max_hops):
        # Retrieve
        docs = retrieve(current_query, k=5)
        context.extend(docs)
        
        # Check if we have enough info
        if has_sufficient_info(query, context):
            break
        
        # Generate follow-up query
        current_query = llm.generate_follow_up(query, context)
    
    # Final answer
    return llm.answer(query, context)
```

## 8. Context Assembly

### Basic Concatenation
```python
context = "\n\n".join([doc.text for doc in top_docs])
prompt = f"Context:\n{context}\n\nQuestion: {query}\nAnswer:"
```

### Structured Context
```python
context_parts = []
for i, doc in enumerate(top_docs, 1):
    context_parts.append(f"""
[Document {i}]
Source: {doc.metadata.source}
Page: {doc.metadata.page}
Content: {doc.text}
""")

context = "\n".join(context_parts)
```

**Benefits**: Model can cite sources, easier to verify.

### Context Compression
```python
# Problem: 10 chunks = 5000 tokens (expensive)

# Solution 1: Summarize each chunk
compressed = [llm.summarize(doc, max_length=200) for doc in top_docs]

# Solution 2: Use compression model (LLMLingua)
from llmlingua import PromptCompressor

compressor = PromptCompressor()
compressed_context = compressor.compress_prompt(context, rate=0.5)
# Compresses to 50% of original length
```

**Tradeoff**: 30-50% token savings, 5-15% quality loss.

## 9. Answer Verification

### Citation Enforcement
```python
system_prompt = """
Answer based ONLY on provided context.
Cite sources using [1], [2] format.
If answer not in context, say "Information not available."
"""
```

### Factual Consistency Check
```python
# Use NLI model to verify answer is supported by context
from transformers import pipeline

nli = pipeline("text-classification", model="microsoft/deberta-large-mnli")

def verify_answer(context, answer):
    result = nli(f"{context} [SEP] {answer}")
    # Returns: entailment (good), neutral, contradiction (bad)
    return result[0]["label"] == "ENTAILMENT"
```

### Self-RAG (Self-Reflective RAG)
```python
# Model critiques its own answer
reflection_prompt = f"""
Question: {query}
Context: {context}
Answer: {answer}

Is this answer:
1. Fully supported by context? (yes/no)
2. Directly addresses the question? (yes/no)
3. Contains any speculation? (yes/no)

If any issues, provide corrected answer.
"""
```

## 10. Common RAG Failure Modes

### Problem 1: Chunking splits critical context
```
Document: "Q3 revenue was $100M. This is a 50% increase from Q2."
Bad chunking: Chunk 1 = "$100M." | Chunk 2 = "50% increase from Q2."
→ Model can't connect the numbers
```

**Solution**: Overlap chunks, semantic chunking, metadata linking.

### Problem 2: Retrieval returns irrelevant results
```
Query: "How do I reset password?"
Retrieved: "Password must be 8 characters..." (high embedding similarity, but irrelevant)
```

**Solution**: Reranking, hybrid search, query clarification.

### Problem 3: Answer in multiple chunks
```
Query: "List all supported features"
Reality: Features spread across 20 chunks
Retrieved: Only 10 chunks (features 1-10 visible, 11-20 missing)
```

**Solution**: Increase K, multi-hop retrieval, hierarchical summarization.

### Problem 4: Hallucination despite RAG
```
Context: "Product A costs $100"
Model: "Product A costs $100 and includes free shipping" (not in context)
```

**Solution**: Strict prompting, citation enforcement, verification layer.

### Problem 5: Cold start (no documents indexed)
```
User: "Tell me about our Q4 plans"
System: No documents found
```

**Solution**: Fallback to general knowledge, suggest document upload, default responses.

## 11. RAG Evaluation Metrics

### Retrieval Metrics
- **Recall@K**: % of relevant docs in top-K results
- **Precision@K**: % of top-K results that are relevant
- **MRR** (Mean Reciprocal Rank): Average of 1/rank of first relevant result
- **NDCG**: Normalized Discounted Cumulative Gain (considers order)

### End-to-End Metrics
- **Answer relevance**: Does answer address the question?
- **Faithfulness**: Is answer grounded in context?
- **Context relevance**: Are retrieved docs relevant?
- **Answer correctness**: Compare to ground truth (if available)

**Tools**: RAGAS, TruLens, LangSmith

### Production Monitoring
```python
# Track these metrics per query
metrics = {
    "retrieval_latency_ms": 45,
    "num_docs_retrieved": 10,
    "num_docs_used": 5,
    "llm_latency_ms": 850,
    "total_tokens": 1200,
    "cost_usd": 0.015,
    "user_rating": 4.5  # if available
}
```

## 12. Advanced RAG Patterns

### Agentic RAG
- Agent decides when to retrieve
- Can retrieve multiple times
- Can use different search strategies

### Corrective RAG (CRAG)
```python
def crag(query):
    docs = retrieve(query, k=5)
    
    # Check relevance
    relevance_scores = [score_relevance(query, doc) for doc in docs]
    
    if max(relevance_scores) < threshold:
        # Fallback to web search
        docs = web_search(query)
    
    return answer(query, docs)
```

### RAG-Fusion
```python
# Generate multiple query variations
queries = [
    query,
    llm.rewrite(query, style="technical"),
    llm.rewrite(query, style="simple")
]

# Retrieve for each
all_docs = []
for q in queries:
    all_docs.extend(retrieve(q, k=10))

# Deduplicate and rerank
unique_docs = deduplicate(all_docs)
final_docs = rerank(query, unique_docs, k=10)
```

**Result**: More robust retrieval, catches different phrasings.

## 13. Cost Optimization

**Typical RAG costs per query:**
- Embedding query: $0.000001 (negligible)
- Vector search: $0.0001 (depends on DB)
- LLM inference: $0.01-0.05 (dominant cost)
- Reranking: $0.001 (if using API)

**Optimization strategies:**
1. **Cache frequent queries**: 40-60% hit rate typical
2. **Smaller context**: Retrieve 5 instead of 10 chunks
3. **Compress context**: LLMLingua saves 30-50% tokens
4. **Cheaper LLM**: Use GPT-3.5 instead of GPT-4 (10x cheaper)
5. **Batch similar queries**: Amortize retrieval cost
6. **Self-hosted embedding**: One-time cost vs per-query API cost

**Break-even analysis**: Self-hosting embedding models is cheaper beyond ~1M queries/month.
