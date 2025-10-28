# Contributing

We welcome contributions to this project! This repository is primarily a **tutorial and educational resource**, so we are focused on changes that enhance clarity, correctness, and pedagogical value. By participating, you agree to abide by our [Code of Conduct](./CODE_OF_CONDUCT.md).

---

## Ways to Contribute

### 1. Reporting Issues

- **Found a bug?** If the instructions or code examples don't work, please open a new **Issue**. Be sure to include your operating system, Docker version, and the exact steps to reproduce the error.
- **Suggesting Enhancements?** If you have an idea to make the tutorial clearer or suggest a common, robust alternative approach (e.g., a different base image), feel free to open an Issue to discuss it first.

### 2. Submitting Pull Requests (PRs)

We are looking for PRs that:

- **Fix typos or incorrect commands** in the `README.md` or configuration files.
- **Improve the explanation or clarity** of the steps.
- **Correct any non-functional code** in `src/index.ts` or configuration files.

## Pull Request Guidelines

1. **Fork** the repository to your own GitHub account.
2. **Clone** your fork locally and navigate into the directory:
   ```bash
   git clone [https://github.com/YOUR_USERNAME/docker-local-api.git](https://github.com/YOUR_USERNAME/docker-local-api.git)
   cd docker-local-api
   ```
3. **Create a new, descriptive branch** for your changes (e.g., `fix/typo-in-readme`).
4. **Make your changes**, ensuring you run `docker compose up` to verify that the project still works after your changes.
5. **Write clear and concise commit messages.** Each commit should address a single logical change.
6. **Push your branch** and open a **Pull Request** against the `main` branch of this repository.

Please clearly describe what you changed and why in the PR description.

---

## Code of Conduct

Please review and adhere to our [Code of Conduct](./CODE_OF_CONDUCT.md) in all your interactions with the project and its community.

Thank you for helping us make this a better resource!
