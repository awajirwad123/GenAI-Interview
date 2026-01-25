# Agents & Tools - Interview Questions

### Q1: Design a tool calling system for a customer support agent.

**Requirements:**
- Check order status
- Process refunds
- Search knowledge base
- Escalate to human

**Design:**

```python
tools = [
    {
        "name": "get_order_status",
        "description": "Get status of customer order",
        "parameters": {
            "order_id": "string"
        }
    },
    {
        "name": "process_refund",
        "description": "Initiate refund. ONLY use if customer explicitly requests refund.",
        "parameters": {
            "order_id": "string",
            "reason": "string"
        },
        "requires_confirmation": True  # Safety
    },
    {
        "name": "search_knowledge_base",
        "description": "Search FAQs and help docs",
        "parameters": {
            "query": "string"
        }
    },
    {
        "name": "escalate_to_human",
        "description": "Transfer to human agent. Use if: customer explicitly requests, issue too complex, or after 3 failed attempts.",
        "parameters": {
            "reason": "string"
        }
    }
]

# Safety: Critical tools require confirmation
def execute_tool(tool_name, params):
    tool = get_tool(tool_name)
    
    if tool.requires_confirmation:
        confirmation = ask_user_confirmation(tool_name, params)
        if not confirmation:
            return "User declined action"
    
    # Validate params
    if not validate_params(params, tool.schema):
        return "Invalid parameters"
    
    # Log for audit
    log_tool_call(tool_name, params, user_id)
    
    return tool.execute(params)
```

**Key features:**
- Safety confirmations for destructive actions
- Clear escalation criteria
- Audit logging

---

### Q2: Agent keeps selecting wrong tools. How do you debug and fix?

**Debugging:**

```python
# 1. Analyze tool selection patterns
traces = get_agent_traces(last_week)

wrong_selections = []
for trace in traces:
    if trace.tool_selected != trace.correct_tool:
        wrong_selections.append({
            "query": trace.query,
            "selected": trace.tool_selected,
            "should_be": trace.correct_tool
        })

# Pattern: Agent confuses "search_knowledge" with "web_search"
```

**Fixes:**

**1. Improve descriptions**
```python
# Before
"search_knowledge": "Search knowledge base"
"web_search": "Search web"

# After
"search_knowledge": "Search INTERNAL company knowledge base and FAQs. Use for: company policies, product docs, how-to guides. Returns: Internal documentation."

"web_search": "Search EXTERNAL web for current information. Use for: news, current events, external data. Returns: Web results from Google."
```

**2. Add few-shot examples**
```python
system_prompt = """
Examples of correct tool usage:

User: "What's your refund policy?"
Tool: search_knowledge_base("refund policy")

User: "What's the weather today?"
Tool: web_search("current weather")

User: "Where's my order #12345?"
Tool: get_order_status("12345")
"""
```

**3. Reduce tool set**
```python
# Before: 15 tools (confusing)
# After: 7 core tools (80% of use cases)

# Specialized tools → optional, only loaded if needed
```

**4. Hierarchical selection**
```python
# Step 1: Select category
category = select_category(["orders", "products", "general"])

# Step 2: Select specific tool within category
if category == "orders":
    tool = select_tool(["get_order", "track_shipment", "cancel_order"])
```

**Results:**
- Tool selection accuracy: 78% → 92%
- User satisfaction: +15%

---

### Q3: Compare Plan-and-Execute vs ReAct agents.

**ReAct:**
```
Thought: Need weather
Action: search_weather("Paris")
Observation: 18°C
Thought: Got it
Answer: 18°C
```

**Pros:**
- Flexible, adapts on-the-fly
- Transparent reasoning

**Cons:**
- Can loop/get stuck
- Verbose (high token cost)
- Each step waits for previous

**Plan-and-Execute:**
```
Plan:
1. Search for competitors
2. Extract features
3. Compare pricing
4. Generate report

Execute each step sequentially
```

**Pros:**
- Better for complex tasks
- Can parallelize independent steps
- Clearer structure

**Cons:**
- Planning can fail
- Less flexible (follows plan even if wrong)
- More LLM calls (plan + execute)

**When to use:**
- **ReAct**: Simple queries, need flexibility (chatbots, Q&A)
- **Plan-and-Execute**: Complex multi-step (research, analysis, reports)

**Hybrid approach (production):**
```python
if task_complexity < threshold:
    return react_agent(task)
else:
    return plan_execute_agent(task)
```

---

### Q4: Design a multi-agent system for code review.

**Agents:**

```python
class CodeReviewSystem:
    def __init__(self):
        self.security_agent = Agent(
            role="Security Reviewer",
            tools=[scan_vulnerabilities, check_secrets],
            instructions="Find security issues"
        )
        
        self.quality_agent = Agent(
            role="Code Quality Reviewer",
            tools=[lint, complexity_check],
            instructions="Check code quality, style, maintainability"
        )
        
        self.test_agent = Agent(
            role="Test Reviewer",
            tools=[run_tests, check_coverage],
            instructions="Verify test coverage and quality"
        )
        
        self.coordinator = Agent(
            role="Coordinator",
            tools=[],
            instructions="Aggregate reviews, make final decision"
        )
    
    def review(self, pull_request):
        # Run reviewers in parallel
        security_review = self.security_agent.run(pull_request.code)
        quality_review = self.quality_agent.run(pull_request.code)
        test_review = self.test_agent.run(pull_request.code)
        
        # Coordinator makes final decision
        decision = self.coordinator.run({
            "security": security_review,
            "quality": quality_review,
            "tests": test_review
        })
        
        return decision
```

**Architecture choice:**
- **Parallel execution**: Reviews independent, run simultaneously
- **Coordinator pattern**: Single agent aggregates results
- **Specialized agents**: Each has domain expertise

**Production considerations:**
- Set timeouts (some tools may hang)
- Agent consensus on severity (is issue blocking?)
- Human override for edge cases

---

### Q5: Scenario: Tool costs are too high. Optimize without losing functionality.

**Analysis:**

```python
tool_usage = analyze_logs()
# Results:
# web_search: 10K calls/day, $0.01 each = $100/day
# calculator: 5K calls/day, free
# database_query: 3K calls/day, $0.05 each = $150/day
# Total: $250/day = $7.5K/month
```

**Optimizations:**

**1. Cache tool results**
```python
@cache(ttl=3600)  # Cache 1 hour
def cached_web_search(query):
    return web_search(query)

# Hit rate: 45%
# Savings: $100 → $55/day
```

**2. Batch database queries**
```python
# Before: 3K individual queries
# After: Batch 10 queries per call

# Calls: 3K → 300
# Cost: $150/day → $15/day
```

**3. Fallback to cheaper alternatives**
```python
def search_with_fallback(query):
    # Try cache first (free)
    if cached_result := cache.get(query):
        return cached_result
    
    # Try knowledge base (free)
    if kb_result := search_kb(query):
        return kb_result
    
    # Last resort: web search ($)
    return web_search(query)

# Web search usage: 10K → 4K calls/day
# Cost: $100 → $40/day
```

**4. Rate limiting**
```python
# Prevent agent loops from burning budget
@ratelimit(calls=100, period=3600)  # Max 100/hour per user
def expensive_tool(params):
    return external_api(params)
```

**Results:**
- Caching: -45% cost
- Batching: -90% database cost
- Fallbacks: -60% search cost
- **Total: $7.5K → $2K/month**

---

### Q6: Tool returns invalid data. How does agent handle it?

**Problem:**
```python
# Tool returns unexpected format
result = get_weather("Paris")
# Expected: {"temp": 18, "condition": "sunny"}
# Got: "Error: API timeout"
```

**Solutions:**

**1. Tool-level validation**
```python
def validated_tool(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        
        # Validate result format
        if not isinstance(result, dict):
            return {
                "error": "Tool returned invalid format",
                "raw_result": result
            }
        
        # Validate required fields
        required = ["temp", "condition"]
        if not all(k in result for k in required):
            return {"error": "Missing required fields"}
        
        return result
    return wrapper
```

**2. Agent-level retry**
```python
def agent_with_retry(agent, max_retries=3):
    for attempt in range(max_retries):
        result = agent.run(input)
        
        if validate_result(result):
            return result
        
        # Inject error feedback
        input += f"\nPrevious attempt failed: {result.error}. Try again."
    
    # All retries failed
    return fallback_response()
```

**3. Graceful degradation**
```python
def agent_with_fallback(input):
    try:
        result = primary_agent(input)
        if result.success:
            return result
    except Exception as e:
        log_error(e)
    
    # Fallback: Simpler approach
    return simple_llm_response(input)  # No tools
```

**Production pattern:**
```python
def robust_tool_call(tool, params):
    # Validate inputs
    if not validate_params(params):
        return {"error": "Invalid input"}
    
    # Execute with timeout
    try:
        result = timeout(tool.execute, params, max_seconds=10)
    except TimeoutError:
        return {"error": "Tool timeout"}
    except Exception as e:
        return {"error": str(e)}
    
    # Validate output
    if not validate_output(result):
        return {"error": "Invalid output", "raw": result}
    
    return result
```

**Monitoring:**
```python
metrics = {
    "tool_success_rate": 0.94,
    "tool_error_rate": 0.06,
    "timeout_rate": 0.02,
    "validation_failure_rate": 0.04
}
# Alert if error_rate > 10%
```
