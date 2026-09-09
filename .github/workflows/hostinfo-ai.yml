name: Host Info AI

on:
  push:
    paths:
      - python/**
  pull_request:
  workflow_dispatch:

permissions:
  security-events: write

jobs:
  build-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.11','3.12']
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install coverage

      - name: Verify code builds
        run: |
          python -m py_compile python/hostinfo_server.py

      - name: Run unit tests with coverage
        run: |
          cd python/
          coverage run -m unittest discover
          coverage report -m

      - name: Analyze Test Quality
        uses: Aetherr-Agency/DeepDive@v1
        with:
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}

  lint:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        linter: [flake8, pylint, ruff, mypy]
    steps:
      - uses: actions/checkout@v3
      - uses: advanced-security/python-lint-code-scanning-action@v1
        with:
          linter: ${{ matrix.linter }}

      - name: AI Lint Feedback (summary of issues)
        if: github.event_name == 'pull_request'
        uses: anc95/ChatGPT-CodeReview@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run PyCharm Security
        uses: tonybaloney/pycharm-security@master
        with:
          path: python/

      - name: AI Security Risk Summary
        uses: anc95/ChatGPT-CodeReview@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

  docker:
    needs: [build-test, security-scan]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      - name: Build Docker image
        uses: docker/build-push-action@v6
        with:
          context: ./python
          load: true
          push: false
          tags: student/hostinfo:latest

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: 'student/hostinfo:latest'
          format: 'table'
          exit-code: '0'
          ignore-unfixed: true
          vuln-type: 'os,library'
          severity: 'CRITICAL,HIGH'

      - name: AI Container Review
        uses: anc95/ChatGPT-CodeReview@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

  ai-code-review:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: anc95/ChatGPT-CodeReview@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

  ai-summary:
    needs: [build-test, lint, security-scan, docker]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: AI Workflow Summary
        uses: anc95/ChatGPT-CodeReview@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
