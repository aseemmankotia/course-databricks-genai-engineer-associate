# Governance with Unity Catalog and AI Guardrails - Exam Cheat Sheet

## Unity Catalog Permissions for GenAI Assets

| Object Type | Permission | Grants Ability To |
|------------|-----------|------------------|
| Registered Model | EXECUTE | Run inference via Model Serving (query model) |
| Registered Model | MODIFY | Create new versions, update metadata |
| Vector Search Index | EXECUTE | Query index (similarity search) - **independent of source table** |
| Delta Table | SELECT | Read table data (required for training, NOT for querying index) |
| Function (UDF) | EXECUTE | Call function in queries or models |

**Key Exam Concept:** Vector Search index EXECUTE permission does NOT require SELECT on underlying Delta table. Users can query embeddings without seeing raw data.

### GRANT Syntax Examples
```sql
-- Allow model inference only
GRANT EXECUTE ON MODEL catalog.schema.model_name TO user@company.com;

-- Service principal (use backticks + app ID)
GRANT EXECUTE ON MODEL my_model TO `a1b2c3d4-uuid-format`;

-- Vector Search index (no table permission needed)
GRANT EXECUTE ON VECTOR SEARCH INDEX support_index TO data_team;

-- Revoke permissions
REVOKE MODIFY ON MODEL my_model FROM analyst@company.com;
```

## Data Lineage
**Automatic tracking:** Delta table → Vector Search index → Registered model → Model Serving endpoint  
**View:** Unity Catalog UI → Data Lineage graph  
**Purpose:** Shows dependencies, impact analysis ("which models use this table?"), compliance documentation

## Audit Logs
**Table:** `system.access.audit`  
**Contains:** principal_email, action_name (EXECUTE, SELECT, MODIFY), request_timestamp, object_type (MODEL, TABLE, INDEX), object_name  
**Query example:**
```sql
SELECT principal_email, request_timestamp, object_name
FROM system.access.audit
WHERE action_name = 'EXECUTE' 
  AND object_type = 'MODEL'
  AND request_timestamp > current_date() - INTERVAL 30 DAYS;
```
**Exam tip:** system.access.audit = compliance auditing; system.billing.usage = cost tracking (different tables)

## Databricks AI Guardrails

| Filter Type | Detects | Action |
|------------|---------|--------|
| **pii** | SSN, email, credit card, phone numbers | Block request OR mask PII (configurable) |
| **toxicity** | Hate speech, profanity, offensive language | Block request, return error |
| **prompt_injection** | Jailbreaks ("ignore instructions"), prompt leaking | Block request before LLM |
| **custom** | User-defined regex, keyword lists | Block or mask |

**Configuration:** Model Serving endpoint → AI Guardrails section → Enable filters  
**Default behavior:** BLOCK request (doesn't send to LLM)  
**Exam trap:** Guardrails blocks by default, not auto-mask. Masking is opt-in configuration.

### Guardrails vs Llama Guard
| Feature | Databricks AI Guardrails | Llama Guard |
|---------|-------------------------|-------------|
| **Focus** | PII, jailbreaks, toxicity (input filtering) | Content safety categories (violence, hate, sexual, self-harm) |
| **Type** | Managed service (built-in to Databricks) | Open-source model (7B parameters, run on inference) |
| **When to use** | Pre-LLM filtering (block bad inputs) | Post-LLM filtering (classify output safety) |
| **Categories** | pii, toxicity, prompt_injection | S1-S7 safety taxonomy |

**Exam tip:** Use BOTH together - Guardrails for PII/jailbreaks, Llama Guard for content safety classification.

## Llama Guard Safety Taxonomy (S1-S7)

| Category | Content Type | Example |
|----------|-------------|----------|
| **S1** | Violence & Harm | Instructions for violence, graphic injury |
| **S2** | Hate Speech | Attacks on race, religion, gender, etc. |
| **S3** | Sexual Content | Explicit sexual descriptions |
| **S4** | Guns & Weapons | Weapon acquisition, bomb-making |
| **S5** | Regulated Substances | Drug synthesis, illegal substance advice |
| **S6** | Self-Harm | Suicide encouragement, eating disorders |
| **S7** | Criminal Planning | Fraud, hacking, theft instructions |

**Exam frequently tests:** S3 (Sexual) vs S6 (Self-Harm) distinction. Remember: S3=Sexual, S6=Self-Harm.

## PII Detection & Masking

**PII Types Detected:**
- Social Security Numbers (SSN): `\d{3}-\d{2}-\d{4}` or `\d{9}`
- Email addresses: standard email regex
- Credit card numbers: Visa, Mastercard, Amex patterns
- Phone numbers: (123) 456-7890, 123-456-7890 formats

**Masking vs Blocking:**
- **Block (default):** Request rejected, LLM not called, error returned to user
- **Mask (opt-in):** Replace PII with `[MASKED_SSN]`, `[MASKED_EMAIL]`, send masked text to LLM

**Exam tip:** Guardrails PII detection is for INPUT prompts (pre-LLM). If question asks about PII in training data, answer is Delta table column masking (different concept).

## Legal & Licensing

**Where to find model license terms:**
1. **Hugging Face model card:** License section (Apache 2.0, MIT, proprietary, etc.)
2. **Model Intended Use section:** Commercial use restrictions, training data provenance
3. **Unity Catalog metadata:** Can store license info as tags (but model card is authoritative source)

**Common license types:**
- Apache 2.0 / MIT: Commercial use OK, attribution required
- Creative Commons: May restrict commercial use (check specific CC license)
- Proprietary: Check vendor terms (OpenAI, Anthropic have specific TOS)

**Exam trap:** Unity Catalog stores license metadata but isn't the authoritative source. Always reference original model card for legal compliance.

## Quick Command Reference
```sql
-- Grant model inference permission
GRANT EXECUTE ON MODEL my_catalog.my_schema.my_model TO user@company.com;

-- Grant Vector Search query (no table permission needed)
GRANT EXECUTE ON VECTOR SEARCH INDEX my_index TO service_principal;

-- Audit log query (model access last 30 days)
SELECT * FROM system.access.audit 
WHERE action_name = 'EXECUTE' 
  AND object_type = 'MODEL'
  AND event_date > current_date() - 30;

-- View lineage (UI only, not SQL queryable)
-- Navigate to: Catalog → Model → Data Lineage tab
```

**Exam weighting:** Governance = 8% (3-4 questions). Focus on Unity Catalog permissions (EXECUTE vs SELECT), Guardrails vs Llama Guard differences, PII blocking vs masking, and audit log querying.
