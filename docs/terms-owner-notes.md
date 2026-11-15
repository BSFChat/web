# Terms of service — owner notes

Moved verbatim on 2026-11-15 from the HTML comment at the top of
`terms/index.html`, where anyone could read it with view-source. This file is
not meant to be public: the nginx snippet in
`nginx-bsfchat.com.conf.example` denies `/docs/` and `*.md`. Until that
is applied on the host it is still fetchable, as the comment was.
(The repo itself is public on GitHub, so treat this as "not advertised",
not secret.)

The text below is unchanged, including its own references to being
public and to the GitHub Pages deploy, which has since been deleted.

```text
  OWNER NOTES — NOT RENDERED, BUT STILL PUBLIC (view-source, and .md files in
  this repo are served). Nothing secret here. These are the unresolved items
  from the visible "Owner: read before publishing" box that was removed on
  26 September 2026, which was live to the public until then, plus what the
  same-day accuracy pass added. Delete each line as it stops being true.

  This page was drafted by an AI assistant. It is not legal advice and no
  lawyer has read it.

  BLOCKS PUBLICATION
  1. RESOLVED 2026-09-26. The contracting party (section 1) is now named:
     Joshua Griffith, trading as BSFChat — the same party the privacy
     policy names as controller. No company was registered; see the note in
     privacy/index.html for the reasoning and for the one event that should
     change it (charging for anything, which makes both stores publish a
     full address).
  2. RESOLVED 2026-09-26 — both mailboxes now exist and are active. This page
     cites both as the route for complaints and vulnerability reports.

  DECIDE
  3. GOVERNING LAW (section 10) now says the United Kingdom. The UK is three
     legal jurisdictions — name England and Wales, Scotland, or Northern
     Ireland before anyone relies on this clause.
  4. THE APP LICENCE (section 2). This page deliberately does NOT try to be an
     end-user licence agreement for the iOS app. The recommendation stands:
     leave the EULA field blank in App Store Connect and accept Apple's
     standard Licensed Application EULA. Reasons are in STORE-LISTING.md. If
     you want a custom EULA instead, this page is not it and you need a lawyer.
  5. AGE (section 3). The old "13, or 16 in the UK or the EEA" was wrong: UK
     GDPR sets 13 via the Data Protection Act 2018, and the 16 is the EU GDPR
     Article 8 default that member states may lower. It now states 13 as the
     floor, says a higher local age governs where one applies, and says plainly
     that no age is verified. Whatever it says must match what you declare in
     App Store Connect and the Play Console, and it must agree with section 11
     of the privacy policy — as of this edit, the two agree.

  VERIFIED, NO ACTION NEEDED FOR NOW
  6. Section 8's "source-available" wording is correct: as of 26 September 2026
     there is no LICENSE file in any of client, server, identity, protocol,
     deploy or web. If you add one, that licence governs and section 8 must be
     updated to say so.
  7. Section 5's CSAM paragraph is deliberately reactive-only. That matches the
     code: there is no scanning, no hash matching against any known-material
     database and no automated detection anywhere in the codebase. If that ever
     changes, this section and the privacy policy must change with it.
```
