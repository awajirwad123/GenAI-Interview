# LLM Fundamentals - Interview Questions

## High-Probability Questions

### Q1: Explain the self-attention mechanism. Why is it better than RNNs?

**Answer:**
Self-attention computes relationships between all tokens in parallel by calculating Query, Key, Value matrices. Each token attends to every other token with learned weights.

**Advantages over RNNs:**
- Parallelizable (no sequential dependency)
- Captures long-range dependencies (no gradient vanishing)
- Constant path length between any two positions

**Formula**: `Attention(Q,K,V) = softmax(QK^T/√d_k)V`

**Follow-up ready**: "Attention is O(n²) in sequence length. For long documents, we use sparse attention or sliding windows."

---

### Q2: You have a 70B model but need sub-200ms latency. What do you do?

**Answer:**
1. **Quantize to INT8/INT4** - 4x smaller, minimal quality loss
2. **Use speculative decoding** - Draft with 7B, verify with 70B
3. **Batch requests** - Amortize KV-cache overhead
4. **Consider distillation** - Train 7B to mimic 70B (weeks of work)
5. **Cache common responses** - Semantic caching with embeddings

**Tradeoff**: May need to switch to 7B-13B model and invest in fine-tuning.

**Follow-up**: "At what request volume does batching help?" → "Depends on GPU memory, but typically >10 requests/sec makes batching worthwhile."

---

### Q3: When would you fine-tune vs use few-shot prompting?

**Answer:**

**Fine-tune when:**
- Consistent formatting needed (JSON, SQL)
- Domain terminology not in pre-training data
- Cost matters (cheaper inference per request)
- Privacy constraints (can't send data to API)

**Few-shot prompting when:**
- Limited labeled data (<100 examples)
- Task changes frequently
- Need quick iteration
- Multiple tasks on same model

**Real-world**: Start with prompting + RAG. Fine-tune only after validation.

**Follow-up**: "How much data for fine-tuning?" → "Minimum 100 examples, ideally 1K-10K. Quality > quantity."

---

### Q4: Explain the difference between LoRA and full fine-tuning.

**Answer:**
LoRA freezes base model and trains low-rank decomposition matrices (ΔW = BA, where B and A are much smaller).

**Key differences:**
- **Parameters**: LoRA trains ~0.1% of total params
- **Memory**: 3-10x less GPU memory needed
- **Speed**: 3x faster training
- **Swappable**: Can load different LoRA adapters on same base model

**Use case**: We used LoRA to create customer-specific adapters while sharing base infrastructure.

**Follow-up**: "What rank do you use?" → "Typically r=8 to r=32. Higher rank = more capacity but slower."

---

### Q5: What causes hallucinations and how do you mitigate them?

**Answer:**

**Root causes:**
1. Training data inconsistencies
2. Overconfident probability distributions
3. No grounding in factual sources
4. Complex queries beyond training distribution

**Mitigations:**
1. **RAG**: Ground responses in retrieved documents
2. **Citations**: Force model to quote sources
3. **Confidence scores**: Use logprobs to detect uncertainty
4. **Structured output**: Constrain to known schema
5. **Verification**: Second LLM validates first's output
6. **Human-in-loop**: Flag low-confidence for review

**Production metric**: Track citation accuracy and factual consistency (using NLI models).

---

### Q6: How do you handle a 200-page document when context limit is 32K tokens?

**Answer:**

**Strategies:**
1. **Chunking + RAG**
   - Split into semantic chunks (512-1024 tokens)
   - Embed and retrieve top-K relevant chunks
   - Most common production approach

2. **Hierarchical summarization**
   - Summarize sections, then summarize summaries
   - Good for "overall insights" questions

3. **Map-reduce pattern**
   - Process chunks in parallel, aggregate results
   - Use for extraction tasks (find all dates, entities)

4. **Sliding window**
   - Overlap chunks by 20% to avoid breaking context
   - Combine results with voting/consensus

**Real choice**: RAG for Q&A, summarization for synthesis, map-reduce for extraction.

**Follow-up**: "What chunk size?" → "512-1024 tokens balances context vs granularity. Test with your queries."

---

### Q7: Explain RLHF. Why is it important?

**Answer:**

**RLHF (Reinforcement Learning from Human Feedback):**
1. **Train reward model** on human preference pairs ("response A > response B")
2. **Optimize LLM** using PPO to maximize reward
3. Result: Model aligns with human values (helpful, harmless, honest)

**Why it matters:**
- Pre-training learns language patterns, not helpfulness
- Instruction tuning helps but doesn't capture subjective preferences
- RLHF makes models feel "natural" and safe

**Production challenge**: PPO is unstable. Industry moving to DPO (Direct Preference Optimization) - same human data, simpler training.

**Follow-up**: "What's wrong with PPO?" → "Requires careful hyperparameter tuning, reward hacking, training instability."

---

### Q8: You see inference latency increasing over time. What could be wrong?

**Answer:**

**Systematic debugging:**

1. **Context length growth**
   - Conversation history accumulating
   - Fix: Summarize or truncate older messages

2. **KV-cache memory pressure**
   - Cache eviction causing recomputation
   - Fix: Increase GPU memory or limit concurrent requests

3. **Thermal throttling**
   - GPUs overheating under sustained load
   - Fix: Better cooling, reduce batch size

4. **Network latency**
   - API provider issues
   - Fix: Monitor p95/p99 latencies, use fallback provider

5. **Cold start issues**
   - Model unloading between requests
   - Fix: Keep-alive requests, reserved instances

**Interview tip**: Show you think about monitoring and debugging, not just initial setup.

---

## Deep-Dive Follow-Up Questions

### Q9: Explain the difference between greedy, beam search, and nucleus sampling.

**Answer:**

| Method | How it works | Use case |
|--------|-------------|----------|
| **Greedy** | Always pick highest probability token | Deterministic, factual Q&A |
| **Beam search** | Maintain top-K sequences | Translation, summaries |
| **Nucleus (top-p)** | Sample from top tokens summing to p% | Creative text, conversation |

**Production settings:**
- Factual tasks: temperature=0, greedy
- Creative tasks: temperature=0.7-0.9, top_p=0.9
- Code generation: temperature=0.2, top_p=0.95

**Follow-up**: "What's temperature?" → "Scales logits before softmax. Higher = more random. 0 = deterministic."

---

### Q10: How do you evaluate if a fine-tuned model is better than the base model?

**Answer:**

**Quantitative metrics:**
1. **Perplexity**: Lower is better (measures prediction confidence)
2. **Task-specific**: BLEU (translation), ROUGE (summarization), Exact Match (Q&A)
3. **Human eval**: Side-by-side comparison (gold standard)

**Production reality:**
- A/B test with 5-10% traffic
- Track downstream metrics (user engagement, task completion)
- Monitor edge cases (where fine-tuning might hurt)

**Red flags:**
- Fine-tuned model worse on general knowledge (catastrophic forgetting)
- Overfitting to training distribution
- Latency/cost regression

**Follow-up**: "Model seems overfit. What do you do?" → "Add regularization (dropout), mix pre-training data, reduce training steps, use LoRA instead of full fine-tuning."

---

### Q11: Scenario: Your model works great in testing but fails in production. Why?

**Answer:**

**Common causes:**

1. **Distribution shift**
   - Test data doesn't match real user queries
   - Fix: Continuously collect production data for evaluation

2. **Prompt injection**
   - Users manipulating system prompt
   - Fix: Input sanitization, prompt sandboxing

3. **Context length issues**
   - Real conversations longer than test cases
   - Fix: Implement context management early

4. **Rate limiting / timeouts**
   - Production load higher than anticipated
   - Fix: Implement retries, fallbacks, circuit breakers

5. **Edge cases**
   - Rare languages, special characters, very long inputs
   - Fix: Comprehensive integration tests

**Interview insight**: Shows you understand production ≠ development.

---

### Q12: You need to reduce inference costs by 50%. What levers do you pull?

**Answer:**

**Cost reduction strategies:**

1. **Smaller model** (biggest impact)
   - GPT-4 → GPT-3.5 or Llama-2-13B (10x cheaper)
   - Fine-tune smaller model to match quality

2. **Prompt optimization**
   - Remove unnecessary examples
   - Shorten system prompts
   - Each 1K tokens saved = $0.01-0.03

3. **Caching**
   - Cache common query responses (40-60% hit rate typical)
   - Semantic caching for similar queries

4. **Request batching**
   - Batch similar requests
   - Reduces per-request overhead

5. **Output length limits**
   - Output tokens cost 2-3x input tokens
   - Set max_tokens conservatively

6. **Self-hosting**
   - If >1M requests/month, self-hosting on GPU cheaper
   - Upfront: $10K-50K, ongoing: $1-5K/month

**Real numbers**: Switching GPT-4 → GPT-3.5 typically saves 80% cost with 10-20% quality drop.

---

## Scenario-Based Questions

### S1: Design the model selection strategy for a customer support chatbot serving 1M users.

**Answer:**

**Tier-based routing:**

1. **Tier 1: Intent classification** (small model, <50ms)
   - 7B model or even traditional ML
   - Route simple queries to templates
   - ~60% of queries handled here

2. **Tier 2: Standard responses** (medium model, 200ms)
   - 13B-30B model for common questions
   - RAG-augmented for knowledge base
   - ~35% of queries

3. **Tier 3: Complex queries** (large model, 1-2s)
   - 70B+ or GPT-4 for edge cases
   - Human escalation for failures
   - ~5% of queries

**Cost savings**: 70% cheaper than using large model for everything.

**Monitoring**: Track routing accuracy, escalation rate, user satisfaction by tier.

---

### S2: You have 10K domain-specific documents. How do you make an LLM expert on them?

**Answer:**

**Approach comparison:**

| Approach | Pros | Cons | When to use |
|----------|------|------|-------------|
| **RAG** | Fast setup, updatable | Retrieval quality critical | Most cases |
| **Fine-tuning** | Internalized knowledge | Expensive, static | Consistent format needed |
| **Long context** | Simple, no chunking | Expensive, slow | <100 docs |
| **Hybrid** | Best quality | Complex | High-value use case |

**Recommended**: Start with RAG
1. Chunk documents (512-1024 tokens, 20% overlap)
2. Generate embeddings (text-embedding-3-large)
3. Store in vector DB (Pinecone/Weaviate)
4. Retrieve top-5 chunks, pass to LLM
5. Add reranking if precision matters (Cohere rerank)

**Fine-tuning consideration**: Only if queries need specific terminology or output format.

**Follow-up**: "How do you handle updates?" → "RAG: just re-embed. Fine-tuning: retrain monthly (expensive)."
