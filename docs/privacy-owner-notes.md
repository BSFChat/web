# Privacy policy — owner notes

Moved verbatim on 2026-11-15 from the HTML comment at the top of
`privacy/index.html`, where anyone could read it with view-source. This file is
not meant to be public: the nginx snippet in
`nginx-bsfchat.com.conf.example` denies `/docs/` and `*.md`. Until that
is applied on the host it is still fetchable, as the comment was.
(The repo itself is public on GitHub, so treat this as "not advertised",
not secret.)

The text below is unchanged, including its own references to being
public and to the GitHub Pages deploy, which has since been deleted.

```text
  OWNER NOTES — NOT RENDERED, BUT STILL PUBLIC (view-source, and .md files in
  this repo are served: bsfchat.com/STORE-LISTING.md is fetchable). Nothing
  secret here. These are the unresolved items from the visible "Owner: read
  this before publishing" box that was removed on 26 September 2026 — it was
  live to the public until then — plus what the same-day accuracy pass turned
  up. Delete each line as it stops being true.

  BLOCKS PUBLICATION
  1. RESOLVED 2026-09-26. The data controller (section 1) is now named:
     Joshua Griffith, trading as BSFChat. No company was registered; the
     owner chose to be named rather than incorporate, having decided his
     name may be public but his address and phone number may not. The terms
     name the same party.
     No address for service is published, and that is deliberate. GDPR
     requires the controller's identity and CONTACT DETAILS, not
     specifically a postal address, and support@bsfchat.com is that route.
     WATCH THIS IF ANYTHING IS EVER CHARGED FOR. Monetising changes it in
     both stores at once: Google Play publishes a personal developer's FULL
     address once they monetise (until then, only the country), and Apple
     would treat the account as a DSA trader, publishing address and phone.
     That is the point to register a company and put an agent's registered
     office in front of both — not before.
  2. RESOLVED 2026-09-26 — both mailboxes now exist and are active in
     Proton (MX, SPF, three DKIM selectors and DMARC p=quarantine all
     verified). Left below for the record. This policy
     cites support@ in sections 1, 6, 10, 11, 12 and 15 as the route for
     data-subject requests, account deletion, child-safety reports and
     everything else, and security@ in sections 9 and 15. Both app stores will
     email the support address. The domain's MX records point at Proton so it
     can receive mail; whether these two mailboxes exist cannot be checked
     from outside and was not assumed.

  PROMISES THE CODE DOES NOT KEEP — the policy no longer states these as fact.
  Tighten it back up once they are real.
  3. NGINX LOG RETENTION (section 9). There is no logrotate config in deploy/
     and no retention we enforce; the host uses whatever its distribution
     ships. The old text promised 14 days flat. It now says the intention is
     14 days and that we do not yet enforce a period of our own. Configure it,
     then make it a plain promise again.
  4. BACKUPS (section 9). deploy/backup.sh has no schedule and no pruning — no
     cron, no systemd timer, no age sweep. The old text promised 30 days. It
     now says there is no automatic schedule and no automatic expiry. Note
     deploy/README.md suggests a cron with a 14-day sweep, which is a
     suggestion to the operator, not deployed code, and does not match the 30
     days the page used to claim. Section 10's deletion limits lean on this.
  5. IDENTITY-ACCOUNT DELETION (section 6) has no code path at all — it is a
     manual edit of the SQLite file. The page still promises to do it by hand
     on request. That is honest, but it is you, by hand, forever.
  6. SECTION 12 promises an answer to data requests within 30 days. That is a
     commitment, and it depends on item 2 existing first.

  FIXED IN THIS PASS — what was wrong before, in case it comes back
  7. "No third-party SDKs at all" (section 2) was false: the apps link Qt,
     OpenSSL, nlohmann/json, jwt-cpp and, for voice, libdatachannel, Opus and
     libyuv, plus libaom and openh264 on desktop. The true claim is narrower
     and stronger and is now stated: no analytics, advertising, attribution or
     crash-reporting SDK. Verified from client/cmake/Dependencies.cmake and
     client/CMakeLists.txt. The built APK's classes.dex contains no androidx,
     no Play Services and no Firebase at all, so the page says so.
  8. "Compiled out of the iPhone and Android apps entirely" (section 2) was
     false. src/core/Updater.cpp is compiled and linked on every platform;
     main.cpp only refuses to construct it under
     #if !defined(Q_OS_ANDROID) && !defined(Q_OS_IOS). No check runs on
     mobile, which is what the page now says.
  9. "At most once every six hours" was wrong: there is no persisted throttle.
     It is one check ~30s after each launch plus every 6 hours while running,
     and the settings toggle is only read at startup.
 10. "Hosted on GitHub Pages, so GitHub operates the web server" (section 13)
     was false. bsfchat.com, chat. and id. are all Cloudflare-proxied; the
     origin is not GitHub Pages — same bytes, different ETag and
     Last-Modified, and none of the GitHub Pages response headers. Cloudflare
     and Proton are now documented as processors.
 11. "Runs no JavaScript at all" (section 13) was false. Cloudflare's Email
     Address Obfuscation (Scrape Shield in the dashboard) is ON: it rewrites
     every mailto: on this page and injects
     /cdn-cgi/scripts/.../email-decode.min.js. There is no script in this
     repo to remove, so the claim had to change instead. Turning the feature
     off removes the script and restores plain working mailto: links, at the
     cost of the addresses being scrapeable — your call, but if you change it,
     section 13 must change with it.
 12. UK CHILDREN'S AGE (section 11) said 16 for the UK. Wrong: UK GDPR sets 13
     via the Data Protection Act 2018, and 16 is the EU GDPR Article 8 default
     that member states may lower. Now states 13 with a higher local age
     governing where one applies, says plainly that no age is verified, and
     invites anyone to tell us if we have their jurisdiction wrong.
 13. Smaller corrections, all verified in the code: typing expires at 30s, not
     2.5 minutes (presence is the 2.5-minute one); call signalling does put
     network addresses in the events table for ~2 minutes, so "no IP in any
     table" needed the exception; the registration rate limiter holds an
     address key for up to an hour, not minutes; signing out does NOT clear
     the message cache, attachments or settings; the log file is the live one
     plus three rotated, not three; nginx token-stripping covers the access
     log only, not the error log, and only from the day it was added; a
     deleted account's avatar is unlinked and becomes eligible for collection
     rather than reliably deleted, because the media reaper defaults to
     dry-run and the deploy template does not arm it.

  STILL OPEN, NOT BLOCKING
 14. Link previews (section 8) have no off switch. The page now says so and
     calls it a thing we intend to change. It is the biggest passive
     disclosure in the client: reading a channel makes your device fetch
     third-party URLs, and two Google endpoints for any YouTube link.
 15. The GitHub Pages deploy (.github/workflows/deploy.yml) still runs on
     every push to main and still serves a full copy of this policy at
     https://bsfchat.github.io/web/ . Nothing points at it, so it goes stale —
     an out-of-date privacy policy at a second public URL. Delete the workflow
     and the Pages site, or point something at it.
 16. Section 5 says push is not deployed. The rendered production
     deploy/config/server.toml is gitignored and could not be checked; the
     template ships push.enabled = false with the gateway allowlist commented
     out, no gateway container or vhost exists, and no client registers a
     pusher, which is the part that actually makes the claim safe.
     push.bsfchat.com currently answers with the default nginx welcome page,
     exactly as any nonexistent subdomain does, because DNS is a wildcard.
 17. Identity registration returns a 409 on a duplicate email, which makes it
     an oracle for whether an address has an account. The code notes this
     (identity AccountHandler.cpp, security audit L2). Not mentioned on the
     page; worth closing when email verification lands.

  DO NOT SOFTEN THESE. They are true, they are unusual to admit, and a policy
  that contradicts the code is worse than no policy:
  message history survives account deletion (section 10); nothing on a chat
  server is encrypted at rest (section 9); calls are not end-to-end encrypted
  (section 7); attachment metadata such as photo GPS is not stripped
  (section 4).
```

## 19 November 2026 update (audit 3, fix-first #2): judgement calls left open

The policy, support page and store documents were brought in line with
0.0.49 / server 0.0.53. What was changed is listed in privacy section 14. The
statements below are facts that were checked against the code; the
*decisions* are yours:

1. **Apple and Google: processors or independent controllers?** Section 13
   names them as companies "in the path" and says the data leaves the UK/EEA,
   but deliberately does not call them processors. Firebase has Data
   Processing Terms you can accept in the Firebase console; APNs has no
   equivalent. Decide whether to accept the Firebase terms and list Google as
   a processor, and how to describe Apple.
2. **The gateway and other people's servers.** When a self-hosted server
   allows push.bsfchat.com (the deploy template now does by default,
   `PUSH_ENABLED=true`), we process notices about its users: room ids with
   that server's name, event ids, unread counts, and names if they opted in.
   Section 5 says so plainly. Not decided: whether we are controller or
   processor for that, and which legal basis section 12 should give (it only
   covers "our servers").
3. **The hosting provider is not named.** Sections 9 and 13 say "object
   storage with our hosting provider" (Contabo, S3-compatible). Decide whether
   to name Contabo, and whether it belongs in the processor list next to
   Cloudflare and Proton. The bucket is still named after another project
   (audit 3, production).
4. **Section 14 promised an in-app notice before push took effect.** Nothing
   in the client shows one (searched the QML for a first-run or policy
   notice). Section 14 now describes push as a change that has arrived, and
   does not claim a notice was shown. Decide whether to add the notice or
   accept the gap.
5. **Play Data Safety.** Device or other IDs is answered collected *and
   shared* for the FCM token, although Google's service-provider exemption
   may make it "not shared". The names tier, which sends another user's
   display name and the room name to Google, is not declared as sharing Name.
   Both are written up in `PLAY-SUBMISSION.md` §3.3.
6. **Things the text depends on that live outside this repo:**
   - relay credentials "expire after a day" (section 7) is the server default
     and the new deploy template; production `server.toml` files must not
     still set `turn_ttl = 3600`;
   - "the gateway has no database … does not keep it" (section 5) is true of
     Sygnal; confirm its log config does not record request bodies or full
     push keys;
   - "the Android app contains Firebase Cloud Messaging" is true of builds
     made with the four `FCM_*` CI variables set (`FCM_API_KEY` was still
     pending in client `docs/push-notifications.md`); without them the APK
     has no Firebase and the Android text overstates;
   - iOS push needs the APNs key on the gateway (its iOS block is commented
     out), and UAT/test have no `[push]` section, so store reviewers get no
     push;
   - section 13 says the site serves no script, which is true only once
     Cloudflare's Email Address Obfuscation is off for the zone;
   - section 13 says every server in section 1 is behind Cloudflare
     (push.bsfchat.com, uat, test included); the DNS names are proxied, but
     confirm.
7. **Client copy, not this repo:** the "Show sender and room names" setting
   says that with it off "Google/Apple learn nothing about who wrote or
   where". The room id includes the server's name, so "where" is slightly
   overstated (the policy says so correctly).
