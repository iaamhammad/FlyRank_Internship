# Agent instructions

Before any task in this repo: **read `skills/README.md`** — it is the router.
Find the task in its table and load exactly **one** skill (plus `skills/flyrank/flyrank-data/SKILL.md`
whenever the task touches the data). Do not load every skill; keep context small.

## Ground rules
- Search the repo before assuming something is missing or not implemented.
- One task per conversation; finish and verify before starting the next.
- Never commit datasets (CI blocks them). Never print private data, client names, or raw queries.
- The intern validates your output — end each task by running the notebook top to bottom.
- Read the label trap before touching features: `trend_direction` / `trend_pct` define the
  label and are never features; warehouse label proxies are built from split-window impressions,
  never used as features. Full column gotchas: `docs/data-dictionary.md` + `flyrank-data` skill.

## Where work goes (read-only vs yours)
- `scripts/01–05` + `run_all.py` are the **reference pipeline — do not edit** (reviewers expect
  it unchanged). For experiments, copy a script into `work/` and edit the copy. The one editable
  scripts file is `scripts/ml_utils.py` (feature lists).
- Everything you produce lives under `work/`: notebooks in `work/notebooks/`,
  `work/outputs/` for JSON metric receipts + figures. `work/README.md` maps assignment → notebook.
- `notebooks/01–03` are the shared first-win notebooks; don't break them (CI re-runs `03`).

## Verify (no unit tests or linters exist)
- Notebooks are verified by executing them end-to-end:
  `python -m jupyter nbconvert --to notebook --execute --inplace work/notebooks/<file>.ipynb`
- minimal env additions are needed: `pip install -r requirements.txt` is NOT enough for
  execution — also `pip install nbformat nbclient ipykernel jupyter`.
- New notebooks (weeks 3+) need a setup cell — a fresh kernel lacks `duckdb`:
  `%pip install -q duckdb huggingface_hub pandas scikit-learn matplotlib`.
- Sample pipeline (offline, no token): `python scripts/run_all.py` (~1 min).
- Restore the one shipped dataset with: `git checkout -- data/raw/content_refresh_anonymized.csv`.
- CI re-runs the pipeline and fails any committed dataset — keep `git status` clean of `*.csv`
  (they're gitignored; `work/outputs/*.json` and figures are the committing artifacts).

## Secrets
- The Hugging Face READ token lives in repo-root `.env` as `HF_token`
  (notebooks also accept `HF_TOKEN`); it is gitignored. Load it with `load_dotenv`
  or `getpass` in the notebook — never hardcode it in a cell (repo is public).
  `.env` loads relative to the notebook: from `work/notebooks/` use `load_dotenv("../../.env")`.
- Warehouse notebooks connect through DuckDB:
  `con.execute("CREATE OR REPLACE SECRET hf (TYPE huggingface, TOKEN '<token>')")`
  before any read of `hf://datasets/FlyRank/internship-warehouse`. A one‑month
  partition scan takes ~1 min — don't repeat full scans in a loop.
