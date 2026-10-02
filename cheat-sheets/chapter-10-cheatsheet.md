# Chapter 10: Full Practice Exam & Test-Taking Strategy

## Exam Format (Memorize)
- **Questions**: 45 scored (may see additional unscored pilot questions)
- **Time**: 90 minutes (2 min/question pace)
- **Passing**: 70/100 (~31-32 correct answers)
- **Format**: Single-answer multiple choice (4 options)
- **Tools**: Online proctored, no external resources

## Domain Weights (Study Priority)
| Domain | Weight | Questions (~) | Priority |
|--------|--------|---------------|----------|
| Application Development | 30% | 13-14 | HIGH |
| Deployment & Operations | 22% | 10 | HIGH |
| Design & Architecture | 14% | 6-7 | MED |
| Data Preparation | 14% | 6-7 | MED |
| Evaluation & Governance | 12% | 5-6 | MED |
| Governance & Security | 8% | 3-4 | LOW |

## Time Management Strategy
1. **First Pass (60 min)**: Answer all questions at 2 min pace, flag uncertain ones
2. **Review Pass (20 min)**: Re-read flagged questions, change answers if certain
3. **Final Check (10 min)**: Verify no unanswered questions, spot-check calculations
4. **Rule**: Never leave questions blank—unanswered = 0 points, guessed = 25% chance

## Common Exam Traps (Eliminate Wrong Answers)

### Trap 1: Over-Engineering
**Pattern**: Scenario has simple requirement, complex option sounds impressive  
**Example**: "Store 1000 documents" → WRONG: "Multi-region replicated Delta table with CDC"  
**Correct**: Basic Delta table with Vector Search  
**How to Spot**: Option includes features not mentioned in requirements

### Trap 2: Under-Scoping
**Pattern**: Option meets main requirement but misses a constraint  
**Example**: "Must support Unity Catalog governance" → WRONG: External vector DB  
**Correct**: Delta table + Vector Search (native UC support)  
**How to Spot**: Re-read requirements, check EVERY constraint is met

### Trap 3: Deprecated/Old Approach
**Pattern**: Option uses older tool/method that's been replaced  
**Example**: WRONG: Manual MLflow tracking → CORRECT: Auto-logging with transformers flavor  
**How to Spot**: If it sounds manual/complex, likely outdated

### Trap 4: Databricks-Specific vs Generic
**Pattern**: Generic GenAI answer that doesn't use Databricks tools  
**Example**: "Use OpenAI API directly" vs "Use Foundation Model API"  
**Rule**: Exam tests Databricks platform—prefer Databricks-native solutions

## Key Numbers That Appear on Exam

### Vector Search
- **Dimensions**: 384, 768, 1024, 1536 (must match model ↔ index)
- **Index Types**: Delta Sync (auto-sync from Delta), Direct Vector Access (manual/external)

### Model Serving
- **Compute**: CPU_SMALL, GPU_SMALL, GPU_MEDIUM, GPU_LARGE
- **Cold Start**: 30-60 sec (avoid with min_instances ≥ 1)
- **Latency Requirement**: <200ms → requires warm endpoint

### MLflow
- **Flavors**: `transformers`, `sentence_transformers`, `langchain`, `openai`, `pyfunc`
- **Deployment Rule**: Must use recognized flavor for Model Serving

### Unity Catalog Permissions
- **Catalog/Schema**: USAGE (navigate), CREATE (add objects)
- **Table**: SELECT (read), MODIFY (write)
- **Index**: USAGE (query), MODIFY (update)
- **Minimum for RAG**: USAGE (catalog + schema) + SELECT (table) + USAGE (index)

## Scenario Question Anatomy
1. **Requirement**: "Deploy a chatbot that..."
2. **Constraints**: "...responds in <200ms, handles 1000 users, minimizes cost"
3. **Approach**: 
   - Identify ALL constraints (latency, scale, cost, governance)
   - Eliminate options that violate ANY constraint
   - Choose simplest option that meets ALL requirements

## Evaluation Metrics (Common Questions)
- **High Precision, Low Recall**: Model too conservative (false negatives) → optimize recall
- **Low Precision, High Recall**: Model too aggressive (false positives) → optimize precision  
- **Class Imbalance**: Accuracy misleading → use F1, precision/recall per class
- **Exam Pattern**: Given metrics + problem → choose correct optimization strategy

## Day-Before Checklist
- [ ] Review flashcards: numbers, limits, permission types
- [ ] Re-read domain summaries (chapters 1-9)
- [ ] Run through practice exam questions flagged as "commonly missed"
- [ ] No new material—focus on reinforcing weak areas
- [ ] Good sleep (exam is mental endurance test)

## During-Exam Tactics
- Read question stem BEFORE reading options (know what's being asked)
- Eliminate obviously wrong answers first (2-3 usually clear)
- If stuck between 2 options: re-read requirements, find the constraint that differentiates
- Trust your preparation—if unsure, go with first instinct
- Flag and move on if spending >3 minutes on one question
- Final 10 minutes: verify no blank answers, don't second-guess unless certain

**Final Tip**: The exam tests *Databricks* GenAI Engineering, not just GenAI. When in doubt, choose the answer that uses Databricks-native tools (Model Serving, Vector Search, Unity Catalog, MLflow) over generic/external solutions.
