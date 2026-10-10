# Code

Finding evidence for scientific claims (SciFact): retrieval + claim verification.

Seed: 42 everywhere. Run on Google Colab, GPU: Tesla T4.

## Layout

Put the bootcamp files next to this folder:

```
data/                  corpus.jsonl, train_claims.jsonl, val_claims.jsonl, eval_claims.jsonl
check_format.py
results_template.json
code/                  the notebooks
```

The notebooks create `predictions/` (submitted files) and `work/` (cached score matrices and a log of all val numbers).

## How to reproduce

Run the notebooks in order. Each one reads what the previous ones wrote to `work/`.

From the folder that contains `data/` and `code/`:

```
pip install numpy pandas scipy torch transformers sentencepiece protobuf PyStemmer jupyter
cd code
jupyter nbconvert --to notebook --execute --inplace 01_task1_bm25.ipynb
jupyter nbconvert --to notebook --execute --inplace 02_task2_dense.ipynb
jupyter nbconvert --to notebook --execute --inplace 03_task3_hybrid_rerank.ipynb
jupyter nbconvert --to notebook --execute --inplace 04_task4_finetune.ipynb
jupyter nbconvert --to notebook --execute --inplace 05_task5_verification.ipynb
jupyter nbconvert --to notebook --execute --inplace 06_package_submission.ipynb
```

On Colab: open each notebook, Runtime > Run all. The first cell asks for `NLP_Inter_IIT.zip` (the project folder as a zip) and the last cell
downloads the updated zip for the next notebook. Notebooks 2 to 5 need a GPU runtime.

| Notebook | Task | Writes |
|---|---|---|
| 01_task1_bm25 | Task 1: BM25 implemented from scratch, tuned on val | t1_bm25.trec, val_bm25.trec |
| 02_task2_dense | Task 2: general and biomedical bi-encoders, timing and memory | t2_general.trec, t2_scientific.trec, val_general.trec |
| 03_task3_hybrid_rerank | Task 3: rank fusion / score interpolation, cross-encoder reranking | t3_hybrid.trec, t3_rerank.trec |
| 04_task4_finetune | Task 4: bi-encoder fine-tuning with random and with hard negatives | t4_random.trec, t4_hard.trec |
| 05_task5_verification | Task 5: NLI classifier, verification predictions, error categories | t5_verification.csv, t5_val_verification.csv, t5_val_errors.csv |
| 06_package_submission | results.json, report.md, final zip, format check | results.json, report.md |

## Models

| Use | Model |
|---|---|
| general embedder | BAAI/bge-large-en-v1.5 |
| scientific embedder | ncbi/MedCPT-Query-Encoder + ncbi/MedCPT-Article-Encoder |
| reranker | ncbi/MedCPT-Cross-Encoder |
| fine-tuned bi-encoder | BAAI/bge-large-en-v1.5 |
| verifier | MoritzLaurer/DeBERTa-v3-base-mnli-fever-anli (finetuned) |

All are open-source encoder models under 400M parameters. Model selection uses only the validation split (highest val nDCG@10, val F1 for the verifier).
Runtimes on a T4: notebook 1 about 2 min (CPU), 2 about 15 min with the model comparison, 3 about 8 min, 4 about 25 min, 5 about 15 min.
