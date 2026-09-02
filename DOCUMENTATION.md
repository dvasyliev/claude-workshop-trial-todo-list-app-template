# Building this app with Claude Code

Step-by-step instructions for reproducing this repo from its starting point. Each step is one prompt, and the prompts are the files in this folder.

Every step follows the same loop: enter plan mode, send the prompt, answer whatever the assistant asks, approve the plan, check the result in the browser.

## Step 0 — Make sure Git is installed

Step 1 clones the repo, so Git has to be there first. Check it in a terminal:

```bash
git --version
```

If it prints something like `git version 2.43.0`, skip ahead to Step 1. If the command is not found, install it for your platform.

### macOS

The quickest path is the Command Line Tools, which ship Git:

```bash
xcode-select --install
```

A dialog opens; accept it and wait for it to finish. If you already use [Homebrew](https://brew.sh), `brew install git` works too and gives you a newer version.

### Linux

Use your distribution's package manager:

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install git

# Fedora
sudo dnf install git

# Arch
sudo pacman -S git
```

### Windows

Install [Git for Windows](https://git-scm.com/download/win) — it bundles Git Bash, which is the terminal to run every command in this document. Accept the installer defaults unless you have a reason not to. With `winget` available you can instead run:

```powershell
winget install --id Git.Git -e
```

### Verify

Open a **new** terminal — the old one will not see the freshly installed Git — and run `git --version` again. Then set your identity, once per machine, so commits between steps are attributed:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

## Step 1 — Get the base and start it

The Vite app with Tailwind already exists — that is the starting point, not something you build. Clone it and check out the scaffold commit, which is the state before any of the steps below were run:

### Clone the repo

```bash
git clone https://github.com/dvasyliev/claude-workshop-trial-todo-list-app.git
cd claude-workshop-trial-todo-list-app
```

### Install dependencies and start the dev server

```bash
npm install
npm run dev
```

It serves the stock Vite page on http://localhost:5173. Leave it running; it hot-reloads through every step below.

## Step 2 — Build the To-Do app

**Prompt:** [docs/1-create-todo-list-app-prompt.md](docs/1-create-todo-list-app-prompt.md)

1. Enter plan mode — `Shift+Tab` twice in the CLI, or type `/plan`.
2. Paste the prompt.
3. Answer whatever it asks. The questions vary between runs.

**Check in the browser**

1. Add a title-only to-do.
2. Open "Show all fields", add a second with description, due date and priority `high`.
3. Check the first one off — it goes struck through and muted.
4. Edit the second inline, save, edit again, cancel.
5. Delete both.

## Step 3 — Add filters and persistence

**Prompt:** [docs/2-add-additional-functionality-prompt.md](docs/2-add-additional-functionality-prompt.md)

1. Enter plan mode again and paste the prompt.
2. Answer whatever it asks, and approve the plan.

**Check in the browser**

1. Add several to-dos at different priorities and complete one.
2. Set Completed to Active, then to Completed — the list narrows both ways.
3. Add a Priority filter on top; confirm the two apply together.
4. Pick a combination that matches nothing and read the empty state.
5. Reload the page: the to-dos come back, the filters reset to All.

## Step 4 — Review the generated code

**Skill:** [docs/3-ai-code-review-skill-instructions.md](docs/3-ai-code-review-skill-instructions.md), installed at `.claude/skills/ai-code-review/`

The app works at the end of Step 3. Code that survives a click-through has not been reviewed, so make the review its own step.

```
/ai-code-review
```

The skill makes three passes — does it do what was asked, does it break, should it look like this — and every finding must name a concrete input and say what happens with it. That rule is what keeps "consider adding validation" out of the report.

Read the findings before accepting them; what turns up depends on what the earlier steps produced. When it asks what to fix, answering **Everything** and **yes** to writing the missing tests is what leaves the repo in the state you see here.

Expect the fixes to move code around — extracting a pure function so it can be tested without rendering, adding a test setup file, adding a test runner to `devDependencies`. Run `npm test` afterwards.

## Working this way in your own repo

- **Use plan mode for anything spanning more than one file.** Plan mode is read-only: the assistant explores, asks, and writes a plan you approve before any edit lands. The alternative is discovering the design decisions in the diff.
- **Put constraints in the prompt, not in review comments.** "All state in App", "one component per file", "no new dependencies" — cheap to state up front, expensive to retrofit.
- **Give it a definition of done it can check.** "npm run dev works, and I can add a to-do with all fields, check it off, edit it inline, and delete it" is testable. "Build a good to-do app" is not.
- **Say what is out of scope.** Naming filtering and persistence as out of scope in Step 2 is what kept Step 2 small enough to review.
- **Review before you trust.** Some defects cannot surface in a click-through at all — a corrupt-storage overwrite needs a specific stored value, and a secure-context crash needs a phone on the LAN.
- **Commit between steps.**
