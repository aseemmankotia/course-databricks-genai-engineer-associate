# Chapter 8 Lab: Governance with Unity Catalog and AI Guardrails

**Estimated time:** 15 minutes

## Exercise

Lab: Apply Unity Catalog GRANT SELECT on a Delta table with customer data to a specific user. Configure a Databricks AI Guardrails policy on a Model Serving endpoint to block PII in inputs. Test with a query containing an email address—verify it's blocked.

## Key takeaway

Unity Catalog governs models, indexes, and data with GRANT/REVOKE; guardrails block PII and harmful content at serving time; lineage tracks dependencies; lacking EXECUTE on a model = 403 error.
