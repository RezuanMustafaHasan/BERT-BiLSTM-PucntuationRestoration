# Bangla Punctuation Restoration

Restore punctuation in unpunctuated Bengali text with fine-tuned BanglaBERT or
a fastText–BiLSTM baseline. The React interface calls an Express API, which
keeps a Python/PyTorch inference worker running for both models.

BanglaBERT aligns subword representations to words. The BiLSTM uses a shared
fastText embedding cache. Both models use constrained gap decoding and a common
renderer to turn predicted labels into punctuated text.

## Evaluation

The repository includes [model and training notes](report_materials/report_materials/README.md)
and [exploratory benchmark results](report_materials/report_materials/metrics_and_benchmarks/test_benchmark_report.md).
The table compares configurations on a reused evaluation set. It does not report
performance on a fresh holdout.

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
