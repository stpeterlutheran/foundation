# AGENTS.md — St. Peter Lutheran Church Foundation

Guidance for AI agents (Claude, Codex, Copilot, Cursor, etc.) and humans working in
this repository. Read this file before writing, editing, or reviewing anything here.

For current Foundation institutional facts — including Board membership, contact
information, and the standard board-meeting agenda/minutes format — also read
[`assets/guidance.md`](assets/guidance.md) and treat it as the concise reference
for recurring Foundation information.

---

## Project overview

A single-page marketing and information site for the **St. Peter Lutheran Church
Foundation**. Its job is to explain the Foundation's mission, describe the ways a
person can give (cash, appreciated stock, IRA distributions, bequests, and similar
planned gifts), and publish the Foundation's governing documents.

- **Audience:** congregation members and prospective donors — largely non-technical,
  many on phones or older desktop browsers.
- **Product shape:** a static site. No build step, no framework, no backend, no
  database. Measurement is one cookieless Cloudflare Web Analytics beacon in the
  `<head>` of each published page — there is no analytics pipeline beyond it.
- **Hosting:** GitHub Pages, deployed by `.github/workflows/static.yml` on every
  push to `main`. The workflow stages the public site into `_site`, verifies its
  local HTML links/assets, uploads that directory as the Pages artifact, and
  deploys only after the required checks pass. Pull requests run read-only checks
  and do not receive Pages write credentials.
- **Public address:** the site is served from the Foundation's own apex domain,
  `stpeterlutheranfoundation.org`, set by the `CNAME` file at the repository root.
  `www.stpeterlutheranfoundation.org` redirects to it. See **Custom domain** below
  before changing either.
- **Dependency updates:** `.github/dependabot.yml` batches GitHub Actions bumps
  into one grouped pull request per month; security advisories open immediately
  and ignore that cadence. Its commits land as `ci(deps): ...`.
- **Pages workflow source:** adapted from the Gogorichielab organization Pages
  template at commit `6599d2688f322bb63a01452e032777d7c0bf6eb9`. Repository-specific
  adaptations preserve the `Spellcheck` check and `.spellcheck.yml`, stage this
  repository's explicit public files, validate local links, and keep deployment
  gated to the default branch and the `github-pages` environment.

### Repo layout

```
/
├── index.html                     ← The entire site (inline CSS, hand-authored)
├── 404.html                       ← Not-found page; standalone, links by root-relative path
├── Gift Acceptance Policy.dc.html ← Published policy document page
├── support.js                     ← GENERATED runtime for .dc.html documents — do not edit
├── favicon.ico                    ← Browser tab icon, 16/32/48, cropped from the church logo
├── apple-touch-icon.png           ← 180px home-screen icon, same crop
├── robots.txt                     ← Allows every crawler; points at the sitemap
├── sitemap.xml                    ← The two real pages; update lastmod when they change
├── CNAME                          ← The one hostname Pages answers on
├── assets/church-logo.png         ← Published logo used by both pages
├── assets/guidance.md             ← Foundation reference guidance; not in Pages artifact
├── AGENTS.md, CLAUDE.md, README.md ← Repository guidance; not in Pages artifact
├── .spellcheck.yml                ← PySpelling config for the Spellcheck check
├── .wordlist.txt                  ← Accepted project terms for that check
├── .github/workflows/static.yml   ← GitHub Pages validation/staging/deploy workflow
└── .github/dependabot.yml         ← Monthly grouped GitHub Actions updates
```

The staged Pages artifact currently contains `index.html`, `404.html`,
`Gift Acceptance Policy.dc.html`, `support.js`, `favicon.ico`,
`apple-touch-icon.png`, `robots.txt`, `sitemap.xml`, `assets/church-logo.png`,
`CNAME`, and `uploads/Church Logo.png` — a legacy published path that no page
links, copied from `assets/church-logo.png` so the image lives in the repository
only once. Repository instructions, Foundation reference guidance, CI
configuration, and spellcheck configuration are not copied into the Pages
artifact.

`404.html` is served for any address that does not exist, including deep ones
like `/giving/old/page.html`, so every link and image on it is written
root-relative (`/`, `/#give`) rather than relative. It is deliberately
standalone: it does not load `support.js`, so nothing can keep the error page
itself from rendering. Adding a page to the site means adding it to
`sitemap.xml` as well.

### Checks

`.github/workflows/static.yml` runs three jobs, each gating the next:

1. **Spellcheck** — lints the workflow with `actionlint`, then runs PySpelling over
   every `*.md` and `*.html` file per `.spellcheck.yml`. Fenced blocks, inline
   code, `<script>`, and `<style>` are ignored; all other prose is checked against
   `.wordlist.txt`. New terminology fails the build until it is added there, one
   word per line, so add it in the same change that introduces the word.
2. **Build Pages artifact** — stages the public files into `_site`, asserts each
   one is present, then walks every local `href`/`src` in the staged HTML and
   fails on a missing file or a missing `#fragment`.
3. **Deploy to GitHub Pages** — runs only for a push to the default branch.

Pull requests run jobs 1 and 2; only a push to `main` uploads and deploys.

### Custom domain

The `CNAME` file holds the single hostname Pages answers on. Changing or deleting
it changes every published URL, so treat it as a breaking change under the commit
rules below. The staging step in `.github/workflows/static.yml` already copies
`CNAME` when it exists — adding it needed no workflow edit.

The matching DNS records live at the registrar, outside this repository. For the
apex domain, GitHub documents four `A` records and four `AAAA` records on `@`:

```
A     @    185.199.108.153
A     @    185.199.109.153
A     @    185.199.110.153
A     @    185.199.111.153
AAAA  @    2606:50c0:8000::153
AAAA  @    2606:50c0:8001::153
AAAA  @    2606:50c0:8002::153
AAAA  @    2606:50c0:8003::153
CNAME www  gogorichielab.github.io
```

This domain is on Cloudflare nameservers. Every record above must be set to
**DNS only** (the grey cloud), never proxied. A proxied record blocks the
certificate challenge, and the `Enforce HTTPS` option in **Settings → Pages**
stays unavailable or the certificate fails to issue. That option can also take up
to 24 hours to become selectable after DNS first resolves, which is expected and
not a failure.

Optionally, an organization-level verification `TXT` record
(`_github-pages-challenge-Gogorichielab`) prevents anyone else claiming this
domain on GitHub. It is not required for the site to work.

### Analytics and the privacy policy

Visitor measurement is **Cloudflare Web Analytics**, added as a single deferred
`<script>` in the `<head>` of `index.html`, `404.html`, `privacy.html`, and
`Gift Acceptance Policy.dc.html`. Three things to know before touching it:

- The `data-cf-beacon` token is a **public** beacon identifier, not a secret. It
  is meant to be readable in page source, so committing it is correct and it does
  not belong in a secret store.
- The beacon is used rather than Cloudflare's automatic injection because the DNS
  records for this domain are **DNS only** (see **Custom domain** above).
  Automatic injection only works for proxied records, which this site cannot use.
- Reports live in the Cloudflare dashboard under **Web Analytics**, on the site
  registered for `stpeterlutheranfoundation.org`.

`privacy.html` is the public description of all of this. It is a standalone page
that carries its own styles and root-relative links, the same pattern as
`404.html`, and it is linked from the footer of every published page. **If the
site's data handling changes — a new third-party script, an embedded form, a
different host — `privacy.html` and its effective date must change in the same
commit.** The page is deliberately plain-spoken; it is read by congregation
members, not lawyers.

### Conventions that matter

- `index.html` carries its styling inline. That is intentional for a
  zero-build site — keep edits local and readable rather than extracting a
  design system.
- `support.js` is generated. Its header names both the source
  (`dc-runtime/src/*.ts`) and the rebuild command (`cd dc-runtime && bun run
  build`); that upstream project does not live in this repository. Never
  hand-edit `support.js` — the change belongs upstream.
- Typography is Cormorant Garamond for headings; keep the existing typographic
  scale and colour palette unless a restyle is explicitly requested. A previous
  colour-scheme change was reverted — do not reintroduce one on your own
  initiative.
- Content about gifts, tax treatment, and Foundation policy is **substantive, not
  filler**. Do not reword, summarise, or "improve" it without an explicit request;
  when in doubt, leave the wording alone and ask.
- Only files explicitly staged by `.github/workflows/static.yml` are published to
  GitHub Pages. When adding a new public page or asset, update the staging and
  link-verification steps in the same change. Do not publish repository guidance,
  drafts, personal data, donor information, or other internal material.

### Working agreement

1. Read the surrounding markup before changing it.
2. Make the smallest change that satisfies the request.
3. Verify by opening `index.html` in a browser at both phone and laptop widths.
4. Keep the site static — no dependencies, no bundler, no advertising or
   behavioural tracking scripts. Cloudflare Web Analytics is the single
   permitted measurement script; anything else needs a decision first.

---

## Commit conventions — Conventional Commits

Every commit in this repository **must** follow the
[Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
specification. This applies to agents and humans alike, and to every branch.

### Format

```
<type>(<optional scope>): <description>

<optional body>

<optional footer(s)>
```

- **Type** is required and lowercase.
- **Scope** is optional, lowercase, and names the area touched (see below).
- **Description** is a short imperative summary — "add", not "added"/"adds" — no
  trailing period, and ideally ≤ 72 characters for the whole subject line.
- **Body** explains *why*, wrapped at ~72 columns, separated by a blank line.
- **Footers** carry metadata: `Refs: #12`, `Closes: #12`, `BREAKING CHANGE: ...`.

### Allowed types

| Type | Use for |
|---|---|
| `feat` | A new user-visible section, page, or capability |
| `fix` | Correcting broken markup, links, layout, or wrong information |
| `docs` | README, this file, or other documentation |
| `style` | Formatting/visual changes with no change in content or behaviour |
| `refactor` | Restructuring markup with no visible change |
| `perf` | Image or load-time improvements |
| `test` | Adding or updating tests |
| `build` | Asset pipeline or generated-file updates |
| `ci` | `.github/workflows/**` changes |
| `chore` | Housekeeping that doesn't fit above |
| `revert` | Reverting a previous commit |

### Suggested scopes

`content`, `giving`, `policy`, `layout`, `assets`, `pages`, `deps`

### Breaking changes

Mark either with a `!` after the type/scope or with a `BREAKING CHANGE:` footer
(both is fine). For this site, "breaking" means a changed URL or a removed page.

```
feat(pages)!: move gift acceptance policy to /policies/gift-acceptance.html

BREAKING CHANGE: the previous /Gift Acceptance Policy.dc.html URL now 404s;
add a redirect or update any printed materials that reference it.
```

### Examples

```
feat(giving): add IRA qualified charitable distribution section
fix(layout): stop hero heading overflowing on 320px screens
docs(agents): document conventional commit requirements
style(content): align section spacing with the rest of the page
ci(pages): pin actions/checkout to v4
chore(assets): compress church-logo.png
```

### Rules of thumb

- One logical change per commit. Do not mix content edits with restyling.
- Never commit generated output and source changes in the same commit as
  unrelated content edits.
- Pull request titles follow the same format as commit subjects, so a squash
  merge produces a valid conventional commit.
- If a commit needs "and" in its description, it should probably be two commits.

---

## Skills to use

This project expects agents to work with the following skill packs. Install them
once, then invoke them by name or slash command as the task warrants.

### 1. ponytail — write the least code that works

<https://github.com/DietrichGebert/ponytail>

Claude Code (send as **two separate prompts**):

```
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```

Other agents: copy the matching rules file from the repo — `.cursor/rules/`,
`.windsurf/rules/`, `.clinerules/`, `.github/copilot-instructions.md`,
`.kiro/steering/ponytail.md`, or the repo's `AGENTS.md` for everything else.

Commands: `/ponytail [lite|full|ultra|off]`, `/ponytail-review`,
`/ponytail-audit`, `/ponytail-debt`, `/ponytail-gain`, `/ponytail-help`.

**Use it here:** this is a static, hand-authored site — ponytail's default posture
is exactly right. Run `/ponytail-review` on the diff before opening a PR, and
treat any suggestion to add a framework, build step, or abstraction as a signal
to stop.

### 2. marketing skills — copy, SEO, and campaign work

<https://github.com/coreyhaines31/marketingskills>

```bash
npx skills add coreyhaines31/marketingskills
# or a subset:
npx skills add coreyhaines31/marketingskills --skill copywriting seo-audit
```

Claude Code plugin:

```
/plugin marketplace add coreyhaines31/marketingskills
/plugin install marketing-skills
```

**Use it here:** headline and body copy for giving sections, page structure and
site architecture, SEO audits, schema markup for a local organisation, and
donor-facing email or announcement copy. Anything that changes what the page
*says* to a donor is marketing work — reach for these skills rather than
improvising. Keep the Foundation's voice: warm, plain, never high-pressure.

### 3. business analysis skills — framing before building

<https://github.com/45ck/business-analysis-skills>

```bash
git clone https://github.com/45ck/business-analysis-skills.git
cd business-analysis-skills
bash install.sh          # installs to ~/.claude/skills/ and ~/.agents/skills/
```

Project-level instead: `cp -R .claude .agents /path/to/this-repo/`.

Useful entry points: `/business-problem-framing`, `/stakeholder-analysis`,
`/requirements-elicitation`, `/acceptance-criteria-writer`,
`/requirements-quality-check`, `/assumptions-constraints-log`.

**Use it here:** when a request arrives as a vague wish ("we should make it
easier to give"), frame the problem and write acceptance criteria before
touching `index.html`. Stakeholders include the Foundation board, the church
office, and donors — `/stakeholder-analysis` is worth the five minutes.

### How they fit together

1. **Frame** with business-analysis skills — what problem, whose, done when?
2. **Draft** the donor-facing content with the marketing skills.
3. **Build** under ponytail — the smallest change that ships it.
4. **Commit** using Conventional Commits, one logical change at a time.
