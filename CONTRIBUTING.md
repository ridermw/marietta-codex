
# Contributing to Crypto Trend–Following System

Thank you for your interest in contributing! We welcome bug fixes, new features, docume## 7. Thank You

Your contributions make this project better. Whether it's a typo fix or a major new feature, we appreciate your effort and time!tion improvements, and test enhancements. To make collaboration smooth and efficient, please follow these guidelines.

---

## 1. Getting Started

1. **Fork & Clone**  

   ```bash
   git clone https://github.com/<your-username>/crypto-trend-system.git
   cd crypto-trend-system
   ```

2. **Create a Branch**
   Use a descriptive branch name, prefixed by type:

   ```text
   feature/<short-description>   # new features  
   fix/<short-description>       # bug fixes  
   docs/<short-description>      # documentation only  
   test/<short-description>      # tests only  
   chore/<short-description>     # build, tooling, or formatting  
   ```

   Example:

   ```bash
   git checkout -b feature/add-donchian-parameter
   ```

3. **Install & Activate Environment**

   ```bash
   pipenv install --dev
   pipenv shell
   ```

4. **Run Linter & Tests Locally**

   ```bash
   pre-commit run --all-files
   pytest --maxfail=1 --disable-warnings -q
   ```

---

## 2. Code Style & Quality

* **Formatting**

  * We use `black` for code formatting.
  * We use `isort` to order imports.
  * Configure your editor or run `pre-commit run --all-files` before committing.

* **Linting**

  * We use `flake8` for style enforcement.
  * Keep line length ≤ 88 characters (Black default).
  * Avoid unused imports and variables.

* **Type Checking**

  * We use `mypy` for static type checks.
  * Ensure new code is type-annotated where appropriate.

---

## 3. Writing Tests

* **Coverage**

  * All new code must include tests.
  * We enforce a minimum of **50% total coverage**.
  * Run coverage locally:

    ```bash
    pytest --cov=src --cov-fail-under=50
    ```

* **Test Structure**

  * Place tests in the `tests/` directory, mirroring `src/` structure.
  * Use descriptive test names and docstrings.
  * Use fixtures for reusable setup.

---

## 4. Submitting a Pull Request

1. **Push Your Branch**

   ```bash
   git push origin feature/add-donchian-parameter
   ```

2. **Open a PR**

   * Base repository: `main` branch.
   * Provide a clear title and description.
   * Reference any related issues (e.g. “Closes #123”).

3. **PR Checklist**

   * [ ] Branch follows naming conventions.
   * [ ] Linter (`pre-commit`) passes.
   * [ ] All tests pass and coverage ≥ 50%.
   * [ ] Documentation updated (if applicable).
   * [ ] Code follows existing style and patterns.
   * [ ] CHANGELOG.md updated (for new features or bug fixes).

4. **Review Process**

   * At least one core contributor will review your PR.
   * Address review comments promptly.
   * Rebase or merge the latest `main` to resolve conflicts.

---

## 5. Reporting Issues

Use the GitHub [Issues](https://github.com/your-org/crypto-trend-system/issues) feature:

* Choose the correct template: **Bug report** or **Feature request**.
* Provide as much detail as possible:

  * Steps to reproduce
  * Expected vs. actual behavior
  * Environment (Python version, OS)
  * Relevant logs or screenshots

---

## 6. Code of Conduct

Please read our [Code of Conduct](CODE_OF_CONDUCT.md). We expect all contributors to adhere to its guidelines.

---

## 7. Thank You!

Your contributions make this project better. Whether it’s a typo fix or a major new feature, we appreciate your effort and time!
