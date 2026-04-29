# Block 2b — Backup, Rollback and Recovery

**Date:** April 29, 2026  
**Environment:** WSL Ubuntu on Windows (VS Code integrated terminal)  
**Roadmap:** Cloud Skills Roadmap — Month 1–2 Foundations  
**Time spent:** ~30 minutes  

---

## What I Learned

Before touching any production environment, always have a rollback plan. This block covers two levels of rollback — file level (manual backup/restore) and Git level (commit history). This is the mindset that separates a careful engineer from one who breaks prod and can't recover.

---

## The Golden Rule

```
Restore first. Investigate second.
Never debug in prod while it's down.
```

---

## Layer 1 — File Level Backup and Restore

### Create a backup before making any change

```bash
cp file.txt file.txt.bak
ls
# file.txt      ← working file
# file.txt.bak  ← backup
```

Always do this before editing any config file, script, or important document on a server.

---

### Simulate a bad change

```bash
echo "BAD CONFIG - this broke prod" > file.txt
cat file.txt
# Output: BAD CONFIG - this broke prod
```

The `>` operator overwrote everything. Original content is gone. This is your "prod is down" moment. 🔴

---

### Rollback — restore from backup

```bash
cp file.txt.bak file.txt
cat file.txt
# Output: original content restored
```

Prod is back up. 🟢 Now you can safely investigate what went wrong.

---

### Delete and retrieve

```bash
# Delete the file
rm file.txt
ls
# file.txt is gone

# Restore from backup
cp file.txt.bak file.txt
cat file.txt
# Content is back
```

---

## Critical Difference — `>>` vs `>`

| Operator | Behavior | Safe? |
|---|---|---|
| `>>` | Append — adds to end, keeps existing content | ✅ Yes |
| `>` | Overwrite — wipes everything, replaces with new | ⚠️ Dangerous |

**Real world example:**
```bash
echo "Deployment started" >> deploy.log    # safe — keeps history
echo "Deployment started" > deploy.log     # dangerous — wipes all previous logs
```

---

## Layer 2 — Git Level Rollback

This is the professional way. Every commit is a complete snapshot of your repo. You can roll back to any point in history.

### Check commit history

```bash
git log --oneline
```

**Output:**
```
d7ca310 (HEAD -> main)  Bad update - this broke prod
32d39c4 (origin/main)   Add Block 2 - SSH key management notes
b3f7be8                 Add Block 1 - Linux terminal fundamentals notes
b0defd5                 Initial commit
```

Each line is a commit — a saved state of your entire repo at that moment.

---

### Simulate a bad commit

```bash
echo "BAD CHANGE - broke prod" >> month-01-02-foundations/block1-linux-fundamentals.md
git add .
git commit -m "Bad update - this broke prod"
git log --oneline
```

Bad commit is now in history. In real prod this would be a broken config, bad code, or a wrong environment variable pushed to the server.

---

### Safe rollback — git revert

```bash
git revert HEAD
```

This opens a text editor for the commit message — press **Ctrl+X** to accept default and save.

```bash
git log --oneline
```

**Output:**
```
3db005c (HEAD -> main)  Revert "Bad update - this broke prod"
d7ca310                 Bad update - this broke prod    ← still in history
32d39c4 (origin/main)   Add Block 2 - SSH key management notes
b3f7be8                 Add Block 1 - Linux terminal fundamentals notes
b0defd5                 Initial commit
```

**Key insight:** `git revert` does NOT delete the bad commit. It creates a new commit that undoes the changes. History is preserved — this is why it's safe for prod and shared repos.

---

### Push clean state to GitHub

```bash
git push origin main
git log --oneline
```

**Output:**
```
3db005c (HEAD -> main, origin/main)  Revert "Bad update - this broke prod"
d7ca310                              Bad update - this broke prod
32d39c4                              Add Block 2 - SSH key management notes
```

`HEAD -> main` and `origin/main` pointing to same commit = local and GitHub in sync. ✅

---

## git revert vs git reset — Know the Difference

| Command | What it does | Safe for prod? |
|---|---|---|
| `git revert HEAD` | Creates new commit that undoes changes. History preserved. | ✅ Yes — always use this on shared repos |
| `git reset --hard HEAD~1` | Deletes the commit completely. Rewrites history. | ⚠️ Only before pushing. Never on shared repos. |

**Rule of thumb:**
- Already pushed to GitHub → use `git revert`
- Not pushed yet, local only → can use `git reset --hard`

---

## Full Rollback Workflow — What You Practiced

```
1. File backup created     → cp file.txt file.txt.bak
         ↓
2. Bad change simulated    → echo "BAD" > file.txt (overwrote content)
         ↓
3. File restored           → cp file.txt.bak file.txt
         ↓
4. Bad Git commit made     → echo "BAD" >> md file && git commit
         ↓
5. Git revert executed     → git revert HEAD (safe, history preserved)
         ↓
6. Clean state pushed      → git push origin main
```

---

## Real Prod Rollback Workflow

```
1. Something breaks in prod
         ↓
2. RESTORE FIRST — git revert or redeploy last good Docker image
         ↓
3. Prod is stable again
         ↓
4. Investigate root cause safely (not in prod)
         ↓
5. Fix, test in staging, deploy properly
```

---

## Key Commands Reference

| Command | Purpose |
|---|---|
| `cp file.txt file.txt.bak` | Create file backup |
| `cp file.txt.bak file.txt` | Restore from backup |
| `rm file.txt` | Delete file |
| `>> file` | Append to file (safe) |
| `> file` | Overwrite file (dangerous) |
| `git log --oneline` | See full commit history |
| `git revert HEAD` | Safe undo — creates new revert commit |
| `git reset --hard HEAD~1` | Hard reset — deletes commit (local only) |
| `tail filename` | View last 10 lines of a file |

---

## Interview Answer — "How do you handle a bad deployment in prod?"

> "First priority is restoring service — either revert the Git commit with `git revert` which preserves history safely, or redeploy the last known good Docker image. Once prod is stable, I investigate the root cause in isolation, fix it properly, and redeploy with a tested change."

---

## Connection to Docker and Kubernetes (coming next)

This same rollback thinking applies at every layer:

| Layer | Rollback method |
|---|---|
| File | `cp backup original` |
| Git | `git revert HEAD` |
| Docker | `docker pull image:previous-tag && docker run` |
| Kubernetes | `kubectl rollout undo deployment/myapp` |
| Azure | Redeploy previous VM image or App Service slot swap |

The mental model is identical — restore first, investigate second.

---

## Resources

- [Git revert documentation](https://git-scm.com/docs/git-revert)
- [Atlassian — undoing commits](https://www.atlassian.com/git/tutorials/undoing-changes)

---

*Part of the [Cloud Skills Roadmap](https://github.com/Mrushi1903/cloud-skills-learning) — Month 1–2 Foundations*
