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
