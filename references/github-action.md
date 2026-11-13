# Optional: open an issue for every new message

Install this only if the team wants a notification outside Git. Save it in the
repository as `.github/workflows/team-messages.yml`. It runs on GitHub with the
repository's own token, reads the new message files and opens one issue per
message, assigned to the recipient. It never changes code.

This workflow is optional and is not part of what the scripts do. Add it only
when the user asks for it: show them the file first, and commit and push it
only with their go-ahead. It runs on GitHub Actions in the user's own GitHub
account, with the two permissions shown below (read the repository, write
issues), and it counts toward that account's Actions usage. Each issue title
carries the recipient's name and the message subject. To turn it off, delete
the workflow file; issues it already opened can be closed by hand.

Add a `github` map to `coordination/team.json` so agent names resolve to GitHub
accounts:

```json
{ "agents": ["sam", "priya"], "github": { "sam": "sam-gh-login", "priya": "priya-gh-login" } }
```

```yaml
name: Team messages
on:
  push:
    paths: ["coordination/messages/**"]
permissions:
  contents: read
  issues: write
jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2
      - name: Open an issue for each new message
        env:
          GH_TOKEN: ${{ github.token }}
          REPO: ${{ github.repository }}
        run: |
          git diff --name-only --diff-filter=A HEAD~1 HEAD -- coordination/messages | while read -r file; do
            [ -f "$file" ] || continue
            to=$(grep -m1 '^TO:' "$file" | cut -d: -f2- | xargs)
            subject=$(grep -m1 '^SUBJECT:' "$file" | cut -d: -f2- | xargs)
            who=$(jq -r --arg a "$to" '.github[$a] // empty' coordination/team.json)
            name=$(basename "$file")
            if gh issue list --repo "$REPO" --state all --search "$name in:body" --json number --jq '.[0].number' | grep -q .; then continue; fi
            gh issue create --repo "$REPO" --title "Message for $to: $subject" \
              --body "New message: \`$file\`. Read it, then claim it with team.py." ${who:+--assignee "$who"}
          done
```

Notes: a message whose recipient has no GitHub login in the map still gets an
issue, unassigned. The search on the file name prevents duplicates when a
commit is pushed twice.
The workflow compares only the last commit of a push with the one before it,
so push after each `send`, as the skill's steps do: a message added in an
earlier commit of a larger push gets no issue.
