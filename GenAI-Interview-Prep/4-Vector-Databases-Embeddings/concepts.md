# Vector Databases & Embeddings - Core Concepts

## 1. Embedding Fundamentals

**What are embeddings?**
Dense vector representations of text that capture semantic meaning.

```python
text = "The cat sat on the mat"
embedding = embed(text)
# Result: [0.23, -0.45, 0.67, ..., 0.12]  # 1536 dimensions for OpenAI
```

**Key properties:**
- **Semantic similarity**: Similar meanings → similar vectors
- **Dense**: Every dimension has a value (vs sparse like TF-IDF)
- **Fixed size**: "cat" and "encyclopedia" → same dimension count
- **Domain-aware**: Models capture context

## 2. Embedding Model Comparison

| Model | Vendor | Dimensions | Max Tokens | Cost/1M | MTEB Score | Best For |
|-------|--------|-----------|-----------|---------|-----------|----------|
| text-embedding-3-large | OpenAI | 3072 | 8191 | $0.13 | 64.6 | Best quality |
| text-embedding-3-small | OpenAI | 1536 | 8191 | $0.02 | 62.3 | Cost-effective |
| text-embedding-ada-002 | OpenAI | 1536 | 8191 | $0.10 | 60.9 | Legacy |
| voyage-large-2 | Voyage | 1536 | 16000 | $0.12 | 68.8 | Max quality |
| cohere-embed-v3 | Cohere | 1024 | 512 | $0.10 | 64.5 | Multilingual |
| all-MiniLM-L6-v2 | OSS | 384 | 512 | Free | 56.3 | Self-hosted |
| BGE-large-en | OSS | 1024 | 512 | Free | 63.5 | Self-hosted |
| e5-mistral-7b | OSS | 4096 | 32768 | Free | 66.6 | Long context |

**MTEB**: Massive Text Embedding Benchmark (higher = better)

**Production choices:**
- **High-scale, cost-sensitive**: text-embedding-3-small
- **Best quality**: voyage-large-2 or text-embedding-3-large
- **Self-hosted**: BGE-large-en or e5-mistral-7b
- **Multilingual**: cohere-embed-v3 or multilingual-e5

## 3. Similarity Metrics

### Cosine Similarity (Most Common)
```python
def cosine_similarity(v1, v2):
    return np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))

# Range: -1 to 1 (higher = more similar)
# Typical thresholds:
# > 0.9: Very similar
# 0.7-0.9: Related
# < 0.7: Different
```

**Why cosine?** 
- Normalized (not affected by vector magnitude)
- Fast to compute
- Works well for text embeddings

### Euclidean Distance (L2)
```python
def euclidean_distance(v1, v2):
    return np.linalg.norm(v1 - v2)

# Range: 0 to ∞ (lower = more similar)
```

**When to use:** Image embeddings, geometric data.

### Dot Product
```python
def dot_product(v1, v2):
    return np.dot(v1, v2)

# Faster than cosine (no normalization)
# Requires normalized vectors
```

**Production tip:** Pre-normalize embeddings, use dot product for speed.

## 4. Vector Database Architecture

### Core Components

1. **Storage Layer**: Persist embeddings + metadata
2. **Index Layer**: ANN (Approximate Nearest Neighbor) algorithms
3. **Query Layer**: Handle search requests
4. **Metadata Filter**: Pre/post-filter by attributes

### Indexing Algorithms

**HNSW (Hierarchical Navigable Small World)**
- **How it works**: Multi-layer graph structure
- **Complexity**: O(log N) search
- **Memory**: High (1.2-1.5× data size)
- **Speed**: Very fast
- **Accuracy**: 95-99%
- **Use case**: Production standard

```python
index = vector_db.create_index(
    type="HNSW",
    m=16,  # edges per node (16-64)
    ef_construction=200  # higher = better accuracy, slower build
)
```

**IVF (Inverted File Index)**
- **How it works**: Cluster vectors, search nearest clusters
- **Complexity**: O(√N) with quantization
- **Memory**: Low (0.1-0.3× data size)
- **Speed**: Fast
- **Accuracy**: 80-95%
- **Use case**: Very large datasets (>10M vectors)

**Flat (Brute Force)**
- **How it works**: Compare query to every vector
- **Complexity**: O(N)
- **Speed**: Slow
- **Accuracy**: 100%
- **Use case**: Small datasets (<10K vectors)

## 5. Vector Database Options

### Cloud-Native

**Pinecone**
- Fully managed, serverless
- Auto-scaling, low latency
- Metadata filtering
- Cost: $0.096/GB/month + query costs
- **Best for:** Startups, fast iteration

**Weaviate**
- Open-source, self-hosted or cloud
- Hybrid search built-in
- GraphQL API
- **Best for:** Hybrid search requirements

**Qdrant**
- Open-source, Rust-based (fast)
- Rich filtering, payload support
- Quantization support
- **Best for:** Performance-critical apps

### Integrated with Existing DBs

**pgvector (PostgreSQL)**
```sql
CREATE TABLE embeddings (
    id SERIAL PRIMARY KEY,
    text TEXT,
    embedding vector(1536)
);

CREATE INDEX ON embeddings USING ivfflat (embedding vector_cosine_ops);

SELECT text FROM embeddings 
ORDER BY embedding <-> query_embedding 
LIMIT 10;
```
- **Best for:** Already using PostgreSQL, small-medium scale

**Redis with RediSearch**
- In-memory, very fast
- Hybrid search
- **Best for:** Low-latency requirements, caching

**Elasticsearch with dense_vector**
- Keyword + vector search
- Mature ecosystem
- **Best for:** Existing Elasticsearch users

### Cloud Provider Options

**Azure AI Search** (formerly Cognitive Search)
- Integrated with Azure ecosystem
- Hybrid search, semantic ranking
- **Best for:** Azure-first organizations

**AWS OpenSearch**
- Based on Elasticsearch
- k-NN plugin for vector search
- **Best for:** AWS-first organizations

## 6. Performance Optimization

### Quantization
Reduce precision to save memory and increase speed.

```python
# Original: FP32 (4 bytes per dimension)
embedding_fp32 = [0.123456789, -0.987654321, ...]  # 1536 dims = 6KB

# Quantized: INT8 (1 byte per dimension)
embedding_int8 = [31, -250, ...]  # 1536 dims = 1.5KB

# Savings: 4× memory reduction
# Accuracy loss: <2%
```

**Types:**
- **Scalar quantization**: Scale float to int8
- **Product quantization**: Compress to 32-64 bytes (10-50× compression)
- **Binary quantization**: 1 bit per dimension (32× compression, 5-10% accuracy loss)

### Dimensionality Reduction
```python
# Reduce from 3072 to 1536 dimensions
from sklearn.decomposition import PCA

pca = PCA(n_components=1536)
reduced_embeddings = pca.fit_transform(original_embeddings)

# Benefits: 2× faster search, 2× less storage
# Cost: 2-5% accuracy loss
```

**When to use:** Very large datasets, cost-sensitive applications.

### Sharding
```python
# Distribute vectors across multiple nodes
shard_1 = vectors[0:1_000_000]      # Users A-M
shard_2 = vectors[1_000_000:2_000_000]  # Users N-Z

# Search in parallel, merge results
results = merge([
    shard_1.search(query),
    shard_2.search(query)
])
```

### Caching
```python
from functools import lru_cache

@lru_cache(maxsize=10000)
def get_embedding(text):
    return embedding_model.encode(text)

# Cache embeddings for frequent queries
# Typical hit rate: 40-60%
```

## 7. Metadata Filtering

### Pre-filtering (Recommended)
```python
# Filter BEFORE vector search
results = vector_db.search(
    embedding=query_embedding,
    k=10,
    filter={
        "department": "engineering",
        "date": {"$gte": "2024-01-01"},
        "access_level": {"$lte": user.access_level}
    }
)

# Only search within filtered subset (faster, more relevant)
```

### Post-filtering (Fallback)
```python
# Search first, filter after
candidates = vector_db.search(query_embedding, k=100)
filtered = [c for c in candidates if c.metadata["department"] == "engineering"]
results = filtered[:10]

# Problem: Might not get 10 results if many filtered out
```

**Production pattern:** Pre-filter when possible, post-filter for complex conditions.

## 8. Hybrid Search Implementation

```python
class HybridSearch:
    def __init__(self):
        self.vector_db = VectorDB()
        self.keyword_db = ElasticsearchDB()
    
    def search(self, query, k=10, vector_weight=0.7):
        # Vector search
        query_embedding = embed(query)
        vector_results = self.vector_db.search(query_embedding, k=k*2)
        
        # Keyword search
        keyword_results = self.keyword_db.search(query, k=k*2)
        
        # Reciprocal Rank Fusion (RRF)
        return self.merge_rrf(vector_results, keyword_results, k)
    
    def merge_rrf(self, list1, list2, k, k_constant=60):
        scores = {}
        
        # Score from list 1
        for rank, item in enumerate(list1, 1):
            scores[item.id] = scores.get(item.id, 0) + 1 / (k_constant + rank)
        
        # Score from list 2
        for rank, item in enumerate(list2, 1):
            scores[item.id] = scores.get(item.id, 0) + 1 / (k_constant + rank)
        
        # Sort and return top k
        sorted_items = sorted(scores.items(), key=lambda x: x[1], reverse=True)
        return [item_id for item_id, score in sorted_items[:k]]
```

## 9. Embedding Quality Issues

### Problem 1: Domain Mismatch
```python
# Generic model trained on Wikipedia
query = "GPU OOM error"
similar = find_similar(query)
# Returns: general GPU info (not error-specific)

# Solution: Fine-tune on domain data
from sentence_transformers import SentenceTransformer, InputExample, losses

model = SentenceTransformer('all-MiniLM-L6-v2')

# Training data: (query, positive_doc, negative_doc)
train_examples = [
    InputExample(texts=["GPU OOM", "Out of memory error on CUDA device", "CPU memory leak"]),
    ...
]

# Fine-tune
model.fit(train_examples, epochs=3)
```

### Problem 2: Length Sensitivity
```python
short = "cat"
long = "cat sitting on a mat near the window on a sunny day"

# Both have "cat" but embeddings may be quite different
# Long text drowns out "cat" signal

# Solution: Chunk long texts or use pooling strategies
```

### Problem 3: Multilingual Challenges
```python
# English: "Good morning"
# Spanish: "Buenos días"
# Embedding similarity may be low despite same meaning

# Solution: Use multilingual models
from sentence_transformers import SentenceTransformer
model = SentenceTransformer('paraphrase-multilingual-mpnet-base-v2')
```

## 10. Cost Analysis

### Embedding Costs (per 1M tokens)
- OpenAI text-embedding-3-small: $0.02
- OpenAI text-embedding-3-large: $0.13
- Cohere embed-v3: $0.10
- Voyage-large-2: $0.12
- Self-hosted: $0 (after infrastructure cost)

### Storage Costs (per 1M vectors @ 1536 dims)
- Pinecone: ~$140/month (0.096/GB/month × 6GB)
- Self-hosted (SSD): ~$0.60/month ($0.10/GB)
- Self-hosted (RAM): ~$4/month ($0.70/GB)

**Break-even calculation:**
```
Self-hosted infrastructure: $500-2000/month
API costs at 10M queries/month: $130-260

Break-even: ~2-5M vectors or 5-10M queries/month
```

## 11. Production Patterns

### Pattern 1: Lazy Loading
```python
# Don't embed all documents upfront
class LazyEmbedding:
    def __init__(self, text):
        self.text = text
        self._embedding = None
    
    @property
    def embedding(self):
        if self._embedding is None:
            self._embedding = embed(self.text)
        return self._embedding
```

### Pattern 2: Batch Processing
```python
# Embed in batches for efficiency
def embed_documents(docs, batch_size=100):
    embeddings = []
    for i in range(0, len(docs), batch_size):
        batch = docs[i:i+batch_size]
        batch_embeddings = embedding_api.embed(batch)  # Single API call
        embeddings.extend(batch_embeddings)
    return embeddings

# 10-100× faster than one-at-a-time
```

### Pattern 3: Embedding Cache
```python
import hashlib
import redis

cache = redis.Redis()

def cached_embed(text):
    # Hash text as key
    key = hashlib.md5(text.encode()).hexdigest()
    
    # Check cache
    cached = cache.get(key)
    if cached:
        return pickle.loads(cached)
    
    # Generate and cache
    embedding = embed(text)
    cache.set(key, pickle.dumps(embedding), ex=86400)  # 24h TTL
    return embedding
```

### Pattern 4: Embedding Update Strategy
```python
# When switching embedding models
def gradual_migration(old_db, new_model):
    # 1. Dual-write: write to both old and new
    for doc in new_documents:
        old_embedding = old_model.embed(doc)
        new_embedding = new_model.embed(doc)
        
        old_db.insert(doc, old_embedding)
        new_db.insert(doc, new_embedding)
    
    # 2. Backfill: gradually re-embed existing docs
    for doc in old_db.documents:
        new_embedding = new_model.embed(doc.text)
        new_db.insert(doc, new_embedding)
    
    # 3. Dual-read: compare results, validate quality
    # 4. Switch traffic to new_db
    # 5. Deprecate old_db
```

## 12. Advanced Techniques

### ColBERT (Late Interaction)
```python
# Instead of single vector per document, one vector per token
# Query: "machine learning" → [v1, v2]
# Doc: "deep learning models" → [v1, v2, v3]

# Similarity: Max pooling over all token pairs
similarity = max(cosine(q_i, d_j) for q_i in query_vecs for d_j in doc_vecs)

# Benefits: 15-30% better quality
# Cost: 10-50× more storage, slower
```

### Matryoshka Embeddings
```python
# Single model produces embeddings at multiple dimensions
embedding_3072 = model.encode(text)

# Can truncate to smaller dimensions without retraining
embedding_1536 = embedding_3072[:1536]
embedding_768 = embedding_3072[:768]
embedding_256 = embedding_3072[:256]

# Use case: Trade quality for speed/storage dynamically
```

### Embedding Ensembles
```python
# Combine multiple embedding models
emb1 = model1.encode(text)  # OpenAI
emb2 = model2.encode(text)  # Cohere
emb3 = model3.encode(text)  # Custom

# Weighted average
ensemble = 0.5 * emb1 + 0.3 * emb2 + 0.2 * emb3

# Benefits: 5-10% quality improvement
# Cost: 3× more expensive
```

## 13. Monitoring & Debugging

### Key Metrics
```python
metrics = {
    "avg_search_latency_ms": 45,
    "p95_search_latency_ms": 120,
    "avg_results_returned": 9.2,
    "zero_results_rate": 0.02,  # 2% of queries
    "index_size_gb": 12.5,
    "index_build_time_hours": 2.3,
    "query_accuracy": 0.87  # If ground truth available
}
```

### Quality Checks
```python
# Test embedding quality
test_pairs = [
    ("cat", "kitten", 0.85),  # Expected similarity
    ("cat", "dog", 0.65),
    ("cat", "car", 0.1)
]

for text1, text2, expected_sim in test_pairs:
    actual_sim = cosine_similarity(embed(text1), embed(text2))
    assert abs(actual_sim - expected_sim) < 0.1, f"Failed: {text1} vs {text2}"
```

### A/B Testing
```python
# Compare embedding models
def compare_models(queries, ground_truth):
    results_a = [search_with_model_a(q) for q in queries]
    results_b = [search_with_model_b(q) for q in queries]
    
    accuracy_a = evaluate(results_a, ground_truth)
    accuracy_b = evaluate(results_b, ground_truth)
    
    return {
        "model_a": {"accuracy": accuracy_a, "cost": cost_a},
        "model_b": {"accuracy": accuracy_b, "cost": cost_b}
    }
```
