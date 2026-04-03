# RAG Systems - Interview Questions

## High-Probability Questions

### Q1: Explain how RAG works and why we need it.

**Answer:**

**RAG = Retrieval-Augmented Generation**

**How it works:**
1. User asks question: "What's our return policy?"
2. Convert question to vector embedding
3. Search vector database for similar content
4. Retrieve top 5-10 relevant chunks
5. Pass chunks + question to LLM as context
6. LLM generates answer grounded in retrieved context

**Why we need RAG:**

| Problem | Solution with RAG |
|---------|------------------|
| LLM knowledge cutoff | Retrieve up-to-date information |
| Hallucinations | Ground answers in factual documents |
| Private/domain data | Access proprietary information |
| Cost of fine-tuning | Cheaper than retraining models |
| Dynamic data | Update documents, not model |

**Real-world impact**: RAG reduced hallucinations by 60-80% in our customer support chatbot.

**Follow-up ready**: "When NOT to use RAG?" → "When answers require reasoning beyond facts (math, logic), or when all needed knowledge fits in prompt."

---

### Q2: What chunk size would you use and why?

**Answer:**

**Recommended: 512-1024 tokens with 10-20% overlap**

**Reasoning:**

| Chunk Size | Pros | Cons | Use Case |
|------------|------|------|----------|
| 128-256 | Precise, fast search | Context fragmentation | Q&A on specific facts |
| 512-1024 | **Balanced** | Good tradeoff | Most production use cases |
| 2048+ | Full context | Slower, less precise | Long-form content |

**Factors to consider:**
1. **Query type**: 
   - Specific facts → smaller chunks (256-512)
   - Conceptual questions → larger chunks (1024+)

2. **Document structure**:
   - Short paragraphs → 512 tokens
   - Technical docs with context → 1024 tokens

3. **LLM context limit**:
   - 10 chunks × 1024 tokens = 10K tokens
   - Leaves room for query + response

4. **Overlap**: 50-100 tokens prevents splitting mid-concept

**Production example**: We tested 256, 512, 1024 tokens. 512 gave best precision/recall balance for our knowledge base.

**Follow-up**: "How do you determine optimal size?" → "A/B test on eval set, measure answer quality and retrieval precision."

---

### Q3: Walk me through debugging poor RAG performance.

**Answer:**

**Systematic debugging process:**

**Step 1: Check retrieval quality**
```python
# For each query in test set:
retrieved_docs = retrieve(query, k=10)

# Are relevant docs in top-K?
recall = measure_recall(retrieved_docs, ground_truth_docs)

if recall < 0.7:
    # Problem: Retrieval failure
    # Solutions: Better embeddings, hybrid search, query expansion
```

**Step 2: Inspect retrieved documents**
```python
print(f"Query: {query}")
for i, doc in enumerate(retrieved_docs):
    print(f"[{i}] Score: {doc.score}")
    print(f"Content: {doc.text[:200]}...")
    
# Are docs actually relevant? Or just semantically similar but unhelpful?
```

**Step 3: Check LLM response**
```python
# Test LLM with perfect context
perfect_context = ground_truth_docs
response = llm(query, perfect_context)

if response is good:
    # Problem: Retrieval
else:
    # Problem: LLM or prompt
```

**Step 4: Test components in isolation**
```python
# Test embedding quality
similar_queries = find_similar("GPU error", k=5)
# Should return ["CUDA error", "GPU crash", "memory error"]

# Test LLM without RAG
response = llm(query, no_context)
# Establishes baseline
```

**Common root causes:**

| Symptom | Root Cause | Fix |
|---------|-----------|------|
| Irrelevant docs retrieved | Poor embeddings | Switch model, hybrid search |
| Right docs, wrong answer | Prompt issues | Improve prompt, citations |
| Inconsistent quality | Chunk size | Adjust chunking strategy |
| Slow responses | Too many chunks | Reduce K, add reranking |

**Production approach**: Log retrieval results for failed queries, analyze patterns weekly, iterate.

---

### Q4: Explain vector search vs keyword search. When would you use hybrid search?

**Answer:**

**Vector Search (Semantic)**
```python
query_embedding = embed("car accident")
results = vector_db.search(query_embedding, k=10)
# Finds: "vehicle collision", "auto crash", "traffic incident"
```

**Pros:**
- Semantic understanding (synonyms, concepts)
- Works across languages
- Finds conceptually similar, not just lexically matching

**Cons:**
- Misses exact terms (IDs, names, model numbers)
- Struggles with rare words, acronyms
- Computationally expensive

**Keyword Search (BM25/Lexical)**
```python
results = bm25_search("car accident", k=10)
# Finds: exact phrase "car accident"
```

**Pros:**
- Fast exact matching
- Great for IDs, names, technical terms
- No embedding needed

**Cons:**
- No semantic understanding
- Misses synonyms ("automobile crash")
- Language-specific

**Hybrid Search**
```python
# Combine both with weights
vector_results = vector_search(query, k=20, weight=0.7)
keyword_results = bm25_search(query, k=20, weight=0.3)

# Merge using RRF (Reciprocal Rank Fusion)
final = merge_results([vector_results, keyword_results], k=10)
```

**When to use hybrid:**
1. **Queries with specific terms**: "Show me invoice #INV-2024-001"
   - Keyword catches exact ID, vector provides context

2. **Domain with jargon**: Medical, legal, technical docs
   - Vector for concepts, keyword for precise terminology

3. **Mixed query types**: Some users search precisely, others conceptually

**Real-world results**: Hybrid search improved our RAG accuracy from 72% to 84%.

**Follow-up**: "How do you tune weights?" → "A/B test different ratios (0.5/0.5, 0.7/0.3, 0.8/0.2), measure retrieval metrics."

---

### Q5: Your RAG system is slow (2-3 seconds per query). How do you optimize?

**Answer:**

**Performance breakdown:**
- Vector search: 50-200ms
- Reranking: 100-300ms (if used)
- LLM inference: 1-2s (dominant)
- Total: 1.5-2.5s

**Optimization strategies:**

**1. Reduce LLM latency** (biggest impact)
```python
# Option A: Smaller model
GPT-4 (1.5s) → GPT-3.5 (300ms)  # 5x faster, 10x cheaper

# Option B: Reduce context
top_10_chunks → top_5_chunks  # -50% tokens

# Option C: Compress context
compressed = llm_compress(chunks, ratio=0.5)  # -50% tokens, <5% quality loss

# Option D: Use streaming
for chunk in llm.stream(prompt):
    yield chunk  # User sees partial response immediately
```

**2. Optimize retrieval**
```python
# Add ANN index (Approximate Nearest Neighbors)
vector_db.create_index(type="HNSW")  # 10x faster search

# Pre-filter by metadata
results = vector_db.search(
    embedding, 
    k=10,
    filter={"category": user_category}  # Reduces search space
)

# Cache frequent queries
@cache(ttl=3600)  # 1 hour
def retrieve(query):
    ...
```

**3. Remove reranking** (if used)
```python
# Reranking adds 100-300ms
# Only use if precision is critical (e.g., legal, medical)
```

**4. Parallel processing**
```python
async def rag_pipeline(query):
    # Run retrieval and query processing in parallel
    embedding, rewritten_query = await asyncio.gather(
        embed_async(query),
        rewrite_query_async(query)
    )
    ...
```

**5. Batch requests** (for multiple users)
```python
# Batch embeddings
queries = [q1, q2, ..., q10]
embeddings = embed_batch(queries)  # 3x faster than sequential

# Batch LLM calls
responses = llm.batch([prompt1, prompt2, ...])
```

**Real impact**: Our optimizations:
- GPT-4 → GPT-3.5: -70% latency
- Context compression: -30% latency
- Caching: 40% hit rate (instant response)
- **Overall: 2.5s → 600ms**

**Follow-up**: "What if we need GPT-4 quality?" → "Use tiered approach: GPT-3.5 for 80% of queries, GPT-4 for complex ones."

---

### Q6: How do you handle documents larger than the context window?

**Answer:**

**Scenario**: 200-page PDF (100K tokens), model limit is 32K tokens.

**Strategy comparison:**

| Approach | When to use | Pros | Cons |
|----------|------------|------|------|
| **RAG** | Q&A, search | Precise, fast | Needs good chunking |
| **Summarization** | "Give me the gist" | Captures overview | Loses details |
| **Map-Reduce** | Extraction tasks | Parallelizable | Complex queries fail |
| **Long-context model** | <100 pages | Simple | Expensive, slow |

**Recommended: RAG (most common)**
```python
# 1. Chunk document
chunks = split_document(
    document, 
    chunk_size=1000, 
    overlap=100
)

# 2. Embed all chunks
embeddings = [embed(chunk) for chunk in chunks]

# 3. Store in vector DB
vector_db.insert(chunks, embeddings, metadata)

# 4. At query time, retrieve relevant chunks
relevant = vector_db.search(embed(query), k=5)

# 5. Pass only relevant chunks to LLM (fits in context)
answer = llm(query, relevant)
```

**Alternative: Hierarchical summarization**
```python
# Use when user wants "overall summary" not specific Q&A

def hierarchical_summarize(document):
    # Level 1: Split into sections
    sections = split_by_sections(document)
    
    # Level 2: Summarize each section
    summaries = [llm.summarize(section) for section in sections]
    
    # Level 3: Summarize the summaries
    final_summary = llm.summarize(summaries)
    
    return final_summary
```

**Alternative: Map-Reduce for extraction**
```python
# Use for: "Extract all dates", "List all people mentioned"

def map_reduce_extract(document, task):
    chunks = split_document(document)
    
    # Map: Extract from each chunk independently
    results = [llm.extract(task, chunk) for chunk in chunks]
    
    # Reduce: Combine and deduplicate
    final = deduplicate_and_merge(results)
    
    return final
```

**Production reality**: 90% of queries use RAG. Summarization and map-reduce for special cases.

---

### Q7: How do you prevent RAG from hallucinating?

**Answer:**

**Defense strategies:**

**1. Strict prompting**
```python
system_prompt = """
You are a helpful assistant. Follow these rules STRICTLY:

1. Answer ONLY based on the provided context
2. If the answer is not in the context, say "I don't have this information"
3. Cite your sources: [Doc 1], [Doc 2]
4. Do NOT use your general knowledge
5. Do NOT make assumptions or inferences beyond what's stated
"""
```

**2. Citation enforcement**
```python
# Format context with source labels
context = ""
for i, doc in enumerate(retrieved_docs, 1):
    context += f"\n[Document {i}]\n{doc.text}\n"

prompt = f"""
{context}

Question: {query}
Answer with citations [1], [2], etc.
"""
```

**3. Answer verification**
```python
def verify_answer(context, answer):
    # Use NLI model to check entailment
    nli_model = load_nli_model()
    
    result = nli_model(premise=context, hypothesis=answer)
    
    if result == "contradiction":
        return "HALLUCINATION DETECTED"
    elif result == "neutral":
        return "UNVERIFIABLE"
    else:
        return answer  # entailment = good
```

**4. Self-consistency check**
```python
# Generate answer 3 times with different temperature
answers = [llm(prompt, temperature=t) for t in [0.3, 0.5, 0.7]]

# Check if answers agree
if all_similar(answers):
    return answers[0]  # Confident
else:
    return "I'm not confident in this answer. Please verify."
```

**5. Confidence scoring**
```python
# Use logprobs to estimate confidence
response = llm(prompt, logprobs=True)

avg_logprob = np.mean([token.logprob for token in response.tokens])

if avg_logprob < -2.0:  # Low confidence
    return "Answer confidence is low. Please verify."
```

**6. Retrieval quality check**
```python
# Check if retrieved docs are relevant
def check_retrieval(query, docs):
    scores = [relevance_score(query, doc) for doc in docs]
    
    if max(scores) < 0.7:
        return "No relevant information found"
    
    # Filter low-quality docs
    filtered = [doc for doc, score in zip(docs, scores) if score > 0.5]
    return filtered
```

**Real impact**: Combination of these reduced hallucinations from 25% to <5% in our production system.

**Follow-up**: "How do you measure hallucination rate?" → "Human eval on sample, or use LLM-as-judge to check factual consistency."

---

### Q8: Explain reranking and when you'd use it.

**Answer:**

**What is reranking?**

Two-stage retrieval:
1. **Stage 1**: Fast, approximate retrieval (vector search) → Get 50 candidates
2. **Stage 2**: Slow, accurate reranking (cross-encoder) → Rerank to top 10

**Why rerank?**

| Metric | Vector Search | After Reranking |
|--------|--------------|----------------|
| Precision@10 | 0.65 | 0.82 (+26%) |
| Latency | 50ms | 150ms (+100ms) |
| Relevance | Good | Excellent |

**How it works:**

```python
# Stage 1: Bi-encoder (fast)
query_emb = embed(query)
doc_embs = [embed(doc) for doc in all_docs]
similarities = cosine_similarity(query_emb, doc_embs)
top_50 = get_top_k(similarities, k=50)

# Stage 2: Cross-encoder (slow but accurate)
reranker = CrossEncoder('ms-marco-MiniLM-L-6-v2')
pairs = [(query, doc.text) for doc in top_50]
scores = reranker.predict(pairs)

# Return best 10 after reranking
top_10 = sort_by_scores(top_50, scores)[:10]
```

**When to use reranking:**

✅ **Use when:**
- Precision is critical (legal, medical, finance)
- Users expect highly relevant results
- You can afford +100-200ms latency
- You have 50+ candidate documents

❌ **Skip when:**
- Latency budget is tight (<200ms)
- Initial retrieval is already good (>90% precision)
- High QPS (queries per second) - reranking doesn't scale well
- Simple queries (exact matches)

**Cost analysis:**
- Reranking 50 candidates: ~150ms, negligible cost (if self-hosted)
- Quality improvement: 15-30%
- **Verdict**: Worth it for high-value queries

**Production tip**: Rerank top 50 down to 10, not top 500 (too slow).

**Follow-up**: "Cross-encoder vs bi-encoder?" → "Bi-encoder encodes query and docs separately (fast, approximate). Cross-encoder encodes both together (slow, accurate)."

---

## Deep-Dive Follow-Up Questions

### Q9: How do you handle multi-hop questions in RAG?

**Answer:**

**Example multi-hop question:**
"Who is the CEO of the company that acquired our main competitor in 2023?"

Requires:
1. Find our main competitor
2. Find who acquired them in 2023
3. Find the CEO of that company

**Approach 1: Iterative RAG**
```python
def multi_hop_rag(question, max_hops=3):
    context = []
    current_q = question
    
    for hop in range(max_hops):
        # Retrieve relevant docs
        docs = retrieve(current_q, k=5)
        context.extend(docs)
        
        # Check if we can answer
        if can_answer(question, context):
            return llm.answer(question, context)
        
        # Generate sub-question
        current_q = llm.decompose(question, context)
        # Example: "What company acquired our competitor in 2023?"
    
    return llm.answer(question, context)
```

**Approach 2: Query decomposition**
```python
# Break complex question into sub-questions
sub_questions = llm.decompose(question)
# ["What is our main competitor?",
#  "Who acquired them in 2023?",
#  "Who is the CEO of that company?"]

# Answer each sub-question
answers = []
for sub_q in sub_questions:
    docs = retrieve(sub_q, k=5)
    answer = llm.answer(sub_q, docs)
    answers.append(answer)

# Synthesize final answer
final_answer = llm.synthesize(question, answers)
```

**Approach 3: Graph RAG**
```python
# Build knowledge graph from documents
graph = build_knowledge_graph(documents)

# Query graph
path = graph.find_path(
    start="our company",
    relation="competitor_acquired_by",
    depth=2
)

# Use path as context
answer = llm.answer(question, path)
```

**Production reality**: Most systems use iterative RAG (simpler, works 80% of the time). Graph RAG for specialized use cases.

---

### Q10: How do you maintain RAG quality as your knowledge base grows from 100 to 100K documents?

**Answer:**

**Challenges with scale:**

| Scale | Docs | Chunks | Challenges |
|-------|------|--------|-----------|
| Small | 100 | 1K | Works easily |
| Medium | 10K | 100K | Retrieval slows |
| Large | 100K | 1M+ | Quality degrades, cost increases |

**Solutions:**

**1. Hierarchical indexing**
```python
# Create document-level summaries
for doc in documents:
    doc.summary = llm.summarize(doc)
    doc.keywords = extract_keywords(doc)

# Two-stage retrieval:
# Stage 1: Find relevant documents (summary search)
relevant_docs = search_summaries(query, k=10)

# Stage 2: Find relevant chunks within those docs
relevant_chunks = search_chunks_in_docs(query, relevant_docs, k=10)
```

**2. Metadata filtering**
```python
# Pre-filter before semantic search
results = vector_db.search(
    query_embedding,
    k=10,
    filter={
        "department": user.department,
        "date": {"$gte": "2024-01-01"},
        "access_level": {"$lte": user.access_level}
    }
)

# Reduces search space 10-100x
```

**3. Namespace/partition by domain**
```python
# Separate indexes for different domains
indexes = {
    "engineering": VectorDB("eng_docs"),
    "sales": VectorDB("sales_docs"),
    "legal": VectorDB("legal_docs")
}

# Route query to relevant index
domain = classify_domain(query)
results = indexes[domain].search(query_embedding, k=10)
```

**4. Approximate nearest neighbors (ANN)**
```python
# Use HNSW or IVF index for fast search
vector_db.create_index(
    type="HNSW",
    m=16,  # connections per node
    ef_construction=200  # build-time accuracy
)

# 10-100x faster than exact search
```

**5. Regular re-indexing**
```python
# As embeddings improve, re-embed periodically
def re_index_all(documents):
    new_embeddings = embed_batch(documents, model="text-embedding-3-large")
    vector_db.update_embeddings(new_embeddings)

# Run monthly or when embedding model upgrades
```

**6. Deduplication**
```python
# Remove duplicate or near-duplicate chunks
def deduplicate(chunks):
    seen = set()
    unique = []
    for chunk in chunks:
        chunk_hash = hash(chunk.text)
        if chunk_hash not in seen:
            unique.append(chunk)
            seen.add(chunk_hash)
    return unique

# Reduces index size 20-40%
```

**Real experience**: At 100K docs, we implemented hierarchical indexing + metadata filtering. Reduced retrieval time from 800ms to 120ms while improving precision.

---

### Q11: Scenario: RAG works great for English but fails for other languages. Why and how to fix?

**Answer:**

**Root causes:**

**1. Embedding model is English-centric**
```python
# text-embedding-ada-002 is primarily English-trained
# Performance degrades for non-English languages

# Test: Compare similarity scores
english_sim = cosine_sim(embed("cat"), embed("dog"))  # 0.75
spanish_sim = cosine_sim(embed("gato"), embed("perro"))  # 0.45 (worse)
```

**2. Documents and queries in different languages**
```python
# User queries in Spanish, documents in English
query = embed("¿Cuál es el precio?")
docs = embed_documents(["Price is $100", ...])
# Poor semantic matching across languages
```

**3. Chunking doesn't respect language boundaries**
```python
# Document has mixed languages
text = "Hello. مرحبا. 你好."
# Chunker may split inappropriately
```

**Solutions:**

**1. Use multilingual embedding model**
```python
# Instead of:
embeddings = OpenAIEmbeddings(model="text-embedding-ada-002")

# Use:
from sentence_transformers import SentenceTransformer
embeddings = SentenceTransformer('paraphrase-multilingual-mpnet-base-v2')

# Or:
embeddings = SentenceTransformer('intfloat/multilingual-e5-large')
```

**2. Translate to pivot language**
```python
def retrieve_multilingual(query, user_language):
    # Translate query to English
    if user_language != "en":
        query_en = translate(query, target="en")
    else:
        query_en = query
    
    # Search in English
    docs = retrieve(query_en, k=10)
    
    # Translate results back (if needed)
    if user_language != "en":
        docs = [translate(doc, target=user_language) for doc in docs]
    
    return docs

# Cost: $0.001-0.01 per query (Google Translate)
```

**3. Language-specific indexes**
```python
# Separate indexes per language
indexes = {
    "en": VectorDB("english_docs"),
    "es": VectorDB("spanish_docs"),
    "zh": VectorDB("chinese_docs")
}

# Route based on detected language
language = detect_language(query)
results = indexes[language].search(query_embedding, k=10)
```

**4. Hybrid approach (best quality)**
```python
# 1. Store documents in original language
# 2. Create English translations for retrieval
# 3. Search using English embedding
# 4. Return original language documents

doc = {
    "text_original": "El precio es $100",
    "text_en": "The price is $100",
    "language": "es"
}

# Search using English
query_en = translate(query, "en")
results = search(embed(query_en))

# Return original Spanish text
return [doc.text_original for doc in results]
```

**Production recommendation**: Use multilingual embeddings (simplest) or translate to English (best quality).

---

### Q12: How do you version and update your RAG knowledge base?

**Answer:**

**Challenge**: Documents change, need to keep RAG up-to-date without breaking existing functionality.

**Architecture:**

```python
class DocumentStore:
    def __init__(self):
        self.vector_db = VectorDB()
        self.metadata_db = SQL()  # Track versions
    
    def add_document(self, doc, version="1.0"):
        # Chunk and embed
        chunks = chunk_document(doc)
        embeddings = embed_batch(chunks)
        
        # Store with version metadata
        for chunk, emb in zip(chunks, embeddings):
            self.vector_db.insert(
                embedding=emb,
                text=chunk.text,
                metadata={
                    "doc_id": doc.id,
                    "version": version,
                    "created_at": datetime.now(),
                    "is_latest": True
                }
            )
        
        # Mark previous versions as not latest
        self.metadata_db.update(
            doc_id=doc.id,
            set={"is_latest": False},
            where={"version": {"$ne": version}}
        )
    
    def update_document(self, doc_id, new_content):
        # Get current version
        current = self.metadata_db.get(doc_id)
        new_version = increment_version(current.version)
        
        # Add new version
        self.add_document(new_content, version=new_version)
        
        # Keep old versions for audit (optional)
        # Or delete: self.delete_version(doc_id, current.version)
    
    def retrieve(self, query, use_latest=True):
        filter_clause = {"is_latest": True} if use_latest else {}
        
        return self.vector_db.search(
            embed(query),
            k=10,
            filter=filter_clause
        )
```

**Update strategies:**

**1. Full re-index** (simple, safe)
```python
# When changing embedding model or chunking strategy
def full_reindex(documents):
    # Create new index
    new_index = VectorDB("v2")
    
    # Re-process all documents
    for doc in documents:
        chunks = chunk_document(doc)  # New chunking
        embeddings = embed_batch(chunks)  # New model
        new_index.insert(chunks, embeddings)
    
    # Switch traffic to new index (zero downtime)
    # 1. Deploy new index
    # 2. Test in shadow mode
    # 3. Gradually shift traffic
    # 4. Retire old index
```

**2. Incremental updates**
```python
def incremental_update(changed_docs):
    for doc in changed_docs:
        # Remove old chunks
        vector_db.delete(filter={"doc_id": doc.id})
        
        # Add new chunks
        chunks = chunk_document(doc)
        embeddings = embed_batch(chunks)
        vector_db.insert(chunks, embeddings)
```

**3. Time-aware retrieval**
```python
# Retrieve most recent versions within time window
results = vector_db.search(
    embed(query),
    k=10,
    filter={
        "created_at": {"$gte": datetime.now() - timedelta(days=30)},
        "is_latest": True
    }
)
```

**Monitoring updates:**
```python
# Track metrics before/after update
metrics = {
    "pre_update": measure_quality(test_queries, old_index),
    "post_update": measure_quality(test_queries, new_index)
}

# Alert if quality degrades >5%
if metrics["post_update"] < metrics["pre_update"] * 0.95:
    alert("Quality regression detected!")
    rollback()
```

**Production best practices:**
- Version everything (docs, embeddings, models)
- Blue-green deployment for major updates
- Keep audit trail (who changed what when)
- Test updates on sample before full rollout
- Monitor quality metrics continuously

---

## Scenario-Based Questions

### S1: Design a RAG system for a 10K-document legal knowledge base where accuracy is critical.

**Answer:**

**Key requirements:**
- 100% accuracy (lives/money at stake)
- Citability (must cite sources)
- Compliance (audit trail, version control)

**Architecture:**

```python
class LegalRAG:
    def __init__(self):
        # Primary: Vector search
        self.vector_db = PineconeDB(index="legal_docs")
        
        # Secondary: Keyword search (for case numbers, statute IDs)
        self.keyword_db = ElasticsearchDB(index="legal_docs")
        
        # Reranker for precision
        self.reranker = CrossEncoder('legal-reranker-model')
        
        # NLI for verification
        self.verifier = NLIModel('legal-nli')
    
    def retrieve(self, query, user_context):
        # Hybrid search
        vector_results = self.vector_db.search(embed(query), k=30)
        keyword_results = self.keyword_db.search(query, k=30)
        
        # Merge with RRF
        candidates = merge_rrf([vector_results, keyword_results], k=50)
        
        # Filter by access control
        accessible = [doc for doc in candidates 
                      if user_context.can_access(doc)]
        
        # Rerank for precision
        scores = self.reranker.predict([(query, doc.text) for doc in accessible])
        top_docs = sort_by_scores(accessible, scores)[:10]
        
        return top_docs
    
    def answer(self, query, user_context):
        docs = self.retrieve(query, user_context)
        
        # Format with citations
        context = self.format_with_citations(docs)
        
        # Generate answer
        prompt = f"""
        You are a legal research assistant. CRITICAL RULES:
        1. Answer ONLY from provided documents
        2. Cite every claim: [Doc X, Page Y]
        3. If unsure, say "Insufficient information"
        4. Flag any contradictions between sources
        
        {context}
        
        Question: {query}
        """
        
        answer = llm(prompt, temperature=0)  # Deterministic
        
        # Verify factual consistency
        is_consistent = self.verifier.check(context, answer)
        
        if not is_consistent:
            return {
                "answer": "VERIFICATION FAILED",
                "explanation": "Generated answer conflicts with source documents",
                "sources": docs
            }
        
        # Log for audit
        self.log_query(query, docs, answer, user_context)
        
        return {
            "answer": answer,
            "sources": docs,
            "confidence": calculate_confidence(answer, docs)
        }
    
    def format_with_citations(self, docs):
        context = ""
        for i, doc in enumerate(docs, 1):
            context += f"""
[Document {i}]
Source: {doc.metadata.case_name}
Citation: {doc.metadata.citation}
Date: {doc.metadata.date}
Page: {doc.metadata.page}

Content:
{doc.text}

---
"""
        return context
```

**Special considerations:**

1. **Chunking**: Respect legal structure
   - Don't split statutes mid-section
   - Keep case holdings together
   - Chunk size: 1024 tokens (legal paragraphs are long)

2. **Metadata**: Rich metadata for filtering
   ```python
   metadata = {
       "case_name": "Smith v. Jones",
       "citation": "123 F.3d 456 (9th Cir. 2024)",
       "date": "2024-01-15",
       "jurisdiction": "9th Circuit",
       "topic": ["contract law", "damages"],
       "precedent_level": "binding",
       "page": 12
   }
   ```

3. **Quality assurance**:
   - Human-in-loop for high-stakes queries
   - Confidence thresholds (if <0.9, escalate to human)
   - Audit trail (all queries logged)

4. **Versioning**: Track document versions, handle precedent changes

**Cost estimate**:
- Hybrid search: $0.001/query
- Reranking: $0.002/query
- GPT-4 (for accuracy): $0.03/query
- **Total: ~$0.035/query** (acceptable for high-value use case)

---

### S2: Users report "RAG sometimes misses obvious information." Debug and fix.

**Answer:**

**Investigation process:**

**Step 1: Collect examples**
```python
failed_queries = [
    {"query": "What is our refund policy?", 
     "expected": "30-day refund",
     "retrieved": ["shipping policy", "terms of service"]},
    
    {"query": "GPU memory error troubleshooting",
     "expected": "CUDA OOM solutions",
     "retrieved": ["CPU errors", "disk space issues"]}
]
```

**Step 2: Test retrieval in isolation**
```python
for case in failed_queries:
    print(f"\nQuery: {case['query']}")
    
    docs = retrieve(case['query'], k=20)
    
    for i, doc in enumerate(docs):
        print(f"[{i}] Score: {doc.score:.3f}")
        print(f"Content: {doc.text[:100]}...")
    
    # Check: Is expected content in top 20?
    if case['expected'] not in [doc.text for doc in docs]:
        print("❌ Expected content NOT retrieved")
    else:
        rank = find_rank(docs, case['expected'])
        print(f"✅ Expected content at rank {rank}")
```

**Common root causes:**

**Problem 1: Chunking split the answer**
```python
# Document: "Our refund policy is simple. We offer 30-day refunds."
# Bad chunking: 
#   Chunk 1: "...policy is simple."
#   Chunk 2: "We offer 30-day..."
# Neither chunk has full context!

# Fix: Increase overlap
chunks = split_text(doc, chunk_size=512, overlap=100)  # 20% overlap
```

**Problem 2: Embedding model doesn't capture domain terminology**
```python
# Query: "GPU OOM" (Out Of Memory)
# Embedding doesn't understand "OOM" is same as "memory error"

# Fix 1: Query expansion
expanded = f"{query} OR {expand_acronyms(query)}"
# "GPU OOM OR GPU out of memory OR GPU memory error"

# Fix 2: Fine-tune embedding model on domain data
```

**Problem 3: Relevant doc has low semantic similarity**
```python
# Query: "refund policy"
# Doc: "Satisfaction guaranteed! Return within 30 days" (relevant but different words)

# Fix: Hybrid search
vector_results = vector_search(query, k=20)
keyword_results = bm25_search(["refund", "policy"], k=20)
merged = merge_results([vector_results, keyword_results])
```

**Problem 4: Metadata filtering too aggressive**
```python
# Query filtered to: date > 2024-01-01
# But relevant doc is from 2023

# Fix: Relax filters if no results
results = search(query, filter={"date": ">=2024-01-01"})

if len(results) < 5:
    # Relax filter
    results = search(query, filter={"date": ">=2023-01-01"})
```

**Step 3: Implement fixes and test**
```python
# Re-run failed queries
fixes = {
    "increase_overlap": True,
    "hybrid_search": True,
    "query_expansion": True
}

for case in failed_queries:
    results = retrieve_v2(case['query'], **fixes)
    
    if contains_expected(results, case['expected']):
        print(f"✅ Fixed: {case['query']}")
    else:
        print(f"❌ Still broken: {case['query']}")
```

**Prevention**: 
- Regression test suite (100+ queries)
- Monitor retrieval metrics weekly
- User feedback loop ("Was this helpful?")
