# Vector Databases & Embeddings - Interview Questions

## High-Probability Questions

### Q1: Explain what embeddings are and why they're useful for search.

**Answer:**

Embeddings are dense vector representations that capture semantic meaning.

```python
"king" → [0.23, -0.45, ..., 0.67]  # 1536 numbers
"queen" → [0.25, -0.43, ..., 0.69]  # Similar values = similar meaning
```

**Why useful for search:**
- **Semantic understanding**: Finds "car" when you search "automobile"
- **Beyond keywords**: Matches concepts, not just exact words
- **Cross-lingual**: Can match across languages (with multilingual models)
- **Context-aware**: "bank" (river) vs "bank" (money) have different embeddings

**Traditional keyword search:**
- Query: "machine learning" → Only matches exact phrase
- Misses: "ML", "artificial intelligence", "deep learning"

**Embedding search:**
- Query embedding: `[0.1, 0.5, ...]`
- Finds all semantically similar content regardless of wording

**Production impact**: Improved search relevance by 35% in our doc search system.

---

### Q2: Compare Pinecone, Weaviate, and pgvector. When would you use each?

**Answer:**

| Feature | Pinecone | Weaviate | pgvector |
|---------|----------|----------|----------|
| **Type** | Managed SaaS | OSS / Cloud | PostgreSQL extension |
| **Setup** | Zero (serverless) | Docker / Cloud | Add to existing Postgres |
| **Scaling** | Automatic | Manual / K8s | Manual |
| **Hybrid search** | No | Yes (built-in) | Manual setup |
| **Cost** | $70-500/month | $50-300/month | Infrastructure only |
| **Max vectors** | Billions | Billions | Millions |
| **Latency** | 20-50ms | 20-80ms | 10-30ms (small scale) |

**When to use Pinecone:**
- ✅ Startup/MVP (fast time-to-market)
- ✅ Don't want to manage infrastructure
- ✅ Need auto-scaling
- ❌ Cost-sensitive at scale (expensive beyond 10M vectors)

**When to use Weaviate:**
- ✅ Need hybrid search out-of-box
- ✅ Complex filtering requirements
- ✅ Want OSS with optional managed service
- ✅ GraphQL API preference

**When to use pgvector:**
- ✅ Already using PostgreSQL
- ✅ <1M vectors
- ✅ Need transactional consistency (ACID)
- ✅ Want to minimize dependencies
- ❌ Need to scale beyond 5-10M vectors

**Real decision**: We started with Pinecone (fast iteration), migrated to Weaviate at 10M vectors (10× cost savings).

---

### Q3: Your vector search is slow (>500ms). How do you optimize?

**Answer:**

**Diagnostic process:**

```python
# Measure components
vector_encode_time = 15ms   # Query embedding
vector_search_time = 450ms  # ← Problem
metadata_filter_time = 10ms
result_formatting_time = 25ms
total = 500ms
```

**Optimization strategies:**

**1. Add/optimize index**
```python
# HNSW index (if not present)
vector_db.create_index(
    type="HNSW",
    m=16,          # connections per node
    ef_search=100  # search accuracy (lower = faster)
)

# Tuning: ef_search 50 → 100 → 200
# Trade-off: 30ms → 50ms → 100ms (speed)
#           92% → 97% → 99% (accuracy)
```

**2. Use quantization**
```python
# Reduce FP32 → INT8
vector_db.enable_quantization(type="scalar")

# Results:
# - 4× less memory
# - 3× faster search
# - <2% accuracy loss
```

**3. Pre-filter optimization**
```python
# Bad: Post-filter (slow)
results = vector_db.search(query, k=1000)  # Retrieve many
filtered = [r for r in results if r.dept == "eng"][:10]  # Filter down

# Good: Pre-filter (fast)
results = vector_db.search(
    query, 
    k=10,
    filter={"dept": "eng"}  # DB-level filter
)
```

**4. Reduce search scope**
```python
# Use namespaces/partitions
results = vector_db.search(
    query,
    k=10,
    namespace=f"user_{user_id}"  # Only search user's data
)

# Reduces search space 100-1000×
```

**5. Caching**
```python
@cache(ttl=3600)
def search(query):
    return vector_db.search(embed(query), k=10)

# Typical hit rate: 30-50%
# Instant response for cached queries
```

**Real results**: Our optimizations:
- Added HNSW index: 450ms → 120ms
- Enabled quantization: 120ms → 40ms
- Added caching: 40% queries → <1ms

---

### Q4: How do you choose the right embedding dimension size?

**Answer:**

**Tradeoff matrix:**

| Dimensions | Quality | Speed | Storage | Use Case |
|------------|---------|-------|---------|----------|
| 384 | OK | Fast | 384 bytes | Simple search, cost-sensitive |
| 768 | Good | Medium | 768 bytes | Balanced |
| 1536 | Better | Slower | 1.5 KB | Production standard |
| 3072 | Best | Slow | 3 KB | Quality-critical |

**Considerations:**

1. **Model availability**
   - OpenAI: 1536 or 3072
   - Cohere: 1024
   - OSS (all-MiniLM): 384

2. **Storage costs**
   - 1M vectors × 1536 dims × 4 bytes = 6 GB
   - At $0.10/GB/month = $0.60/month (self-hosted)
   - At Pinecone prices = $140/month

3. **Search speed**
   - 384 dims: 10ms
   - 1536 dims: 30ms
   - 3072 dims: 60ms
   - (Approximate, varies by DB)

4. **Quality gains diminishing**
   - 384 → 768: +8% accuracy
   - 768 → 1536: +5% accuracy
   - 1536 → 3072: +2% accuracy

**Production recommendation**: 
- Start with 1536 (industry standard)
- Use 384 if cost/speed critical and quality acceptable
- Use 3072 only for quality-critical apps (legal, medical)

**Matryoshka embeddings**: New technique - single model produces variable dimensions. Truncate as needed.

---

### Q5: Explain cosine similarity vs dot product vs Euclidean distance.

**Answer:**

**Cosine Similarity**
```python
cos_sim = np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))
# Range: -1 to 1 (higher = more similar)
```
- **Measures**: Angle between vectors
- **Normalization**: Yes (divides by magnitude)
- **Use case**: Text embeddings (most common)
- **Pros**: Ignores magnitude, only cares about direction

**Dot Product**
```python
dot_prod = np.dot(v1, v2)
# Range: -∞ to ∞
```
- **Measures**: Magnitude × angle
- **Normalization**: No
- **Use case**: When vectors are pre-normalized
- **Pros**: Faster (no division)
- **Relation**: dot(v1, v2) = cos_sim(v1, v2) if ||v1||=||v2||=1

**Euclidean Distance (L2)**
```python
euclidean = np.linalg.norm(v1 - v2)
# Range: 0 to ∞ (lower = more similar)
```
- **Measures**: Straight-line distance
- **Normalization**: No
- **Use case**: Image embeddings, spatial data
- **Cons**: Sensitive to magnitude

**Production pattern:**
```python
# Store normalized embeddings
embedding = embedding / np.linalg.norm(embedding)

# Use dot product (fastest)
similarities = np.dot(embedding, all_embeddings.T)
# Equivalent to cosine similarity when normalized
```

**Performance:**
- Dot product: 1× (baseline)
- Cosine similarity: 1.5× slower (extra divisions)
- Euclidean: 1.3× slower (squaring)

---

### Q6: How do you handle embedding model updates without breaking production?

**Answer:**

**Challenge**: Switching embedding models means all vectors become incompatible.

**Migration strategies:**

**1. Blue-Green Deployment** (Recommended)
```python
# Step 1: Create new index with new model
new_index = VectorDB("v2")

for doc in documents:
    embedding = new_model.embed(doc.text)
    new_index.insert(doc, embedding)

# Step 2: Dual-write (new docs → both indexes)
def insert_document(doc):
    old_index.insert(doc, old_model.embed(doc.text))
    new_index.insert(doc, new_model.embed(doc.text))

# Step 3: A/B test
results_old = old_index.search(old_model.embed(query), k=10)
results_new = new_index.search(new_model.embed(query), k=10)
# Compare quality

# Step 4: Gradual cutover
# 5% traffic → new_index
# 50% traffic → new_index
# 100% traffic → new_index

# Step 5: Deprecate old index
```

**2. Lazy Migration**
```python
# Migrate vectors on-demand
def search(query):
    new_embedding = new_model.embed(query)
    
    # Search in new index first
    results = new_index.search(new_embedding, k=10)
    
    if len(results) < 10:
        # Fallback to old index
        old_embedding = old_model.embed(query)
        old_results = old_index.search(old_embedding, k=10)
        
        # Migrate retrieved documents
        for doc in old_results:
            new_emb = new_model.embed(doc.text)
            new_index.insert(doc, new_emb)
        
        results.extend(old_results)
    
    return results[:10]
```

**3. Parallel Scoring** (Small datasets)
```python
# Keep both embeddings per document
doc = {
    "text": "...",
    "embedding_v1": [...],  # Old model
    "embedding_v2": [...]   # New model
}

# Use appropriate embedding based on query model
if use_new_model:
    results = search_by_field("embedding_v2", query_emb)
else:
    results = search_by_field("embedding_v1", query_emb)
```

**Timeline:**
- Week 1: Build new index (background)
- Week 2: A/B test (5% traffic)
- Week 3: Ramp up (50% traffic)
- Week 4: Full cutover (100% traffic)
- Week 5: Deprecate old index

**Rollback plan**: Keep old index for 2-4 weeks post-migration.

---

## Deep-Dive Questions

### Q7: Your search returns semantically similar but irrelevant results. What's wrong and how do you fix it?

**Answer:**

**Example problem:**
- Query: "python programming tutorial"
- Returns: "snake species in Python national park" (semantically similar words, wrong context)

**Root causes:**

**1. Embedding model lacks domain context**
```python
# Generic model sees "python" as ambiguous
# Solution: Fine-tune on domain data

from sentence_transformers import SentenceTransformer, InputExample

model = SentenceTransformer('all-MiniLM-L6-v2')

train_data = [
    InputExample(texts=[
        "python programming",  # anchor
        "python coding tutorial",  # positive (similar)
        "snake species"  # negative (dissimilar)
    ]),
    ...
]

model.fit(train_data, epochs=3)
```

**2. Missing metadata filters**
```python
# Add category/domain metadata
doc = {
    "text": "python tutorial",
    "embedding": [...],
    "category": "programming",  # ← Add this
    "domain": "technology"
}

# Search with filter
results = vector_db.search(
    query_embedding,
    k=10,
    filter={"category": "programming"}  # Avoid snake results
)
```

**3. Hybrid search helps**
```python
# Keyword search catches exact "programming"
keyword_results = bm25_search("python programming", k=20)

# Vector search for semantic similarity
vector_results = vector_search(embed(query), k=20)

# Merge (RRF)
final = merge_results([keyword_results, vector_results], k=10)

# "programming" keyword boosts relevant results
```

**4. Query clarification**
```python
# Ask LLM to add context
clarified = llm.clarify(query)
# Input: "python tutorial"
# Output: "python programming language tutorial for beginners"

# Search with clarified query
results = search(embed(clarified), k=10)
```

**5. Reranking**
```python
# Get 50 candidates (some irrelevant)
candidates = vector_search(query, k=50)

# Rerank with cross-encoder (understands full context)
reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')
pairs = [(query, doc.text) for doc in candidates]
scores = reranker.predict(pairs)

# Top 10 after reranking will be more relevant
top_10 = sort_by_scores(candidates, scores)[:10]
```

**Production fix**: Implemented hybrid search + domain filters. Reduced irrelevant results from 30% to 8%.

---

### Q8: How do you scale vector search to billions of vectors?

**Answer:**

**Scaling challenges:**

| Scale | Vectors | Storage | RAM | Search Time |
|-------|---------|---------|-----|-------------|
| Small | 1M | 6 GB | 8 GB | 20ms |
| Medium | 100M | 600 GB | 800 GB | 50ms |
| Large | 1B | 6 TB | 8 TB | 100ms |
| Huge | 10B+ | 60 TB | 80 TB | 200ms+ |

**Strategies:**

**1. Sharding**
```python
# Partition vectors across nodes
def distribute_vectors(vectors, num_shards=10):
    for i, vec in enumerate(vectors):
        shard_id = hash(vec.id) % num_shards
        shards[shard_id].insert(vec)

# Search in parallel
async def search_sharded(query, k=10):
    tasks = [shard.search(query, k=k) for shard in shards]
    all_results = await asyncio.gather(*tasks)
    
    # Merge results from all shards
    return merge_top_k(all_results, k=k)
```

**2. Hierarchical indexing**
```python
# Level 1: Cluster centers (10K vectors)
cluster_centers = kmeans(all_vectors, n_clusters=10000)

# Level 2: Full vectors in each cluster (1M each)
clusters = {i: [] for i in range(10000)}

# Search: Find nearest cluster, then search within cluster
nearest_cluster = find_nearest(query, cluster_centers)
results = clusters[nearest_cluster].search(query, k=10)

# Reduces search space 1000× (1B → 1M vectors)
```

**3. Product Quantization**
```python
# Compress vectors dramatically
# Original: 1536 dims × 4 bytes = 6 KB
# PQ: 64 bytes (96× compression)

# Trade-off: 5-10% accuracy loss
# Benefit: Fit 96× more vectors in RAM
```

**4. IVFPQ (Inverted File + Product Quantization)**
```python
# Combine clustering + quantization
index = faiss.IndexIVFPQ(
    quantizer,
    dimension=1536,
    nlist=4096,  # number of clusters
    m=64,  # number of subquantizers
    nbits=8  # bits per subquantizer
)

# Memory: ~100 bytes per vector (vs 6 KB)
# Speed: 10-50ms for billions of vectors
# Accuracy: 85-95% recall@10
```

**5. Approximate search tolerances**
```python
# Don't need perfect results, top-10 is approximate
index.set_ef(50)  # Lower = faster, less accurate

# 99% accuracy: ef=200, 100ms
# 95% accuracy: ef=100, 50ms
# 90% accuracy: ef=50, 25ms
```

**6. Filtering before search**
```python
# Metadata filtering reduces search space
results = index.search(
    query,
    k=10,
    filter={
        "user_id": user_id,  # Only user's 100K vectors
        "date": "last_30_days"  # Not all history
    }
)

# Search 100K instead of 1B (10,000× smaller)
```

**Real-world example**: Pinterest
- 5 billion pins
- Uses IVFPQ + sharding
- Search latency: 20-50ms
- Infrastructure: 100+ nodes

**Cost estimate (1B vectors):**
- Storage: 60-600 GB (with quantization)
- RAM: 60-600 GB (depends on index type)
- Servers: 10-50 machines
- Monthly cost: $2K-10K

---

### Q9: Compare HNSW vs IVF indexing algorithms.

**Answer:**

**HNSW (Hierarchical Navigable Small World)**

**How it works:**
```
Layer 2: Few nodes, long-range connections
Layer 1: More nodes, medium connections
Layer 0: All nodes, short connections

Search: Start at top layer, navigate down
```

**Characteristics:**
- **Build time**: Slow (hours for 100M vectors)
- **Search speed**: Very fast (10-50ms)
- **Memory**: High (1.5× raw data)
- **Accuracy**: 95-99% recall
- **Updates**: Supports incremental inserts (but slower)

**Parameters:**
```python
index = hnswlib.Index(space='cosine', dim=1536)
index.init_index(
    max_elements=1000000,
    M=16,  # connections per node (16-64)
    ef_construction=200  # build accuracy
)
index.set_ef(100)  # search accuracy
```

**Tuning:**
- M: 16 (fast), 32 (balanced), 64 (accurate)
- ef_construction: 100 (fast build), 200 (balanced), 400 (slow, accurate)
- ef: 50 (fast search), 100 (balanced), 200 (accurate)

---

**IVF (Inverted File)**

**How it works:**
```
1. Cluster vectors into N groups (k-means)
2. Query: Find nearest clusters
3. Search only within those clusters
```

**Characteristics:**
- **Build time**: Fast (minutes for 100M vectors)
- **Search speed**: Fast (20-100ms)
- **Memory**: Low (0.1-0.3× raw data with quantization)
- **Accuracy**: 80-95% recall
- **Updates**: Requires rebuild for new clusters

**Parameters:**
```python
index = faiss.IndexIVFFlat(
    quantizer,
    dim=1536,
    nlist=4096  # number of clusters
)
index.nprobe = 32  # search this many clusters
```

**Tuning:**
- nlist: 1024 (fast), 4096 (balanced), 16384 (accurate)
- nprobe: 10 (fast), 32 (balanced), 128 (accurate)

---

**Comparison Table:**

| Metric | HNSW | IVF |
|--------|------|-----|
| Search speed | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Memory usage | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Build time | ⭐⭐ | ⭐⭐⭐⭐ |
| Accuracy | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Incremental updates | ⭐⭐⭐⭐ | ⭐⭐ |

**When to use HNSW:**
- ✅ Need best accuracy (>95% recall)
- ✅ Can afford memory (2× RAM)
- ✅ Frequent incremental updates
- ✅ <100M vectors
- ❌ Memory constrained

**When to use IVF:**
- ✅ Memory constrained
- ✅ Billions of vectors
- ✅ Can tolerate 85-90% recall
- ✅ Batch updates (not real-time)
- ❌ Need >95% accuracy

**Production recommendation**: HNSW for <100M vectors, IVF+PQ for billions.

---

##Scenario-Based Questions

### S1: Design embedding strategy for multi-tenant SaaS with 1000 customers, each with 10K docs.

**Answer:**

**Requirements:**
- Isolation: Customer A can't see Customer B's data
- Performance: Fast search within each tenant
- Cost: Efficient storage
- Scalability: New customers shouldn't slow down existing ones

**Strategy:**

**Option 1: Separate index per tenant** (Small scale)
```python
# Create index per customer
indexes = {
    "customer_001": VectorDB("customer_001"),
    "customer_002": VectorDB("customer_002"),
    ...
}

# Search within customer's index
def search(customer_id, query):
    index = indexes[customer_id]
    return index.search(embed(query), k=10)
```

**Pros:** Perfect isolation, simple
**Cons:** Doesn't scale beyond 100-1000 tenants, high overhead

**Option 2: Shared index with namespace** (Production choice)
```python
# Single index, metadata filtering
vector_db = VectorDB("multi_tenant")

# Insert with customer_id
vector_db.insert(
    embedding=emb,
    text=text,
    metadata={"customer_id": "customer_001"}
)

# Search with filter
def search(customer_id, query):
    return vector_db.search(
        embed(query),
        k=10,
        filter={"customer_id": customer_id}
    )
```

**Pros:** Scales to 10K+ tenants, efficient
**Cons:** Must ensure filter security (no customer_id leakage)

**Option 3: Hybrid (Large tenants separate)**
```python
# VIP customers (>100K docs): dedicated index
# Regular customers: shared index

def get_index(customer_id):
    if is_vip_customer(customer_id):
        return dedicated_indexes[customer_id]
    else:
        return shared_index

def search(customer_id, query):
    index = get_index(customer_id)
    
    if index == shared_index:
        return index.search(
            embed(query),
            k=10,
            filter={"customer_id": customer_id}
        )
    else:
        return index.search(embed(query), k=10)
```

**Cost analysis (1000 customers × 10K docs each):**
- Total vectors: 10M
- Storage: 60 GB (with quantization)
- Shared index: $140/month (Pinecone) or $10/month (self-hosted)
- Separate indexes: $140K/month (Pinecone) 😱

**Recommendation:** Shared index with metadata filtering. Upgrade large tenants to dedicated as needed.

**Security measures:**
```python
# Never trust client-provided customer_id
def search(request):
    # Get customer_id from authenticated session
    customer_id = auth.get_customer_id(request.token)
    
    # Enforce at DB level
    return vector_db.search(
        embed(request.query),
        k=10,
        filter={"customer_id": customer_id}  # Server-side, tamper-proof
    )
```
