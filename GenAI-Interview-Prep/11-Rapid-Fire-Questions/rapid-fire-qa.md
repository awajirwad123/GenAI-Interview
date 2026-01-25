# Rapid Fire Q&A - GenAI Interview Prep

**Format:** Quick questions with concise answers (30-60 seconds each)
**Based on:** Real interview data from 2024-2026

---

## LLM Fundamentals

**Q: What does GPT stand for?**
A: Generative Pre-trained Transformer

**Q: What's the difference between parameters and tokens?**
A: Parameters = model weights (e.g., 7B in Llama-2-7B). Tokens = text units (~4 chars), used for input/output counting.

**Q: What's the context window in GPT-4?**
A: 128K tokens (GPT-4-turbo), 8K (GPT-4 base). Claude-3: 200K. Gemini-1.5: 1M.

**Q: What is temperature in LLMs?**
A: Controls randomness. 0 = deterministic, 1 = balanced, 2 = very creative.

**Q: What's the difference between tokens and words?**
A: 1 token ≈ 0.75 words. "ChatGPT" = 2 tokens ("Chat" + "GPT"). Important for cost calculation.

**Q: What's quantization?**
A: Reducing numerical precision (FP32 → INT8) to save memory and speed up inference.

**Q: What does "pre-training" mean?**
A: Initial training on massive text corpus (web, books) to learn language patterns.

**Q: What's the difference between GPT-3.5 and GPT-4?**
A: GPT-4 is larger, more accurate, supports images, 10× more expensive, slower.

**Q: What is fine-tuning?**
A: Additional training on specific dataset to adapt model for particular tasks/domains.

**Q: What's a prompt?**
A: Input text given to an LLM to generate a response.

**Q: What does BERT stand for?**
A: Bidirectional Encoder Representations from Transformers

**Q: What's the difference between encoder and decoder models?**
A: Encoder (BERT) = understands text, good for classification. Decoder (GPT) = generates text.

**Q: What is attention mechanism?**
A: Allows model to focus on relevant parts of input when processing each token.

**Q: What's RLHF?**
A: Reinforcement Learning from Human Feedback - aligns model with human preferences.

**Q: What's the typical cost per 1M tokens for GPT-4?**
A: ~$10 input, ~$30 output (as of 2026).

---

## Embeddings & Vector Search

**Q: What are embeddings?**
A: Dense vector representations of text that capture semantic meaning (e.g., [0.2, -0.5, 0.8, ...]).

**Q: What's a typical embedding dimension?**
A: 384 (small models), 768 (BERT), 1536 (OpenAI ada-002), 3072 (text-embedding-3-large).

**Q: What's cosine similarity?**
A: Measures angle between vectors. Range: -1 to 1 (1 = identical, 0 = unrelated).

**Q: Name 3 vector databases.**
A: Pinecone, Weaviate, Qdrant, Chroma, pgvector, Milvus.

**Q: What's the difference between dot product and cosine similarity?**
A: Dot product = magnitude matters. Cosine = only direction matters (normalized).

**Q: What's HNSW?**
A: Hierarchical Navigable Small World - graph-based indexing algorithm for fast vector search.

**Q: What's ANN?**
A: Approximate Nearest Neighbor - fast but not 100% accurate vector search (vs exact KNN).

**Q: How do you handle vector search for 1B vectors?**
A: Sharding, quantization, HNSW/IVF indexing, filtering, caching.

**Q: What's semantic search?**
A: Search by meaning (not keywords). "Cheap flights" matches "affordable tickets".

**Q: Typical embedding API latency?**
A: 20-50ms (OpenAI), 30ms (open-source local).

---

## RAG Systems

**Q: What does RAG stand for?**
A: Retrieval-Augmented Generation

**Q: Explain RAG in one sentence.**
A: Retrieve relevant documents, add to prompt, LLM generates answer using that context.

**Q: What's chunking?**
A: Splitting documents into smaller pieces for embedding and retrieval.

**Q: Typical chunk size?**
A: 256-512 tokens with 10-20% overlap.

**Q: What's the RAG pipeline order?**
A: Query → Embed → Search vectors → Retrieve docs → Rerank → Add to prompt → LLM → Answer.

**Q: What's reranking?**
A: Scoring retrieved docs by relevance, keeping top K (e.g., 50 → 5).

**Q: Name 3 chunking strategies.**
A: Fixed-size, semantic (by paragraphs), recursive (by hierarchy).

**Q: What's hybrid search?**
A: Combines vector search (semantic) + keyword search (BM25) for better recall.

**Q: Why does RAG reduce hallucinations?**
A: Grounds responses in retrieved facts rather than relying on model's memorized knowledge.

**Q: What's a typical RAG system latency?**
A: 1-3 seconds (retrieval 100-300ms, LLM 600-2000ms).

**Q: What's the difference between RAG and fine-tuning?**
A: RAG = add context at query time. Fine-tuning = modify model weights.

**Q: When would you use RAG over fine-tuning?**
A: When data changes frequently, need explainability, or have limited budget.

**Q: What's query expansion?**
A: Generating multiple variations of user query to improve retrieval (e.g., synonyms).

**Q: What's HyDE?**
A: Hypothetical Document Embeddings - generate fake answer, use it to search (better retrieval).

**Q: What's multi-hop retrieval?**
A: Iteratively retrieve → read → retrieve again for complex questions needing multiple sources.

---

## Prompt Engineering

**Q: What's Chain-of-Thought (CoT)?**
A: Prompting technique: "Let's think step by step" - improves reasoning.

**Q: What's few-shot prompting?**
A: Providing examples in prompt (e.g., 3 examples before asking question).

**Q: What's zero-shot prompting?**
A: No examples, just ask the question directly.

**Q: What's ReAct?**
A: Reasoning + Acting - LLM alternates between thinking and using tools.

**Q: What's a system prompt?**
A: Instructions that define model's role/behavior (e.g., "You are a helpful assistant").

**Q: What's prompt injection?**
A: Malicious input that overrides system instructions (e.g., "Ignore previous instructions").

**Q: How do you prevent prompt injection?**
A: Input sanitization, role separation, output validation, instruction hierarchy.

**Q: What's the optimal prompt structure?**
A: System instruction → Few-shot examples → User query → Output format.

**Q: What's prompt versioning?**
A: Tracking prompt changes like code (v1, v2) to measure impact.

**Q: What's dynamic prompting?**
A: Adjusting prompt based on context (user type, query complexity, etc.).

**Q: What's self-consistency?**
A: Generate multiple answers, take majority vote for better accuracy.

**Q: What's Tree-of-Thoughts?**
A: Exploring multiple reasoning paths (tree structure) vs single chain.

---

## Agents & Tools

**Q: What's an AI agent?**
A: LLM that can use tools and make decisions autonomously to complete tasks.

**Q: What's function calling?**
A: LLM outputs structured data (JSON) to invoke external functions/APIs.

**Q: What's the ReAct loop?**
A: Thought → Action → Observation → repeat until done.

**Q: Name 3 common agent tools.**
A: Web search, calculator, database query, code execution, email sender.

**Q: What's tool selection?**
A: Agent deciding which tool to use based on task requirements.

**Q: What's a multi-agent system?**
A: Multiple specialized agents collaborating (e.g., researcher + writer + critic).

**Q: What's the difference between agents and chains?**
A: Chains = fixed steps. Agents = dynamic, LLM decides next action.

**Q: What's plan-and-execute?**
A: Agent plans all steps first, then executes (vs ReAct which interleaves).

**Q: Common agent failure mode?**
A: Infinite loops, wrong tool selection, hallucinated tool inputs.

**Q: How do you debug failing agents?**
A: Log each step (thought, action, observation), set max iterations, add fallbacks.

---

## Azure OpenAI

**Q: Difference between OpenAI and Azure OpenAI?**
A: Azure: Enterprise security, SLA, regional deployment, private endpoints. OpenAI: Faster updates.

**Q: What's TPM in Azure OpenAI?**
A: Tokens Per Minute - rate limit for API calls.

**Q: What's PTU?**
A: Provisioned Throughput Units - reserved capacity (vs pay-per-token).

**Q: What's content filtering in Azure?**
A: Automatic detection/blocking of harmful content (hate, violence, self-harm).

**Q: How do you handle 429 errors?**
A: Retry with exponential backoff, multiple deployments, queue requests.

**Q: What's a deployment in Azure OpenAI?**
A: Instance of a model in specific region with allocated quota.

**Q: Best Azure region for OpenAI?**
A: East US, West Europe (most models, highest quota).

**Q: What's Managed Identity?**
A: Passwordless authentication using Azure AD (more secure than API keys).

**Q: What's VNET injection?**
A: Deploying Azure OpenAI in private network (not public internet).

**Q: Typical Azure OpenAI quota?**
A: 240K TPM (GPT-4), 600K TPM (GPT-3.5) - varies by region and subscription.

---

## LangChain & Frameworks

**Q: What's LangChain?**
A: Python framework for building LLM applications (chains, agents, RAG).

**Q: What's a chain in LangChain?**
A: Sequence of operations (e.g., prompt → LLM → parser).

**Q: What's LangGraph?**
A: Extension of LangChain for building stateful, multi-step agent workflows as graphs.

**Q: What's LangSmith?**
A: Debugging/monitoring tool for LangChain applications (tracing, evaluation).

**Q: Alternatives to LangChain?**
A: LlamaIndex, Haystack, Semantic Kernel, custom code.

**Q: What's memory in LangChain?**
A: Storing conversation history (BufferMemory, SummaryMemory, VectorMemory).

**Q: What's a LangChain agent?**
A: Autonomous system that uses LLM to decide which tools to call.

**Q: When NOT to use LangChain?**
A: Production-critical systems, need <100ms latency, simple use cases.

**Q: What's LCEL?**
A: LangChain Expression Language - declarative way to build chains (pipe operator |).

**Q: What's a document loader in LangChain?**
A: Component that loads data from sources (PDF, web, database) into Document objects.

---

## Production & MLOps

**Q: What's model versioning?**
A: Tracking different versions of models/prompts to enable rollback and A/B testing.

**Q: What's A/B testing for LLMs?**
A: Sending 50% traffic to model A, 50% to B, comparing metrics.

**Q: What's semantic caching?**
A: Caching responses for similar (not identical) queries using embeddings.

**Q: Typical cache hit rate?**
A: 30-40% for FAQs, 10-20% for general queries.

**Q: What's prompt caching?**
A: Caching common prompt prefixes (system instructions) to reduce latency/cost.

**Q: What's latency budget for chatbots?**
A: <500ms ideal, <2s acceptable, >3s users drop off.

**Q: What's observability for LLMs?**
A: Logging inputs/outputs, latency, costs, errors for monitoring and debugging.

**Q: Name 3 LLM observability tools.**
A: LangSmith, Weights & Biases, Helicone, Arize, Phoenix.

**Q: What's guardrails?**
A: Safety checks on inputs/outputs (block harmful content, PII, hallucinations).

**Q: What's the typical LLM API error rate?**
A: <1% for well-configured systems, spikes during outages.

**Q: What's model drift?**
A: Model performance degrading over time due to data distribution changes.

**Q: How do you monitor model quality in production?**
A: Sample evaluation (RAGAS), user feedback, A/B testing, human review.

**Q: What's the cost of human evaluation?**
A: $2-10 per evaluation (vs $0.01 for LLM-as-judge).

**Q: What's shadow deployment?**
A: Running new model alongside production without affecting users (for testing).

**Q: What's canary deployment?**
A: Gradually rolling out to 5% → 20% → 50% → 100% of traffic.

---

## Evaluation & Metrics

**Q: What's RAGAS?**
A: Retrieval-Augmented Generation Assessment - metrics for RAG systems.

**Q: Name 3 RAGAS metrics.**
A: Faithfulness (grounded in context?), answer relevance, context relevance.

**Q: What's faithfulness in RAG?**
A: Answer contains only information from retrieved documents (no hallucination).

**Q: What's LLM-as-judge?**
A: Using GPT-4 to evaluate other LLM outputs (cheaper than humans).

**Q: What's BLEU score?**
A: Measures n-gram overlap with reference (used for translation, 0-1 scale).

**Q: What's ROUGE score?**
A: Recall-Oriented Understudy for Gisting Evaluation - overlap metric for summaries.

**Q: What's perplexity?**
A: How "surprised" model is by text. Lower = better (fluency measure).

**Q: How do you evaluate without ground truth?**
A: LLM-as-judge, user feedback, consistency checks, fact verification.

**Q: What's inter-annotator agreement?**
A: How much human evaluators agree (Cohen's kappa, should be >0.7).

**Q: What's the gold standard for evaluation?**
A: Human evaluation on representative test set (expensive but most reliable).

**Q: What's precision vs recall in retrieval?**
A: Precision = % retrieved docs relevant. Recall = % relevant docs retrieved.

**Q: What's MRR (Mean Reciprocal Rank)?**
A: Average of 1/rank of first relevant doc. Higher = better (0-1 scale).

---

## Model Selection & Architecture

**Q: What's the difference between base and instruct models?**
A: Base = raw pre-trained. Instruct = fine-tuned to follow instructions.

**Q: When would you use Llama-2 vs GPT-4?**
A: Llama-2: Self-hosted, cost-sensitive, privacy. GPT-4: Best quality, less setup.

**Q: What's Claude good for?**
A: Long documents (200K context), reduced harmful outputs, good reasoning.

**Q: What's Mistral?**
A: Open-source alternative to GPT-3.5, good quality/speed tradeoff.

**Q: What's a mixture of experts (MoE)?**
A: Model with multiple sub-networks, activates subset per input (faster, larger capacity).

**Q: What's GPT-4-turbo?**
A: Faster, cheaper version of GPT-4 with 128K context and newer knowledge cutoff.

**Q: What's the knowledge cutoff problem?**
A: Model only knows info from training data (e.g., GPT-4 trained on data up to Apr 2023).

**Q: How do you get current information?**
A: RAG with web search, plugins, function calling, fine-tuning on recent data.

**Q: What's multimodal LLM?**
A: Processes multiple types (text, images, audio) - e.g., GPT-4V, Gemini.

**Q: What's vision-language model?**
A: Understands both images and text (e.g., GPT-4V can describe images).

---

## Cost Optimization

**Q: How do you reduce LLM costs?**
A: Use cheaper models (GPT-3.5), caching, shorter prompts, batching, fine-tune small model.

**Q: What's the cost difference between GPT-3.5 and GPT-4?**
A: GPT-4 is ~15× more expensive than GPT-3.5-turbo.

**Q: How much does caching save?**
A: 30-40% cost reduction if 40% cache hit rate (for repeated queries).

**Q: What's tiered model routing?**
A: Route simple queries to cheap model, complex to expensive (80% GPT-3.5, 20% GPT-4).

**Q: How do you reduce prompt tokens?**
A: Remove examples, compress context, use abbreviations, optimize formatting.

**Q: What's the ROI calculation for fine-tuning?**
A: Upfront cost $100-1000 vs per-query savings × expected queries.

**Q: What's provisioned throughput?**
A: Pre-pay for capacity (PTU) vs pay-per-token. Better if >500K tokens/day.

**Q: How do you estimate monthly LLM costs?**
A: Daily requests × avg tokens × token price × 30 days.

**Q: What's cheaper: RAG or fine-tuning?**
A: RAG for most cases (no upfront cost, flexible). Fine-tuning if very high volume.

**Q: How much does embedding cost?**
A: OpenAI: $0.13/1M tokens. Open-source: Free (just compute).

---

## Fine-tuning & Training

**Q: What's LoRA?**
A: Low-Rank Adaptation - efficient fine-tuning method (trains <1% of parameters).

**Q: What's QLoRA?**
A: Quantized LoRA - even more memory-efficient (can fine-tune on single GPU).

**Q: How many examples for fine-tuning?**
A: Minimum 50-100, ideal 1K-10K, diminishing returns after 50K.

**Q: What's catastrophic forgetting?**
A: Model loses general knowledge when fine-tuned too aggressively on narrow task.

**Q: What's the typical fine-tuning cost?**
A: $100-500 for LoRA, $5K-50K for full fine-tuning (depends on model size).

**Q: What's distillation?**
A: Training small model to mimic large model (compress knowledge).

**Q: What's transfer learning?**
A: Using pre-trained model as starting point for new task (vs training from scratch).

**Q: What's supervised fine-tuning (SFT)?**
A: Training on input-output pairs (e.g., question-answer).

**Q: What's instruction tuning?**
A: Fine-tuning on diverse instructions to improve zero-shot task following.

**Q: What's DPO?**
A: Direct Preference Optimization - alternative to RLHF (simpler, more stable).

---

## System Design

**Q: How would you scale a RAG system to 1M users?**
A: Load balancing, caching, vector DB sharding, CDN for embeddings, rate limiting.

**Q: How do you handle PII in LLM systems?**
A: Redact before sending to LLM, use Azure private endpoints, log sanitization.

**Q: What's a good RAG system architecture?**
A: API Gateway → Intent classifier → RAG pipeline (retrieve → rerank → LLM) → Cache → Response.

**Q: How do you ensure 99.9% uptime?**
A: Multi-region deployment, health checks, automatic failover, circuit breakers.

**Q: What's a circuit breaker?**
A: Stop sending requests to failing service, retry after cooldown (prevent cascade failures).

**Q: How do you handle rate limits?**
A: Multiple API keys/deployments, queue requests, exponential backoff, quotas per user.

**Q: What's data privacy concern with LLMs?**
A: Training data leakage, API sends data to third party, prompt injection extracts info.

**Q: How do you make LLM systems explainable?**
A: Cite sources (RAG), log reasoning (CoT), confidence scores, human-in-loop.

**Q: What's the typical architecture for a chatbot?**
A: User → Load balancer → API server → LLM + Vector DB → Memory store → Response.

**Q: How do you test LLM applications?**
A: Unit tests (components), integration tests (pipeline), evaluation sets, human review.

---

## Recent Trends (2024-2026)

**Q: What's GPT-5?**
A: Next OpenAI model (rumored), expected to have better reasoning, efficiency.

**Q: What's Gemini Ultra?**
A: Google's most capable model, 1M token context, multimodal.

**Q: What's Claude 3?**
A: Anthropic's latest (Opus, Sonnet, Haiku), 200K context, reduced refusals.

**Q: What's Llama 3?**
A: Meta's open-source model, competitive with GPT-3.5, free commercial use.

**Q: What's o1 model (OpenAI)?**
A: Reasoning-focused model that "thinks" before answering (more accurate, slower).

**Q: What's RAG 2.0?**
A: Advanced RAG with agentic retrieval, query rewriting, multi-hop, self-correction.

**Q: What's agentic RAG?**
A: Agent decides when/what to retrieve dynamically (vs fixed retrieval step).

**Q: What's small language models (SLM)?**
A: Efficient models <10B params (Phi-3, Mistral-7B) - good quality, fast, cheap.

**Q: What's on-device AI?**
A: Running LLMs locally on phones/laptops (privacy, no latency). Example: Phi-3.

**Q: What's prompt caching (Anthropic)?**
A: Cache up to 90% of prompt, pay only for new tokens (massive cost savings).

**Q: What's structured outputs (OpenAI)?**
A: Guaranteed valid JSON output matching schema (no parsing errors).

**Q: What's Copilot Studio?**
A: Microsoft's platform for building custom copilots with low-code tools.

---

## Tricky Conceptual Questions

**Q: Can LLMs reason?**
A: Debated. They pattern-match well but lack symbolic reasoning. CoT helps but not true reasoning.

**Q: Do LLMs understand language?**
A: They model statistical patterns. "Understanding" is philosophical - they behave as if they do.

**Q: Why do LLMs hallucinate?**
A: Trained to generate plausible text, not necessarily true. No grounding in facts.

**Q: Can LLMs replace humans?**
A: Augment, not replace. Good for repetitive tasks, need humans for creativity, judgment, context.

**Q: Are embeddings biased?**
A: Yes, reflect biases in training data (gender, race, etc.). Need debiasing techniques.

**Q: Can you trust LLM outputs?**
A: No - always verify critical information. Use for drafts, suggestions, not final decisions.

**Q: Do LLMs have memory?**
A: No persistent memory. Only see current conversation (context window).

**Q: Can LLMs learn in real-time?**
A: No (without fine-tuning). In-context learning ≠ updating weights.

**Q: What's emergent abilities?**
A: Surprising capabilities that appear only in large models (e.g., reasoning in 175B+ models).

**Q: What's the Chinchilla scaling law?**
A: Optimal model size = balanced with data size. 70B model needs 1.4T tokens training data.

---

## Debugging & Troubleshooting

**Q: LLM returns empty/truncated output. Why?**
A: Hit max_tokens limit, content filter triggered, malformed prompt, API timeout.

**Q: RAG system returns wrong answers. How to debug?**
A: Check retrieved docs (relevant?), reranking (working?), prompt (clear?), eval metrics.

**Q: Agent gets stuck in loop. Fix?**
A: Add max_iterations, better tool descriptions, add "FINISH" condition, log each step.

**Q: High latency (5+ seconds). Optimize?**
A: Use smaller model, caching, streaming, batching, reduce retrieval time.

**Q: Cost explosion ($10K → $50K). Root cause?**
A: Check: long prompts, using GPT-4 unnecessarily, no caching, inefficient retries, abuse.

**Q: Low RAG recall (<50%). Improve?**
A: Better chunking, query expansion, hybrid search, increase k, better embedding model.

**Q: LLM ignores instructions. Why?**
A: Prompt too long (diluted), conflicting instructions, model limitations, need fine-tuning.

**Q: Vector search returns irrelevant docs. Debug?**
A: Check embedding quality, similarity threshold, metadata filters, query preprocessing.

**Q: Function calling fails. Common issues?**
A: Malformed JSON, wrong parameter types, unclear function descriptions, model limitations.

**Q: Embeddings clustering poorly. Fix?**
A: Try different model, fine-tune embeddings, normalize vectors, check data quality.

---

## Quick Statistics to Memorize

- **GPT-4**: 1.76T params (rumored), $0.03/1K tokens
- **GPT-3.5**: 175B params, $0.002/1K tokens
- **Llama-2**: 7B/13B/70B params, open-source
- **BERT**: 110M (base), 340M (large) params
- **Token-to-word ratio**: ~0.75 (1 token ≈ 4 chars)
- **Typical embedding dimensions**: 384, 768, 1536, 3072
- **Context windows**: GPT-4 (128K), Claude-3 (200K), Gemini (1M)
- **Average RAG latency**: 1-3 seconds
- **Embedding latency**: 20-50ms
- **Cache hit rate**: 30-40% (FAQs)
- **LLM API error rate**: <1% (healthy system)
- **Human eval cost**: $2-10 per evaluation
- **LLM-as-judge cost**: $0.01 per evaluation
- **Fine-tuning LoRA**: $100-500
- **Vector DB cost**: $140/month for 1M vectors (Pinecone)

---

## How to Use This Document

**Before interview (15 mins):**
- Scan all questions
- Read answers you're unsure about
- Memorize key statistics

**During interview:**
- Answer concisely (30-60 seconds)
- If asked to elaborate, then provide details
- Use examples from your experience

**Pro tips:**
- Don't just memorize - understand WHY
- Relate answers to your projects
- If unsure, reason through it out loud
- It's okay to say "I don't know, but here's how I'd find out"

**Common follow-ups:**
- "Have you used this in production?"
- "What challenges did you face?"
- "How would you improve it?"
- "What's the cost/latency tradeoff?"

Good luck! 🎯
