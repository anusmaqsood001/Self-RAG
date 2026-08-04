# Self-RAG

A self-contained Retrieval-Augmented Generation (RAG) prototype for answering company-specific questions using internal PDF documents.

## Overview

This project demonstrates a business-focused RAG pipeline built with LangChain-compatible components and Google Gemini for text generation.

Key features:
- PDF document loading and chunking
- Vector embedding with Hugging Face `all-MiniLM-L6-v2`
- FAISS-based semantic retrieval
- Google Gemini LLM for question answering
- Multi-stage state graph with retrieval decision, relevance filtering, grounding verification, and answer usefulness checks
- Iterative retrieval query rewriting and answer revision

## Repository Structure

- `self-rag.py` - Main Python implementation of the self-RAG workflow
- `self_rag_step7.ipynb` - Notebook used for interactive exploration and experimentation
- `documents/` - Expected folder for PDF sources such as company policies, profile, and pricing docs

## Requirements

- Python 3.10+
- `python-dotenv`
- `pydantic`
- `langchain-core`
- `langchain-community`
- `langchain-google-genai`
- `langchain-huggingface`
- `langchain-text-splitters`
- `langgraph`
- `faiss-cpu` or compatible FAISS package

## Setup

1. Clone or open this repository in your environment.
2. Install dependencies, for example:

```powershell
python -m pip install python-dotenv pydantic langchain-core langchain-community langchain-google-genai langchain-huggingface langchain-text-splitters langgraph faiss-cpu
```

3. Create a `.env` file in the repository root with your Gemini API key:

```env
GEMINI_API_KEY=your_google_gemini_api_key
```

4. Place the company PDF documents inside `documents/`.

## How It Works

The pipeline in `self-rag.py` follows these stages:

1. Load PDFs using `PyPDFLoader`
2. Split documents into overlapping chunks with `RecursiveCharacterTextSplitter`
3. Embed chunks via `HuggingFaceEmbeddings`
4. Store chunk embeddings in FAISS and expose a retriever
5. Decide whether the question needs retrieval or can be answered from general knowledge
6. If retrieval is needed, fetch candidate documents and filter them for relevance
7. Generate an answer from the retrieved context
8. Verify answer grounding with a strict support check (`IsSUP`)
9. Validate answer usefulness for the question (`IsUSE`)
10. If the answer is not useful, rewrite the retrieval query and retry

## Running the Project

There is no CLI entrypoint yet. To run the workflow, execute `self-rag.py` or import the module in a notebook.

Example from a notebook:

```python
from self_rag import g, State

initial_state = {
    "question": "What is the refund policy for NexaAI?",
    "retrieval_query": "",
    "rewrite_tries": 0,
    "need_retrieval": False,
    "docs": [],
    "relevant_docs": [],
    "context": "",
    "answer": "",
    "issup": "no_support",
    "evidence": [],
    "retries": 0,
    "isuse": "not_useful",
    "use_reason": "",
}

result = g.run(initial_state)
print(result)
```

> Note: You may need to adapt the final call to match the graph library API if it differs.

## Customization

- Change the Gemini model by updating `model="gemini-3.1-flash-lite"` in `self-rag.py`
- Adjust embedding model in `HuggingFaceEmbeddings`
- Tune chunk size / overlap in `RecursiveCharacterTextSplitter`
- Add more PDF sources to the `documents/` loader list

## Notes

- The project expects local Hugging Face model downloads for `all-MiniLM-L6-v2` via `local_files_only=True`
- The retrieval query rewrite step helps improve vector search for hard questions
- The verification loop is designed to reduce hallucinations by enforcing direct evidence from context

## License

This repository has no license specified.
