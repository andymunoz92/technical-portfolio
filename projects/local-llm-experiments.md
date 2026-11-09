# 🤖 Local LLM & Database-Backed Experiments

> **Status:** exploration/prototyping; not described as a production system.

[← Back to portfolio](../README.md)

## Overview

Experimented with running language models locally through **Ollama** and connecting application logic to **PostgreSQL**. One use case explored combining multiple records of descriptive text and creating a compact consolidated description for analysis.

## Concepts explored

- Local model serving and inference configuration.
- Prompt design for structured inputs and concise outputs.
- Context-window and generation-length trade-offs.
- Retrieving and grouping records from a relational database before model inference.
- Separating research/classification intent from unrestricted text generation.

## Planned demo

A small, synthetic-data example that demonstrates: SQL lookup → input grouping → model request → validated output. Keep all service credentials in local environment files outside version control.

**Tools:** Ollama, Python, PostgreSQL, Linux/WSL2.
