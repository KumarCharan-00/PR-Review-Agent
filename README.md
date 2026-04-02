# GitHub PR Review Agent

An AI-powered **Multi-Agent system** that automatically reviews Pull Requests using **LlamaIndex AgentWorkflow**, **Google Gemini**, and the **GitHub API** — triggered on every PR via GitHub Actions.

[![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat&logo=python)](https://python.org)
[![LlamaIndex](https://img.shields.io/badge/LlamaIndex-AgentWorkflow-purple?style=flat)](https://llamaindex.ai)
[![Gemini](https://img.shields.io/badge/Google-Gemini-orange?style=flat&logo=google)](https://ai.google.dev)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-2088FF?style=flat&logo=github-actions)](https://github.com/features/actions)

---
Note: I am using Google Gemini in this Project but you can use any AI Model

## What This Does

This project implements a **Multi-Agent AI pipeline** that acts as an intelligent code reviewer. When a Pull Request is raised, the agent system automatically:

1. **Fetches PR context** — title, author, description, diffs, and commit details
2. **Drafts a review** — analyzes code quality, missing tests, undocumented endpoints, and contribution guideline compliance
3. **Self-reviews the draft** — a dedicated agent validates the review meets quality standards before posting
4. **Posts the review to GitHub** — as a formal PR review comment, directly on the PR

No human reviewer needs to be online. The agent does the first pass for you.

---

##  Architecture — Multi-Agent Workflow

                     ┌─────────────────────────────────────────────┐
                     │          AgentWorkflow (LlamaIndex)         │
                     └─────────────────────────────────────────────┘
                                           │
                    ┌──────────────────────┼───────────────────────┐
                    ▼                      ▼                       ▼
          ┌──────────────────┐  ┌──────────────────┐   ┌───────────────────────┐
          │   ContextAgent   │  │  CommentorAgent  │   │  ReviewAndPosting     │
          │                  │  │                  │   │      Agent            │
          │ • get_pr_details │  │ • Draft review   │   │ • Quality check       │
          │ • get_commits    │  │ • Address author │   │ • Post to GitHub      │
          │ • get_file_      │  │ • Follow rubric  │   │ • Re-request rewrite  │
          │   contents       │  │                  │   │   if needed           │
          └──────────────────┘  └──────────────────┘   └───────────────────────┘
                    │                      │                       │
                    └──────────────────────┴───────────────────────┘
                                           │
                                  Shared Context Store
                              (gathered_contexts, draft_comment,
                               review_comment)


**Agent Handoff Flow:**
```
CommentorAgent ──► ContextAgent (fetch data)
                       │
                       ▼
               CommentorAgent (draft review)
                       │
                       ▼
           ReviewAndPostingAgent (validate + post)  ──►  Add Comment in GitHub
                       │
                       ▼ (if review fails quality check)
               CommentorAgent (rewrite)
```

Each agent communicates via a **shared state store** (LlamaIndex `Context`), enabling clean decoupling between context gathering, drafting, and posting.

---

## 🛠️ Tech Stack

| Layer | Technology                                      |
|---|-------------------------------------------------|
| **Agent Framework** | LlamaIndex `AgentWorkflow` + `FunctionAgent`    |
| **LLM** | Google Gemini (`gemini-3.1-flash-lite-preview`) |
| **GitHub Integration** | PyGitHub (`PyGithub`)                           |
| **CI/CD Trigger** | GitHub Actions                                  |
| **Environment** | Python 3.10+                                    |

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- A GitHub Personal Access Token (PAT) with the following scopes:
  - `repo` (Full control of private repositories)
  - `pull_requests: write` (Post review comments)
- A Google Gemini API Key (or Api Key based on the AI model you prefer to use)

---

## Approach 1 — Run Locally Against Any PR

Use this to quickly test the agent against **any existing PR in any repo** — no changes needed to the target repository.

### 1. Clone this repo

```bash
git clone https://github.com/KumarCharan-00/pr-review-agent.git
cd pr-review-agent
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Create a `.env` file
You can rename .env.sample and use it
Note: If you are using any third Party AI Agent then Specify its key.

> **PAT Permissions Required:**
> Go to GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
> Enable: `repo`, `read:org`, `pull_requests` (write)

### 4. Point the agent at a target repo

```
# Point this to any repo you want to review PRs on in the .env
GITHUB_REPO="https://github.com/<TARGET-USERNAME>/<TARGET-REPO>.git"
```

> 🧪 **Want a ready-made test target?**
> Use [`KumarCharan-00/recipes-api`](https://github.com/KumarCharan-00/recipes-api) — a sample REST API used to validate this agent end-to-end.
> Fork it, raise a PR (edit any file), grab the PR number from the URL, and run the agent.

### 5. Run the agent (For Local testing)

```bash
python agent.py
```

The agent fetches the PR, drafts a review, self-validates it, and posts it as a GitHub review comment.

---

## Approach 2 — Automated via GitHub Actions (Recommended)

Embed the agent directly into any repository so it **triggers automatically on every PR raised**.

### Step 1 — Add the agent to your target repo

Copy `agent.py` into your target repository.

### Step 2 — Choose your dependency manager
 
Two ready-to-use setups are provided — pick one:
 
| Approach | Folder | Guide |
|---|---|---|
| pip + requirements.txt | `requirements-sample/` | [README](./requirements-sample/README.md) |
| Poetry | `poetry-sample/` | [README](./poetry-sample/README.md) |
 
Each folder contains the exact `pr-review.yml` workflow file and step-by-step instructions.
### Step 3 — Add GitHub Actions Secrets

Navigate to your GitHub repository:
```
Settings → Secrets and Variables → Actions → New Repository Secret
```

| Secret Name                                         | Value                            |
|-----------------------------------------------------|----------------------------------|
| `GITHUB_TOKEN` (This can be ignored no need to add) | Auto-provided by GitHub Actions  | 
| `GEMINI_API_KEY`                                    | Your Google Gemini API key       |

Note: If you're using any other provided add the respective secret name.

Make sure to 

### Step 4 — Create the GitHub Actions Workflow

Create `.github/workflows/pr-review.yml` in your repo:

Sample for this is already provided you can use it from `poetry-sample` or else `requirements-sample`

### Step 5 — Raise a PR and watch the agent work 🎉

Every time a PR is opened or updated, GitHub Actions triggers the multi-agent workflow and a detailed AI review is posted as a comment.

> **See it live:** Validated end-to-end on [`KumarCharan-00/recipes-api`](https://github.com/KumarCharan-00/recipes-api) using this exact setup.

---

## 📁 Project Structure

```
pr-review-agent/
├── agent.py                   # Core multi-agent workflow
├── requirements.txt           # Python dependencies
├── .env.example               # Example environment variables
├── .github/
│   └── workflows/
│       └── pr-review.yml      # GitHub Actions trigger
└── README.md
```

---

## 🔑 Environment Variables Reference

| Variable | Description |
|---|---|---|
| `GITHUB_TOKEN` | GitHub PAT with `repo` + `pull_requests:write` scope |
| `GEMINI_API_KEY`  | Google Gemini API key |
| `PR_NUMBER` | PR number to review (auto-injected by GitHub Actions) |

---

## 🤝 Contributing

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push: `git push origin feature/your-feature`
5. Open a PR — and let the agent review it 😄


---
If this project helped or inspired you, drop a star ⭐️ — it helps others to find it!
  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=flat&logo=linkedin)](https://linkedin.com/in/your-profile)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=flat&logo=github)](https://github.com/KumarCharan-00)

> If You like my work connect with me on LinkedIn and Follow on GitHub

---
## Attribution

> Special Thanks to Hyperskill for such a wonderful project
---
