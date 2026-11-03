# first-RAG

A minimal Retrieval-Augmented Generation (RAG) example that runs entirely on your machine. It indexes Markdown files, retrieves the most relevant passages for a question, and asks a local LLM to answer using only those passages.

## How it works

1. **Load**: every `docs/*.md` file is split into chunks on blank lines (one paragraph per chunk).
2. **Embed**: each chunk is embedded with `nomic-embed-text` through Ollama and kept in memory as a NumPy array.
3. **Retrieve**: the question is embedded the same way, and the top 3 chunks by cosine similarity are selected.
4. **Generate**: the chunks and the question go to `qwen3:4b-instruct` with an instruction to answer only from the context, or say "I don't know".

There is no vector database. Embeddings are recomputed on every start.

## Requirements

- [Python](https://www.python.org/) 3.14 or newer
- [uv](https://docs.astral.sh/uv/)
- [Ollama](https://ollama.com/) running at `http://localhost:11434`

## Setup

Pull the two models:

```bash
ollama pull nomic-embed-text
ollama pull qwen3:4b-instruct
```

Install the dependencies (`numpy`, `requests`):

```bash
uv sync
```

## Usage

Put your Markdown files in `docs/` (a sample, `handbook.md`, is included), then run:

```bash
uv run rag.py
```

The script prints how many chunks it indexed and then prompts for questions:

```
Indexed 42 chunks

Ask: How many vacation days do I get?
```

Stop it with `Ctrl+C`.

## Configuration

The constants at the top of [rag.py](rag.py) control the setup:

| Constant      | Default                  | Purpose                  |
| ------------- | ------------------------ | ------------------------ |
| `OLLAMA`      | `http://localhost:11434` | Ollama server URL        |
| `EMBED_MODEL` | `nomic-embed-text`       | Model used for embedding |
| `CHAT_MODEL`  | `qwen3:4b-instruct`      | Model used for answers   |

The number of retrieved chunks is the `k` parameter of `retrieve()` (default 3).

## Limitations

- Chunking is by paragraph only, so very long paragraphs aren't split.
- Only top-level `docs/*.md` files are read, not subfolders or other formats.
- The index lives in memory and is rebuilt on each run.
- There is no error handling for Ollama being offline or a model missing.
