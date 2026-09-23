# BSFChat — store submission pack

**Not published.** This file lives in the `web` repo for convenience, but it is
not part of the site and is not linked from anywhere. It is working material for
filling in App Store Connect and the Google Play Console.

**Drafted by an AI assistant on 2026-09-23, then reviewed and substantially
corrected by a second one the same day.** Both read `client`, `server`,
`identity` and `deploy`; every factual claim is cited so you can re-run the
check. Everything that is a *decision* — pricing, entity name, licence, age
rating appeal — is yours and is marked **[owner: decide]**.

**What the second pass changed, in case you read the first one:** the App Store
copy was written for a text-only client, because iOS voice was off in the tree
it was read from. Voice is now on by default and has run on a real iPhone, so
§0.1, §0.2, §1.3, §1.4, §4.1, §4.2, §5.1 and §6 have all been rewritten. The
blocker list changed too: the *client* side of block/report/delete is merged,
but the *server* side of reporting is not, and that is now the only hard gate
(§0.3). The licence question is written up as a decision with a recommendation
(§0.6) rather than settled unilaterally. Play needs four foreground-service
declarations, not three (§0.7).

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

### 0.1 iOS voice: this file used to say there was none. There is. (read this first)

**Superseded on 2026-09-23.** An earlier draft of this document opened with
"iOS has no voice. None." and wrote the entire App Store listing around a
text-only client. That was true of the tree it was read from. It is not true of
the tree you are shipping, and every piece of iOS copy that follows has been
rewritten accordingly.

What the integration branch actually does (`wt/integrate`, HEAD `1121acf`):

| Was claimed | Actually | Evidence |
| --- | --- | --- |
| `BSFCHAT_ENABLE_VOICE` is `OFF` for iOS | **`ON`**, by default | `CMakeLists.txt:57-59`, rationale at `:19-30` ("built, installed and USED on a real iPhone 16 Pro Max") |
| voice wiring is gated on `!defined(Q_OS_IOS)` | that guard covers only the **media-state announcement** block, which needs the `screenShare` object iOS does not have | `src/main.cpp:284`; the voice and camera wiring at `:271-283` has no iOS exclusion |
| nothing configures `AVAudioSession` | `playAndRecord` / `voiceChat` / `defaultToSpeaker`, plus interruption and route-change recovery | `src/voice/IosAudioSession.mm:128-170`, `:65-112` |
| `UIBackgroundModes` is deliberately absent | emitted as `audio` whenever voice is on, which it now is | `CMakeLists.txt:676-685` → `ios/Info.plist.in:207` |
| the iOS CI job does not pass `-DBSFCHAT_ENABLE_VOICE=ON` | true, and deliberate — it passes **no** flag so CI builds the same default configuration a release does | `.github/workflows/ci.yml:1419-1425` |

**So the first TestFlight build is a voice-capable client.** Consequences, and
they cut the opposite way from the old advice:

- The description, screenshots and review notes **should** mention voice and
  camera video. Saying "no voice on iPhone" while the binary contains a
  microphone prompt, an active `AVAudioSession` and `UIBackgroundModes: audio`
  is the same 2.3.1 accurate-metadata problem in reverse — and the background
  mode is exactly the thing a reviewer opens the Info.plist to check.
- **Screen sharing still does not exist on iOS.** `src/main.cpp:259-270` builds
  `AndroidScreenShareController` on Android only; there is no iOS equivalent.
  Do not put screen sharing in the Apple copy, do not screenshot it, and do not
  let the Play copy and the Apple copy be unified.
- The App Store screenshot set **should** now include a voice channel. See §8.

### 0.2 Echo cancellation — implemented on Apple, never heard on a device

The earlier draft said there was no software AEC anywhere. There is now, on
Apple platforms, and it arrived the same day as iOS voice.

`src/voice/DarwinVpioBackend.mm` is a full `kAudioUnitSubType_VoiceProcessingIO`
capture-and-render backend — Apple's own echo cancellation, noise suppression
and automatic gain control, in one audio unit, shared by macOS and iOS. Not
WebRTC's APM (explicitly rejected, `src/voice/VoiceGain.h:25-50`), not
Opus-side. Selection logic at `src/voice/AudioBackendSelect.h:59-78`; build flag
`BSFCHAT_DARWIN_VPIO` defaults **ON** for Apple and is forced off elsewhere
(`CMakeLists.txt:71-82`); runtime setting `audio/voiceProcessing` defaults true
(`src/core/Settings.cpp:597-608`) and is surfaced as "Echo cancellation" in
`qml/components/ClientSettings.qml:522-535`. A VPIO failure falls back to the Qt
backend for the rest of the run (`src/voice/AudioWorker.cpp:899-920`).

| Platform | Voice | Camera video | Screen share | Echo cancellation |
| --- | --- | --- | --- | --- |
| macOS | yes | yes | yes | **VPIO — implemented, not yet validated on device** |
| iOS | **yes** | **yes** | **no** | **VPIO — implemented, not yet validated on device** |
| Android | yes | yes | yes | platform/hardware, via `MODE_IN_COMMUNICATION` (`src/voice/AndroidAudioRouting.h:4-20`) |
| Windows / Linux | yes | yes | yes | **none** (`src/voice/VoiceGain.h:66-70`) |

`CMakeLists.txt:37-44` says it in the tree: **"IMPLEMENTED 2026-09-23, NOT YET
HEARD"**. That is the whole difference between what you may write and what you
may not:

- **Safe:** "Echo cancellation and noise suppression on Mac and iPhone, using
  the system's own voice processing."
- **Not safe:** "crystal-clear", "echo-free", "studio quality", or any
  superlative. One tester in a live room with speakers on decides whether that
  sentence was true, and a store listing is a bad place to find out.
- The support page already carries the per-platform version of this. Keep the
  two in step.

**[owner: decide]** Whether to mention AEC in the listings at all before it has
been heard on a device. The recommendation is yes with the cautious wording
above — it is a real feature and reviewers do not test audio quality — but it is
your claim to make.

### 0.3 The server-side report endpoint is still unmerged. This is the one hard blocker. (highest risk)

The client half of blocking, reporting and account deletion **is** merged into
the integration branch — `qml/components/BlockedUsersDialog.qml`,
`ReportDialog.qml`, `DeleteAccountDialog.qml`, `src/net/IgnoredUsers.h`,
`src/net/ReportRequest.h`, `src/net/DeactivateFlow.h`, all present at
`wt/integrate`. That part of the earlier draft is now out of date.

**The server half is not.** `server` main (`295156f`) has no
`src/api/ReportHandler.cpp`; it exists only on `feat/ugc-safety` (`d5d0fe6`,
worktree `wt/server-ugc-safety`). Against the deployed `chat.bsfchat.com`,
`POST /_matrix/client/v3/rooms/{roomId}/report/{eventId}` **404s**.

So today:

| Mechanism | Client | Server | Works against production? |
| --- | --- | --- | --- |
| Block | merged | `m.ignored_user_list` is standard account data, already served | **yes** |
| Account deletion | merged | deactivate already served | **yes** |
| **Report** | merged | **unmerged, undeployed** | **no — the button fails** |

An App Review tester following your own review notes will tap Report and watch
it fail. That is a **guideline 1.2** rejection, and Google's UGC policy asks the
same question. It is also the single item on this whole list that neither
paperwork nor credentials can fix.

**[owner: decide]** Merge and deploy `feat/ugc-safety` to `chat.bsfchat.com`
before submitting, or do not submit. There is no third option that survives
review. (The merge and the production deploy are both out of scope for the
assistant that wrote this file.)

### 0.4 Google Play closed-testing rule

A Google Play developer account registered as an **individual after 13 November
2023** must run a closed test with **at least 12 testers opted in for 14
continuous days** before it can apply for production access. An organisation
account, or an individual account older than that, is exempt.

**You must check which one yours is**, in Play Console → Setup → Advanced
settings, because it decides whether "closed testing today" means "production in
two weeks" or "production whenever you like". Nothing in the code tells me this
and I have not looked at your account.

### 0.5 Things you must create before you can submit

| # | Thing | Why |
| --- | --- | --- |
| 1 | `support@bsfchat.com` mailbox or alias | Both stores email it. Cited on every page. A bounce is a rejection. |
| 2 | `security@bsfchat.com` mailbox or alias | Cited in the policy and terms. |
| 3 | **Demo account** on `chat.bsfchat.com` with a **password** (not OIDC) | Review notes hand it to the reviewer. See §5. |
| 4 | **A second account**, and some seeded conversation | So the reviewer has someone to block and something to report. Apple reviewers routinely fail 1.2 because there is nothing to demonstrate against. |
| 5 | Screenshots — iPhone 6.9"; Play phone set | See §8 for the shot lists. The iPhone set must come off a real device; the Play set can come off an emulator. |
| 6 | 1024×1024 App Store icon, 512×512 Play icon, 1024×500 Play feature graphic | No alpha and no transparency on the Apple icon. |
| 7 | Legal entity name + jurisdiction | For `/terms`, `/privacy` §1, and the Play "Developer name". |
| 8 | **Four** Play foreground-service demo videos | See §0.7. |
| 9 | Merge + deploy the server side of §0.3 | The one blocker that is not paperwork. |

### 0.6 The licence question — a decision, not a task

**There is still no LICENSE file in any of the six repos, and there never has
been in any commit.** Neither listing may say "open source" until there is one.

The evidence is genuinely mixed, which is why this is flagged rather than fixed:

- **For open source:** the only licence declaration the owner has ever written
  is `LABEL org.opencontainers.image.licenses=MIT` in `server/Dockerfile:59` and
  `identity/Dockerfile:51` (both committed by Josh Griffith, 2026-04-15) — and
  those labels are already published to the world in the GHCR images. The org is
  public, `protocol` is cloned anonymously inside the server's build container,
  the site links to source in four places and the footer says "BSFChat
  **contributors**".
- **Against:** 0 of 347 first-party source files carry a copyright or SPDX
  header, no commit message in any repo has ever mentioned licensing, and
  `bsfchat.com/terms` currently tells the public the opposite — that the source
  is **source-available, not open source**, with no rights granted.

Adding a licence is an irrevocable grant of rights to everyone, forever, over
your own property, and it would contradict text already published on your own
site. So it has not been added.

**[owner: decide] — the specific questions, in the order they unblock things:**

1. **Was `licenses=MIT` deliberate, or boilerplate copied into an OCI label
   block?** Everything else follows from this one answer.
2. **Whose name goes on it?** Your legal name, or a company. `/terms` §1 and
   `/privacy` §1 need the same answer, and `installer.nsi:35` currently says
   `(c) BSFChat`, which is not a legal entity.
3. **Has anyone else committed non-trivial code?** If so they hold copyright and
   must agree before anything is relicensed.
4. **One licence for all six repos, or a split?** (An AGPL server with an MIT
   client is defensible, but `protocol` links into both, which complicates it.)
5. **Qt on iOS — this one is independent of your choice and needs a lawyer, not
   a LICENSE file.** Qt for iOS is static-only and you are on the open-source
   (LGPL-3.0) build; the CI pin comment at `.github/workflows/ci.yml:74-76` says
   so explicitly. LGPLv3 §4(d)(0) expects a user to be able to relink your app
   against a modified Qt, and an App Store binary cannot be relinked. The
   realistic options are a commercial Qt licence for the iOS target, or
   publishing relinkable object code and accepting the residual Apple-EULA risk.
   **Choosing MIT does not solve this.**

**Recommendation if you want one: MIT, across all six repos, plus a separate
`TRADEMARK.md` for the name and logo** (MIT grants no trademark rights and
people will assume it does). It ratifies the declaration you already made rather
than inventing a new one, it retro-validates the MIT-labelled images already on
GHCR, and nothing in the dependency set constrains it — everything is
MIT/BSD/Apache-2.0/public-domain except MPL-2.0 (libdatachannel, file-level
copyleft, compatible) and LGPL-3.0 Qt (which permits permissive application
licensing). **Apache-2.0 is the defensible alternative** for its explicit
patent grant, which is worth weighing given H.264/AV1/Opus and the fact that
openh264 is built from Cisco *source* rather than shipped as Cisco's
patent-covered binary. **AGPL/GPL would be actively self-defeating here** — that
is the VLC-on-the-App-Store conflict, and you are mid-submission.

Until that decision lands: the copy below says **"the source is public"**, which
is true today. The word `foss` has been removed from the Apple keyword list,
where it directly contradicted `/terms`.

### 0.7 Four Play foreground-service declarations, not three

Each foreground-service type needs a Play Console declaration **and a short
screen recording** showing the feature in use. The earlier draft counted three.
There are four, because the voice service declares camera as well:

| Service | `foregroundServiceType` | Manifest | What the video must show |
| --- | --- | --- | --- |
| `VoiceService` | `microphone` | `android/AndroidManifest.xml:256-259` | Joining a voice channel, talking, then backgrounding the app while the call continues |
| `VoiceService` | `camera` | same line — the type is `microphone\|camera` | Turning the camera on in a voice channel |
| `SyncService` | `dataSync` | `:265-268` | A message arriving as a notification with the app backgrounded |
| `MediaProjectionService` | `mediaProjection` | `:276-279` | The system screen-capture consent dialog, then a shared screen |

`dataSync` is the one Google pushes back on hardest; the declaration text should
say plainly that it keeps the Matrix sync connection open to deliver messages
**without Firebase**, and that no third party is involved
(`src/core/AndroidNotifier.cpp:309`).

### 0.8 Smaller things that will bite

- **Do not name Discord, Slack, TeamSpeak or Matrix-the-brand in Apple
  keywords.** Competitor trademarks in keywords are a 2.3.7 rejection. The
  keyword list below is clean; if you add to it, keep it clean. The *description*
  may describe the app as "Discord-style" as plain comparative language, but the
  safest copy avoids it, and the copy below does.
- **`ITSAppUsesNonExemptEncryption` is `true`** in `ios/Info.plist.in:163-164`,
  and that is the honest answer now more than ever: with voice on, the iOS
  binary genuinely links DTLS-SRTP from libdatachannel against an OpenSSL we
  cross-compile ourselves (`scripts/build-openssl-ios.sh`), which is not Apple's
  OS crypto. **But the comment above that key is wrong about what `true` does.**
  It claims the declaration stops App Store Connect asking. It does the
  opposite: `false` is what skips the questionnaire; `true` means App Store
  Connect *will* ask for export-compliance information until you paste back an
  `ITSEncryptionExportComplianceCode`. Budget for answering it once per version,
  for the French declaration, and for the annual self-classification report that
  goes with shipping standard crypto. See `client/docs/ios-release.md` §3.
- **iOS minimum version is unpinned.** No `IPHONEOS_DEPLOYMENT_TARGET` is set
  anywhere; CI deliberately lets Qt 6.10.3 choose and prints it
  (`.github/workflows/ci.yml:1462-1469`). Read the real number off the CI log
  line "Minimum iOS version (chosen by the Qt toolchain)" before you type it
  into the listing. Do not guess.
- **iPhone only.** `XCODE_ATTRIBUTE_TARGETED_DEVICE_FAMILY "1"`
  (`CMakeLists.txt:701-702`). There is no iPad layout, so set the listing to
  iPhone only rather than letting it ship an upscaled iPad build.
- **No push notifications on iOS — and in fact no OS-level notifications at
  all.** There is no APNs, no PushKit, and `NotificationManager` only has a
  `QSystemTrayIcon` path (desktop) and an `AndroidNotifier` path
  (`src/core/NotificationManager.cpp:18`, `:38`). On iPhone, new messages appear
  in the UI while the app is open and nowhere else. Worth a line in the
  description so it does not read as a bug; the copy below has one.
- **Link previews fetch third-party pages from the reader's device, with no
  setting to turn them off.** `qml/components/LinkPreview.qml:337-377` issues a
  raw `XMLHttpRequest` GET against any URL anyone posts, following up to three
  redirects, capped at two per message (`qml/components/MessageBubble.qml:1381-1387`);
  YouTube links additionally hit `youtube.com/oembed` and `img.youtube.com`
  (`:386-390`, `:46`, `:158`). This discloses the **reader's** IP to any site a
  **sender** links, which sits awkwardly beside the "Hide my IP address" voice
  setting. The privacy policy already discloses it honestly (§8, "Link
  previews") and the Data Safety free text below mentions it.
  **[owner: decide]** whether to add a setting to disable link previews before
  submitting. Not a blocker; reviewers will not find it. It is the kind of thing
  someone writes a blog post about later.

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
| **Version** | the tag you cut, with the suffix stripped — `v0.0.48-rc.2` → `0.0.48` (`cmake/Version.cmake:59`). **Read it off the build, do not copy this number.** | |
| **Subtitle note** | voice is now in the iOS build; see §0.1 before reusing any older copy | |

### 1.2 Promotional text (170 max — editable without review) · **161 chars**

```
Chat that does chat. Run the server yourself, sign in once, talk to your
people. No analytics, no crash reporting, no ads, no third-party SDKs. Check
the source.
```

### 1.3 Description (4000 max) · **3656 chars**

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
• Voice channels, with camera video when you want it
• Private channels with real role-based permissions
• Direct messages
• Threads, replies, edits, reactions, pinned messages and mentions
• File attachments and inline images
• Full-text search across your history
• Multiple servers in one app, each with its own account
• Themes, accent colours and density that stay out of your way

VOICE THAT KEEPS UP

Join a voice channel and talk. Calls keep running when you switch apps or lock
your phone, so walking to the next room does not end the conversation. Echo
cancellation and noise suppression come from the system's own voice processing.
Turn your camera on if you want to be seen, or leave it off.

Calls connect directly between participants where your network allows it, which
means the people in the call can see your IP address — the same as every
peer-to-peer voice app. If you would rather they did not, turn on "Hide my IP
address" and every call is routed through a relay instead. If that setting is on
and no relay is available, the call is refused rather than quietly connected
directly.

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
• Screen sharing is not available on iPhone. It works on Mac, Windows, Linux
  and Android.
• There are no push notifications. BSFChat shows you new messages while it is
  open, and nothing at all while it is closed. Nothing is handed to Apple's
  notification servers because there is nothing to hand.
• Echo cancellation is new. It uses Apple's own voice processing and it is on by
  default, but if a room gives you trouble, headphones still win.
• Messages are not end-to-end encrypted. Traffic is protected by TLS and your
  server holds your history. We will not claim otherwise.

bsfchat.com/privacy · bsfchat.com/support · github.com/BSFChat
```

### 1.4 Keywords (100 max, comma-separated, no spaces) · **96 chars**

```
selfhosted,selfhost,privacy,chat,messaging,voice,community,team,server,nologging,noads,guild,irc
```

Rules applied: no competitor trademarks (2.3.7), no repetition of words already
in the name or subtitle (Apple indexes those separately, so repeating wastes
characters), singular forms only (Apple stems automatically).

Two changes from the earlier draft:

- **`voice` added.** It is now a real feature of the iOS build and it is the
  word people search for.
- **`foss` removed.** There is no LICENSE file in any repo, and
  `bsfchat.com/terms` currently tells the public the software is
  source-available rather than open source. Claiming `foss` in the keyword field
  contradicts your own published terms. Put it back the day §0.6 is settled.

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

Android has **screen sharing** and iOS does not, so this copy still differs from
Apple's on purpose. Do not unify them. Voice and camera video are no longer a
difference between the two — both have them (see §0.1) — so the older
instruction to strip all voice language from the Apple copy no longer applies.

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
| **versionCode / versionName** | derived, not typed. `cmake/Version.cmake:70-135`: `MAJOR*1000000 + MINOR*10000 + PATCH*100`, minus 100 plus N for an `-rc.N`. So `0.0.48-rc.2` → **4702**, and the eventual `0.0.48` → **4800**. **Read the real pair off the CI log ("BSFChat version:" line) or `aapt2 dump badging` on the artefact — a versionCode is burned permanently by any upload to any track.** | |
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
| **Web browsing history** | **No** | No | — | — | No WebView anywhere (`qml/mobile/MobileMain.qml:776`). |
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

When a message contains a link, the app fetches that page from the user's own
device to show a preview, so the linked site receives the reader's IP address in
the same way it would if the reader opened the link in a browser. No preview
data is sent to or stored by the developer.
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

### 4.1 Answers — iOS build

| Apple data type | Collected | Linked to user | Used to track | Purpose | Reasoning |
| --- | --- | --- | --- | --- | --- |
| **Contact Info › Name** | **Yes** | Yes | No | App Functionality | Display name and nickname, stored on the server, shown to others. |
| **Contact Info › Email Address** | **Yes** | Yes | No | App Functionality | **Optional and unverified**, only via BSFChat ID. Apple has no "optional" flag — declare it; the policy explains. |
| **Contact Info › Phone, Physical Address, Other** | No | — | — | — | Never requested; no column exists. |
| **User Content › Photos or Videos** | **Yes** | Yes | No | App Functionality | Attachments and avatars — **and, since iOS voice landed, live camera video in a voice channel**, which is transmitted and never recorded. |
| **User Content › Audio Data** | **Yes** | Yes | No | App Functionality | **Changed — iOS voice now ships** (`CMakeLists.txt:57-59`). Microphone audio is transmitted live to the other participants in a voice channel. Apple's "collect" means retained beyond servicing the request, and nothing records it (`src/voice/` writes no audio anywhere) — but it leaves the device, so declare it, and say "not retained" in the purpose notes rather than answering No. |
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

### 4.2 Microphone and camera: the strings, the prompt, and the background mode

`ios/Info.plist.in:85-88` declares `NSMicrophoneUsageDescription` and
`NSCameraUsageDescription`, and — now that voice is on — the app **does** prompt.
`CMakeLists.txt:21-24` records that the microphone prompt appeared at first join
on a real iPhone, which is exactly the behaviour Apple wants. Denial is handled
with iOS-specific copy pointing at Settings → Privacy & Security → Microphone
(`src/voice/VoiceStartPolicy.h:66-69`).

Both strings are also load-bearing for the build, not just for the user: the
file's own header explains that Qt decides at *configure* time whether to link
the `QDarwinMicrophonePermission` / `QDarwinCameraPermission` backends by running
PlistBuddy over the named plist. Removing a key silently breaks the prompt.
Leave them alone.

**The microphone string already describes background use** — "including while
BSFChat is in the background, so the call continues when you switch apps". That
is deliberate and it must stay, because the build ships
`UIBackgroundModes: audio` (`CMakeLists.txt:676-685` → `ios/Info.plist.in:207`).
A background mode whose usage string implies foreground-only capture is the
precise inconsistency App Review looks for. The two ship together or not at all:
`src/voice/IosAudioSession.mm:128-170` sets the `AVAudioSession` category to
`playAndRecord` with mode `voiceChat`, which is what makes the declaration true
rather than merely present.

**The Apple and Play answers now match** on audio and video. The earlier draft
told you to expect an asymmetry — Play saying audio is collected and Apple
saying it is not — and to revisit the Apple label "the day iOS voice ships".
That day was 2026-09-23. It has been revisited; §4.1 is the result.

---

## 5. App review notes

### 5.1 Apple — App Review Information → Notes (4000 max) · **3979 chars**

```
Thank you for reviewing BSFChat.

WHAT THIS APP IS

BSFChat is a client for self-hosted chat servers, as an email client is a
client for mail servers. Messages live on a server run by the user or their
community. We operate one public server; the demo account is on it.

There is no in-app registration on purpose: an account belongs to a server and
is made by that server's operator. Please use the account below.

DEMO ACCOUNT

  Server:   https://chat.bsfchat.com
  Username: <owner: fill in>
  Password: <owner: fill in>

HOW TO SIGN IN

1. Launch. The "Add a server" screen appears.
2. Tap "Show manual server entry", below the OR divider.
3. Enter https://chat.bsfchat.com and tap "Check".
4. Tap "Use password instead"; enter the username and password above. You
   land in a server with seeded channels and conversation.

("Sign in with BSFChat ID" is our own OpenID Connect provider at
id.bsfchat.com: first-party, not a social login, so 4.8 does not apply. It uses
ASWebAuthenticationSession, not a web view.)

GUIDELINE 1.2 - UGC

* Block: tap any name or avatar to open the profile card, then the lock
  button. Immediate and server-side; their messages and invitations stop
  reaching the blocker on every device, and they are not told. Manage at
  "..." > Your profile > Blocked accounts.
* Report: long-press any message you did not send > "Report message...", or
  "Report user...". Free-text reason plus an "also block them" checkbox, ticked
  by default. Reports reach that server's administrators with a retained copy
  of the message.
* Moderation: each server's admins remove content and ban accounts. On the
  server we operate we act on reports within 24 hours.
* Published contact: https://bsfchat.com/support. Our terms at
  https://bsfchat.com/terms state zero tolerance for abusive content.

GUIDELINE 5.1.1(v) - ACCOUNT DELETION

"..." > Your profile > bottom of the list > "Delete account". It lists what is
deleted; to confirm you type your username and enter your password. (Accounts
created through our sign-in provider have no chat-server password; the typed
username alone confirms those.)

We disclose rather than hide: deletion destroys the credentials, erases the
profile and removes the person from every channel, but messages they sent
remain, because a group conversation belongs to everyone in it. The dialog says
so before confirming; https://bsfchat.com/privacy#deletion explains it.

VOICE AND BACKGROUND AUDIO

This build has voice channels and camera video: tap a voice channel, then
Join. iOS asks for the microphone the first time, and for the camera when you
first use it.

UIBackgroundModes is "audio" and it is used: a call continues while the app is
backgrounded or the screen is locked. To see it, join the channel and swipe
home - audio keeps flowing. Without the mode iOS silences the session and every
call dies on backgrounding. AVAudioSession is playAndRecord / voiceChat, and
the microphone usage string describes the background case explicitly. Nothing
is recorded: media is transmitted live and written nowhere.

WHAT THIS BUILD DOES NOT DO

* No screen sharing on iOS. It exists on the other platforms; the control is
  not shown here and the description says so.
* No push notifications: no APNs, no PushKit, no notification framework at all.
  New messages appear while the app is open and nowhere else. The background
  audio mode is for calls only.
* No in-app purchases, advertising, analytics, crash reporting or third-party
  SDKs.

OTHER

* https://bsfchat.com/privacy is explicit that messages are not E2E encrypted.
* ITSAppUsesNonExemptEncryption is true: call media is DTLS-SRTP built against
  an OpenSSL we cross-compile ourselves, not only Apple's cryptography.
* NSLocalNetworkUsageDescription is declared because self-hosted servers often
  live on a LAN and because calls gather host ICE candidates. Connecting to
  chat.bsfchat.com does not trigger the prompt.

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

  In-app at ... > Your profile > Delete account. Confirmation is typing your
  username, plus your password if the account has one (accounts created through
  our sign-in provider have no chat-server password). Also available by email to
  support@bsfchat.com. Web route for the Play account-deletion declaration:
  https://bsfchat.com/support

FOREGROUND SERVICES — why each one exists

  • microphone (VoiceService): keeps the process alive during a voice call so
    Android does not kill it when the user switches apps mid-conversation.
  • camera (VoiceService, same service): the user can turn their camera on
    inside a voice channel, and the call — video included — must survive
    backgrounding for the same reason.
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
| In-app path | `... menu → Your profile → Delete account` (type the username; password too, for password accounts) |
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
| **Does the app allow users to interact or exchange content with other users?** | **Yes** | The entire app. Text, files, images, live voice and camera video on every platform, and screen share everywhere except iOS. This is the single answer that drives the rating. |
| **Can users share their current location with others?** | **No** | No location feature. Be ready to defend this: peer-to-peer calls expose an IP address to other participants, and an IP is coarsely geolocatable. IARC's question is about a *location-sharing feature*, and there is none — the user cannot send their location and the app never reads it. Disclosed in the privacy policy under Voice and video, with a "Hide my IP address" setting as the mitigation. Do not tick this box. |
| **Can users share personal information with other users?** | **Yes** | Free-text messages and arbitrary file uploads. A user can type anything. |
| Does the app let users purchase digital goods? | **No** | No IAP, no billing library, no payment code. |
| Does the app contain ads? | **No** | |
| Does the app provide unrestricted access to the internet (a browser)? | **No** | No WebView, WebEngine or in-app browser. Links open in the system browser, which IARC does not count. |
| Is user-generated content moderated? | **Yes** | Per-server administrators, plus in-app reporting and blocking. Note in free text that moderation is by each server's operator, since the app is self-hosted. **Only answer Yes once the server-side report endpoint of §0.3 is deployed** — until then the Report button 404s and the answer is not true. |
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

Ordered so that nothing waits on something further down. The full end-to-end
runbook, including every CI step and both upload paths, is
`client/docs/store-submission-runbook.md` — this list is the paperwork subset.

**Blockers (nothing ships until these are true)**

- [ ] Merge `feat/ugc-safety` into `server` main and deploy it to
      `chat.bsfchat.com`, then tap Report on a real build and watch it succeed
      (§0.3). This is the only item that is not paperwork.
- [ ] Create `support@bsfchat.com` and `security@bsfchat.com` and send a test
      message to each. Both stores email them; a bounce is a rejection.
- [ ] Create the demo account, a second account, and a seeded conversation on
      `chat.bsfchat.com`. Paste the credentials into §5.1 and §5.2.

**Decisions only you can make**

- [ ] The licence (§0.6) — at minimum, answer question 1: was
      `org.opencontainers.image.licenses=MIT` deliberate?
- [ ] Legal entity name and jurisdiction → `/terms` §1 and §10, `/privacy` §1,
      Play "Developer name", App Store "Copyright".
- [ ] The two retention numbers in `/privacy` §9 (nginx logs, backups) — make
      them true on the host, or change the numbers.
- [ ] Whether to claim echo cancellation in the copy before it has been heard on
      a device (§0.2).
- [ ] Whether to ship a setting to disable link previews first (§0.8). Not a
      blocker.

**Paperwork**

- [ ] Delete the owner-review banners from `/privacy`, `/terms` and `/support`
      once you have read them.
- [ ] Confirm which Play closed-testing rule your account falls under (§0.4).
- [ ] Read the real minimum iOS version off the CI log line "Minimum iOS version
      (chosen by the Qt toolchain)" and type that into the listing (§0.8).
- [ ] Record **four** foreground-service demo videos for Play — microphone,
      camera, dataSync, mediaProjection (§0.7).
- [ ] Answer the App Store Connect export-compliance questions; expect them,
      because `ITSAppUsesNonExemptEncryption` is `true` (§0.8).

**Screenshots** (shot lists: `client/docs/store-submission-runbook.md` §8)

- [ ] iPhone 6.9" set — **must come off a real device**, and the set now
      includes a voice channel, which the earlier draft told you to avoid.
- [ ] Play phone set — an emulator is acceptable; the assistant can capture
      these.
