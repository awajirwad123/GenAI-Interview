# Scenario-Based & Architecture Questions - GenAI System Design

**Focus:** Real-world system design scenarios for chatbots, agentic RAG, and production GenAI applications

---

## Scenario 1: Enterprise Customer Support Chatbot

### Problem Statement
Design a customer support chatbot for a SaaS company with:
- 50,000 customers
- 10,000 support tickets/month
- 200,000 documents (product docs, tickets, FAQs)
- Multi-tenant (customers can't see each other's data)
- Requirements: <2s response time, 99.9% uptime, <$5K/month

### Architecture Design

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer                             │
│  Web App | Mobile App | Slack | Teams | Email               │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  API Gateway (Rate Limiting)                 │
│              Auth | Logging | Request Routing                │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Intent Classification (Fast)                    │
│          ┌──────────┬──────────┬──────────┐                │
│          │ Greeting │   FAQ    │ Complex  │                │
│          │ (Cached) │(GPT-3.5) │ (GPT-4)  │                │
└──────────┴────┬─────┴────┬─────┴────┬─────┴────────────────┘
                │          │          │
┌───────────────▼──────────▼──────────▼──────────────────────┐
│                  Conversation Manager                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Session Store (Redis)                              │   │
│  │  - User context                                     │   │
│  │  - Conversation history                             │   │
│  │  - Customer metadata                                │   │
│  └─────────────────────────────────────────────────────┘   │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                    RAG Pipeline                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 1. Query Preprocessing                               │  │
│  │    - Extract entities (customer_id, ticket_id)       │  │
│  │    - Query expansion                                 │  │
│  │ 2. Multi-tenant Vector Search                        │  │
│  │    - Filter by customer_id (isolation)               │  │
│  │    - Hybrid search (vector + keyword)                │  │
│  │ 3. Reranking (top 50 → 5)                           │  │
│  │ 4. Context Assembly                                  │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│               LLM Generation Layer                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Prompt Template Engine                               │  │
│  │ + Retrieved Context                                  │  │
│  │ + Conversation History                               │  │
│  │ + Customer Metadata                                  │  │
│  └────────────────────┬─────────────────────────────────┘  │
│                       │                                      │
│  ┌────────────────────▼─────────────────────────────────┐  │
│  │ Model Router                                         │  │
│  │  - 70% GPT-3.5-turbo (simple)                       │  │
│  │  - 25% GPT-4 (complex)                              │  │
│  │  - 5% Human escalation                              │  │
│  └────────────────────┬─────────────────────────────────┘  │
└────────────────────────┼────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│               Post-Processing Layer                          │
│  - Content filtering                                         │
│  - PII redaction                                            │
│  - Citation formatting                                       │
│  - Confidence scoring                                        │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Semantic Cache (Redis)                          │
│  Cache key: embed(query + customer_context)                 │
│  TTL: 24 hours                                              │
│  Hit rate: 35-40%                                           │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Response + Feedback Loop                        │
│  - Log interaction                                          │
│  - Track metrics                                            │
│  - Collect user feedback                                    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  Background Services                         │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Document    │  │ Embeddings   │  │ Analytics &      │  │
│  │ Ingestion   │  │ Generator    │  │ Monitoring       │  │
│  │ Pipeline    │  │ (Async)      │  │ Dashboard        │  │
│  └─────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Key Design Decisions

**1. Multi-tenant Isolation**
```python
# Vector search with customer filter
def search(query, customer_id):
    query_embedding = embed(query)
    
    results = vector_db.search(
        vector=query_embedding,
        filter={
            "customer_id": customer_id  # Critical: tenant isolation
        },
        k=50
    )
    
    return results
```

**2. Tiered Model Routing**
```python
def route_to_model(query, intent_classification):
    # Classify intent
    if intent_classification["type"] in ["greeting", "goodbye", "thanks"]:
        # Use cached responses
        return cached_responses[intent_classification["type"]]
    
    elif intent_classification["complexity"] == "simple":
        # 70% of queries: FAQ, account info
        model = "gpt-3.5-turbo"  # $0.002/1K tokens
        
    elif intent_classification["complexity"] == "complex":
        # 25% of queries: troubleshooting, debugging
        model = "gpt-4"  # $0.03/1K tokens
        
    else:
        # 5% of queries: angry customer, edge cases
        escalate_to_human()
        return None
    
    return llm_generate(query, model)
```

**3. Conversation Context Management**
```python
class ConversationManager:
    def __init__(self, session_id, customer_id):
        self.session_id = session_id
        self.customer_id = customer_id
        self.redis = Redis()
    
    def get_context(self):
        # Get last 5 turns + summary of older
        recent_turns = self.redis.lrange(f"session:{self.session_id}", -5, -1)
        summary = self.redis.get(f"summary:{self.session_id}")
        customer_info = self.get_customer_metadata()
        
        return {
            "recent_turns": recent_turns,
            "summary": summary,
            "customer_tier": customer_info["tier"],  # Premium vs Free
            "customer_name": customer_info["name"],
            "open_tickets": customer_info["open_tickets"]
        }
    
    def add_turn(self, user_msg, bot_msg):
        # Store in Redis
        self.redis.rpush(f"session:{self.session_id}", {
            "user": user_msg,
            "bot": bot_msg,
            "timestamp": now()
        })
        
        # Update summary every 10 turns
        length = self.redis.llen(f"session:{self.session_id}")
        if length % 10 == 0:
            self.update_summary()
```

**4. Semantic Caching**
```python
def semantic_cache(query, customer_id):
    # Generate cache key
    query_embedding = embed(query)
    cache_key = f"{customer_id}:{hash(query_embedding)}"
    
    # Check for similar cached query
    cached_queries = redis.get(f"cache:{customer_id}:*")
    
    for cached_query_emb, cached_response in cached_queries:
        similarity = cosine_similarity(query_embedding, cached_query_emb)
        
        if similarity > 0.95:  # Very similar
            logger.info(f"Cache hit! Saved ${cost_per_query}")
            return cached_response
    
    # Cache miss: generate and cache
    response = generate_response(query)
    redis.setex(
        f"cache:{customer_id}:{cache_key}",
        ttl=86400,  # 24 hours
        value=(query_embedding, response)
    )
    
    return response
```

### Scalability & Performance

**Latency Breakdown:**
```
Total target: <2000ms

Intent classification:     80ms  (small model)
Context retrieval:         50ms  (Redis)
Vector search:            150ms  (Pinecone)
Reranking:                120ms  (Cohere rerank)
LLM generation:           800ms  (GPT-3.5) / 1500ms (GPT-4)
Post-processing:           50ms
Response formatting:       50ms
--------------------------------
Total (GPT-3.5):         1300ms ✅
Total (GPT-4):           1800ms ✅
Cache hit:                180ms ✅✅
```

**Cost Analysis:**
```python
# Monthly cost calculation
monthly_queries = 10000  # tickets
avg_turns_per_session = 4

total_interactions = monthly_queries * avg_turns_per_session = 40,000

# Model costs
gpt35_cost = 40000 * 0.70 * $0.004 = $112
gpt4_cost = 40000 * 0.25 * $0.03 = $300
human_escalation = 40000 * 0.05 * $0 = $0 (internal)

# Infrastructure costs
vector_db = $200  # 200K vectors
redis_cache = $100
embedding_api = $50
monitoring = $50

# With caching (40% hit rate)
effective_llm_cost = ($112 + $300) * 0.6 = $247

Total: $247 + $400 = $647/month ✅ (under $5K budget)
```

### Interview Questions & Answers

**Q1: How do you ensure customer data isolation in multi-tenant system?**

**Answer:**
```python
# Three-layer isolation strategy

# 1. Database level: Separate indices per customer
vector_db_config = {
    "index_name": f"customer_{customer_id}_docs",
    "isolation": "physical"
}

# 2. Query level: Mandatory filters
def search(query, customer_id):
    if not customer_id:
        raise SecurityError("Customer ID required")
    
    return vector_db.search(
        query,
        filter={"customer_id": {"$eq": customer_id}},
        enforce_filter=True  # Cannot be bypassed
    )

# 3. Application level: Auth middleware
@require_auth
def chat_endpoint(request):
    customer_id = request.user.customer_id
    # All operations scoped to this customer
    
# 4. Audit logging
audit_log.record({
    "customer_id": customer_id,
    "query": query,
    "documents_accessed": [doc.id for doc in results],
    "timestamp": now()
})
```

**Q2: How do you handle conversation context when it exceeds token limits?**

**Answer:**
Use hierarchical memory:
```python
class HierarchicalMemory:
    def __init__(self):
        self.immediate = []      # Last 3 turns (verbatim)
        self.short_term = ""     # Last 10 turns (summary)
        self.entities = {}       # Tracked entities
    
    def build_context(self, current_query):
        context = []
        
        # 1. Entity memory (always include)
        context.append(f"Customer: {self.entities['customer_name']}")
        context.append(f"Issue: {self.entities['current_issue']}")
        
        # 2. Short-term summary
        if self.short_term:
            context.append(f"Previous discussion: {self.short_term}")
        
        # 3. Recent turns (verbatim)
        for turn in self.immediate:
            context.append(f"User: {turn['user']}")
            context.append(f"Assistant: {turn['assistant']}")
        
        # 4. Retrieve relevant past context
        if count_tokens(context) < 2000:
            relevant = self.vector_search(current_query, k=2)
            context.extend(relevant)
        
        return "\n".join(context)
```

**Q3: What happens when the system experiences high load (10× traffic spike)?**

**Answer:**
Implement graceful degradation:
```python
class LoadManager:
    def handle_request(self, query, customer_id):
        current_load = self.get_current_qps()
        
        if current_load < 100:  # Normal
            return self.full_pipeline(query, customer_id)
        
        elif current_load < 300:  # High load
            # Skip reranking
            return self.fast_pipeline(query, customer_id)
        
        elif current_load < 500:  # Very high
            # Use only cache + simple responses
            cached = self.semantic_cache.get(query)
            if cached:
                return cached
            return self.template_response(query)
        
        else:  # Overload
            # Queue request
            return {
                "message": "High demand. You're in queue.",
                "position": self.queue.add(query, customer_id),
                "eta_seconds": 30
            }
```

**Q4: How do you handle model failures or API outages?**

**Answer:**
Multi-layer fallback strategy:
```python
class ResilientLLM:
    def __init__(self):
        self.primary = OpenAIClient()
        self.secondary = AnthropicClient()
        self.tertiary = LocalModelClient()
        self.cache = SemanticCache()
    
    def generate(self, prompt, max_retries=3):
        # Layer 1: Check cache
        if cached := self.cache.get(prompt):
            return cached
        
        # Layer 2: Try primary (OpenAI)
        try:
            response = self.primary.generate(prompt, timeout=3)
            return response
        except (RateLimitError, TimeoutError) as e:
            logger.warning(f"Primary failed: {e}")
        
        # Layer 3: Try secondary (Anthropic)
        try:
            response = self.secondary.generate(prompt, timeout=5)
            return response
        except Exception as e:
            logger.error(f"Secondary failed: {e}")
        
        # Layer 4: Local model (degraded quality)
        try:
            response = self.tertiary.generate(prompt)
            return response + "\n[Using backup model]"
        except Exception as e:
            logger.critical(f"All models failed: {e}")
        
        # Layer 5: Static fallback
        return {
            "message": "I'm experiencing technical difficulties. A human agent will help you shortly.",
            "escalate": True
        }
```

---

## Scenario 2: Agentic RAG for Research Assistant

### Problem Statement
Build an AI research assistant that:
- Answers complex questions requiring multi-step reasoning
- Searches multiple data sources (internal docs, web, databases)
- Plans its own retrieval strategy
- Self-corrects when information is insufficient
- Target: PhD-level accuracy, <30s response time

### Architecture Design

```
┌─────────────────────────────────────────────────────────────┐
│                      User Query                              │
│  "What are the latest treatments for Alzheimer's disease    │
│   approved in the last 2 years?"                            │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Query Understanding (LLM)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 1. Extract requirements:                             │  │
│  │    - Time constraint: "last 2 years"                 │  │
│  │    - Domain: Medical                                 │  │
│  │    - Type: Factual + Recent                          │  │
│  │ 2. Decompose into sub-questions:                     │  │
│  │    - What is Alzheimer's disease?                    │  │
│  │    - What treatments exist?                          │  │
│  │    - Which approved 2024-2026?                       │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│           Agentic Planning System (ReAct)                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Planning Agent:                                      │  │
│  │ "I need to:                                          │  │
│  │  1. Search medical knowledge base for background    │  │
│  │  2. Search FDA database for recent approvals        │  │
│  │  3. Search PubMed for clinical trials               │  │
│  │  4. Verify information across sources               │  │
│  │  5. Synthesize findings"                            │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│               Agent Execution Loop                           │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ ITERATION 1                                          │  │
│  │ Thought: "Need background on Alzheimer's"           │  │
│  │ Action: search_knowledge_base("Alzheimer's")        │  │
│  │ Observation: [Retrieved 10 docs about disease]      │  │
│  │ Reflection: "Good background, now need treatments"  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ ITERATION 2                                          │  │
│  │ Thought: "Need FDA approval data"                   │  │
│  │ Action: search_fda_db(                              │  │
│  │     query="Alzheimer treatment",                    │  │
│  │     date_range="2024-01-01 to 2026-01-24"          │  │
│  │ )                                                    │  │
│  │ Observation: [Found 3 approved drugs: Lecanemab,   │  │
│  │               Donanemab, Aducanumab]                │  │
│  │ Reflection: "Found treatments, need more details"   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ ITERATION 3                                          │  │
│  │ Thought: "Get clinical trial data for each drug"    │  │
│  │ Action: search_pubmed("Lecanemab clinical trial")   │  │
│  │ Observation: [Retrieved 15 papers]                  │  │
│  │ Reflection: "Good data, repeat for other drugs"     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ ITERATION 4                                          │  │
│  │ Thought: "Verify information is consistent"         │  │
│  │ Action: cross_verify(sources=[FDA, PubMed, KB])     │  │
│  │ Observation: [All sources agree on 3 drugs]         │  │
│  │ Reflection: "Information verified, ready to answer" │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ ITERATION 5                                          │  │
│  │ Thought: "Have all information needed"              │  │
│  │ Action: FINISH                                       │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Answer Synthesis                                │
│  - Combine findings from all sources                        │
│  - Generate comprehensive answer                            │
│  - Add citations                                            │
│  - Include confidence scores                                │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│            Quality Verification                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 1. Fact-check: Are all claims verified?             │  │
│  │ 2. Completeness: Did we answer all parts?           │  │
│  │ 3. Recency: Is information current?                 │  │
│  │ 4. Citations: Are sources properly cited?           │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
                  [Answer]
```

### Implementation: Agentic RAG System

```python
class AgenticRAG:
    def __init__(self):
        self.llm = GPT4()
        self.tools = {
            "search_knowledge_base": self.search_kb,
            "search_fda": self.search_fda,
            "search_pubmed": self.search_pubmed,
            "search_web": self.search_web,
            "cross_verify": self.cross_verify,
            "calculate": self.calculate
        }
        self.max_iterations = 10
    
    def run(self, query):
        # Step 1: Decompose query
        sub_questions = self.decompose_query(query)
        
        # Step 2: Plan retrieval strategy
        plan = self.create_plan(query, sub_questions)
        
        # Step 3: Execute plan with ReAct loop
        evidence = []
        for iteration in range(self.max_iterations):
            # Thought: What should I do next?
            thought = self.llm.generate(f"""
You are a research assistant. You have gathered:
{evidence}

Original question: {query}
Plan: {plan}

What should you do next?
Available tools: {list(self.tools.keys())}

Respond in format:
Thought: <your reasoning>
Action: <tool_name>(<parameters>)
""")
            
            # Parse thought and action
            thought_text, action = self.parse_react_output(thought)
            
            # Check if done
            if "FINISH" in action:
                break
            
            # Execute action
            tool_name, tool_params = self.parse_action(action)
            observation = self.tools[tool_name](**tool_params)
            
            # Reflect: Is this information sufficient?
            reflection = self.llm.generate(f"""
Observation: {observation}
Is this information:
1. Relevant to the question?
2. Sufficient to answer?
3. Trustworthy?

Should we:
a) Continue gathering information
b) Verify this information
c) Ready to answer
""")
            
            # Store evidence
            evidence.append({
                "iteration": iteration,
                "thought": thought_text,
                "action": action,
                "observation": observation,
                "reflection": reflection
            })
            
            # Self-correction: Check if we're on track
            if "verify" in reflection.lower():
                verified = self.cross_verify(evidence)
                evidence.append(verified)
        
        # Step 4: Synthesize answer
        answer = self.synthesize_answer(query, evidence)
        
        # Step 5: Verify quality
        quality_check = self.verify_answer(query, answer, evidence)
        
        return {
            "answer": answer,
            "evidence": evidence,
            "sources": self.extract_sources(evidence),
            "confidence": quality_check["confidence"],
            "reasoning_path": [e["thought"] for e in evidence]
        }
    
    def decompose_query(self, query):
        """Break complex query into sub-questions"""
        prompt = f"""
Break this complex question into simpler sub-questions:
"{query}"

Generate 3-5 sub-questions that must be answered to address the main question.
"""
        return self.llm.generate(prompt)
    
    def create_plan(self, query, sub_questions):
        """Generate retrieval strategy"""
        prompt = f"""
Create a search strategy to answer: "{query}"

Sub-questions: {sub_questions}

Available data sources:
- knowledge_base: Internal documents
- fda: FDA drug approval database
- pubmed: Medical research papers
- web: General web search

Plan the optimal sequence of searches.
"""
        return self.llm.generate(prompt)
    
    def search_kb(self, query, k=10):
        """Search internal knowledge base"""
        embedding = embed(query)
        results = vector_db.search(
            embedding,
            k=k,
            filter={"verified": True}
        )
        return results
    
    def search_fda(self, query, date_range=None):
        """Search FDA approval database"""
        # Simulated FDA API call
        results = fda_api.search(
            query=query,
            date_range=date_range
        )
        return results
    
    def cross_verify(self, evidence):
        """Verify claims across multiple sources"""
        claims = self.extract_claims(evidence)
        
        verified_claims = []
        for claim in claims:
            sources = [e for e in evidence if claim in e["observation"]]
            
            if len(sources) >= 2:  # Confirmed by 2+ sources
                verified_claims.append({
                    "claim": claim,
                    "sources": sources,
                    "confidence": "high"
                })
        
        return verified_claims
    
    def synthesize_answer(self, query, evidence):
        """Generate final answer from evidence"""
        prompt = f"""
Original question: {query}

Evidence gathered:
{self.format_evidence(evidence)}

Generate a comprehensive, well-cited answer.
Include:
1. Direct answer to the question
2. Supporting details
3. Citations [1], [2], etc.
4. Any caveats or limitations
"""
        return self.llm.generate(prompt)
    
    def verify_answer(self, query, answer, evidence):
        """Quality check the generated answer"""
        checks = {
            "factual_accuracy": self.check_facts(answer, evidence),
            "completeness": self.check_completeness(query, answer),
            "recency": self.check_recency(evidence),
            "citation_quality": self.check_citations(answer, evidence)
        }
        
        confidence = sum(checks.values()) / len(checks)
        
        return {
            "checks": checks,
            "confidence": confidence,
            "issues": [k for k, v in checks.items() if v < 0.8]
        }
```

### Key Differentiators: Agentic vs Traditional RAG

| Feature | Traditional RAG | Agentic RAG |
|---------|----------------|-------------|
| **Retrieval** | Single fixed retrieval | Multi-step adaptive retrieval |
| **Planning** | No planning | Agent plans search strategy |
| **Data sources** | One vector DB | Multiple sources (DBs, APIs, web) |
| **Iteration** | One-shot | Iterative refinement |
| **Self-correction** | No | Verifies and corrects |
| **Complexity** | Simple questions | Multi-hop reasoning |
| **Latency** | 1-2s | 10-30s |
| **Cost** | $0.01/query | $0.05-0.20/query |

### Interview Questions & Answers

**Q1: How do you prevent the agent from getting stuck in infinite loops?**

**Answer:**
```python
class SafeAgentExecutor:
    def __init__(self, max_iterations=10):
        self.max_iterations = max_iterations
        self.action_history = []
        self.loop_detector = LoopDetector()
    
    def execute(self, query):
        for i in range(self.max_iterations):
            action = self.agent.next_action()
            
            # Check 1: Max iterations
            if i >= self.max_iterations:
                logger.warning("Max iterations reached")
                return self.synthesize_partial_answer()
            
            # Check 2: Repeated actions
            if self.loop_detector.is_repeating(action, self.action_history):
                logger.warning("Loop detected")
                # Break loop: Try different approach
                action = self.agent.next_action(
                    constraint="Do not repeat previous actions"
                )
            
            # Check 3: No progress
            if not self.is_making_progress():
                logger.warning("No progress in last 3 iterations")
                return self.fallback_to_simple_rag()
            
            # Execute action
            result = self.execute_action(action)
            self.action_history.append((action, result))
            
            # Check if done
            if self.is_complete():
                break
        
        return self.synthesize_answer()
    
    def is_making_progress(self):
        """Check if new information is being gathered"""
        if len(self.action_history) < 3:
            return True
        
        recent_results = [r for _, r in self.action_history[-3:]]
        # Check if results are substantially different
        similarities = [
            similarity(recent_results[i], recent_results[i+1])
            for i in range(len(recent_results)-1)
        ]
        
        return any(sim < 0.9 for sim in similarities)
```

**Q2: How do you handle conflicting information from different sources?**

**Answer:**
```python
class ConflictResolver:
    def resolve_conflicts(self, claims):
        """Handle conflicting information"""
        # Group by claim topic
        claim_groups = self.group_similar_claims(claims)
        
        resolved = []
        for topic, conflicting_claims in claim_groups.items():
            if len(conflicting_claims) == 1:
                resolved.append(conflicting_claims[0])
                continue
            
            # Strategy 1: Source credibility
            credibility_scores = [
                self.source_credibility(claim["source"])
                for claim in conflicting_claims
            ]
            
            if max(credibility_scores) > 0.9:
                # Trust highest credibility source
                best_idx = credibility_scores.index(max(credibility_scores))
                resolved.append(conflicting_claims[best_idx])
                continue
            
            # Strategy 2: Majority vote
            claim_counts = Counter([c["text"] for c in conflicting_claims])
            most_common = claim_counts.most_common(1)[0]
            
            if most_common[1] >= 2:  # At least 2 sources agree
                resolved.append({
                    "claim": most_common[0],
                    "support_count": most_common[1],
                    "confidence": "medium"
                })
                continue
            
            # Strategy 3: Recency (for time-sensitive info)
            if self.is_time_sensitive(topic):
                most_recent = max(
                    conflicting_claims,
                    key=lambda c: c["timestamp"]
                )
                resolved.append(most_recent)
                continue
            
            # Strategy 4: Report disagreement
            resolved.append({
                "claim": f"Sources disagree on {topic}",
                "options": conflicting_claims,
                "recommendation": "Human review needed"
            })
        
        return resolved
    
    def source_credibility(self, source):
        """Rank source credibility"""
        credibility_map = {
            "fda": 0.95,          # Official government
            "pubmed": 0.90,       # Peer-reviewed
            "medical_kb": 0.85,   # Curated
            "web_search": 0.60    # Variable quality
        }
        return credibility_map.get(source, 0.50)
```

**Q3: How do you optimize the cost of agentic RAG (reduce from $0.20 to $0.05 per query)?**

**Answer:**
```python
class CostOptimizedAgenticRAG:
    def __init__(self):
        self.cheap_model = GPT35Turbo()  # Planning & simple tasks
        self.expensive_model = GPT4()     # Complex reasoning only
        self.cache = SemanticCache()
    
    def run(self, query):
        # 1. Check cache first
        if cached := self.cache.get(query):
            return cached  # $0 cost
        
        # 2. Use cheap model for planning
        plan = self.cheap_model.create_plan(query)  # $0.002
        
        # 3. Parallel retrieval (not sequential)
        with ThreadPoolExecutor() as executor:
            futures = [
                executor.submit(self.search_kb, query),
                executor.submit(self.search_external, query)
            ]
            results = [f.result() for f in futures]
        # Saves: 2× LLM calls for sequential planning
        
        # 4. Cheap model for most iterations
        evidence = []
        for iteration in range(max_iterations):
            # Use cheap model unless complex reasoning needed
            complexity = self.assess_complexity(query, evidence)
            
            if complexity < 0.7:
                model = self.cheap_model  # $0.002/call
            else:
                model = self.expensive_model  # $0.03/call
            
            thought = model.next_action(query, evidence)
            evidence.append(self.execute(thought))
        
        # 5. Use expensive model only for final synthesis
        answer = self.expensive_model.synthesize(query, evidence)
        
        # 6. Cache result
        self.cache.set(query, answer)
        
        return answer
    
    # Cost breakdown:
    # - Planning: $0.002 (GPT-3.5)
    # - Iterations (avg 4): 4 × $0.002 = $0.008
    # - Final synthesis: $0.03 (GPT-4)
    # - Retrieval: $0.005
    # Total: ~$0.045 per query ✅
    
    # vs Original:
    # - Planning: $0.03 (GPT-4)
    # - Iterations (avg 6): 6 × $0.03 = $0.18
    # - Synthesis: $0.03
    # Total: $0.24 per query
```

---

## Scenario 3: Production-Ready Document Intelligence System

### Problem Statement
Build a production-grade document processing system:
- Process 50K documents/day (PDFs, contracts, invoices)
- Extract structured data + Answer questions
- SLA: 99.9% uptime, <5s processing per doc
- Handle failures gracefully
- Full observability and monitoring
- Cost: <$10K/month

### Production Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Ingestion Layer                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Document Upload API                                  │  │
│  │  - Rate limiting (100/min per user)                  │  │
│  │  - File validation (size, type, malware)            │  │
│  │  - Duplicate detection (hash check)                 │  │
│  └───────────────────┬──────────────────────────────────┘  │
└────────────────────────┼────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│              Message Queue (Kafka/RabbitMQ)                  │
│  - Buffering (handle spikes)                                │
│  - Retry logic (failed messages)                            │
│  - Priority queues (urgent vs normal)                       │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│             Processing Workers (Auto-scaling)                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Worker Pool (Kubernetes)                             │  │
│  │  - Min: 5 workers                                    │  │
│  │  - Max: 50 workers                                   │  │
│  │  - Scale trigger: Queue depth > 100                  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Processing Pipeline:                                 │  │
│  │                                                       │  │
│  │  1. Document Classification                          │  │
│  │     ├─ Invoice → Invoice processor                   │  │
│  │     ├─ Contract → Contract processor                 │  │
│  │     └─ Other → General processor                     │  │
│  │                                                       │  │
│  │  2. OCR (if scanned)                                 │  │
│  │     - Azure Document Intelligence                    │  │
│  │     - Tesseract (fallback)                           │  │
│  │     - Quality check                                  │  │
│  │                                                       │  │
│  │  3. Text Extraction & Cleaning                       │  │
│  │     - Remove headers/footers                         │  │
│  │     - Fix encoding issues                            │  │
│  │     - Normalize whitespace                           │  │
│  │                                                       │  │
│  │  4. Entity Extraction                                │  │
│  │     - Dates, amounts, names                          │  │
│  │     - Custom entities (contract terms)               │  │
│  │                                                       │  │
│  │  5. Chunking & Embedding                             │  │
│  │     - Semantic chunking                              │  │
│  │     - Generate embeddings                            │  │
│  │     - Store in vector DB                             │  │
│  │                                                       │  │
│  │  6. Metadata Extraction                              │  │
│  │     - Store in SQL database                          │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  Storage Layer                               │
│  ┌────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │ S3/Blob    │  │ Vector DB    │  │ PostgreSQL       │   │
│  │ (Raw docs) │  │ (Embeddings) │  │ (Metadata)       │   │
│  └────────────┘  └──────────────┘  └──────────────────┘   │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  Query Interface                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ API Gateway                                          │  │
│  │  - Authentication (JWT)                              │  │
│  │  - Rate limiting                                     │  │
│  │  - Request validation                                │  │
│  └───────────────────┬──────────────────────────────────┘  │
└────────────────────────┼────────────────────────────────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
┌─────────▼──────┐ ┌────▼─────┐ ┌─────▼──────────┐
│ Structured     │ │ Semantic │ │ LLM Q&A        │
│ Query (SQL)    │ │ Search   │ │                │
│                │ │ (Vector) │ │ (RAG)          │
└────────────────┘ └──────────┘ └────────────────┘
                     
┌─────────────────────────────────────────────────────────────┐
│              Observability & Monitoring                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Metrics (Prometheus)                                 │  │
│  │  - Processing latency (p50, p95, p99)                │  │
│  │  - Queue depth                                       │  │
│  │  - Error rate                                        │  │
│  │  - Cost per document                                 │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Logging (ELK Stack)                                  │  │
│  │  - All requests/responses                            │  │
│  │  - Error traces                                      │  │
│  │  - Audit logs                                        │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Alerting (PagerDuty)                                 │  │
│  │  - SLA violations                                    │  │
│  │  - Error spikes                                      │  │
│  │  - Cost anomalies                                    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Production Implementation

```python
class ProductionDocumentProcessor:
    def __init__(self):
        # Core services
        self.ocr = AzureDocumentIntelligence()
        self.embedder = EmbeddingService()
        self.vector_db = VectorDatabase()
        self.sql_db = PostgreSQL()
        self.storage = S3Client()
        
        # Reliability
        self.circuit_breaker = CircuitBreaker()
        self.retry_policy = RetryPolicy(max_attempts=3)
        self.dead_letter_queue = DeadLetterQueue()
        
        # Monitoring
        self.metrics = PrometheusMetrics()
        self.logger = StructuredLogger()
        self.tracer = DistributedTracer()
    
    @trace_request
    @monitor_metrics
    async def process_document(self, doc_id, doc_path):
        """
        Main processing pipeline with full error handling
        """
        start_time = time.time()
        
        try:
            # Step 1: Validate document
            await self.validate_document(doc_path)
            
            # Step 2: Check for duplicates
            if await self.is_duplicate(doc_path):
                self.logger.info(f"Duplicate document: {doc_id}")
                return {"status": "duplicate", "existing_id": existing_id}
            
            # Step 3: Extract text (with OCR if needed)
            text = await self.extract_text_with_fallback(doc_path)
            
            # Step 4: Process in parallel
            async with TaskGroup() as tg:
                # Parallel tasks
                entities_task = tg.create_task(
                    self.extract_entities(text)
                )
                chunks_task = tg.create_task(
                    self.chunk_document(text)
                )
                metadata_task = tg.create_task(
                    self.extract_metadata(text, doc_path)
                )
            
            entities = entities_task.result()
            chunks = chunks_task.result()
            metadata = metadata_task.result()
            
            # Step 5: Generate embeddings
            embeddings = await self.embed_chunks_batch(chunks)
            
            # Step 6: Store everything
            await self.store_document(
                doc_id=doc_id,
                text=text,
                chunks=chunks,
                embeddings=embeddings,
                entities=entities,
                metadata=metadata
            )
            
            # Step 7: Record metrics
            latency = time.time() - start_time
            self.metrics.record_latency(latency)
            self.metrics.increment_success_count()
            
            return {
                "status": "success",
                "doc_id": doc_id,
                "latency_ms": latency * 1000,
                "entities": entities,
                "chunk_count": len(chunks)
            }
        
        except Exception as e:
            # Comprehensive error handling
            return await self.handle_error(doc_id, doc_path, e)
    
    async def extract_text_with_fallback(self, doc_path):
        """
        Text extraction with multiple fallbacks
        """
        # Try primary OCR service
        try:
            if self.circuit_breaker.is_open("azure_ocr"):
                raise ServiceUnavailable("Circuit breaker open")
            
            text = await self.retry_policy.execute(
                self.ocr.extract_text,
                doc_path
            )
            
            self.circuit_breaker.record_success("azure_ocr")
            return text
        
        except (TimeoutError, RateLimitError) as e:
            self.circuit_breaker.record_failure("azure_ocr")
            self.logger.warning(f"Primary OCR failed: {e}")
        
        # Fallback to Tesseract
        try:
            text = await self.tesseract_ocr(doc_path)
            self.logger.info("Used fallback OCR")
            return text
        
        except Exception as e:
            self.logger.error(f"All OCR methods failed: {e}")
            
            # Last resort: Basic text extraction
            text = self.basic_text_extraction(doc_path)
            if not text or len(text) < 100:
                raise ProcessingError("Could not extract text")
            
            return text
    
    async def handle_error(self, doc_id, doc_path, error):
        """
        Comprehensive error handling
        """
        error_type = type(error).__name__
        
        # Log error
        self.logger.error(
            f"Processing failed for {doc_id}",
            extra={
                "doc_id": doc_id,
                "error_type": error_type,
                "error_message": str(error),
                "stack_trace": traceback.format_exc()
            }
        )
        
        # Record metric
        self.metrics.increment_error_count(error_type)
        
        # Classify error
        if isinstance(error, RetryableError):
            # Send to retry queue
            await self.retry_queue.push({
                "doc_id": doc_id,
                "doc_path": doc_path,
                "attempt": self.get_attempt_count(doc_id) + 1
            })
            
            return {
                "status": "retry_queued",
                "doc_id": doc_id,
                "error": error_type
            }
        
        elif isinstance(error, InvalidDocumentError):
            # Non-retryable error
            await self.mark_as_failed(doc_id, error)
            
            return {
                "status": "failed",
                "doc_id": doc_id,
                "error": error_type,
                "message": "Invalid document format"
            }
        
        else:
            # Unknown error: Send to DLQ
            await self.dead_letter_queue.push({
                "doc_id": doc_id,
                "doc_path": doc_path,
                "error": error_type,
                "timestamp": now()
            })
            
            # Alert on-call engineer
            await self.alert_oncall(
                severity="high",
                message=f"Document processing failed: {doc_id}",
                error=error
            )
            
            return {
                "status": "dlq",
                "doc_id": doc_id,
                "error": error_type
            }
    
    async def store_document(self, **kwargs):
        """
        Store document with transactional guarantees
        """
        try:
            # Start transaction
            async with self.sql_db.transaction() as tx:
                # 1. Store raw document
                await self.storage.upload(
                    kwargs["doc_id"],
                    kwargs["text"]
                )
                
                # 2. Store metadata in SQL
                await tx.execute("""
                    INSERT INTO documents (
                        id, entities, metadata, created_at
                    ) VALUES ($1, $2, $3, $4)
                """, kwargs["doc_id"], kwargs["entities"],
                     kwargs["metadata"], now())
                
                # 3. Store embeddings in vector DB
                await self.vector_db.insert_batch(
                    ids=[f"{kwargs['doc_id']}_{i}" for i in range(len(kwargs["chunks"]))],
                    vectors=kwargs["embeddings"],
                    metadata=[
                        {"doc_id": kwargs["doc_id"], "chunk_text": chunk}
                        for chunk in kwargs["chunks"]
                    ]
                )
                
                # Commit transaction
                await tx.commit()
        
        except Exception as e:
            # Rollback on any failure
            await tx.rollback()
            
            # Clean up partially stored data
            await self.cleanup_partial_data(kwargs["doc_id"])
            
            raise StorageError(f"Failed to store document: {e}")
    
    def get_health_status(self):
        """
        Health check for monitoring
        """
        return {
            "status": "healthy" if self.is_healthy() else "unhealthy",
            "checks": {
                "database": self.sql_db.ping(),
                "vector_db": self.vector_db.ping(),
                "storage": self.storage.ping(),
                "ocr_service": not self.circuit_breaker.is_open("azure_ocr"),
                "embedding_service": self.embedder.ping(),
            },
            "metrics": {
                "queue_depth": self.get_queue_depth(),
                "processing_rate": self.metrics.get_rate(),
                "error_rate": self.metrics.get_error_rate(),
                "avg_latency_ms": self.metrics.get_avg_latency(),
            },
            "timestamp": now()
        }
```

### Production Readiness Checklist

**Reliability:**
- ✅ Circuit breakers for external services
- ✅ Retry logic with exponential backoff
- ✅ Dead letter queue for failed messages
- ✅ Graceful degradation
- ✅ Health checks and readiness probes
- ✅ Auto-scaling based on load

**Monitoring:**
- ✅ Metrics: Latency (p50/p95/p99), error rate, throughput
- ✅ Distributed tracing (correlate requests across services)
- ✅ Structured logging (JSON format)
- ✅ Alerts for SLA violations
- ✅ Cost tracking per document

**Security:**
- ✅ Authentication & authorization (JWT)
- ✅ Rate limiting per user
- ✅ Input validation & sanitization
- ✅ Secrets management (Key Vault)
- ✅ Audit logging
- ✅ Encryption at rest and in transit

**Performance:**
- ✅ Async processing (non-blocking I/O)
- ✅ Parallel processing where possible
- ✅ Batch operations (embeddings, DB inserts)
- ✅ Caching (duplicate detection)
- ✅ Database indexing
- ✅ Connection pooling

**Cost Optimization:**
- ✅ Auto-scaling (scale down during low traffic)
- ✅ Spot instances for workers
- ✅ Efficient storage (compression)
- ✅ Batch API calls
- ✅ Model selection (GPT-3.5 vs GPT-4)

### Interview Questions & Answers

**Q1: How do you ensure 99.9% uptime?**

**Answer:**
```
99.9% uptime = 8.76 hours downtime/year = 43.8 minutes/month

Strategies:

1. Multi-region deployment
   - Primary: us-east-1
   - Failover: us-west-2
   - Health checks every 30s
   - Automatic failover in <2 minutes

2. Redundancy at every layer
   - Load balancer: 2+ instances
   - API servers: Min 3 instances (cross-AZ)
   - Database: Primary + read replicas
   - Vector DB: Replicated
   - Message queue: Clustered

3. Circuit breakers
   - Prevent cascade failures
   - Fast fail if service degraded
   - Automatic recovery

4. Graceful degradation
   - If OCR down → Use cached/simpler extraction
   - If vector DB slow → Return partial results
   - If LLM rate-limited → Queue requests

5. Zero-downtime deployments
   - Blue-green deployment
   - Rolling updates
   - Database migrations (backward compatible)

6. Comprehensive monitoring
   - Synthetic monitoring (test critical paths every 1 min)
   - Alert on-call within 1 minute of issue
   - Automated remediation where possible
```

**Q2: How do you handle a sudden 10× spike in traffic?**

**Answer:**
```python
class AutoScalingConfig:
    # Kubernetes HPA (Horizontal Pod Autoscaler)
    min_replicas = 5
    max_replicas = 50
    target_cpu_utilization = 70%
    target_queue_depth = 100  # messages
    
    scale_up_policy = {
        "threshold": "queue_depth > 100 OR cpu > 80%",
        "action": "Add 5 replicas",
        "cooldown": "30 seconds"
    }
    
    scale_down_policy = {
        "threshold": "queue_depth < 20 AND cpu < 40%",
        "action": "Remove 1 replica",
        "cooldown": "5 minutes"  # Slower scale-down
    }

# What happens during 10× spike:

# T+0: Traffic increases 10× (500 → 5000 requests/min)
# - Queue depth increases: 0 → 500
# - Alert: "High queue depth"

# T+30s: Auto-scaling triggers
# - Adds 5 replicas (5 → 10 workers)
# - Queue starts draining

# T+1min: Still high load
# - Adds 5 more replicas (10 → 15)

# T+2min: Queue under control
# - Queue depth: 500 → 100
# - Processing normally

# T+10min: Spike ends, traffic returns to normal
# - Workers idle
# - Gradually scale down over 30 minutes
# - Cost: Paid for burst capacity only during spike

# Cost calculation:
# Normal: 5 workers × $0.10/hr = $0.50/hr
# Spike (10 min): 15 workers × $0.10/hr × (10/60) = $0.25
# Total spike cost: $0.25 (vs rejecting customers)
```

**Q3: How do you debug when documents are processing slowly (>10s instead of <5s)?**

**Answer:**
```python
# Debugging methodology:

# Step 1: Check metrics dashboard
metrics = {
    "p50_latency": 11500ms,  # ⚠️ Above SLA
    "p95_latency": 18000ms,  # ⚠️ Way above SLA
    "p99_latency": 25000ms,
    "error_rate": 0.02,      # Normal
    "queue_depth": 250,       # ⚠️ High
}

# Step 2: Break down latency by stage
latency_breakdown = {
    "upload": 100ms,          # ✅ Normal
    "ocr": 8500ms,           # ⚠️ Problem here!
    "entity_extraction": 200ms,  # ✅ Normal
    "embedding": 500ms,       # ✅ Normal
    "storage": 300ms          # ✅ Normal
}

# Found root cause: OCR service slow (8.5s vs normal 2s)

# Step 3: Investigate OCR service
ocr_metrics = {
    "requests": 5000,
    "p95_latency": 8500ms,   # ⚠️ High
    "error_rate": 0.05,
    "rate_limits_hit": 150   # ⚠️ Getting rate limited!
}

# Root cause: Hitting Azure OCR rate limits

# Step 4: Solutions

# Option A: Increase rate limit quota
# Contact Azure support → Increase from 15 req/s to 30 req/s
# Cost: +$500/month
# Time: 2-3 days approval

# Option B: Add second OCR service (immediate)
class LoadBalancedOCR:
    def __init__(self):
        self.services = [
            AzureOCR(subscription=1),
            AzureOCR(subscription=2),  # Second subscription
            TesseractOCR()              # Fallback
        ]
        self.current = 0
    
    def extract_text(self, doc):
        # Round-robin across services
        service = self.services[self.current % len(self.services)]
        self.current += 1
        
        try:
            return service.extract(doc)
        except RateLimitError:
            # Try next service
            return self.services[(self.current + 1) % len(self.services)].extract(doc)

# Result: Latency drops from 8.5s → 2.5s ✅
# Cost: +$200/month (second subscription)
# Implementation time: 30 minutes
```

---

## Additional Rapid-Fire Architecture Questions

**Q: Design a GenAI system for real-time customer sentiment analysis during support calls.**

**A:** Stream audio → Speech-to-text (Whisper) → Sentiment analysis (fine-tuned BERT) → Real-time dashboard. Alert if sentiment negative for >1 minute. Architecture: Kafka for streaming, Redis for real-time state, WebSocket for dashboard updates.

**Q: How would you build a code review assistant that suggests improvements?**

**A:** Git webhook → Parse diff → Run linters (static analysis) → Vector search for similar code patterns → LLM (GPT-4) generates suggestions → Post as PR comments. Cache suggestions for similar patterns. Cost: $0.05 per PR review.

**Q: Design a multi-tenant GenAI platform where each customer can fine-tune their own model.**

**A:** Shared infrastructure with isolated model endpoints. Use LoRA adapters (not full models) per customer. Store adapters separately, load dynamically at inference. Cost: $50-100 per customer for training, $0.10/hr per active endpoint.

**Q: How do you build a GenAI system that learns from user corrections?**

**A:** Log all interactions → Track user edits to LLM outputs → Build correction dataset → Periodic fine-tuning (weekly/monthly) → A/B test new model. Metrics: Track edit distance, user satisfaction before/after.

**Q: Architecture for a GenAI system that must work offline?**

**A:** Local LLM (Llama-2-7B quantized) → Local vector DB (Chroma) → Sync embeddings when online → Local inference on device (laptop/phone). Size: ~4GB model + vectors. Use case: Medical devices, field operations.

---

## Key Takeaways

**For Chatbot Architecture:**
1. Tiered model routing (80% cheap, 20% expensive)
2. Multi-tenant isolation at every layer
3. Conversation context management with summarization
4. Semantic caching (30-40% hit rate)
5. Intent classification before RAG

**For Agentic RAG:**
1. LLM plans retrieval strategy
2. Multi-source retrieval (not just vector DB)
3. Iterative refinement with self-correction
4. Conflict resolution across sources
5. Cost optimization (use cheap model for planning)

**For Production Systems:**
1. Circuit breakers + retries + DLQ
2. Comprehensive monitoring (metrics, logs, traces)
3. Auto-scaling based on queue depth
4. Health checks and graceful degradation
5. Transactional guarantees for data consistency

**Interview Success Tips:**
- Always discuss trade-offs (cost vs latency vs quality)
- Include specific numbers (latency targets, costs, scale)
- Address failures ("What if X service goes down?")
- Show production mindset (monitoring, security, scaling)
- Draw architecture diagrams
- Start simple, then add complexity based on requirements

---

## Scenario 4: Real-Time Content Moderation System (Social Media Platform)

### Problem Statement
Design a GenAI-powered content moderation system for a social media platform with:
- 1 million posts/day (text, images, video)
- Real-time moderation (<500ms latency requirement)
- Categories: Hate speech, violence, NSFW, misinformation, spam
- 95% precision, 98% recall required
- Handle adversarial attacks (users bypassing filters)
- Cost target: <$0.50 per 1000 posts

### Architecture Design

```
┌─────────────────────────────────────────────────────────────┐
│                     Content Ingestion                        │
│  User Posts → API Gateway → Content Queue (Kafka)           │
│  (Text | Image | Video | Audio)                             │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Content Pre-Processing Pipeline                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Text: OCR extraction from images                     │  │
│  │ Image: Hash matching (PhotoDNA, pHash)              │  │
│  │ Video: Key frame extraction (1 fps)                 │  │
│  │ Audio: Speech-to-text (if present)                  │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│            Multi-Stage Moderation Pipeline                   │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │ Stage 1: Fast Filters (Blocklist Check) - <10ms   │    │
│  │ - Exact match against known bad hashes            │    │
│  │ - Keyword blocklist                               │    │
│  │ - URL reputation check                            │    │
│  │ → Auto-reject ~30% of violations                  │    │
│  └────────────────────────────────────────────────────┘    │
│                       │                                      │
│  ┌────────────────────▼───────────────────────────────┐    │
│  │ Stage 2: ML Classifiers (Fast Models) - <100ms    │    │
│  │ - Fine-tuned DistilBERT for text                  │    │
│  │ - ResNet-50 for image classification              │    │
│  │ - Multi-label: [toxicity, nsfw, spam, etc.]      │    │
│  │ → Catch 60% of remaining violations               │    │
│  └────────────────────────────────────────────────────┘    │
│                       │                                      │
│  ┌────────────────────▼───────────────────────────────┐    │
│  │ Stage 3: LLM Reasoning (Complex Cases) - <400ms   │    │
│  │ - GPT-3.5-turbo for contextual analysis          │    │
│  │ - Check sarcasm, coded language, context         │    │
│  │ - Generate explanation for decision               │    │
│  │ → Only 10% of posts reach this stage             │    │
│  └────────────────────────────────────────────────────┘    │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Decision & Action Engine                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Confidence Thresholds:                               │  │
│  │ - High confidence (>0.95): Auto-action              │  │
│  │ - Medium (0.7-0.95): Human review queue            │  │
│  │ - Low (<0.7): Allow but flag                       │  │
│  │                                                      │  │
│  │ Actions:                                             │  │
│  │ - Remove content                                     │  │
│  │ - Shadow ban user                                    │  │
│  │ - Add warning label                                  │  │
│  │ - Notify user (appeal process)                       │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Human Review Interface                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Priority Queue: High-risk flagged content           │  │
│  │ - Show all context (user history, thread)          │  │
│  │ - Display AI confidence + reasoning                 │  │
│  │ - Track reviewer agreement rate                     │  │
│  │ - Feedback loop to improve models                   │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│          Adversarial Defense & Continuous Learning           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ - Monitor for evasion patterns (l33t speak, etc.)   │  │
│  │ - Active learning: Human corrections → retraining   │  │
│  │ - Red team testing: Generate adversarial samples    │  │
│  │ - Model updates: Weekly retraining with new data    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Implementation (Python Code)

```python
from typing import List, Dict, Tuple
from enum import Enum
import asyncio
from openai import AsyncOpenAI
from PIL import Image
import imagehash
import torch
from transformers import pipeline

class ModerationCategory(Enum):
    HATE_SPEECH = "hate_speech"
    VIOLENCE = "violence"
    NSFW = "nsfw"
    MISINFORMATION = "misinformation"
    SPAM = "spam"
    SAFE = "safe"

class ModerationDecision:
    def __init__(self, category: ModerationCategory, confidence: float, 
                 explanation: str, action: str):
        self.category = category
        self.confidence = confidence
        self.explanation = explanation
        self.action = action  # "allow", "remove", "review", "warn"

class ContentModerationSystem:
    def __init__(self):
        self.client = AsyncOpenAI()
        
        # Stage 1: Blocklists (in-memory for speed)
        self.hash_blocklist = set()  # Known bad content hashes
        self.keyword_blocklist = set()  # Exact match keywords
        
        # Stage 2: ML Models (load once, reuse)
        self.text_classifier = pipeline(
            "text-classification",
            model="distilbert-base-uncased-finetuned-moderation",
            device=0 if torch.cuda.is_available() else -1
        )
        self.image_classifier = None  # ResNet-50 loaded on demand
        
        # Stage 3: LLM (rate limited)
        self.llm_semaphore = asyncio.Semaphore(100)  # Max concurrent LLM calls
        
    async def moderate_content(self, content: Dict) -> ModerationDecision:
        """Main moderation pipeline"""
        
        # Stage 1: Fast Filters (~10ms)
        decision = await self._stage1_fast_filters(content)
        if decision:
            return decision
        
        # Stage 2: ML Classifiers (~100ms)
        decision = await self._stage2_ml_classifiers(content)
        if decision.confidence > 0.95:
            return decision
        
        # Stage 3: LLM Reasoning (only for ambiguous cases)
        if decision.confidence < 0.7:
            decision = await self._stage3_llm_reasoning(content, decision)
        
        return decision
    
    async def _stage1_fast_filters(self, content: Dict) -> ModerationDecision:
        """Exact match filters - highest speed"""
        
        # Hash matching for images/videos
        if content["type"] in ["image", "video"]:
            content_hash = self._compute_hash(content["data"])
            if content_hash in self.hash_blocklist:
                return ModerationDecision(
                    category=ModerationCategory.NSFW,
                    confidence=1.0,
                    explanation="Exact match with known violating content",
                    action="remove"
                )
        
        # Keyword blocklist for text
        if content["type"] == "text":
            text_lower = content["text"].lower()
            for keyword in self.keyword_blocklist:
                if keyword in text_lower:
                    return ModerationDecision(
                        category=ModerationCategory.HATE_SPEECH,
                        confidence=1.0,
                        explanation=f"Blocklisted keyword detected",
                        action="remove"
                    )
        
        return None
    
    async def _stage2_ml_classifiers(self, content: Dict) -> ModerationDecision:
        """Fast ML models for classification"""
        
        if content["type"] == "text":
            result = self.text_classifier(content["text"])[0]
            
            category_map = {
                "toxic": ModerationCategory.HATE_SPEECH,
                "nsfw": ModerationCategory.NSFW,
                "spam": ModerationCategory.SPAM
            }
            
            predicted_category = category_map.get(
                result["label"], 
                ModerationCategory.SAFE
            )
            
            confidence = result["score"]
            
            action = "allow"
            if confidence > 0.95:
                action = "remove"
            elif confidence > 0.7:
                action = "review"
            
            return ModerationDecision(
                category=predicted_category,
                confidence=confidence,
                explanation=f"ML classifier prediction: {result['label']}",
                action=action
            )
        
        # Similar logic for image classification with ResNet-50
        # (omitted for brevity)
        
        return ModerationDecision(
            category=ModerationCategory.SAFE,
            confidence=0.5,
            explanation="No strong signal from fast classifiers",
            action="allow"
        )
    
    async def _stage3_llm_reasoning(self, content: Dict, 
                                    ml_decision: ModerationDecision) -> ModerationDecision:
        """LLM-based contextual analysis for complex cases"""
        
        async with self.llm_semaphore:
            prompt = f"""You are a content moderator. Analyze this content for policy violations.

Content: {content["text"]}
ML Classifier says: {ml_decision.category.value} (confidence: {ml_decision.confidence:.2f})

Context:
- User history: {content.get("user_context", "No history")}
- Thread context: {content.get("thread_context", "No thread")}

Evaluate for:
1. Hate speech (including coded language, sarcasm)
2. Violence (threats, graphic content)
3. NSFW content
4. Misinformation (false claims)
5. Spam

Respond in JSON:
{{
    "category": "hate_speech|violence|nsfw|misinformation|spam|safe",
    "confidence": 0.0-1.0,
    "explanation": "Brief reasoning (1-2 sentences)",
    "action": "allow|remove|review|warn"
}}"""

            response = await self.client.chat.completions.create(
                model="gpt-3.5-turbo",
                messages=[{"role": "user", "content": prompt}],
                response_format={"type": "json_object"},
                temperature=0.1,
                max_tokens=150
            )
            
            result = eval(response.choices[0].message.content)
            
            return ModerationDecision(
                category=ModerationCategory(result["category"]),
                confidence=result["confidence"],
                explanation=result["explanation"],
                action=result["action"]
            )
    
    def _compute_hash(self, data: bytes) -> str:
        """Perceptual hash for near-duplicate detection"""
        img = Image.open(data)
        return str(imagehash.average_hash(img))

class HumanReviewQueue:
    """Priority queue for human reviewers"""
    
    def __init__(self):
        self.queue = []  # Priority heap
    
    async def add_to_review(self, content: Dict, decision: ModerationDecision):
        """Add content to human review queue"""
        
        # Priority scoring
        priority = self._calculate_priority(content, decision)
        
        review_item = {
            "content": content,
            "ai_decision": decision,
            "priority": priority,
            "timestamp": asyncio.get_event_loop().time()
        }
        
        # Insert into priority queue (use heapq in production)
        self.queue.append(review_item)
        self.queue.sort(key=lambda x: -x["priority"])
    
    def _calculate_priority(self, content: Dict, decision: ModerationDecision) -> float:
        """Higher priority = needs review sooner"""
        
        priority = 0.0
        
        # Confidence: Lower confidence = higher priority
        priority += (1 - decision.confidence) * 50
        
        # Category severity
        severity_map = {
            ModerationCategory.VIOLENCE: 100,
            ModerationCategory.HATE_SPEECH: 90,
            ModerationCategory.NSFW: 70,
            ModerationCategory.MISINFORMATION: 60,
            ModerationCategory.SPAM: 20
        }
        priority += severity_map.get(decision.category, 0)
        
        # User history: Repeat offenders = lower priority
        if content.get("user_violations", 0) > 3:
            priority -= 30
        
        # Virality: Popular posts = higher priority
        engagement = content.get("likes", 0) + content.get("shares", 0)
        priority += min(engagement / 100, 50)
        
        return priority
    
    async def get_next_for_review(self) -> Dict:
        """Get highest priority item for human reviewer"""
        if not self.queue:
            return None
        return self.queue.pop(0)

# Usage Example
async def main():
    moderator = ContentModerationSystem()
    review_queue = HumanReviewQueue()
    
    # Simulate content moderation
    content = {
        "type": "text",
        "text": "This is a test post with some borderline content...",
        "user_context": "User has 0 previous violations",
        "thread_context": "Part of political discussion",
        "likes": 150,
        "shares": 30
    }
    
    decision = await moderator.moderate_content(content)
    
    if decision.action == "remove":
        print(f"Auto-removed: {decision.explanation}")
    elif decision.action == "review":
        await review_queue.add_to_review(content, decision)
        print(f"Added to human review queue (confidence: {decision.confidence:.2f})")
    else:
        print(f"Allowed: {decision.explanation}")
```

### Cost Analysis

**1M posts/day breakdown:**

**Stage 1: Fast Filters (300K posts caught)**
- Cost: $0 (in-memory lookups, hashing)
- Latency: ~10ms

**Stage 2: ML Classifiers (600K posts reach here, 420K caught)**
- Infrastructure: GPU instances for inference
- Cost: ~$2/day for GPU compute (batch processing)
- Latency: ~100ms per post

**Stage 3: LLM Reasoning (180K posts reach here)**
- GPT-3.5-turbo: $0.0005 per post (150 tokens avg)
- Cost: 180K × $0.0005 = $90/day = $2,700/month
- Latency: ~300-400ms

**Human Review (50K posts/day with medium confidence)**
- Reviewers: 20 reviewers × $25/hr × 8 hrs = $4,000/day = $120K/month
- Each reviewer handles ~300 posts/hr

**Total Monthly Cost:**
- ML Infrastructure: $60/month
- LLM API: $2,700/month
- Human reviewers: $120,000/month
- Storage & logging: $500/month
- **Total: ~$123,260/month** for 1M posts/day

**Optimization to reduce to $50K/month:**
- Improve Stage 2 models (reduce LLM calls by 50%) → Save $1,350
- Reduce human review through active learning (30% reduction) → Save $36,000
- Use GPT-3.5-turbo-instruct (cheaper, faster) → Save $1,000
- **Optimized total: ~$85K/month**

### Scaling & Performance

**Latency Breakdown:**
- P50: 110ms (caught by Stage 2)
- P95: 450ms (reaches Stage 3 LLM)
- P99: 800ms (complex cases with retry)

**Throughput:**
- Stage 1: 100K requests/sec (in-memory)
- Stage 2: 10K requests/sec (GPU batching)
- Stage 3: 500 requests/sec (LLM API limited)

**Scaling Strategy:**
- Auto-scale GPU instances based on queue depth
- Use Redis for distributed rate limiting
- Horizontal scaling: 10 replicas handle 100K posts/sec
- Kafka for ingestion buffering (handle spikes)

### Adversarial Defense

**Common Evasion Techniques:**
1. **Leetspeak** (h8 speech) → Add character normalization
2. **Zero-width characters** → Unicode normalization
3. **Image text overlay** → OCR extraction + text moderation
4. **Context injection** ("Just kidding!") → Thread-level analysis
5. **Synonym replacement** ("unalive" for violence) → Semantic embeddings

**Defense Mechanisms:**
```python
class AdversarialDefense:
    def __init__(self):
        self.evasion_patterns = []
        self.active_learner = ActiveLearner()
    
    async def detect_evasion(self, content: str) -> bool:
        """Detect known evasion patterns"""
        
        # Normalize text
        normalized = self._normalize_text(content)
        
        # Check for character substitution patterns
        if self._has_leetspeak(normalized):
            return True
        
        # Check for zero-width characters
        if self._has_hidden_chars(content):
            return True
        
        # Semantic similarity to known bad phrases
        similarity = await self._semantic_similarity(normalized)
        if similarity > 0.85:
            return True
        
        return False
    
    def _normalize_text(self, text: str) -> str:
        """Normalize leetspeak, Unicode tricks"""
        replacements = {
            '3': 'e', '4': 'a', '1': 'i', '0': 'o',
            '@': 'a', '$': 's', '!': 'i'
        }
        for old, new in replacements.items():
            text = text.replace(old, new)
        return text
    
    async def learn_from_misses(self, missed_content: str, true_label: str):
        """Active learning from false negatives"""
        # Add to training set for next model update
        self.active_learner.add_example(missed_content, true_label)
        
        # If enough examples, trigger retraining
        if len(self.active_learner.buffer) > 1000:
            await self.active_learner.retrain_model()
```

### Interview Questions & Answers

**Q: How do you balance false positives vs false negatives in content moderation?**

**A:** It's a business decision based on platform values:
- **False negatives** (miss violations): Bad user experience, legal risk, brand damage → Set recall target at 98%
- **False positives** (remove good content): User frustration, censorship concerns → Set precision target at 95%

**Trade-off:** Use confidence thresholds:
- High confidence (>0.95): Auto-action (accept some false positives)
- Medium (0.7-0.95): Human review (catch false positives)
- Low (<0.7): Allow but flag (minimize false positives)

**Tuning:** Adjust thresholds based on category severity. Violence/CSAM → Favor recall (catch everything, accept false positives). Spam → Favor precision (don't annoy users).

**Q: How do you handle adversarial attacks where users deliberately try to bypass filters?**

**A:** Multi-layered defense:

1. **Text Normalization:** Leetspeak (h8 → hate), Unicode tricks, zero-width chars
2. **Semantic Understanding:** Use embeddings to catch synonym replacements ("unalive" for violent terms)
3. **Context Analysis:** LLM evaluates intent, not just keywords ("I hate Mondays" vs "I hate [group]")
4. **Active Learning:** Human reviewers flag misses → Retrain models weekly
5. **Red Team Testing:** Dedicated team tries to bypass filters, use failures to improve
6. **Behavioral Signals:** Users who repeatedly test boundaries get flagged

**Example:** User posts "I h@t3 th3m" (leetspeak). Normalization converts to "I hate them". Then context analysis determines if "them" refers to a protected group.

**Q: Your LLM moderation is too slow (400ms). How do you speed it up?**

**A:** Optimization strategies:

1. **Reduce LLM calls:** Only use LLM for 10-15% of posts (ambiguous cases from Stage 2)
2. **Faster model:** Use GPT-3.5-turbo-instruct (50% faster than chat models)
3. **Batch processing:** Group posts in batches of 10-20, process together
4. **Prompt caching:** Cache LLM responses for similar posts (semantic dedup)
5. **Fine-tune smaller model:** DistilGPT-3.5 on moderation data (10x faster, 90% accuracy)
6. **Speculative execution:** Start LLM call in parallel with Stage 2, cancel if not needed
7. **Edge inference:** Deploy small model (Llama-2-7B quantized) on edge for <100ms latency

**Real-world:** Combination of fine-tuned DistilBERT (Stage 2) + GPT-3.5 (Stage 3) achieves P95 latency of 450ms while maintaining 95% precision.

**Q: How do you measure the success of your moderation system?**

**A:** Metrics across 4 dimensions:

**1. Accuracy Metrics:**
- Precision: 95% (of removed content, how much was actually bad?)
- Recall: 98% (of bad content, how much did we catch?)
- F1-score: 0.965
- False negative rate: 2% (miss 20K violations/day at 1M scale)

**2. Latency Metrics:**
- P50: 110ms
- P95: 450ms
- P99: 800ms
- % meeting SLA (<500ms): 96%

**3. Cost Metrics:**
- Cost per 1K posts: $0.123 (target: <$0.50 ✓)
- LLM cost percentage: 2% (well optimized)
- Human review cost: 97% (opportunity for automation)

**4. User Experience Metrics:**
- Appeal rate: 5% of removed content
- Appeal overturn rate: 10% (catching false positives)
- Time to review: <4 hours for human queue
- User satisfaction: 7.5/10 (post-moderation survey)

**Dashboard:** Real-time Grafana dashboard tracking all metrics, alerts for anomalies (sudden spike in violations, latency degradation).

---

## Scenario 5: Personalized Learning Assistant for EdTech Platform

### Problem Statement
Design a GenAI-powered personalized learning assistant for an online education platform with:
- 100,000 active students
- 5,000 courses across 20 subjects
- Requirements: Personalized explanations, adaptive assessments, learning path recommendations
- Each student interaction: <3s response time
- Track learning progress and adjust difficulty dynamically
- Cost target: <$2 per student per month

### Architecture Design

```
┌─────────────────────────────────────────────────────────────┐
│                     Student Interface                        │
│  Web App | Mobile App | LMS Integration (Canvas/Moodle)     │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Learning Interaction Gateway                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Interaction Types:                                   │  │
│  │ - Question answering ("Explain photosynthesis")     │  │
│  │ - Problem hints ("I'm stuck on integral calculus")  │  │
│  │ - Concept clarification ("What is polymorphism?")   │  │
│  │ - Assessment feedback ("Why is my answer wrong?")   │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Student Model & Context Layer                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Student Profile (PostgreSQL):                        │  │
│  │ - Learning style (visual/auditory/kinesthetic)       │  │
│  │ - Knowledge level per topic (beginner/advanced)     │  │
│  │ - Past performance (quiz scores, time spent)        │  │
│  │ - Interaction history (questions asked, topics)     │  │
│  │ - Learning goals & preferences                       │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Knowledge Graph (Neo4j):                             │  │
│  │ - Course prerequisites (Calculus → Physics)         │  │
│  │ - Concept dependencies (Variables → Functions)      │  │
│  │ - Student mastery per concept node                  │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│               RAG Pipeline for Course Content                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 1. Retrieve Relevant Course Materials               │  │
│  │    - Query: "Explain photosynthesis for 8th grade" │  │
│  │    - Vector DB: Pinecone (course content embedded) │  │
│  │    - Hybrid search: Semantic + metadata filters    │  │
│  │      (grade level, subject, difficulty)            │  │
│  │                                                      │  │
│  │ 2. Personalization Layer                            │  │
│  │    - Adjust explanation complexity based on:       │  │
│  │      * Student's current level (from profile)      │  │
│  │      * Learning style preference                   │  │
│  │      * Past interaction patterns                   │  │
│  │                                                      │  │
│  │ 3. Multi-Modal Content                              │  │
│  │    - Text explanations                              │  │
│  │    - Diagrams/images (from course materials)       │  │
│  │    - Video timestamps (link to relevant section)   │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│           Personalized Response Generation (LLM)             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ GPT-4 Prompt Engineering:                            │  │
│  │                                                      │  │
│  │ System: "You are a patient tutor. Student Profile:" │  │
│  │ - Name: Alex                                         │  │
│  │ - Grade: 8th                                         │  │
│  │ - Learning style: Visual                            │  │
│  │ - Strengths: Math, Science                          │  │
│  │ - Weaknesses: Abstract concepts                     │  │
│  │ - Recent struggle: Understanding photosynthesis     │  │
│  │                                                      │  │
│  │ Context (from RAG):                                  │  │
│  │ [Course material excerpts about photosynthesis]     │  │
│  │                                                      │  │
│  │ User: "Can you explain photosynthesis simply?"      │  │
│  │                                                      │  │
│  │ Instructions:                                        │  │
│  │ - Use analogies and visual descriptions             │  │
│  │ - Start with fundamentals                           │  │
│  │ - Suggest diagram from course materials             │  │
│  │ - End with a practice question                      │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Adaptive Assessment Engine                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 1. Question Selection (Item Response Theory - IRT)  │  │
│  │    - Estimate student ability (θ)                   │  │
│  │    - Select questions matching ability              │  │
│  │    - Adjust difficulty based on responses           │  │
│  │                                                      │  │
│  │ 2. Feedback Generation                               │  │
│  │    - If wrong: Identify misconception               │  │
│  │    - Provide targeted hint (not full answer)        │  │
│  │    - Suggest prerequisite review if needed          │  │
│  │                                                      │  │
│  │ 3. Progress Tracking                                 │  │
│  │    - Update knowledge graph with mastery            │  │
│  │    - Unlock next concepts when prerequisites met    │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│           Learning Path Recommendation System                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Analyze:                                             │  │
│  │ - Current mastery (from knowledge graph)            │  │
│  │ - Learning goals (student-defined)                  │  │
│  │ - Time available (student schedule)                 │  │
│  │ - Interest signals (topics engaged with)            │  │
│  │                                                      │  │
│  │ Recommend:                                           │  │
│  │ - Next topics to study (based on prerequisites)    │  │
│  │ - Practice problems (spaced repetition)             │  │
│  │ - Review sessions (for forgotten concepts)          │  │
│  │ - Enrichment content (for fast learners)            │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Implementation (Python Code)

```python
from typing import List, Dict
from dataclasses import dataclass
from openai import AsyncOpenAI
import numpy as np
from datetime import datetime

@dataclass
class StudentProfile:
    student_id: str
    name: str
    grade: int
    learning_style: str  # "visual", "auditory", "kinesthetic"
    knowledge_levels: Dict[str, float]  # topic → mastery (0.0-1.0)
    interaction_history: List[Dict]
    learning_goals: List[str]
    
@dataclass
class CourseContent:
    topic: str
    difficulty: str  # "beginner", "intermediate", "advanced"
    content: str
    media_urls: List[str]  # diagrams, videos
    prerequisites: List[str]

class PersonalizedLearningAssistant:
    def __init__(self):
        self.client = AsyncOpenAI()
        self.vector_db = None  # Pinecone client
        self.knowledge_graph = None  # Neo4j client
        
    async def answer_question(self, student: StudentProfile, 
                             question: str) -> str:
        """Generate personalized answer to student question"""
        
        # Step 1: Retrieve relevant course content (RAG)
        retrieved_content = await self._retrieve_content(
            query=question,
            student_level=student.grade,
            topic_filters=self._extract_topics(question)
        )
        
        # Step 2: Personalize prompt based on student profile
        system_prompt = self._build_personalized_prompt(student)
        
        # Step 3: Generate response with GPT-4
        context = "\n\n".join([c.content for c in retrieved_content])
        
        response = await self.client.chat.completions.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": f"""Context from course materials:
{context}

Student question: {question}

Provide a personalized explanation that:
1. Matches the student's learning style ({student.learning_style})
2. Uses appropriate complexity for grade {student.grade}
3. Includes relevant diagrams/videos from course materials
4. Ends with a practice question to check understanding"""}
            ],
            temperature=0.7,
            max_tokens=800
        )
        
        answer = response.choices[0].message.content
        
        # Step 4: Update student interaction history
        await self._log_interaction(student, question, answer)
        
        return answer
    
    def _build_personalized_prompt(self, student: StudentProfile) -> str:
        """Construct system prompt based on student profile"""
        
        # Analyze student's strengths/weaknesses
        strong_topics = [topic for topic, mastery in student.knowledge_levels.items() 
                        if mastery > 0.7]
        weak_topics = [topic for topic, mastery in student.knowledge_levels.items() 
                      if mastery < 0.4]
        
        prompt = f"""You are a patient, encouraging tutor helping {student.name}, a {student.grade}th grader.

Student Profile:
- Learning style: {student.learning_style}
  {"- Prefers visual diagrams and illustrations" if student.learning_style == "visual" else ""}
  {"- Learns best through listening and discussion" if student.learning_style == "auditory" else ""}
  {"- Learns through hands-on activities and examples" if student.learning_style == "kinesthetic" else ""}

- Strong areas: {", ".join(strong_topics) if strong_topics else "Still building foundation"}
- Needs support in: {", ".join(weak_topics) if weak_topics else "Doing well overall"}

Teaching approach:
1. Start with concepts they already know (build on strengths)
2. Use analogies and real-world examples
3. Break complex ideas into simple steps
4. Encourage and praise effort
5. {"Include visual descriptions and suggest diagrams" if student.learning_style == "visual" else ""}
6. Check understanding with follow-up questions"""
        
        return prompt
    
    async def _retrieve_content(self, query: str, student_level: int, 
                               topic_filters: List[str]) -> List[CourseContent]:
        """RAG: Retrieve relevant course content"""
        
        # Embed query
        query_embedding = await self._embed(query)
        
        # Hybrid search: Semantic + metadata filters
        results = await self.vector_db.query(
            vector=query_embedding,
            top_k=5,
            filter={
                "grade_level": {"$lte": student_level + 1},  # Slightly above student level
                "subject": {"$in": topic_filters}
            }
        )
        
        # Convert to CourseContent objects
        content_list = []
        for result in results["matches"]:
            content_list.append(CourseContent(
                topic=result["metadata"]["topic"],
                difficulty=result["metadata"]["difficulty"],
                content=result["metadata"]["text"],
                media_urls=result["metadata"].get("media_urls", []),
                prerequisites=result["metadata"].get("prerequisites", [])
            ))
        
        return content_list
    
    def _extract_topics(self, question: str) -> List[str]:
        """Extract subject topics from question"""
        # Simple keyword matching (in production, use NER or classifier)
        topics = []
        keywords = {
            "math": ["algebra", "calculus", "geometry", "trigonometry"],
            "science": ["photosynthesis", "chemistry", "physics", "biology"],
            "programming": ["python", "function", "loop", "variable"]
        }
        
        question_lower = question.lower()
        for subject, terms in keywords.items():
            if any(term in question_lower for term in terms):
                topics.append(subject)
        
        return topics if topics else ["general"]
    
    async def _log_interaction(self, student: StudentProfile, 
                               question: str, answer: str):
        """Log interaction and update knowledge graph"""
        
        interaction = {
            "timestamp": datetime.now().isoformat(),
            "question": question,
            "answer_length": len(answer),
            "topics": self._extract_topics(question)
        }
        
        student.interaction_history.append(interaction)
        
        # Update knowledge graph (simplified)
        # In production: Analyze answer quality, update mastery estimates
        
    async def _embed(self, text: str) -> List[float]:
        """Generate embedding for text"""
        response = await self.client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        return response.data[0].embedding

class AdaptiveAssessment:
    """Adaptive assessment using Item Response Theory (IRT)"""
    
    def __init__(self):
        self.question_bank = {}  # question_id → IRT parameters (difficulty, discrimination)
        
    def select_next_question(self, student_ability: float, 
                            answered_questions: List[str]) -> str:
        """Select optimal next question based on student ability (IRT)"""
        
        # IRT: P(correct) = 1 / (1 + exp(-a*(θ - b)))
        # where: θ = student ability, a = discrimination, b = difficulty
        
        best_question = None
        best_info = 0
        
        for qid, params in self.question_bank.items():
            if qid in answered_questions:
                continue
            
            # Fisher information: How much we learn from this question
            info = self._calculate_information(student_ability, params)
            
            if info > best_info:
                best_info = info
                best_question = qid
        
        return best_question
    
    def _calculate_information(self, ability: float, params: Dict) -> float:
        """Calculate Fisher information (IRT)"""
        a = params["discrimination"]
        b = params["difficulty"]
        
        prob = 1 / (1 + np.exp(-a * (ability - b)))
        info = a ** 2 * prob * (1 - prob)
        
        return info
    
    def update_ability_estimate(self, student_ability: float, 
                               question_params: Dict, 
                               is_correct: bool) -> float:
        """Update student ability estimate based on response"""
        
        # Bayesian update (simplified)
        # In production: Use full IRT estimation (Maximum Likelihood or Bayes)
        
        learning_rate = 0.2
        a = question_params["discrimination"]
        b = question_params["difficulty"]
        
        expected_prob = 1 / (1 + np.exp(-a * (student_ability - b)))
        error = (1 if is_correct else 0) - expected_prob
        
        new_ability = student_ability + learning_rate * error
        
        return new_ability
    
    async def provide_feedback(self, question: str, student_answer: str, 
                              correct_answer: str, student: StudentProfile) -> str:
        """Generate personalized feedback on assessment"""
        
        client = AsyncOpenAI()
        
        prompt = f"""You are providing feedback on a student's answer.

Student: {student.name} (Grade {student.grade})
Question: {question}
Student's answer: {student_answer}
Correct answer: {correct_answer}

Provide constructive feedback that:
1. Identifies what they got right (start positive)
2. Explains the misconception (if wrong)
3. Provides a targeted hint (not full solution)
4. Suggests related concept to review if needed
5. Encourages them to try again

Tone: Encouraging and supportive"""
        
        response = await client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.7,
            max_tokens=300
        )
        
        return response.choices[0].message.content

# Usage Example
async def main():
    # Initialize student profile
    student = StudentProfile(
        student_id="12345",
        name="Alex",
        grade=8,
        learning_style="visual",
        knowledge_levels={
            "algebra": 0.7,
            "biology": 0.4,
            "physics": 0.5
        },
        interaction_history=[],
        learning_goals=["Master photosynthesis", "Prepare for algebra test"]
    )
    
    # Question answering
    assistant = PersonalizedLearningAssistant()
    answer = await assistant.answer_question(
        student=student,
        question="Can you explain photosynthesis in simple terms?"
    )
    print(f"Answer: {answer}")
    
    # Adaptive assessment
    assessment = AdaptiveAssessment()
    next_question = assessment.select_next_question(
        student_ability=0.5,  # Estimated from knowledge graph
        answered_questions=[]
    )
    print(f"Next question: {next_question}")
```

### Cost Analysis

**100K active students, each asking 10 questions/month:**

**Question Answering (RAG + LLM):**
- Vector search: $0.001 per query (Pinecone)
- GPT-4 response: $0.03 per question (400 input + 800 output tokens)
- Total per question: $0.031
- Monthly cost: 100K students × 10 questions × $0.031 = $31,000

**Adaptive Assessment Feedback:**
- GPT-3.5-turbo: $0.002 per feedback (cheaper model)
- 5 assessments/student/month: 100K × 5 × $0.002 = $1,000

**Embeddings (Course Content):**
- One-time cost: 5K courses × 50 chunks/course × $0.0001 = $25
- Updates: $100/month (new content)

**Infrastructure:**
- PostgreSQL (student profiles): $200/month
- Neo4j (knowledge graph): $500/month
- Pinecone (vector DB): $200/month
- Monitoring & logging: $300/month

**Total Monthly Cost:**
- LLM API: $32,000
- Infrastructure: $1,300
- **Total: $33,300/month** for 100K students = **$0.33 per student**

**Well under target of $2/student! ✓**

**Optimization opportunities:**
- Cache common questions (30% hit rate) → Save $9,300
- Use GPT-3.5-turbo for simple questions → Save $10,000
- **Optimized: $23,000/month = $0.23/student**

### Scaling & Performance

**Response time:**
- RAG retrieval: ~200ms
- GPT-4 generation: ~2s
- Total: <3s ✓ (meets requirement)

**Concurrent users:**
- Peak: 10K students online simultaneously
- Requests/sec: ~100 (assuming 1 question per 10 min session)
- Infrastructure: Auto-scaling to handle spikes

**Data storage:**
- Student profiles: 100K × 10KB = 1GB (PostgreSQL)
- Knowledge graph: 100K students × 500 concepts = 50M nodes (Neo4j)
- Interaction history: 100K × 100 interactions × 1KB = 10GB

### Personalization Strategies

**1. Learning Style Adaptation:**
```python
def adapt_explanation_style(content: str, learning_style: str) -> str:
    """Modify explanation based on learning style"""
    
    if learning_style == "visual":
        # Add visual descriptions, suggest diagrams
        additions = [
            "Imagine this as a diagram...",
            "Picture this process as a flowchart...",
            "[See diagram: {url}]"
        ]
    elif learning_style == "auditory":
        # Use conversational tone, suggest videos
        additions = [
            "Let's talk through this step-by-step...",
            "Think of it like a story...",
            "[Listen to explanation: {url}]"
        ]
    else:  # kinesthetic
        # Suggest hands-on activities, experiments
        additions = [
            "Try this yourself...",
            "Here's an experiment you can do...",
            "Practice problem: ..."
        ]
    
    return content + "\n\n" + additions[0]
```

**2. Spaced Repetition (Forgetting Curve):**
```python
def schedule_review(topic: str, last_review: datetime, 
                   mastery: float) -> datetime:
    """Schedule next review based on forgetting curve"""
    
    # Ebbinghaus forgetting curve: R = e^(-t/S)
    # where R = retention, t = time, S = memory strength
    
    if mastery > 0.8:
        interval_days = 30  # Strong memory
    elif mastery > 0.6:
        interval_days = 14  # Medium memory
    else:
        interval_days = 3   # Weak memory, review soon
    
    next_review = last_review + timedelta(days=interval_days)
    return next_review
```

**3. Knowledge Graph Updates:**
```python
async def update_knowledge_graph(student_id: str, assessment_result: Dict):
    """Update student mastery in knowledge graph"""
    
    # Cypher query (Neo4j)
    query = """
    MATCH (s:Student {id: $student_id})-[r:KNOWS]->(c:Concept {name: $concept})
    SET r.mastery = $new_mastery,
        r.last_assessed = datetime()
    
    // Unlock next concepts if prerequisites met
    WITH s, c
    MATCH (next:Concept)-[:REQUIRES]->(c)
    WHERE NOT (s)-[:KNOWS]->(next)
    AND all(prereq IN [(next)-[:REQUIRES]->(p) | p] 
            WHERE (s)-[:KNOWS {mastery >= 0.7}]->(prereq))
    CREATE (s)-[:KNOWS {mastery: 0.0, unlocked: datetime()}]->(next)
    """
    
    await neo4j_client.run(query, 
        student_id=student_id,
        concept=assessment_result["topic"],
        new_mastery=assessment_result["mastery"]
    )
```

### Interview Questions & Answers

**Q: How do you personalize explanations for different learning styles without over-engineering?**

**A:** Three-tier approach:

**Tier 1: Profile-based (Simple):**
- Store student preference (visual/auditory/kinesthetic) in profile
- Adjust prompt template: "Use visual descriptions and suggest diagrams" (visual), "Use conversational tone and stories" (auditory), "Suggest hands-on activities" (kinesthetic)
- Cost: $0, just prompt engineering

**Tier 2: Content-based (Moderate):**
- Include multimedia in RAG retrieval: Retrieve relevant diagrams (visual), video links (auditory), interactive exercises (kinesthetic)
- Cost: Metadata filtering in vector DB (negligible)

**Tier 3: Adaptive (Advanced):**
- Track engagement metrics: Time on page, scroll depth, media clicks
- Use ML to infer actual learning style (may differ from self-reported)
- Dynamically adjust over time
- Cost: ML model inference ($0.001 per prediction)

**Start with Tier 1 (90% of value, 10% of effort). Add Tier 2/3 based on data.**

**Q: Your system recommends next topics. How do you prevent students from skipping prerequisites?**

**A:** Knowledge graph with hard constraints:

```python
# Neo4j graph structure
(Algebra_Basics)-[:REQUIRES]->(Arithmetic_Mastery)
(Calculus)-[:REQUIRES]->(Algebra_Basics)

# Query to find available topics
MATCH (s:Student {id: $student_id})
MATCH (next:Concept)
WHERE NOT (s)-[:KNOWS]->(next)  // Not already learned
AND all(prereq IN [(next)-[:REQUIRES]->(p) | p]   // All prerequisites met
        WHERE (s)-[:KNOWS {mastery >= 0.7}]->(prereq))
RETURN next
```

**Validation:** 
- Backend enforces prerequisites (can't access content without unlocking)
- Frontend shows locked/unlocked status
- Exception: "Placement test" to skip prerequisites if student already knows them

**Result:** Students follow optimal learning path, no knowledge gaps.

**Q: How do you balance between helping students and giving away answers?**

**A:** Graduated hint system:

**Level 1 (Nudge):** "Think about what you know about [related concept]"
- Prompt hints at approach, doesn't give solution
- Example: For "Solve x² - 5x + 6 = 0", hint is "Try factoring into two binomials"

**Level 2 (Scaffold):** "Start by [first step], then..."
- Breaks problem into steps, student fills in details
- Example: "First, find two numbers that multiply to 6 and add to -5"

**Level 3 (Worked Example):** Similar problem with full solution
- Shows process for analogous problem
- Example: "Here's how to solve x² - 3x + 2 = 0: ..."
- Student applies same process to original

**Implementation:**
```python
def generate_hint(problem: str, attempt_count: int) -> str:
    hint_level = min(attempt_count, 3)  # Cap at level 3
    
    prompt = f"""Generate a Level {hint_level} hint for this problem:
Problem: {problem}

Level 1: Nudge (hint at approach)
Level 2: Scaffold (break into steps)
Level 3: Worked example (similar problem solved)

Do NOT give the direct answer."""
    
    # LLM generates appropriate hint
```

**Metrics:** Track if students solve after each hint level (success rate: 60% after L1, 85% after L2, 95% after L3).

**Q: How do you handle 100K students asking questions simultaneously at peak times (e.g., night before exam)?**

**A:** Multi-tier scaling strategy:

**1. Caching Layer:**
- Common questions cached (Redis): "Explain photosynthesis" asked by 1000 students → Cache 1 answer
- Semantic deduplication: Similar questions get same cached response
- Cache hit rate: 40% during exam prep periods
- Latency: 50ms (from cache) vs 2s (LLM generation)

**2. Auto-scaling:**
- Monitor queue depth (Kafka/RabbitMQ)
- Scale out API servers when queue > 1000 messages
- Use serverless (AWS Lambda) for burst capacity
- Example: Normally 10 instances, scale to 100 during peak

**3. Prioritization:**
- Premium students (paid tier) get priority queue
- Free tier: "High traffic, expect 30s delay" message
- Load shedding: Return cached/pre-generated answers if overloaded

**4. Async Processing:**
- Don't make students wait for LLM response
- Immediate response: "Generating personalized explanation..."
- WebSocket push notification when ready (5-10s later)

**Cost during 10x spike:** 
- Normal: $32K/month
- Spike (1 day): $3,200 extra → Acceptable
- With caching: $1,500 extra (50% reduction)

**Real-world:** EdTech platforms see 5-10x traffic spikes during exam prep weeks. Design for 5x normal capacity, degrade gracefully beyond that.
