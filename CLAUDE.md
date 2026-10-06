# PHP 2026-09-24 Security Releases Tracking — Claude Code Context

This repository contains a living tracking document for the **PHP
security releases of 2026-09-24**: **8.5.11, 8.4.26, 8.3.35, and
8.2.34**, one per supported branch. Together they close **twelve
GitHub security advisories**, eleven with a CVE ID, spread across
SOAP, PHP-FPM, OpenSSL, the HTTP stream wrapper, stream filters, Phar,
mysqlnd, the filter extension, and (Windows only) path handling.

- **Seed:** <https://www.php.net/ChangeLog-8.php> (sections `#8.5.11`,
  `#8.4.26`, `#8.3.35`, `#8.2.34`).
- **Advisories:** <https://github.com/php/php-src/security/advisories>.
  The pages are JS-rendered; read them through the REST API instead:
  `https://api.github.com/repos/php/php-src/security-advisories/<GHSA-id>`
  (per advisory) or `…/security-advisories?per_page=40&sort=published&direction=desc`.
- **CVE records:** reserved but not yet published in the CVE List when
  the tracker was seeded (the CVE Services API 404s), so the assigning
  CNA isn't public yet — some neighbouring IDs are the PHP Group's.
  The GHSA API is the fact source until the records are published.
- **Highest severity:** CVE-2026-91765 (SOAP server recursion, CVSS 3.1
  7.5 high).

The twelve advisories, and the ids the lookup helpers search for, are
listed once in `scripts/tracked-ids` and once in the tracker's
*Advisories* table; keep the two in step.

Two advisories are special, and the tracker says so:

- **GHSA-ch8v-r6jh-4vvr** (no CVE, `FILTER_SANITIZE_ENCODED` and
  `0xFF`) is fixed in **8.2.34 and 8.3.35 only**. 8.4 and 8.5 fixed it
  a release earlier, in 8.4.25 and 8.5.10, so the 8.4.26 and 8.5.11
  changelogs don't list it.
- **CVE-2026-17545 / GHSA-9f67-6fw4-hpfp** affects **PHP on Windows
  only**. None of the tracked distributions builds for Windows, so it
  never gates a row's verdict (the user decided this at seed). Debian's
  tracker marks it `not-affected` for the same reason.

8.4.26 and 8.5.11 also fix memory-safety bugs that got no advisory
(DOM/Intl use-after-frees, 8.4's `hash_pbkdf2()` overflow). They are
mentioned once in the tracker's prose and are **not tracked**: never
make a verdict depend on them.

The rendered site is published at <https://kimmo.cloud/php-20260924/>.

The Unbound tracker <https://kimmo.cloud/CVE-2026-81642/>
(`~/src/CVE-2026-81642`) is the live multi-CVE package-tracker sibling
this one was modelled on; its single-branch table does not carry over
unchanged (see below).

## Your task

Keep `site/content/_index.md` (the canonical tracker) up to date as
fixes land in the tracked distros.

**In an interactive session**, rebuild with `make build` and publish
with `make dist` — publishing is externally visible, so ask first.
**In the scheduled headless run**, do neither: edit and commit onto the
`auto-update` branch and stop. `make` is deliberately absent from that
run's allowlist, and the publish happens later, from a human-reviewed
merge.

A scheduled background agent runs against this repo to refresh the
tracker on its own. If you find the file has been edited since you
last looked, that's likely why — re-read before assuming stale state.

To retire (archive) this tracker — when every tracked row has shipped
a fix, or the releases are otherwise no longer worth active tracking —
follow `~/src/cve-tracker-template/LIFECYCLE.md` § "Retiring a tracker".

## Repo layout

```
.
├── site/                                        # Hugo project
│   ├── content/_index.md                        # the tracker — single source of truth
│   ├── hugo.toml                                # config (subpath baseURL — don't break)
│   ├── assets/css/extended/custom.css           # CSS overrides (PaperMod extension point)
│   ├── assets/php-20260924-tracker.svg          # social-banner source (rasterised by `make banner`)
│   ├── static/php-20260924-tracker.png          # rendered OpenGraph banner (committed)
│   ├── layouts/partials/post_meta.html          # overrides PaperMod: adds labels + lastmod
│   ├── layouts/partials/extend_footer.html      # client-side table grouping + status tagging + code-copy
│   ├── layouts/shortcodes/details.html          # collapsible-section shortcode (verification log)
│   └── go.mod, go.sum                           # Hugo Modules — pulls PaperMod theme
├── scripts/                                     # auto-update agent: prompt + driver + lookups
│   ├── auto-update                              # wrapper invoked by the systemd timer
│   ├── auto-update-prompt.txt                   # prompt fed to headless Claude
│   ├── debian-versions                          # Debian: branch per suite + PHP release from the changelog
│   ├── nixpkgs-versions                         # nixpkgs: php82..php85 per channel/branch
│   ├── pkgsrc-versions                          # pkgsrc: lang/php82..85 identifiers per branch
│   ├── tracked-ids                              # the advisory/CVE ids the helpers search for
│   ├── nixos-first-shipped                      # dates a channel's flip (see below)
│   ├── alas-cve                                 # kernel-tracker helper; unused here, carried from the template
│   └── check-shape                              # prose and log size check (`make check`)
├── tests/                                       # helper tests (`make check`)
├── systemd/                                     # user-level timer + service units
│   ├── php-20260924-tracker-update.service      # runs scripts/auto-update
│   └── php-20260924-tracker-update.timer        # twice daily
├── flake.nix, .envrc                            # Nix dev shell: hugo + go + git + resvg + curl
├── Makefile                                     # `make build`, `make dist`, `make banner`, `make check`
├── LICENSE                                      # CC BY 4.0
├── README.md                                    # user-facing project README
├── WEBSITE.md                                   # publication plan / decisions log
└── CLAUDE.md                                    # this file
```

## The tracker file (`site/content/_index.md`) — important constraints

- It has Hugo front-matter with these required fields: `title`,
  `description`, `layout: "single"`, `date` (published), `lastmod` (last
  updated). Keep all five; the rendering depends on them.
- There is no H1 in the body — Hugo emits the title from front-matter
  via PaperMod's single-post layout. Don't add an H1 back.
- The TOC is generated by PaperMod's auto-TOC (`ShowToc = true` +
  `UseHugoToc = true` in `hugo.toml`). Don't add a manual TOC.
- The "Last updated" date lives in the `lastmod` front-matter field.
  Bump it on every content edit; leave it alone on a no-op run.
- The `cover:` front-matter block points at `php-20260924-tracker.png`
  and feeds the OpenGraph / Twitter / RSS social image. Keep it;
  `hiddenInSingle: true` keeps the image out of the rendered page body.
- **The tracker is for its human readers, not for you.** Write every
  section for the operator deciding whether their PHP is exposed —
  which advisories matter to them, per-distro status, what to do. Keep
  out anything that only explains how the tracker is *built*:
  row-inclusion policy, verdict-axis or column mechanics, and tracking
  methodology. Those live here in `CLAUDE.md`. State the
  reader-relevant *fact*, never the policy behind it.
- **Per-distro `###` prose: shape and budget.** Open a section with one
  or two sentences; put anything enumerable (advisories, kernel series,
  streams, flake refs) in a bullet list; give every other idea its own
  short paragraph — none longer than about eight rendered lines. Before
  adding a sentence, check that it says something no table cell says:
  never restate a row's version, date, verdict, or *Status* note.
  Provenance (commit SHAs, "confirmed via …", mirror lag) belongs in the
  verification log. **When a row flips, rewrite the section to describe
  the current state** — never append "now fixed" / "has now" sentences to
  the old text. The section should read as if written fresh today.
- **Verification log: one fact per sub-bullet.** Log entries are one
  top-level bullet per source, with a terse bold lead naming the source
  or method — at most four lines, no facts — followed by one fact per
  nested sub-bullet of at most six lines. When a run learns something
  new, add or edit a sub-bullet; never extend the lead or append a
  clause to a neighbouring sub-bullet. Name each pairing explicitly
  rather than relying on the order of an `A / B` list.
  `scripts/check-shape` (run by `make check`) enforces these limits and
  the eight-line limit on per-distro paragraphs and bullets. The
  auto-update wrapper runs it too, on every run in which the worktree's
  page differs from `origin/main`, and fails the unit when the page is
  over budget.
- **One command per fenced code block, no inline comments.** Each `sh`
  fence holds a single command with nothing after it on the line, so
  PaperMod's copy button yields something runnable. Clarifying notes go
  in the prose *before* the block.

## Update workflow

1. Edit `site/content/_index.md` (or any file under `site/`).
2. Optional: `cd site && hugo server` for a local live preview at
   <http://localhost:1313/php-20260924/>.
3. `make build` — emits to `site/public/` (gitignored).
4. `make dist` — runs `make build`, then rsyncs `site/public/` with
   `--delete` to `haig:/php-20260924/`, which the server's restricted
   rsync resolves to
   `~/.www/sites/kimmo.cloud/htdocs/php-20260924/`. The short form in
   the `Makefile` is not a typo; don't "fix" it to the full path.
   Publishing is externally visible — in an interactive session ask
   before running it, and never run it from the headless job.

`make check` runs the helper tests under `tests/`; run it after any
change to `scripts/`.

## Social banner

The OpenGraph / social-preview image is **generated**, not hand-edited:

- Source: `site/assets/php-20260924-tracker.svg` (a 1200×630 SVG — the
  standard OG size). Edit this.
- Output: `site/static/php-20260924-tracker.png`, produced by
  `make banner` (which runs `resvg`). Hugo copies it from `static/`
  into `public/` and PaperMod references it via the `cover:`
  front-matter.

The PNG is committed so `make build` / `make dist` never need a
rasteriser. Run `make banner` only after editing the SVG, then commit
the regenerated PNG.

`make banner` needs `resvg` and the **Roboto** and **Liberation Mono**
fonts. The Nix flake provides `resvg`; on Debian, `apt install
fonts-roboto fonts-liberation` provides the fonts. resvg's `Fallback
from Roboto` warnings mean the SVG asked for something the Roboto faces
can't supply — only weights Roboto ships (400/500/700/900, *not* 800),
and only glyphs it covers (plain Latin text; no arrows or other
symbols — draw those as SVG `<path>`/`<polygon>`).

## One table with a PHP column — the verdict per row

PHP ships four supported branches at once, each with its own fixed
release, and distributions differ in which branches they package:
Debian one per release, NixOS and pkgsrc all four side by side. The
tracker carries **one** distribution table — `Distribution | Release |
PHP | Package | Fixed since | Status`, with `{.distros}` on the line
immediately after it — with **one row per (release, PHP branch)
pair**. This departs from the template's one-table-per-track rule
(`~/src/cve-tracker-template/DESIGN.md`); the user chose it at seed so
each row shows the PHP release a distribution packages, and the
decision is recorded in `WEBSITE.md`. Do not split it per branch or per
ecosystem, and do not add a column per advisory.

- **PHP** is the upstream PHP release the package builds — `8.4.24` —
  never the distribution's package version. Where a row's package
  carries backports on top of that release, say so in the cell
  (`8.2.33 + backports`) and in the prose.
- **Package** is what the distribution calls it: `php8.4` plus the
  Debian version, `php84-8.4.26nb1` (what `pkg_info -e` prints), the
  nixpkgs attribute `php84`. The branch its release selects by default
  (the unversioned `php`) is marked `(default)` and comes first within
  its release.
- **Status** is judged against the row's **own branch's** 2026-09-24
  release: 8.2.34, 8.3.35, 8.4.26, or 8.5.11. The note names it
  (`needs 8.4.26`), since that threshold varies by row and is the one
  thing the columns don't show.

The verdict keys on every applicable advisory of that release at once:

- `:white_check_mark: Fixed` — the package closes **all** of them:
  PHP at or above the branch's fixed release, or backports of every
  applicable fix. For 8.2 and 8.3 rows that is eleven advisories
  (ten CVEs plus GHSA-ch8v-r6jh-4vvr); for 8.4 and 8.5 rows ten CVEs.
  CVE-2026-17545 never counts (Windows only). Set *Fixed since* from
  the source that shipped the last of them.
- `:warning: Partial — <ids> open` — the package closes some but not
  all. Name every advisory still open in the note. *Fixed since* stays
  `—` until the row is fully fixed; the partial fix's date and what it
  closed go in the `###` prose and the verification log. Expect this
  from a Debian LTS upload that backports a subset, or from a nixpkgs
  or pkgsrc patch for one CVE.
- `:x: Vulnerable` — none closed, or not verified to be.
- `:grey_question: Unverified` — not yet verified.

A version compare **is** meaningful here, unlike most package
trackers: every tracked distribution normally takes PHP's upstream
point releases rather than backporting, so a row at or past its
branch's fixed release is fixed. The Debian helper verifies the PHP
release from the changelog, not from the version string; a Debian
version whose upstream part is still below the fixed release can
nonetheless be fixed by backports, which the changelog and the
security tracker's per-CVE pages reveal.

### When a distribution changes branch

Distributions move to newer PHP branches over the tracker's life —
sid's `php-defaults` may switch to 8.5, a quarterly pkgsrc branch or a
nixpkgs revision may drop 8.2 once it reaches end of life. The helpers
report every branch they find and which one is the default, so watch
for:

- **A new branch in a tracked release** (e.g. `php8.5` migrating into
  sid): add a row for it within that release, judged against its own
  branch's fixed release, with a verification-log bullet saying when
  it appeared.
- **The default moving** (`php-defaults`, nixpkgs' `php = …`,
  pkgsrc's `PHP_VERSION_DEFAULT`): move the `(default)` marker and
  reorder the release's rows so the default comes first. The rows'
  verdicts don't change.
- **A branch leaving a release** (the helper prints `absent`, or a
  Debian suite stops listing the source): keep the row, frozen at its
  final verdict, and say in the prose that the release no longer
  carries that branch. Remove it only if it was never fixed *and* the
  user agrees.

### Row order inside the table

Rows sharing a **Distribution** value must be **contiguous**.
`layouts/partials/extend_footer.html` renders each consecutive run as
one group heading row, so a row inserted between two `nixpkgs` rows
renders the label twice and breaks the grouping.

Order within a group: Debian sid → forky → trixie → bookworm; pkgsrc
`pkgsrc-current` first, then the quarterly branches **descending**;
NixOS channels paired with their `-small` gate (`nixos-unstable`,
`nixos-unstable-small`, `nixos-26.05`, `nixos-26.05-small`); nixpkgs
`master`, then `release-26.05`, then `nixpkgs-unstable`. Within each
release or channel: the default branch first, then the other branches
**descending** (8.5, 8.3, 8.2 under a default of 8.4). The NixOS
pairs are a deliberate exception to date order: the `-small` gate
usually leads its sibling by a day or two. Don't "fix" that.

## Conventions for status entries

A note after the em dash must add information the row's columns don't:
don't restate the verdict (`Vulnerable — no fix yet`) or the version
already shown. The branch threshold (`needs 8.4.26`) is the standard
note on a vulnerable row; add an awaited advisory (`DLA pending`) or
a posture marker (`LTS`) only when it adds something. On a Partial row
the note lists the still-open advisories.

- The distribution table is the single source for its rows' columns
  (status, versions, fixed-since). Per-distro prose is for notes that
  don't fit a table — don't restate a row's columns there, and don't
  add a parallel table that duplicates it.
- A value column holds the value, not a verdict word — don't write
  "fixed" / "vulnerable" / "patched" in a version cell.

Record the fixed package version, the date it became available, and
the source of confirmation. When you re-verify entries, update the
`## Verification log` section in place — edit the relevant
`#### Upstream` / `#### Distributions` bullet rather than appending a
new line per re-check. The log carries no dates of its own — the
`lastmod` front-matter is the document's only recency marker; don't
write verification dates inline. Method/source attribution without a
date is fine (e.g. `(via scripts/debian-versions)`).

The log records **facts**, not run outcomes. Never write "no change",
"unchanged from prior run", or similar prose anywhere in the tracker —
an entry that is still accurate reports that by staying untouched.

Once a row reads **fixed**, it is sticky: a later point release
(`8.4.26` → `8.4.27`) or a packaging rebuild (`-1` → `-2`, a new
`nbN`) that doesn't flip the verdict is not a tracker update. Leave the
cells at the version that introduced the fix and don't bump `lastmod`.
A version change earns an update only while the row is still
vulnerable or partial.

## "Fixed since" is derived, never "today"

A row's *Fixed since* is the date the fix **actually became available**
in that release, derived from the source that shipped it. It is not the
date the agent noticed, and not the date of the run.

| Ecosystem | Derive from |
|---|---|
| Debian | sid: the changelog entry of the fixing upload, or the fixed version's `first_seen` in <https://snapshot.debian.org/>; forky: the testing-migration date (see the Debian recipe); stable: the DSA/DLA or point-release date |
| pkgsrc | the commit date of the `lang/phpXY: update to …` commit (or backport patch) on that branch — see the pkgsrc recipe |
| NixOS / nixpkgs | `scripts/nixos-first-shipped` for channels; the bump commit's date for the `master` / `release-26.05` branch rows |

Once set, the date is **sticky**. Use `—` while a row is vulnerable,
partial, or unverified. If a date genuinely cannot be derived, record
the first-observation date and say so in the verification log.

## Dating a NixOS channel flip — `scripts/nixos-first-shipped`

A channel's git-revision pointer only says where the channel is *now*.
`nixos-first-shipped` lists the channel's published releases, finds
the earliest whose revision contains a given nixpkgs commit, and
reports that release's publication date.

All four PHP branches live in one file, so one commit usually bumps
several of them. **Pick the commit by subject, not by recency**:

```sh
git -C ~/src/nixos/nixpkgs log --format='%H %cd %s' --date=short origin/master -- pkgs/development/interpreters/php/default.nix
```

The wanted commit is the one whose subject names the bump for the
row's branch (`php84: 8.4.25 -> 8.4.26`, or a combined
`php: 8.2.34, 8.3.35, 8.4.26, 8.5.11`); each PHP branch can have its
own commit, and each nixpkgs branch has its own SHA. Then date each
channel:

```sh
~/src/php-20260924/scripts/nixos-first-shipped nixos-unstable <commit>
```

| Channels | Branch to read the SHA from |
|---|---|
| `nixos-unstable`, `nixos-unstable-small`, `nixpkgs-unstable` | `origin/master` |
| `nixos-26.05`, `nixos-26.05-small` | `origin/release-26.05` |

Release-branch channels ship the **backport**, not the `master` commit.

**Invoke every helper under `scripts/` by that absolute
primary-checkout path, never as `./scripts/…`.** A routine run
executes inside the auto-update worktree, where `scripts/` is
agent-writable on the `auto-update` branch; the wrapper's guard only
compares it against `origin/main` once at start-up, so only the
primary checkout's copy is reliably the reviewed one.

## Distros tracked

Intentionally narrow: Debian, NetBSD/pkgsrc, and NixOS, as in the
sibling trackers. Other distros (Rocky / Amazon / Ubuntu / Arch /
Fedora / Proxmox) are out of scope (the user decided this at seed) and
should not be added.

| Rows | Source of truth |
|---|---|
| Debian sid, forky, 13 (trixie), 12 (bookworm) — one row per suite and packaged `php8.X` | `scripts/debian-versions`, plus the security tracker's per-CVE pages |
| pkgsrc `pkgsrc-current`, `pkgsrc-2026Q3` × `lang/php82`…`lang/php85` | `scripts/pkgsrc-versions` against the local clone |
| NixOS `nixos-unstable`, `nixos-unstable-small`, `nixos-26.05`, `nixos-26.05-small` × `php82`…`php85` | `scripts/nixpkgs-versions` at each channel's revision |
| nixpkgs `master`, `release-26.05` branches and `nixpkgs-unstable` × `php82`…`php85` | `scripts/nixpkgs-versions` |

Debian bookworm is in LTS; its PHP updates arrive as DLAs from the LTS
team, into `bookworm-security`. PHP 8.3 never reached a Debian release
(it was only ever in experimental), and `php8.5` is only in
experimental, which is not tracked.

`nixos-25.11` is superseded by 26.05 and is not tracked; when 26.11
releases, the 26.05 rows move up the same way.

**New pkgsrc quarterly branches.** `pkgsrc-2026Q3` was cut on
2026-09-21, before the releases. When `pkgsrc-2026Q4` is cut, add its
rows above `pkgsrc-2026Q3`, add it to `BRANCHES` in
`scripts/pkgsrc-versions` in an interactive session (the headless agent
cannot edit `scripts/`), and retire the oldest quarterly's rows only if
they have already flipped — still-vulnerable rows stay. The same goes
for `CHANNELS` in `scripts/nixpkgs-versions` when 26.11 releases.

**No rows for a release/channel already dead when the tracker was
seeded**, and no rows for PHP branches already end-of-life (8.1 and
older). A release that goes end-of-life *while tracked* keeps its rows,
frozen at their final verdict.

## Build environment

- Hugo **≥ 0.146.0** (PaperMod's minimum); the standard edition
  suffices. Debian apt is too old: `go install
  github.com/gohugoio/hugo@latest`, kept current with `gup update` (see
  `~/src/cve-tracker-template/NEW-TRACKER.md` § "Host prerequisites
  (Debian)").
- Go (any recent version) — for Hugo Modules to pull PaperMod, and to
  build Hugo itself.
- `make check` needs `python3` and `git`; `scripts/debian-versions`
  also needs the host's `dpkg` (for `dpkg --compare-versions`), present
  on any Debian host — the script sets its own `PATH`, so a dev-shell
  copy would not be seen.
- The Nix flake provides these for an interactive shell: `nix develop`
  (or `cd` in if direnv is set up). The timer service runs on the host
  `PATH`, though, so the auto-update host still needs the apt packages
  — the `apt install` line in `NEW-TRACKER.md` § "Host prerequisites
  (Debian)" lists them.

## Auto-update worktree

The auto-update job works in a dedicated git worktree at
`~/src/auto-update/php-20260924`, checked out on a single long-lived
branch named `auto-update`. The wrapper merges `origin/main` forward
into that branch on each run, then hands off to headless Claude, which
commits any tracker changes back onto `auto-update` only. The agent
must not create per-run branches, switch branches, push, or open PRs —
merges of `auto-update` into `main` are done manually by the user.

The wrapper runs `git fetch origin` under `set -eu`, so `main` must be
on `origin` before the timer is worth enabling. A freshly seeded
tracker must `git remote add origin
https://github.com/suominen/php-20260924.git` and publish `main` there
first — a push is a breakpoint, so the user runs it.

One-time setup (from the primary checkout at `~/src/php-20260924`):

```
git worktree add -b auto-update ~/src/auto-update/php-20260924 main
```

The systemd units ship in `systemd/`. Wiring the timer means
symlinking both units into `~/.config/systemd/user/` with `ln -sr` and
enabling the timer:

```
ln -sr ~/src/php-20260924/systemd/php-20260924-tracker-update.service \
       ~/src/php-20260924/systemd/php-20260924-tracker-update.timer \
       ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now php-20260924-tracker-update.timer
```

The timer fires at `06,18:05` — a slot no other tracker uses, so the
shared `~/src/nixos/nixpkgs` and `~/src/netbsd/pkgsrc` clones are not
fetched simultaneously. Verify the live set with
`systemctl --user list-timers | grep tracker` before changing it.

**The timer runs the wrapper from the primary checkout, not from the
worktree.** `ExecStart` points at `<primary>/scripts/auto-update`, and
the wrapper reads its prompt from there too, so what executes is always
code you have reviewed and merged to `main`. The agent can commit to
`auto-update`, so anything under `scripts/`, `.claude/`, `CLAUDE.md`,
or `.mcp.json` on that branch is untrusted: after merging `origin/main`
forward the wrapper refuses to run if any of them differs from
`origin/main`, and it refuses outright if it was invoked from inside
the worktree at all.

The service unit adds a kernel-level backstop — `ProtectSystem=strict`
with a short `ReadWritePaths` list (the worktree, the `.git`
directories git has to update, and `~/.claude/session-env`). Each
repository's `.git/hooks` and `.git/config` are handed back as
`ReadOnlyPaths`.

A refusal is a stop-and-look rather than something to clear
reflexively. Inspect it with `git -C ~/src/auto-update/php-20260924
diff origin/main -- scripts .claude CLAUDE.md .mcp.json` before doing
anything else.

To run a refresh immediately (the same path the timer takes):

```
systemctl --user start php-20260924-tracker-update.service
```

It is a `oneshot`, so the command blocks until the run finishes; follow
its output with `journalctl --user -u php-20260924-tracker-update`.
Run the trackers **one at a time**, never in parallel — they share the
reference clones under `~/src`.

## Tearing down the auto-update

Unwire it in this order — `systemctl disable` needs the unit
definition to still be resolvable when it runs.

1. **Disable the timer first**, while the unit symlinks are still in
   place:

   ```
   systemctl --user disable --now php-20260924-tracker-update.timer
   ```

2. **Remove the service symlink** (the `.timer` link is already gone
   after step 1):

   ```
   rm ~/.config/systemd/user/php-20260924-tracker-update.service
   ```

3. **Reload** so the running user manager drops the units:

   ```
   systemctl --user daemon-reload
   ```

4. **Clear the failed residue**:

   ```
   systemctl --user reset-failed php-20260924-tracker-update.timer
   ```

If the definition symlinks were removed *before* disabling, delete
`~/.config/systemd/user/timers.target.wants/php-20260924-tracker-update.timer`
directly, then reload and `reset-failed` as above.

Finally, remove the worktree and its branch (from the primary checkout
at `~/src/php-20260924`):

```
git worktree remove ~/src/auto-update/php-20260924
git branch -d auto-update
```

## Debian verification recipe

Debian packages PHP as one versioned source per branch (`php8.2`,
`php8.4`, …). The unversioned `php*` packages come from
`php-defaults`, whose version (`93`, `96`, `99`, and `2:8.4+96` for
the binaries) carries no PHP release at all; its `debian/rules`
`PHP_DEFAULT_VERSION` names the branch. One command does the whole
lookup:

```sh
~/src/php-20260924/scripts/debian-versions
```

It prints, per suite and per `php8.X` source the suite carries (the
default first), a tab-separated
`<suite> <source> <version> <php> <default|-> <ids|->` line:

- `<suite>` is `<suite>-security` when the security archive carries
  a newer version than the main archive — the apt-installable one —
  otherwise the plain suite. When both hold the same version the
  tracker's page lists only the plain suite, and so does the helper.
- `<php>` is the upstream PHP release, taken from the newest
  `New upstream version X.Y.Z` line of that version's changelog and
  cross-checked against the version string. `?` means the changelog
  could not be read or disagreed (a warning says which) — read the
  changelog yourself before recording anything.
- `<ids>` are tracked advisory ids named in changelog entries *above*
  that upstream line: fixes backported onto that release.

It fails loudly if a page cannot be fetched, a page no longer parses,
or a suite's default source is missing; don't paper over a failure by
guessing.

**Verdict.** A row whose `<php>` is at or past its branch's fixed
release (8.2.34 / 8.4.26 / 8.5.11) is fixed. Below it, the row is
fixed only if backports cover every applicable advisory: cross-check
the `<ids>` column and the security tracker's per-CVE pages —

```
WebFetch https://security-tracker.debian.org/tracker/CVE-2026-91765
```

— for each of the ten applicable CVEs (CVE-2025-1218, CVE-2025-14181,
CVE-2026-6103, CVE-2026-91765, CVE-2026-91766, CVE-2026-91767,
CVE-2026-91768, CVE-2026-91769, CVE-2026-92842, CVE-2026-93682).
GHSA-ch8v-r6jh-4vvr (8.2 rows only) has no CVE and no tracker page; for
it the changelog is the only source. Some but not all is
`:warning: Partial`.

Each Debian row's *Fixed since* is the date that **suite** first
carried the version that closes the last of them. For sid that is the
upload date, from the changelog trailer or the fixed version's
`first_seen`:

```
curl -fsSL 'https://snapshot.debian.org/mr/package/php8.4/<version>/srcfiles?fileinfo=1'
```

Both of those date the upload to unstable, which is **not** forky's
date: the build reaches testing only after the age delay and
autopkgtests, often a week or more later. Never copy sid's date to
forky. forky's date is the testing migration, from the `php8.4
<version> MIGRATED to testing` entry on the package news page (it is
posted about a day after the migration):

```
curl -fsSL 'https://tracker.debian.org/pkg/php8.4/news/'
```

If the entry is not there yet, record the date the run first saw forky
at the fixed version and say so in the verification log; correct it
once the entry appears. A stable suite's date is its DSA/DLA, or the
point release that carried the fix.

sid is the canary; forky inherits via the usual sid → testing
migration; trixie advances via DSAs, bookworm via DLAs.

## pkgsrc verification recipe

pkgsrc (the NetBSD package collection; also used on SmartOS, macOS via
pkgin, etc.) lives in a local clone at `~/src/netbsd/pkgsrc`. Two
branches are tracked: `origin/trunk` (= `pkgsrc-current`) and the
current quarterly `origin/pkgsrc-2026Q3`. Quarterly branches receive
updates by pullup, so they can lag trunk by weeks.

Every PHP branch is its own package, `lang/php82` … `lang/php85`, but
**the PHP version is not in the package's Makefile**: all of them read
`PHPxx_VERSION` from `lang/php/phpversion.mk`, which also sets
`PHP_VERSION_DEFAULT`. The package is named
`${PHP_PKG_PREFIX}-${PHP_VERSION}` (`php84-8.4.26`), plus `nbN` when
`lang/phpXY/Makefile` sets `PKGREVISION` — which bumps for many
reasons besides security patches (the 2026Q3 `nb1` is a PCRE2
rebuild). One command assembles all of it:

```sh
~/src/php-20260924/scripts/pkgsrc-versions
```

It prints `<branch> <package> <identifier> <php> <default|-> <ids|->`
per branch and PHP package; `<ids>` lists tracked advisory ids named in
the package's `patches/`. The version decides unless `<ids>` is
non-empty, in which case read the patches for every applicable
advisory. An `absent` row means the branch does not carry that
package.

The script reads the clone through its `origin/...` refs and never
fetches; it warns if the clone was not fetched in the last 24 hours.

Derive a flip's *Fixed since* from the branch's history of
`phpversion.mk`, picking the commit that set the row's version by
**content**, not by subject or recency — quarterly commits are titled
`Pullup ticket #NNNN … lang/php84: Security fix`, and trunk subjects
are not reliable either (`lang/php84: udpate to 8.4.26`):

```sh
git -C ~/src/netbsd/pkgsrc log --format='%h %ad %s' --date=short -G '^PHP84_VERSION=.*8\.4\.26' origin/pkgsrc-2026Q3 -- lang/php/phpversion.mk
```

If `~/src/netbsd/pkgsrc` is missing, fail loudly rather than guessing.

### The clone is the source; a commit mail is a one-off exception

The git clone is the only pkgsrc source to check. Don't poll
`mail-index.netbsd.org`, don't read the CVS tree, and don't go looking
for a newer source when the clone reports a row as vulnerable.

If the git conversion is down, a commit mail **the user hands over**
may confirm an update the clone cannot show yet; cite it in the
tracker. Once a fact is recorded that way it outranks a lagging clone:
if a row is ahead of what `scripts/pkgsrc-versions` reports, believe
the tracker and leave the row alone.

## NixOS channel verification recipe

nixpkgs builds all four PHP branches from
`pkgs/development/interpreters/php/default.nix`, one
`phpXY = mkPhp { version = …; hash = …; }` block each, sharing
`generic.nix`; `pkgs/top-level/all-packages.nix` sets `php = php84;`.
One command reads every tracked channel and branch at its current
revision:

```sh
~/src/php-20260924/scripts/nixpkgs-versions
```

It prints `<channel> <attribute> <php> <default|-> <ids|->` for
`branch:master`, `branch:release-26.05`, the four NixOS channels, and
`nixpkgs-unstable`, following each channel's `channels.nixos.org`
pointer. Pass channel names or `branch:<name>` arguments to check a
subset. `<ids>` lists tracked ids named inside the attribute's block,
then — prefixed `generic:` — ids named in shared code, which may still
be gated to some branches: read the expression before trusting either.
It fails loudly on a parse problem and warns on a stale clone.

The `branch:master` and `branch:release-26.05` rows exist because a
flake input pinned to `github:NixOS/nixpkgs/<branch>` resolves to that
branch; an input following a *channel* name is answered by that
channel's rows. **A branch row's *Fixed since* is the commit date of
the version bump**, not a channel release date. The branch dates also
bound the channel dates.

Verdict: `<php>` at or past the branch's fixed release is fixed; a
patch named after the ids on an older base needs every applicable
advisory covered, else Partial.

Never record channel git-revisions in the tracker. The **release
name** reported by `scripts/nixos-first-shipped` belongs in the
verification log.

If `~/src/nixos/nixpkgs` is missing, fail loudly rather than guessing.

## Key sources to monitor

| Source | URL |
|---|---|
| PHP 8 ChangeLog | <https://www.php.net/ChangeLog-8.php> |
| Release announcements | <https://www.php.net/releases/8_5_11.php>, `8_4_26`, `8_3_35`, `8_2_34` |
| php-src advisories (API) | <https://api.github.com/repos/php/php-src/security-advisories> |
| CVE Services API | <https://cveawg.mitre.org/api/cve/CVE-2026-91765> (and the other ten) |
| NVD API (JSON — the NVD web pages are JS-only) | <https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-2026-91765> |
| Debian security tracker | <https://security-tracker.debian.org/tracker/CVE-2026-91765> (per CVE), `/tracker/source-package/php8.4` (per source) |
| Debian changelogs | <https://tracker.debian.org/pkg/php8.4> |
| NixOS security tracker | <https://tracker.security.nixos.org/> |
| NixOS channel pointers | <https://channels.nixos.org/nixos-unstable/git-revision> (and the other channels) |
| pkgsrc — `lang/php/phpversion.mk` (GitHub mirror) | <https://github.com/NetBSD/pkgsrc/blob/trunk/lang/php/phpversion.mk> |

Prefer plain-text and JSON endpoints and the Debian tracker's HTML over
JS-rendered SPAs — those return nothing to WebFetch, and reading one
falsely sees "no advisory". The GitHub advisory pages, the NVD web
pages, and the NixOS security tracker are all SPAs: link them, but take
the facts from the APIs and the local clones.

When the CVE records are published in the CVE List, check whether they
change anything the tracker states (scores, CWEs, affected ranges) and
update the Summary and verification log; the GitHub advisories stay
the primary source for fixed versions.

## Known harmless warnings during build

PaperMod's templates still call `.Language.LanguageDirection` and
`.Language.LanguageCode`, which Hugo deprecated in 0.158.0. The build
emits `WARN deprecated:` lines for both. Upstream theme issue — don't
try to fix it in this repo.

## License

The tracker content is licensed under **CC BY 4.0** (see `LICENSE` at
the repo root). Copyright © 2026 Kimmo Suominen. The site footer
credits both the author and the licence.
