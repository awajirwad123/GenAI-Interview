# GenAI Interview Quick Reference

> **Last-minute cheat sheet** - Review this 1 hour before the interview

---

## 1. Key Numbers to Remember

### Model Sizes
- **GPT-3.5**: 175B params, 4K context (old) / 16K (new)
- **GPT-4**: ~1.7T params (estimated), 8K / 32K / 128K context
- **Llama-2**: 7B/13B/70B variants
- **Llama-3**: 8B/70B/405B variants

### Latency Targets
- **Real-time chat**: <500ms
- **Batch processing**: <30s
- **Search**: <100ms (vector), <200ms (hybrid)
- **Code completion**: <100ms

### Cost Benchmarks (per 1K tokens)
- **GPT-3.5-turbo**: $0.0015 (input), $0.002 (output)
- **GPT-4**: $0.03 (input), $0.06 (output)
- **GPT-4o**: $0.005 (input), $0.015 (output)
- **Claude-3.5-Sonnet**: $0.003 (input), $0.015 (output)

### Embedding Dimensions
- **OpenAI ada-002**: 1536 dims
- **OpenAI text-embedding-3**: 256/1024/3072 dims (configurable)
- **Sentence-Transformers**: 384/768 dims

---

## 2. Common Architectures (Draw These)

### Basic RAG
```
Query → Embedding → Vector Search → Top-K Docs → LLM → Answer
```

### Advanced RAG
```
Query → Query Expansion
         ↓
     Hybrid Search (Vector + Keyword)
         ↓
     Reranking (top 50 → 10)
         ↓
     LLM with CoT
         ↓
     Answer + Citations
```

### ReAct Agent
```
Thought: I need to search for X
Action: search("X")
Observation: Results...
Thought: I should calculate Y
Action: calculator(Y)
Observation: Result...
Thought: I can now answer
Final Answer: ...
```

---

## 3. Must-Know Concepts

### Fine-tuning Methods
| Method | Params | Use Case | Cost |
|--------|--------|----------|------|
| **Full** | 100% | Domain shift | $$$$ |
| **LoRA** | 0.1-1% | Task-specific | $$ |
| **QLoRA** | 0.1-1% | Budget-constrained | $ |
| **Prompt tuning** | <0.01% | Quick adaptation | $ |

### Chunking Strategies
1. **Fixed-size**: 512 tokens, 50 overlap
2. **Sentence-based**: Natural boundaries
3. **Semantic**: Break on topic shifts
4. **Recursive**: Hierarchical (best for code)

### Retrieval Methods
- **Dense (vector)**: Semantic similarity, 70-80% recall
- **Sparse (BM25)**: Exact match, 50-60% recall
- **Hybrid**: Combine both, 85-90% recall

---

## 4. Common Interview Questions (Quick Answers)

**Q: Why not just increase context window?**
- Cost scales quadratically with context
- Quality degrades (lost in the middle)
- Latency increases
- RAG is more cost-effective for large knowledge bases

**Q: GPT-4 vs GPT-3.5 - when to use which?**
- **GPT-3.5**: Simple tasks, high volume, cost-sensitive
- **GPT-4**: Complex reasoning, high accuracy needed, low volume

**Q: How to reduce hallucinations?**
1. RAG with citations
2. Temperature = 0
3. Prompt: "Only use provided context"
4. Confidence scoring
5. Human-in-loop for critical outputs

**Q: Vector DB selection criteria?**
- **Pinecone**: Managed, easy, $$$
- **Weaviate**: Hybrid search, open-source
- **Qdrant**: Fast, quantization support
- **pgvector**: If already using PostgreSQL

**Q: How to handle rate limits?**
1. Multiple deployments (round-robin)
2. Exponential backoff
3. Queue + retry logic
4. Caching (40-60% hit rate typical)

---

## 5. Troubleshooting Playbook

### RAG returning wrong answers
1. Check retrieval quality (are right docs retrieved?)
2. Inspect chunk sizes (too small = no context, too large = noise)
3. Try hybrid search instead of pure vector
4. Add reranking step
5. Check embedding model (multilingual support?)

### Agent stuck in loops
1. Add max iterations limit (e.g., 10)
2. Check tool descriptions (ambiguous?)
3. Implement loop detection (same action 3× = break)
4. Add reflection step ("Did this work?")

### High latency
1. Profile: Retrieval vs LLM vs processing
2. Reduce context (fewer chunks, smaller chunks)
3. Use faster model (GPT-4 → GPT-3.5)
4. Enable streaming
5. Implement caching

### Cost explosion
1. Check prompt length (input tokens)
2. Audit agent iterations (is it 10× expected?)
3. Implement caching
4. Use cheaper models where possible
5. Rate limit per user

---

## 6. Code Snippets (Copy-Paste Ready)

### Basic RAG
```python
from openai import OpenAI
import numpy as np

client = OpenAI()

# 1. Embed query
query_emb = client.embeddings.create(
    input="What is X?",
    model="text-embedding-3-small"
).data[0].embedding

# 2. Search vector DB
results = vector_db.search(query_emb, k=5)

# 3. Generate answer
context = "\n".join([r.text for r in results])
response = client.chat.completions.create(
    model="gpt-4",
    messages=[
        {"role": "system", "content": "Answer using only the context."},
        {"role": "user", "content": f"Context:\n{context}\n\nQuestion: What is X?"}
    ]
)
```

### ReAct Agent
```python
def react_agent(question, max_iterations=10):
    messages = [{"role": "user", "content": question}]
    
    for i in range(max_iterations):
        response = client.chat.completions.create(
            model="gpt-4",
            messages=messages,
            tools=[{"type": "function", "function": {...}}]
        )
        
        if response.choices[0].finish_reason == "tool_calls":
            # Execute tool
            tool_call = response.choices[0].message.tool_calls[0]
            result = execute_tool(tool_call)
            
            # Add to history
            messages.append(response.choices[0].message)
            messages.append({"role": "tool", "content": result})
        else:
            return response.choices[0].message.content
```

### Hybrid Search
```python
def hybrid_search(query, alpha=0.7):
    # Vector search
    vector_results = vector_db.search(embed(query), k=20)
    
    # Keyword search (BM25)
    keyword_results = bm25_search(query, k=20)
    
    # Combine scores
    combined = {}
    for doc, score in vector_results:
        combined[doc.id] = alpha * score
    
    for doc, score in keyword_results:
        combined[doc.id] = combined.get(doc.id, 0) + (1-alpha) * score
    
    # Return top K
    return sorted(combined.items(), key=lambda x: x[1], reverse=True)[:10]
```

---

## 7. Architecture Decision Tree

```
Need to build GenAI app?
│
├─ Simple Q&A, no external data?
│  └─ Direct LLM call (GPT-4/Claude)
│
├─ Need company knowledge?
│  └─ RAG system
│     ├─ <100K docs → Vector DB (Pinecone/Qdrant)
│     └─ >100K docs → Hybrid search + SQL metadata
│
├─ Multi-step reasoning?
│  └─ Agent (ReAct/Plan-Execute)
│     ├─ Simple tools → LangChain
│     └─ Complex workflows → LangGraph
│
├─ High throughput (>1000 QPS)?
│  └─ Smaller model (Llama-2 70B) + caching
│
└─ Strict privacy?
   └─ Self-hosted model (Llama-3)
```

---

## 8. Red Flags to Avoid

❌ "I'd use the latest GPT model for everything"
✅ "I'd start with GPT-3.5 for simple tasks and GPT-4 for complex ones"

❌ "Just use embeddings for all search"
✅ "Hybrid search combining vector + keyword performs better"

❌ "Fine-tune the model on our data"
✅ "Try RAG first, fine-tune only if necessary"

❌ "Store entire documents in vector DB"
✅ "Chunk documents into 512-token segments"

❌ "No need for monitoring, LLMs are deterministic"
✅ "Monitor latency, cost, quality metrics, and log failures"

---

## 9. Azure OpenAI Specifics

### Key Differences
- **Deployment-based**: Create deployments, not direct model calls
- **Quotas**: TPM (tokens/min), RPM (requests/min)
- **Regions**: GPT-4 not available everywhere
- **Content filters**: Enabled by default (can adjust)

### Common Setup
```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key=os.getenv("AZURE_OPENAI_KEY"),
    api_version="2024-02-01",
    azure_endpoint="https://YOUR-RESOURCE.openai.azure.com"
)

response = client.chat.completions.create(
    model="gpt-4",  # Your deployment name
    messages=[...]
)
```

### Handle 429 (Rate Limit)
```python
import time

def call_with_retry(func, max_retries=3):
    for i in range(max_retries):
        try:
            return func()
        except RateLimitError:
            time.sleep(2 ** i)  # Exponential backoff
    raise Exception("Max retries exceeded")
```

---

## 10. Sample System Design Answer (30-second version)

**Q: Design a customer support chatbot for 10K users**

"I'd build a **tiered RAG system**:

1. **Intent classifier** (small model, 50ms) routes queries:
   - Simple FAQs → GPT-3.5 ($0.002/request)
   - Complex issues → GPT-4 ($0.03/request)

2. **Vector DB** (Pinecone) with 100K documents chunked into 1M segments, **hybrid search** for 85% retrieval quality

3. **Semantic cache** (Redis) for 40% hit rate (free + instant)

4. **Response time**: Intent (50ms) + Retrieval (80ms) + LLM (800ms) = ~1s

5. **Cost**: 100K requests/day = 3M/month
   - 80% GPT-3.5 = $4.8K
   - 20% GPT-4 = $18K
   - With cache: $14K/month

6. **Scaling**: Load balancer → API servers → Vector DB (sharded)

7. **Monitoring**: Latency, cost per request, answer quality (thumbs up/down), escalation rate"

---

## 11. Before the Interview

✅ **Print this guide**
✅ **Review your recent projects** (be ready to discuss architecture decisions)
✅ **Practice drawing architectures** (whiteboard or paper)
✅ **Know your resume** (every LLM/GenAI project you listed)
✅ **Prepare 3 questions** to ask the interviewer

### Common Follow-ups
- "How did you evaluate success?"
- "What was the biggest challenge?"
- "How did you handle production issues?"
- "What would you do differently?"

### Your Questions to Ask
1. "What GenAI use cases are you currently working on?"
2. "What's your LLM evaluation strategy?"
3. "How do you balance innovation with production stability?"

---

**Good luck! You've got this! 🚀**
