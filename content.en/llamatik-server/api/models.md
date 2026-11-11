---
title: "Models Status"
weight: 5
---

# Models Status

### `GET /v1/models`

Returns which models are currently loaded in each inference bridge, plus the capabilities of the generation model. Use it to discover server state (for example, after startup loading) without calling each bridge separately.

Only model **filenames** are returned — full server paths are never exposed.

```bash
curl -s http://localhost:8080/v1/models | jq
```

```json
{
  "generation": {
    "loaded": true,
    "model": "llama-3.2-3b-instruct-q8_0.gguf",
    "capabilities": { "finetuneType": "instruct", "hasChatTemplate": true }
  },
  "embedding": { "loaded": false, "model": null },
  "whisper": { "loaded": true, "model": "ggml-base-q8_0.bin" },
  "stableDiffusion": {
    "loaded": false, "model": null,
    "vae": null, "t5xxl": null, "clipVision": null, "supportsVideo": null
  },
  "multimodal": { "loaded": false, "model": null, "mmproj": null },
  "mtp": { "loaded": false, "model": null }
}
```

| Field | Description |
|---|---|
| `generation.capabilities` | Present only when a generation model is loaded. `finetuneType` is the GGUF `general.finetune` value (`instruct`, `chat`, `base` or `null`). `hasChatTemplate` is `true` when `POST /v1/generation/model/applyChatTemplate` can render messages for this model. |
| `stableDiffusion.supportsVideo` | `true` when the loaded model supports `txt2video`; `null` when nothing is loaded. |
| `stableDiffusion.vae` / `t5xxl` / `clipVision` | Filenames of companion files, when loaded. |

The state is updated by the `/init`, `/shutdown` and `/release` endpoints as well as by [startup loading](../../configuration/#startup-model-loading).
