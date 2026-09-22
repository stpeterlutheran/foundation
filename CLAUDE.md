# CLAUDE.md — St. Peter Lutheran Church Foundation

Instructions for Claude Code working in this repository.

> **Read [`AGENTS.md`](AGENTS.md) first.** It is the authoritative project brief and
> is kept tool-agnostic. This file adds the Claude-specific bits and repeats only
> the rules that are easy to get wrong.
>
> For current Foundation institutional facts — including Board membership,
> contact information, and standard board-meeting agenda/minutes guidance — read
> [`assets/guidance.md`](assets/guidance.md) before drafting Foundation records or
> donor-facing materials that depend on those facts.

---

## Project overview

Static, single-page site for the St. Peter Lutheran Church Foundation. It explains
the Foundation's mission, the ways a person can give (cash, appreciated stock, IRA
distributions, bequests and other planned gifts), and publishes its governing
documents. Audience is the congregation and prospective donors — mostly
non-technical, many on phones.

- **Stack:** hand-authored HTML with inline CSS. No framework, no build step, no
  backend, no dependencies.
- **Entry point:** `index.html` is the whole site.
- **Also published:** `privacy.html` (standalone, self-styled like `404.html`),
  `Gift Acceptance Policy.dc.html` (rendered by the generated
  `support.js` runtime — **never hand-edit `support.js`**), the church logo, and
  the `favicon.ico` / `apple-touch-icon.png` pair.
- **Public address:** `stpeterlutheranfoundation.org`, set by the `CNAME` file;
  `www.` redirects to it. Changing `CNAME` changes every published URL, so it is
  a breaking change.
- **Deploy:** GitHub Pages via `.github/workflows/static.yml` on push to `main`.
  Only the files that workflow explicitly stages are published — dropping a file
  at the repo root does **not** put it on the site. Add new public files to the
  staging and link-verification steps in the same change.

### Non-negotiables

- Keep it static. No bundler, no npm, no advertising or behavioural tracking
  scripts. The one measurement script allowed is Cloudflare Web Analytics,
  which is cookieless and does not fingerprint or profile visitors; it is
  described on `privacy.html`. Adding any other third-party script is a
  change to ask about first.
- Do not restyle the site on your own initiative — a prior colour-scheme change
  was reverted. Preserve the Cormorant Garamond typography and existing palette
  unless a restyle is the explicit request.
- Do not reword giving, tax, or policy content without being asked. It is
  substantive text, not placeholder.
- Nothing donor-identifying or draft-quality gets committed. The repository is
  public, so committing a file discloses it whether or not Pages serves it.
- CI runs a spellcheck over every `*.md` and `*.html` file. New prose wording
  fails the build until the term is added to `.wordlist.txt` in the same change;
  code spans, fenced blocks, `<script>` and `<style>` are ignored.
- Verify changes by opening the page at phone and laptop widths.

---

## Commit conventions

**Every commit must follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).**

```
<type>(<optional scope>): <imperative description>

<optional body explaining why>

<optional footers — Refs: #12, BREAKING CHANGE: ...>
```

- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`,
  `ci`, `chore`, `revert`.
- Scopes used here: `content`, `giving`, `policy`, `layout`, `assets`, `pages`,
  `deps`.
- Imperative mood, no trailing period, subject ≤ 72 chars.
- Breaking = a changed or removed URL. Use `type(scope)!:` and/or a
  `BREAKING CHANGE:` footer.
- One logical change per commit; PR titles use the same format so squash merges
  stay valid.

```
feat(giving): add IRA qualified charitable distribution section
fix(layout): stop hero heading overflowing on 320px screens
ci(pages): pin actions/checkout to v4
```

Full type table and more examples: [`AGENTS.md`](AGENTS.md#commit-conventions--conventional-commits).

---

## Skills to use

| Skill pack | Install (Claude Code) | Use it for |
|---|---|---|
| [ponytail](https://github.com/DietrichGebert/ponytail) | `/plugin marketplace add DietrichGebert/ponytail` then `/plugin install ponytail@ponytail` (two separate prompts) | Default posture for all code work. `/ponytail-review` the diff before every PR. |
| [marketing skills](https://github.com/coreyhaines31/marketingskills) | `/plugin marketplace add coreyhaines31/marketingskills` then `/plugin install marketing-skills` — or `npx skills add coreyhaines31/marketingskills` | Donor-facing copy, page structure, SEO audits, schema markup, announcement emails. |
| [business analysis skills](https://github.com/45ck/business-analysis-skills) | `git clone https://github.com/45ck/business-analysis-skills.git && cd business-analysis-skills && bash install.sh` | Framing vague requests, stakeholder analysis, requirements and acceptance criteria before you touch HTML. |

Order of operations: **frame** (business analysis) → **draft** (marketing) →
**build** (ponytail) → **commit** (Conventional Commits).

Handy commands: `/ponytail-review`, `/ponytail-audit`, `/business-problem-framing`,
`/stakeholder-analysis`, `/acceptance-criteria-writer`, `/requirements-quality-check`.

Details, non-Claude install paths, and per-skill guidance: [`AGENTS.md`](AGENTS.md#skills-to-use).
