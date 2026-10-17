# SafetyLens AR

An offline Android app that trains and certifies industrial workers in safety by testing what they would actually do in an emergency, in English, Hindi and Santali.

Built for problem statement PS 26041 (Smart India Hackathon): *AR-based vocational training simulator for industrial safety in Jharkhand's mining and manufacturing sector.*

## The problem

Jharkhand's mines, steel plants and mica units employ many young recruits with no industrial experience. Classroom safety training with static manuals shows under 20% retention after a week, live drills disrupt production, and VR headsets are out of reach for small mines and contract workers. DGMS Dhanbad recorded 48 fatal mine accidents in Jharkhand in 2022–23, many involving workers with fewer than 30 days on the job.

The gap isn't knowledge. It's behaviour under pressure, and nothing in common use tests that or tracks how skills fade over time.

## How it works

Each training module is a chain of steps. A step gives a short scenario briefing and then places four candidate objects at fixed bearings around the room. The worker pans the phone to find the right one, taps it, and answers a question about how to use it. A wrong choice triggers a consequence screen with a hint in the worker's language, and then a retry.

What makes it different from a quiz:

- **Wrong answers are instinct traps.** For example, a light switch can ignite gas, and gas incapacitates before you notice.
- **Pressure instead of a timer.** A rising gauge, a red edge pulse, screen shake and a synthesized siren model the stress of a real incident.
- **Certificates that fade.** A certificate carries a signed readiness score that decays over time, and the admin dashboard shows who is at risk and which skill faded fastest.
- **Built for low literacy.** Icons, colour and text all carry each state, and every question can be read aloud.

## What's built

**Training:** a zero-pressure practice run that teaches the three gestures (move to find, tap the object, choose an answer), plus five modules of three steps each: gas leak and confined space, fire and explosion, machinery safety, electrical hazards, and transport and heavy vehicles.

**Certification:** the score starts at 96 and loses points for wrong turns. Each certificate is signed with HMAC-SHA256 and encoded as a QR code. The in-app scanner decodes the QR, recomputes the signature and shows VALID or INVALID, with no network needed.

**Admin dashboard:** each admin sees only their own clan (a mine site, contractor or supervisor group). It shows workers, runs, average readiness, at-risk workers, per-module score, mistakes and time, and a readiness decay curve.

**Accounts and sync:** separate admin and worker logins. Firebase Auth and Firestore provide cloud sync with offline persistence, and the app falls back to local storage if Firebase isn't available.

**Languages:** the whole interface, briefings, questions, hints and dashboard are in English, Hindi and Santali.

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | Vanilla HTML, CSS and JavaScript |
| AR effect | Canvas HUD, `DeviceOrientationEvent`, CSS 3D transforms, SVG sprites |
| Signing and QR | Pure-JS HMAC-SHA256, qrcode-generator, jsQR |
| Offline | Service worker cache, `localStorage` fallback, Firestore offline persistence |
| Backend | Firebase Auth and Firestore |
| Mobile shell | Kotlin WebView (`minSdk 24`, `compileSdk 34`) |

It deliberately doesn't use Unity, ARCore, ML object detection or a custom server. That keeps it running on mid-range Android phones.

## Run and build

**In a browser:** open `web/index.html`. The camera, orientation and QR-scan features need a real phone, and a static fallback keeps the demo usable on a desktop.

**Android APK** (needs Android Studio or Gradle, an Android SDK and JDK 17):

```
cd android
gradlew assembleDebug
```

The APK is written to `android/app/build/outputs/apk/debug/app-debug.apk`. It bundles the whole `web/` app, so re-run the build after changing anything in `web/`.

## Repository layout

```
SIH_AR/
├── web/            the app (HTML, CSS, JS, service worker, dashboard)
│   └── js/         modules, search chain, AR HUD, store, dashboard, QR and HMAC
└── android/        Kotlin WebView wrapper with bundled assets
```

## Security

The `apiKey` in `js/firebase-config.js` is a Firebase web key, which is public by design because it ships inside every client app. Protection comes from restricting the key to this app's package in Google Cloud Console and from authenticated-only Firestore rules, for example:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /clans/{clanId} {
      allow read: if request.auth != null;
      allow write: if false;
    }
    match /workers/{workerId} {
      allow read: if request.auth != null;
      allow create, update: if request.auth != null;
      allow delete: if false;
    }
    match /certificates/{certId} {
      allow read, create: if request.auth != null;
      allow update, delete: if false;
    }
  }
}
```

Demo accounts are seeded for local testing only. Set your own credentials before any real deployment.

## Status and limitations

- A debug APK builds and the core logic (auth, clan scoping, dashboard, step chain, HMAC, QR round trip) is covered by a headless test harness.
- With Firebase disabled, data lives in `localStorage` and doesn't sync across devices.
- The walk-closer step uses the accelerometer with a fallback, because indoor motion sensing is unreliable.
- A release-signed APK and a demo video are not done yet.
- Further safety domains (PPE and dust exposure, blasting) are planned, and modules 1–5 are what's built.
