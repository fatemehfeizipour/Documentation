# Learning GitHub: Beyond Code Storage

A record of what I learned building my first real Git/GitHub workflow, the mistakes I made, what they taught me, and what I now understand GitHub is actually for.

## Where I started

I thought Git tracks changes and GitHub stores the code. That's true, but incomplete, it treats GitHub like a filing cabinet instead of what it actually is: a collaboration and automation platform built around Git.

## What I got wrong, and what it taught me

### 1. I thought `git add` was for new files and `git commit` was for changes

Wrong on both counts. Both commands work on any file, new or modified. The real distinction is **stage**, not file type:

- `git add` stages a file, new or edited, meaning "include this in my next commit"
- `git commit -m "message"` saves everything currently staged as a permanent snapshot

`git status` is what actually tells you a file's state: untracked, modified, or staged. That's the distinction that matters, not which command to reach for.

### 2. I confused forking with cloning

I forked a course repo on GitHub, then separately cloned the *original* repo (not my fork) to my machine, without realizing those were two disconnected actions. Running `git remote -v` showed someone else's GitHub username instead of mine, which is what surfaced the mistake.

**What I now understand:**
- **Branch**, isolate work inside a repo I already have write access to
- **Fork**, my own full copy of a repo I *don't* have write access to, made specifically so I can work on it independently

The fix: clone your own fork directly, or redirect `origin` to point at it after the fact.

### 3. Rebase looked like magic until I broke it down

When my local branch and the remote branch had both moved forward independently, I needed a way to reconcile them. Two options:

- **Merge**, ties both histories together with a merge commit; nothing about existing commits changes
- **Rebase**, removes my local commits, applies the remote's new commits underneath, then replays mine back on top, producing one straight, linear history

**The problem rebase actually solves:** keeping history readable. Frequent merge commits clutter `git log` with bookkeeping that isn't real code change. Rebase avoids that, at the cost of rewriting commit hashes, which is why it should only be used on local, unpushed work, never on commits already shared with others.

### 4. I didn't know GitHub disables Actions on new forks, on purpose

Forking a repo with CI/CD workflows configured meant those workflows were automatically disabled on my fork. This isn't a bug, it's a security default. Workflows can execute arbitrary code and use secrets, so GitHub requires an explicit opt-in before any fork's automation runs. This is the moment it really clicked that GitHub is managing trust and permissions between people who don't know each other, not just hosting files.

## What GitHub actually solves

| Feature | Problem it solves |
|---|---|
| Forks | Contributing to repos you don't have write access to |
| Branches | Isolating work-in-progress without breaking `main` |
| Pull Requests | Review and accountability before code merges |
| Remotes (`origin`, `upstream`) | Keeping distributed local copies synced to a shared source of truth |
| Actions (with fork restrictions) | Automation, gated by permission so it can't be abused by untrusted forks |

None of that is storage. It's coordination infrastructure for people working on the same code without stepping on each other.

### 5. I mixed up single-dash and double-dash flag syntax

I wrote `git add --Av .` and `git commit --am ""`, both invalid. The rule: single dashes combine short, single-letter flags (`-A`, `-v` → `-Av`); double dashes spell out the full-word version of one flag at a time (`--all`, `--verbose`). `--Av` isn't valid syntax at all, Git reads it as a long flag literally named "Av," which doesn't exist.

Correct versions:
- `git add -Av .`, `-A` stages everything in the whole repo (new, modified, and deleted files, regardless of current folder); `-v` just prints each file as it's staged, it doesn't change what gets staged. Pairing `-A` (whole repo) with `.` (current folder onward) is slightly redundant, usually you'd see one or the other on its own.
- `git commit -am "message"`, `-a` auto-stages modified *tracked* files (skipping `git add`), `-m` attaches a message. An empty message (`-am ""`) will fail; Git requires a real, non-empty message. `-a` still never picks up brand-new untracked files, those always need an explicit `git add <filename>` first.

## Still learning

This is a living document, I'll keep adding to it as I hit new concepts and new mistakes worth learning from.
