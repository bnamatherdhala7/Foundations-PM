# Lecture 09 — Multimodal Models
## Stanford CME295 | PM Reference Notes

**One-line concept:** Multimodal models process and generate multiple types of data — text, images, audio, video, code — by learning a shared representation space. Each modality adds capability but also multiplies evaluation complexity and data pipeline cost.

**Why it matters for PMs:** Splunk's move into AI means connecting structured logs, unstructured text, visual dashboards, and alert signals. Understanding multimodal architecture tells you what's possible, what requires new data infrastructure, and what the quality tradeoffs are across modalities.

---

## 1. What Is a Multimodal Model?

A unimodal model processes one type of input. A multimodal model processes two or more.

**Current frontier multimodal capabilities:**

| Input | Output | Example |
|---|---|---|
| Text → Text | Language model (GPT-4, Claude) | Standard LLM |
| Image + Text → Text | Vision-Language Model | "Describe this chart" |
| Text → Image | Image generation | Firefly, DALL-E, Midjourney |
| Audio → Text | Speech recognition (ASR) | Whisper |
| Text → Audio | Text-to-speech (TTS) | ElevenLabs, Google TTS |
| Video + Text → Text | Video understanding | Gemini 1.5, GPT-4V |
| Text + Code → Text | Code models | GPT-4, Claude, Codex |

**The frontier trend:** Models are expanding to understand and generate across all modalities simultaneously (Gemini, GPT-4o, Claude 3.5 with vision).

---

## 2. How Vision-Language Models Work

The dominant architecture for understanding images + generating text:

### 2.1 CLIP — The Foundation

**CLIP (Contrastive Language-Image Pre-training, OpenAI 2021):**
- Trained on 400M (image, caption) pairs from the web
- Uses two encoders: one for images, one for text
- Trained with contrastive loss: pull matching image-text pairs together in embedding space; push non-matching pairs apart

```
Image: [photo of a dog] → Image Encoder → Image Vector
Text: "a dog playing in the park" → Text Encoder → Text Vector

If image matches caption: vectors should be similar (high cosine similarity)
If image doesn't match: vectors should be distant
```

**What CLIP enables:**
- Image-text similarity scoring (used in CLIP Score eval)
- Zero-shot image classification: compare image to text descriptions of classes
- Semantic image search: embed images and queries in the same space

**Limitation:** CLIP understands meaning but doesn't generate. It's an encoder, not a generator.

### 2.2 Connecting Vision to Language — LLaVA Pattern

Modern vision-language models typically:
1. Use a pretrained vision encoder (CLIP or similar) to embed the image
2. Project the image embeddings into the LLM's token space (a small linear projection layer)
3. Pass the projected image tokens alongside text tokens through the LLM decoder

```
Image → Vision Encoder (CLIP) → Image Embeddings
                                        ↓
                              Projection Layer (learnable)
                                        ↓
Text Tokens + Image Tokens → LLM Decoder → Generated Text
```

**What's trained:**
- The projection layer is trained (small, fast)
- The LLM may be fine-tuned or kept frozen
- The vision encoder is usually frozen (CLIP weights are high quality and stable)

**Examples:** LLaVA, InstructBLIP, GPT-4V, Claude 3 Vision, Gemini

### 2.3 Interleaved Image-Text Models

More advanced: models that can reason over sequences of alternating text and images.

```
Input: [Image 1] "These are system logs from 9 AM." [Image 2] "These are from 9:05 AM. What changed?"
Output: "The CPU utilization spike visible in Image 2 was absent in Image 1..."
```

This requires the model to attend across both modalities simultaneously — a harder training problem.

---

## 3. Image Generation Architectures

The dominant approaches for text → image:

### 3.1 Diffusion Models (Dominant)

**How they work:**
1. Forward process: gradually add Gaussian noise to a real image until it becomes pure noise
2. Reverse process: train a neural network (the denoising model) to predict and remove noise step by step
3. Inference: start from pure noise; iteratively denoise conditioned on a text prompt

**Why diffusion works:** The denoising network learns the structure of the image distribution. Given a noisy image and a text condition, it predicts the direction toward a clean, prompt-matching image.

**Key models:** Stable Diffusion, DALL-E 2/3, Midjourney, Adobe Firefly Image Model, Imagen

**Advantages:** High visual quality, photorealism, controllability
**Disadvantages:** Slow inference (10–50+ denoising steps), hard to precisely control fine details

**Flow Matching:** A more efficient variant of diffusion that learns straighter denoising paths, enabling quality images in fewer steps (1–4 steps vs 20–50). Flux, Stable Diffusion 3, and modern Firefly versions use flow matching.

### 3.2 Autoregressive Image Generation

**How they work:** Tokenize images into discrete tokens (using a VQ-VAE), then train an autoregressive transformer to generate image tokens — same architecture as a language model.

**Key models:** DALL-E 1, Parti, some GPT-4o components
**Advantages:** Can naturally interleave image and text generation in the same sequence
**Disadvantages:** Less visually sharp than diffusion; slower at generating high-resolution images

### 3.3 GANs (Generative Adversarial Networks — Historical)

Pre-diffusion dominant approach. Generator + Discriminator trained adversarially. Largely superseded by diffusion for image generation but still used in some video and domain-specific applications.

---

## 4. Audio Modality

### Speech Recognition (ASR — Automatic Speech Recognition)
**Architecture:** Encoder-decoder transformer trained on (audio, transcript) pairs.
**Dominant model:** OpenAI Whisper — trained on 680K hours of multilingual audio from the web. Strong multilingual performance; handles noise well.
**Use in security/ops:** Converting voice alerts, incident call recordings, or verbal runbooks to text for LLM processing.

### Text-to-Speech (TTS)
**Modern approach:** Diffusion or autoregressive models conditioned on text; generates mel spectrograms → waveforms.
**Quality bar:** Indistinguishable from human speech for some systems (ElevenLabs, Google WaveNet).

---

## 5. Video Understanding

**The challenge:** Video = sequences of frames + audio. A 1-minute video at 30fps = 1,800 frames. Processing all frames through a vision encoder is prohibitively expensive.

**Current approaches:**
- **Frame sampling:** Select keyframes (every N frames or scene-change-triggered)
- **Temporal encoding:** Use video-specific architectures that model motion across frames
- **Long-context models:** Gemini 1.5's 1M token context allows more frames but at high cost

**Frontier video models:** Sora (OpenAI), Veo (Google), Runway Gen-3, Pika

**For security/ops use case:** Video models could interpret screen recordings, operational dashboards, or network visualization captures — emerging but not yet mature for production.

---

## 6. Multimodal for Security/Observability (Splunk Context)

The most relevant multimodal applications for a Splunk Foundations PM:

| Use Case | Modalities | What It Enables |
|---|---|---|
| **Log + screenshot analysis** | Text (logs) + Image (dashboard screenshot) | "Explain what this dashboard is showing and correlate with these log events" |
| **Anomaly visualization** | Text (alert) + Image (time series chart) | "Is this spike in the chart consistent with the alert I'm seeing?" |
| **Document + log correlation** | Text (runbook PDF) + Text (logs) | Long-context multi-document reasoning over operational data |
| **Alert audio transcription** | Audio → Text | Converting verbal incident calls to structured summaries |
| **Code + log analysis** | Code + Text | "This is the deployment code; these are the post-deploy error logs. What failed?" |

---

## 7. Data and Eval Complexity by Modality

| Modality | Training Data Challenge | Key Eval Metric | Human Eval Requirement |
|---|---|---|---|
| Text | Large but increasingly scarce at high quality | MMLU, task-specific benchmarks | Required for open-ended generation |
| Image (understanding) | Image-text pairs; alignment quality | VQA accuracy, captioning BLEU | Spot-check for grounding accuracy |
| Image (generation) | Licensed, diverse, high-quality images | FID, CLIP Score, Human ELO | Required for aesthetic quality |
| Audio | Speech recordings with transcripts | WER (Word Error Rate) | Required for naturalness |
| Video | Video + captions; temporal coherence labels | FVD, temporal consistency | Required for motion quality |
| Code | Code + tests; execution-verified | Pass@k, execution accuracy | Optional if tests pass |

---

## Key Terms

| Term | Definition |
|---|---|
| **Multimodal model** | Model that processes or generates multiple types of data (text, image, audio, video) |
| **CLIP** | Contrastive Language-Image Pre-training — joint text-image embedding model |
| **Contrastive learning** | Training by pulling similar pairs together and pushing dissimilar pairs apart in embedding space |
| **Vision encoder** | Neural network that converts an image into an embedding vector |
| **Projection layer** | Small learned linear transformation connecting vision encoder outputs to LLM input space |
| **Diffusion model** | Generative model that learns to reverse a noise-adding process; current standard for image generation |
| **Flow matching** | Efficient variant of diffusion learning straighter denoising paths; fewer inference steps |
| **VQ-VAE** | Vector Quantized Variational Autoencoder — encodes images as discrete tokens for autoregressive generation |
| **GAN** | Generative Adversarial Network — generator + discriminator; pre-diffusion image generation approach |
| **ASR** | Automatic Speech Recognition — converting speech to text |
| **TTS** | Text-to-Speech — converting text to audio |
| **WER** | Word Error Rate — primary metric for ASR quality |
| **VQA** | Visual Question Answering — benchmark for image understanding |
| **FVD** | Fréchet Video Distance — video quality metric equivalent to FID |
| **Interleaved** | Model that can process alternating image and text in the same input sequence |

---

## Product Questions This Unlocks

1. "Can we build a feature that lets analysts paste a dashboard screenshot and get log correlation?" — Yes, via vision-language model; key question is latency and accuracy on your specific chart types.
2. "How much more data do we need for our video understanding feature?" — Video requires temporal coherence labels on top of content labels; expect 3–5× the labeling cost of images.
3. "Why is the model worse at reading tables in images vs. text?" — OCR-style tasks require specific training data; general vision-language models are weak on structured data visualization.
4. "Can we use the same eval infrastructure across modalities?" — No. Each modality has different metrics, different human eval rubrics, and different quality dimensions. Budget for modality-specific eval.
5. "What's the latency cost of adding vision to our text model?" — Vision encoder adds ~50–200ms per image; the main cost is context length (image tokens can be 256–2048 tokens per image).

---

*Lecture 9 of 12 — Stanford CME295 LLM Foundations | PM Reference*
