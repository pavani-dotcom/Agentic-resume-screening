<img width="1536" height="1024" alt="ChatGPT Image Jan 19, 2026, 12_27_50 PM" src="https://github.com/user-attachments/assets/8c96e0a3-463d-42a5-891b-9ae6643fcd40" /># **Pitcrew Agentic Resume Screening Assistant**  

## **Overview**  
This system automates the initial screening of job candidates by analyzing resumes against job descriptions. It uses an **agentic architecture**—three specialized agents working in sequence—to extract structured data, compare qualifications, and produce a transparent, explainable hiring decision. The solution is designed to run locally, requires no payment for external APIs, and handles real-world edge cases like ambiguous job posts or parsing errors.


---

## **Architecture**  

### **Agent Design**  
The system implements three distinct agents, each with a single responsibility:  

1. **Resume Parser Agent**  
   - Input: Raw text from a PDF resume  
   - Output: Structured candidate profile (`name`, `years_experience`, `skills`, `education`, `domain_experience`)  
   - Uses LLM to extract and normalize unstructured resume content  

2. **Job Description Parser Agent**  
   - Input: Plain-text job description  
   - Output: Structured requirements (`role`, `required_skills`, `required_experience_years`, `strictness_level`)  
   - Classifies job strictness as `"strict"`, `"flexible"`, or `"vague"`  

3. **Matcher & Decision Agent**  
   - Input: Candidate profile + job requirements  
   - Output: Final decision (`match_score`, `recommendation`, `requires_human`, `confidence`, `reasoning_summary`)  
   - Uses **rule-based logic** (not an LLM) for transparency and efficiency  

### **Orchestration with LangGraph**  
Agents are orchestrated using **LangGraph**, which:  
- Manages a shared, typed state object that evolves through the workflow  
- Ensures clean separation of concerns and explicit data passing  
- Enables future extensibility (e.g., adding human review nodes)  

### **LLM Provider**  
- Uses **Groq’s `llama-3.1-8b-instant`** model  
- Chosen for its **unlimited free tier**, speed, and reliability  
- Avoids Google Gemini due to quota exhaustion (`429` errors) on experimental models  

---

## **Key Features**  

###  **Explainable Decisions**  
Every output includes a human-readable `reasoning_summary`, e.g.:  
> *"Match score 0.37. Matched skills: ['python', 'postgresql']. Experience: 2y vs 4y required. Strictness: strict."*  

###  **Human-in-the-Loop Logic**  
- **Vague JDs** → Always `requires_human: true`  
- **Strict roles**:  
  - Score < 0.3 → Auto-reject  
  - Score 0.3–0.85 → Manual review  
  - Score ≥ 0.85 → Proceed  
- **Flexible roles**: Lower thresholds for manual review  

### ✅ **Robust Error Handling**  
- **PDF parsing failures**: Gracefully reported  
- **LLM JSON errors**: Robust extraction using `{...}` delimiters  
- **LLM call failures**: Fallback to default structured profiles (no crash)  
- **Model deprecation**: Migrated from `llama3-8b-8192` → `llama-3.1-8b-instant` per Groq’s Aug 30, 2025 deprecation notice  

---

## **How to Run**  

### **Prerequisites**  
1. A **Groq API key** (free): [https://console.groq.com/keys](https://console.groq.com/keys)  
2. Google Colab (or local Python environment)  

### **Steps**  
1. **Set up secrets in Colab**:  
   - Runtime → Manage secrets → Add `GROQ_API_KEY`  

2. **Run the notebook/script**:  
   - The system will prompt you to upload:  
     - A resume (PDF)  
     - A job description (TXT)  

3. **View output**:  
   - The final JSON decision prints automatically  

> **Note**: The system is designed for Colab but can be adapted to run locally with minor path adjustments.

---

## **Sample Outputs**  

### **Strong Match**  
**Resume**: `resume_01_priya_sharma.pdf`  
**JD**: `jd_01_backend_python_standard.txt`  
```json
{
  "match_score": 0.88,
  "recommendation": "Proceed to interview",
  "requires_human": false,
  "confidence": 0.95,
  "reasoning_summary": "Match score 0.88. Matched skills: ['python', 'fastapi', 'postgresql']..."
}
```

### **Gray Zone (Manual Review)**  
**Resume**: `resume_02_rahul_verma.pdf`  
**JD**: `jd_02_senior_fintech_strict.txt`  
```json
{
  "match_score": 0.37,
  "recommendation": "Needs manual review",
  "requires_human": true,
  "confidence": 0.75,
  "reasoning_summary": "Match score 0.37. Matched skills: ['python', 'postgresql']..."
}
```

### **Clear Mismatch**  
**Resume**: `resume_03_ananya_patel.pdf`  
**JD**: `jd_03_junior_flexible.txt`  
```json
{
  "match_score": 0.28,
  "recommendation": "Reject",
  "requires_human": false,
  "confidence": 0.85,
  "reasoning_summary": "Match score 0.28. Matched skills: []. Experience: 0y vs 0y required..."
}
```

---

## **Trade-offs & Assumptions**  

| Decision | Reason |
|--------|--------|
| **Rule-based matcher** | More transparent and quota-efficient than a third LLM call |
| **Exact skill matching** | No synonym mapping (e.g., “Postgres” ≠ “PostgreSQL”) — out of scope |
| **Sequential agent flow** | Simpler and sufficient; parallelism not needed for this use case |
| **PDF-only input** | Word (.docx) support deferred to future work |

---

## **Future Improvements**  

- Add **caching** of parsed resumes/JDs to reduce LLM calls  
- Implement **skill normalization** (e.g., via embeddings or a knowledge base)  
- Integrate a **human review node** in LangGraph (pause workflow for manual input)  
- Support **Word (.docx)** resumes via `python-docx`  
- Add **unit tests** for matcher logic and edge cases  

---

## **Why This Approach?**  

This system reflects real-world engineering priorities:  
- **Clarity over complexity**: Rule-based decisions are auditable  
- **Resilience over fragility**: Graceful degradation ensures uptime  
- **Efficiency over excess**: Minimal LLM calls stay within free tiers  
- **Agentic design**: Specialized components > monolithic prompts  

It fulfills all Pitcrew requirements while demonstrating thoughtful, production-ready design.
