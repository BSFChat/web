# BSFChat — store submission pack

**Not published.** This file lives in the `web` repo for convenience, but it is
not part of the site and is not linked from anywhere. It is working material for
filling in App Store Connect and the Google Play Console.

**Drafted by an AI assistant on 2026-09-23** by reading `client`, `server`,
`identity` and `deploy`. Everything factual in it was checked against code, and
the checks are cited so you can re-run them. Everything that is a *decision* —
pricing, entity name, age rating appeal, whether to ship iOS without voice — is
yours and is marked.

Companion pages, all on branch `docs/store-submission` in this repo:

| Field the stores ask for | URL |
| --- | --- |
| Privacy policy URL (**both stores, mandatory**) | `https://bsfchat.com/privacy` |
| Support URL (**both stores, mandatory**) | `https://bsfchat.com/support` |
| Marketing URL (Apple, optional) | `https://bsfchat.com` |
| Terms of use / EULA URL (Apple, optional) | `https://bsfchat.com/terms` |

---

## 0. Read this first: what will stop you tomorrow

Ordered by how likely it is to cost you a review cycle.

### 0.1 iOS has no voice. None. (highest risk)

You told me "iOS voice works but echo cancellation is not yet implemented."
**The code says otherwise, and the difference matters for your listing.**

`client/CMakeLists.txt:24-25`

```cmake
if(IOS)
    option(BSFCHAT_ENABLE_VOICE "Enable voice chat (requires libdatachannel + opus)" OFF)
```

`src/main.cpp:274` gates the entire voice wiring on
`defined(BSFCHAT_VOICE_ENABLED) && !defined(Q_OS_IOS)`, and the iOS CI job
(`.github/workflows/ci.yml:1420`) does not pass `-DBSFCHAT_ENABLE_VOICE=ON`.
Nothing in the tree configures `AVAudioSession`, and `UIBackgroundModes` is
deliberately absent.

**The first TestFlight build is a text-chat client.** If the App Store
description, subtitle, keywords or screenshots mention voice, video or screen
sharing, that is a **guideline 2.3.1 (accurate metadata)** rejection, and it is
the kind reviewers catch reliably because they will look for the button.

The iOS copy below therefore never mentions voice. Do not add it back.

### 0.2 Echo cancellation — the real picture

There is **no software AEC anywhere in the client**, on any platform. Grepped
`AEC|VoiceProcessingIO|kAudioUnitSubType_VoiceProcessing|AcousticEcho|APM|noise suppress`;
every hit is a comment explaining the absence, and `src/voice/VoiceGain.h:25-60`
states it outright.

| Platform | Voice | Echo cancellation |
| --- | --- | --- |
| macOS / Windows / Linux | yes | none — headphones required in practice |
| Android | yes | platform/hardware AEC only, via `MODE_IN_COMMUNICATION` (`src/voice/AndroidAudioRouting.h`), **not yet verified on a real device** |
| iOS | **not built** | n/a |

Consequence for the Play listing: do not claim "crystal-clear" or
"echo-free" voice. The copy below says voice works and leaves it there.

### 0.3 Blocking, reporting and account deletion are on unmerged branches

You said these were on `server` main. They are not.

- `server` main is `295156f`. The feature is `d5d0fe6` on **`feat/ugc-safety`**.
- `client` main is `ada5998`. The UI is `f7b806e` on **`feat/ugc-safety-ui`**.
- The integration state that has everything (plus Play/App Store prep and the
  mobile UI work) is the worktree **`wt/android-verify`**, detached at `2c8c857`
  = `origin/feat/mobile-ui-touch`.

**Build the submission from that integration state, not from main.** Apple
guideline 1.2 and Google's UGC policy both make block+report a hard gate for a
chat app, and 5.1.1(v) makes in-app account deletion a hard gate for any app
with accounts. Submitting a `main` build is an automatic rejection on all three.

### 0.4 Google Play closed-testing rule

A Google Play developer account registered as an **individual after 13 November
2023** must run a closed test with **at least 12 testers opted in for 14
continuous days** before it can apply for production access. An organisation
account, or an individual account older than that, is exempt.

**You must check which one yours is**, in Play Console → Setup → Advanced
settings, because it decides whether "closed testing tomorrow" means "production
in two weeks" or "production whenever you like". Nothing in the code tells me
this and I have not looked at your account.

### 0.5 Things you must create before you can submit

| # | Thing | Why |
| --- | --- | --- |
| 1 | `support@bsfchat.com` mailbox or alias | Both stores email it. Cited on every page. A bounce is a rejection. |
| 2 | `security@bsfchat.com` mailbox or alias | Cited in the policy and terms. |
| 3 | **Demo account** on `chat.bsfchat.com` with a **password** (not OIDC) | Review notes hand it to the reviewer. See §5. |
| 4 | **A second account**, and some seeded conversation | So the reviewer has someone to block and something to report. Apple reviewers routinely fail 1.2 because there is nothing to demonstrate against. |
| 5 | Screenshots — iPhone 6.9" and 6.5"; Play phone set | Apple requires at least one 6.9" set. **Do not screenshot a voice channel for iOS.** |
| 6 | 1024×1024 App Store icon, 512×512 Play icon, 1024×500 Play feature graphic | No alpha on the Apple icon. |
| 7 | Legal entity name + jurisdiction | For `/terms`, and for the Play "Developer name". |
| 8 | Merge the branches in §0.3 and cut a build | |

### 0.6 Smaller things that will bite

- **Do not name Discord, Slack, TeamSpeak or Matrix-the-brand in Apple
  keywords.** Competitor trademarks in keywords are a 2.3.7 rejection. The
  keyword list below is clean; if you add to it, keep it clean. The *description*
  may describe the app as "Discord-style" as plain comparative language, but the
  safest copy avoids it, and the copy below does.
- **`ITSAppUsesNonExemptEncryption` is already `true`** in `ios/Info.plist.in:157`,
  deliberately (see the long justification at `:127-156`). That means App Store
  Connect will want the **French encryption declaration** for distribution in
  France, and will stop nagging once Apple issues you a code you can paste back
  as `ITSEncryptionExportComplianceCode`. This is correct as-is; do not "simplify"
  it to `false`.
- **Three Android foreground-service types** (`microphone`, `dataSync`,
  `mediaProjection`) each need a **Play Console declaration plus a demo video**
  showing the feature in use. Budget time for three short screen recordings.
- **iOS minimum version is unpinned.** No `IPHONEOS_DEPLOYMENT_TARGET` is set
  anywhere; CI deliberately lets Qt 6.10.3 choose and prints it. Read the real
  number off the CI log line "Minimum iOS version (chosen by the Qt toolchain)"
  before you type it into the listing. Do not guess.
- **iPhone only.** `XCODE_ATTRIBUTE_TARGETED_DEVICE_FAMILY "1"`
  (`CMakeLists.txt:498`). There is no iPad layout, so set the listing to iPhone
  only rather than letting it ship an upscaled iPad build.
- **No LICENSE file exists in any BSFChat repo.** The site links to source. Do
  not write "open source" in either listing until one exists — the copy below
  says "source is public", which is true.
- **No push notifications on iOS.** Nothing arrives while the app is closed.
  Worth a line in the description so it does not read as a bug; the copy below
  has one.

---

## 1. Apple App Store

### 1.1 Fields

| Field | Value | Limit |
| --- | --- | --- |
| **App Name** | `BSFChat` | 30 |
| *(alternative, if you want keyword weight)* | `BSFChat: Self-Hosted Chat` | 30 |
| **Subtitle** | `Self-hosted chat, no tracking` | 30 |
| **Category** | Primary: **Social Networking**. Secondary: **Productivity** | |
| **Age rating** | expect **12+** — see §6.3 | |
| **Price** | Free, no in-app purchases (there is no purchase code in the tree) | |
| **Privacy Policy URL** | `https://bsfchat.com/privacy` | |
| **Support URL** | `https://bsfchat.com/support` | |
| **Marketing URL** | `https://bsfchat.com` | |
| **EULA** | **Leave blank — accept Apple's standard EULA.** See §1.5 | |
| **Copyright** | `2026 <legal entity>` | |
| **Bundle ID** | `com.bsfchat.app` | |
| **Version** | `0.0.48` (from tag `0.0.48-rc.2`; the suffix is stripped by `cmake/Version.cmake:59`) | |

### 1.2 Promotional text (170 max — editable without review) · **161 chars**

```
Chat that does chat. Run the server yourself, sign in once, talk to your
people. No analytics, no crash reporting, no ads, no third-party SDKs. Check
the source.
```

### 1.3 Description (4000 max) · **2742 chars**

```
BSFChat is chat without the bullshit.

No ads. No analytics. No crash reporting. No tracking pixels, no advertising
identifiers, no third-party SDKs of any kind. Not "anonymised" analytics — none
at all. The source is public, so this is a claim you can check rather than one
you have to believe.

And no servers of ours in the middle. BSFChat is self-hosted: your messages live
on a server that you or your community runs. That might be a rented box, a
machine in an office, or a Raspberry Pi in a spare room. We do not have a copy.
We cannot read your channels, and on most servers we do not know they exist.

WHAT YOU GET

• Channels and categories, organised how you want them
• Private channels with real role-based permissions
• Direct messages
• Threads, replies, edits, reactions, pinned messages and mentions
• File attachments and inline images
• Full-text search across your history
• Multiple servers in one app, each with its own account
• Themes, accent colours and density that stay out of your way

ONE ACCOUNT, EVERY SERVER

Sign in once with a BSFChat ID and every server you have joined comes back.
Sign-in uses standard OpenID Connect in Apple's secure sign-in sheet — never an
embedded browser that could read what you type. Or skip it entirely: a server
can issue plain accounts with no identity provider at all.

YOU ARE NOT THE PRODUCT

There is no paid tier, no upsell, no "upgrade for larger uploads", no
artificially degraded free experience, no engagement mechanics and no dark
patterns. There is nothing to buy in this app because there is nothing we are
holding back.

SAFETY

Block anyone, from their profile card, on any server. They are not told, and
their messages and invitations stop reaching you everywhere you sign in. Report
a message or a user to that server's administrators. Delete your account from
inside the app — read the privacy policy first, because messages you have
already sent stay in their channels, and we would rather tell you that now.

BEFORE YOU DOWNLOAD — HONEST LIMITS

• You need a BSFChat server to connect to. The app is a client. If you do not
  have one, you can join the public server at chat.bsfchat.com, or run your own
  from github.com/BSFChat in about five minutes.
• Voice and video are not available on iPhone yet. They work on Mac, Windows,
  Linux and Android; the iPhone app is text for now.
• There are no push notifications yet. BSFChat shows you new messages while it
  is open. Nothing is handed to Apple's notification servers because there is
  nothing to hand.
• Messages are not end-to-end encrypted. Traffic is protected by TLS and your
  server holds your history. We will not claim otherwise.

bsfchat.com/privacy · bsfchat.com/support · github.com/BSFChat
```

### 1.4 Keywords (100 max, comma-separated, no spaces) · **95 chars**

```
selfhosted,selfhost,privacy,chat,messaging,community,team,server,nologging,noads,guild,irc,foss
```

Rules applied: no competitor trademarks (2.3.7), no repetition of words already
in the name or subtitle (Apple indexes those separately, so repeating wastes
characters), singular forms only (Apple stems automatically).

### 1.5 EULA: use Apple's standard one — recommended

**Recommendation: leave the EULA field in App Store Connect blank, which applies
Apple's
[standard Licensed Application End User License Agreement](https://www.apple.com/legal/internet-services/itunes/dev/stdeula/).**

Reasons, in order of weight:

1. **A custom EULA is reviewed; Apple's is not.** Schedule 1 of the Developer
   Agreement requires a custom EULA to meet or exceed Apple's minimum terms. If
   yours falls short anywhere, that is a review cycle spent on legal text rather
   than on the app.
2. **It already contains the Apple-specific clauses that are easy to get
   wrong** — Apple as a third-party beneficiary entitled to enforce it, Apple's
   disclaimer of warranty and of maintenance obligations, the product-claims and
   IP-claims allocation, US export and government-end-user terms. A hand-written
   EULA that omits any of them is non-compliant, and there is no upside to
   rewriting them.
3. **Guideline 1.2 does not require a custom EULA.** What it requires is
   mechanisms in the app: filtering, reporting, blocking, and published contact
   information. You have all four. The "terms of use with zero tolerance for
   objectionable content" that reviewers look for can live in ordinary published
   terms, which is exactly what `/terms` is.
4. **You are not selling anything**, so there is no payment, subscription,
   refund or licence-tier language that Apple's EULA fails to cover.

So: Apple's EULA governs the licence to the iOS binary; `bsfchat.com/terms`
governs acceptable use and the service we run. The terms page says this
explicitly and defers to Apple's EULA where the two overlap, so there is no
conflict for a reviewer to find. Put `https://bsfchat.com/terms` in the optional
"Terms of Use (EULA)" **URL** field if you like — that is a link, not a custom
EULA, and does not trigger the review above.

*(Google Play has no equivalent requirement: the Play Terms of Service apply
automatically and a custom EULA is optional there too.)*

---

## 2. Google Play

Android **does** have voice, video and screen sharing, so this copy differs from
Apple's on purpose. Do not unify them.

### 2.1 Fields

| Field | Value | Limit |
| --- | --- | --- |
| **App name** | `BSFChat` | 30 |
| *(alternative)* | `BSFChat: Self-Hosted Chat` | 30 |
| **Category** | Communication | |
| **Tags** | Chat, Messaging, Social | |
| **Content rating** | expect **Teen / PEGI 12** — see §6 | |
| **Contains ads** | **No** | |
| **In-app purchases** | **No** | |
| **Privacy Policy URL** | `https://bsfchat.com/privacy` | |
| **Support email** | `support@bsfchat.com` | |
| **Support website** | `https://bsfchat.com/support` | |
| **Application ID** | `com.bsfchat.app` — **permanent, verify before first upload** | |
| **versionCode / versionName** | `4702` / `0.0.48-rc.2` | |
| **minSdk / targetSdk** | 28 (Android 9) / 36 | |

### 2.2 Short description (80 max) · **78 chars**

```
Self-hosted chat for teams and communities. No ads, no tracking, no telemetry.
```

### 2.3 Full description (4000 max) · **2951 chars**

```
BSFChat is chat without the bullshit.

No ads. No analytics. No crash reporting. No advertising ID, no Firebase, no
Google Play Services dependency, no third-party SDKs of any kind. Not
"anonymised" analytics — none at all. The source is public, so you can check
rather than trust.

And no servers of ours in the middle. BSFChat is self-hosted: your messages live
on a server you or your community runs — a rented box, a machine in the office,
a Pi in a cupboard. We do not have a copy, we cannot read your channels, and on
most servers we do not know they exist.

WHAT YOU GET

• Channels and categories, organised how you want them
• Private channels with real role-based permissions
• Direct messages
• Voice channels over Opus, peer-to-peer where your network allows it
• Camera video in a voice channel
• Screen sharing from your phone
• Threads, replies, edits, reactions, pinned messages and mentions
• File attachments and inline images, plus "Share to BSFChat" from other apps
• Full-text search across your history
• Multiple servers in one app, each with its own account
• Background notifications on your own device — no Firebase, nothing sent to
  Google

ONE ACCOUNT, EVERY SERVER

Sign in once with a BSFChat ID and every server you have joined comes back.
Sign-in is standard OpenID Connect in your own browser — never an embedded web
view. Or skip it: a server can issue plain accounts with no identity provider at
all.

VOICE THAT DOES NOT PHONE HOME

Voice and video go directly between participants when the network allows, and
through a relay when it does not. A direct call means the other participants'
devices can see your IP address — that is how direct connections work
everywhere — so there is a "Hide my IP address" setting that forces every call
through the relay. If it is on and no relay is available, the call is refused
rather than quietly connected anyway.

YOU ARE NOT THE PRODUCT

No paid tier. No upsell. No "upgrade for bigger uploads". No engagement
mechanics, no dark patterns, nothing to buy, because there is nothing we are
holding back.

SAFETY

Block anyone from their profile card, on any server. They are not told, and
their messages and invitations stop reaching you on every device you sign in on.
Report a message or a user to that server's administrators. Delete your account
from inside the app — read the privacy policy first, because messages you have
already sent stay in their channels, and we would rather say so now.

HONEST LIMITS

• You need a BSFChat server to connect to. Join the public one at
  chat.bsfchat.com, or run your own from github.com/BSFChat in about five
  minutes.
• Messages are not end-to-end encrypted. Traffic is protected by TLS and your
  server holds your history. We will not claim otherwise.
• Echo cancellation relies on your device's own. Use headphones on speakerphone
  and everyone will thank you.

bsfchat.com/privacy · bsfchat.com/support · github.com/BSFChat
```

---

## 3. Google Play Data Safety form

Google's form asks, per data type: **Collected?** (leaves the device to a server
you or a third party control), **Shared?** (goes to a *third party*),
**Processed ephemerally?**, **Required or optional?**, **Purpose?**

The central judgement call, which you should be ready to defend: **for a
self-hosted app, "collected" is answered against the server the user chooses,
not against us.** Google's definition is device-leaves-to-server, and it does
not care who owns the server — so "we do not run it" is not a reason to answer
No. Answering Yes and explaining in the description is both accurate and the
safer posture; answering No because "we never see it" is how apps get
enforcement actions.

### 3.1 Answers

| Data type | Collected | Shared | Purpose | Required | Reasoning / evidence |
| --- | --- | --- | --- | --- | --- |
| **Name** (display name, nickname) | **Yes** | No | App functionality | Required | `users.display_name`, `users.nickname` — `server/src/store/SqliteStore.cpp:445`, `Migrations.cpp:831`. Shown to other members. |
| **Email address** | **Yes** | No | App functionality, Account management | **Optional** | `accounts.email` is nullable and never verified — `identity/src/store/IdentityStore.cpp:106-117`, `AccountHandler.cpp:427-448`. Nothing is ever sent to it; the service has no mail transport at all. Only collected if the user signs in with a BSFChat ID and types one. |
| **User IDs** | **Yes** | No | App functionality, Account management | Required | Matrix user ID and a server-issued device ID (`access_tokens.device_id`). Not a device or advertising identifier. |
| **Messages — in-app messages** | **Yes** | No | App functionality | Required | `events.content` in plaintext, plus a second copy in the FTS index (`Migrations.cpp:592`). This is the app. |
| **Photos / Videos** | **Yes** | No | App functionality | Optional | Only files the user attaches. Note EXIF is not stripped (`storage/LocalStorage.cpp:33`) — the upload is byte-for-byte. |
| **Files and docs** | **Yes** | No | App functionality | Optional | Any MIME type via the system picker, and the `ACTION_SEND` share target. |
| **Audio — voice or sound recordings** | **Yes** *(see note)* | No | App functionality | Optional | Live voice only. **Nothing is recorded or stored.** Tick "collected" because the stream leaves the device; use the free-text to say it is real-time and never retained. If Play's form offers "processed ephemerally", use it — that is the accurate box. |
| **Photos — camera / live video** | **Yes** *(ephemeral)* | No | App functionality | Optional | Same reasoning, for camera video and screen share. Never recorded. |
| **App activity — other actions** | **Yes** | No | App functionality | Required | Reactions, read positions, block list, notification prefs. Server-stored so they follow the account. |
| **App info and performance — crash logs** | **No** | No | — | — | **No crash reporter exists.** Grepped `crashlytics/sentry/breakpad/crashpad/bugsnag` across the tree: zero. |
| **App info and performance — diagnostics** | **No** | No | — | — | No telemetry. `src/voice/video/VideoSendStats.h:44` — every number stays in-process. |
| **Device or other IDs** | **No** | No | — | — | No IDFA, no Android ID, no `getSerial`, no `identifierForVendor`. The only `QUuid::createUuid()` uses are ephemeral request/transaction IDs. |
| **Location (approximate or precise)** | **No** | No | — | — | No location permission, no geolocation, no IP-to-location. |
| **Personal info — address, phone, race, politics, sexual orientation, religion** | **No** | No | — | — | No such column exists in either schema. |
| **Financial info** | **No** | No | — | — | No payment code, no IAP, no billing library. |
| **Health and fitness** | **No** | No | — | — | |
| **Contacts / Calendar / SMS / Call logs** | **No** | No | — | — | No permission requested, no API touched. |
| **Web browsing history** | **No** | No | — | — | No WebView anywhere (`qml/mobile/MobileMain.qml:719`). |
| **Installed apps** | **No** | No | — | — | |
| **Photos — photo library** | **No** | — | — | — | No photo-library permission on either platform; attachments come from the file picker. |

### 3.2 Security practices section

| Question | Answer | Reasoning |
| --- | --- | --- |
| Is data encrypted in transit? | **Yes** | TLS to the server; DTLS/SRTP for call media. |
| Can users request data deletion? | **Yes** | In-app **Delete account**, plus `support@bsfchat.com`. |
| Do you follow the Play Families policy? | **No** (app is not for children) | |
| Has the app been independently security-reviewed? | **No** | Two internal audits exist (47 findings, `RC-SECURITY-PLAN.md`), but they were not independent. Do not tick this. |
| Is data shared with third parties? | **No** | No SDK, no ad network, no analytics vendor, no push provider. |

**Do not tick "data is encrypted at rest."** Google does not ask this directly,
but if any free-text invites the claim, resist it: the server database is
plaintext (`server/config/bsfchat-server.example.toml:52-55` says so in as many
words).

### 3.3 Suggested free-text for the "data collection" explanation

```
BSFChat is self-hosted. Messages, attachments and profile details are sent to a
BSFChat server chosen by the user, which in most cases is operated by the user
or their own community rather than by us. The developer operates one optional
public server and one optional sign-in provider; neither is required to use the
app. The app contains no analytics, no crash reporting, no advertising and no
third-party SDKs, and sends nothing to the developer about how it is used.
Voice, video and screen-share streams are transmitted live between participants
and are never recorded or stored.
```

---

## 4. App Store privacy "nutrition label"

App Store Connect → App Privacy. For each type: **is it collected**, is it
**linked to the user's identity**, and is it **used to track** (Apple's "track"
means combining with third-party data for advertising or data brokerage —
answer **No** everywhere, truthfully).

Apple's definition of "collect" is data **transmitted off the device and
retained beyond the time needed to service the request**, which lets you exclude
live media streams honestly, unlike Google's.

### 4.1 Answers — iOS build (remember: no voice)

| Apple data type | Collected | Linked to user | Used to track | Purpose | Reasoning |
| --- | --- | --- | --- | --- | --- |
| **Contact Info › Name** | **Yes** | Yes | No | App Functionality | Display name and nickname, stored on the server, shown to others. |
| **Contact Info › Email Address** | **Yes** | Yes | No | App Functionality | **Optional and unverified**, only via BSFChat ID. Apple has no "optional" flag — declare it; the policy explains. |
| **Contact Info › Phone, Physical Address, Other** | No | — | — | — | Never requested; no column exists. |
| **User Content › Photos or Videos** | **Yes** | Yes | No | App Functionality | Attachments and avatars. |
| **User Content › Audio Data** | **No** | — | — | — | **iOS build has no voice** (`CMakeLists.txt:24-25`). If you later ship voice on iOS, this becomes Yes. |
| **User Content › Customer Support** | **Yes** | Yes | No | App Functionality | Report reasons, plus a stored copy of the reported message. |
| **User Content › Other User Content** | **Yes** | Yes | No | App Functionality | Messages, reactions, threads, pins. |
| **Identifiers › User ID** | **Yes** | Yes | No | App Functionality | Matrix user ID + server-issued device ID. |
| **Identifiers › Device ID** | **No** | — | — | — | No IDFA/IDFV/vendor ID is ever read. |
| **Usage Data › Product Interaction, Advertising Data, Other** | **No** | — | — | — | No analytics of any kind. |
| **Diagnostics › Crash Data, Performance Data, Other** | **No** | — | — | — | No crash reporter, no performance telemetry. |
| **Purchases** | No | — | — | — | No IAP. |
| **Financial Info** | No | — | — | — | |
| **Location** | No | — | — | — | |
| **Health & Fitness** | No | — | — | — | |
| **Contacts** | No | — | — | — | No contacts permission. |
| **Browsing History / Search History** | **No** | — | — | — | In-app search runs against the user's own server and is not retained as a history. No web browsing. |
| **Sensitive Info** | No | — | — | — | |

**"Do you or your third-party partners use data for tracking?" → No.** There are
no third-party partners. This also means you do **not** need App Tracking
Transparency, and `ATTrackingManager` correctly appears nowhere in the tree.

### 4.2 The awkward one: microphone and camera usage strings with no voice

`ios/Info.plist.in` declares `NSMicrophoneUsageDescription` and
`NSCameraUsageDescription` even though voice is compiled out. This is
**deliberate and load-bearing**, not an oversight: the file's own header explains
that Qt decides at *configure* time whether to link the
`QDarwinMicrophonePermission` / `QDarwinCameraPermission` backends by running
PlistBuddy over the named plist, so removing the keys would permanently break
permission prompts when voice does land.

Reviewers do occasionally query a usage string for hardware the app never asks
for. It is not a rejection in itself — declaring a string is not the same as
requesting access, and the app never triggers a prompt. **§5.1 of the review
notes pre-empts the question**; leave the keys alone.

Note the asymmetry this creates and be ready for it: the **Play** Data Safety
form says audio is collected (Android has voice) and the **Apple** label says it
is not (iOS does not). That is correct, not inconsistent. Revisit the Apple
label the day iOS voice ships.

---

## 5. App review notes

### 5.1 Apple — App Review Information → Notes (4000 max) · **3904 chars**

```
Thank you for reviewing BSFChat.

WHAT THIS APP IS

BSFChat is a client for self-hosted chat servers, as an email client is a
client for mail servers. Messages live on a BSFChat server run by the user or
their community. We operate one public server; the demo account is on it.

There is no in-app registration on purpose: an account belongs to a server, so
accounts are made by that server's operator. Please use the account below.

DEMO ACCOUNT

  Server:   https://chat.bsfchat.com
  Username: <owner: fill in>
  Password: <owner: fill in>

HOW TO SIGN IN

1. Launch. The "Add a server" screen appears.
2. Tap "Show manual server entry", below the OR divider.
3. Enter https://chat.bsfchat.com and tap "Check".
4. Tap "Use password instead"; enter the username and password above.
5. You land in a server with seeded channels and conversation.

("Sign in with BSFChat ID" is our own OpenID Connect provider, id.bsfchat.com:
first-party, not a social login, so 4.8 does not apply and no Sign in with
Apple is required. It uses ASWebAuthenticationSession, not a web view.)

GUIDELINE 1.2 - USER-GENERATED CONTENT

* Block a user: tap any person's name or avatar on a message, or in the member
  list, to open their profile card, then tap the lock button ("Block - you stop
  seeing their messages. They are not told."). Immediate and server-side: their
  messages and invitations stop reaching the blocker on every device. Manage at
  "..." menu > Your profile > Blocked accounts, with Unblock per row.
* Report content: long-press any message you did not send > "Report message...",
  or "Report user..." for the person. Free-text reason, plus an "also block
  them" checkbox ticked by default. Reports go to that server's administrators
  with a retained copy of the message. The bolt button on a profile card
  reports the user directly.
* Moderation: each server's administrators can remove content and ban accounts.
  On the server we operate we act on reports ourselves, within 24 hours.
* Published contact info: https://bsfchat.com/support. Our terms at
  https://bsfchat.com/terms state zero tolerance for abusive content and users.

GUIDELINE 5.1.1(v) - ACCOUNT DELETION

"..." menu > Your profile > bottom of the list > "Delete account" (below "Log
out"). It lists what is deleted and requires the username and password to
confirm.

One thing we disclose rather than hide: deletion destroys the credentials,
erases the profile and removes the person from every channel, but messages they
already sent remain in those channels. A group conversation belongs to everyone
in it. The dialog says so before confirming ("Messages you have sent stay in
their channels"), and https://bsfchat.com/privacy#deletion explains it.

WHAT THIS BUILD DOES NOT DO

* No voice or video on iOS in this release. They exist on macOS, Windows, Linux
  and Android but are not compiled into the iOS target, and the App Store
  description does not mention them.
* No push notifications: no APNs or PushKit, and no UIBackgroundModes, all
  deliberate. The app shows new messages while it is open.
* Info.plist declares microphone and camera usage strings although this build
  requests neither. The Qt build system decides at configure time whether to
  link the permission backends by reading those keys, so removing them would
  break permissions when voice ships. No prompt is ever triggered.
* No in-app purchases, advertising, analytics, crash reporting, or third-party
  SDKs of any kind.

OTHER

* Privacy policy: https://bsfchat.com/privacy - explicit that messages are not
  end-to-end encrypted.
* ITSAppUsesNonExemptEncryption is true: we link OpenSSL we cross-compile
  ourselves, not only Apple's cryptography.
* NSLocalNetworkUsageDescription is declared because self-hosted servers often
  live on a LAN. Connecting to chat.bsfchat.com will not trigger the prompt.

support@bsfchat.com reaches a human.
```

### 5.2 Google Play — App content / "Instructions for reviewer"

```
BSFChat is a client for self-hosted chat servers. Messages live on a server the
user or their community runs; we operate one public server, used by the demo
account below. Accounts are not created in the app, so please use these
credentials rather than registering.

  Server:   https://chat.bsfchat.com
  Username: <owner: fill in>
  Password: <owner: fill in>

Sign in: launch > "Show manual server entry" > enter the server URL > "Check" >
"Use password instead" > username and password.

USER-GENERATED CONTENT CONTROLS (Play UGC policy)

  • Block: tap a person's name or avatar > profile card > lock button.
    Server-side, immediate, follows the account across devices. Manage at
    ... > Your profile > Blocked accounts.
  • Report: long-press a message > "Report message..." (or "Report user...").
    Free-text reason, "also block them" ticked by default. Reports go to that
    server's administrators with a retained copy of the message.
  • Moderation: server administrators can remove content and ban accounts. On
    the server we operate we respond to reports within 24 hours.
  • Published terms with a zero-tolerance position: https://bsfchat.com/terms

ACCOUNT DELETION

  In-app at ... > Your profile > Delete account (username and password
  confirmation). Also available by email to support@bsfchat.com. Web route for
  the Play account-deletion declaration: https://bsfchat.com/support

FOREGROUND SERVICES — why each one exists

  • microphone (VoiceService): keeps the process alive during a voice call so
    Android does not kill it when the user switches apps mid-conversation.
  • dataSync (SyncService): maintains the connection to the user's chosen server
    so messages can raise a local notification while the app is backgrounded.
    There is no Firebase Cloud Messaging in this app — notifications are
    generated on the device, and no notification content is sent to Google.
  • mediaProjection (MediaProjectionService): required by the platform for
    screen sharing into a voice channel. Started only after the user accepts
    the system screen-capture consent dialog.

NO ADS, NO ANALYTICS, NO THIRD-PARTY SDKS

  The app contains no advertising, no analytics, no crash reporting, no
  advertising identifier and no Google Play Services dependency. Android Auto
  Backup is disabled (allowBackup="false") so that sign-in tokens are never
  copied to Google Drive.

Privacy policy: https://bsfchat.com/privacy
Support: https://bsfchat.com/support
```

### 5.3 Play "Data deletion" declaration

Play asks separately for an account-deletion route reachable **from the web**,
not only in-app.

| Field | Answer |
| --- | --- |
| Can users request account deletion? | Yes |
| In-app path | `... menu → Your profile → Delete account` |
| Web URL | `https://bsfchat.com/support` |
| Is any data retained after deletion, and why? | Yes — messages the user sent remain in the conversations they were part of, along with the display name recorded in historical membership events, because a group conversation belongs to all its participants. Moderation and audit records are retained. Backups age out within 30 days. Explained at `https://bsfchat.com/privacy#deletion`. |

---

## 6. Play content rating (IARC) questionnaire cheat-sheet

Category: **Social / Communication** (not Game — do not pick Game, the whole
questionnaire changes).

| Question | Answer | Reasoning |
| --- | --- | --- |
| Does the app contain violence? | **No** | The app ships no content. IARC asks about content *you* provide. |
| Sexual content or nudity? | **No** | Same. |
| Profanity or crude humour? | **No** | Same — the UGC question below is where user behaviour is captured. |
| References to drugs, alcohol or tobacco? | **No** | |
| Gambling, simulated or real? | **No** | |
| Horror or fear themes? | **No** | |
| **Does the app allow users to interact or exchange content with other users?** | **Yes** | The entire app. Text, files, images, and on Android live voice, video and screen share. This is the single answer that drives the rating. |
| **Can users share their current location with others?** | **No** | No location feature. Be ready to defend this: peer-to-peer calls expose an IP address to other participants, and an IP is coarsely geolocatable. IARC's question is about a *location-sharing feature*, and there is none — the user cannot send their location and the app never reads it. Disclosed in the privacy policy under Voice and video, with a "Hide my IP address" setting as the mitigation. Do not tick this box. |
| **Can users share personal information with other users?** | **Yes** | Free-text messages and arbitrary file uploads. A user can type anything. |
| Does the app let users purchase digital goods? | **No** | No IAP, no billing library, no payment code. |
| Does the app contain ads? | **No** | |
| Does the app provide unrestricted access to the internet (a browser)? | **No** | No WebView, WebEngine or in-app browser. Links open in the system browser, which IARC does not count. |
| Is user-generated content moderated? | **Yes** | Per-server administrators, plus in-app reporting and blocking. Note in free text that moderation is by each server's operator, since the app is self-hosted. |
| Is the app directed at children? | **No** | Minimum age 13 (16 in UK/EEA) in the terms. |
| Does the app share data with third parties? | **No** | |
| Does the app collect precise location? | **No** | |

**Expected outcomes:** IARC typically returns **Teen** (ESRB), **PEGI 12**,
**USK 12**, **ClassInd 12** for an unmoderated-UGC communication app. That is
normal and is the same band every chat app lands in. Do not try to argue it
down: understating UGC is a policy violation, and Teen costs you nothing.

**Apple's equivalent** (App Store Connect → Age Rating) asks a shorter set. The
answers that matter:

| Apple question | Answer |
| --- | --- |
| Unrestricted Web Access | **No** |
| **App allows users to communicate / user-generated content** | **Yes**, and select **frequent/intense** for "Mature/Suggestive Themes" only if you believe it; for a general chat client, "None" on the content categories plus the UGC flag is the honest combination |
| Gambling / Contests | No |
| Medical/Treatment Info, Alcohol Tobacco Drugs, Horror, Violence, Sexual Content, Profanity | **None** |
| Age Rating outcome | expect **12+** |

Apple will also ask, in the same flow, whether the app has **age verification**
— it does not, and that is fine at 12+.

---

## 7. Checklist for tomorrow

- [ ] Create `support@bsfchat.com` and `security@bsfchat.com`
- [ ] Fill the legal entity into `/terms` §1 and §10, and delete the three
      review banners from `/privacy`, `/terms` and `/support`
- [ ] Decide the two retention numbers in `/privacy` §9 (logs, backups) and make
      them true on the host, or change them
- [ ] Merge `feat/ugc-safety` (server) and the mobile/UGC client branches; do
      not build from `main`
- [ ] Create the demo account + a second account + seeded conversation on
      `chat.bsfchat.com`, and paste the credentials into §5.1 and §5.2
- [ ] Confirm which Play closed-testing rule your account falls under (§0.4)
- [ ] Read the real minimum iOS version off the CI log (§0.6)
- [ ] Record three short foreground-service demo videos for Play
- [ ] Screenshots — and no voice channel in the iPhone set
- [ ] Publish the `web` branch (it is deliberately unmerged right now)
