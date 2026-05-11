# gh CLI setup

When `gh --version` or `gh auth status` fails, follow the appropriate
case below.

## Case 1: gh CLI not installed

Installation involves elevated privileges, tens of seconds of work, and
network access, so **never start it on your own — get explicit consent
from the user first**.

### Procedure

1. **Ask via `AskUserQuestion` whether to install.**

   ```
   question: gh CLI is not installed. Install it now?
   header: gh install
   options:
     - label: "Install (Recommended)"
       description: "Detect OS and install via winget / brew / apt"
     - label: "Show me the command only"
       description: "Run it yourself in another terminal"
     - label: "Abort"
       description: "Skip the review for now (no gh)"
   ```

2. **If "Install" is chosen, run the OS-specific command.**

   - Detect the OS via env vars / `uname` / platform info.
   - Run the command through the `Bash` / `PowerShell` tool.
   - After it finishes, run `gh --version` to verify, then continue to
     the auth flow (Case 2).

   #### Windows
   ```powershell
   winget install --id GitHub.cli --silent --accept-package-agreements --accept-source-agreements
   ```

   In the same session, `gh` may not be on PATH right after install.
   Either add it temporarily with
   `$env:Path += ";C:\Program Files\GitHub CLI"`, or invoke it directly
   via `& "C:\Program Files\GitHub CLI\gh.exe" --version`.

   #### macOS
   ```bash
   brew install gh
   ```

   If `brew` itself is missing, switch to guiding a brew install first.

   #### Linux (Debian/Ubuntu)
   ```bash
   sudo apt update && sudo apt install -y gh
   ```

   On older distros where `gh` isn't in the default repo, point the user
   to the GitHub apt repo setup
   (<https://github.com/cli/cli/blob/trunk/docs/install_linux.md>).

   #### Linux (Fedora/RHEL)
   ```bash
   sudo dnf install gh
   ```

3. **If "Show me the command only" is chosen, just print the OS-specific
   command and stop.** Ask the user to run it themselves and re-issue
   the same request once done.

4. **If "Abort" is chosen, end the review workflow.** Optionally suggest
   what can still be done without `gh` (e.g. read local `git diff`
   directly).

### Fallbacks when install fails

- Old Windows without `winget` → point to the MSI download URL
  (<https://cli.github.com/>).
- macOS without `brew` → ask "install Brew, or use the official
  installer?"
- Permissions error (`sudo` password required) → ask the user to run it
  in a separate terminal.

## Case 2: Not authenticated

Guide the user to:

```bash
gh auth login
```

This is browser-based and interactive. Claude Code only verifies
completion:

```bash
gh auth status
```

`Logged in to github.com` means success.

## Case 3: Missing token scope

When posting a PR comment returns `HTTP 403`, the token is missing
scopes. Guide the user to:

```bash
gh auth refresh -h github.com -s repo
```

## Trouble: corporate SAML SSO

If the org enforces SAML SSO, also:

```bash
gh auth refresh -h github.com -s repo --hostname github.com
```

After that, the org admin UI may require SSO authorization for the
token. Tell the user: "On your org's GitHub settings, authorize the
personal access token for SSO."
