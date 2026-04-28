# Block 2 — SSH Key Management

**Date:** April 26, 2026  
**Environment:** WSL Ubuntu on Windows (VS Code integrated terminal)  
**Roadmap:** Cloud Skills Roadmap — Month 1–2 Foundations  
**Time spent:** ~30 minutes  

---

## What I Learned

SSH (Secure Shell) is the standard protocol for securely connecting to remote servers. Instead of passwords, SSH uses cryptographic key pairs — a private key you keep and a public key you share. This is how every cloud engineer connects to Azure VMs, pushes to GitHub, and sets up automated deployments.

---

## Why SSH Keys Instead of Passwords?

| Method | Security | Convenience | Industry Standard |
|---|---|---|---|
| Password | ❌ Can be brute-forced, intercepted | ❌ Type every time | ❌ No |
| SSH Key Pair | ✅ Cryptographic, never sent over network | ✅ Passwordless once set up | ✅ Yes |

---

## What I Did

### Step 1 — Checked for existing SSH keys

```bash
ls ~/.ssh/
# Output: No such file or directory
```

No SSH folder existed — starting fresh. This is normal on a new machine or fresh WSL install.

---

### Step 2 — Generated a key pair

```bash
ssh-keygen -t ed25519 -C "mrushireddy2232@gmail.com"
```

**What each part means:**
- `ssh-keygen` — the tool that generates SSH keys
- `-t ed25519` — the algorithm to use. Ed25519 is modern, fast, and more secure than older RSA keys
- `-C "email"` — a comment to identify which key belongs to whom. Useful when managing multiple keys

**Accepted all defaults:**
- File location: `/home/rushimaddi/.ssh/id_ed25519` (default)
- Passphrase: none (press Enter twice)

**Output:**
```
Your identification has been saved in /home/rushimaddi/.ssh/id_ed25519
Your public key has been saved in /home/rushimaddi/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:r5NGMkIVX+ViH4RzjSJ5DLV7mD6LLvNE0r82yO3A5Jc mrushireddy2232@gmail.com
```

---

### Step 3 — Verified the key files

```bash
ls -la ~/.ssh/
```

**Output:**
```
-rw-------  id_ed25519      (419 bytes) ← private key
-rw-r--r--  id_ed25519.pub  (107 bytes) ← public key
```

**Why permissions matter:**

| File | Permissions | Meaning |
|---|---|---|
| `id_ed25519` | `rw-------` (600) | Only owner can read/write. SSH refuses to use keys with looser permissions — security feature |
| `id_ed25519.pub` | `rw-r--r--` (644) | Anyone can read — safe to share publicly |

---

### Step 4 — Read the public key

```bash
cat ~/.ssh/id_ed25519.pub
```

**Output:**
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHc71isrmbvvRbe1uYNZhPa8l+9eVnz/PuDnjXn0WLxU mrushireddy2232@gmail.com
```

This is what gets shared — pasted into GitHub, Azure VMs, and any server you want to access. The private key never leaves your machine.

---

### Step 5 — Started SSH agent and loaded the key

```bash
eval $(ssh-agent -s)
# Output: Agent pid 19613

ssh-add ~/.ssh/id_ed25519
# Output: Identity added: /home/rushimaddi/.ssh/id_ed25519
```

**What the SSH agent does:**

The SSH agent is a background process that holds your private key in memory. Without it, you'd need to specify your key file manually on every connection. With it, SSH automatically uses the right key.

**Auto-load on WSL startup** (added to ~/.bashrc):
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519 2>/dev/null
```

This runs every time WSL opens so the agent is always ready.

---

### Step 6 — Added public key to GitHub

1. Went to `github.com/settings/ssh/new`
2. Title: `WSL Ubuntu - Office Laptop`
3. Key type: **Authentication Key**
4. Pasted the public key from `cat ~/.ssh/id_ed25519.pub`
5. Clicked **Add SSH key**

---

### Step 7 — Tested the connection

```bash
ssh -T git@github.com
```

**Output:**
```
The authenticity of host 'github.com (140.82.112.3)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi Mrushi1903! You've successfully authenticated, but GitHub does not provide shell access.
```

**What happened behind the scenes:**
1. WSL connected to GitHub's server at `140.82.112.3`
2. GitHub challenged: "prove who you are"
3. SSH agent used the private key to respond cryptographically
4. GitHub matched it against the uploaded public key
5. Authentication successful — "Hi Mrushi1903!"

**"GitHub does not provide shell access"** — this is normal. It means auth worked but GitHub doesn't let you log into a Linux shell on their servers.

**`known_hosts` file** — after connecting, GitHub's server fingerprint was saved to `~/.ssh/known_hosts`. Next time you connect, SSH verifies it matches. This prevents man-in-the-middle attacks.

---

## The Lock and Key Mental Model

```
Private key  (id_ed25519)      = Your house key
                                  Never share.
                                  Never copy to a server.
                                  Lives only on your machine.

Public key   (id_ed25519.pub)  = Your padlock
                                  Safe to share.
                                  Put it on GitHub, Azure VMs,
                                  any server you want to access.
```

When you connect, your machine uses the private key to solve a challenge that only the matching public key could have created. No password is ever sent over the network.

---

## Key Concepts for Interviews

**Q: What is the difference between symmetric and asymmetric encryption?**  
SSH uses asymmetric (public key) cryptography. Two different keys — one encrypts, the other decrypts. You can't derive the private key from the public key.

**Q: Why is ed25519 preferred over RSA?**  
Ed25519 uses elliptic curve cryptography — shorter keys, faster operations, and equivalent or better security compared to 4096-bit RSA.

**Q: What happens if you lose your private key?**  
You lose access to anything it was used for. Best practice: back up your private key securely, or generate a new pair and update the public key on all servers.

**Q: What is the known_hosts file?**  
Stores fingerprints of servers you've connected to. Prevents connecting to a spoofed server — if the fingerprint changes unexpectedly, SSH warns you of a potential attack.

---

## Where This SSH Key Gets Used Next

| Block | How SSH is used |
|---|---|
| Block 4 | SSH into Vagrant VM (`vagrant ssh` uses SSH under the hood) |
| Block 5 | SSH into Azure VM using this exact key pair (`ssh azureuser@[ip]`) |
| GitHub | All git push/pull operations use this key — no password needed |
| CI/CD (Month 8) | GitHub Actions uses SSH keys to deploy to servers |

---

## Files Created

```
~/.ssh/
├── id_ed25519        ← private key (never share)
├── id_ed25519.pub    ← public key (on GitHub)
└── known_hosts       ← created after first GitHub connection
```

---

## Resources Used

- [GitHub SSH Documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [Ed25519 vs RSA](https://security.stackexchange.com/questions/5096/rsa-vs-dsa-for-ssh-authentication-keys)

---

*Part of the [Cloud Skills Roadmap](https://github.com/Mrushi1903/cloud-skills-learning) — Month 1–2 Foundations*
