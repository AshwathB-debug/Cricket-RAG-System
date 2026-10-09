# Cricket RAG System

A Retrieval-Augmented-Generation (RAG) chatbot that answers cricket questions using live-ingested Wikipedia data, a locally-hosted small language model, and an LLM-judged evaluation harness to measure answer quality.

## Overview

This system retrieves cricket-related data from Wikipedia, feeds it as context to an SLM, and returns an answer backed by a quoted evidence excerpt. It was built to test how well a small open-weight model (Qwen3-4B) can stay grounded in retrieved context through prompt engineering alone, without fine-tuning.

The pipeline is a full ETL process:
- **Extract** — pulls pages from Wikipedia via `WikipediaReader` through LlamaIndex, with fallback handling for disambiguation and missing pages.
- **Transform** — cleans raw text (strips bibliography/reference sections, converts MediaWiki headers to Markdown), chunks it, and generates vector embeddings.
- **Load** — stores vectors in Pinecone database

## Architecture / Tech Stack

| Component | Tool |
|---|---|
| LLM (generation) | Qwen 3-4B (Hugging Face Transformers) |
| Embedding model | Qwen 3-Embedding-0.6B |
| Vector database | Pinecone (serverless, cosine similarity) |
| Orchestration | LlamaIndex (`IngestionPipeline`, `MarkdownNodeParser`, `SentenceSplitter`) |
| Data source | Wikipedia (`WikipediaReader`) |
| Frontend | Streamlit |
| Tunneling | Cloudflare Tunnel (`cloudflared`) |
| Evaluation judge | Claude (Anthropic API) |

## Results

Evaluated on a custom 50-question benchmark spanning cricket rules, tournaments, formats, and players, with a Claude-graded judge scoring each answer as CORRECT / INCORRECT / UNSUPPORTED (UNSUPPORTED = claim may be true but isn't backed by the retrieved context).

| Metric | Score |
|---|---|
| Judge-verified correct | **82%** |
| Retrieval hit rate | **86%** |

Both figures were stable across repeated runs (deterministic generation + deterministic judge configuration), rather than a single best-case result.

### Known limitations
- The model occasionally conflates details between similar events (e.g. mixing up a tournament final and semi-final when both appear in the same retrieved context).
- On a small number of questions, the model answers correctly from its own pretrained knowledge rather than the retrieved context — a faithfulness gap distinct from factual accuracy, since a RAG system should ideally say "not found" rather than fall back on parametric knowledge.
- When using quotations, the model sometimes won't be able to provide a reliable inference. 

## How to Run

1. **Get API keys**: [Pinecone](https://docs.pinecone.io/guides/get-started/quickstart) (required) and [Claude](https://platform.claude.com/docs/en/get-api-key) (optional — only needed to run the evaluation harness, not the chatbot itself). Set them as environment variables (`PINECONE_API_KEY`, `ANTHROPIC_API_KEY`) or paste them directly in place of the `"INSERT_YOUR_API_KEY_HERE"` placeholders in the code.
2. **Use a GPU runtime.** Qwen3-4B is downloaded and run locally via Transformers, so a GPU is strongly recommended. [Google Colab](https://colab.research.google.com/) is a good starting point — the free tier's T4 GPU works, though a Colab Pro subscription ($9.99/mo) gives access to faster GPUs like the A100 or G4. On Intel hardware, OpenVINO can be used instead of Transformers, with a noticeable performance tradeoff.
3. **Running the chatbot only**: run every cell in order, skipping the evaluation cell.
4. **Running the full evaluation**: run the dependency-install cell, then the RAG system cell, then the evaluation cell last.



## Acknowledgments

This project uses the following open-source/open-weight models and datasets:
- [Qwen3-4B](https://huggingface.co/Qwen/Qwen3-4B) (Apache 2.0) — generation model
- [Qwen3-Embedding-0.6B](https://huggingface.co/Qwen/Qwen3-Embedding-0.6B) (Apache 2.0) — embedding model
- Wikipedia content, used under [CC BY-SA 4.0](https://en.wikipedia.org/wiki/Wikipedia:Copyrights)