This specification document outlines the architecture and functional requirements for a "Python Output Predictor" application. It leverages **Google Colab** as a remote execution environment via the **Model Context Protocol (MCP)**, allowing a separate application to orchestrate code execution and LLM-driven feedback.

---

## 1. Project Overview
The application is an interactive tutor where students guess the output of Python code snippets (under 10 lines). It uses a "Judge LLM" running inside a Google Colab notebook to compare the student's guess against the actual execution result and provide pedagogical feedback.

---

## 2. System Architecture
The system follows a three-tier model enabled by MCP:

* **Tier 1: MCP Host (Frontend Application)**
    * An interface (web or CLI) where students view code and enter guesses.
    * Manages user "streaks" and concept mastery levels.
* **Tier 2: MCP Server (The Bridge)**
    * A local process that connects the Host to the Colab Runtime.
    * Translates the Host's requests into remote execution commands.
* **Tier 3: Remote Sandbox (Google Colab)**
    * Runs the actual Python kernel to execute code snippets.
    * Hosts a local LLM (e.g., **Ollama/Qwen3-Coder**) to act as the "Judge" for feedback generation.



---

## 3. Functional Specification

### A. Code Bank & Concept Selection
The system maintains a library of snippets categorized by concept:
* **Concepts**: List comprehensions, Slicing, Dictionary methods, Variable scoping, etc.
* **Constraint**: Every snippet must be $\le$ 10 lines to ensure focus on a single logic gate.

### B. The Evaluation Loop (The "Judge" Workflow)
1.  **Selection**: The system picks a snippet based on the student's current practice focus.
2.  **Execution**: The code is sent to Colab via MCP and executed in a hidden cell to get the **Actual Output**.
3.  **Student Input**: The student submits their **Guess**.
4.  **Judging**: The snippet, Actual Output, and Student Guess are passed to the local LLM in Colab.
5.  **Feedback Generation**: The LLM analyzes the gap using a **Chain-of-Thought** prompt.

### C. Feedback Logic
The LLM response must follow a structured JSON format:
```json
{
  "is_correct": boolean,
  "actual_output": "string",
  "error_type": "None | Syntax | Logic | Misconception",
  "pedagogical_feedback": "Short explanation of the concept (e.g., 'You missed that slicing [:-1] excludes the last element.')",
  "recommendation": "Repeat Concept | Move to Next"
}
```

---

## 4. Technical Configuration

### MCP Server Setup
To connect the application to Colab, the environment uses the following configuration:
* **Transport**: Standard Input/Output (stdio) for local agents or SSE for remote.
* **Command**: `uvx` to run the open-source Google Colab MCP server.

### Colab Local LLM Engine
To avoid external API latency and costs, the Colab environment runs **Ollama**:
* **Model**: `qwen3:0.6b` (for speed) or `llama4:8b` (for accuracy).
* **Execution**: Triggered via `ollama run` commands within a Colab cell.

---

## 5. User Mastery Flow
* **Correct Guess**: Student earns +1 mastery in that concept.
* **Incorrect Guess**: The system identifies the specific "misconception" (e.g., confusing `append` vs `extend`) and selects a different code snippet targeting that exact behavior for the next turn.
* **Persistence**: Progress is tracked across sessions using the MCP server's ability to maintain persistent notebook context.

---

**Next Steps**: Would you like a prompt template for the "Judge LLM" to ensure it identifies specific Python misconceptions accurately?
