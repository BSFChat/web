# BSFChat — web

Static landing site for [bsfchat.com](https://bsfchat.com). Plain HTML and CSS,
no JavaScript and no build step.

Preview locally by opening `index.html` directly, or run `python3 -m http.server`
in this directory.

## Deploy

bsfchat.com is served by nginx on the project's own VPS, behind Cloudflare.
The web root, `/var/www/bsfchat.com`, is a git clone of this repo; deploying is
`git pull` on the host. Nothing deploys automatically on push, so a merged
change is not live until someone pulls it there.

Because the web root *is* the clone, every file in the repo is reachable over
HTTP unless nginx says otherwise. See "Hardening" below.

## Download links

The download cards link to
`https://github.com/BSFChat/client/releases/latest/download/<asset>`, which
GitHub redirects to the newest non-prerelease release. The asset names
(`BSFChat-macOS.dmg`, `BSFChat-Setup.exe`, `BSFChat-linux-x86_64.tar.gz`) must
match the `files:` list of the release step in BSFChat/client
`.github/workflows/ci.yml`; rename one there and the link here breaks.

## Hardening (nginx and Cloudflare)

[`nginx-bsfchat.com.conf.example`](nginx-bsfchat.com.conf.example) is the
config the host needs, with how to apply and check it. It must be applied by
hand on the VPS: it stops nginx serving `/.git/`, dotfiles, `*.md` and `docs/`,
installs the custom 404 page, removes the nginx version from responses, makes
the trailing-slash redirect relative (it currently goes to `http://`), and adds
HSTS, a CSP and the other security headers. The end of that file lists the
Cloudflare dashboard settings to change (SSL mode Full (strict), and the Email
Address Obfuscation trade-off).

Until it is applied, the example file itself, `docs/` and the Markdown files are
publicly fetchable from bsfchat.com. None of them contain secrets, and the repo
is public on GitHub anyway, but **never commit anything to this repo that must
not be public** (passwords, demo-account credentials, keys): the web root is the
clone.

`docs/` holds owner notes that used to sit in HTML comments on the legal and
support pages. They are not linked from the site.
