# Project Diagnosis — AI Screenshot Detector

**Date:** 2026-04-30
**Scope:** Honest assessment of current state, before any rework.

---

## TL;DR

This is a **well-engineered course project** that ships end-to-end (training → calibration → deployment → guardrails → paper). The student-authored design choices are all defensible. But the project has **two structural ceilings** that block the leap from "course submission" to "commercial-grade screenshot detector":

1. **Training data = Stable Diffusion v1.4 only.** Instagram is dominated by Midjourney, Sora, Flux, DALL-E 3, Imagen, and Ideogram — generators with very different visual signatures. The model was never shown those.
2. **Architecture is dated.** EfficientNet-B0 with a frozen ImageNet backbone is a 2019-era choice. State-of-the-art AI-image detectors in 2025–26 use frozen CLIP/DINOv2 features + a small head, or fine-tuned ConvNeXt/ViT.

You already documented #1 honestly in `QA_PREP.md` Q8 ("the fix isn't a better augment; it's a more diverse training set"). That is exactly what we should do.

---

## What's strong (keep these)

| Area | What's good |
|------|------|
| **Engineering hygiene** | Clean repo layout, smoke tests, ablation framework, calibration with temperature scaling, deployment guardrails (file size, dim bounds, uncertainty band). Beginner students rarely have this. |
| **`simulate_screenshot` augmentation** | The core insight is real: training on simulated platform-degraded images is the right move, and your ablation shows it closes the gap. |
| **Calibration (T=0.563, ECE 10% → 1.9%)** | Most projects skip this. Yours doesn't. |
| **Deployment** | Hugging Face Space with proper guardrails, model repo separate from Space code. Solid pattern. |
| **Honest limitations** | `QA_PREP.md` reads like a senior engineer wrote it. Self-aware about IG weakness, adversarial bypass, label noise. |

**Do not throw any of this away.** It's the foundation for the next version.

---

## What's weak (these are our fix targets)

### 1. Training-data generator coverage is too narrow
**Symptom:** CIFAKE = Stable Diffusion v1.4 only. Real-world IG eval shows the model fails on different generators.
**Fix direction:** Add GenImage, DiffusionForensics, and ArtiFact to training. These cover Midjourney, ADM, GLIDE, BigGAN, VQDM, etc. We may also need to scrape post-2024 generations (Sora, Flux, Ideogram) — likely from public datasets on Hugging Face.

### 2. Frozen backbone leaves performance on the table
**Symptom:** Only the final 2-class head trains (~6,500 params). The backbone never sees a single AI image.
**Fix direction:** Either (a) unfreeze last 1–2 backbone blocks with small LR, or (b) ditch EfficientNet entirely and use frozen CLIP / DINOv2 features (current SOTA approach for detection — see UniversalFakeDetect).

### 3. No frequency-domain features
**Symptom:** Diffusion models leave characteristic artifacts in the **frequency domain** (FFT/DCT spectrum) that survive screenshot compression better than pixel-space features. Your model never looks at them.
**Fix direction:** Add a parallel FFT branch that consumes the magnitude spectrum, fuses with the spatial branch. This is a known winner for diffusion detection.

### 4. Eval set is too small to make commercial claims
**Symptom:** 49 hand-labeled Instagram images is fine for a course project, not for a product. We have no way to know if a 1% accuracy improvement is real or noise.
**Fix direction:** Build a much larger held-out test set: at minimum 500/500 real-vs-fake screenshots, drawn from multiple platforms (IG, X, TikTok, Reddit) and multiple generators. Some will need to be auto-collected.

### 5. No mechanism to handle generator drift over time
**Symptom:** A model trained today on Sora outputs will be obsolete in 6 months when Sora 2 ships. There's no plan for continuous data collection or re-training.
**Fix direction:** This becomes important for commercial deployment. For now, log it as future work.

### 6. Paper-mode metrics, not production-mode metrics
**Symptom:** You report accuracy. Production needs precision/recall per class, false-positive rate at fixed thresholds, latency p95, AUROC. Especially: "what's the FPR when we hit 90% recall on AI?"
**Fix direction:** Expand `eval.py` to emit a full classification report.

---

## What's missing entirely

| Missing | Why it matters |
|------|------|
| **Larger eval dataset** | Without it, no commercial-grade claim is defensible. |
| **Frequency-domain analysis** | Single biggest known accuracy lever for diffusion detection. |
| **Multi-generator training data** | Single biggest known generalization lever. |
| **Continuous monitoring story** | Required for any real deployment. |
| **README "Limitations" section** | The QA_PREP doc has these; the README should too — it's what recruiters read. |

---

## Roadmap proposal (what we'd do next, ordered)


**Phase 1 — Foundations:**
- Python skills refresh
- CNNs from scratch
- Walk through `train.py` line by line

**Phase 2 — Eval-first upgrade:**
- Build a larger, multi-generator, multi-platform held-out test set
- Re-evaluate the existing model on it → establish a real baseline
- Why first: every future change has to beat this baseline. No baseline = no progress signal.

**Phase 3 — Architecture upgrade:**
- Swap EfficientNet-B0 for a frozen CLIP- or DINOv2-based feature extractor + small head
- Add an FFT/frequency-domain branch
- Re-run ablation against the new baseline

**Phase 4 — Data upgrade:**
- Add GenImage / DiffusionForensics / ArtiFact training data
- Optionally add post-2024 generator outputs

**Phase 5 — Production hardening:**
- Full classification report (P/R/F1, AUROC, FPR @ fixed recall, latency)
- Better calibration (per-class temperature, or beta calibration)
- Drift-detection plan
- Updated paper / blog post / portfolio README

**Phase 6 — Commercial readiness:**
- API endpoint (FastAPI + Docker)
- Stronger UX
- Pricing / market positioning
