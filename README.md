# case/homebrew-tap

A personal [Homebrew tap](https://docs.brew.sh/Taps).

## Casks

| Cask                        | Version | What it is                                                    |
| --------------------------- | ------- | ------------------------------------------------------------- |
| `font-case-office-code-pro` | 1.004   | Office Code Pro, a monospaced coding font (SIL OFL v1.1)      |

## Install

```
brew install --cask case/tap/font-case-office-code-pro
```

Installing by fully qualified name taps this repository and trusts only that
one cask. Homebrew 6.0.0 requires explicit trust for non-official taps — see
[Tap Trust](https://docs.brew.sh/Tap-Trust).

In a `Brewfile`:

```ruby
tap "case/tap", trusted: { cask: "font-case-office-code-pro" }
cask "case/tap/font-case-office-code-pro"
```

## Why Office Code Pro is here and not in homebrew-cask

The upstream repository `nathco/Office-Code-Pro` was deleted from GitHub around
2025-08-18, which disabled the `font-office-code-pro` cask in
[homebrew-cask#224323](https://github.com/Homebrew/homebrew-cask/pull/224323).

Two attempts to restore it upstream were declined:

- [#264724](https://github.com/Homebrew/homebrew-cask/pull/264724) — Homebrew
  does not let a new owner take over an existing cask without the original
  owner's blessing.
- [#281420](https://github.com/Homebrew/homebrew-cask/pull/281420) — a mirror
  still has to meet the notability requirements for inclusion. The maintainers
  suggested a personal tap.

The cask downloads from
[case/font-office-code-pro-mirror](https://github.com/case/font-office-code-pro-mirror).
That release tarball is **byte-identical to the original 1.004 release**,
recovered from the Internet Archive and verified against the SHA256 pinned in
the historical Homebrew cask:

```
9bca923d17f6c47a586d8e4567d46ccfa58fb8b8e2247b5ee2a19da1597c58f6
```

The font itself is unmodified. The Reserved Font Name "Office Code Pro" applies
per the SIL Open Font License v1.1 by Nathan Rutzky and Adobe Systems.

## Development

`bin/lint` is the gate. CI runs the same script, so a clean local run means a
clean CI run.

```
./bin/lint
```

It runs `ruby -c`, `brew style`, and `brew audit --cask` over every cask, plus
`shellcheck` and `yamllint`. It audits the **working tree**, not the last
commit: it symlinks this directory into Homebrew's tap directory rather than
using `brew tap`, which clones.

`bin/lint` makes no network call of its own, so it never fetches the release
asset and cannot catch a re-uploaded or deleted tarball. Verify that separately:

```
curl -sL https://github.com/case/font-office-code-pro-mirror/releases/download/1.004/Office-Code-Pro-1.004.tar.gz \
  | shasum -a 256
```

GitHub Actions are pinned to commit SHAs with a trailing `# vX.Y.Z` comment.
Dependabot reads that comment and opens weekly bump PRs; see
[.github/dependabot.yml](.github/dependabot.yml).

It is not fully offline, though. `brew style` and `brew audit` call
`install_bundler_gems!`, which runs `bundle install` against rubygems.org when
Homebrew's style gems are missing or stale, so a fresh machine or a rubygems
outage can fail the pre-commit hook on a network error.

> [!WARNING]
> Never delete and re-upload a release asset. GitHub will not reproduce the
> original bytes, the pinned `sha256` stops matching, and every install fails.

## Moving off GitHub

**Installing** is host-agnostic. Homebrew's two-argument tap form clones from
any URL git understands, and it normalises the local directory name itself, so
a future host need not use the `homebrew-` prefix:

```
brew tap case/tap https://git.example.com/case/homebrew-tap.git
```

Two lines in the cask carry the only host coupling on that path, `url` and
`homepage`. The `sha256` is a claim about bytes, not about a host, so re-host
the identical tarball anywhere and only `url` changes.

**Maintenance** does not move for free. These are GitHub-specific and need a
replacement on another host:

| File                         | What it does          | On another host                      |
| ---------------------------- | --------------------- | ------------------------------------ |
| `.github/workflows/lint.yml` | Runs `bin/lint` in CI | Rewrite as that platform's CI config |
| `.github/dependabot.yml`     | Bumps the action pins | Replace with Renovate or drop        |

`bin/lint` itself is portable and is all the CI config calls, so porting CI is
a config rewrite rather than a logic rewrite. Both action pins become moot,
since the actions only exist on GitHub.

## License

The packaging in this repository is MIT. See [LICENSE](LICENSE). The fonts it
installs are licensed separately under the SIL Open Font License v1.1.
