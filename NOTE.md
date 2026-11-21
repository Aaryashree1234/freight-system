
## 2026-11-21: Fixing the GitHub push error (403 / wrong account)

### Problem
I set up the project folder, ran `git init`, made my first commit, and
connected it to my GitHub repo (freight-system). When I ran
`git push -u origin main`, it failed.

### Error message
remote: Permission to Aaryashree1234/freight-system.git denied to aaryashree214.
fatal: unable to access '...': The requested URL returned error: 403

The repo belongs to my account Aaryashree1234, but my Mac was sending
a different account, aaryashree214. 403 means "I know who you are,
but you're not allowed to do this here."

### What I tried that did not work
- Deleted the github.com entry in Keychain Access. It kept showing up
  (its Date Modified was still old), so the delete had not gone through.
- Ran a terminal command to erase the saved GitHub login from Git's
  keychain helper.
- Ran `git -c credential.helper= push -u origin main` to push without
  saved logins.
- Changed the remote URL to include my username.
- Signed VS Code out of the other account.
Same 403 every time.

### Root cause (most likely)
My Mac, probably through VS Code's own GitHub sign-in or the saved
Keychain login, kept using the old account (aaryashree214) when I
pushed over HTTPS. I never proved exactly which one it was, but
switching away from HTTPS login fixed it.

### What fixed it
1. Switched the remote from HTTPS to SSH:
   `git remote set-url origin git@github.com:Aaryashree1234/freight-system.git`
2. `ssh -T git@github.com` first said "Permission denied (publickey)".
   `ssh -vT git@github.com` showed "agent contains no identities",
   which meant I had no key yet.
3. Created a key with `ssh-keygen -t ed25519 -C "my GitHub email"` and
   pressed Enter to accept the default location and no passphrase.
4. Copied the right file with `pbcopy < ~/.ssh/id_ed25519.pub`, pasted
   it at GitHub > Settings > SSH and GPG keys > New SSH key, and it
   was accepted.
5. `ssh -T git@github.com` and `git push -u origin main` worked.

### What I learned
- SSH keys come in pairs. The public key (.pub, starts with ssh-ed25519)
  goes to GitHub. The private key never leaves my Mac.
- A fingerprint (SHA256:...) is not the key.
- `cd` moves into a folder. `mkdir` creates one. `~` means my home
  folder, and `/` alone means the top of the file system.
- Read the error message closely. The 403 line named the wrong
  account, which was the biggest clue.
- Changing one thing at a time and checking the result helps find
  the real cause.