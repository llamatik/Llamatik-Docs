---
title: "Embeddings"
weight: 20
---

# Embeddings — `/v1/embeddings`

The embeddings endpoints wrap `LlamaBridge` embedding inference. Use this for RAG pipelines, semantic search, and similarity ranking.

## Initialize

### `POST /v1/embeddings/init`

Load an embedding model. Must be called before `/embed`.

**Request:**
```json
{ "modelPath": "/models/nomic-embed-text-v1.5.Q8_0.gguf" }
```

**Response:**
```json
{ "ok": true }
```

Returns `400` on failure.

## Generate an embedding

### `POST /v1/embeddings/embed`

Compute an embedding vector for the given input string.

**Request:**
```json
{ "input": "Kotlin Multiplatform" }
```

**Response:**
```json
{ "embedding": [0.12, -0.04, 0.87, "..."] }
```

The `embedding` array length matches the model's output dimension. Typical values:
- `nomic-embed-text-v1.5` → 768 dimensions
- `mxbai-embed-large` → 1024 dimensions

## Notes

- One model at a time. Call `init` again to swap models.
- Embedding models are different from generation models — use a model specifically trained for embeddings.
- The server does not expose a `/release` endpoint for embeddings; the embedding model shares the `LlamaBridge` lifecycle with the generation model.
