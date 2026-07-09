# Handoff: publish the portfolio rebuild

Purpose: deploy the rebuilt portfolio (paged layout + live Book Club) to GitHub Pages.
This session that built it was NOT connected to Sheenna's GitHub, so the push has to
happen from an environment that is (a Claude with GitHub access, or Sheenna's terminal).

## Repo
- Local path: `C:\Users\slee1\github\sheenna.github.io-main`
- Assumed remote: `github.com/sheenna/sheenna.github.io` (a GitHub Pages user site).
  CONFIRM this before pushing. If the repo name differs, also update the `REPO`
  constant inside the HTML (it is only a fallback, see notes).

## What to deploy
`preview-pages.html` is the source of truth (the "bookclub/preview-pages" version).
`index.html` should be an exact copy of it. They currently match, but re-copy to be safe.

## Steps (a Claude with GitHub access, or Sheenna in a terminal)

Run from the repo folder. Works in Git Bash or PowerShell.

```bash
# 1. Make index.html match the approved version
#    Git Bash / macOS / Linux:
cp preview-pages.html index.html
#    PowerShell instead:
#    Copy-Item preview-pages.html index.html -Force

# 2. Remove scratch files no longer wanted in the repo
git rm -f preview.html preview-playful.html preview-pages.html style.css _config.yml

# 3. Stage the site + supporting files (.nojekyll already exists)
git add index.html BOOKCLUB-SETUP.md HANDOFF.md .nojekyll

# 4. Commit + push
git commit -m "Rebuild portfolio: paged layout + live Book Club; drop scratch files and Jekyll"
git push
```

Note in step 2: `preview-pages.html` is deleted from the repo AFTER it was copied to
`index.html` in step 1, so the content survives as `index.html`. If you would rather
keep `preview-pages.html` in the repo as a working copy, drop it from the `git rm` line.

Pages redeploys about 1-2 minutes after the push. Check the live URL
(https://sheenna.github.io or the repo's Pages URL).

## Still pending on Supabase (needed for Book Club to be fully live)
Run these two lines once in the Supabase project's SQL editor (project
`qqfeuvnnatzfljyivqfg`). They are also in `BOOKCLUB-SETUP.md`.

```sql
alter table public.book_suggestions add column if not exists comment text;
alter publication supabase_realtime add table public.book_suggestions;
```

- First line: enables the optional "note" field on suggestions. Without it, submitting
  a suggestion that includes a note will fail.
- Second line: makes new suggestions push to OTHER visitors' browsers without a refresh.
  If it says "already a member of publication," that is fine.
- The submitter already sees their own card instantly (optimistic render), so this only
  affects cross-visitor live updates.

## Notes / gotchas
- Font: the site uses a `Geist Pixel` override (`font-family: 'Geist Pixel' !important`
  on everything). This was an intentional experiment by Sheenna. Keep it unless she says
  otherwise. If `Geist Pixel` does not load from Google Fonts, text falls back to
  sans-serif and the intended Doto/JetBrains Mono styling underneath is overridden, so
  double-check it renders as expected on the live site.
- Supabase keys in the HTML are the PUBLIC `sb_publishable_...` key and project URL.
  Safe to commit; row-level security policies are what protect the data. The secret
  `service_role` key must never be placed in this file.
- `.nojekyll` is present so GitHub serves the static `index.html` directly (no Jekyll
  theme processing). That is why `_config.yml` is being removed.
- `REPO` constant in the script (`sheenna/sheenna.github.io`) is only used as a fallback
  that opens a GitHub issue if Supabase is not configured. Since Supabase is live it is
  unused, but keep it correct.

## Quick verification after deploy
1. Site loads at the Pages URL; sidebar clock ticks (12-hour).
2. Arrow keys / on-screen arrows page through Home > Work > Projects > Details > Book Club > Say hi.
3. Book Club: "Invisible Women" shows its real cover; typing a title shows live search
   results with covers; submitting adds a card instantly.
4. Resume button opens `assets/Resume_Sheenna202508.pdf`.
