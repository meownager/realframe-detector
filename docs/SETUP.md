# Setup commands — run these yourself

Anything that touches your GitHub, Hugging Face, or Colab account has to be run by you (I don't have your credentials). Below is every command you need, copy-paste ready.

---

## 1. Create the GitHub repo and push

Run these in a terminal, from the project folder (`AI - Project on AI-image detection`).

### 1a. Create the empty repo on GitHub

Go to https://github.com/new and create a repo with:

- **Owner:** meownager
- **Repository name:** `ai-image-screenshot-detector`
- **Visibility:** Public (so the Hugging Face Space and the Colab notebook can clone it without auth)
- **Do not** initialize with README, .gitignore, or LICENSE — those are already in the project.

### 1b. Initialize git locally and push

```bash
cd realframe-detector

git init
git add .
git commit -m "Initial scaffold: CP1+CP2 snippets, training loop, eval, Gradio app"
git branch -M main
git remote add origin https://github.com/meownager/ai-image-screenshot-detector.git
git push -u origin main
```

If prompted for credentials: GitHub no longer accepts passwords. Use a **personal access token** instead of your password. Create one at https://github.com/settings/tokens (scope: `repo`). Paste the token where it asks for password.

### Verify

After pushing, `https://github.com/meownager/ai-image-screenshot-detector` should show all the files. If it does, you're good.

---

## 2. Run training on Colab

1. Open https://colab.research.google.com/.
2. File → Upload notebook → pick `notebooks/train_colab.ipynb` from this project.
3. **Runtime → Change runtime type → T4 GPU**.
4. Run cells top-to-bottom. First time, it'll ask for Google Drive permission (optional — only if you want outputs to persist).
5. Expected wall-clock: 2–4 hours for the full ablation. For a fast dry run first, set `--max-per-class 500` in the download cell and `--epochs 1` in the ablation cell.

The notebook saves results to:

- `./runs/ablation/*/best.pt` — model checkpoints
- `./runs/ablation/results.json` — ablation table
- `./runs/ablation/screenshot_jitter/test_report.json` — test metrics
- `./runs/ablation/screenshot_jitter/calibration.json` — temperature scaling

And, if you mounted Drive, copies all of that to `/content/drive/MyDrive/ece57000_ai_screenshot_detector/runs/`.

---

## 3. Hugging Face token (for the optional upload cell)

1. https://huggingface.co/settings/tokens → "New token" → Role: **Write** → name it `colab-upload`.
2. Copy the token. Paste it when the Colab notebook asks.
3. The final cell will push `best.pt` and `calibration.json` to `meownager/ai-image-screenshot-detector` on HF — this is what the Gradio Space loads at runtime.

---

## 4. Deploy the Gradio Space (Phase E, later)

After training completes, I'll walk you through creating the HF Space. That's a separate phase.

---

## Troubleshooting

- **`ModuleNotFoundError: datasets`** — the `pip install` cell at the top of the notebook didn't run. Re-run it.
- **Colab kicks you out mid-training** — mount Drive first (cell 3 in the notebook); training resumes from the last-completed config because `ablation.py` writes `results.json` after each config.
- **GPU out of memory** — reduce `--batch-size` from 64 to 32 in the ablation cell.
- **Download stalls** — Hugging Face rate-limits anonymous downloads; run `!huggingface-cli login` in a cell above the download cell and paste any token (read is fine for this).
