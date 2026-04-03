# GitHub Actions Setup — pip + requirements.txt

> This folder contains the GitHub Actions workflow to integrate the **PR Review Agent** into your repository using **pip** and a `requirements.txt` file as the dependency manager.

---

## What's in This Folder

```
requirements-sample/
└── .github/            # GitHub Actions workflow — copy this folder directly into your repo
    └── workflows/
        └── pr-review.yml    
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

### Step 2 — Add `requirements.txt` to your target repo

Create a `requirements.txt` in the root of your target repo with the following:

```txt
llama-index
llama-index-llms-google-genai
PyGithub
python-dotenv
```

If you're using any other LLM Agent use the packages associated to them.
No need to install all of them just install the one which is needed

```txt
llama-index-llms-ollama 
llama-index-llms-anthropic
llama-index-llms-openai
```

> If your target repo already has a `requirements.txt`, just append these four packages to it.

---

### Step 3 — Copy the workflow file

Copy `pr-review.yml` from this folder into your target repo at:

```
your-target-repo/
└── .github/
    └── workflows/
        └── pr-review.yml
```

Create the `.github/workflows/` folders if they don't exist yet.

---

### Step 4 — Add GitHub Actions Secrets

If your using PAT make sure to add this permission:
```
GitHub → Settings → Developer Settings → Personal Access Tokens → Edit your token → check the workflow scope → Regenerate.
```

In your **target repository** on GitHub:

```
Settings → Secrets and Variables → Actions → New Repository Secret
```
```
Settings → Actions → General → Workflow permissions → set to "Read and write permissions"
```


| Secret Name (example) | Value                      |
|-----------------------|----------------------------|
| `GEMINI_API_KEY`      | Your Google Gemini API key |

Or any other API KEY associated with your LLM Model

> `GITHUB_TOKEN` is **automatically provided** by GitHub Actions — you do not need to add it.
>
> Using a different LLM? Add that model's API key with the appropriate name and update `agent.py` accordingly.

---

### Step 5 — Raise a PR and watch it work 🎉

Open any Pull Request on your target repo. GitHub Actions will:

1. Spin up a fresh Ubuntu runner
2. Install Python and pip dependencies from `requirements.txt`
3. Run `agent.py` with the PR number automatically injected
4. Post a detailed AI review comment on your PR

---

## Workflow Trigger

The `pr-review.yml` triggers on:

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
```

| Event         | When it fires                            |
|---------------|------------------------------------------|
| `opened`      | A new PR is created                      |
| `synchronize` | A new commit is pushed to an existing PR |
| `reopened`    | A previously closed PR is reopened       |

The agent only runs on actual code changes — not on label updates, assignments, or review requests.

---

## Environment Variables Injected by the Workflow

| Variable         | Source                                         | Description                                   |
|------------------|------------------------------------------------|-----------------------------------------------|
| `GITHUB_TOKEN`   | Auto (GitHub Actions)                          | Reads PR details and posts the review comment |
| `GEMINI_API_KEY` | Repository Secret                              | Your LLM API key                              |
| `PR_NUMBER`      | Auto (`github.event.pull_request.number`)      | The PR number that triggered the workflow     |

You Should manage only LLM API Key in secrets PR Number, PAT will be auto inserted.
GITHUB_REPOSITORY → this is a reserved word in github so it auto refers to current repo with out any changes

---

## Troubleshooting

**`agent.py` not found**
→ Make sure `agent.py` is in the **root** of your target repo, at the same level as `.github/`.

**Workflow does not trigger**
→ Confirm the workflow file is at exactly `.github/workflows/pr-review.yml`.

**`ModuleNotFoundError`**
→ Check that all four packages are present in `requirements.txt` and the install step ran successfully in the Actions log.

**LLM API errors**
→ Verify your `GEMINI_API_KEY` is correctly added as a GitHub Secret and the name matches exactly what's in the workflow file.

**Permission denied posting review**
→ Make sure the workflow has `pull-requests: write` in its permissions block.

---

> For full project overview, local testing (Approach 1), and architecture details — see the [main README](../README.md).