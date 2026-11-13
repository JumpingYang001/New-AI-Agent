# New-AI-Agent AIVoice - Multimodal AI Content, Audio & Video Orchestration
An AI Agent to generate content, audio and video

AIVoice connects local language models, speech synthesis, image generation, and talking-head engines to turn a topic into a narrated, subtitled discussion video.

The project explores how to make multi-model AI workflows practical: structured intermediate artifacts, editable transcripts, isolated inference environments, resumable execution, and explicit quality checks.

> **About this repository:** This is a documentation-only project showcase. It does not contain implementation code, model weights, or generated media. The architecture and capabilities below describe the separate AIVoice implementation workspace; cloning this repository alone does not provide a runnable application.

## Project Overview

The implementation provides two complementary workflows:

- **Automated talking-video generation:** A standalone runner generates speaker portraits, dialogue, per-turn speech, backgrounds, talking-head clips, subtitles, and a final MP4 without review pauses.
- **Reviewable short-drama orchestration:** AIVideoFactory provides an installable `avf` CLI with persistent project state, script/storyboard/image approval gates, structured manifests, rendering, and media quality reports.

Supporting tools handle standalone text generation, speech generation, transcription, image generation, and simple image-based video animation.

## Core Capabilities

| Area | Implemented capabilities |
| --- | --- |
| Content and dialogue | Local Qwen-based English and Chinese generation; speaker-labelled conversations with 1-6 participants; configurable turn counts and generation settings |
| Dialogue quality | Structured-output validation, minimum-length checks, near-duplicate rejection, and per-turn regeneration when batch output is malformed |
| Speech synthesis | Microsoft Edge TTS with distinct voices, stable speaker-to-voice assignments, configurable speech rate, and optional seeded voice selection |
| Audio artifacts | Combined MP3, individual turn recordings, and segment manifests recording speaker, text, voice, file, and measured duration |
| Visual assets | Stable Diffusion backgrounds and generated speaker portraits; support for supplied background images and reusable portraits |
| Talking heads | SadTalker and MuseTalk integrations through isolated subprocess environments; only the active speaker is animated |
| Video composition | Multi-speaker layouts, subtitles, background labels, synchronized audio, and final MP4 output |
| Transcription | Separate faster-whisper and SenseVoice utilities, including Simplified Chinese normalization |
| Workflow control | Standalone automatic execution, individually runnable transcript/audio/video stages, and factory approval gates |
| Reliability | Preflight checks, input-sensitive artifact reuse, downstream invalidation, persistent manifests, and factory JSONL run logs |

## Architecture

### Automated Talking-Video Workflow

```text
Topic + language + speaker settings
                  |
                  v
       Generate speaker portraits
                  |
                  v
       Generate Qwen dialogue
                  |
                  v
       Synthesize per-turn speech
                  |
                  v
       Generate or load background
                  |
                  v
       Render talking-head clips
          SadTalker / MuseTalk
                  |
                  v
       Compose layout + subtitles
                  |
                  v
                MP4
```

Portrait generation completes before later model stages. The dialogue model is released before background generation, avoiding simultaneous residency of those models in GPU memory. Rendering and encoding remain CPU-based in the talking-video workflow.

### Reviewable Factory Workflow

```text
Project configuration
    -> Script [review]
    -> Shot plan [review]
    -> Images [review]
    -> Speech
    -> Motion sources
    -> Lip sync
    -> Timeline / editing
    -> Subtitles
    -> Rendering
    -> Quality report
```

The factory uses versioned JSON Schemas and semantic cross-reference validation for project, story, shot-plan, and timeline documents. Preview and production adapter profiles share downstream manifests, so editing, validation, caching, and quality checks use the same orchestration structure.

## Engineering Highlights

### Artifact-Based Orchestration

Stages exchange explicit files and manifests rather than relying solely on in-memory state. Transcripts can be reviewed and edited independently, while audio manifests provide measured turn durations for synchronization.

### Resumable, Input-Aware Execution

The standalone runner reuses complete artifacts while their relevant inputs remain unchanged. Editing a transcript invalidates dependent audio and video; changing final composition settings can reuse expensive talking-head clips.

The factory separately tracks stage state and input/output hashes, skips unchanged work, invalidates downstream stages, and records execution events.

### Model and Environment Isolation

SadTalker and MuseTalk run with separately configured Python interpreters to accommodate different Python, CUDA, and library requirements. Preflight checks validate paths, imports, model files, and device availability before expensive generation.

### Review and Quality Controls

The factory supports approval gates for creative stages, FFmpeg H.264/AAC rendering, embedded subtitles, FFprobe-based media inspection, and machine-readable quality reports. Preview artifacts and placeholder adapters are explicitly identified rather than presented as completed production assets.

### Focused Automated Tests

The implementation includes tests for schema validation, stage state, review gates, resume behavior, rendering and quality checks, adapter integrations, portraits, talking-head handling, and standalone execution. These tests are separate from this documentation-only repository; their presence does not constitute a benchmark of model quality.

## Implementation Components

These are component names in the separate implementation workspace, not directories distributed in this repository.

| Component | Responsibility |
| --- | --- |
| `Standalone` | Automatic, resumable topic-to-talking-video runner |
| `Multi-AI-Talking` | Qwen dialogue generation and multi-speaker Edge TTS |
| `Multi-AI-Talking-Video` | Video orchestration, layouts, subtitles, and engine integration |
| `AIVideoFactory` | Staged CLI orchestration, review gates, schemas, state, and quality checks |
| `AI-Content` | Standalone local text generation |
| `AI-Picture` | Text-to-image generation |
| `AI-Video` | Generated still-image animation using pan and zoom |
| `faster-whisper` / `SenseVoice` | Separate speech-recognition utilities |
| SadTalker / MuseTalk integrations | Third-party talking-head inference backends |

## Technology Stack

- **Language and interfaces:** Python, CLI tooling, subprocess orchestration
- **Language models:** Qwen, Hugging Face Transformers, PyTorch
- **Speech:** Microsoft Edge TTS, faster-whisper, SenseVoice/FunASR
- **Images:** Stable Diffusion, Hugging Face Diffusers
- **Talking heads:** SadTalker, MuseTalk
- **Media:** FFmpeg, FFprobe, MP3/WAV, MP4, SRT
- **Workflow engineering:** JSON Schema, persistent state, input hashing, JSONL logs
- **Validation:** pytest
- **Execution:** Windows, CPU inference where supported, NVIDIA CUDA inference

## Current Status and Boundaries

- The standalone talking-video workflow supports SadTalker or MuseTalk. Output quality depends on portraits, speech, model behavior, and hardware.
- AIVideoFactory is an orchestration MVP. Its production adapter profile includes Qwen dialogue, Edge TTS, static shot sources, and MuseTalk, but its image stage still uses placeholders. Selecting the production profile does not make every stage production-ready.
- Basic `AI-Video` output animates a generated still image with pan and zoom; it is not temporal text-to-video diffusion.
- Transcription tools are separate utilities, not an automatic speech-recognition feedback loop in the generation pipeline.
- Edge TTS requires internet access. Local model inference can run after model downloads, but the complete default workflow is not fully offline.
- MuseTalk requires CUDA. Some other stages offer CPU execution, with substantially slower inference possible.
- Additional motion-model and multi-GPU fleet ideas remain plans, not implemented distributed scheduling.
- The project demonstrates application-level orchestration of existing models; it does not claim original foundation-model training, enterprise deployment, or measured commercial impact.

## Example Use Cases

- English or Chinese multi-speaker discussions and AI-hosted podcasts
- Educational explainers with distinct synthetic presenters
- Editable, staged content-production experiments
- Rapid prototyping of narrated and subtitled talking-head videos

## Responsible Use and Attribution

Use only portraits, voices, and source media you have permission to process. Disclose synthetic content where appropriate, and do not impersonate people without authorization.

Qwen, Stable Diffusion, Edge TTS, SadTalker, MuseTalk, faster-whisper, and SenseVoice are external models, tools, or services. Their respective licenses, usage terms, and attribution requirements apply independently. This showcase does not redistribute them.
