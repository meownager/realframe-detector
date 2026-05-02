
# realframe-detector

**Author:** Syeda Monowara (smonowar@purdue.edu)

> A computer vision classifier for AI-generated images in social media screenshots — designed for the JPEG compression and resizing that come with real-world distribution.

## What this is

AI-generated image detection is a fast-moving research problem, and what makes it especially interesting to me is the gap between **clean benchmark conditions** and **what people actually see on social platforms**. By the time an AI image reaches you on Instagram or Reddit, it's been re-encoded, compressed, and resized — and detectors built on pristine training data don't always survive that.

realframe-detector is my exploration of that gap: a screenshot-robust classifier built to work in the messy conditions where real-world distribution actually happens. It started as a course project at Purdue and is now a portfolio project I'm building out toward something deployable.

## Status

- **v1** — deployed and live ([Hugging Face Space](https://huggingface.co/spaces/meownager/ai-image-screenshot-detector)). EfficientNet-B0 fine-tuned on CIFAKE with a screenshot-simulating augmentation. Strong baseline; trained on Stable Diffusion v1.4 outputs, so generalization to newer generators (Midjourney, Flux, Sora) is limited — that's where v2 picks up.
- **v2** — in active development. See [`DIAGNOSIS.md`](./DIAGNOSIS.md) for the v1 → v2 plan: foundation model backbone (DINOv2 or CLIP), frequency-domain branch, multi-generator training data.

## Live demo

Try it at https://huggingface.co/spaces/meownager/ai-image-screenshot-detector. Upload any image and it returns Real / AI-Generated / Uncertain.

## How it works (v1)

```
Screenshot
    │
    ▼
Resize 224×224  ──►  EfficientNet-B0 (ImageNet pretrained, frozen)  ──►  2-class head  ──►  Real or AI
                                                                              │
                                                                              ▼
                                                                  Temperature-scaled softmax
                                                                              │
                                                                              ▼
                                                                Real / AI / Uncertain (< 0.6 confidence)
```

The key v1 idea: at training time, every image is passed through `simulate_screenshot` — a randomized chain of JPEG compression, downscale + upscale, and Gaussian blur — so the model learns features that survive social-media re-encoding rather than relying on clean pixel-level artifacts.

## Repo layout

```
src/                    Core code
  screenshot_augment.py   simulate_screenshot augmentation
  train.py                model + training loop
  ablation.py             4-config ablation study
  data.py                 CIFAKE dataset + stratified splits
  eval.py                 clean / screenshot / real-world eval
  calibration.py          temperature scaling
  app.py                  Gradio interface
hf-space/               Self-contained Hugging Face Space app
notebooks/              Colab training notebook
data/
  ig-data/                49 hand-collected Instagram screenshots (eval only)
tests/                  Smoke tests
DIAGNOSIS.md            v1 → v2 analysis and roadmap
CLAUDE.md               Working rules for AI assistants on this repo
```

## Quick start

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Train:
```bash
python -m src.ablation --data-root ./data/cifake --out-root ./runs/ablation --epochs 3
```

Evaluate:
```bash
python -m src.eval realworld --real-dir ./data/ig-data/real --fake-dir ./data/ig-data/fake --checkpoint ./runs/ablation/screenshot_jitter/best.pt --out ./runs/realworld_report.json
```

Serve locally:
```bash
python -m src.app
```

## Roadmap

See [`DIAGNOSIS.md`](./DIAGNOSIS.md) for the full v2 plan. Highlights:

- [ ] Replace EfficientNet-B0 with frozen DINOv2 features
- [ ] Add frequency-domain (FFT) branch for diffusion artifact detection
- [ ] Expand training data beyond Stable Diffusion v1.4 (GenImage, multi-generator coverage)
- [ ] Build a larger held-out eval set across multiple generators and platforms
- [ ] Production-grade calibration and per-class metrics
- [ ] FastAPI / Docker deployment for non-Gradio integrations

## License

MIT — see [`LICENSE`](./LICENSE).

## Origin

This project began as my submission for ECE 57000 (Spring 2026, Purdue) under the name `ai-image-screenshot-detector`. The course version is preserved at https://github.com/meownager/ai-image-screenshot-detector for grading reference. `realframe-detector` is the productized continuation.