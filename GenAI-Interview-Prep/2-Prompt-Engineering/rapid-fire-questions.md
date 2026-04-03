# Prompt Engineering - Rapid-Fire Interview Questions

Quick-fire questions covering all prompt engineering concepts. Ideal for last-minute revision and interview preparation.

---

## 1. Prompts - Basics

**Q1:** What is a prompt?  
**A:** The input text/instruction you provide to an LLM to guide its response, defining the task, providing context, and shaping the output.

**Q2:** What are the four components of a well-structured prompt?  
**A:** (1) Instruction (what to do), (2) Context (background info), (3) Input Data (content to process), (4) Output Indicator (format/structure).

**Q3:** What makes a good prompt?  
**A:** Clear, specific, provides enough context without being verbose.

---

## 2. Prompt Engineering Fundamentals

**Q4:** What is prompt engineering?  
**A:** The practice of designing, testing, and optimizing prompts to get the best possible outputs from LLMs without fine-tuning.

**Q5:** Why is prompt engineering important compared to fine-tuning?  
**A:** 100x cheaper ($0 vs $1K-$10K), iterate in minutes vs days/weeks, easy to adapt, and it represents 80% of AI Engineering work.

**Q6:** What are the five core activities of prompt engineering?  
**A:** (1) Design prompts, (2) Test on diverse inputs, (3) Iterate based on feedback, (4) Optimize quality/cost/latency, (5) Version control.

**Q7:** What principle should guide prompt design: clarity or cleverness?  
**A:** Clarity over cleverness - simple, direct instructions work best.

**Q8:** Should you describe behavior or show examples in prompts?  
**A:** Show examples (few-shot learning) is more effective than lengthy descriptions.

**Q9:** How should prompt engineering be treated organizationally?  
**A:** As a software engineering discipline with versioning, testing, optimization, and production deployment.

---

## 3. Sampling Parameters

**Q10:** What is temperature in LLM sampling?  
**A:** A parameter (0.0-2.0) that controls randomness in token selection.

**Q11:** What temperature should you use for deterministic, factual tasks?  
**A:** 0.0-0.3 (low temperature).

**Q12:** What temperature is good for creative writing?  
**A:** 1.5-2.0 (high temperature).

**Q13:** What's the default temperature value typically used?  
**A:** 0.7

**Q14:** Give examples of when to use low temperature.  
**A:** Data extraction, code generation, factual Q&A.

**Q15:** What does high temperature (1.5-2.0) risk?  
**A:** Very creative but potentially incoherent or nonsensical responses.

**Q16:** What is Top-K sampling?  
**A:** Considers only the top K most probable tokens at each generation step (e.g., K=40 means choose from 40 most likely words).

**Q17:** What is the typical Top-K value?  
**A:** 40-50

**Q18:** What is Top-P (nucleus sampling)?  
**A:** Consider tokens whose cumulative probability ≥ P (e.g., Top-P=0.9 uses smallest set covering 90% probability mass).

**Q19:** What's the advantage of Top-P over Top-K?  
**A:** Dynamically adapts to the probability distribution shape rather than using a fixed count.

**Q20:** What's the typical Top-P value?  
**A:** 0.9-0.95

**Q21:** What sampling parameters should you use in production for factual tasks?  
**A:** temperature=0.0-0.3, top_p=1.0 (deterministic).

---

## 4. Output Control

**Q22:** What does the max_tokens parameter control?  
**A:** Maximum number of tokens the model can generate, controlling response length and preventing runaway generation.

**Q23:** What are typical max_tokens values for different use cases?  
**A:** Short answers: 50-100, Summaries: 200-500, Long-form: 1000-2000, Code: 500-1500.

**Q24:** Why is max_tokens important for cost?  
**A:** Output tokens cost 2-3x more than input tokens.

**Q25:** What are stop sequences?  
**A:** Strings that trigger generation to stop immediately when encountered.

**Q26:** Give three use cases for stop sequences.  
**A:** (1) Stop at end of code block `["```"]`, (2) Stop after list item `["\n\n"]`, (3) Stop at custom delimiter `["---END---"]`.

**Q27:** Why are stop sequences crucial for structured output?  
**A:** They control output structure and prevent unwanted continuation in lists, code blocks, and multi-turn generation.

---

## 5. Repetition Penalties

**Q28:** What is frequency penalty?  
**A:** Penalizes tokens based on how often they've appeared (linear penalty per occurrence), reducing word repetition.

**Q29:** What's the typical range and value for frequency penalty?  
**A:** Range: -2.0 to 2.0, Typical: 0.3-0.7

**Q30:** What is presence penalty?  
**A:** Penalizes tokens based on whether they've appeared at all (binary), encouraging topic diversity.

**Q31:** What's the difference between frequency and presence penalty?  
**A:** Frequency penalizes based on count (how many times), presence penalizes if appeared at all (binary).

**Q32:** When do you use frequency penalty vs presence penalty?  
**A:** Frequency: Avoid exact word repetition. Presence: Encourage exploring new topics.

**Q33:** What does a negative penalty value do?  
**A:** Encourages repetition (opposite effect).

---

## 6. System, Role, and Contextual Prompting

**Q34:** What is a system prompt?  
**A:** Sets the assistant's behavior, personality, and constraints separately from user messages, with higher priority.

**Q35:** What should a system prompt contain?  
**A:** Tone, expertise, constraints, behavioral guidelines.

**Q36:** What's the recommended length for system prompts?  
**A:** 50-200 tokens (concise).

**Q37:** What is role prompting?  
**A:** Explicitly assigning the model a role/persona to guide responses (e.g., "You are a senior software architect...").

**Q38:** What is contextual prompting?  
**A:** Providing relevant context/background information before the task to ground the model's response.

**Q39:** How do system, role, and contextual prompts differ?  
**A:** System: Global behavior, Role: Expertise/persona, Contextual: Situational grounding.

---

## 7. Advanced Prompt Techniques

**Q40:** What is prompt debiasing?  
**A:** Techniques to reduce model biases (gender, race, cultural) through explicit instructions, demographic neutrality, or counter-examples.

**Q41:** Give two strategies for prompt debiasing.  
**A:** (1) Use explicit instructions like "provide balanced view considering multiple perspectives", (2) Use gender-neutral language and diverse examples.

**Q42:** What is prompt ensembling?  
**A:** Generating multiple responses (with different prompts/params) and aggregating them for improved accuracy.

**Q43:** Name three methods of prompt ensembling.  
**A:** (1) Self-consistency (same prompt, different temps), (2) Multi-prompt (different phrasings), (3) Model ensemble (multiple models).

**Q44:** What's the cost-quality tradeoff for ensembling?  
**A:** 3-5x more expensive but 10-30% accuracy improvement.

**Q45:** What is LLM self-evaluation?  
**A:** Using the LLM itself to evaluate/critique its own outputs to catch errors and improve quality.

**Q46:** Describe the self-evaluation pattern.  
**A:** (1) Generate initial response, (2) Ask LLM to rate it 1-5, (3) If score < 4, ask what's wrong and regenerate.

**Q47:** What is LLM calibration?  
**A:** Adjusting model confidence scores to match actual accuracy (LLMs are often overconfident).

**Q48:** Why is calibration important in production?  
**A:** Helps decide when to route to human review in high-stakes decisions (medical, legal).

**Q49:** Name two calibration methods.  
**A:** (1) Temperature calibration using validation set, (2) Prompting for confidence scores.

---

## 8. Prompting Best Practices

**Q50:** How many few-shot examples should you provide?  
**A:** 2-5 examples (diminishing returns after 5).

**Q51:** Why should prompts be kept short and concise?  
**A:** Reduces cost, latency, and confusion.

**Q52:** What's the target token count for instruction prompts?  
**A:** <500 tokens

**Q53:** Should you tell the model what to do or what not to do?  
**A:** Tell what TO do - clearer than listing constraints.

**Q54:** Why is controlling max output length important?  
**A:** Prevents runaway generation and controls costs (output tokens cost 2-3x input).

**Q55:** Why use variables/placeholders in prompts?  
**A:** Easier to maintain, test, and configure at scale.

**Q56:** Name three input formats to experiment with.  
**A:** Plain text, JSON, XML, markdown, code blocks.

**Q57:** What's the recommended sampling for deterministic outputs?  
**A:** temperature=0.0, top_p=1.0

**Q58:** What's the recommended sampling for creative outputs?  
**A:** temperature=0.9, top_p=0.95

**Q59:** How should you guard against prompt injection?  
**A:** Sanitize input (remove special chars, limit length), delimit sections with XML tags, validate outputs.

**Q60:** What are three automation practices for prompt engineering?  
**A:** (1) Unit tests for format/content, (2) Regression tests across versions, (3) Automated metrics (accuracy, latency, cost).

**Q61:** How should prompts be versioned?  
**A:** Use Git for version control, maintain changelogs, perform A/B testing.

**Q62:** Name three ways to optimize for latency and cost.  
**A:** (1) Use smaller models, (2) Reduce max_tokens, (3) Remove redundant tokens, cache responses, batch requests.

**Q63:** What are three delimiters for prompt sections?  
**A:** (1) Triple backticks for code, (2) XML tags for structured data, (3) Headers (###) for organization.

---

## 9. Advanced Prompting Techniques

**Q64:** What's the difference between zero-shot and few-shot prompting?  
**A:** Zero-shot: Direct instruction without examples. Few-shot: Includes 2-5 examples in the prompt.

**Q65:** When should you use few-shot over zero-shot?  
**A:** Only when zero-shot fails, as few-shot increases token cost.

**Q66:** What is Chain-of-Thought (CoT) prompting?  
**A:** Instructing the model to "think step by step", improving reasoning by 20-40% on complex tasks.

**Q67:** What's the cost tradeoff of CoT?  
**A:** Generates 2-3x more tokens.

**Q68:** What is Tree-of-Thoughts (ToT)?  
**A:** Exploring multiple reasoning paths, self-evaluating, and backtracking (expensive and slow, rarely used in production).

**Q69:** What is ReAct prompting?  
**A:** Interleaving reasoning with tool calls: Thought → Action → Observation → repeat.

**Q70:** Where is ReAct commonly used?  
**A:** Standard pattern for agent frameworks.

**Q71:** What is self-consistency?  
**A:** Generating N responses and voting on the most common answer, improving accuracy 10-30%.

**Q72:** What's the cost of self-consistency?  
**A:** N times more expensive (if N=5, 5x cost).

---

## 10. Prompt Optimization Patterns

**Q73:** What is role-based prompting?  
**A:** Assigning specific expertise and behavioral expectations (e.g., "You are an expert Python developer with 10 years of experience").

**Q74:** What are the components of a structured prompt?  
**A:** Task, Context, Constraints, Output Format.

**Q75:** What is negative prompting?  
**A:** Telling the model what NOT to do (e.g., "Do not make up information. If unsure, say 'I don't know'").

**Q76:** What is the PCT Framework?  
**A:** Persona + Context + Task + Constraints + Examples.

---

## 11. Prompt Injection & Security

**Q77:** Name three prompt injection attack vectors.  
**A:** (1) Direct injection ("Ignore previous instructions"), (2) Indirect injection (malicious content in retrieved docs), (3) Jailbreaking (bypass safety filters).

**Q78:** What is data exfiltration in prompt injection?  
**A:** Tricking the model to reveal system prompts or sensitive information.

**Q79:** Name three defense strategies against prompt injection.  
**A:** (1) Input sanitization, (2) Prompt sandboxing with XML delimiters, (3) Output validation.

**Q80:** What should you do to prevent system prompt leakage?  
**A:** Separate system/user prompts using API's system role, not concatenated strings; validate outputs for leakage.

**Q81:** What are three production security best practices?  
**A:** (1) Block prompts >10K tokens and special characters, (2) PII detection and content safety checks, (3) Log and alert on injection patterns.

---

## 12. Prompt Versioning & Management

**Q82:** Why should prompts be version controlled?  
**A:** Prompts are code - need to track changes, A/B test, and rollback if quality degrades.

**Q83:** Name two prompt management tools.  
**A:** LangSmith (versioning, testing, monitoring), PromptLayer (management + logging).

**Q84:** How should prompt directories be structured?  
**A:** Versions (v1/, v2/), experiments folder for A/B tests.

---

## 13. Token Optimization

**Q85:** Name three techniques to reduce token count.  
**A:** (1) Remove redundancy (filler words), (2) Use abbreviations, (3) Compress few-shot examples.

**Q86:** What's more efficient: verbose instructions or JSON format?  
**A:** JSON format (much more concise).

**Q87:** If you reduce a 180-token prompt to 50 tokens, what's the cost savings?  
**A:** 72% fewer tokens = 72% lower cost.

---

## 14. Dynamic Prompting

**Q88:** What is context-adaptive prompting?  
**A:** Adjusting prompts based on context characteristics (e.g., if context is long, add "focus on key points").

**Q89:** What is user personalization in prompting?  
**A:** Adapting prompts based on user preferences (verbosity, formality) and history.

---

## 15. Multi-Turn Conversation Patterns

**Q90:** Name three context management strategies for long conversations.  
**A:** (1) Sliding window (last N turns), (2) Summarization (summarize old messages), (3) Semantic compression (keep only relevant messages).

**Q91:** What is system prompt reinforcement?  
**A:** Re-injecting key instructions every N turns to prevent instruction drift.

---

## 16. Constrained Generation

**Q92:** What is JSON mode?  
**A:** A response format setting that forces the LLM to output valid JSON.

**Q93:** What are grammar-based constraints?  
**A:** Using libraries (Outlines, Guidance, LMQL) to force output to match a schema.

**Q94:** How do you constrain output to specific formats like phone numbers?  
**A:** Use explicit format instructions and regex patterns in the prompt.

---

## 17. Prompt Testing & Evaluation

**Q95:** What should a test dataset contain?  
**A:** Input, expected output, context, and metadata (difficulty, category).

**Q96:** Name four evaluation metrics for prompts.  
**A:** (1) Exact match, (2) Semantic similarity (embeddings), (3) LLM-as-judge (rating), (4) Human eval.

**Q97:** What is regression testing for prompts?  
**A:** Running test suite on every prompt change, tracking metrics over versions, alerting if accuracy drops >5%.

---

## 18. Common Anti-Patterns

**Q98:** What's wrong with over-prompting?  
**A:** 2000-word prompts with redundant instructions - should be concise, structured, and tested instead.

**Q99:** What's the "magic prompt obsession" anti-pattern?  
**A:** Chasing perfect wording instead of focusing on RAG quality, model selection, or fine-tuning.

**Q100:** Why is using static prompts an anti-pattern?  
**A:** Same prompt for all users/contexts - should dynamically adapt based on query type.

**Q101:** What's wrong with editing prompts directly in code?  
**A:** No versioning - should use centralized prompt management with version control.

---

## 19. Production Prompt Template

**Q102:** What sections should a production system prompt include?  
**A:** Role, Behavior, Constraints, Safety guidelines.

**Q103:** What sections should a production user prompt include?  
**A:** Context (retrieved), Query (user input), Instructions (dynamic).

---

## 20. Cost-Quality Tradeoffs

**Q104:** What's the quality gain vs cost impact of few-shot prompting?  
**A:** +15-25% quality, +30-50% token cost.

**Q105:** What's the quality gain vs cost impact of Chain-of-Thought?  
**A:** +20-40% quality, +200-300% token cost.

**Q106:** What's the quality gain vs cost impact of self-consistency?  
**A:** +10-30% quality, +N× cost (if N=5, 5x cost).

**Q107:** When should you use CoT despite its high cost?  
**A:** For reasoning tasks where quality improvement justifies the 2-3x cost increase.

**Q108:** What's the last resort for improving quality?  
**A:** Using a better (larger) model, which costs +10-20× more.

**Q109:** When discussing prompt optimization techniques, what should you always mention?  
**A:** Cost implications and tradeoffs.

---

## Quick Numbers to Remember

- **Prompt Engineering ROI**: 100x cheaper than fine-tuning ($0 vs $1K-$10K)
- **Default temperature**: 0.7
- **Deterministic temperature**: 0.0-0.3
- **Creative temperature**: 1.5-2.0
- **Top-K typical**: 40-50
- **Top-P typical**: 0.9-0.95
- **Output cost multiplier**: 2-3x input tokens
- **System prompt size**: 50-200 tokens
- **Instruction prompt target**: <500 tokens
- **Few-shot examples**: 2-5 (diminishing returns after 5)
- **CoT quality improvement**: +20-40%
- **CoT cost increase**: +200-300% tokens
- **Self-consistency quality gain**: +10-30%
- **Ensembling cost**: 3-5x more expensive
- **Ensembling quality gain**: +10-30%
- **Token optimization savings**: Up to 72% in costs
- **Prompt engineering workload**: 80% of AI Engineering

---

## Interview Tips

1. **Frame as engineering**: Treat prompts like code - version control, testing, production deployment
2. **Always discuss tradeoffs**: Quality vs cost vs latency (critical for production)
3. **Show iteration**: First prompt rarely perfect - emphasize testing and refinement
4. **Security awareness**: Know prompt injection attacks and defenses
5. **Metrics matter**: Discuss specific improvements (e.g., "reduced costs 40%", "improved accuracy 25%")
6. **Few-shot judiciously**: Only when zero-shot fails (cost implications)
7. **Temperature is key**: Know when to use 0.0 (deterministic) vs 0.7 (balanced) vs 1.5 (creative)
8. **Production patterns**: System prompts, structured formats, validation
9. **Cost consciousness**: Output tokens cost 2-3x input; optimize token usage
10. **Advanced techniques**: CoT for reasoning, ReAct for agents, self-consistency for accuracy

---

*Reference: See [concepts.md](concepts.md) for detailed explanations of all topics*
