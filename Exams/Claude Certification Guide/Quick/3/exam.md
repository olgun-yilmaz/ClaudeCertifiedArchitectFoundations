# QUESTIONS

## Agentic Architecture & Orchestration (3/6)

### 1.4 Workflow Enforcement & Handoff
![Section 1.4 - Question 21](Screenshots/1-AgenticArchitectureOrchestration/1.4-Q21.png)

### Inferences:

- Splitting the request accross agents add s *coordination overhead* and may lose the shared context between the two issues. 
- The correct pattern is multi-concern decomposition within the agent's workflow, investigate each issue in parallel with shared context.  


### 1.5 Agent SDK Hooks
![Section 1.5 - Question 8](Screenshots/1-AgenticArchitectureOrchestration/1.5-Q8.png)

### Inferences:

- **Don't** use hooks for everything. Prompt is acceptable for a *non critical* best practice.

---

![Section 1.5 - Question 14](Screenshots/1-AgenticArchitectureOrchestration/1.5-Q14.png)

### Inferences:

- **PreToolUse** hooks intercept **outgoing** tool calls, not incoming results.
- **PostToolUse** is the correct hook type for redacting data **from results**.

---

## Tool Design & MCP Integration (5/5)

> 🟢 **Evaluation:** 5/5 (All questions were answered correctly.)

---

## Claude Code Configuration & Workflows (5/6)

### 3.5 Iterative Refinement Techniques
![Section 3.5 - Question 16](Screenshots/3-ClaudeCodeConfigurationWorkflows/3.5-Q16.png)

### Inferences:

- It was overlooked. There is no need to take notes.

---

## Prompt Engineering & Structured Output (4/6)

### 4.1 Prompts with Explicit Criteria
![Section 4.1 - Question 15](Screenshots/4-PromptEngineeringStructuredOutput/4.1-Q15.png)

### Inferences:

- When said "system prompt" you need fix overlap in the instruction. Descriptions are not first solution.

### 4.5 Batch Processing Strategies
![Section 4.5 - Question 5](Screenshots/4-PromptEngineeringStructuredOutput/4.5-Q5.png)

### Inferences:

- The **custom_id** field exists specifically for correlating batc request/response pairs.

---

## Context Management & Reliability (4/5)

### 5.4 Context in Large Codebase Exploration
![Section 5.4 - Question 3](Screenshots/5-ContextManagementReliability/5.4-Q3.png)

### Inferences:

- **Jitter** prevents multiple clients from synchronising their retries and creating burst patterns.

---

# RESULT

![Result report](Screenshots/Result.png)

---
