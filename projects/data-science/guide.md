# Data Science Projects with Claude Code

## Tooling

| Tool | Purpose |
|------|---------|
| **jupyter** | Interactive notebooks for exploration |
| **pandas** | Data manipulation and analysis |
| **numpy** | Numerical computing |
| **scikit-learn** | Machine learning models |
| **matplotlib / seaborn** | Visualization |
| **pytest** | Testing data pipelines |

## Project Structure

```
myproject/
├── notebooks/          — exploratory Jupyter notebooks
│   ├── 01_eda.ipynb
│   └── 02_modeling.ipynb
├── src/
│   ├── data/           — data loading and cleaning
│   ├── features/       — feature engineering
│   ├── models/         — model training and evaluation
│   └── utils/          — shared utilities
├── data/
│   ├── raw/            — original immutable data
│   ├── processed/      — cleaned data
│   └── external/       — third-party data
├── models/             — serialized trained models
├── tests/
├── pyproject.toml
└── CLAUDE.md
```

## Workflow

1. **Explore** data interactively in notebooks
2. **Prototype** models and transformations in notebooks
3. **Extract** working code into `src/` modules
4. **Test** data pipelines and model behavior
5. **Document** findings and decisions

## Testing Data Pipelines

```python
def test_clean_data_removes_nulls():
    raw = pd.DataFrame({"a": [1, None, 3]})
    result = clean_data(raw)
    assert result["a"].isna().sum() == 0

def test_model_output_shape():
    X = np.random.rand(10, 5)
    model = load_model("models/latest.pkl")
    predictions = model.predict(X)
    assert predictions.shape == (10,)
```

## Reproducibility

- **Set random seeds** everywhere: `np.random.seed(42)`, `random_state=42`
- **Version your data** — track data files with DVC or record checksums
- **Pin all dependency versions** in `pyproject.toml`
- **Track experiments** — use MLflow, Weights & Biases, or a simple CSV log
- **Never modify raw data** — keep `data/raw/` immutable

## Common Commands

```bash
jupyter lab                     # start notebook server
pytest                          # run pipeline tests
python -m src.models.train      # run training script
ruff check . && ruff format .   # lint and format
```

## Tips for Claude

- When asked to analyze data, start by examining shape, dtypes, and nulls
- Prefer vectorized pandas/numpy operations over loops
- Always show plots inline when working in notebooks
- Suggest `scikit-learn` pipelines for reproducible transforms
- Keep notebook cells small and focused
