# BSFChat — first Google Play upload

**Working document. Not published, not linked from the site.** Companion to
`STORE-LISTING.md`, which covers both stores; this one covers *only* the first
Play upload and is written to be worked through top to bottom in one sitting.

Prepared 2026-09-29 against:

| Thing | Value | How it was checked |
| --- | --- | --- |
| Bundle | GitHub Actions artifact `BSFChat-Android-AAB`, run **36489227004** | `gh api` — artifact id `11001387550`, 48,187,120 bytes, **expires 2026-12-27 21:55 UTC** |
| Tag / commit | `v0.0.48-rc.8` / `5c643aa` | run metadata; CI conclusion **success** |
| applicationId | `com.bsfchat.app` | `aapt2 dump badging` line in the run's `android (arm64-v8a)` log |
| versionCode | **4708** | same log line — *burned permanently by the first upload* |
| versionName | `0.0.48-rc.8` | same |
| minSdk / targetSdk / compileSdk | 28 / **36** / 36 | same |
| Signing | signed by `CN=BSFChat, OU=BSFChat, O=BSFChat, L="Cambridge, UK", ST=Cambridgeshire, C=GB`, SHA-256 `d6cfb799…ac90`, self-signed | `sign-android.sh` output in the log |
| ABIs | **arm64-v8a only** | `QT_ANDROID_ABIS=arm64-v8a` in CI |
| Privacy policy | <https://bsfchat.com/privacy/> — fetched and read in full, 2026-09-29 | source of truth for §3 below |
| Terms | <https://bsfchat.com/terms/> — fetched and read in full | |

> **Read §7 first if you read nothing else.** It lists the four places where the
> answers that were already drafted contradict the live policy or the code, and
> one place where the *brief* for this document was wrong about the policy.

---

## 0. Status in one screen

**Cleared since `STORE-LISTING.md` was written:** the server-side report
endpoint — the file's "one hard blocker" (§0.3) — is merged and live. See §7.1.

**Ready to paste:** every Data Safety answer (§3), every content-rating answer
(§4), every other App-content declaration (§5), the short and full description
(§8.1, §8.2), the reviewer instructions (§8.5), the four foreground-service
justifications (§5.6).

**Blocks the upload itself:** nothing, once the app exists in the Console and
Play App Signing is accepted. The AAB is signed, targets 36, and carries an
unused versionCode.

**Blocks the rollout (even to internal testing):** feature graphic, phone
screenshots, and the App-content declarations in §5. Plus the four
foreground-service demo videos — those are the longest-lead item in the whole
list, because they have to be recorded on a phone.

**Cannot be verified from here:** whether your Play developer account is subject
to the 12-testers-for-14-days closed-testing rule (§6.2), and whether a report
actually files a row on production end-to-end (§7.1).

---

## 1. Before you open the Console

Fifteen minutes of prerequisites. Do these first or you will stop halfway.

- [ ] **`support@bsfchat.com` and `security@bsfchat.com` must receive mail.**
      Both are cited on `/privacy`, `/terms` and `/support`, and Play emails the
      support address. Send yourself a test to each.
- [ ] **Demo account on `chat.bsfchat.com` with a password** (not OIDC — a
      reviewer cannot complete a browser OIDC flow reliably). Plus a **second
      account** and a short seeded conversation, so there is something to block
      and something to report.
- [ ] **Upload keystore backed up** somewhere offline. CI already holds it as
      `ANDROID_KEYSTORE_BASE64`; if that is your only copy, export it now.
      (`client/docs/android-release.md` §3.)
- [ ] **Decide the developer name** that appears publicly on the listing. The
      policy and terms both name *Joshua Griffith, trading as BSFChat*, and say
      plainly that BSFChat is not a registered company. Whatever you type in the
      Console must not contradict that.
- [ ] **Decide country availability.** `/privacy` §11 and `/terms` §3 both say
      *"We have tried to keep the app out of the regions where it would be
      unlawful for the people there to use it."* That sentence is a claim about
      an action you take in this Console, on the Countries/regions page. If you
      publish to all countries it stops being true.

---

## 2. Getting the AAB onto your Mac

The Play Developer Publishing API **cannot publish an app's first bundle** —
Google requires the first artefact of a new application to be uploaded through
the Console by hand. So this is a download, then a drag.

### 2.1 Download

Easiest, from anywhere in the `client` checkout:

```sh
cd ~/dev/gamechat/client
gh run download 36489227004 --repo BSFChat/client --name BSFChat-Android-AAB --dir ~/Downloads/bsfchat-aab
```

That writes **`~/Downloads/bsfchat-aab/BSFChat-android.aab`** (the artifact zip
contains exactly one file; `gh` unpacks it for you).

Browser alternative — <https://github.com/BSFChat/client/actions/runs/36489227004>
→ *Artifacts* → **BSFChat-Android-AAB** → downloads `BSFChat-Android-AAB.zip`
(48.2 MB); unzip it and you get the same `BSFChat-android.aab`.

### 2.2 Check it before you upload

A versionCode is burned permanently the moment *any* track sees it, so confirm
you have the right file. Thirty seconds:

```sh
AAPT=$(ls -d ~/Library/Android/sdk/build-tools/*/ | sort -V | tail -1)aapt2
"$AAPT" dump badging ~/Downloads/bsfchat-aab/BSFChat-android.aab 2>/dev/null | head -3
```

Expect:

```
package: name='com.bsfchat.app' versionCode='4708' versionName='0.0.48-rc.8' ...
```

If `aapt2` is not installed, skip it — CI already asserted exactly this
(`android (arm64-v8a)` job, and the job fails if the id is not
`com.bsfchat.app` or targetSdk is below 36). The check is belt-and-braces.

> **The artifact expires 2026-12-27.** After that the bundle has to be rebuilt
> from the tag, which produces the same versionCode 4708 — fine if nothing has
> been uploaded, a hard collision if something has.

### 2.3 Create the app

Play Console → **All apps** → **Create app**.

| Field | Value |
| --- | --- |
| App name | `BSFChat` |
| Default language | English (United Kingdom) — matches the copy's spelling |
| App or game | **App** |
| Free or paid | **Free** (irreversible for an app that has ever been free-to-paid; free is correct — there is no IAP and no paid tier) |
| Declarations | Tick both (Developer Programme Policies; US export laws) |

Then **Test and release → Setup → App signing**: accept **Play App Signing**
with *Let Google create and manage my app signing key* (the default). Your CI
key becomes the **upload** key, which means losing it is recoverable rather than
terminal.

### 2.4 Upload

**Test and release → Testing → Internal testing → Create new release.**

Internal testing, not closed and not production, for three reasons: it has no
tester-count minimum, it does not start any 14-day clock, and it is the only
track where a bad first upload costs you nothing but a versionCode.

1. **App bundles** → drag `BSFChat-android.aab` in.
2. Release name: Play will propose `4708 (0.0.48-rc.8)`. Leave it.
3. Release notes — paste:

```
First internal build. Voice, camera video and screen sharing on Android;
text chat, threads, attachments and search. Self-hosted: you need a BSFChat
server to sign in to. Report bugs to support@bsfchat.com.
```

4. **Save**. Do not press *Review release* yet — §5 has to be done first, and
   the Console will list the missing items for you.

Add yourself as an internal tester under **Testers** (an email list of one is
allowed) so you can install the build while you work through the rest.

---

## 3. Data Safety form

**Path:** Play Console → *Policy and programmes* → **App content** → **Data
safety** → *Start*.

This is a legal declaration. Everything below is cited either to the live
privacy policy or to a file. Where I could not verify something, it says so
instead of guessing.

The one judgement that governs every row: **for a self-hosted app, "collected"
is answered against the server the user chooses, not against us.** Google's
definition of collection is *transmitted off the user's device*, and it does not
care who owns the receiving server. "We never see it" is not a reason to answer
No — it is a reason to explain in the free text (§3.6), which is what the policy
itself does in its section 1.

### 3.1 Screen 1 — Data collection and security

Asked in this order.

| # | Question | Answer | Justification |
| --- | --- | --- | --- |
| 1 | Does your app collect or share any of the required user data types? | **Yes** | Messages, attachments, display name and a user ID all leave the device for a chat server. Policy §4 enumerates exactly what the server stores. |
| 2 | Is all of the user data collected by your app encrypted in transit? | **Yes** — *but read §7.3 before you tick it* | Policy §4: *"Traffic between the app and the server is protected by TLS, and on our deployment the connection is HTTPS throughout."* `src/net/ServerDiscovery.cpp:32` prepends `https://` to anything typed without a scheme. The caveat in §7.3 is a user-typed `http://` LAN server. |
| 3 | Do you provide a way for users to request that their data be deleted? | **Yes** | Policy §10: *"There is a Delete account button in the app, in your settings below Log out."* Implemented client-side in `client/src/net/DeactivateFlow.h` + `client/qml/components/DeleteAccountDialog.qml`; server route `POST /_matrix/client/v3/account/deactivate` is live on production (probed 2026-09-29: returns 401 auth-required, not 404). Plus `support@bsfchat.com` for anything beyond that scope, per policy §10 and §12. |
| 4 | *(optional)* Your app has been independently validated against a global security standard (MASA) | **No** | Two internal audits exist (`RC-SECURITY-PLAN.md`, 47 findings) and neither was independent. Ticking this without a MASA assessment is a false declaration. |

There is no Data Safety question about **encryption at rest**, and nothing you
write should imply it. Policy §9: *"Everything else on a chat server is stored
unencrypted, and you should know that."*

### 3.2 Screen 2 — Data types

Tick exactly these boxes and no others. Categories are given as the Console
labels them.

**Personal info**
- [x] **Name**
- [x] **Email address**
- [x] **User IDs**
- [ ] Address, Phone number, Race and ethnicity, Political or religious beliefs, Sexual orientation, Other info — **none**

**Messages**
- [x] **Other in-app messages**
- [ ] Emails, SMS or MMS

**Photos and videos**
- [x] **Photos**
- [x] **Videos**

**Audio files**
- [x] **Voice or sound recordings**
- [ ] Music files, Other audio files

**Files and docs**
- [x] **Files and docs**

**App activity**
- [x] **Other actions**
- [ ] App interactions, In-app search history, Installed apps, Other user-generated content

**Everything else — leave entirely unticked:** Location (both), Financial info
(all), Health and fitness (both), Calendar, Contacts, Web browsing history, App
info and performance (crash logs, diagnostics, other), Device or other IDs.

Two of those deserve a sentence each because they look like they should be
ticked:

- **Device or other IDs — No.** The device ID BSFChat stores is *issued by the
  chat server per sign-in* (policy §4: *"Not a hardware identifier: it
  identifies a session, not your phone"*). It belongs under **User IDs**, which
  is ticked. No IDFA, no Android Advertising ID, no `getSerial`, no MAC —
  grepped across `client/src`, `client/android`, `client/qml`: zero hits for any
  analytics, attribution, crash or ad SDK. Manifest declares no
  `com.google.android.gms.permission.AD_ID`.
- **In-app search history — No.** Search runs as a request against the user's
  own server; nothing stores the query. `server/src/api/SearchHandler.cpp` reads
  the FTS index and writes nothing; no `searchHistory`/`recentSearch` anywhere
  in the client.

### 3.3 Screen 3 — Data usage and handling, type by type

For each ticked type the Console asks four things: **Collected / Shared**,
**Processed ephemerally?**, **Required or optional?**, and **Purposes**.

**Shared is No for every single type.** Google's definition of sharing is
*transfer to a third party*, and it explicitly excludes transfers the user
initiates and reasonably expects. Sending a message to a channel is exactly
that. There is no SDK, no ad network, no analytics vendor and no push provider
in the app — policy §2 and §5, and the manifest confirms it (no Play Services,
no Firebase).

| Data type | Collected | Shared | Ephemeral | Required / Optional | Purposes | Justification |
| --- | --- | --- | --- | --- | --- | --- |
| **Name** | Yes | No | No | **Required** | App functionality | Display name and per-server nickname; policy §4 *"Your account"*. Shown to other members, so it cannot be optional. |
| **Email address** | Yes | No | No | **Optional** | App functionality; Account management | Policy §6: *"An email address, only if you chose to give one. It is optional, it is never verified, and nothing is ever sent to it."* Only exists if the user signs in with a BSFChat ID *and* types one. |
| **User IDs** | Yes | No | No | **Required** | App functionality; Account management | Matrix user ID plus a server-issued device ID (policy §4). |
| **Other in-app messages** | Yes | No | No | **Required** | App functionality | Policy §4 *"Messages"*: stored in plain text, with a second copy in the search index. This is the product. |
| **Photos** | Yes | No | No | **Optional** | App functionality | Two distinct things under one box: attachments and avatars (retained on the server, EXIF **not** stripped — policy §4 says so explicitly), and live camera frames in a call (never recorded). Because the retained case exists, do **not** mark this ephemeral. |
| **Videos** | Yes | No | **Yes** | **Optional** | App functionality | Video attachments, plus live camera video and screen share. Policy §7: call media is transmitted between participants; nothing records it. Tick *processed ephemerally* only if you upload **no** video attachments — see the note below. |
| **Voice or sound recordings** | Yes | No | **Yes** | **Optional** | App functionality | Live voice only. Policy §7 describes the stream; nothing in `client/src/voice/` writes audio to disk or to the server. The stream leaves the device, so it is *collected*; it is not retained, so it is *ephemeral*. |
| **Files and docs** | Yes | No | No | **Optional** | App functionality | Any MIME type through the system file picker plus the `ACTION_SEND` share target (manifest `intent-filter`). Retained on the server indefinitely — policy §10 *"Messages and attachments: indefinitely"*. |
| **Other actions** | Yes | No | No | **Required** | App functionality | Reactions, reading position, block list, notification preferences. Policy §4 lists all four as server-stored so they follow the account. |

> **On "Videos" and ephemerality.** The Console applies one ephemeral flag per
> data type, and BSFChat's video is two different things: a live stream that is
> never stored, and an uploaded `.mp4` attachment that is stored for ever. If a
> user can attach a video file — they can, the picker takes any MIME type —
> then **Videos is not processed ephemerally** and the honest answer is to leave
> that box unticked. *Recommended: leave Videos unticked for ephemeral.* Tick it
> only for **Voice or sound recordings**, where there genuinely is no retained
> case. I have written the table above the conservative way round and flagged it
> here rather than quietly choosing.

> **On "Other actions" vs "App interactions".** Reading position is arguably an
> app-interaction record. I have put it under *Other actions* because Google's
> *App interactions* means usage records kept by the developer (page views,
> taps), and the read marker is synchronisation state for the user's own unread
> badges that never reaches us — policy §2: *"Your own reading position does go
> to your own server, so that your unread badges agree across your phone and
> your desktop; it is not shared onwards."* If you would rather be maximally
> conservative, also ticking **App interactions** costs nothing on the public
> label and removes the only arguable under-declaration in the form.

### 3.4 What the form has no box for, and what to do about it

Three real disclosures in the policy have no matching Data Safety data type.
Google's list has no IP-address category, so these go in the free text (§3.6)
and nowhere else. Do not invent a category for them.

| Disclosure | Where it is in the policy |
| --- | --- |
| Peer-to-peer calls expose each participant's IP to the others, mitigated by the "Hide my IP address" setting | §7 |
| Link previews fetch third-party pages **from the reader's device**, disclosing the reader's IP to any site a sender links | §8 |
| nginx on `chat.bsfchat.com` / `id.bsfchat.com` writes an access log containing IP addresses | §9 |

### 3.5 Things that will change this form later

Two are already written into the policy as planned (§14), and both would require
the Data Safety form to be edited **before** the release that ships them:

- **Push notifications** (§5) — would add Apple and Google as recipients of
  notification content. That is a *shared with third parties* change.
- **Verified email and password reset** (§6) — would move Email address from
  optional-and-inert to a delivery path through a mail provider.

### 3.6 Free text — paste into the Data Safety "additional details" box

Verified against the policy line by line. The link-preview paragraph matches
policy §8; the "never recorded" clause matches §7.

```
BSFChat is self-hosted. Messages, attachments and profile details are sent to a
BSFChat server chosen by the user, which in most cases is operated by the user
or their own community rather than by us. The developer operates one optional
public server and one optional sign-in provider; neither is required to use the
app. The app contains no analytics, no crash reporting, no advertising and no
third-party SDKs, and sends nothing to the developer about how it is used.
Voice, video and screen-share streams are transmitted live between participants
and are never recorded or stored.

Voice and video calls connect directly between participants where the network
allows, which means participants' devices see each other's IP addresses. A
"Hide my IP address" setting forces every call through a relay instead.

When a message contains a link, the app fetches that page from the user's own
device to show a preview, so the linked site receives the reader's IP address in
the same way it would if the reader opened the link in a browser. No preview
data is sent to or stored by the developer.

Messages are not end-to-end encrypted, and data on a chat server is not
encrypted at rest. The privacy policy states both plainly.
```

---

## 4. Content rating (IARC)

**Path:** App content → **Content rating** → *Start questionnaire*.

### 4.1 Before the questions: email and category

| Field | Answer |
| --- | --- |
| Email address | `support@bsfchat.com` |
| Category | **Utility, Productivity, Communication, or Other** |

**Not "Social Networking".** This is the choice that most affects the outcome
and it is the one that has to line up with the Apple answers. You answered
Apple's *Social Media* question **No**, and that is defensible here for the same
reason: BSFChat has no public feed, no follower graph, no profile discovery and
no public directory of servers or channels — verified, there is no
`publicRooms`/room-directory code path in the client, and `ServerDiscovery` is
Matrix `.well-known` resolution for a URL the user types, not a browse feature
(`client/src/net/ServerDiscovery.h:1-25`). It is a communication client. Picking
*Social Networking* would open a block of questions written for social networks
and would be answering a different app's questionnaire.

### 4.2 Content questions

| Question | Answer | Justification |
| --- | --- | --- |
| Violence — realistic, cartoon, or otherwise | **No** | IARC asks about content *the app provides*. BSFChat ships no content. |
| Sexuality / nudity | **No** | Same. |
| Language / profanity | **No** | Same. The UGC question below is where user behaviour is captured; answering Yes here would be declaring that *you* supply profanity. |
| Controlled substances — drugs, alcohol, tobacco | **No** | |
| Gambling — simulated or real | **No** | No gambling mechanics, no loot mechanics, no currency. |
| Fear / horror | **No** | |
| Discrimination | **No** | |
| Miscellaneous — does the app contain any other content a parent might object to | **No** | The UGC questions cover it. |

### 4.3 Interactive-elements questions — these set the rating

| Question | Answer | Justification |
| --- | --- | --- |
| **Does the app allow users to interact or communicate with each other?** | **Yes** | The whole app. Text, files, images, live voice, camera video and screen share. Matches the Apple **UGC = Yes** answer exactly. |
| **Can users share their current location with other users?** | **No** | No location permission is declared in `client/android/AndroidManifest.xml`, no geolocation API is called, and there is no send-my-location feature. *Be ready to defend it:* peer-to-peer calls expose an IP address and an IP is coarsely geolocatable — policy §7 says so. IARC's question is about a location-*sharing feature*, and there is none. Do not tick this. |
| **Can users share personal information with other users?** | **Yes** | Free-text messages and arbitrary file uploads. A user can type anything. |
| **Does the app allow users to purchase digital goods?** | **No** | No billing library, no IAP, no payment code — grepped `client/src`, `client/android`, `CMakeLists.txt`: zero hits for billing/StoreKit/SKProduct. |
| **Does the app contain ads?** | **No** | Policy §2: *"No advertising and no ad identifiers."* |
| **Does the app provide unrestricted access to the internet — a browser or search?** | **No** | There is no WebView, no WebEngine and no in-app browser (`client/qml/mobile/MobileMain.qml:583` says so in the tree, and a grep confirms it). Links open in the system browser, which IARC does not count. Matches Apple **Unrestricted Web Access = No**. |
| Is user-generated content moderated? | **Yes** | In-app blocking and reporting on every platform, plus per-server administrators. The report endpoint is now live — see §7.1. |
| Does the app share user data with third parties? | **No** | Consistent with §3.3. |
| Does the app collect precise location? | **No** | |

### 4.4 Expected outcome, and where the two stores diverge

Expect **ESRB Teen / PEGI 12 / USK 12 / ClassInd 12 / IARC "Rated for 12+"** —
the band every unmoderated-UGC communication app lands in. Do not appeal it.
Understating UGC is a policy violation and Teen costs you nothing.

Places where Play and Apple **force different answers, by design** — none of
these is an inconsistency to fix:

| Apple asked | You answered | Play's nearest equivalent | Why they differ |
| --- | --- | --- | --- |
| Social Media | **No** | IARC *category* choice | Apple asks whether the app is a social-media app. IARC has no such question — it asks you to pick a category up front. Picking *Utility, Productivity, Communication, or Other* is the consistent choice. |
| Age Assurance | **No** | *no equivalent* | Play has no age-assurance question. Its analogue is **Target audience and content** (§5.3), which asks which age brackets the app targets rather than whether you verify age. Consistent with policy §11: *"We do not verify anybody's age."* |
| Parental Controls | **No** | *no equivalent* | Play only asks about parental controls inside the Families programme, which this app is not in. |
| Unrestricted Web Access | **No** | Same question, verbatim | Identical answer. |
| UGC | **Yes** | *Users interact* + *Users share personal info* | Identical substance, split across two questions. |
| Outcome **13+** | | Outcome **Teen / PEGI 12** | Different scales, same band. Nothing to reconcile. |

One genuine divergence worth knowing about: Apple's rating flow produced
**13+**, and IARC will produce **PEGI 12** in Europe — a *lower* number than the
policy's own minimum of 13. That is normal (PEGI has no 13 band) and is not a
contradiction of the policy, because the policy's 13 is a contractual minimum in
`/terms` §3, not a content rating. Do not try to force IARC to 16 to "match".

---

## 5. The rest of App content

Each of these blocks the rollout. None blocks the upload.

### 5.1 Privacy policy

`https://bsfchat.com/privacy/` — live, mandatory for any app requesting camera
or microphone.

### 5.2 App access

Select **All or some functionality is restricted**, then add one instruction
set:

| Field | Value |
| --- | --- |
| Name | `Sign in to the public BSFChat server` |
| Username | *(the demo account from §1)* |
| Password | *(ditto)* |
| Any other information | See the block in §8.5 — paste the sign-in steps there |

### 5.3 Target audience and content

| Question | Answer | Justification |
| --- | --- | --- |
| Target age groups | **13–15, 16–17, 18 and over** | `/terms` §3 and `/privacy` §11: *"You must be at least 13."* Selecting 18+ only would contradict your own published terms. |
| Could your app unintentionally appeal to children? | **No** | Policy §11: *"we do not collect anything designed to be attractive to or targeted at children."* No cartoon styling, no games, no rewards. |
| Do you want the app in the Designed for Families programme? | **No** | The app is not for children and has no age gate. |

Selecting a 13–15 bracket does not put the app in Families, but it does mean the
store listing must not be child-directed. The copy in §8 is not.

### 5.4 Ads

**Does your app contain ads?** → **No.** Policy §2 and §7 of the terms both say
there is no advertising and no plan for any.

### 5.5 Advertising ID

**Does your app use an advertising ID?** → **No.** The manifest declares no
`com.google.android.gms.permission.AD_ID` and the packaged app contains no Play
Services (policy §2: *"the packaged app contains no Google Play Services, no
Firebase and no AndroidX code"*).

### 5.6 Foreground service permissions — **four** declarations

App content → **Foreground service permissions**. Verified against
`client/android/AndroidManifest.xml`: three `<service>` elements declaring four
types between them, and four matching `FOREGROUND_SERVICE_*` permissions.

| Declared type | Service | Manifest evidence | Video must show |
| --- | --- | --- | --- |
| `microphone` | `com.bsfchat.client.VoiceService` | `foregroundServiceType="microphone\|camera"`; `FOREGROUND_SERVICE_MICROPHONE` | Joining a voice channel, speaking, then switching to another app while the call keeps running |
| `camera` | same service, same attribute | `FOREGROUND_SERVICE_CAMERA` | Turning the camera on inside a voice channel |
| `dataSync` | `com.bsfchat.client.SyncService` | `foregroundServiceType="dataSync"`; `FOREGROUND_SERVICE_DATA_SYNC` | A message arriving as a notification with the app backgrounded |
| `mediaProjection` | `com.bsfchat.client.MediaProjectionService` | `foregroundServiceType="mediaProjection"`; `FOREGROUND_SERVICE_MEDIA_PROJECTION` | The system screen-capture consent dialog, then a shared screen |

Each needs a written justification and a video link (an unlisted YouTube URL is
the normal route). Paste these:

**microphone**
```
BSFChat is a voice and text chat client. While the user is connected to a voice
channel, VoiceService runs in the foreground so Android does not kill the
process and drop the call when the user switches to another app or locks the
screen. The service runs only while the user is in a voice channel and stops
when they leave it. It is user-initiated in every case: there is no way for the
app to start capturing audio without the user joining a channel.
```

**camera**
```
The same VoiceService anchors camera capture, because a user can turn their
camera on inside a voice channel and the video call must survive backgrounding
for the same reason the audio must. The camera is off by default in every call
and is turned on only by an explicit user action.
```

**dataSync**
```
SyncService maintains the connection to the chat server the user has chosen, so
that an incoming message can raise a notification while the app is backgrounded.

BSFChat deliberately does not use Firebase Cloud Messaging. The app is a client
for self-hosted servers: routing every user's notification traffic through
Google would mean the content of private messages passing through a third party
that the user did not choose, which contradicts the product's core promise and
its published privacy policy. There is no FCM, no Firebase and no Google Play
Services dependency anywhere in the app. Notifications are generated locally on
the device from data the app already holds, and nothing about them is sent to
any third party.

The service runs only while the user is signed in to at least one server and
stops when they sign out.
```

**mediaProjection**
```
MediaProjectionService exists because the Android platform requires a foreground
service of type mediaProjection to hold a MediaProjection token. It is started
only after the user has accepted the system screen-capture consent dialog, and
only to share the screen into a voice channel the user is already in. It stops
as soon as screen sharing stops.
```

> `dataSync` is the one Google pushes back on hardest; their stated position is
> that it is not for indefinite background work and that apps should use FCM.
> The argument above is the honest one and it is the one your own policy makes.
> If it is rejected, the fallback documented in `client/docs/android-release.md`
> §4 is to sync only while the app is foreground or recently used — a code
> change, not a paperwork one.

### 5.7 Declarations with nothing to declare

| Section | Answer |
| --- | --- |
| Government apps | **No** |
| Financial features | **My app doesn't provide any financial features** |
| Health apps | **No** — none of the health declarations apply |
| News apps | **No** |
| Data deletion | see §5.8 |
| Photo and video permissions | **not shown** — the manifest declares neither `READ_MEDIA_IMAGES` nor `READ_MEDIA_VIDEO`; attachments come from the system file picker (policy §2). If the Console shows the form anyway, answer that the app does not request those permissions. |
| Sensitive permissions (all-files access, SMS/call log, `QUERY_ALL_PACKAGES`, accessibility, VPN, exact alarm) | **none declared** — the manifest requests only `INTERNET`, `ACCESS_NETWORK_STATE`, `CAMERA`, `RECORD_AUDIO`, `MODIFY_AUDIO_SETTINGS`, `WAKE_LOCK`, `POST_NOTIFICATIONS` and the four `FOREGROUND_SERVICE_*` permissions. `CAMERA`, `RECORD_AUDIO` and `POST_NOTIFICATIONS` are ordinary runtime permissions with no separate Play declaration form. |

### 5.8 Data deletion declaration

Play asks for this separately from Data Safety, and it wants a route reachable
**from the web**, not only in-app.

| Field | Answer |
| --- | --- |
| Can users request that their account and data be deleted? | **Yes** |
| In-app route | `… menu → Your profile → Delete account`. Confirmation is typing your username, plus your password if the account has one — accounts created through the BSFChat ID sign-in provider have no chat-server password and are deleted on the bearer token alone (`client/src/net/DeactivateFlow.h`). |
| Web URL | `https://bsfchat.com/support` |
| Does deletion remove all data? | **No — some data is retained.** Use the text below. |

Retention explanation — **corrected**; the version in `STORE-LISTING.md` §5.3
claims backups age out within thirty days, which the live policy contradicts
(see §7.2):

```
Deleting an account destroys the password and every sign-in token, erases the
display name, nickname and avatar, removes the account from every channel and
direct message, and deletes the block list, notification settings and reading
positions.

Some data is deliberately retained, and the privacy policy says so before the
user presses the button:

  - Messages the user has already sent remain in the conversations they were
    part of, along with the display name recorded in the historical membership
    events of those channels, because a group conversation belongs to all of
    its participants and deleting one account cannot silently cut holes in
    other people's history. Individual messages can be deleted before the
    account is.
  - Files the user uploaded remain attached to those messages.
  - The username is permanently retired so that nobody can re-register it and
    appear to be that person in old conversations.
  - Moderation reports and the administrative audit log are retained as
    moderation records.
  - Backups taken before the deletion still contain the data. Backups are
    currently taken and removed by hand, with no automatic expiry schedule, so
    no fixed retention period is claimed.
  - Deleting a chat-server account does not delete a BSFChat ID at
    id.bsfchat.com. There is no in-app button for that yet; it is removed by
    hand on request to support@bsfchat.com.

Full detail: https://bsfchat.com/privacy#deletion (and section 6 of the same
page for the BSFChat ID).
```

---

## 6. What Play blocks the release on

### 6.1 The Console checklist

When you press *Review release* on internal testing, Play lists what is missing.
Expect, in rough order of how long each takes:

1. Store listing — app icon, feature graphic, phone screenshots, short and full
   description (§8).
2. App content — everything in §5, including the four foreground-service videos.
3. Data safety — §3.
4. Content rating — §4.
5. Countries/regions selection.
6. Internal testers list.

### 6.2 Production access — **check this, I could not**

A Play developer account registered as an **individual on or after 13 November
2023** must run a **closed** test with **at least 12 testers opted in for 14
continuous days** before it can apply for production access. Organisation
accounts, and individual accounts older than that, are exempt.

Check **Play Console → Setup → Advanced settings** (or the account type shown on
the developer account page). It decides whether "internal test today" means
"production in two weeks plus review" or "production whenever you like". Nothing
in the code tells me this and I have not touched your account.

If the rule applies: internal testing (§2.4) does **not** count. You will need a
separate **closed** track with twelve real people on it.

---

## 7. Findings — where the drafted answers meet reality

These are the places where an answer contradicted the live policy, the code, or
the brief I was given. Read all five.

### 7.1 The "one hard blocker" is cleared — the report endpoint is live

`STORE-LISTING.md` §0.3 is out of date, and it was the file's highest-risk item.

- `server` main is now `f53c2ad`, *"Merge pull request #3 from
  BSFChat/feat/ugc-safety"* (2026-09-22). `src/api/ReportHandler.cpp` and `.h`
  are present on main.
- Production is serving the route. Probed 2026-09-29 against
  `chat.bsfchat.com`:

  | Path | Status |
  | --- | --- |
  | `POST /_matrix/client/v3/rooms/{id}/report/{event}` | **401** |
  | `POST /_matrix/client/v3/users/{user}/report` | **401** |
  | `POST /_matrix/client/v3/account/deactivate` | **401** |
  | `POST /_matrix/client/v3/rooms/{id}/nonexistent_thing/{x}` *(control)* | 404 |
  | `POST /_matrix/client/v3/totally_made_up` *(control)* | 404 |

  401-not-404 against a 404 control means the routes exist and are behind auth.

**Two things I could not verify, and you should:**

1. **The deployed build is untagged.** `git tag --contains f53c2ad` in `server`
   returns nothing — the UGC merge is in **no** tag, including
   `v0.0.52-rc.1` (tagged the same day). Production is therefore running a
   `:sha` image newer than the last tag, which is consistent with the
   deploy-by-`:sha` convention, but there is no unauthenticated version endpoint
   on the host (`/_bsfchat/version`, `/version`, `/healthz`, `/capabilities` all
   404) so I could not read the exact build.
2. **Nobody has filed a report end-to-end.** A 401 proves the route is mounted,
   not that an authenticated report writes a row and reaches an administrator.
   **Do this before you tick "user-generated content is moderated" in §4.3:**
   sign in as the demo account, long-press a message from the second account,
   report it, and confirm it appears for an admin. That is also the exact
   sequence a reviewer will follow from §8.5.

### 7.2 `STORE-LISTING.md` §5.3 states a backup retention the policy denies

The known defect, confirmed. The drafted deletion declaration says *"Backups age
out within 30 days."* The live policy, §9, says the opposite in as many words:

> There is no automatic schedule and no automatic expiry. Backups are taken by
> hand and deleted by hand … assume a backup of your data may exist for longer
> than thirty days.

Declaring a 30-day backup expiry to Google would be declaring something your own
published policy tells users is not true. §5.8 above replaces it with
"currently taken and removed by hand, with no automatic expiry schedule, so no
fixed retention period is claimed."

**The adjacent claim that ran ahead of reality in the same way:** policy §9's
nginx log paragraph — *"we do not yet enforce a period of our own … The
intention is fourteen days."* Data Safety has no retention question, so this one
does not bite on any form; it becomes a problem only if you write "logs are kept
14 days" into free text somewhere. Don't.

### 7.3 "Encrypted in transit" is not literally true for a LAN server

This is the answer I am least comfortable with, so it is written out rather than
smoothed over.

Recommended answer: **Yes**. But:

- `client/src/net/ServerDiscovery.cpp:24-40` accepts a `http://` scheme if the
  user types one explicitly, and does not warn.
- The UI actively suggests one: `client/qml/components/ServerSidebar.qml:504`
  reads *"Use the full base URL, e.g. `http://192.168.1.20:8448`"*, with
  `http://localhost:8448` as the placeholder at `:513` and again at
  `client/qml/components/LoginDialog.qml:366`. A LAN address is not localhost.
- There is no `android:usesCleartextTraffic` attribute and no
  `network_security_config.xml` in `client/android/`. The platform's cleartext
  block is enforced by Android's Java networking stack; Qt uses its own sockets,
  so it is very unlikely to apply here. **I did not test this on a device** —
  running the app is out of scope — so treat it as unverified either way.

So a user who types `http://192.168.1.20:8448` almost certainly sends messages
in cleartext on their own LAN, and the Console question is *"is **all** of the
user data collected by your app encrypted in transit?"*.

Why **Yes** is still the right answer: HTTPS is the default and is auto-prepended
for anything typed without a scheme; every server the developer operates is
HTTPS-only behind Cloudflare (policy §13); Google's guidance is about whether the
app uses industry-standard encryption for its network traffic, not whether a
user can point it at their own plaintext host; and every comparable self-hosted
client answers Yes. **Your call, not mine.** If you want the answer to be
unambiguously true, the fix is small: a cleartext warning in the add-server
dialog, or an Android `networkSecurityConfig` that permits cleartext only for
`localhost` and RFC1918 ranges. Neither is a blocker.

### 7.4 The brief for this document was wrong about how deletion works

The task I was given said *"the policy says deletion is done **by hand** on
request — make sure the answer reflects the truth, not the aspiration."*
Checked: that is not what the policy says, and the aspiration runs the other
way.

Policy §10 describes a real, immediate, self-service deletion:

> There is a Delete account button in the app, in your settings below Log out.
> You must type your username, and your password, to confirm it. It is immediate
> and it cannot be undone.

It is implemented (`client/src/net/DeactivateFlow.h`,
`client/qml/components/DeleteAccountDialog.qml`) and the server route is live
(§7.1). By-hand deletion is the *secondary* route, and it covers two specific
things:

- **The BSFChat ID at `id.bsfchat.com`.** Policy §6: *"The 'Delete account'
  button in the app deletes your account on the chat server. It does not yet
  delete your BSFChat ID; there is no button for that. Email
  support@bsfchat.com and we will remove it by hand."*
- **Anything beyond what §10 deletes** — a particular message, a particular
  file. Policy §10 and §12.

So Data Safety question 3 is **Yes**, and the §5.8 declaration must name the
BSFChat ID caveat — which `STORE-LISTING.md` §5.3 currently omits. That omission
is the second defect in that table, alongside the backups line.

### 7.5 Smaller corrections to `STORE-LISTING.md`

| Where | Says | Actually |
| --- | --- | --- |
| §6, Apple outcome | "expect **12+**" | Apple's bands are now 4+/9+/13+/16+/18+. There is no 12+. You already have **13+**. |
| §6, reasoning | "Minimum age 13 (16 in UK/EEA) in the terms" | `/terms` §3 and `/privacy` §11 say **13**, with "where the law where you live sets a higher age, that is the age that applies to you". Neither page names 16 or the EEA. Do not repeat the 16 anywhere in the Console. |
| §7, §8 | points at `client/docs/store-submission-runbook.md` for the screenshot shot lists and the end-to-end runbook | **That file does not exist.** `client/docs/` contains only `android-release.md`, `ios-release.md` and `design/`. The Android half of the runbook is `client/docs/android-release.md`; the shot lists are nowhere. §8.4 below is the replacement for the Play set. |
| §3.1 | data types named "Photos — camera / live video", "Photos — photo library" | Not Console labels. The real categories are **Photos and videos → Photos / Videos**. §3.2 uses the real ones. |
| §2.1 | versionCode worked from `0.0.48-rc.2 → 4702` | Correct arithmetic, wrong build. The artifact you are uploading is `0.0.48-rc.8` → **4708**, confirmed by `aapt2` in the CI log. |
| §2.3 | full description hard-wrapped at ~80 columns | Play renders newlines literally, so it would display ragged on a phone. §8.2 is the same text, unwrapped, 2,941 characters. |

---

## 8. Store listing

### 8.1 Short description — 78 / 80

```
Self-hosted chat for teams and communities. No ads, no tracking, no telemetry.
```

### 8.2 Full description — 2,941 / 4,000

Unwrapped so Play renders it as paragraphs. Identical wording to
`STORE-LISTING.md` §2.3.

```
BSFChat is chat without the bullshit.

No ads. No analytics. No crash reporting. No advertising ID, no Firebase, no Google Play Services dependency, no third-party SDKs of any kind. Not "anonymised" analytics — none at all. The source is public, so you can check rather than trust.

And no servers of ours in the middle. BSFChat is self-hosted: your messages live on a server you or your community runs — a rented box, a machine in the office, a Pi in a cupboard. We do not have a copy, we cannot read your channels, and on most servers we do not know they exist.

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
• Background notifications on your own device — no Firebase, nothing sent to Google

ONE ACCOUNT, EVERY SERVER

Sign in once with a BSFChat ID and every server you have joined comes back. Sign-in is standard OpenID Connect in your own browser — never an embedded web view. Or skip it: a server can issue plain accounts with no identity provider at all.

VOICE THAT DOES NOT PHONE HOME

Voice and video go directly between participants when the network allows, and through a relay when it does not. A direct call means the other participants' devices can see your IP address — that is how direct connections work everywhere — so there is a "Hide my IP address" setting that forces every call through the relay. If it is on and no relay is available, the call is refused rather than quietly connected anyway.

YOU ARE NOT THE PRODUCT

No paid tier. No upsell. No "upgrade for bigger uploads". No engagement mechanics, no dark patterns, nothing to buy, because there is nothing we are holding back.

SAFETY

Block anyone from their profile card, on any server. They are not told, and their messages and invitations stop reaching you on every device you sign in on. Report a message or a user to that server's administrators. Delete your account from inside the app — read the privacy policy first, because messages you have already sent stay in their channels, and we would rather say so now.

HONEST LIMITS

• You need a BSFChat server to connect to. Join the public one at chat.bsfchat.com, or run your own from github.com/BSFChat in about five minutes.
• Messages are not end-to-end encrypted. Traffic is protected by TLS and your server holds your history. We will not claim otherwise.
• Echo cancellation relies on your device's own. Use headphones on speakerphone and everyone will thank you.

bsfchat.com/privacy · bsfchat.com/support · github.com/BSFChat
```

Two sentences in there are claims about the product, not the app: *"The source
is public"* is true today and is deliberately not the words "open source", which
`/terms` §8 contradicts until a LICENSE file exists. Leave it as written.

### 8.3 Graphics

| Asset | Requirement | Status |
| --- | --- | --- |
| **App icon** | 512 × 512, 32-bit PNG with alpha, ≤ 1 MB | **Exists.** `client/branding/BSFChat.png` is 512 × 512, 8-bit RGBA. Upload as-is. |
| **Feature graphic** | 1024 × 500, PNG or JPEG, no alpha, ≤ 15 MB | **Does not exist.** Blocks the rollout. It is the banner at the top of the listing and it must not be a screenshot collage. The simplest honest one: the logo from `client/branding/BSFChat-1024.png` on a flat brand background with the words *"Self-hosted chat. No ads, no tracking."* |
| **Phone screenshots** | 2 minimum (4–8 recommended), PNG or JPEG, 320–3840 px on each side, aspect ratio between 16:9 and 9:16 | **Do not exist for Android.** Blocks the rollout. |
| Tablet screenshots | optional | Skip. No tablet layout. |

**The nine iPhone screenshots in `~/dev/gamechat/shots-ios-20260925` cannot be
reused.** They are 1320 × 2868, an aspect ratio of 0.460, which is taller than
Play's 9:16 (0.5625) limit — Play will reject them on upload. They also carry an
iOS status bar. Android screenshots have to be captured separately.

### 8.4 Shot list for the Play phone set

`STORE-LISTING.md` points at a runbook file that does not exist, so here is the
list. An emulator is acceptable for Play — **do not run anything on the owner's
own Mac's camera or microphone**; an emulator or the test phone is the place for
this. Capture at the device's native resolution; a 1080 × 2340 phone gives
0.462, so **crop or letterbox to 1080 × 1920 (9:16)** before uploading.

1. A channel with a live conversation in it — the product in one frame.
2. The server/channel sidebar, showing categories and more than one server.
3. A voice channel with two participants connected and the speaking ring lit.
4. Camera video in a voice channel.
5. Screen sharing — the Android-only feature and the reason this copy differs
   from Apple's.
6. The profile card with the **Block** control visible.
7. Settings, showing *"Hide my IP address"* — the claim the description makes.
8. Search results across history.

### 8.5 Store settings and reviewer instructions

| Field | Value |
| --- | --- |
| App category | **Communication** |
| Tags | Chat, Messaging |
| Store listing contact — email | `support@bsfchat.com` |
| Store listing contact — website | `https://bsfchat.com/support` |
| Store listing contact — phone | leave blank |
| External marketing | leave off |

Reviewer instructions — verified against the app's actual navigation as
described in `/support` and `STORE-LISTING.md` §5.2, with the retention line
corrected and the BSFChat ID caveat added:

```
BSFChat is a client for self-hosted chat servers. Messages live on a server the
user or their community runs; we operate one public server, used by the demo
account below. Accounts are not created in the app, so please use these
credentials rather than registering.

  Server:   https://chat.bsfchat.com
  Username: <fill in>
  Password: <fill in>

Sign in: launch > "Show manual server entry" > enter the server URL > "Check" >
"Use password instead" > username and password.

USER-GENERATED CONTENT CONTROLS

  • Block: tap a person's name or avatar > profile card > lock button.
    Server-side, immediate, follows the account across every device the user
    signs in on. Manage at ... > Your profile > Blocked accounts.
  • Report: long-press a message > "Report message..." (or "Report user...").
    Free-text reason, with "also block them" ticked by default. Reports go to
    the administrators of the server the message was sent on, together with a
    retained copy of the message so that deleting it does not destroy the
    evidence.
  • Moderation: server administrators can remove content and ban accounts. On
    the server we operate we respond to reports within 24 hours.
  • Published terms with a zero-tolerance position on abusive content and on
    child sexual abuse material: https://bsfchat.com/terms

ACCOUNT DELETION

  In-app at ... > Your profile > Delete account. Confirmation is typing the
  username, plus the password if the account has one. Immediate and
  irreversible. Messages already sent remain in their channels, which the
  privacy policy states before the button is pressed. Deleting a chat-server
  account does not delete a BSFChat ID at id.bsfchat.com; that is removed by
  hand on request to support@bsfchat.com. Web route:
  https://bsfchat.com/support

FOREGROUND SERVICES

  • microphone (VoiceService): keeps the process alive during a voice call so
    Android does not kill it when the user switches apps mid-conversation.
  • camera (VoiceService, same service): the user can turn their camera on
    inside a voice channel, and the call - video included - must survive
    backgrounding for the same reason.
  • dataSync (SyncService): maintains the connection to the user's chosen
    server so messages can raise a local notification while the app is
    backgrounded. There is no Firebase Cloud Messaging in this app -
    notifications are generated on the device and no notification content is
    sent to Google.
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

---

## 9. Gap list

Everything Play will want that does not exist yet, and whether it stops the
upload or only the rollout.

### 9.1 Blocks the upload

Nothing, once the app entry exists and Play App Signing is accepted. The AAB is
signed, `targetSdk 36` clears the current minimum (35), the applicationId is
`com.bsfchat.app`, and versionCode 4708 has never been used.

Two things that *would* block it if they were wrong, and are not:

- **targetSdk** — 36, verified from the CI badging output. Play's floor for new
  apps is 35.
- **Signing** — the AAB is signed with the upload key. An unsigned bundle is
  rejected at upload. (CI makes a tag build with no keystore secret a hard
  failure, precisely for this.)

### 9.2 Blocks the release

| # | Gap | Notes |
| --- | --- | --- |
| 1 | **Feature graphic, 1024 × 500** | Does not exist anywhere. Cannot be a screenshot collage. |
| 2 | **Phone screenshots, ≥ 2 at 9:16–16:9** | Do not exist for Android; the iPhone set is the wrong aspect ratio (§8.3). Shot list at §8.4. |
| 3 | **Four foreground-service demo videos** | `microphone`, `camera`, `dataSync`, `mediaProjection` — §5.6. Longest lead time in the list; they need a real phone and an unlisted upload host. |
| 4 | **Data Safety form** | §3. Ready to paste. |
| 5 | **Content rating questionnaire** | §4. Ready to paste. |
| 6 | **App access credentials** | Needs the demo account from §1 to exist first. |
| 7 | **Target audience, ads, advertising ID, government/financial/health/news declarations** | §5.3–§5.7. Five minutes. |
| 8 | **Data deletion declaration** | §5.8. Ready to paste, with the backups line corrected. |
| 9 | **Countries and regions** | Must be chosen consistently with `/privacy` §11 — see §1. |
| 10 | **Internal tester list** | One email is enough for internal testing. |
| 11 | **`support@bsfchat.com` receiving mail** | Play emails it. A bounce is a rejection. |
| 12 | **Report filed end-to-end at least once** | §7.1. This is what makes §4.3's *"user-generated content is moderated: Yes"* a true statement. |

### 9.3 Does not block anything, but you should know

| Gap | Effect |
| --- | --- |
| **arm64-v8a only** | No `x86_64` or `armeabi-v7a` in the bundle. The listing will be unavailable on x86 Chromebooks and on the Play-enabled emulators reviewers sometimes use, and on 32-bit-only phones (rare at minSdk 28). Not a policy problem — Play's requirement is 64-bit *support*, which arm64-only satisfies. |
| **Icon is 512 px, not larger** | Play wants exactly 512 × 512, so this is right. `BSFChat-1024.png` is the source if you ever need to regenerate. |
| **No `networkSecurityConfig`** | See §7.3. |
| **No LICENSE file in any repo** | The listing copy says "the source is public", not "open source". Keep it that way until `STORE-LISTING.md` §0.6 is decided. |
| **Push notifications and email are in the policy as planned changes** | Both would require editing the Data Safety form before the release that ships them (§3.5). |
| **Artifact expires 2026-12-27** | After that, rebuild from `v0.0.48-rc.8`; same versionCode 4708. |

---

## 10. Order of operations

1. §1 prerequisites — mailboxes, demo accounts, seeded conversation.
2. File one report end-to-end against the demo accounts (§7.1). If it fails,
   stop; §4.3 and §8.5 both become untrue.
3. §2.1–§2.2 — download and check the AAB.
4. §2.3 — create the app, accept Play App Signing.
5. §2.4 — internal testing release, upload the AAB, **Save** only.
6. §3 Data Safety, §4 content rating, §5 remaining declarations. All
   copy-pasteable, roughly an hour.
7. Record the four foreground-service videos (§5.6) and capture the eight Play
   screenshots (§8.4). This is the long pole.
8. Produce the feature graphic (§8.3).
9. Fill the store listing (§8.1, §8.2, §8.3, §8.5).
10. Countries/regions (§1).
11. *Review release* → *Start roll-out to Internal testing*.
12. Check whether §6.2's closed-testing rule applies to your account before you
    plan anything beyond internal.
