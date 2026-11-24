# Bangla Punctuation Restoration

Restore punctuation in unpunctuated Bengali text with fine-tuned BanglaBERT or
a fastText–BiLSTM baseline. The React interface calls an Express API, which
keeps a Python/PyTorch inference worker running for both models.

BanglaBERT aligns subword representations to words. The BiLSTM uses a shared
fastText embedding cache. Both models use constrained gap decoding and a common
renderer to turn predicted labels into punctuated text.

## Trained models

| Model | API value | Architecture | Git LFS checkpoint |
|---|---|---|---|
| Aligned BanglaBERT | `bert` | Fine-tuned BanglaBERT (ELECTRA) with subword-to-word pooling | [best.pt](inference/models/banglabert/best.pt) (422 MiB) |
| Packed fastText–BiLSTM | `bilstm` | Two-layer bidirectional LSTM with frozen 300-dimensional fastText embeddings | [best.pt](inference/models/bilstm/best.pt) (8.4 MiB) |

Run `git lfs pull` from the repository root to fetch both checkpoints and the
[fastText embedding cache](inference/embeddings/cache) used by the BiLSTM.
Checkpoint sizes cover the weights alone.

## Evaluation

| Evaluation split | Model | Macro-F1 (%) | Micro-F1 (%) | Ordered exact match (%) |
|---|---|---:|---:|---:|
| Validation | Aligned BanglaBERT | 68.58 | 93.05 | 83.74 |
| Validation | Packed fastText–BiLSTM | 50.32 | 89.61 | 77.81 |
| Reused benchmark | Aligned BanglaBERT | 68.93 | 92.96 | 83.46 |
| Reused benchmark | Packed fastText–BiLSTM | 52.05 | 89.77 | 77.93 |

Macro-F1 gives each punctuation class equal weight. Ordered exact match is the
percentage of sentences whose punctuation sequences match the reference in
every gap.

The benchmark covers 185,370 sentences from a previously used evaluation set.
Treat these scores as exploratory. See the
[benchmark report](report_materials/report_materials/metrics_and_benchmarks/test_benchmark_report.md)
for additional configurations and parenthesis span scores, and the
[model and training notes](report_materials/report_materials/README.md)
for validation results by punctuation mark.

## Run locally

Install Node.js 22.12 or newer, Python, and [Git LFS](https://git-lfs.com/).
From the repository root, fetch the model artifacts first. The checkpoints and
embedding cache total about 825 MB.

```bash
git lfs install
git lfs pull
npm ci
python3 -m venv venv
venv/bin/python -m pip install -r requirements.txt
npm run dev
```

Open [the React app](http://127.0.0.1:5173). On Windows, use
`py -m venv venv` and `venv\Scripts\python.exe -m pip install -r requirements.txt`.
The API detects the repository's `venv` on both platforms. Set `PYTHON_BIN`
to use another interpreter. Set `MONGO_URI` only if you want request history,
using [the environment template](.env.example) as a reference.

If the worker reports a missing checkpoint or embedding cache, use `git lfs ls-files`
to check that the files were downloaded. Git LFS pointers alone cannot be used
for inference.

## API and repository layout

- `GET /api/health` reports model-worker readiness and local artifact availability.
- `POST /api/restore` accepts `text` and a `model` of `bert` or `bilstm` and
  returns restored text with inference metadata.
- [client/](client/) contains the React interface.
- [server/](server/) handles validation, inference-worker communication, and optional request history.
- [inference/](inference/) contains the worker, model loading, decoding, and rendering.
- [bert/](bert/) and [bilstm/](bilstm/) contain training entry points.

For a production-style local build, run `npm run build` and then `npm start`.
Open [the application](http://127.0.0.1:5000).
