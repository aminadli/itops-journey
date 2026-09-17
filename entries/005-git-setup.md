### What We Did

1. Created a private GitHub repo: `it-ops-stack`
2. Configured Git identity on the PC
3. Created the repo structure (`docker/`, `runbooks/`, `network/`)
4. Generated a personal SSH key on the PC and added to GitHub as **account key**
5. Generated **deploy keys** on the server and added to the repo as **deploy keys**
6. Configured `~/.ssh/config` on the server for GitHub
7. Cloned the repo on the server

### Difficulties

#### Difficulty 1: "Permission denied (publickey)" When Pushing

**What happened:** `git push` from the PC failed with `git@github.com: Permission denied (publickey)`.

**Root cause:** The public key on the PC was not added to GitHub.

**Fix:** Added `~/.ssh/id_ed25519.pub` to GitHub → Settings → SSH and GPG Keys.

#### Difficulty 2: "Nothing to Commit" After Editing

**What happened:** Edited a file, ran `git commit -m "..."`, got "nothing to commit, working tree clean".

**Root cause:** Forgot `git add`. `git commit` only commits **staged** changes.

**Fix:**
```bash
git add .
git commit -m "message"
```

#### Difficulty 3: Server Pull Blocked by Local Changes

**What happened:** `git pull` failed with `Your local changes would be overwritten`.

**Root cause:** Files were edited **directly on the server** with `nano`. Those edits diverged from Git.

**Fix:**
```bash
git checkout -- .
git pull
```

#### Difficulty 4: Typo Created a Ghost File

**What happened:** Ran `nano dockercompose.yml` (no hyphen) by mistake, creating an empty ghost file that confused the workflow.

**Root cause:** A typo in the filename.

**Fix:** `rm dockercompose.yml`.

#### Difficulty 5: `sudo nano` on the PC

**What happened:** Edited repo files on the PC with `sudo nano`. Files became owned by `root`, causing permission issues.

**Fix:** `sudo chown -R user:user ~/it-ops-stack`. And never use `sudo` for files in the home directory.

### What We Learned

- **Edit on the PC. Commit, push. Pull on the server. Never edit on the server.**
- **The server is a mirror, not a source of truth.**
- **`git add` before `git commit`.**
- **Deploy keys are read-only and per-repo. Account keys are per-user.**
- **`sudo nano` in your home directory creates permission problems.**

The Git workflow took more iterations to get right than any other phase. Every later service benefited from this discipline.

