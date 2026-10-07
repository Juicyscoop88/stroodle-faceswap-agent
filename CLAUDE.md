# stroodle-faceswap-agent

## What This Is

A Stroodle-registered A2A agent that hosts the FaceSwap MiniMax H3 Ref2VA model as a service. Takes a reference face image + source video, returns a face-swapped video. Discoverable and callable through Stroodle's agent registry.

## Model

- **Base**: MiniMax H3 Ref2VA (image-to-video)
- **LoRA**: [UntMods/FaceSwap_MiniMaxH3_REF2VA](https://huggingface.co/UntMods/FaceSwap_MiniMaxH3_REF2VA) (Apache 2.0)
- **Trigger word**: "Faceswap"
- **Strength**: 1.0
- **Quantization**: Use int8 variant for inference (e.g. PulpCut/MiniMax-H3-Ref2VA-Turbo-INT8-ConvRot, 23k downloads)

## Architecture

```
Client (via Stroodle MCP)
  → call_agent("faceswap-agent", {face_image, source_video, prompt})
  → Stroodle registry routes to this agent
  → A2A endpoint receives task
  → Inference server runs MiniMax H3 + FaceSwap LoRA on GPU
  → Returns video result via complete_task
  → Stroodle logs latency, success/fail → feeds StroodleScore
```

## Infra

- AWS g5.xlarge (single A10G GPU, 24GB VRAM) — enough for int8 inference
- Docker container with model weights baked in (or pulled from HF on startup)
- A2A endpoint via stroodle runtime (`npx stroodle init`)
- Registered on api.stroodle.ai

## API

Input:
- `face_image`: base64 or URL of the reference face
- `source_video`: base64 or URL of the source video (MH3 format)
- `prompt`: text prompt (must include "Faceswap" trigger)

Output:
- `video`: base64-encoded result video
- `duration_seconds`: generation time

## Commands

- `docker build -t faceswap-agent .` — build container
- `docker run --gpus all -p 8080:8080 faceswap-agent` — run locally
- `npx stroodle init` — register as A2A agent on Stroodle

## TODO

- [ ] Inference server (FastAPI + diffusers pipeline)
- [ ] Dockerfile with model download
- [ ] A2A endpoint via stroodle runtime
- [ ] Register on Stroodle registry
- [ ] Deploy to AWS g5.xlarge
- [ ] Test end-to-end via `call_agent` through Stroodle MCP
