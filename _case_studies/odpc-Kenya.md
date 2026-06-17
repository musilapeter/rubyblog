---
title: "ODPC Kenya Shield Bot — AI-Powered RAG Chatbot"
short_title: "ODPC Kenya Shield Bot"
date: 2026-06-01 11:35:00 +0300
categories: [Work, Case Study]
tags: [FastAPI, Llama 3.1, ChromaDB, PostgreSQL, Docker, HuggingFace]
card_icon: "fas fa-shield-alt"
card_excerpt: "An intelligent AI chatbot for the Office of the Data Protection Commissioner Kenya, powered by Retrieval-Augmented Generation (RAG)."
order: 3
---

## Overview
An intelligent AI chatbot for the Office of the Data Protection Commissioner (ODPC) Kenya, powered by Retrieval-Augmented Generation (RAG). The system automatically crawls the official ODPC website, indexes content into a ChromaDB vector database, and provides accurate, source-backed answers to queries about Kenyan data protection laws.

## Key Features & Stack
- **FastAPI**: High-performance async handling for rapid query processing and API orchestration.
- **Llama 3.1 (Groq API)**: LLM backbone delivering accurate, context-aware responses about data protection regulations.
- **ChromaDB**: Vector database for semantic search and efficient content retrieval from indexed ODPC documents.
- **PostgreSQL**: Conversational memory storage for maintaining context across user sessions.
- **Docker**: Containerized deployment for consistent, reproducible environments.
- **HuggingFace Embeddings**: State-of-the-art sentence transformers for accurate document vectorization.

## The Technical Challenges
- **Automated Knowledge Base Construction**: Built a web crawler that automatically scrapes the ODPC Kenya website, processes legal documents, and indexes them into ChromaDB for real-time retrieval.
- **Source Attribution**: Every response includes direct source links back to the official ODPC pages, ensuring transparency and verifiability of legal information.
- **Conversational Memory**: PostgreSQL-backed session management allows multi-turn conversations where the bot remembers prior context within a session.
- **Semantic Search Accuracy**: Fine-tuned embedding models and retrieval parameters to ensure highly relevant document chunks are surfaced for legal queries.

## Metrics & Impact
- **Accurate Legal Responses**: Source-backed answers with direct links to official ODPC documentation.
- **Automated Indexing**: Zero manual intervention required to keep the knowledge base current.
- **Multi-Turn Conversations**: Persistent session context for natural, flowing user interactions.

## Links
- **ODPC Kenya**: [odpc.go.ke](https://www.odpc.go.ke/)
- **Source Code**: [GitHub Repository](https://github.com/musilapeter)
