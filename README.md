# Cognitive Ergonomics for AI

Site at **https://cogergo.protocol-institute.org**: a catalogue of habits people
form when working with AI. Members submit habits, vote "I do this / I don't",
and comment; admins approve submissions and set severity.

A Protocol Institute project ([project page](https://protocol-institute.org/projects/project?slug=cognitive-ergonomics)).
Split out of the `Protocol-Institute/website` repo on 2026-10-11, with its
history, so it can evolve independently of the Institute site.

## Stack

Plain HTML/CSS/JS plus Cloudflare Pages Functions (`functions/api/`). No build
step. Deployed as its own Cloudflare Pages project, `cogergo`.

## What it still shares with protocol-institute.org

These are the dependencies to keep in mind when changing anything here:

- **Database.** The D1 binding `DB` is the Institute's `pi-members` database
  (see `wrangler.toml`). Cogergo owns three tables, `cogergo_items`,
  `cogergo_votes` and `cogergo_comments`, defined in `db/schema.sql` (a copy of
  the website's migration `012_cogergo.sql`). It also reads two Institute
  tables: `members` (name, slug, is_admin) and `auth_pins` (sessions). Never
  write to the Institute tables from here. A schema change to the cogergo
  tables is applied with `wrangler d1 execute pi-members --remote --file ...`
  and affects the live database directly, so back it up first.
- **Login.** There is no separate login. `functions/_shared/auth.js` reads the
  Institute's `pi_session` cookie (scoped to `.protocol-institute.org`) and
  checks it against `auth_pins`. Logging in, profiles and admin rights all live
  on protocol-institute.org (`/members/join`, `/members/profile?slug=`).
- **Markdown.** Item text and comments are markdown, rendered and edited with the
  Institute's `js/markdown.js` and `js/md-editor.js`, loaded from
  protocol-institute.org by absolute URL. The matching `.md` / `.mde` CSS rules
  are copied into `css/style.css`.
- **Branding.** The logo comes from `protocol-institute.org/assets/`.

## Deploying

The Pages project must be connected to this repository (Cloudflare dashboard →
Workers & Pages → cogergo → Settings → Builds) with no build command and output
directory `.`. Until then, deploy by hand from a clean export of a commit, so
that nothing untracked is published:

```bash
tmp=$(mktemp -d) && git archive HEAD | tar -x -C "$tmp" && npx wrangler pages deploy "$tmp" --project-name cogergo --branch main
```
