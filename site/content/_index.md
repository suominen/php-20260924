---
title: "PHP 2026-09-24 security releases — 11 CVEs across 8.2–8.5"
description: "PHP 8.5.11, 8.4.26, 8.3.35, and 8.2.34 security releases (11 CVEs, 12 GitHub advisories: SOAP, FPM, OpenSSL, HTTP stream wrapper, Phar, mysqlnd) — Debian, pkgsrc, and NixOS patch status tracker"
layout: "single"
date: 2026-09-25
lastmod: 2026-10-05
cover:
  image: "php-20260924-tracker.png"
  alt: "PHP 2026-09-24 security releases — patch status tracker"
  hiddenInSingle: true
---

## Summary

| Field | Detail |
|---|---|
| Releases | PHP **8.5.11**, **8.4.26**, **8.3.35**, and **8.2.34**, all released 2026-09-24 — every supported branch at once |
| Advisories | 12 GitHub security advisories, 11 of them with a CVE ID — listed under [Advisories](#advisories) |
| Highest severity | [CVE-2026-91765][ghsa-rgrp] — **7.5 high** (CVSS 3.1): unauthenticated remote crash of any `SoapServer` endpoint |
| Types | Stack exhaustion, heap buffer overflows and over-reads, an access-control bypass, TLS hostname verification flaws, a credential leak, archive entry injection |
| Differs by branch | [GHSA-ch8v-r6jh-4vvr][ghsa-ch8v] is fixed in 8.2.34 and 8.3.35 only — 8.4 and 8.5 already fixed it in 8.4.25 and 8.5.10. [CVE-2026-17545][ghsa-9f67] affects PHP on Windows only |
| CVE records | Published in the CVE List, assigned by the PHP project's own CNA — scores mostly match the GitHub advisories; three (CVE-2025-1218, CVE-2026-93682, CVE-2026-6103) carry a higher CVSS score on a Scope: Changed vector, see the [verification log](#verification-log) |
| KEV listed | no |
| Public exploit | none known — the advisories carry proof-of-concept inputs, but no exploit has been published |
{.summary}

The releases close a dozen unrelated bugs across the interpreter's
extensions rather than one headline flaw, and which of them matter
depends on what a host's PHP is used for:

- **Servers accepting requests.** A `SoapServer` endpoint can be
  crashed by one deeply nested request (CVE-2026-91765), and PHP-FPM's
  `listen.allowed_clients` lets in any IPv6 address that shares the
  first 96 bits with an allowed one (CVE-2026-91768).
- **PHP as a client.** Code that fetches URLs, talks TLS, or connects
  to SOAP or MySQL servers trusts what those servers send back. A
  malicious server can overflow or over-read heap buffers
  (CVE-2026-91767, CVE-2025-14181, CVE-2026-93682, CVE-2025-1218), pass
  hostname verification with a certificate that names a different host
  (CVE-2026-91769), or collect the `Authorization`, `Cookie`, and
  `Proxy-Authorization` headers the `http://` wrapper forwards across a
  redirect to another origin or from HTTPS to HTTP (CVE-2026-91766).
- **Untrusted input to library code.** A crafted TAR archive opened
  through Phar injects entries of the attacker's choosing
  (CVE-2026-6103); an attacker-influenced `line-break-chars` option on
  a `convert.*` stream filter reads past a heap buffer
  (CVE-2026-92842); and `FILTER_SANITIZE_ENCODED` may pass the byte
  `0xFF` through unencoded (GHSA-ch8v-r6jh-4vvr).

> :information_source: Each branch has its own fixed release: a host
> on PHP 8.4 needs 8.4.26, not 8.5.11. Which branch a system runs
> depends on its distribution — Debian 12 ships PHP 8.2, Debian 13
> ships 8.4, while NixOS and pkgsrc offer all four side by side — so
> the table below gives each row's PHP branch and the release it
> needs. A distribution may also backport the fixes onto an older
> release of the same branch, so the PHP version alone does not
> settle every row.

## Advisories

The [PHP 8 changelog][changelog] lists what each release fixed. Every
advisory below is fixed in all four 2026-09-24 releases, with one
exception, GHSA-ch8v-r6jh-4vvr; CVE-2026-17545 matters only on
Windows.

| Advisory | CVE | Area | Severity | Fixed in |
|---|---|---|---|---|
| [GHSA-rgrp-mwpx-f6rm][ghsa-rgrp] | CVE-2026-91765 | SOAP: unbounded recursion in server-side `cleanup_xml_node()` | 7.5 high | all four |
| [GHSA-62xp-839h-2637][ghsa-62xp] | CVE-2026-91768 | FPM: IPv6 `listen.allowed_clients` compares only 12 of 16 bytes | 6.5 medium | all four |
| [GHSA-xr7j-rvgx-xq5p][ghsa-xr7j] | CVE-2026-91767 | OpenSSL: heap over-read on a crafted wildcard certificate name | 6.5 medium | all four |
| [GHSA-cj93-vc83-wgqv][ghsa-cj93] | CVE-2025-14181 | SOAP: integer overflow to heap buffer overflow in HTTP response parsing | 6.5 medium | all four |
| [GHSA-88hq-2827-7pg6][ghsa-88hq] | CVE-2026-92842 | Streams: `convert.*` filters over-read when `line-break-chars` contains NUL | 5.9 medium | all four |
| [GHSA-fpwc-w8rq-cr92][ghsa-fpwc] | CVE-2026-91766 | Streams: HTTP wrapper forwards credentials across cross-origin redirects | 5.9 medium | all four |
| [GHSA-7875-c8px-7q5f][ghsa-7875] | CVE-2026-93682 | Streams: HTTP wrapper over-reads on an empty `Location` redirect | 5.3 medium | all four |
| [GHSA-vvx9-73fr-5jjx][ghsa-vvx9] | CVE-2026-91769 | OpenSSL: hostname verification falls back to the CN after a SAN mismatch | 4.3 medium | all four |
| [GHSA-j3wh-g957-2m85][ghsa-j3wh] | CVE-2026-6103 | Phar: TAR size integer overflow allows archive entry injection | 4.0 medium | all four |
| [GHSA-r6x9-5r99-36j7][ghsa-r6x9] | CVE-2025-1218 | mysqlnd: packet over-reads from a malicious server | 3.1 low | all four |
| [GHSA-ch8v-r6jh-4vvr][ghsa-ch8v] | — | Filter: `FILTER_SANITIZE_ENCODED` may leave `0xFF` unencoded | 3.7 low | **8.2.34 and 8.3.35**; already in 8.4.25 and 8.5.10 |
| [GHSA-9f67-6fw4-hpfp][ghsa-9f67] | CVE-2026-17545 | Windows: reserved device names (`CON`, `NUL`, …) not rejected in paths | 6.9 medium (CVSS 4.0) | all four — **Windows only** |

Severities are the scores in PHP's advisories, CVSS 3.1 unless marked.

> :information_source: **Where the branches differ.** 8.2 and 8.3
> receive security fixes only, so their releases carry exactly the
> twelve advisories above. 8.4 and 8.5 are still in active support, so
> their security releases also carry regular bug fixes, among them
> memory-safety bugs that got no advisory — use-after-frees in DOM,
> Intl, and Phar, out-of-bounds reads in BCMath and `mb_ereg_replace()`,
> and, in 8.4 only, a buffer overflow in `hash_pbkdf2()` with a large
> output length. And
> GHSA-ch8v-r6jh-4vvr is new in 8.2.34 and 8.3.35 only: the 8.4 and
> 8.5 fix shipped a month earlier, so an 8.4 or 8.5 host still older
> than 8.4.25 or 8.5.10 lacks that fix as well.

CVE-2026-17545 applies only to PHP running on Windows: the device-name
check it adds is Windows code. None of the distributions tracked here
build PHP for Windows, so their rows need only the other advisories of
their branch.

## Distribution status

A row is **Fixed** once its package closes every advisory its PHP
branch's 2026-09-24 release fixes (CVE-2026-17545 aside): by carrying
that release or a later one, or by backporting all of the fixes onto
an older release of the branch. A package that closes some but not all
is **Partial**, with the advisories still open named.

*PHP* is the upstream PHP release the package builds; *Package* is
what the distribution calls it. Where a release offers several PHP
branches, the one its unversioned `php` package or attribute selects
is marked *default*. Debian versions are the
apt-installable value: the `<suite>-security` archive where it
carries the package, since that is what hosts run and where a DSA or
DLA fix lands first.

| Distribution | Release | PHP | Package | Fixed since | Status |
|---|---|---|---|---|---|
| Debian | sid | `8.4.26` | `php8.4` `8.4.26-1` | 2026-09-29 | :white_check_mark: Fixed |
| Debian | forky (testing) | `8.4.24` | `php8.4` `8.4.24-1` | — | :x: Vulnerable — needs 8.4.26 |
| Debian | 13 (trixie) | `8.4.26` | `php8.4` `8.4.26-1~deb13u1` | 2026-09-25 | :white_check_mark: Fixed |
| Debian | 12 (bookworm) | `8.2.34` | `php8.2` `8.2.34-1~deb12u1` | 2026-10-04 | :white_check_mark: Fixed |
| pkgsrc | `pkgsrc-current` | `8.4.26` | `php84-8.4.26` (default) | 2026-09-24 | :white_check_mark: Fixed |
| pkgsrc | `pkgsrc-current` | `8.5.11` | `php85-8.5.11` | 2026-09-24 | :white_check_mark: Fixed |
| pkgsrc | `pkgsrc-current` | `8.3.35` | `php83-8.3.35` | 2026-09-24 | :white_check_mark: Fixed |
| pkgsrc | `pkgsrc-current` | `8.2.34` | `php82-8.2.34` | 2026-09-24 | :white_check_mark: Fixed |
| pkgsrc | `pkgsrc-2026Q3` | `8.4.25` | `php84-8.4.25nb1` (default) | — | :x: Vulnerable — needs 8.4.26 |
| pkgsrc | `pkgsrc-2026Q3` | `8.5.10` | `php85-8.5.10nb1` | — | :x: Vulnerable — needs 8.5.11 |
| pkgsrc | `pkgsrc-2026Q3` | `8.3.33` | `php83-8.3.33nb1` | — | :x: Vulnerable — needs 8.3.35 |
| pkgsrc | `pkgsrc-2026Q3` | `8.2.33` | `php82-8.2.33nb1` | — | :x: Vulnerable — needs 8.2.34 |
| NixOS | `nixos-unstable` | `8.4.26` | `php84` (default) | 2026-09-28 | :white_check_mark: Fixed |
| NixOS | `nixos-unstable` | `8.5.11` | `php85` | 2026-09-28 | :white_check_mark: Fixed |
| NixOS | `nixos-unstable` | `8.3.35` | `php83` | 2026-09-28 | :white_check_mark: Fixed |
| NixOS | `nixos-unstable` | `8.2.34` | `php82` | 2026-09-28 | :white_check_mark: Fixed |
| NixOS | `nixos-unstable-small` | `8.4.26` | `php84` (default) | 2026-09-27 | :white_check_mark: Fixed |
| NixOS | `nixos-unstable-small` | `8.5.11` | `php85` | 2026-09-27 | :white_check_mark: Fixed |
| NixOS | `nixos-unstable-small` | `8.3.35` | `php83` | 2026-09-27 | :white_check_mark: Fixed |
| NixOS | `nixos-unstable-small` | `8.2.34` | `php82` | 2026-09-27 | :white_check_mark: Fixed |
| NixOS | `nixos-26.05` | `8.4.26` | `php84` (default) | 2026-09-28 | :white_check_mark: Fixed |
| NixOS | `nixos-26.05` | `8.5.11` | `php85` | 2026-09-28 | :white_check_mark: Fixed |
| NixOS | `nixos-26.05` | `8.3.35` | `php83` | 2026-09-28 | :white_check_mark: Fixed |
| NixOS | `nixos-26.05` | `8.2.34` | `php82` | 2026-09-28 | :white_check_mark: Fixed |
| NixOS | `nixos-26.05-small` | `8.4.26` | `php84` (default) | 2026-09-27 | :white_check_mark: Fixed |
| NixOS | `nixos-26.05-small` | `8.5.11` | `php85` | 2026-09-27 | :white_check_mark: Fixed |
| NixOS | `nixos-26.05-small` | `8.3.35` | `php83` | 2026-09-27 | :white_check_mark: Fixed |
| NixOS | `nixos-26.05-small` | `8.2.34` | `php82` | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `master` branch | `8.4.26` | `php84` (default) | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `master` branch | `8.5.11` | `php85` | 2026-09-24 | :white_check_mark: Fixed |
| nixpkgs | `master` branch | `8.3.35` | `php83` | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `master` branch | `8.2.34` | `php82` | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `release-26.05` branch | `8.4.26` | `php84` (default) | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `release-26.05` branch | `8.5.11` | `php85` | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `release-26.05` branch | `8.3.35` | `php83` | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `release-26.05` branch | `8.2.34` | `php82` | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `nixpkgs-unstable` | `8.4.26` | `php84` (default) | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `nixpkgs-unstable` | `8.5.11` | `php85` | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `nixpkgs-unstable` | `8.3.35` | `php83` | 2026-09-27 | :white_check_mark: Fixed |
| nixpkgs | `nixpkgs-unstable` | `8.2.34` | `php82` | 2026-09-27 | :white_check_mark: Fixed |
{.distros}

### Debian

Debian ships one PHP branch per release, as a versioned source
package: `php8.2` in bookworm, `php8.4` in trixie, forky, and sid. The
unversioned `php`, `php-cli`, `php-fpm` packages come from
`php-defaults` and only depend on the branch's packages, so their
version (`2:8.4+96` on trixie) says nothing about which PHP release is
installed — look at the `php8.X-*` packages instead. `php8.5` is in
experimental only, and PHP 8.3 never reached a Debian release — it
was only ever in experimental.

Debian's PHP maintainer normally imports each upstream point release,
security uploads to the stable suites included, so the upstream part
of the version (everything before the last `-`) is the PHP release
it builds.

The [Debian security tracker][debian-91765] has a page per CVE; trixie
and sid closed all ten applicable CVEs via [DSA-6514-1][dsa-6514] and
a direct 8.4.26 upload, and bookworm closed them via
[DLA-4819-1][dla-4819] (8.2.34). forky is still open on every
applicable CVE. The tracker marks CVE-2026-17545 not affected ("Only
affects PHP on Windows"). GHSA-ch8v-r6jh-4vvr has no CVE and so no
tracker page; it reaches a suite with the upstream release or a
changelog entry naming it.

Bookworm is in its LTS period, so its PHP updates come as DLAs from
the LTS team, into `bookworm-security`; that archive can run ahead of
the main archive, which only catches up at a point release.

### pkgsrc

pkgsrc (NetBSD's package collection, also used on SmartOS, macOS via
pkgin, and elsewhere) packages every PHP branch side by side as
`lang/php82` through `lang/php85`; the installed package is named
after the branch (`php84-8.4.26`). `PHP_VERSION_DEFAULT` selects the
branch a PHP-dependent package builds against unless the host
overrides it — 8.4 on both tracked branches.

`pkgsrc-2026Q3`, the quarterly branch most pkgsrc hosts run, was cut
on 2026-09-21 — before the releases — and receives them by pullup
from `pkgsrc-current`.

The identifier carries an `nbN` suffix when the package's
`PKGREVISION` is set. That revision bumps for many reasons besides
security patches — the `nb1` on the 2026Q3 packages comes from a
rebuild for a new PCRE2 — so an `nbN` suffix on its own says nothing
about these fixes.

### NixOS

nixpkgs builds all four branches from one expression,
`pkgs/development/interpreters/php/default.nix`, as the attributes
`php82` through `php85`; the unversioned `php` (and `phpPackages`,
`phpExtensions`) is `php84`. The NixOS `services.phpfpm` module uses
`pkgs.php` unless a pool sets its own `phpPackage`. A channel row is
fixed once its pinned version reaches the branch's fixed release, or
the expression carries the fixes as patches.

#### Flake users

A flake input like `github:NixOS/nixpkgs/nixos-unstable` resolves to
the **git branch** of that name, not to the channel — but the channel
bot advances the branch to exactly the revision the channel publishes,
so an input following a channel name is answered by that channel's
rows.

Two branches have no channel gating them and will carry the fix
earlier — they are listed above as `master` and `release-26.05`:

- `github:NixOS/nixpkgs/master` — the fix's first appearance anywhere
  in nixpkgs, the moment it is merged.
- `github:NixOS/nixpkgs/release-26.05` — the ungated 26.05 branch,
  ahead of the `nixos-26.05` channel by however long Hydra takes.

No channel can be fixed before the branch it is cut from.

## Detection

Check the installed package rather than the running interpreter: a
host can carry several PHP branches, and the web server's may not be
the one on `PATH`.

### Debian

List the PHP packages dpkg knows about, with their state and version:

```sh
dpkg-query -W -f='${db:Status-Abbrev} ${Package} ${Version}\n' 'php8*'
```

Lines starting `ii` are installed. The branch is in the package name (`php8.4-fpm`) and the upstream PHP
release is the version up to the last `-`. Compare it with
the row for your suite above; if a later Debian revision of the same
release appears (`8.4.24-1~deb13u2`), check its changelog for
backported fixes:

```sh
apt changelog php8.4-common
```

### pkgsrc

```sh
pkg_info -e 'php8[0-9]-[0-9]*'
```

The output is `php<branch>-<version>` plus any `nb<PKGREVISION>`
suffix — for example `php84-8.4.25nb1`. Cross-reference it against the
pkgsrc rows above.

### NixOS

Query the PHP versions in the current system closure:

```sh
nix-store -q --requisites /run/current-system | grep -- '-php-[0-9]'
```

Each store path ends in the PHP version it builds (`php-8.4.25`, or
`php-8.4.25-dev` for the development output).
Cross-reference against your channel's rows above; a project built
from a flake or a `nix-shell` carries its own PHP, pinned by its own
nixpkgs input.

## Mitigation

Upgrade to your branch's fixed release — or to a distribution package
that carries it — as soon as one is available. Until then, what helps
depends on the advisory:

- **SOAP servers (CVE-2026-91765).** An unauthenticated request to any
  `SoapServer` endpoint crashes the worker. If SOAP is not used,
  disable the extension (`phpdismod soap` on Debian, then restart
  PHP-FPM or Apache); otherwise limit who can reach the endpoint.
- **PHP-FPM over IPv6 (CVE-2026-91768).** Only pools listening on an
  IPv6 TCP socket with `listen.allowed_clients` set are affected. A
  Unix socket, or a firewall rule admitting only the web server's
  exact address, closes it.
- **Outbound HTTP (CVE-2026-91766, CVE-2026-93682).** Code passing
  `Authorization`, `Cookie`, or `Proxy-Authorization` headers to `file_get_contents()` or
  `fopen()` on `http://` and `https://` URLs can set the stream
  context option `follow_location` to `0` and handle redirects
  itself.
- **Outbound TLS (CVE-2026-91767, CVE-2026-91769).** Both are
  triggered by the certificate a server presents to PHP as a client;
  there is no configuration workaround beyond not connecting to
  untrusted hosts.
- **Untrusted archives (CVE-2026-6103).** Don't open user-supplied
  TAR files through Phar or `PharData` until patched.

No exploit has been published for any of these. The advisories
describe proof-of-concept inputs, so crashing a `SoapServer` or
reading past a buffer takes little effort once the fix is public.

## Verification log

Every verdict in the table above is backed by a checkable source. This
log records the provenance — the advisory, repository index, or git
reference that established each fact — so any row can be audited or
reproduced. Most readers never need it.

{{< details summary="Full verification log" >}}
#### Upstream

- **Releases** (via the [PHP 8 changelog][changelog] and the release
  announcements for [8.5.11][rel-8511], [8.4.26][rel-8426],
  [8.3.35][rel-8335], and [8.2.34][rel-8234]):
  - All four dated 2026-09-24; each announcement calls its release
    "a security release" and asks all users of the branch to upgrade.
  - 8.3.35 and 8.2.34 list the twelve advisories and nothing else.
  - 8.5.11 and 8.4.26 list eleven advisories — all but
    GHSA-ch8v-r6jh-4vvr — among their regular bug fixes.
- **GitHub advisories** (via the GitHub REST API,
  `api.github.com/repos/php/php-src/security-advisories`, since the
  advisory pages render only through JavaScript):
  - Twelve advisories published 2026-09-24; each lists the fixed
    version per branch as quoted in [Advisories](#advisories).
  - GHSA-ch8v-r6jh-4vvr: patched in 8.2.34, 8.3.35, 8.4.25, and
    8.5.10; no CVE ID.
  - Severities and CVSS scores as quoted in the Advisories table:
    CVSS 3.1 for eleven, CVSS 4.0 only for CVE-2026-17545.
- **CVE records** (via the CVE Services API, `cveawg.mitre.org`):
  - All eleven CVE IDs now have a `PUBLISHED` record, assigned by
    `assignerShortName: php` (the PHP project's own CNA).
  - Eight of the eleven carry the same CVSS 3.1 vector and score as
    the matching GitHub advisory.
  - Three score higher than their GitHub advisory on a `Scope:
    Changed` (`S:C`) vector rather than the advisory's `S:U`:
    CVE-2025-1218 (3.1 → 3.4), CVE-2026-93682 (5.3 → 5.8), and
    CVE-2026-6103 (4.0 → 4.3). The GitHub advisory's version and
    severity stay the tracker's source per its own convention; this
    is recorded here as the discrepancy, not acted on.
  - CVE-2026-17545's CVSS 4.0 score (6.9) matches its GitHub advisory.
- **KEV**: none of the eleven CVE IDs is in CISA's catalog.

#### Distributions

- **Debian** (via `scripts/debian-versions`, reading the security
  tracker's source-package pages, `php-defaults`' `debian/rules`, and
  the package changelogs on tracker.debian.org):
  - Default branch per suite: `PHP_DEFAULT_VERSION` in the suite's
    `php-defaults` `debian/rules`.
  - Branch per suite: `php8.2` in bookworm, `php8.4` in trixie, forky,
    and sid.
  - PHP release per row: the newest "New upstream version" line of the
    package's changelog.
  - No tracked advisory id in any changelog entry above that line.
  - bookworm's fixed version comes from `bookworm-security`, an LTS
    team upload: `php8.2` `8.2.34-1~deb12u1`, first seen 2026-10-04 on
    snapshot.debian.org.
  - trixie's fixed version comes from `trixie-security`: `php8.4`
    `8.4.26-1~deb13u1`, first seen 2026-09-25 on
    snapshot.debian.org.
  - sid's fixed version is `php8.4` `8.4.26-1`, first seen
    2026-09-29 on snapshot.debian.org.
- **Debian security tracker** (per-CVE pages):
  - All ten applicable CVEs closed for `php8.2` in bookworm via
    [DLA-4819-1][dla-4819] (`8.2.34-1~deb12u1`), for `php8.4` in
    trixie via [DSA-6514-1][dsa-6514] (`8.4.26-1~deb13u1`), and in sid
    (`8.4.26-1`); still open for `php8.4` in forky.
  - CVE-2026-17545 `not-affected` ("Only affects PHP on Windows").
- **pkgsrc** (via `scripts/pkgsrc-versions` against the local clone):
  - Versions from `lang/php/phpversion.mk`; `PHP_VERSION_DEFAULT` 84
    on both branches.
  - `origin/trunk`: the four releases landed 2026-09-24, one commit
    per PHP branch.
  - `origin/pkgsrc-2026Q3`: branch point 2026-09-21; the `nb1`
    revisions come from the 2026-09-02 recursive bump for PCRE2 10.48.
  - No tracked advisory id in any `patches/` directory on either
    branch.
- **NixOS / nixpkgs** (via `scripts/nixpkgs-versions` against the
  local clone at each channel's revision pointer):
  - Versions from the `php82` … `php85` blocks of
    `pkgs/development/interpreters/php/default.nix`; `php = php84`.
  - No tracked advisory id in `default.nix` or `generic.nix`.
  - `branch:master` and `branch:release-26.05` carry all four fixed
    releases; php85's 8.5.11 bump landed 2026-09-24, the php84/83/82
    bumps 2026-09-27 (commit dates on each branch).
  - `nixos-unstable-small` and `nixos-26.05-small` carry all four
    fixed releases as of `nixos-26.11pre1080371.545c226a9af7` and
    `nixos-26.05.10742.a71ca2a7b9c4` respectively, both published
    2026-09-27 (via `scripts/nixos-first-shipped`).
  - `nixos-unstable` and `nixos-26.05` (non-`-small`) caught up a day
    later, carrying all four fixed releases as of
    `nixos-26.11pre1080855.7a0f122f5090` and
    `nixos-26.05.10769.cf5e76507c6e` respectively, both published
    2026-09-28 (via `scripts/nixos-first-shipped`).
  - `nixpkgs-unstable` carries all four fixed releases as of
    `nixpkgs-26.11pre1080404.3181085bfd08`, published 2026-09-27 (via
    `scripts/nixos-first-shipped`).
- **NixOS security tracker** ([tracker.security.nixos.org][nixos-sec])
  — a JS-rendered application, linked for readers.
{{< /details >}}

## References

| Source | URL |
|---|---|
| [PHP 8 ChangeLog][changelog] | <https://www.php.net/ChangeLog-8.php> |
| [PHP 8.5.11 release announcement][rel-8511] | <https://www.php.net/releases/8_5_11.php> |
| [PHP 8.4.26 release announcement][rel-8426] | <https://www.php.net/releases/8_4_26.php> |
| [PHP 8.3.35 release announcement][rel-8335] | <https://www.php.net/releases/8_3_35.php> |
| [PHP 8.2.34 release announcement][rel-8234] | <https://www.php.net/releases/8_2_34.php> |
| [php-src security advisories][ghsa-index] | <https://github.com/php/php-src/security/advisories> |
| [GHSA-rgrp-mwpx-f6rm — CVE-2026-91765][ghsa-rgrp] | <https://github.com/php/php-src/security/advisories/GHSA-rgrp-mwpx-f6rm> |
| [GHSA-62xp-839h-2637 — CVE-2026-91768][ghsa-62xp] | <https://github.com/php/php-src/security/advisories/GHSA-62xp-839h-2637> |
| [GHSA-xr7j-rvgx-xq5p — CVE-2026-91767][ghsa-xr7j] | <https://github.com/php/php-src/security/advisories/GHSA-xr7j-rvgx-xq5p> |
| [GHSA-cj93-vc83-wgqv — CVE-2025-14181][ghsa-cj93] | <https://github.com/php/php-src/security/advisories/GHSA-cj93-vc83-wgqv> |
| [GHSA-88hq-2827-7pg6 — CVE-2026-92842][ghsa-88hq] | <https://github.com/php/php-src/security/advisories/GHSA-88hq-2827-7pg6> |
| [GHSA-fpwc-w8rq-cr92 — CVE-2026-91766][ghsa-fpwc] | <https://github.com/php/php-src/security/advisories/GHSA-fpwc-w8rq-cr92> |
| [GHSA-7875-c8px-7q5f — CVE-2026-93682][ghsa-7875] | <https://github.com/php/php-src/security/advisories/GHSA-7875-c8px-7q5f> |
| [GHSA-vvx9-73fr-5jjx — CVE-2026-91769][ghsa-vvx9] | <https://github.com/php/php-src/security/advisories/GHSA-vvx9-73fr-5jjx> |
| [GHSA-j3wh-g957-2m85 — CVE-2026-6103][ghsa-j3wh] | <https://github.com/php/php-src/security/advisories/GHSA-j3wh-g957-2m85> |
| [GHSA-r6x9-5r99-36j7 — CVE-2025-1218][ghsa-r6x9] | <https://github.com/php/php-src/security/advisories/GHSA-r6x9-5r99-36j7> |
| [GHSA-ch8v-r6jh-4vvr][ghsa-ch8v] | <https://github.com/php/php-src/security/advisories/GHSA-ch8v-r6jh-4vvr> |
| [GHSA-9f67-6fw4-hpfp — CVE-2026-17545][ghsa-9f67] | <https://github.com/php/php-src/security/advisories/GHSA-9f67-6fw4-hpfp> |
| [Debian security tracker — CVE-2026-91765][debian-91765] | <https://security-tracker.debian.org/tracker/CVE-2026-91765> |
| [DSA-6514-1][dsa-6514] | <https://security-tracker.debian.org/tracker/DSA-6514-1> |
| [DLA-4819-1][dla-4819] | <https://security-tracker.debian.org/tracker/DLA-4819-1> |
| [Debian security tracker — `php8.4`][debian-php84] | <https://security-tracker.debian.org/tracker/source-package/php8.4> |
| [Debian security tracker — `php8.2`][debian-php82] | <https://security-tracker.debian.org/tracker/source-package/php8.2> |
| [pkgsrc — `lang/php/phpversion.mk` (GitHub mirror)][pkgsrc-phpversion] | <https://github.com/NetBSD/pkgsrc/blob/trunk/lang/php/phpversion.mk> |
| [nixpkgs — PHP expressions][nixpkgs-php] | <https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/interpreters/php/default.nix> |
| [NixOS security tracker][nixos-sec] | <https://tracker.security.nixos.org/> |
{.references}

[changelog]:          https://www.php.net/ChangeLog-8.php
[rel-8511]:           https://www.php.net/releases/8_5_11.php
[rel-8426]:           https://www.php.net/releases/8_4_26.php
[rel-8335]:           https://www.php.net/releases/8_3_35.php
[rel-8234]:           https://www.php.net/releases/8_2_34.php
[ghsa-index]:         https://github.com/php/php-src/security/advisories
[ghsa-rgrp]:          https://github.com/php/php-src/security/advisories/GHSA-rgrp-mwpx-f6rm
[ghsa-62xp]:          https://github.com/php/php-src/security/advisories/GHSA-62xp-839h-2637
[ghsa-xr7j]:          https://github.com/php/php-src/security/advisories/GHSA-xr7j-rvgx-xq5p
[ghsa-cj93]:          https://github.com/php/php-src/security/advisories/GHSA-cj93-vc83-wgqv
[ghsa-88hq]:          https://github.com/php/php-src/security/advisories/GHSA-88hq-2827-7pg6
[ghsa-fpwc]:          https://github.com/php/php-src/security/advisories/GHSA-fpwc-w8rq-cr92
[ghsa-7875]:          https://github.com/php/php-src/security/advisories/GHSA-7875-c8px-7q5f
[ghsa-vvx9]:          https://github.com/php/php-src/security/advisories/GHSA-vvx9-73fr-5jjx
[ghsa-j3wh]:          https://github.com/php/php-src/security/advisories/GHSA-j3wh-g957-2m85
[ghsa-r6x9]:          https://github.com/php/php-src/security/advisories/GHSA-r6x9-5r99-36j7
[ghsa-ch8v]:          https://github.com/php/php-src/security/advisories/GHSA-ch8v-r6jh-4vvr
[ghsa-9f67]:          https://github.com/php/php-src/security/advisories/GHSA-9f67-6fw4-hpfp
[debian-91765]:       https://security-tracker.debian.org/tracker/CVE-2026-91765
[dsa-6514]:           https://security-tracker.debian.org/tracker/DSA-6514-1
[dla-4819]:           https://security-tracker.debian.org/tracker/DLA-4819-1
[debian-php84]:       https://security-tracker.debian.org/tracker/source-package/php8.4
[debian-php82]:       https://security-tracker.debian.org/tracker/source-package/php8.2
[pkgsrc-phpversion]:  https://github.com/NetBSD/pkgsrc/blob/trunk/lang/php/phpversion.mk
[nixpkgs-php]:        https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/interpreters/php/default.nix
[nixos-sec]:          https://tracker.security.nixos.org/
