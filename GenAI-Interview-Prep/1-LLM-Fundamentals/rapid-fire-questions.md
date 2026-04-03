# LLM Fundamentals - Rapid-Fire Interview Questions

Quick-fire questions covering all fundamental concepts. Ideal for last-minute revision and interview preparation.

---

## 1. AI Engineer Role & Career

**Q1:** What is the primary difference between an AI Engineer and an ML Engineer?  
**A:** AI Engineers build applications using pre-trained LLMs (via APIs), focusing on prompt engineering, RAG, and orchestration. ML Engineers train custom models from scratch, focusing on feature engineering and model training pipelines.

**Q2:** Name 3 core responsibilities of an AI Engineer.  
**A:** (1) Build GenAI applications (chatbots, RAG systems, agents), (2) Integrate LLM APIs and optimize cost/latency trade-offs, (3) Monitor and evaluate AI systems in production.

**Q3:** What's the typical tech stack for an AI Engineer?  
**A:** Python, LangChain/LlamaIndex, vector databases (Pinecone, Weaviate), OpenAI/Azure OpenAI APIs, Docker/Kubernetes.

**Q4:** Do AI Engineers train models from scratch?  
**A:** No, they primarily use pre-trained LLMs via APIs and focus on building production-ready applications.

---

## 2. Large Language Models (LLMs)

**Q5:** What architecture are modern LLMs based on?  
**A:** Transformer architecture with self-attention mechanisms.

**Q6:** Name 3 popular LLMs in 2026 and their key features.  
**A:** (1) GPT-4 Turbo - best quality, $0.01/1K tokens, (2) Claude 3.5 Sonnet - strong reasoning, 200K context, (3) Llama 3 - open-source, 8B-70B params, free.

**Q7:** What are the three main architectural variants of transformers?  
**A:** Encoder-only (BERT), Decoder-only (GPT), Encoder-Decoder (T5).

**Q8:** What is the parameter count range for modern LLMs?  
**A:** Billions to trillions of parameters (GPT-4: ~1.7T, Llama-3: 70B).

**Q9:** What are the key capabilities of LLMs?  
**A:** Text generation, summarization, Q&A, code generation, reasoning, and instruction following.

**Q10:** Why do GPT models dominate GenAI applications?  
**A:** Autoregressive generation (decoder-only) is more flexible than masked language modeling for generative tasks.

---

## 3. Inference

**Q11:** What is inference in the context of LLMs?  
**A:** The process of using a trained LLM to generate predictions/responses (not training).

**Q12:** Explain autoregressive generation.  
**A:** Generate one token at a time, feed the generated token back as input for the next token prediction.

**Q13:** What are the three main sampling methods for token generation?  
**A:** (1) Greedy (take highest probability), (2) Top-K sampling, (3) Top-P/nucleus sampling.

**Q14:** What is KV-cache and how much speedup does it provide?  
**A:** Caches previous tokens' key-value pairs to avoid recomputation during generation, providing ~30% speedup.

**Q15:** Name three key inference performance metrics.  
**A:** (1) Latency (TTFT + TPT), (2) Throughput (tokens/second), (3) Cost ($/1K tokens).

**Q16:** What is the typical cost of a 1000-token GPT-4 response?  
**A:** ~$0.06 (inference costs dominate at scale).

**Q17:** What is quantization and what memory reduction does it provide?  
**A:** Converting model weights to lower precision (INT8/INT4), reducing memory by 2-4x.

**Q18:** What is speculative decoding?  
**A:** A small model drafts tokens, large model verifies them in parallel, achieving ~2x speedup.

**Q19:** What is Flash Attention?  
**A:** A memory-efficient attention mechanism that provides 2-4x faster inference.

**Q20:** What does dynamic batching do for inference?  
**A:** Processes multiple requests in parallel, increasing throughput 3-10x.

---

## 4. Training

**Q21:** What are the three training stages for LLMs?  
**A:** (1) Pre-training (foundation models), (2) Instruction tuning (SFT), (3) Alignment (RLHF/DPO).

**Q22:** What is pre-training and how much does it cost?  
**A:** Learning language patterns from massive text (1-10T tokens) via self-supervised next-token prediction. Cost: $10M-$100M+.

**Q23:** What is instruction tuning (SFT)?  
**A:** Teaching the model to follow instructions using ~100K instruction-response pairs. Cost: $10K-$100K.

**Q24:** What is RLHF?  
**A:** Reinforcement Learning from Human Feedback - aligning the model with human preferences using a reward model trained on human comparisons.

**Q25:** What is the difference between full fine-tuning and LoRA?  
**A:** Full fine-tuning updates all parameters (requires 100K+ examples). LoRA trains low-rank adapters (~0.1% of parameters), saving 90% memory.

**Q26:** What is QLoRA?  
**A:** Quantized LoRA - combines quantization with LoRA for even more memory-efficient fine-tuning.

**Q27:** When should you fine-tune an LLM?  
**A:** When base model consistently fails on domain-specific tasks, you need consistent output formats, or have privacy requirements.

**Q28:** What does the Chinchilla paper's scaling law suggest?  
**A:** Optimal ratio is ~20 tokens per parameter for training efficiency.

**Q29:** What is catastrophic forgetting?  
**A:** When fine-tuning causes the model to lose pre-trained knowledge. Mitigate by mixing pre-training data or replay techniques.

**Q30:** What is DPO?  
**A:** Direct Preference Optimization - a simpler alternative to RLHF that's more stable to train.

---

## 5. Embeddings

**Q31:** What are embeddings?  
**A:** Dense vector representations of text that capture semantic meaning, where similar text produces similar vectors.

**Q32:** How is similarity measured between embeddings?  
**A:** Using cosine similarity or dot product.

**Q33:** What are typical embedding dimensions?  
**A:** 512-3072 dimensions (OpenAI text-embedding-3-small: 1536 dims).

**Q34:** Name three use cases for embeddings.  
**A:** (1) Semantic search, (2) RAG (retrieval), (3) Document clustering/recommendations.

**Q35:** What's the cost difference between OpenAI's small and large embedding models?  
**A:** text-embedding-3-small: $0.02/1M tokens, text-embedding-3-large: $0.13/1M tokens.

**Q36:** What is the typical latency for generating embeddings?  
**A:** 10-50ms per embedding.

**Q37:** Name two popular open-source embedding models.  
**A:** Sentence-Transformers (all-MiniLM), BGE models.

**Q38:** Why are embeddings important for RAG?  
**A:** They enable semantic similarity search to retrieve relevant context from knowledge bases.

---

## 6. Vector Databases

**Q39:** What is a vector database?  
**A:** A database optimized for storing and querying high-dimensional vectors using similarity search.

**Q40:** Why can't traditional databases efficiently handle vector similarity search?  
**A:** They require O(n) linear scan. Vector DBs use indexing (HNSW, IVF) for O(log n) approximate nearest neighbor search.

**Q41:** Name three popular vector databases.  
**A:** (1) Pinecone (managed, serverless), (2) Weaviate (open-source, hybrid search), (3) Qdrant (open-source, Rust-based, fast).

**Q42:** What is HNSW indexing?  
**A:** Hierarchical Navigable Small World graphs - an indexing method optimized for accuracy in vector search.

**Q43:** What is hybrid search?  
**A:** Combining vector similarity search with keyword matching (e.g., BM25) for better results.

**Q44:** What is the typical latency for searching across 1M vectors?  
**A:** 10-50ms for top-10 results.

**Q45:** What is the typical cost for vector database storage?  
**A:** $0.05-$0.20 per GB per month.

**Q46:** What is pgvector?  
**A:** A PostgreSQL extension for vector search, good for small-scale applications.

**Q47:** What are metadata filters in vector databases?  
**A:** Filters applied before or during vector search (e.g., "date > 2024 AND category = 'finance'").

---

## 7. RAG (Retrieval-Augmented Generation)

**Q48:** What is RAG?  
**A:** Retrieval-Augmented Generation - enhancing LLM responses by retrieving relevant context from external knowledge bases before generation.

**Q49:** Describe the three stages of RAG.  
**A:** (1) Indexing: chunk documents → embeddings → vector DB, (2) Retrieval: query → embed → search → top-K chunks, (3) Generation: inject chunks into LLM prompt → answer.

**Q50:** What are the main benefits of RAG?  
**A:** Reduces hallucinations, provides up-to-date knowledge, enables citations, cheaper than fine-tuning for knowledge updates.

**Q51:** What are three chunking strategies?  
**A:** (1) Fixed-size (500 tokens), (2) Semantic (paragraphs/sections), (3) Hybrid (combination).

**Q52:** What is reranking in RAG?  
**A:** Using a cross-encoder model (like Cohere Rerank, ColBERT) to reorder top-K retrieved results for better relevance.

**Q53:** What is query expansion?  
**A:** Rephrasing the query multiple ways and merging results to improve retrieval coverage.

**Q54:** What's the typical latency for a RAG query?  
**A:** 200-500ms (100ms retrieval + 300ms generation).

**Q55:** How much does RAG improve quality over base LLM?  
**A:** 30-50% better on domain-specific questions.

**Q56:** What is the typical cost per RAG query?  
**A:** $0.01-$0.05 (embedding + LLM + vector DB costs).

---

## 8. Prompt Engineering

**Q57:** What is prompt engineering?  
**A:** The art of crafting effective prompts to get desired outputs from LLMs without fine-tuning.

**Q58:** What are the four core principles of effective prompts?  
**A:** (1) Be specific, (2) Use examples (few-shot), (3) Think step-by-step (CoT), (4) Constrain output format.

**Q59:** What is zero-shot prompting?  
**A:** Asking the model to perform a task without any examples.

**Q60:** What is few-shot prompting?  
**A:** Providing 2-3 examples in the prompt to demonstrate the desired behavior.

**Q61:** What is Chain-of-Thought (CoT) prompting?  
**A:** Prompting the model to solve problems step-by-step, showing intermediate reasoning.

**Q62:** What is ReAct prompting?  
**A:** Reasoning + Acting - alternating between thought, action (tool use), and observation steps.

**Q63:** What is the difference between system prompts and user prompts?  
**A:** System prompts set the behavior/role of the assistant, user prompts are the actual queries/instructions.

**Q64:** What temperature value should you use for deterministic output?  
**A:** 0.0 (greedy decoding).

**Q65:** What temperature range is good for creative tasks?  
**A:** 0.7-1.5 for balanced to highly creative outputs.

**Q66:** How do you constrain output format in prompts?  
**A:** Explicitly specify format (e.g., "Respond in JSON: {\"answer\": \"...\", \"confidence\": 0.9}").

---

## 9. AI Agents

**Q67:** What is an AI agent?  
**A:** An autonomous system that uses LLMs to reason, plan, and take actions using tools to achieve goals.

**Q68:** What are the five key components of an AI agent?  
**A:** (1) LLM brain, (2) Tools/functions, (3) Memory, (4) Planning, (5) Execution loop.

**Q69:** Describe the ReAct agent pattern.  
**A:** Iterative loop: Thought → Action (tool use) → Observation → Thought → ... → Final Answer.

**Q70:** What is the difference between ReAct and Plan-and-Execute patterns?  
**A:** ReAct iterates step-by-step, Plan-and-Execute plans all steps upfront then executes sequentially.

**Q71:** Name three types of tools agents commonly use.  
**A:** (1) Search (web, database), (2) Code execution (Python), (3) APIs (weather, CRM, file ops).

**Q72:** What are multi-agent systems?  
**A:** Multiple specialized agents (e.g., Researcher, Writer, Critic) that coordinate to solve complex problems.

**Q73:** Name three popular agent frameworks.  
**A:** (1) LangGraph (stateful agents with graphs), (2) CrewAI (multi-agent), (3) Semantic Kernel (Microsoft).

**Q74:** What is the typical success rate of agents on complex tasks?  
**A:** 50-70% (agents fail 30-50% of the time on complex tasks).

**Q75:** What are the main challenges with AI agents?  
**A:** Reliability (~30-50% failure rate), cost ($0.10-$1 per task), latency (5-30s), safety (uncontrolled tool usage).

**Q76:** What is function calling (tool calling)?  
**A:** LLMs generating structured function/tool invocations with parameters based on user requests.

---

## 10. AI vs AGI

**Q77:** What is the difference between AI and AGI?  
**A:** AI is narrow intelligence for specific tasks. AGI is human-level general intelligence across ALL domains.

**Q78:** Do we have AGI today (2026)?  
**A:** No, we have increasingly powerful narrow AI but no system matches human general intelligence across all domains.

**Q79:** What are three key capabilities AGI would need?  
**A:** (1) Learn any task humans can, (2) Transfer knowledge across domains, (3) Reason and adapt to novel situations without training.

**Q80:** What's an example of why GPT-4 is not AGI?  
**A:** It's superhuman at text tasks but can't transfer learning like humans, lacks common sense in novel situations, and has no consciousness.

**Q81:** When do experts predict AGI might be achieved?  
**A:** Predictions range from 2030-2050 (no consensus).

---

## 11. Transformer Architecture

**Q82:** What makes self-attention powerful?  
**A:** Enables parallel processing and captures long-range dependencies in sequences.

**Q83:** What is multi-head attention?  
**A:** Multiple attention mechanisms running in parallel, each capturing different representation subspaces.

**Q84:** What are three types of positional encoding?  
**A:** (1) Absolute (sinusoidal), (2) Relative, (3) Learned.

**Q85:** What's the difference between encoder-only, decoder-only, and encoder-decoder models?  
**A:** Encoder-only (BERT) for understanding, decoder-only (GPT) for generation, encoder-decoder (T5) for seq2seq tasks.

---

## 12. Model Sizes & Tradeoffs

**Q86:** What's the tradeoff between small (7B-13B) and large (175B+) models?  
**A:** Small: lower latency (~100ms), lower cost, edge deployment. Large: better quality, higher latency (1-3s), higher cost.

**Q87:** What's the production reality regarding model sizes?  
**A:** Most companies use 7B-13B models with fine-tuning rather than 175B+ base models.

---

## 13. Context Windows

**Q88:** What are typical context window sizes for modern LLMs?  
**A:** 8K-32K (standard), 100K-200K (long context like Claude 3, GPT-4 Turbo), 1M+ (Gemini 1.5).

**Q89:** What's the cost challenge with long context windows?  
**A:** Cost scales linearly with context length - 100K context is 25x more expensive than 4K.

**Q90:** Name two strategies for managing long contexts.  
**A:** (1) Intelligent chunking with semantic boundaries, (2) Context compression (LLMLingua, reranking).

---

## 14. Common Failure Modes

**Q91:** What are hallucinations and how do you mitigate them?  
**A:** Models generating plausible but false information. Mitigate with RAG, citations, and confidence scores.

**Q92:** What is recency bias?  
**A:** The model weights recent context more heavily than earlier tokens in long conversations. Mitigate with summarization or re-ranking.

**Q93:** What is instruction drift?  
**A:** When models ignore system prompts during long conversations. Mitigate with periodic re-injection or conversation summarization.

---

## 15. Token Economics

**Q94:** Why is code 2-3x more tokens than natural language?  
**A:** Code has more special characters, punctuation, and structure that require more subword tokens.

**Q95:** Why are non-English languages more expensive to process?  
**A:** They require 2-5x more tokens due to tokenizer design (trained primarily on English).

**Q96:** Why are output tokens more expensive than input tokens?  
**A:** Generation is more computationally expensive than encoding (2-3x cost multiplier).

**Q97:** What's the approximate cost for embeddings?  
**A:** ~$0.0001 per 1K tokens (much cheaper than LLM generation).

---

## 16. Emergent Abilities

**Q98:** What are emergent abilities in LLMs?  
**A:** Capabilities that appear at scale (>100B params) like chain-of-thought reasoning, improved few-shot learning, and multi-step problem solving.

**Q99:** Can smaller models achieve emergent abilities?  
**A:** Yes, often through fine-tuning or advanced prompt engineering techniques.

---

## 17. Advanced Concepts

**Q100:** What is the difference between BPE and WordPiece tokenization?  
**A:** Both are subword tokenization methods. BPE (Byte Pair Encoding) merges frequent byte pairs, WordPiece optimizes for likelihood.

**Q101:** What's the production tradeoff between managed (Pinecone) and self-hosted (Weaviate/Qdrant) vector databases?  
**A:** Managed: easier, serverless, higher cost ($0.096/GB/month). Self-hosted: more control, lower cost, requires ops overhead.

**Q102:** When is RAG better than fine-tuning?  
**A:** When knowledge changes frequently, you need citations, or you want to update without retraining.

**Q103:** When is fine-tuning better than RAG?  
**A:** When you need consistent behavior/format, style adaptation, or can't send data to external APIs (privacy).

**Q104:** What percentage of AI Engineering work is prompt engineering?  
**A:** Approximately 80% - iterating and optimizing prompts for quality.

---

## Quick Stats to Remember

- **GPT-4 cost**: ~$0.06 per 1000-token response
- **RAG latency**: 200-500ms typical
- **RAG improvement**: 30-50% better than base LLM
- **KV-cache speedup**: ~30%
- **Quantization**: 2-4x memory reduction
- **Dynamic batching**: 3-10x throughput increase
- **Agent failure rate**: 30-50% on complex tasks
- **Inference cost dominance**: At scale, inference >> training costs
- **LoRA efficiency**: ~0.1% parameters, 90% memory savings
- **Context cost scaling**: Linear (100K = 25x cost of 4K)

---

## Interview Tips

1. **Focus on production**: Emphasize real-world deployments, not just theory
2. **Know tradeoffs**: Cost vs quality vs latency (critical for AI Engineers)
3. **RAG is #1**: Be able to explain RAG architecture in detail
4. **Prompt engineering**: Show examples of iterative prompt improvement
5. **Agent experience**: ReAct pattern and function calling are hot topics
6. **Be specific**: Use concrete numbers (costs, latencies, improvement %)
7. **Don't claim AGI**: We have powerful narrow AI, not general intelligence
8. **LoRA over full fine-tuning**: Know when and why to use PEFT
9. **Vector DBs**: Understand indexing methods and when to use which DB
10. **Failure modes**: Show awareness of hallucinations, costs, latency issues

---

*Reference: See [concepts.md](concepts.md) for detailed explanations of all topics*
