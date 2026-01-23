# AI Foundation Addon v1.6.1

The AI Foundation addon provides on-device AI inference capabilities for mimOE. It includes a Model Registry for managing AI models and an OpenAI-compatible Inference API.

## What's Included

This addon bundles two mims:

| mim | Description | API Base URL |
|-----|-------------|--------------|
| `mmodelstore-v1` | Model Registry - upload, download, manage AI models | `/mimik-ai/store/v1` |
| `milm-v1` | Inference API - OpenAI-compatible chat/completions | `/mimik-ai/openai/v1` |

## Supported Model Formats

| Format | Kind | Use Case |
|--------|------|----------|
| GGUF | `llm` | Large Language Models (chat, text generation) |
| GGUF | `embed` | Text embeddings (semantic search, RAG) |
| GGUF + mmproj | `vlm` | Vision-Language Models (image understanding) |
| ONNX | `onnx` | Predictive models (classification, regression) |

## Installation

### Via Install Script (Recommended)

The AI Foundation addon is automatically installed when using the mimOE install script:

**macOS / Linux:**
```bash
curl -L https://raw.githubusercontent.com/mimik-mimOE/mimOE-SE/main/install-mimOE-ai.sh | bash
```

**Windows (Command Prompt):**
```cmd
curl -L https://raw.githubusercontent.com/mimik-mimOE/mimOE-SE/main/install-mimOE-ai.bat -o install.bat && install.bat
```

### Manual Installation

1. Download `ai-foundation-v1.6.1.addon` from [GitHub Releases](https://github.com/mimik-mimOE/mimOE-addon-ai-foundation/releases/tag/v1.6.1)

2. Place in the `addon/` folder of your mimOE installation:
   ```
   mimOE-SE/
   └── addon/
       └── ai-foundation.addon
   ```

3. Create configuration file `addon/ai-foundation.ini` (see Configuration below)

4. Restart mimOE - the addon will be deployed automatically

## Configuration

Create `addon/ai-foundation.ini` to customize the addon:

```ini
[milm-v1]
# API key for Inference API authentication
API_KEY=1234

# Execution timeout for AI inference (default: 30s)
# Increase for larger models or longer generations
MCM.MAX_EXECUTION_TIME_SEC=180

# Model Registry API key (if mmodelstore API_KEY is changed)
# Must match the API_KEY in [mmodelstore-v1] section
# MMODELSTORE_API_KEY=1234

# [mmodelstore-v1]
# Model Registry API key
# IMPORTANT: If you change this, you must also set MMODELSTORE_API_KEY
# in the [milm-v1] section above to the same value
# API_KEY=1234
```

### Configuration Notes

- The `[milm-v1]` section configures the Inference API
- The `[mmodelstore-v1]` section configures the Model Registry API
- If you change the Model Registry `API_KEY`, you must also set `MMODELSTORE_API_KEY` in the milm section so they can communicate
- `MCM.MAX_EXECUTION_TIME_SEC` should be increased for AI workloads (default mimOE timeout is 30s)

## API Endpoints

### Model Registry API

Base URL: `http://localhost:8083/mimik-ai/store/v1`

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/models` | List all models |
| POST | `/models` | Create model metadata |
| GET | `/models/{id}` | Get model info |
| DELETE | `/models/{id}` | Delete a model |
| POST | `/models/{id}/upload` | Upload model file (multipart) |
| POST | `/models/{id}/download` | Download model from URL (SSE progress) |

### Inference API

Base URL: `http://localhost:8083/mimik-ai/openai/v1`

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/models` | List loaded models |
| POST | `/models` | Load model into memory |
| DELETE | `/models` | Unload model from memory |
| POST | `/chat/completions` | Generate chat response |
| POST | `/embeddings` | Generate text embeddings |

## Quick Start

### 1. Provision a Model

```bash
# Create model metadata
curl -X POST "http://localhost:8083/mimik-ai/store/v1/models" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 1234" \
  -d '{
    "id": "smollm2-360m",
    "version": "1.0.0",
    "kind": "llm"
  }'

# Download from Hugging Face
curl -X POST "http://localhost:8083/mimik-ai/store/v1/models/smollm2-360m/download" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 1234" \
  -d '{
    "url": "https://huggingface.co/lmstudio-community/SmolLM2-360M-Instruct-GGUF/resolve/main/SmolLM2-360M-Instruct-Q8_0.gguf"
  }'
```

### 2. Run Inference

```bash
curl -X POST "http://localhost:8083/mimik-ai/openai/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 1234" \
  -d '{
    "model": "smollm2-360m",
    "messages": [{"role": "user", "content": "Write a haiku about AI"}]
  }'
```

### 3. Generate Embeddings

```bash
curl -X POST "http://localhost:8083/mimik-ai/openai/v1/embeddings" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 1234" \
  -d '{
    "model": "your-embed-model",
    "input": "Hello, world!"
  }'
```

## Addon Structure

```
ai-foundation.addon (tar file)
├── manifest.json           # Addon metadata and mim configurations
├── mmodelstore-v1.tar      # Model Registry mim image
└── milm-v1.tar             # Inference API mim image
```

### manifest.json

```json
{
  "name": "ai-foundation",
  "version": "1.6.1",
  "id": "mimik.ai",
  "mims": [
    {
      "name": "mmodelstore-v1",
      "image": {
        "name": "mmodelstore-v1",
        "file": "mmodelstore-v1.tar",
        "indexFileChecksum": "sha256:..."
      },
      "env": {
        "MCM.BASE_API_PATH": "/store/v1"
      }
    },
    {
      "name": "milm-v1",
      "image": {
        "name": "milm-v1",
        "file": "milm-v1.tar",
        "indexFileChecksum": "sha256:..."
      },
      "env": {
        "MCM.BASE_API_PATH": "/openai/v1",
        "MMODELSTORE_URL": "http://localhost:8083/mimik-ai/store/v1"
      }
    }
  ]
}
```

## Release Notes

### v1.6.1

- Improved SSE progress reporting for model downloads
- Better error handling for invalid model formats
- Support for `.ini` configuration files
- Performance optimizations for inference

## Documentation

- **[AI Foundation Overview](https://developer.mimik.com/docs/ai-foundation)** - Getting started guide
- **[Model Registry API](https://developer.mimik.com/docs/api/model-registry)** - Full API reference
- **[Inference API](https://developer.mimik.com/docs/api/inference)** - OpenAI-compatible API reference
- **[MCM Environment Variables](https://developer.mimik.com/docs/api/mcm#environment-variables)** - Configuration options
- **[Finding Models](https://developer.mimik.com/docs/ai-foundation/examples/finding-models)** - How to find compatible models

## Requirements

- mimOE v3.18.0 or later
- Sufficient RAM for your chosen model (varies by model size)
- Sufficient disk space for model storage

## License

The AI Foundation addon is included with mimOE Standard Edition. See [LICENSE](LICENSE) for details.

## Support

- **Documentation**: https://developer.mimik.com
- **Issues**: https://github.com/mimik-mimOE/mimOE-addon-ai-foundation/issues
