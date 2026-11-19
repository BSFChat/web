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

## Mobile sign-in: app-link files

The mobile app signs in through `https://bsfchat.com/oauth/callback`, a
verified app link, instead of the `bsfchat://` scheme any app could claim. The
OS hands that URL to the app only if this domain vouches for it:

* [`.well-known/assetlinks.json`](.well-known/assetlinks.json) (Android App
  Links) for package `com.bsfchat.app`.
* [`.well-known/apple-app-site-association`](.well-known/apple-app-site-association)
  (iOS) for `<Team ID>.com.bsfchat.app`: `applinks` scoped to
  `/oauth/callback*` (the rest of the site stays in the browser) and
  `webcredentials`, which is what `ASWebAuthenticationSession` checks.
* [`oauth/callback/`](oauth/callback/index.html), the page shown only when
  the OS did not hand the redirect to the app. No script, no external
  resources, no referrer; keep it that way (see the comment in the file).

`com.bsfchat.app` is the release bundle id and Android application id
(`BSFCHAT_BUNDLE_ID` in the client's `CMakeLists.txt`). Debug builds use
`com.bsfchat.app.dev` and are deliberately not listed, so they cannot complete
the verified sign-in.

No mobile app is published yet, so both files carry placeholders. Until they
are filled in, verification fails closed: the callback opens in the browser,
on the page above, and sign-in does not complete. Fill in before the first
mobile release:

| Placeholder | File | Where to get it |
| --- | --- | --- |
| Play app signing SHA-256 (filled in) | `assetlinks.json` | Play Console → the app → Test and release → **App integrity** → App signing → **App signing key certificate** → SHA-256 certificate fingerprint (colon-separated hex, e.g. `14:6D:E9:…`). This is the key Play-installed copies are signed with. |
| Upload key SHA-256 (filled in: `D6:CF:…:AC:90`) | `assetlinks.json` | The CI upload keystore's certificate, as printed by `sign-android.sh` in the CI log (also Play Console → App integrity → **Upload key certificate**). Listed second so that APKs installed straight from CI, which are signed with the upload key rather than Google's, can finish sign-in too. Remove it if the upload key is ever rotated or reset. |
| Apple Team ID (filled in: `V282S7K8TX`) | `apple-app-site-association` | Apple Developer (developer.apple.com/account) → **Membership details** → **Team ID** (10 characters, e.g. `ABCDE12345`). |

Both files must stay valid JSON; check with
`python3 -m json.tool .well-known/assetlinks.json` and the same for the AASA.
After deploying, `adb shell pm get-app-links com.bsfchat.app` should show
`bsfchat.com: verified`.

`apple-app-site-association` has no extension, and Apple requires it to be
served as `application/json` without redirects. The nginx example has a
`location = /.well-known/apple-app-site-association` block with
`default_type application/json` for exactly this; it only takes effect once
that config is applied on the host. Check with the `curl` lines at the top of
the example.
