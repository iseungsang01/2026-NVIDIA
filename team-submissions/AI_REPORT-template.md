> **Note to Students:** > The questions and examples provided in the specific sections below are **prompts to guide your thinking**, not a rigid checklist. 
> * **Adaptability:** If a specific question doesn't fit your strategy, you may skip or adapt it.
> * **Depth:** You are encouraged to go beyond these examples. If there are other critical technical details relevant to your specific approach, please include them.
> * **Goal:** The objective is to convince the reader that you have employed AI agents in a thoughtful way.

**Required Sections:**

# AI Agent Utilization Report: Code Evaluation Lab Solutions

## 1. The Workflow: Triple-Agent Orchestration
I organized a hierarchical AI workflow to ensure that code logic, environmental constraints, and performance optimization were handled by specialized agents.

* **Logic Architect (ChatGPT-4o):** Acted as the lead developer to scaffold the initial OOP structure. It interpreted the core requirements of the **Circular Queue** and **Coordinate Grouping** algorithms.
* **Technical Researcher (Perplexity AI):** Functioned as a "Compliance Officer" to cross-reference the **Code Evaluation Lab**'s environment. It verified that all proposed solutions used only Python standard libraries, preventing the use of forbidden external packages like `numpy`.
* **Performance Optimizer (Gemini 3 Flash):** Served as the **Senior Code Reviewer**. Gemini was used to refactor the initial drafts for time complexity, identifying bottlenecks in nested loops and suggesting memory-efficient data structures (e.g., swapping lists for `collections.deque`).

---

## 2. Verification Strategy
To combat AI hallucinations—where code looks correct but fails on edge cases—I implemented a rigid validation layer focused on **Constraint Validation**.

### Unit Testing Against Hallucinations
I wrote specific tests to catch "Logic Hallucinations" where the AI assumed the "happy path":

* **Circular Buffer Boundary Test:** AI often fails the "Full" vs. "Empty" state logic. I wrote a test to fill the queue, dequeue one item, and enqueue another to ensure the pointers ($i \pmod N$) wrapped around correctly without overwriting valid data.
* **The "Exact Threshold" Test:** In the distance grouping task, I tested points at exactly $D = K$. This verified if the AI correctly used `distance <= threshold`. This test caught a logic error where the AI used `<` instead of `<=`, which would have caused failures on hidden test cases.

---

## 3. The "Vibe" Log

### 🏆 Win: Gemini’s Computational Efficiency
* **Scenario:** A grouping algorithm was failing due to a "Time Limit Exceeded" (TLE) error on large datasets ($N > 10,000$).
* **The Save:** I fed the code to **Gemini 3 Flash**. It immediately noticed that calculating `math.sqrt()` inside a nested loop was redundant. By suggesting a comparison of **squared distances** ($dist^2 \leq K^2$) instead of calculating the square root every time, the execution speed increased by ~30%, passing the TLE constraint.

### 💡 Learn: From Task-Based to Skill-Based Prompting
* **The Shift:** Initially, I asked for "the whole solution," which resulted in monolithic, buggy code.
* **The Lesson:** I altered my strategy by providing a `skills.md` context. I told the AI: *"First, define the Data Structure. Second, implement only the logic for 'wrap-around'. Do not write the full class until I approve the pointers."* This "Atomic Prompting" led to 100% logical accuracy in the first draft.

### ❌ Fail: The "Shadow Library" Hallucination
* **The Fail:** Perplexity suggested using `scipy.spatial.KDTree` for optimal clustering. However, the lab environment was restricted to standard Python.
* **The Fix:** I had to manually intervene and "downgrade" the AI's intelligence. I prompted: *"Assume no external libraries. Rewrite the clustering using only basic lists and recursion."* This forced the AI to provide a compliant solution.

---

## 4. Context Dump

### Gemini Optimization Prompt:
> "Act as a Senior Python Performance Engineer. Review this Circular Queue implementation for Code Evaluation Labs. Identify any operations with $O(N)$ complexity that can be reduced to $O(1)$. Ensure no external imports are used."

### `skills.md` / Logic Rules:
* **Constraint Awareness:** Always check if $N > 10^5$ to decide between $O(N)$ or $O(N \log N)$.
* **Precision:** Use squared distance comparisons for coordinate geometry to avoid floating-point overhead.
* **Safety:** Every input must be validated for `None` or empty list states before processing to prevent `AttributeError`.
