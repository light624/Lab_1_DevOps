# ✨ CLI Collaboration Guide — Elegant & Animated (A → Z)

Welcome! This is a polished, creative, terminal-friendly guide to collaborate with Git and GitHub from the command line. It focuses on clarity and usability while giving a stylish terminal experience (colors, ASCII banners, and simple animations). Run the small helper scripts below to get the colored/animated output in your shell.

Note: This guide assumes:
- You already have a GitHub account.
- Git is installed.
- One team member created the remote repo and invited collaborators.

---

## Quick setup for color & animation (optional, fun)
Install a few packages to get colorful banners and easy-to-read output.
- Debian/Ubuntu:
```bash
sudo apt update
sudo apt install -y figlet lolcat jq
```
- macOS (Homebrew):
```bash
brew install figlet lolcat jq
```

`figlet` = ASCII art banners, `lolcat` = rainbow colorizer, `jq` = JSON parsing (handy), optional.

If you don't want these tools, all steps work without them — the scripts will automatically fall back to plain text.

Example banner (try this in your terminal):
```bash
figlet "GIT LAB" | lolcat
```

---

## 1) Identity: configure Git (per student)
Each student sets their global identity once:
```bash
git config --global user.name "Your Firstname Lastname"
git config --global user.email "you@example.com"
git config --list
```

Tip: use `git config --global core.editor "code --wait"` to use VS Code for commit messages.

---

## 2) Repository creation (one person)
- One member creates the repo on GitHub (via the browser) and invites teammates: Settings → Manage access → Invite collaborators.
- Choose `main` as the primary branch (or `master` depending on your org).

---

## 3) Clone the repo (every member)
Pick the folder where you keep projects, then clone:

```bash
cd ~/Documents             # example
git clone https://github.com/ORG_OR_USER/REPO_NAME.git
cd REPO_NAME
```

Colorized status (example):
```bash
echo -e "\e[1;34mCloned:\e[0m $(pwd)"
git --no-pager status --short
```

---

## 4) Branching strategy (recommended)
- main — production-ready, protected
- develop — shared development branch
- personal branches — `dev-<firstname>`, `feature/<short>`, `fix/<id>`

Naming examples:
- feature/add-login
- fix/typo-readme
- dev-alex

---

## 5) Create and publish `develop` (one member)
Create `develop` and push it upstream so everyone can use it:
```bash
git checkout -b develop       # create & switch
git push -u origin develop    # publish & set upstream
```

Others fetch and switch:
```bash
git fetch origin
git checkout develop
```

---

## 6) Daily workflow (each task)
1. Sync develop:
```bash
git checkout develop
git pull origin develop
```
2. Create your branch:
```bash
git checkout -b dev-<yourname>
```
3. Work & test locally.
4. Stage and commit:
```bash
git add -A
git commit -m "type: short description (ex: feat: add health endpoint)"
```
5. Push your branch:
```bash
git push -u origin dev-<yourname>
```

Tip: use semantic commit prefixes: feat, fix, chore, docs, refactor, test.

---

## 7) Integrate into `develop`
Two recommended approaches:

A) Quick local merge (fast, when trusted)
```bash
git checkout develop
git pull origin develop
git merge dev-<yourname>
git push origin develop
```

B) Pull Request (recommended)
- On GitHub: open a PR from `dev-<yourname>` → `develop`.
- Request review, run CI, then merge via GitHub once approved.

---

## 8) Conflict scenario & resolution (concrete)
When two people edit the same lines, a merge conflict happens.

Example flow:
- Student A merges first (no problem).
- Student B tries to merge and sees:
```text
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

Open the file — you'll see markers:
```text
<<<<<<< HEAD
Text from develop (or earlier commit)
=======
Text from dev-<yourname>
>>>>>>> dev-<yourname>
```

Resolve by editing the file:
- Decide which version to keep (or combine both).
- Remove the markers `<<<<<<<`, `=======`, `>>>>>>>`.
- Save.

Finish the merge:
```bash
git add README.md
git commit -m "fix: resolve merge conflict in README.md"
git push origin develop
```

Communicate! If the conflict is about behavior, talk with your teammate before choosing an outcome.

---

## 9) Handy scripts (copy & paste)
Save this helper as `bin/gitlab-helper.sh` and make executable: `chmod +x bin/gitlab-helper.sh`
It prints a banner, does a safe clone, creates develop if missing, and shows a spinner while fetching.

```bash
#!/usr/bin/env bash
# name: gitlab-helper.sh
# usage: ./gitlab-helper.sh https://github.com/ORG/REPO.git

REPO_URL="$1"
REPO_DIR="$(basename -s .git "$REPO_URL")"
BANNER() {
  if command -v figlet >/dev/null 2>&1 && command -v lolcat >/dev/null 2>&1; then
    figlet "GIT LAB" | lolcat
  else
    echo -e "\e[1;35m=== GIT LAB ===\e[0m"
  fi
}

spinner() {
  local pid=$1
  local delay=0.08
  local spinstr='|/-\'
  while ps a | awk '{print $1}' | grep -q "$pid"; do
    for i in $(seq 0 3); do
      printf "\r\e[1;33m%s\e[0m" "${spinstr:i:1}"
      sleep $delay
    done
  done
  printf "\r"
}

BANNER
if [ -z "$REPO_URL" ]; then
  echo -e "\e[1;31mUsage:\e[0m ./gitlab-helper.sh https://github.com/ORG/REPO.git"
  exit 1
fi

echo -e "\e[1;34mCloning:\e[0m $REPO_URL"
git clone "$REPO_URL" &
spinner $!
if [ -d "$REPO_DIR" ]; then
  echo -e "\e[1;32mCloned into:\e[0m $REPO_DIR"
  cd "$REPO_DIR" || exit
  # ensure develop exists
  if ! git ls-remote --exit-code --heads origin develop >/dev/null 2>&1; then
    echo -e "\e[1;36mCreating 'develop' branch...\e[0m"
    git checkout -b develop
    git push -u origin develop
  else
    echo -e "\e[1;36m' develop ' branch already exists. Fetching...\e[0m"
    git fetch origin develop
  fi
fi
git --no-pager status --short
```

---

## 10) Example conflict demo (scripted)
Two quick commands to simulate conflict locally (for training):

```bash
# start from develop
git checkout -b dev-A
echo "Change by A" >> README.md
git add README.md
git commit -m "chore: change by A"
git push -u origin dev-A

# in another shell, simulate B
git checkout develop
git pull origin develop
git checkout -b dev-B
echo "Change by B" >> README.md
git add README.md
git commit -m "chore: change by B"
git push -u origin dev-B

# Now merge A into develop (simulate remote merge)
git checkout develop
git merge dev-A
git push origin develop

# Now attempt to merge B locally and see conflict
git checkout develop
git pull origin develop
git merge dev-B   # will show conflict in README.md
```

Resolve as shown in section 8.

---

## 11) Best practices & checklist (short)
- Keep commits small and atomic.
- Use clear commit messages: `type(scope): short description`
- Pull before you push: `git pull --rebase origin develop` (if your team uses rebase)
- Use PRs for code review and CI checks.
- Communicate when a large change may cause conflicts.
- Add a README section describing local setup & commands to run.

---

## 12) Nice-to-have extras
- Add a CONTRIBUTING.md with branch policy, code style, and PR checklist.
- Protect `main` in GitHub (branch protection rules).
- Configure required CI checks on PRs.
- Add CODEOWNERS for automatic review assignments.

---

## 13) Quick reference cards (copyable snippets)

Create branch and push:
```bash
git checkout -b dev-<yourname>
git push -u origin dev-<yourname>
```

Sync develop:
```bash
git checkout develop
git pull origin develop
```

Merge and push:
```bash
git checkout develop
git pull origin develop
git merge dev-<yourname>
git push origin develop
```

Resolve conflict:
```bash
# edit file to remove markers then:
git add <file>
git commit -m "fix: resolve merge conflict in <file>"
git push origin develop
```

---


