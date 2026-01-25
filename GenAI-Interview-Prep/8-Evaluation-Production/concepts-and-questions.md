# Evaluation & Production MLOps - Complete Guide

## Evaluation Metrics

### 1. RAG-Specific Metrics (RAGAS)

**Faithfulness** (0-1)
- Does answer match retrieved context?
- No hallucinations?

**Answer Relevance** (0-1)
- Does answer address the question?

**Context Relevance** (0-1)
- Are retrieved docs relevant to query?

**Context Recall** (0-1)
- Are all relevant docs retrieved?

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy

results = evaluate(
    dataset=test_dataset,
    metrics=[faithfulness, answer_relevancy]
)
# Output: {"faithfulness": 0.87, "answer_relevancy": 0.92}
```

### 2. LLM-as-Judge

```python
judge_prompt = """
Rate this response 1-5:
Question: {question}
Answer: {answer}

Criteria:
- Accuracy (factually correct?)
- Completeness (addresses all parts?)
- Clarity (easy to understand?)
- Conciseness (not too verbose?)

Output JSON: {{"accuracy": X, "completeness": Y, ...}}
"""

score = judge_llm(judge_prompt)
```

**Pros:** Scalable, correlates 85-90% with human eval
**Cons:** Biased toward certain styles, needs calibration

### 3. Production Metrics

**System metrics:**
- Latency (p50, p95, p99)
- Throughput (requests/sec)
- Error rate
- Cost per request

**Quality metrics:**
- User ratings (thumbs up/down)
- Task completion rate
- Conversation length (shorter = better for support)
- Escalation rate (to humans)

## Production Architecture Patterns

### 1. Model Serving

```python
# FastAPI serving
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Request(BaseModel):
    query: str
    context: Optional[List[str]]

@app.post("/generate")
async def generate(request: Request):
    # Retrieve context
    if not request.context:
        request.context = retrieve(request.query)
    
    # Generate
    response = llm.generate(
        query=request.query,
        context=request.context
    )
    
    # Log
    log_request(request, response, latency, cost)
    
    return response
```

### 2. Caching Strategy

**Semantic caching:**
```python
from sentence_transformers import SentenceTransformer

cache_model = SentenceTransformer('all-MiniLM-L6-v2')

def semantic_cache_lookup(query, threshold=0.95):
    query_emb = cache_model.encode(query)
    
    # Check cache
    for cached_query, cached_response in cache:
        similarity = cosine_sim(query_emb, cached_query.embedding)
        if similarity > threshold:
            return cached_response  # Cache hit
    
    return None  # Cache miss
```

**Hit rate:** 30-50% typical, saves 30-50% cost

### 3. A/B Testing

```python
class ABTest:
    def __init__(self, variant_weights):
        self.variants = variant_weights  # {"control": 0.8, "test": 0.2}
    
    def get_variant(self, user_id):
        # Consistent assignment
        hash_val = hash(user_id) % 100
        
        cumulative = 0
        for variant, weight in self.variants.items():
            cumulative += weight * 100
            if hash_val < cumulative:
                return variant
    
    def log_result(self, user_id, variant, metric, value):
        # Log to analytics
        log_event({
            "user_id": user_id,
            "variant": variant,
            "metric": metric,
            "value": value,
            "timestamp": now()
        })

# Usage
ab_test = ABTest({"control": 0.9, "test": 0.1})
variant = ab_test.get_variant(user_id)

if variant == "test":
    response = new_model(query)
else:
    response = current_model(query)

ab_test.log_result(user_id, variant, "satisfaction", user_rating)
```

### 4. Monitoring Dashboard

**Key metrics to track:**
```python
dashboard_metrics = {
    # Latency
    "p50_latency_ms": 850,
    "p95_latency_ms": 1500,
    "p99_latency_ms": 3000,
    
    # Quality
    "avg_user_rating": 4.2,
    "thumbs_up_rate": 0.78,
    "error_rate": 0.02,
    
    # Cost
    "cost_per_request": 0.015,
    "daily_cost": 450,
    
    # Volume
    "requests_per_second": 50,
    "daily_requests": 4_320_000,
    
    # Model
    "avg_tokens_per_request": 1200,
    "avg_retrieval_docs": 7
}
```

## Interview Questions

### Q1: How do you evaluate a RAG system without ground truth?

**Answer:**

**1. LLM-as-judge** (most practical)
```python
# Judge answer quality
judge_score = judge_llm(question, answer, context)
```

**2. User feedback**
```python
# Implicit: click, copy, follow-up
# Explicit: thumbs up/down, ratings
```

**3. Component-level metrics**
```python
# Retrieval: Are relevant docs in top-K?
retrieval_precision = relevant_in_topK / K

# Generation: Is output faithful to context?
faithfulness_score = nli_model(context, answer)
```

**4. Proxy metrics**
```python
# Answer length (too short = bad)
# Citation presence
# Keyword overlap with context
```

**Production approach:** Combine all 4, weight by reliability.

---

### Q2: Your model works in staging but fails in production. Why?

**Common causes:**

**1. Data distribution shift**
- Staging: Clean test queries
- Production: Typos, abbreviations, multi-lingual

**2. Load issues**
- Staging: 1 req/sec
- Production: 100 req/sec → rate limits, slowdowns

**3. Context differences**
- Staging: Perfect retrieval
- Production: Sometimes retrieves irrelevant docs

**4. User behavior**
- Staging: Single queries
- Production: Long conversations (context overflow)

**5. Edge cases**
- Staging: Happy path only
- Production: Malformed inputs, prompt injection, extreme lengths

**Solution:**
```python
# 1. Shadow mode: Run new model alongside old
# 2. Compare outputs, log differences
# 3. Gradually increase traffic (1% → 10% → 100%)
# 4. Monitor metrics at each stage
```

---

### Q3: Design a monitoring system for GenAI in production.

**Architecture:**

```python
class GenAIMonitoring:
    def __init__(self):
        self.metrics_db = TimeSeriesDB()
        self.alert_system = AlertSystem()
    
    def log_request(self, request, response, metadata):
        # Core metrics
        self.metrics_db.write({
            "timestamp": now(),
            "user_id": request.user_id,
            "latency_ms": metadata.latency,
            "tokens_used": metadata.tokens,
            "cost": metadata.cost,
            "model": metadata.model,
            "error": metadata.error,
            "rating": response.user_rating  # if available
        })
        
        # Quality checks
        if response.user_rating < 3:
            self.investigate_bad_response(request, response)
        
        if metadata.latency > 3000:  # >3s
            self.alert_system.send("High latency", priority="medium")
        
        if metadata.error:
            self.alert_system.send("Error", priority="high")
    
    def aggregate_metrics(self, window="1h"):
        return {
            "avg_latency": self.metrics_db.avg("latency_ms", window),
            "p95_latency": self.metrics_db.percentile("latency_ms", 95, window),
            "error_rate": self.metrics_db.count("error") / self.metrics_db.count("*"),
            "avg_cost": self.metrics_db.avg("cost", window),
            "total_requests": self.metrics_db.count("*", window)
        }
```

**Alerts:**
- Error rate >5%: Critical
- Latency p95 >2s: Warning
- Cost spike >50%: Warning
- Low ratings (<4.0): Warning

---

### Q4: How do you version and deploy GenAI models?

**Strategy:**

**1. Model versioning**
```python
models = {
    "v1.0": {"llm": "gpt-3.5", "prompt": "prompt_v1.txt"},
    "v1.1": {"llm": "gpt-3.5", "prompt": "prompt_v1.1.txt"},
    "v2.0": {"llm": "gpt-4", "prompt": "prompt_v2.txt"}
}

def get_model(user_id):
    # Route based on experiment
    if user_id in beta_users:
        return models["v2.0"]
    else:
        return models["v1.1"]
```

**2. Deployment process**
```
1. Dev testing (eval set)
2. Staging deployment (shadow mode)
3. Canary (5% production traffic)
4. Gradual rollout (25% → 50% → 100%)
5. Monitor at each stage
6. Rollback if metrics degrade
```

**3. Rollback plan**
```python
if metrics["error_rate"] > 0.1:
    rollback_to_previous_version()
    alert_team("Automatic rollback triggered")
```

---

### Q5: Cost per request is $0.05 but budget is $0.01. Optimize.

**Analysis:**
```
Current: GPT-4 (1500 tokens avg)
- Input: 1000 tokens × $0.03/1K = $0.03
- Output: 500 tokens × $0.06/1K = $0.03
- Total: $0.06/request
```

**Optimizations:**

**1. Model swap** (biggest impact)
```
GPT-4 → GPT-3.5
Cost: $0.06 → $0.006 (10× cheaper)
Quality: -10 to -20% (test on your data)
```

**2. Prompt optimization**
```
Remove verbose instructions
Before: 500 token system prompt
After: 200 token system prompt
Savings: 20%
```

**3. Caching**
```
Cache hit rate: 40%
Effective cost: $0.06 × 0.6 = $0.036
```

**4. Response length limits**
```
max_tokens=300 (instead of unlimited)
Average output: 500 → 300 tokens
Savings: 40% on output cost
```

**5. Tiered routing**
```python
if query_complexity == "simple":
    use gpt_3.5  # 90% of queries
else:
    use gpt_4  # 10% of queries

# Blended cost: $0.006 × 0.9 + $0.06 × 0.1 = $0.012
```

**Result: $0.06 → $0.01** ✅

---

## Production Checklist

**Pre-launch:**
- [ ] Load testing (1000+ req/sec)
- [ ] Error handling & retries
- [ ] Rate limiting
- [ ] Caching layer
- [ ] Monitoring & alerts
- [ ] Rollback plan
- [ ] Cost estimation
- [ ] Security review (PII, prompt injection)

**Post-launch:**
- [ ] Daily metrics review
- [ ] User feedback analysis
- [ ] Cost tracking
- [ ] A/B test new versions
- [ ] Quarterly evaluation with updated test sets

**Incident response:**
1. Detect (monitoring alerts)
2. Triage (classify severity)
3. Mitigate (rollback or hotfix)
4. Root cause analysis
5. Prevention measures
