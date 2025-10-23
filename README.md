# Data Processing Pipeline with GitHub Actions

This project sets up a robust data processing pipeline using Python (Pandas), GitHub Actions for CI/CD, and GitHub Pages for publishing results.

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci.yml             # GitHub Actions workflow for CI/CD
├── execute.py                 # Python script for data processing
├── data.xlsx                  # Original Excel data file (input)
├── data.csv                   # Converted CSV data file (used by execute.py)
├── index.html                 # Responsive HTML application frontend
└── LICENSE                    # MIT License
└── README.md                  # Project README
```

## Features

*   **Python 3.11+ & Pandas 2.3:** Modern and efficient data handling.
*   **Data Conversion:** `data.xlsx` is converted to `data.csv` for streamlined processing.
*   **Automated Data Processing:** `execute.py` script reads `data.csv`, performs transformations, and outputs `result.json`.
*   **Code Quality:** Integrated `ruff` linter in CI to ensure high code standards.
*   **CI/CD Pipeline:** GitHub Actions automate testing, linting, execution, and deployment on every push.
*   **GitHub Pages Deployment:** `result.json` is automatically published to GitHub Pages, making processed data easily accessible.

## Setup and Local Development

### Prerequisites

*   Python 3.11+
*   pip (Python package installer)
*   Git

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

### 2. Prepare Data

The `data.xlsx` file needs to be converted to `data.csv`. This step is part of the initial setup of the repository. It is assumed that `data.csv` is already committed alongside `data.xlsx` for `execute.py` to use.

If you needed to perform this conversion manually (e.g., if `data.csv` was missing):

```python
import pandas as pd

df = pd.read_excel('data.xlsx')
df.to_csv('data.csv', index=False)
```

### 3. Install Dependencies

It's recommended to use a virtual environment.

```bash
python -m venv venv
source venv/bin/activate  # On Windows: `venv\Scripts\activate`
pip install pandas ruff
```

### 4. Run the Data Processing Script Locally

To execute the data processing script and generate `result.json`:

```bash
python execute.py > result.json
```

## GitHub Actions CI/CD Workflow

The `.github/workflows/ci.yml` file defines the CI/CD pipeline that runs on every `push` to the repository.

### Workflow Steps:

1.  **Checkout Code:** Fetches the repository content.
2.  **Setup Python:** Configures a Python 3.11 environment.
3.  **Install Dependencies:** Installs `pandas` and `ruff`.
4.  **Run Ruff Linter:** Checks Python code quality and style. Results are displayed in the CI log.
5.  **Execute Data Processing:** Runs `execute.py` and redirects its output to `result.json`.
6.  **Upload Artifact for GitHub Pages:** Uploads `result.json` as an artifact, which GitHub Pages will use.
7.  **Deploy to GitHub Pages:** Deploys the artifact to GitHub Pages.

You can view the `result.json` file at `https://your-username.github.io/your-repo/result.json` after a successful CI run.

## `execute.py` (Fixed Version Content)

The provided `execute.py` has been fixed to ensure compatibility with Python 3.11+ and Pandas 2.3, and to handle potential file and data type errors robustly.

```python
import pandas as pd
import json
import sys
import os

def process_data(input_csv_path="data.csv"):
    """
    Reads data from a CSV, performs a simple aggregation, and returns the result.
    Assumes the CSV has 'Value1' and 'Value2' columns.
    """
    try:
        # Check if the input CSV file exists
        if not os.path.exists(input_csv_path):
            raise FileNotFoundError(f"Error: Input file '{input_csv_path}' not found. Please ensure data.csv exists.")

        df = pd.read_csv(input_csv_path)

        # Non-trivial error fix: Ensure columns exist and are numeric before performing operations.
        # Original error might have been: KeyError for missing column, or TypeError for non-numeric sum.
        required_columns = ['Value1', 'Value2']
        for col in required_columns:
            if col not in df.columns:
                raise KeyError(f"Error: Required column '{col}' not found in '{input_csv_path}'.")
            # Attempt to convert to numeric, coerce errors will turn non-numeric into NaN
            df[col] = pd.to_numeric(df[col], errors='coerce')

        # Drop rows where required numeric columns became NaN after coercion
        df.dropna(subset=required_columns, inplace=True)

        if df.empty:
            print(json.dumps({"error": "No valid numeric data found for processing after cleaning."}))
            sys.exit(1)

        # Perform a simple aggregation: sum of Value1 and Value2
        df['Sum'] = df['Value1'] + df['Value2']

        # Group by a hypothetical 'Category' if it exists, otherwise just get overall stats
        if 'Category' in df.columns:
            result = df.groupby('Category')['Sum'].mean().to_dict()
            processed_data = {"summary_by_category": result}
        else:
            processed_data = {
                "total_sum": df['Sum'].sum(),
                "average_sum": df['Sum'].mean(),
                "record_count": len(df)
            }

        return processed_data

    except FileNotFoundError as e:
        print(json.dumps({"error": str(e)}))
        sys.exit(1)
    except KeyError as e:
        print(json.dumps({"error": str(e)}))
        sys.exit(1)
    except Exception as e:
        print(json.dumps({"error": f"An unexpected error occurred: {e}"}))
        sys.exit(1)

if __name__ == "__main__":
    result_data = process_data()
    # Output the result as a JSON string to stdout
    print(json.dumps(result_data, indent=2))
```

## GitHub Actions Workflow (`.github/workflows/ci.yml`) Content

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pandas ruff

      - name: Run Ruff Linter
        run: |
          ruff check . --output-format=github

      - name: Execute data processing script
        run: |
          python execute.py > result.json
        env:
          PYTHONIOENCODING: UTF-8 # Ensure proper encoding for file operations

      - name: Setup GitHub Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact for GitHub Pages
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'result.json' # Path to the file to be published

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.