# GitHub Actions Setup — Poetry

> This folder contains the GitHub Actions workflow to integrate the **PR Review Agent** into your repository using **Poetry** as the dependency manager.

---

## What's in This Folder

```
poetry-sample/
├── pr-review.yml       # GitHub Actions workflow — copy this into your repo
└── README.md           # This guide
```

---

## Step-by-Step Setup

### Step 1 — Copy `agent.py` into your target repo

From the root of `PR-Review-Agent`, copy `agent.py` into the **root of the repository** where you want PRs to be automatically reviewed:

```bash
cp agent.py /path/to/your-target-repo/agent.py
```

---

### Step 2 — Set up Poetry in your target repo (if not already)

If your target repo doesn't have Poetry set up yet, initialise it first:

```bash
poetry init
```

Follow the prompts. This creates a `pyproject.toml` in your repo root.

---

### Step 3 — Add the agent dependencies via `poetry add`

Run the following commands inside your target repo to add the required packages:

```bash
poetry add llama-index
poetry add llama-index-llms-google-genai
poetry add PyGithub
poetry add python-dotenv
```

> Each `poetry add` command automatically updates your `pyproject.toml` and `poetry.lock` file.
> Make sure to commit **both** files to your repo — the workflow depends on `poetry.lock` to install the exact same versions in CI.

After running the above, your `pyproject.toml` should include:

```toml
[tool.poetry.dependencies]
python = "^3.10"
llama-index = "*"
llama-index-llms-google-genai = "*"
PyGithub = "*"
python-dotenv = "*"
```

---

### Step 4 — Copy the workflow file

Copy `pr-review.yml` from this folder into your target repo at:

```
your-target-repo/
└── .github/
    └── workflows/
        └── pr-review.yml
```

Create the `.github/workflows/` folders if they don't exist yet.

---

### Step 5 — Add GitHub Actions Secrets

In your **target repository** on GitHub:

```
Settings → Secrets and Variables → Actions → New Repository Secret
```

| Secret Name | Value |
|---|---|
| `GEMINI_API_KEY` | Your Google Gemini API key |

> `GITHUB_TOKEN` is **automatically provided** by GitHub Actions — you do not need to add it.
>
> Using a different LLM? Add that model's API key with the appropriate secret name and update `agent.py` accordingly.

---

### Step 6 — Raise a PR and watch it work 

Open any Pull Request on your target repo. GitHub Actions will:

1. Spin up a fresh Ubuntu runner
2. Install Poetry
3. Run `poetry install` to restore dependencies from `poetry.lock`
4. Run `agent.py` with the PR number automatically injected
5. Post a detailed AI review comment on your PR

---

## Workflow Trigger

The `pr-review.yml` triggers on:

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
```

| Event | When it fires |
|---|---|
| `opened` | A new PR is created |
| `synchronize` | A new commit is pushed to an existing PR |
| `reopened` | A previously closed PR is reopened |

The agent only runs on actual code changes — not on label updates, assignments, or review requests.

---

## Environment Variables Injected by the Workflow

| Variable | Source | Description |
|---|---|---|
| `GITHUB_TOKEN` | Auto (GitHub Actions) | Reads PR details and posts the review comment |
| `GEMINI_API_KEY` | Repository Secret | Your LLM API key |
| `PR_NUMBER` | Auto (`github.event.pull_request.number`) | The PR number that triggered the workflow |
| `GITHUB_REPO` | Auto (`github.server_url`/`github.repository`) | Full repo URL passed to `agent.py` |

---

## Troubleshooting

**`agent.py` not found**
→ Make sure `agent.py` is in the **root** of your target repo, at the same level as `.github/`.

**Workflow does not trigger**
→ Confirm the workflow file is at exactly `.github/workflows/pr-review.yml`.

**`poetry: command not found`**
→ The workflow installs Poetry automatically. If you see this error check that the Poetry install step completed successfully in the Actions log.

**`poetry.lock` not found or out of sync**
→ Run `poetry lock` locally and commit the `poetry.lock` file. The workflow runs `poetry install` which requires the lock file to be present in the repo.

**`ModuleNotFoundError` at runtime**
→ Confirm all four `poetry add` commands ran successfully locally and that both `pyproject.toml` and `poetry.lock` are committed and pushed.

**LLM API errors**
→ Verify your `GEMINI_API_KEY` is correctly added as a GitHub Secret and the name matches exactly what's in the workflow file.

**Permission denied posting review**
→ Make sure the workflow has `pull-requests: write` in its permissions block.

---

> For full project overview, local testing (Approach 1), and architecture details — see the [main README](../README.md).