# Agents & Tools - Core Concepts

## Agent Architectures

### 1. ReAct (Reason + Act)
**Most common pattern in production.**

```python
# Agent loop
while not done:
    thought = llm.think(context)
    action = llm.select_tool(thought, available_tools)
    observation = execute_tool(action)
    context += f"{thought}\n{action}\n{observation}"
```

**Pros**: Transparent reasoning, good for multi-step
**Cons**: Can loop infinitely, verbose (high token cost)

### 2. Function Calling
**Modern standard (GPT-4, Claude 3+)**

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string"}
            }
        }
    }
]

response = openai.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Weather in Paris?"}],
    tools=tools
)

if response.tool_calls:
    result = execute_tool(response.tool_calls[0])
```

**Pros**: Structured, reliable, native support
**Cons**: Model-specific APIs

### 3. Plan-and-Execute
**Best for complex, multi-step tasks**

```python
# Step 1: Create plan
plan = planner_llm(
    "Research competitors and create comparison report"
)
# Output: ["Search for competitors", "Extract key features", 
#          "Compare pricing", "Generate report"]

# Step 2: Execute each step
for step in plan:
    result = executor_llm(step, previous_results)
    results.append(result)

# Step 3: Synthesize
final = synthesizer_llm(plan, results)
```

**Pros**: Better for complex tasks, clearer structure
**Cons**: Slower (multiple LLM calls), planning can fail

## Tool Design Best Practices

### 1. Tool Description
```python
# Bad
Tool(name="search", description="Search")

# Good
Tool(
    name="web_search",
    description="""Search the web for current information.
    
    Use when:
    - User asks about recent events
    - Need current data (stocks, weather, news)
    - Information not in knowledge base
    
    Don't use for:
    - Math calculations
    - General knowledge (use your training)
    - Historical facts
    
    Returns: List of web results with title, snippet, URL
    """
)
```

**Why it matters**: Clear descriptions reduce wrong tool selection by 40%.

### 2. Tool Parameters
```python
def search_tool(
    query: str,
    num_results: int = 5,
    date_range: Optional[str] = None  # "past_day", "past_week"
) -> List[Dict]:
    """Search with type hints and defaults"""
    pass
```

### 3. Error Handling
```python
def safe_tool(func):
    def wrapper(*args, **kwargs):
        try:
            return {"success": True, "result": func(*args, **kwargs)}
        except Exception as e:
            return {"success": False, "error": str(e)}
    return wrapper

@safe_tool
def calculator(expression: str):
    return eval(expression)  # Dangerous! Use ast.literal_eval
```

## Multi-Agent Systems

### Patterns

**1. Sequential (Pipeline)**
```
Researcher → Analyst → Writer → Reviewer
```
Each agent specializes in one task.

**2. Hierarchical**
```
Manager
  ├── Researcher
  ├── Analyst
  └── Writer
```
Manager delegates to specialists.

**3. Collaborative**
```
Agents discuss in group chat, vote on decisions
```

### Example: Research Team
```python
class ResearchTeam:
    def __init__(self):
        self.researcher = Agent(
            role="researcher",
            tools=[web_search, scrape_page],
            goal="Find accurate information"
        )
        
        self.analyst = Agent(
            role="analyst",
            tools=[calculator, data_viz],
            goal="Analyze data and find insights"
        )
        
        self.writer = Agent(
            role="writer",
            tools=[grammar_check],
            goal="Write clear, concise reports"
        )
    
    def run(self, task):
        # Sequential execution
        data = self.researcher.run(task)
        insights = self.analyst.run(f"Analyze: {data}")
        report = self.writer.run(f"Write report: {insights}")
        return report
```

## Tool Calling Challenges

### Problem 1: Hallucinated Tool Calls
```
Agent: search_weather("Fake City, Mars")
```

**Solution:**
- Validate tool inputs
- Provide example calls in prompt
- Use constrained generation (function calling API)

### Problem 2: Tool Selection Errors
Agent picks wrong tool 15-20% of the time.

**Solutions:**
1. Better descriptions (most impactful)
2. Few-shot examples
3. Smaller tool set (5-10 tools max)
4. Hierarchical tool selection (categories first)

### Problem 3: Infinite Loops
```
search → analyze → search → analyze → ...
```

**Solutions:**
- Max iterations limit
- State tracking (don't repeat same action)
- Early stopping if no progress

## Production Patterns

### 1. Tool Result Caching
```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def cached_search(query: str):
    return expensive_search(query)
```

### 2. Rate Limiting
```python
from ratelimit import limits

@limits(calls=10, period=60)  # 10 calls per minute
def api_tool(query):
    return external_api.call(query)
```

### 3. Fallback Tools
```python
def search_with_fallback(query):
    try:
        return primary_search(query)
    except Exception:
        log_error("Primary search failed, using fallback")
        return fallback_search(query)
```

### 4. Tool Monitoring
```python
def monitored_tool(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        try:
            result = func(*args, **kwargs)
            log_metric("tool_success", func.__name__)
            return result
        except Exception as e:
            log_metric("tool_error", func.__name__, error=str(e))
            raise
        finally:
            latency = time.time() - start
            log_metric("tool_latency", func.__name__, latency=latency)
    return wrapper
```

## Key Interview Points

**Q: How many tools should an agent have?**
- Optimal: 5-10 tools
- Max: 20-30 (beyond this, selection accuracy drops)
- Solution for many tools: Hierarchical (category → specific tool)

**Q: Tool calling vs RAG?**
- Tools: For **actions** (search, calculate, API calls)
- RAG: For **knowledge** (retrieve documents)
- Often combined: RAG tool is one of many tools

**Q: How to make agents more reliable?**
1. Clear tool descriptions (+40% accuracy)
2. Few-shot examples (+20%)
3. Max iterations limit (prevents spiraling)
4. Validation & error handling
5. Logging & monitoring
6. Fallback to fixed workflow when agent fails

**Cost awareness:**
- Each tool call = 1 LLM inference
- Agents use 2-5× more tokens than direct prompts
- Cache tool results, limit iterations
