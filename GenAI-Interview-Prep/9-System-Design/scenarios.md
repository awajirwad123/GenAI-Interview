# System Design Scenarios - GenAI Applications

## Scenario 1: Design a RAG-based Customer Support Chatbot

**Requirements:**
- 100K documents (product docs, FAQs, policies)
- 10K concurrent users
- <2s response time
- Multi-lingual (English, Spanish, French)
- Cost: <$10K/month

**Solution:**

### Architecture
```
User → API Gateway → Load Balancer
            ↓
    Intent Classifier (7B model, 50ms)
            ↓
    ┌───────┴───────┐
    │               │
Simple Q&A     Complex Issues
(GPT-3.5)      (GPT-4 + Human)
    │               │
    └───────┬───────┘
            ↓
    Vector DB (Pinecone/Weaviate)
    ├── 100K docs → 1M chunks
    ├── Hybrid search
    └── Reranking (top 50 → 10)
            ↓
    Response Generation
            ↓
    Content Filter + PII Redaction
            ↓
    Cache Layer (Redis)
            ↓
    User
```

### Key Design Decisions

**1. Tiered routing**
```python
# 80% simple queries → GPT-3.5 (cheap)
# 15% complex → GPT-4
# 5% escalate → Human agent

if intent == "greeting" or intent == "simple_faq":
    model = "gpt-3.5-turbo"  # $0.002/request
elif intent == "complex_troubleshooting":
    model = "gpt-4"  # $0.03/request
else:
    escalate_to_human()
```

**2. Multi-lingual strategy**
```python
# Translate query to English (unified knowledge base)
if detected_language != "en":
    query_en = translate(query, target="en")
else:
    query_en = query

# Search in English
docs = retrieve(query_en, k=10)

# Generate answer in English
answer_en = llm(query_en, docs)

# Translate back
if detected_language != "en":
    answer = translate(answer_en, target=detected_language)
```

**3. Caching**
```python
# Semantic cache (40% hit rate)
cache_key = embed(query)
if similar_query := cache.find_similar(cache_key, threshold=0.95):
    return cache[similar_query]  # Instant, free
```

**4. Response time breakdown**
```
Intent classification: 50ms (small model)
Retrieval: 80ms (vector search + rerank)
LLM inference: 800ms (GPT-3.5) or 1500ms (GPT-4)
Translation: 100ms (if needed)
Total: ~1000ms (within budget)
```

### Cost Analysis
```
10K concurrent users, 10 requests/day = 100K requests/day = 3M/month

Breakdown:
- 80% GPT-3.5: 2.4M × $0.002 = $4.8K
- 15% GPT-4: 450K × $0.03 = $13.5K
- 5% human: 150K × $0 (internal cost)

With caching (40% hit rate):
- Effective: ($4.8K + $13.5K) × 0.6 = $11K

Optimization needed: More aggressive GPT-3.5 routing
- 90% GPT-3.5, 10% GPT-4 → $7.5K/month ✅
```

### Scaling Considerations
- Vector DB: Pinecone $140/month for 1M vectors
- Redis cache: $50/month
- Translation API: $0.5K/month
- Total infrastructure: ~$8K/month

### Failure Modes & Mitigations
1. **Rate limits**: Multiple LLM deployments, queue requests
2. **Vector DB down**: Fallback to keyword search
3. **LLM quality issues**: Human-in-loop for low confidence
4. **Spike in traffic**: Auto-scale API servers, rate limiting per user

---

## Scenario 2: Build a Code Generation Assistant (like GitHub Copilot)

**Requirements:**
- Real-time code completion (<100ms)
- Context-aware (understand codebase)
- Support 10+ languages
- IDE integration (VS Code, PyCharm)
- Privacy: Code never leaves company

**Solution:**

### Architecture
```
IDE → Language Server Protocol
        ↓
    Local Model (Llama-2 7B quantized)
        ↓
    Context Gathering:
    ├── Current file (last 1000 tokens)
    ├── Imports (retrieve relevant files)
    ├── Function signatures
    └── Comments/docstrings
        ↓
    Completion Generation (FIM - Fill In Middle)
        ↓
    Post-processing:
    ├── Syntax validation
    ├── Security checks
    └── Filter sensitive patterns
        ↓
    IDE (show suggestion)
```

### Key Design Decisions

**1. Local vs API model**
```
Local (Llama-2 7B):
- Latency: 50-100ms ✅
- Privacy: Code stays local ✅
- Cost: One-time $2K GPU/dev
- Quality: Good but not SOTA

API (GPT-4):
- Latency: 500-1000ms ❌
- Privacy: Code leaves network ❌
- Cost: $100-500/dev/month
- Quality: Best

Decision: Local model for privacy + latency
```

**2. Context selection**
```python
def gather_context(cursor_position, codebase):
    context = []
    
    # 1. Current file (before cursor)
    context.append(current_file[:cursor_position])
    
    # 2. Imported modules
    imports = extract_imports(current_file)
    for imp in imports:
        context.append(get_file_summary(imp))
    
    # 3. Similar code (semantic search)
    similar = vector_search(current_file, k=3)
    context.extend(similar)
    
    # Keep under 2K tokens
    return truncate(context, max_tokens=2000)
```

**3. Fill-In-Middle (FIM) prompting**
```python
# Model trained with special tokens
prompt = f"""
<fim_prefix>{code_before_cursor}<fim_suffix>{code_after_cursor}<fim_middle>
"""

# Model generates: missing code
completion = model.generate(prompt, max_tokens=50)
```

**4. Multi-line vs single-line**
```python
if cursor_at_function_start():
    # Generate full function body
    max_tokens = 200
    temperature = 0.7  # More creative
else:
    # Single line completion
    max_tokens = 30
    temperature = 0.2  # More deterministic
```

### Optimization for <100ms Latency

**1. Model quantization**
```
FP16: 14GB VRAM, 100ms
INT8: 7GB VRAM, 70ms
INT4: 4GB VRAM, 50ms

Choice: INT8 (good quality/speed tradeoff)
```

**2. Speculative decoding**
```
Small model (1B): Draft 10 tokens in 20ms
Large model (7B): Verify + generate in 60ms
Total: 80ms (vs 100ms without)
```

**3. KV-cache**
```
Cache context embeddings
New tokens: Only compute diff
Speedup: 30-40%
```

**4. Batching (multi-user)**
```
Batch 4 requests: 80ms per request
Individual: 50ms per request

Trade-off: Slight increase per request, but 4× throughput
```

### Privacy & Security

**1. Data isolation**
- Each dev's model runs locally
- No code sent to central server
- Telemetry: Only anonymized metrics

**2. Sensitive pattern filtering**
```python
def filter_completion(code):
    patterns = [
        r"password\s*=\s*['\"].*['\"]",  # Hardcoded passwords
        r"api[_-]?key\s*=\s*['\"].*['\"]",  # API keys
        r"\d{3}-\d{2}-\d{4}",  # SSNs
    ]
    
    for pattern in patterns:
        if re.search(pattern, code):
            return "[REDACTED]"
    
    return code
```

### Cost Analysis
```
Per developer:
- GPU (one-time): $2000 (RTX 4090)
- Or cloud GPU: $200/month (A10)

For 100 developers:
- Upfront: $200K (amortize over 2 years = $8.3K/month)
- Or cloud: $20K/month

vs GitHub Copilot: $10/dev/month = $1K/month

Break-even: 100 devs, if privacy is required
```

---

## Scenario 3: Build a Document Intelligence Platform

**Requirements:**
- Process 10K documents/day (PDFs, contracts, invoices)
- Extract structured data (dates, amounts, parties)
- Answer questions about documents
- Audit trail (who accessed what)
- SLA: 99.9% uptime

**Solution:**

### Architecture
```
Document Upload → Storage (S3/Blob)
        ↓
    OCR (if scanned) - Tesseract/Azure Vision
        ↓
    Document Processing Pipeline:
    ├── Text extraction
    ├── Layout analysis (tables, sections)
    ├── Entity extraction (NER)
    └── Chunking (semantic)
        ↓
    Vector DB + SQL DB
    ├── Vector: Semantic search
    └── SQL: Structured metadata
        ↓
    Query Interface:
    ├── Structured queries (SQL)
    ├── Semantic search (Vector)
    └── LLM Q&A
        ↓
    Audit Log → Analytics
```

### Key Components

**1. Document processing**
```python
def process_document(doc_path):
    # Extract text
    if doc_path.endswith('.pdf'):
        text = extract_pdf(doc_path)
    
    # OCR if needed
    if is_scanned(text):
        text = ocr(doc_path)
    
    # Extract entities
    entities = ner_model.extract(text)  # Dates, amounts, names
    
    # Chunk for RAG
    chunks = chunk_document(text, size=512, overlap=50)
    
    # Generate embeddings
    embeddings = embed_batch(chunks)
    
    # Store
    vector_db.insert(chunks, embeddings, metadata={
        "doc_id": doc_id,
        "entities": entities,
        "upload_date": now()
    })
    
    sql_db.insert({
        "doc_id": doc_id,
        "title": extract_title(text),
        "entities": entities,
        "page_count": count_pages(doc_path)
    })
```

**2. Hybrid query system**
```python
def query(question, filters=None):
    # Classify query type
    if is_structured_query(question):
        # SQL query
        # "How many contracts signed in 2024?"
        sql = llm_to_sql(question)
        return sql_db.query(sql)
    
    else:
        # Semantic search
        # "What are the payment terms?"
        results = vector_db.search(
            embed(question),
            k=10,
            filter=filters  # department, date range, etc.
        )
        
        # LLM synthesis
        answer = llm.generate(question, results)
        
        return {
            "answer": answer,
            "sources": results,
            "confidence": calculate_confidence(answer, results)
        }
```

**3. Audit logging**
```python
def audit_log(user_id, doc_id, action):
    log_db.insert({
        "timestamp": now(),
        "user_id": user_id,
        "doc_id": doc_id,
        "action": action,  # view, download, query
        "ip_address": get_ip(),
    })

# Compliance reports
def get_access_report(doc_id):
    return log_db.query(f"SELECT * FROM logs WHERE doc_id={doc_id}")
```

### Scaling to 10K docs/day

**Processing pipeline:**
```
10K docs/day = 416 docs/hour = 7 docs/minute

Per document:
- OCR: 30s (if scanned)
- NER: 5s
- Chunking + embedding: 10s
- Total: 45s

Parallelism needed: 7 docs/min × 45s = ~6 workers
```

**Cost:**
```
- OCR (Azure Vision): $1.50/1000 pages
  - 10K docs × 10 pages × $1.50/1000 = $150/day = $4.5K/month
  
- Embeddings: 10K docs × 20 chunks × $0.0001/1K = $20/day = $600/month

- Storage: 10K × 30 days × 1MB = 300GB = $7/month (S3)

- Vector DB: 600K vectors = ~$100/month

Total: ~$5.2K/month
```

### High Availability

**1. Redundancy**
```
- Multi-region deployment
- Load balancer with health checks
- Database replication
```

**2. Failure handling**
```python
def resilient_processing(doc):
    max_retries = 3
    
    for attempt in range(max_retries):
        try:
            return process_document(doc)
        except Exception as e:
            log_error(e)
            if attempt == max_retries - 1:
                # Move to dead-letter queue
                dlq.push(doc)
                notify_ops_team()
            else:
                time.sleep(2 ** attempt)  # Exponential backoff
```

**3. Monitoring**
```python
metrics = {
    "processing_queue_depth": 42,
    "avg_processing_time_s": 38,
    "error_rate": 0.02,
    "uptime": 0.999,  # 99.9%
    "ocr_accuracy": 0.97
}

# Alerts
if metrics["queue_depth"] > 1000:
    alert("Processing backlog")

if metrics["error_rate"] > 0.05:
    alert("High error rate")
```

---

## General System Design Tips

**Always discuss:**
1. **Tradeoffs**: Cost vs quality, latency vs accuracy
2. **Scale**: Numbers matter (requests/sec, data size)
3. **Failure modes**: What breaks and how to handle
4. **Cost**: Estimate operational costs
5. **Monitoring**: How to know if it's working
6. **Iteration**: Start simple, scale up

**Common mistakes:**
- Over-engineering initially
- Ignoring cost implications
- Not considering failure scenarios
- Forgetting about data privacy/security
- No monitoring plan

**Interview structure:**
1. Clarify requirements (5 min)
2. High-level architecture (5 min)
3. Deep dive on 2-3 components (10 min)
4. Scaling & tradeoffs (5 min)
5. Q&A (5 min)
