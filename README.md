# NS8 software repository for danb35 modules

A [NethServer 8](https://www.nethserver.org) software repository for:

- [automx](https://github.com/danb35/ns8-automx): email client autoconfiguration for NS8 mail domains
- [powerdns](https://github.com/danb35/ns8-powerdns): authoritative DNS server with DNSSEC and cluster replication

**[dnshelper](https://github.com/danb35/ns8-dnshelper) has moved to
[NethForge](https://github.com/NethServer/ns8-nethforge)** and is no longer listed here. Every NS8
cluster already has the NethForge repository, disabled by default: in the cluster admin UI, open
**Settings → Software repositories** and enable `nethforge`. An installed dnshelper keeps running and
needs no reinstall; it is offered updates again once NethForge is enabled.

## Using the repository

In the cluster admin UI, open **Settings → Software repositories → Add repository** and enter:

```
https://danb35.github.io/ns8-repomd/
```

Turn on **Testing** for the repository to be offered pre-release tags (for example `1.0.0-dev.1`).

Or from a shell on the leader node:

```
api-cli run add-repository --data '{"name":"danb35","url":"https://danb35.github.io/ns8-repomd/","status":true}'
```

## How it works

Each top-level directory is one module; its name is the module id and the image name under
`ghcr.io/danb35/`. A directory holds:

- `logo.png` (256×256 PNG; `logo.svg` is its source),
- optionally `screenshots/*.png`, shown in name order,
- optionally `metadata.json`; normally it is **not** committed here but fetched at build time from
  the module's repository (`danb35/ns8-<id>`, branch `master`, `ui/public/metadata.json`), so a
  change there shows up on the next build. Commit one here only to override that.

`createrepo.py` (adapted from [NethServer/ns8-repomd](https://github.com/NethServer/ns8-repomd))
reads each image's semver tags from ghcr.io with `skopeo`, keeps the newest stable tag plus any
newer pre-release, reads their labels, and writes `repodata.json`. `pins.yml` can keep an older tag
listed or hide a broken one (same format as upstream). The GitHub Actions workflow builds it and
publishes the directories plus `repodata.json` with GitHub Pages on every push to `main`, every
six hours, and on demand. The build fails, and nothing is published, if any module cannot be indexed.

A new release appears here within six hours. To publish it immediately, run the workflow by hand:

```
gh workflow run repomd.yml -R danb35/ns8-repomd
```

## Adding a module

1. Publish the module image as `ghcr.io/danb35/<id>` (public) with at least one semver tag.
2. Create `<id>/logo.png` (and optionally `<id>/screenshots/`) here and push.

## Building locally

Needs Python 3 with `semver==3.0.1`, `filetype` and `PyYAML`, and `skopeo` on the `PATH`:

```
python3 createrepo.py && python3 -m http.server
```

## License

GPL-3.0-or-later (see `COPYING`), as the upstream NethServer/ns8-repomd scripts it is derived from.
