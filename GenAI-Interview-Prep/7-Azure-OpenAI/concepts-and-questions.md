# Azure OpenAI - Essentials & Interview Guide

## Key Concepts

### 1. Deployment Model
```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key="...",
    api_version="2024-02-01",
    azure_endpoint="https://YOUR_RESOURCE.openai.azure.com"
)

# Each model needs a deployment
response = client.chat.completions.create(
    model="gpt-4-deployment-name",  # Your deployment name, not "gpt-4"
    messages=[{"role": "user", "content": "Hello"}]
)
```

**Key difference from OpenAI:**
- Azure: Deploy models to your resource (you name deployments)
- OpenAI: Use model names directly

### 2. Quotas & Rate Limits
- **TPM** (Tokens Per Minute): 10K, 60K, 240K, etc.
- **RPM** (Requests Per Minute): 100, 300, 1000, etc.

**Production gotcha:** Need to request quota increases (can take days)

### 3. Regions & Availability
- **US East**: Most models, high capacity
- **West Europe**: Good latency for EU
- **East US 2**: Often has capacity when others don't

**Interview tip:** Always have backup region for failover

### 4. Content Filtering
```python
# Azure has built-in content filters
# Levels: Low, Medium, High
# Categories: Hate, Sexual, Violence, Self-harm
```

Can be disabled for approved use cases (requires form submission)

### 5. Private Endpoints
```python
# Deploy in VNet for security
# No public internet access
# Connect via Private Link
```

**When needed:** Regulated industries (healthcare, finance)

## High-Probability Questions

### Q1: Azure OpenAI vs OpenAI API?

**Azure OpenAI:**
- ✅ Enterprise features (VNet, Private Link, SLA)
- ✅ Data residency (stays in your region)
- ✅ Integration with Azure ecosystem
- ✅ No rate limits (only quota)
- ❌ Slower model updates
- ❌ More complex setup

**OpenAI API:**
- ✅ Latest models first
- ✅ Simple setup
- ✅ No quota management
- ❌ No data residency guarantees
- ❌ Public endpoint only
- ❌ Rate limits per API key

**Production choice:** Azure for enterprise, OpenAI for startups/prototypes

---

### Q2: How do you handle quota limits?

**Strategies:**

**1. Multiple deployments**
```python
deployments = ["gpt-4-deployment-1", "gpt-4-deployment-2"]

def call_with_failover(prompt):
    for deployment in deployments:
        try:
            return client.chat.completions.create(
                model=deployment,
                messages=[{"role": "user", "content": prompt}]
            )
        except RateLimitError:
            continue  # Try next deployment
    
    raise Exception("All deployments rate limited")
```

**2. Token bucket**
```python
class TokenBucket:
    def __init__(self, tpm_limit=10000):
        self.limit = tpm_limit
        self.tokens = tpm_limit
        self.last_update = time.time()
    
    def consume(self, tokens_needed):
        # Refill bucket
        now = time.time()
        elapsed = now - self.last_update
        self.tokens = min(self.limit, self.tokens + elapsed * (self.limit / 60))
        self.last_update = now
        
        # Check availability
        if self.tokens >= tokens_needed:
            self.tokens -= tokens_needed
            return True
        return False  # Rate limited
```

**3. Regional failover**
```python
regions = [
    "https://eastus.openai.azure.com",
    "https://westeurope.openai.azure.com"
]

for endpoint in regions:
    try:
        client = AzureOpenAI(azure_endpoint=endpoint)
        return client.chat.completions.create(...)
    except RateLimitError:
        continue
```

**4. Request quota increase**
- Fill form in Azure Portal
- Justification: Expected usage, business case
- Timeline: 3-7 business days

---

### Q3: Cost optimization for Azure OpenAI?

**Tactics:**

**1. Provisioned throughput** (if high volume)
```
Standard: Pay per token
- GPT-4: $0.03/1K input, $0.06/1K output
- Variable cost

Provisioned: Pay for reserved capacity
- $300-5000/month fixed
- Break-even: ~10M tokens/month
```

**2. Model selection**
```python
def route_request(query, complexity):
    if complexity == "simple":
        return gpt_35_turbo(query)  # 10× cheaper
    elif complexity == "medium":
        return gpt_4_turbo(query)
    else:
        return gpt_4(query)  # Most expensive
```

**3. Caching**
```python
@cache(ttl=3600)
def cached_completion(prompt):
    return client.chat.completions.create(...)

# Hit rate: 30-50% typical
# Cost savings: 30-50%
```

**4. Prompt compression**
- Remove redundant context
- Summarize long histories
- Each 1K tokens saved = $0.03

---

### Q4: Security best practices?

**Key points:**

1. **Managed Identity** (no API keys)
```python
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
client = AzureOpenAI(
    azure_ad_token_provider=credential,
    azure_endpoint="..."
)
```

2. **Private Endpoints** (no public access)
3. **Key Vault** (if using API keys)
4. **Audit logging** (track all API calls)
5. **Content filtering** (prevent harmful outputs)
6. **Data residency** (choose region carefully)

---

### Q5: Handling 429 (rate limit) errors?

**Implementation:**

```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(5),
    wait=wait_exponential(multiplier=1, min=1, max=60)
)
def resilient_completion(prompt):
    try:
        return client.chat.completions.create(...)
    except RateLimitError as e:
        # Check if quota or TPM limit
        if "quota" in str(e):
            # Need to request increase
            notify_ops_team()
        raise  # Retry
```

**Production pattern:**
- Exponential backoff: 1s, 2s, 4s, 8s, 16s
- Circuit breaker: Stop trying after N failures
- Fallback: Switch to alternative deployment/region

---

## Scenario Questions

### S1: Design Azure OpenAI architecture for 1M users

**Architecture:**

```
Users → API Gateway → Rate Limiter → Load Balancer
                                        ↓
                            [Azure OpenAI Deployments]
                            ├── East US: gpt-4 (60K TPM)
                            ├── East US: gpt-3.5 (240K TPM)
                            └── West EU: gpt-4 (60K TPM, failover)
                                        ↓
                            Cache Layer (Redis)
                                        ↓
                            Monitoring & Logging
```

**Key components:**
1. **Rate limiter**: Protect quotas
2. **Multiple deployments**: Horizontal scaling
3. **Geo-redundancy**: East US + West EU
4. **Caching**: 30-50% cost savings
5. **Monitoring**: Track TPM usage, errors

**Cost estimate:**
- 1M users, 10 requests/day = 10M requests
- Avg 1K tokens/request = 10B tokens/month
- GPT-3.5: ~$20K/month
- GPT-4: ~$300K/month
- **Mix (80% GPT-3.5, 20% GPT-4)**: ~$80K/month

---

### S2: Production goes down due to quota exhaustion. How to prevent?

**Prevention measures:**

**1. Monitoring & Alerting**
```python
def monitor_quota_usage():
    current_tpm = get_current_usage()
    quota_limit = 60000
    
    if current_tpm > quota_limit * 0.8:
        alert("80% quota used")
    
    if current_tpm > quota_limit * 0.95:
        alert("CRITICAL: 95% quota used", priority="high")
```

**2. Request queuing**
```python
from queue import PriorityQueue

request_queue = PriorityQueue()

def queue_request(request, priority="normal"):
    request_queue.put((priority_score(priority), request))

def process_queue():
    while not request_queue.empty():
        if quota_available():
            _, request = request_queue.get()
            process(request)
        else:
            time.sleep(1)  # Wait for quota refresh
```

**3. Auto-scaling deployments**
- Provision multiple deployments
- Auto-switch when one is rate limited

**4. Circuit breaker**
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5):
        self.failures = 0
        self.threshold = failure_threshold
        self.state = "closed"  # closed, open, half-open
    
    def call(self, func):
        if self.state == "open":
            raise Exception("Circuit breaker open")
        
        try:
            result = func()
            self.failures = 0
            return result
        except RateLimitError:
            self.failures += 1
            if self.failures >= self.threshold:
                self.state = "open"
                notify_ops()
            raise
```

**Runbook:**
1. Alert fires at 80% quota
2. Ops team requests quota increase
3. Meanwhile, activate backup deployment
4. Monitor until quota increase approved
