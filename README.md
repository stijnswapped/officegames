# SWAPPED Office Games — Hoop Showdown

A GitHub Pages scoreboard with a three-step Dutch score flow:

1. Select a player.
2. Set the score from 0 to 10.
3. Select a witness and save.

Scores are stored in `scores.json` in the GitHub repository. The dashboard keeps all historical attempts, shows each player's all-time best, and shows each player's best score for the current ISO week (Monday through Sunday).

## Files

- `index.html` — the complete scoreboard and score-entry interface.
- `scores.json` — the shared score history. It starts as an empty array.
- `.nojekyll` — tells GitHub Pages to serve the files directly, when present.

## Important limitation

GitHub Pages is a static website. Browser code cannot write safely to a repository without authentication.

This project therefore uses a **fine-grained GitHub personal access token entered once on a trusted office browser**. The token is saved in that browser's `localStorage`; it is never written into `index.html` or committed to GitHub.

This mode is suitable for one trusted office kiosk or computer. Do not paste the token into the source code. Anyone with physical access to that browser and its developer tools may be able to retrieve the stored token.

Other devices can read the public scoreboard. Without their own configured token, their new scores are saved only in that device's browser until a token is configured.

For fully anonymous submissions from every phone, a small backend or database service is still required.

## 1. Create and publish the repository

1. Create a GitHub repository, for example `hoop-showdown`.
2. Upload `index.html`, `scores.json`, and `.nojekyll` to the repository root.
3. Commit them to `main`.
4. Open **Settings → Pages**.
5. Select **Deploy from a branch**, choose `main` and `/ (root)`, then save.
6. Open the GitHub Pages URL GitHub provides.

On a standard project Pages URL such as `https://YOUR-NAME.github.io/hoop-showdown/`, the page automatically detects the GitHub owner and repository name.

For a custom domain or local testing, enter the owner and repository manually through the gear button.

## 2. Create a restricted GitHub token

Create a **fine-grained personal access token** in GitHub with the smallest possible permissions:

- Repository access: **Only select repositories**
- Selected repository: only the Hoop Showdown repository
- Repository permissions → **Contents: Read and write**
- Give the token a reasonable expiration date
- Do not add other permissions

GitHub documentation:

- Fine-grained token authentication: https://docs.github.com/en/rest/authentication/authenticating-to-the-rest-api
- Repository contents endpoint: https://docs.github.com/en/rest/repos/contents

## 3. Connect the trusted office browser

1. Open the live GitHub Pages scoreboard on the office computer.
2. Press the **⚙** button next to the plus button.
3. Confirm:
   - GitHub owner
   - Repository name
   - Branch: `main`
   - Score file: `scores.json`
4. Paste the fine-grained token.
5. Press **Opslaan & testen**.

The token stays in that browser. New attempts are first stored locally, then committed into `scores.json` through GitHub's Contents API.

## How scores are saved

Every attempt contains:

- Unique ID
- Timestamp
- Date
- ISO week
- Week start
- Player name
- Score
- Witness
- Approval status

Each save creates a commit in the repository. The code reads the latest file SHA before writing and retries when another submission changed the file at the same moment.

The leaderboard groups all attempts by player:

- **All-time:** each player's best historical score
- **Weekly:** each player's best score between Monday 00:00 and Sunday 23:59 of the current week

## Adding people

Edit the player list near the bottom of `index.html`:

```js
PLAYERS: ["Stijn", "Cas", "Dennis", "Rob", "Marco", "Esther", "Gast"]
```

Add another quoted name to the array and commit the change.

## Local and offline behavior

A new score is written to browser storage immediately. If GitHub is temporarily unavailable or the token is missing, it stays in a local synchronization queue.

When the same unique score ID appears in `scores.json`, the local queued copy is removed automatically.

## Removing or replacing the token

Press **⚙**, then choose **Token verwijderen**. You can paste a replacement token afterward.

Revoking the token in GitHub immediately prevents further repository writes from browsers that stored it.

## Optional: avoid a Pages deployment for every score

By default, `scores.json` is written to `main`, which may trigger a GitHub Pages rebuild after every score.

To separate data from the site:

1. Create a branch such as `scores-data` containing `scores.json`.
2. Keep GitHub Pages publishing from `main`.
3. Set **Branch** in the ⚙ settings to `scores-data`.

The dashboard will then read and write the JSON file on that branch without changing the published site branch.
