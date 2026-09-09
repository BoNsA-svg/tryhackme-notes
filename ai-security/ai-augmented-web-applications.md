# From Traditional to AI-Augmented Web Applications

## 1. Traditional vs. AI-Augmented Apps

Traditional applications have predictable data flows:

**User → UI → API → Database → Response**

AI-augmented applications introduce new components such as **LLMs, vector stores, RAG, prompt construction, and AI tools**, creating new security risks.

| Area         | Traditional App      | AI-Augmented App                |
| ------------ | -------------------- | ------------------------------- |
| Input        | Forms/API parameters | Free-form natural language      |
| Processing   | Deterministic code   | Probabilistic LLM inference     |
| Data Access  | Database queries     | RAG/model-mediated retrieval    |
| Output       | Templates            | AI-generated text               |
| Dependencies | Libraries/frameworks | Libraries + models + embeddings |

### Key Security Change

The biggest change is **structured → unstructured input**.

Traditional apps expect specific inputs such as numbers, dates, or predefined choices.

AI systems can accept almost **any natural-language input**, making traditional input validation insufficient and introducing new attack surfaces.

---

# 2. TryAssist Architecture

TryAssist contains **9 major components**:

1. **User Interface**

   * Chat interface used by developers.

2. **API Gateway**

   * Authentication
   * Rate limiting
   * Request routing

3. **Orchestration Layer**

   * Manages conversations.
   * Routes requests.
   * Coordinates AI components.

4. **Prompt Construction**

   * Combines:

     * System prompt
     * User input
     * Retrieved context

5. **LLM**

   * Generates responses.
   * May be internally hosted or accessed through an API.

6. **Tool Layer**

   * Allows the LLM to perform actions such as:

     * Database queries
     * Documentation searches
     * CI/CD checks

7. **Output Processing**

   * Formats responses.
   * Applies content/security filters.
   * Enforces output limits.

8. **Logging & Monitoring**

   * Stores conversations.
   * Tracks usage.
   * Creates audit trails.

9. **Vector Store**

   * Stores embeddings of documents.
   * Used by **RAG (Retrieval-Augmented Generation)** to provide relevant information to the LLM.

---

# 3. Trust Boundaries

A **trust boundary** is a point where data moves between different security contexts.

Every trust boundary should be considered a potential **attack surface**.

### Five TryAssist Trust Boundaries

**1. User → System**

* Untrusted natural-language input enters the application.

**2. System → LLM**

* System prompt + user input + retrieved context are sent to the model.

**3. LLM → Tools**

* LLM output may trigger actions such as API calls, database queries, or file operations.

**4. External Data → System**

* Documents or other external information enter the LLM's context through RAG.

**5. System → User**

* AI-generated content is returned to the user.

---

# 4. Data Flow of One Request

Example user request:

> "Does this pull request handle authentication correctly?"

Flow:

**1. User Input**
Developer submits the question.

↓

**2. API Gateway**
Authenticates user and applies rate limits.

↓

**3. Orchestration Layer**
Loads conversation history and determines what components are needed.

↓

**4. Prompt Construction**
Combines:

`System Prompt + User Input + Retrieved RAG Context`

↓

**5. LLM**
Processes the prompt and generates a response.

↓

**6. Tool Request (if needed)**
The LLM may request an action such as checking CI/CD status.

↓

**7. Tool Execution**
The tool performs the authorized action and sends the result back.

↓

**8. LLM Final Response**
The model incorporates the tool result.

↓

**9. Output Processing**
Filters and formats the response.

↓

**10. User + Logging**
The response is delivered and the interaction is logged.

---

# 5. Security Takeaway

The main security problem with AI applications is that **untrusted data can travel through many components and influence model behavior or actions**.

When analyzing an AI application, ask:

* **Where does untrusted data enter?**
* **Where does it cross a trust boundary?**
* **Can it influence the LLM?**
* **Can the LLM invoke tools or access sensitive data?**
* **Is retrieved RAG data trusted?**
* **Is the generated output validated?**
* **Are important actions logged and monitored?**

### Easy Way to Remember

**Input → Prompt → LLM → Tools → Output**

At every step:

**What can the attacker control, what can the AI access, and what security control protects the boundary?**

