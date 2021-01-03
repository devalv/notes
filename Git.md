## Undo

Delete the most recent commit, keeping the work you’ve done:

`git reset --soft HEAD~1`

Delete the most recent commit, destroying the work you’ve done:

`git reset --hard HEAD~1`

## Git hooks

### pre-commit

#### Install

[https://pre-commit.com/](https://pre-commit.com/)
1. pip install pre-commit
1. pre-commit install
1. pre-commit run –all-files

#### linter config

```
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.3.0
    hooks:
    -   id: check-yaml
    -   id: end-of-file-fixer
    -   id: trailing-whitespace
-   repo: https://github.com/psf/black
    rev: 19.3b0
    hooks:
    -   id: black
        language_version: python3
-   repo: https://gitlab.com/PyCQA/flake8
    rev: 3.8.3
    hooks:
    -   id: flake8
        additional_dependencies:
            - flake8-builtins
            - flake8-bugbear
            - flake8-import-order
            - flake8-docstrings
            - flake8-quotes
        args: [--inline-quotes=double, --max-line-length=90]
```

#### Github Actions

```
# Linting: run black, flake8 and small auto fixers.

name: Linter

on:
  pull_request:
    branches: [ develop, main ]
  push:
    branches: [ feature/* ]

jobs:
  pre-commit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
        with:
            python-version: '3.8'
      - uses: pre-commit/action@v2.0.0
```

### flake8 --install-hook

#### Install

1. pip install flake8 flake8-builtins flake8-bugbear

2. flake8 --install-hook git

3. git config --bool flake8.strict true
  
#### Github Actions
Пример автозапуска тестов при push и pull_request в дев и мастер.

```
# Runs tests and coverage measuring.
name: tests

on:
  push:
    branches: [ master, develop ]
  pull_request:
    branches: [ master, develop ]

jobs:
  build:

    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.5]

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v1
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        python -m pip install pipenv
        pip install flake8
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
        if [ -f Pipfile ]; then pipenv install --dev; fi
    - name: Lint with flake8
      run: |
        # stop the build if there are Python syntax errors or undefined names
        pipenv run flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
        # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
        pipenv run flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
    - name: Test with unittest and build coverage xml report
      run: |
        pipenv run coverage run -m unittest discover tests/ && pipenv run coverage xml
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v1
      with:
        flags: unittests
        env_vars: OS,PYTHON
        name: codecov-umbrella
        fail_ci_if_error: true
```