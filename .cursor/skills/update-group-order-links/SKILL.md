---
name: update-group-order-links
description: >-
  Parses Foodhub automation .eml test report files to extract group order URLs,
  updates index.html in the group-ordering-links repo with the new URLs, then
  commits and pushes to git. Use when the user uploads .eml report files and
  asks to update group order links, update URLs from the test report, or push
  link changes to git.
---

# Update Group Order Links

## Workflow

1. Read each attached `.eml` file
2. Parse it to extract `(envKey → URL)` pairs
3. Update matching rows in `index.html`
4. Commit and push to git

---

## Repo

`/Users/ezhilanb/group-ordering-links/index.html`

---

## Parsing .eml files

`.eml` files use **quoted-printable** encoding — long lines are soft-wrapped with `=` at the end of the line. Decode before parsing:

- Strip soft line-breaks: replace `=\r\n` and `=\n` with nothing (joins the wrapped line)

After decoding, extract pairs using these two patterns (applied sequentially — the env header always appears before its URL within the same test block):

- **Env key**: `Test Environment for\s*:\s*GroupOrder_Creator_(\w+)`
- **URL**: `Share this link with participants:\s*(https://t2s\.app/\w+)`

Pair them in order: the first env key found is paired with the first URL found after it, and so on.

---

## Env key → row title mapping

| Env key (after `GroupOrder_Creator_`) | Row title in index.html |
|---|---|
| `Chrome` | Desktop_Chrome |
| `Safari` | Desktop_Safari |
| `Mobile_Chrome` | Mobile_Chrome |
| `Mobile_Safari` | Mobile_Safari |

For `AOS_X_Y` and `IOS_X_Y` keys (e.g. `AOS_12_1`, `IOS_12_5`), derive the title:
- Replace the first `_` with a space
- Replace remaining `_` with `.`
- Example: `AOS_12_1` → `AOS 12.1`, `IOS_12_5` → `IOS 12.5`

---

## Updating index.html

For each `(envKey, url)` pair extracted:

1. Derive the row title using the mapping above
2. Find the `.scenarios-row` where the `.title` div text matches the row title (trim and compare)
3. In that row, update:
   - `.title` div — set its text content to the derived row title (renames the row to match the email)
   - `.url` div — set its text content to the new URL
   - `.open` button — set `data-url` attribute to the new URL
   - `.copy` button — set `data-url` attribute to the new URL

Use `StrReplace` on the file directly — do not rewrite the entire file.

---

## Committing and pushing

After all rows are updated:

```bash
cd /Users/ezhilanb/group-ordering-links
git add index.html
git commit -m "Update group order links from test report"
git push
```

---

## Notes

- One `.eml` file may contain multiple test environments (e.g. the Android report has AOS 11.28, 11.27, 11.26, 11.24 all in one email). Parse all of them.
- If an extracted env key has no matching row in index.html, skip it and note it to the user.
- If a URL is already the same as what's in the file, skip that row silently.
- After pushing, tell the user how many rows were updated and which ones were skipped.
