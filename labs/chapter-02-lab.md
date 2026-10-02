# Chapter 2 Lab: Preparing Data for RAG on Databricks

**Estimated time:** 20 minutes

## Exercise

Lab: Chunk a 20-page PDF with recursive chunking (chunk_size=512, overlap=50), generate embeddings with BGE-large, write to a Delta table, create a Vector Search index from that table

## Key takeaway

Delta tables are the source of truth—chunks and embeddings live there, Vector Search indexes sync from them; chunk overlap solves boundary context loss; recursive chunking respects structure.
