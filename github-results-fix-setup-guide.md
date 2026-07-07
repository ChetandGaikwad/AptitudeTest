# Setup Guide: Fixing "No Results in Admin Panel" + Publishing to GitHub

## Why results weren't showing up

Your app was storing everything (questions, results) in the browser's own local storage. That storage is **private to one browser on one device** — it isn't shared between the candidate's laptop and your laptop, even on the same website. So when a candidate completed the test on their machine, the result was saved *only there*, and your admin panel — running in a different browser — had nothing to show. Not a bug in the logic, just a hard limit of "no backend at all."

## What changed in this version

Results are now sent to **GitHub Issues** in your own repository the moment a candidate submits — using GitHub's own API, which is already part of the hosting you're using, so no new outside service was added. Your admin panel now fetches that same list of issues to build the Results table, from any device. Deleting a result closes the underlying issue; "Reset All" closes all of them.

If a candidate's browser can't reach GitHub when they submit (offline, or you haven't configured it yet), the result is saved locally as a safety net and a banner in the Results tab lets you retry syncing it later — nothing is silently lost, but it also won't show up centrally until that retry succeeds.

## Step 1 — Create a scoped GitHub token (5 minutes)

1. On GitHub, click your profile photo (top right) → **Settings**.
2. Scroll to the bottom of the left sidebar → **Developer settings**.
3. **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.
4. Give it a name like `aptitude-test-results`.
5. **Resource owner**: your account.
6. **Repository access**: select **Only select repositories** → choose your `aptitude-test` repo specifically. (Don't grant access to all repos — keep the blast radius small.)
7. Under **Permissions → Repository permissions**, find **Issues** and set it to **Read and write**. Leave every other permission at "No access."
8. Click **Generate token** → **copy the token immediately** (starts with `github_pat_...`) — GitHub only shows it once.

## Step 2 — Edit the 3 config lines in the file

Open `index.html` in any text editor (Notepad, VS Code, etc.) and find these lines near the top of the `<script>` section:

```javascript
const GITHUB_OWNER = 'YOUR_GITHUB_USERNAME';   // <-- EDIT ME
const GITHUB_REPO  = 'YOUR_REPO_NAME';          // <-- EDIT ME
const GITHUB_TOKEN = 'YOUR_FINE_GRAINED_TOKEN'; // <-- EDIT ME
```

Replace each placeholder:
- `GITHUB_OWNER` — your GitHub username
- `GITHUB_REPO` — the exact repo name (e.g. `aptitude-test`)
- `GITHUB_TOKEN` — the token you copied in Step 1

Save the file.

## Step 3 — Upload to GitHub

1. Go to your repo on GitHub → **Add file → Upload files**.
2. Upload the edited `index.html` (it will overwrite the existing one).
3. **Commit changes.**
4. Give it a minute — your existing GitHub Pages link will now serve the updated file automatically. No need to redo the Pages setup.

## Step 4 — Test it

1. Open your live GitHub Pages link.
2. Go through the candidate flow once yourself (any browser, even a different device or your phone).
3. Open Admin → Results (on your own laptop, a different device from the one you just tested on) → click **Refresh** — your test result should appear.

## One thing to be fully aware of

Because this repository is **public** (required for free GitHub Pages), the token is technically visible to anyone who looks at your repo's file source. This is why Step 1 matters so much: by scoping the token to **only this one repository** and **only Issues: Read and write**, the worst thing someone could do with it if they found it is create junk issues in this one repo — they cannot touch your other repositories, your account, or any other data. If that residual risk isn't acceptable for your situation, regenerate the token periodically (Step 1 again, then Step 2–3 with the new value) and delete the old one from GitHub's token settings — takes under a minute.

This does **not** change the separate, still-open item from before: the correct-answer key is still visible in the page's JavaScript to anyone who opens dev tools during the test. That's the specific problem the Power Automate/SharePoint path solves — this update only fixes the "results aren't reaching the admin panel" problem.
