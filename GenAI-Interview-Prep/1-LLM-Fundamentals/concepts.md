# LLM Fundamentals - Core Concepts

## 1. What is an AI Engineer?

**Definition**: An AI Engineer builds, deploys, and maintains AI-powered applications in production. They bridge the gap between ML research and real-world software systems.

**Key Responsibilities:**
- Design and implement GenAI applications (chatbots, RAG systems, agents)
- Integrate LLM APIs (OpenAI, Azure OpenAI, Anthropic) into applications
- Build data pipelines for embeddings and vector databases
- Optimize for cost, latency, and quality trade-offs
- Monitor and evaluate AI systems in production
- Handle prompt engineering and fine-tuning

**Tech Stack:** Python, LangChain/LlamaIndex, vector DBs (Pinecone, Weaviate), OpenAI/Azure OpenAI APIs, Docker/Kubernetes

**Focus**: Production-ready applications, not training models from scratch.

---

## 2. AI Engineer vs ML Engineer

| Aspect | AI Engineer | ML Engineer |
|--------|-------------|-------------|
| **Primary Focus** | Building GenAI applications with LLMs | Training and deploying custom ML models |
| **Models** | Use pre-trained LLMs (GPT, Claude) via APIs | Train models from scratch (XGBoost, Neural Nets) |
| **Skills** | Prompt engineering, RAG, vector DBs, LLM orchestration | Feature engineering, model training, hyperparameter tuning, MLOps |
| **Tools** | LangChain, OpenAI SDK, Pinecone, ChromaDB | TensorFlow, PyTorch, scikit-learn, MLflow |
| **Output** | Chatbots, agents, semantic search, Q&A systems | Predictive models, recommendation systems, image classifiers |
| **Data** | Text, documents, unstructured data | Structured data (CSVs, tables), labeled datasets |
| **Training** | Fine-tuning (optional), mostly inference | Full training pipelines, data preparation |

**Key Insight**: AI Engineers are "application developers" using LLMs as building blocks. ML Engineers are "model builders" creating custom ML solutions.

**Interview Tip**: Emphasize your experience with LLM APIs, prompt engineering, RAG pipelines, and production deployment—not deep neural network training.

---

## 3. Large Language Models (LLMs)

**Definition**: Neural networks trained on massive text corpora (trillions of tokens) to predict next tokens, enabling text generation, understanding, and reasoning.

**Key Characteristics:**
- **Architecture**: Transformer-based (GPT, BERT, T5)
- **Scale**: Billions to trillions of parameters (GPT-4: ~1.7T, Llama-3: 70B)
- **Training**: Self-supervised pre-training + instruction tuning + RLHF
- **Capabilities**: Text generation, summarization, Q&A, code generation, reasoning

**Popular LLMs (2026):**
- **GPT-4 Turbo**: OpenAI's flagship, best quality, $0.01/1K tokens
- **Claude 3.5 Sonnet**: Anthropic, strong reasoning, 200K context
- **Llama 3**: Meta's open-source, 8B-70B params, free
- **Gemini 1.5 Pro**: Google, 1M context window
- **Mistral**: Open-source, European, strong performance

**Interview Insight**: Know trade-offs (cost, latency, quality) and when to use which model.

---

## 4. Inference

**Definition**: The process of using a trained LLM to generate predictions/responses (not training).

**Key Concepts:**
- **Autoregressive Generation**: Generate one token at a time, feed back as input
- **Sampling Methods**: Greedy (take highest prob), Top-K, Top-P (nucleus), Temperature
- **KV-Cache**: Cache previous tokens' key-value pairs to avoid recomputation (30% speedup)
- **Batching**: Process multiple requests in parallel (10x throughput)

**Performance Metrics:**
- **Latency**: Time to first token (TTFT) + time per token (TPT)
- **Throughput**: Tokens/second or requests/second
- **Cost**: $/1K tokens (input + output)

**Optimization Techniques:**
- **Quantization**: INT8/INT4 reduces memory 2-4x (GPTQ, AWQ formats)
- **Speculative Decoding**: Small model drafts, large model verifies (2x speedup)
- **Flash Attention**: Memory-efficient attention (2-4x faster)

**Interview Tip**: Inference costs dominate at scale. A 1000-token GPT-4 response costs ~$0.06.

---

## 5. Training

**Definition**: The process of teaching a model from data (pre-training, fine-tuning, or RLHF).

**Three Training Stages:**

**1. Pre-training** (Foundation Models)
- Learn language patterns from massive text (1-10T tokens)
- Self-supervised: Predict next token (no labels needed)
- Cost: $10M-$100M+ (GPT-4, Llama-3)
- **Who does this**: Big tech (OpenAI, Meta, Google)

**2. Instruction Tuning** (SFT - Supervised Fine-Tuning)
- Teach model to follow instructions
- Dataset: 100K instruction-response pairs
- Cost: $10K-$100K
- **Who does this**: Companies building domain-specific models

**3. Alignment (RLHF/DPO)**
- Align with human preferences (helpful, harmless, honest)
- Reward model trained on human comparisons
- Cost: $50K-$500K
- **Who does this**: Companies needing safe, aligned models

**Fine-Tuning Methods:**
- **Full Fine-Tuning**: Update all parameters (expensive, requires 100K+ examples)
- **LoRA/QLoRA**: Train low-rank adapters (~0.1% parameters, 90% memory savings)
- **Prompt Tuning**: Add trainable soft prompts (rare in production)

**Interview Insight**: As an AI Engineer, you'll mostly do fine-tuning (LoRA), not pre-training.

---

## 6. Embeddings

**Definition**: Dense vector representations of text that capture semantic meaning. Similar text → similar vectors.

**How it Works:**
- Text → Embedding Model → Vector (e.g., 1536 dimensions for OpenAI `text-embedding-3-small`)
- Similarity measured by cosine similarity or dot product

**Use Cases:**
- **Semantic Search**: Find documents similar to query
- **RAG**: Retrieve relevant context for LLM
- **Clustering**: Group similar documents
- **Recommendations**: Find similar items

**Popular Embedding Models:**
- **OpenAI `text-embedding-3-small`**: 1536 dims, $0.02/1M tokens, fast
- **OpenAI `text-embedding-3-large`**: 3072 dims, $0.13/1M tokens, best quality
- **Sentence-Transformers**: Open-source (all-MiniLM, BGE), free
- **Voyage AI**: Specialized for long documents

**Key Metrics:**
- **Dimensions**: Higher = more expressive but slower search (512-3072 typical)
- **Cost**: $0.02-$0.13 per 1M tokens
- **Latency**: ~10-50ms per embedding

**Interview Tip**: Embeddings are the foundation of RAG. Know how to generate, store, and query them.

---

## 7. Vector Databases

**Definition**: Databases optimized for storing and querying high-dimensional vectors (embeddings) using similarity search.

**Why Needed:**
- Traditional DBs (Postgres) are slow for similarity search (O(n) scan)
- Vector DBs use indexing (HNSW, IVF) for fast approximate nearest neighbor search (O(log n))

**Popular Vector Databases:**
- **Pinecone**: Managed, serverless, easy, $0.096/GB/month
- **Weaviate**: Open-source, hybrid search (vector + keyword), self-hosted
- **Qdrant**: Open-source, Rust-based, fast, self-hosted
- **Chroma**: Lightweight, in-memory, great for prototyping
- **pgvector**: Postgres extension, good for small-scale

**Key Features:**
- **Indexing**: HNSW (accuracy), IVF (speed), FAISS (CPU)
- **Hybrid Search**: Combine vector similarity + keyword matching
- **Filtering**: Metadata filters (e.g., "date > 2024 AND category = 'finance'")
- **Multi-tenancy**: Isolate data per user/customer

**Performance:**
- **Latency**: 10-50ms for top-10 search across 1M vectors
- **Scale**: 100M+ vectors typical, billions possible
- **Cost**: $0.05-$0.20 per GB per month

**Interview Tip**: For RAG, you need a vector DB. Know trade-offs: Managed (Pinecone) vs self-hosted (Weaviate/Qdrant).

---

## 8. RAG (Retrieval-Augmented Generation)

**Definition**: Enhance LLM responses by retrieving relevant context from external knowledge base before generating answer.

**How RAG Works:**
1. **Indexing** (offline): Chunk documents → Generate embeddings → Store in vector DB
2. **Retrieval** (query time): Query → Embed query → Search vector DB → Top-K relevant chunks
3. **Generation**: Inject retrieved chunks into LLM prompt → Generate answer

**Architecture:**
```
User Query → Embed → Vector DB Search → Top-K Chunks → LLM Prompt → Answer
```

**Benefits:**
- **Grounding**: Reduce hallucinations with factual context
- **Up-to-date**: Add new knowledge without retraining
- **Citations**: Source attribution for answers
- **Cost**: Cheaper than fine-tuning for knowledge updates

**Key Techniques:**
- **Chunking**: Fixed-size (500 tokens), semantic (paragraphs), hybrid
- **Hybrid Search**: Combine vector similarity + keyword (BM25)
- **Reranking**: Use cross-encoder to reorder top-K results (Cohere Rerank, ColBERT)
- **Query Expansion**: Rephrase query multiple ways, merge results
- **Metadata Filtering**: Filter by date, author, category before vector search

**Performance:**
- **Latency**: 200-500ms (100ms retrieval + 300ms generation)
- **Cost**: $0.01-$0.05 per query (embedding + LLM + vector DB)
- **Quality**: 30-50% better than base LLM on domain-specific questions

**Interview Tip**: RAG is the #1 GenAI application. Know advanced techniques (hybrid search, reranking, chunking strategies).

---

## 9. Prompt Engineering

**Definition**: The art of crafting effective prompts to get desired outputs from LLMs without fine-tuning.

**Core Principles:**
1. **Be Specific**: Clear instructions + context + format
2. **Use Examples**: Few-shot learning (show 2-3 examples)
3. **Think Step-by-Step**: Chain-of-thought prompting
4. **Constrain Output**: JSON format, word limits, tone

**Key Techniques:**

**1. Zero-Shot Prompting**
```
Classify sentiment: "I love this product!" → Positive
```

**2. Few-Shot Prompting**
```
Q: What is 2+2? A: 4
Q: What is 3+5? A: 8
Q: What is 7+6? A: ?
```

**3. Chain-of-Thought (CoT)**
```
Solve step-by-step:
1. First, identify...
2. Then, calculate...
3. Finally, conclude...
```

**4. ReAct (Reasoning + Acting)**
```
Thought: I need to find the population of Paris
Action: search("Paris population")
Observation: 2.1 million
Thought: Now I can answer
Answer: Paris has 2.1 million people
```

**5. System Prompts**
```
System: You are a helpful assistant. Be concise and accurate.
User: What is photosynthesis?
```

**Advanced Patterns:**
- **Role Prompting**: "You are an expert Python developer..."
- **Temperature Control**: 0.0 (deterministic), 0.7 (balanced), 1.5 (creative)
- **Output Formatting**: "Respond in JSON: {\"answer\": \"...\", \"confidence\": 0.9}"
- **Negative Prompting**: "Do NOT include..."

**Interview Tip**: Prompt engineering is 80% of AI Engineering. Show examples of how you iterated prompts to improve quality.

---

## 10. AI Agents

**Definition**: Autonomous systems that use LLMs to reason, plan, and take actions using tools to achieve goals.

**Key Components:**
1. **LLM Brain**: Reasoning and decision-making
2. **Tools/Functions**: APIs, databases, search, calculators
3. **Memory**: Conversation history, task state
4. **Planning**: Break goals into steps
5. **Execution**: Call tools, process results, iterate

**Agent Patterns:**

**1. ReAct Agent** (Most Common)
```
Thought: What do I need to do?
Action: Use tool X with input Y
Observation: Tool returned Z
Thought: Based on Z, I should...
Action: Use tool A with input B
...
Final Answer: Here's the result
```

**2. Plan-and-Execute**
- Plan all steps upfront → Execute sequentially
- Better for complex multi-step tasks

**3. Multi-Agent Systems**
- Multiple specialized agents (Researcher, Writer, Critic)
- Coordinate to solve complex problems

**Tools/Functions:**
- **Search**: Web search (Bing, Google), database queries
- **Code Execution**: Python interpreter (execute calculations)
- **APIs**: Weather, stock prices, CRM systems
- **File Operations**: Read/write documents

**Frameworks:**
- **LangGraph**: Build stateful agents with graphs (Anthropic-backed)
- **AutoGPT**: Autonomous task completion (early framework)
- **CrewAI**: Multi-agent collaboration
- **Semantic Kernel**: Microsoft's agent framework

**Challenges:**
- **Reliability**: Agents fail ~30-50% on complex tasks
- **Cost**: Multiple LLM calls ($0.10-$1 per task)
- **Latency**: 5-30 seconds for multi-step tasks
- **Safety**: Uncontrolled tool usage, prompt injection

**Interview Tip**: Agents are the future of GenAI. Know ReAct pattern and tool calling (function calling).

---

## 11. AI vs AGI

**AI (Artificial Intelligence):**
- **Definition**: Narrow intelligence for specific tasks
- **Examples**: ChatGPT (text), DALL-E (images), GPT-4 (reasoning)
- **Capabilities**: Superhuman at narrow tasks (translation, chess, image recognition)
- **Limitations**: Can't transfer knowledge across domains, no consciousness, no general reasoning
- **Status**: We have AI today (2026)

**AGI (Artificial General Intelligence):**
- **Definition**: Human-level intelligence across ALL domains
- **Capabilities**: 
  - Learn any task a human can learn
  - Transfer knowledge across domains
  - Reason, plan, and adapt to new situations
  - Self-improve and innovate
- **Tests**: Pass Turing Test, solve novel problems without training, common sense reasoning
- **Status**: Not achieved yet (experts predict 2030-2050)

**Key Differences:**

| Aspect | AI (Today) | AGI (Future) |
|--------|-----------|--------------|
| **Scope** | Narrow (one task) | General (any task) |
| **Learning** | Requires training data | Learn from few examples like humans |
| **Transfer** | Doesn't transfer | Transfers knowledge across domains |
| **Reasoning** | Pattern matching | True understanding and reasoning |
| **Examples** | GPT-4, Claude, Llama | None exist yet |

**Current State (2026):**
- **GPT-4, Claude 3.5**: Very capable but still narrow AI
- **Emergent abilities**: CoT reasoning, planning suggest progress toward AGI
- **Limitations**: Still fail at common sense, novel situations, and true understanding

**Interview Insight**: When asked "Do we have AGI?", answer is **NO**. We have increasingly powerful narrow AI, but no system can match human general intelligence across all domains yet.

---

## 12. Transformer Architecture

**What interviewers care about:**
- Self-attention mechanism enables parallel processing and long-range dependencies
- Multi-head attention captures different representation subspaces
- Positional encoding: absolute (sinusoidal) vs relative vs learned
- Encoder-only (BERT), Decoder-only (GPT), Encoder-Decoder (T5)

**Key insight**: GPT models dominate GenAI because autoregressive generation is more flexible than masked language modeling.

## 2. Model Sizes & Tradeoffs

| Model Size | Parameters | Use Case | Latency | Cost |
|------------|------------|----------|---------|------|
| Small | 7B-13B | Edge, low-latency | ~100ms | Low |
| Medium | 30B-70B | Balanced quality | ~500ms | Medium |
| Large | 175B+ | Max quality | 1-3s | High |

**Production reality**: Most companies use 7B-13B models with fine-tuning vs 175B+ base models.

## 3. Inference Optimization

**Techniques you should know:**
- **KV-cache**: Cache key-value pairs from previous tokens (~30% speedup)
- **Quantization**: INT8/INT4 reduces memory 2-4x (GPTQ, AWQ, GGUF formats)
- **Batching**: Dynamic batching increases throughput 3-10x
- **Speculative decoding**: Small model drafts, large model verifies (2x speedup)
- **Flash Attention**: Memory-efficient attention (2-4x faster)

**Cost insight**: Inference costs dominate at scale. A 1000-token response on GPT-4 costs ~$0.06.

## 4. Fine-Tuning Methods

### Full Fine-Tuning
- Updates all parameters
- Requires: 100K+ examples, large compute
- Use case: Domain-specific models (legal, medical)

### Parameter-Efficient Fine-Tuning (PEFT)
- **LoRA/QLoRA**: Train low-rank matrices (~0.1% parameters)
  - Reduces memory 90%, trains 3x faster
  - Swappable adapters for multi-task models
- **Prefix Tuning**: Prepend trainable embeddings
- **Prompt Tuning**: Soft prompts (rare in production)

**When to fine-tune**:
- Base model consistently fails on domain-specific tasks
- Need consistent output format (structured JSON, code)
- Privacy requirements (can't send data to API)

## 5. Training Dynamics

**Concepts for senior candidates:**
- **Pre-training**: Self-supervised on massive text (1-10T tokens)
- **Instruction tuning**: Teach to follow instructions (100K examples)
- **RLHF**: Align with human preferences
  - Reward model trained on human comparisons
  - PPO optimization (unstable, being replaced by DPO)
- **DPO** (Direct Preference Optimization): Simpler RLHF alternative

**Scaling laws**: Performance scales predictably with compute, data, and parameters. Chinchilla paper: optimal ratio is ~20 tokens per parameter.

## 6. Context Window Management

**Practical limits:**
- 4K tokens: Legacy models (GPT-3.5 early)
- 8K-32K: Standard (GPT-4, Claude)
- 100K-200K: Long context (Claude 3, GPT-4 Turbo)
- 1M+: Gemini 1.5 (experimental, high latency)

**Production challenge**: Cost scales linearly with context length. 100K context = 25x more expensive than 4K.

**Strategies:**
- Intelligent chunking (semantic, sliding window)
- Context compression (LLMLingua, reranking)
- Hybrid: RAG + long context for top-K results

## 7. Common Failure Modes

1. **Hallucinations**: Model generates plausible but false information
   - Mitigation: RAG, citations, confidence scores
2. **Catastrophic forgetting**: Fine-tuning loses pre-trained knowledge
   - Mitigation: Mix pre-training data, replay techniques
3. **Recency bias**: Weights recent context over earlier tokens
   - Mitigation: Summarization, re-ranking
4. **Instruction drift**: Model ignores system prompt over long conversations
   - Mitigation: Periodic re-injection, conversation summarization

## 8. Token Economics

**Tokenization nuances:**
- Subword tokens (BPE, WordPiece): "unhappiness" = ["un", "happiness"]
- Different models have different tokenizers (GPT vs Claude vs Llama)
- Code is 2-3x more tokens than natural language
- Non-English languages: 2-5x more tokens

**Cost implications**:
- Input tokens: $0.01-0.03 per 1K (GPT-4)
- Output tokens: 2-3x input cost (generation is more expensive)
- Embeddings: ~$0.0001 per 1K tokens

## 9. Emergent Abilities

**Abilities that appear at scale (>100B params):**
- Chain-of-thought reasoning
- Few-shot learning quality jump
- Instruction following without fine-tuning
- Multi-step problem solving

**Interview insight**: Smaller models can achieve these with fine-tuning or prompt engineering.
