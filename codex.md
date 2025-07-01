## Summary

This document defines detailed, structured instructions for the Codex agent to autonomously navigate, understand, and modify the **Crypto Trend–Following System** repository. It covers agent role definition, repository comprehension, tool usage, coding conventions, workflows, error handling, and collaboration guidelines—ensuring reliable, maintainable, and secure task execution([openai.com](https://openai.com/index/introducing-codex/?utm_source=chatgpt.com))([cdn.openai.com](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf?utm_source=chatgpt.com)).

---

## 1. Agent Role & Goals

* **Primary Function:** Act as a software-engineering assistant that reads repository contents, reasons over code, and performs atomic code changes (e.g., implementing TODO items, fixing bugs, writing tests)([openai.com](https://openai.com/index/introducing-codex/?utm_source=chatgpt.com)).
* **Task Granularity:** Target small, self-contained issues—each corresponding to one logical change—to facilitate incremental commits and reviews([medium.com](https://medium.com/%40vikuman/mastering-the-react-pattern-build-smarter-ai-agents-that-can-think-and-act-50f863718115?utm_source=chatgpt.com)).
* **Autonomy Boundaries:** Only modify code within `src/`, config files under `src/config/`, and workflow files in `.github/`; do not alter user data in `data/` or repository metadata unless explicitly instructed([cdn.openai.com](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf?utm_source=chatgpt.com)).

---

## 2. Repository Comprehension

* **File Structure Awareness:**

  * Recognize top-level documentation (`README.md`, `TODO.md`, `CODEX.md`) as guidance, not code.
  * Map `src/data/`, `src/backtest/`, `src/logging/`, `src/config/`, and `.github/workflows/` to functional modules([reddit.com](https://www.reddit.com/r/ChatGPTCoding/comments/1gvjpfd/building_ai_agents_that_actually_understand_your/?utm_source=chatgpt.com)).
* **Dependency Graph:** Build an internal representation of module imports and function/class definitions to support context-aware code modifications([reddit.com](https://www.reddit.com/r/ChatGPTCoding/comments/1gvjpfd/building_ai_agents_that_actually_understand_your/?utm_source=chatgpt.com)).
* **Configuration Loading:** Treat `config.sample.yml` as schema; actual `config.yml` and `.env` are ignored but their keys define runtime parameters([docs.github.com](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot?utm_source=chatgpt.com)).

---

## 3. Toolchain & Environment

* **Python–Centric Execution:** Assume Python 3.10–3.12 environment with dependencies listed in `requirements.txt`. Use `pipenv` or `venv` as appropriate([platform.openai.com](https://platform.openai.com/docs/codex/overview?utm_source=chatgpt.com)).
* **Backtrader Integration:** Prefer leveraging Backtrader for backtesting when implementing strategy logic; fall back to custom code if unavailable([cdn.openai.com](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf?utm_source=chatgpt.com)).
* **Tenacity for Retries:** Employ `tenacity` decorators for API calls in `src/data/` modules to implement exponential backoff and retry logging([dailydoseofds.com](https://www.dailydoseofds.com/ai-agents-crash-course-part-10-with-implementation/?utm_source=chatgpt.com)).
* **Docker Awareness:** Recognize multi-stage `Dockerfile` and conditional CI build flags in `.github/workflows/docker.yml` for containerized tasks([promptingguide.ai](https://www.promptingguide.ai/agents/introduction?utm_source=chatgpt.com)).

---

## 4. Prompt & Action Patterns

* **ReAct Paradigm:** Use the ReAct pattern—alternate between chain-of-thought reasoning and tool-invoking actions—for complex code changes (e.g., “think: verify function signature; action: update code; think: re-run tests”)([medium.com](https://medium.com/%40vikuman/mastering-the-react-pattern-build-smarter-ai-agents-that-can-think-and-act-50f863718115?utm_source=chatgpt.com)).
* **Instruction Focus:**

  1. Read function/class docstrings and signatures.
  2. Identify missing behavior per TODO or issue specification.
  3. Generate minimal patch.
  4. Add or update unit tests in `tests/` to cover new behavior.
  5. Run lint and type checks locally (simulate CI step).
  6. Summarize changes in commit message following “type(scope): description” format (e.g., `feat(backtest): add 20% rebalance threshold logic`)([cloud.google.com](https://cloud.google.com/discover/what-is-prompt-engineering?utm_source=chatgpt.com)).

---

## 5. Coding Conventions

* **Style & Formatting:**

  * Comply with `black` formatting (88-char line length) and `isort` import ordering([lekha-bhan88.medium.com](https://lekha-bhan88.medium.com/openais-agent-best-practices-the-blueprint-we-ve-all-been-waiting-for-c86bc1f805e1?utm_source=chatgpt.com)).
  * Adhere to PEP 8 naming and docstring standards; annotate public functions with type hints([cloud.google.com](https://cloud.google.com/discover/what-is-prompt-engineering?utm_source=chatgpt.com)).
* **Logging & Telemetry:** Use `src/logging/telemetry.py` logger for all new code; log at appropriate levels (`DEBUG` for internal state, `INFO` for high-level events, `ERROR` on exceptions)([medium.com](https://medium.com/%40filipespacheco/i-created-an-ai-agent-to-build-readme-files-here-is-what-i-learn-3ae207771d37?utm_source=chatgpt.com)).
* **Configuration Access:** Read settings via centralized config loader (`src/config/config_loader.py`), not via direct file reads in modules([docs.github.com](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot?utm_source=chatgpt.com)).

---

## 6. Testing & Quality Gates

* **Unit Tests:** Create tests in `tests/` using `pytest`; each new feature or bug fix must include one or more tests covering normal, edge, and failure cases([cdn.openai.com](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf?utm_source=chatgpt.com)).
* **Coverage Enforcement:** Ensure total coverage ≥50% using `pytest-cov`; write focused tests for data validation, strategy signals, and backtest edge conditions([platform.openai.com](https://platform.openai.com/docs/guides/agents?utm_source=chatgpt.com)).
* **Lint & Type Checks:** Simulate CI steps by running `flake8`, `mypy`, and `pre-commit` hooks locally before finalizing patches([lekha-bhan88.medium.com](https://lekha-bhan88.medium.com/openais-agent-best-practices-the-blueprint-we-ve-all-been-waiting-for-c86bc1f805e1?utm_source=chatgpt.com)).

---

## 7. CI/CD Workflow Interaction

* **CI Actions (`ci.yml`):**

  * On PRs: install dependencies, run lint, type checks, and tests+coverage.
  * Report status checks back to GitHub; fail PR if any step errors or coverage falls below threshold([code.visualstudio.com](https://code.visualstudio.com/docs/copilot/copilot-customization?utm_source=chatgpt.com)).
* **Docker Build (`docker.yml`):**

  * Triggered manually or on tags; build multi-stage container.
  * Honor `if: env.BUILD_DOCKER == 'true'` to respect free-tier constraints([promptingguide.ai](https://www.promptingguide.ai/agents/introduction?utm_source=chatgpt.com)).

---

## 8. Data Management & Validation

* **Flat-File Format:** CSVs in `data/` follow `{TICKER}-{SOURCE}-OHLCV-{START}-{END}.csv` with header `Timestamp,Open,High,Low,Close,Volume,NotValid`([servicenow.com](https://www.servicenow.com/docs/bundle/yokohama-intelligent-experiences/page/administer/now-assist-ai-agents/concept/gg-creating-aia.html?utm_source=chatgpt.com)).
* **Validation Script:** Use `data/scripts/validate_data.py` to mark `NotValid=True` for missing or malformed rows; CI should flag any CSV with `NotValid` entries([cdn.openai.com](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf?utm_source=chatgpt.com)).

---

## 9. Security & Secrets

* **.env Usage:** Load sensitive keys (e.g., CryptoCompare API) from `.env`; ensure `.env` is git-ignored and values are injected via GitHub Secrets for CI runs([docs.github.com](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot?utm_source=chatgpt.com)).
* **Secret Rotation:** Prompt maintainers when usage patterns change or if consumption limits are reached; do not hard-code keys anywhere([docs.github.com](https://docs.github.com/en/copilot/customizing-copilot/adding-personal-custom-instructions-for-github-copilot?utm_source=chatgpt.com)).

---

## 10. Collaboration & Issue Management

* **Mapping TODO.md to Issues:** Treat each bullet in `TODO.md` as a GitHub Issue; when an issue is closed, remove or mark the entry as done([medium.com](https://medium.com/%40vikuman/mastering-the-react-pattern-build-smarter-ai-agents-that-can-think-and-act-50f863718115?utm_source=chatgpt.com)).
* **PR Etiquette:** Draft clear, concise commit messages; reference related issues (e.g., `Closes #123`) and include test status badges in PR description([docs.github.com](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot?utm_source=chatgpt.com)).

---

## 11. Governance & Safety

* **Agent Safeguards:**

  * Do not attempt destructive actions (e.g., permission changes, remote deletions) without explicit instructions.
  * Log each action in a changelog entry or audit trail.
* **Ethical Guidelines:** Follow OpenAI's practices for governing agentic AI systems to mitigate risks—ensure transparency, review generated code for biases or vulnerabilities, and maintain human oversight([cdn.openai.com](https://cdn.openai.com/papers/practices-for-governing-agentic-ai-systems.pdf?utm_source=chatgpt.com)).
