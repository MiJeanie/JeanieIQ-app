# JeanieIQ apps

Native iOS and Android shells for two JeanieIQ apps, built the same way as the MiJeanie app:

| Folder | App | Opens |
|---|---|---|
| `locum/` | JeanieIQ Locum | `/locum` on the JeanieIQ platform |
| `employee/` | JeanieIQ Employee | `/employee` on the JeanieIQ platform |

Each is a [Capacitor 8](https://capacitorjs.com) project whose `server.url` loads the live web
app, so a platform deploy reaches phones without a new store build. Only shell changes (name,
icon, colours, native plugins, address) need a rebuild. `www/index.html` is the offline page
shown when the phone cannot reach the site at launch.

The apps themselves live in the platform repo (`jeanieiq-web`, `apps/web`). Nothing here
duplicates them.

## App IDs

| | iOS bundle id (Apple) | Android package (Google Play) |
|---|---|---|
| Locum | `com.jeanieiq.locum.dev` | `com.jeanieiq.locum` |
| Employee | `com.jeanieiq.employee.dev` | `com.jeanieiq.employee` |

- **iOS uses `.dev` ids** because testing runs on a personal Apple Developer account. Once an
  App Store Connect app exists for a bundle id, that id belongs to that account, so the plain
  ids are kept free for the company account that will publish.
- **Android uses the real ids** because the Google Play account is the company's. A package
  name is permanent on Play: changing it later is a new listing, not an update.

## Address

Both apps currently open **`https://main.dev.jeanieiq.com`**, which is VPN-only. Test phones
must be on the VPN. Apple and Google reviewers cannot reach it, so a public release needs the
production address in `server.url` of both `capacitor.config.json` files.

## Builds (Codemagic)

`codemagic.yaml` has four workflows: `locum-ios`, `locum-android`, `employee-ios`,
`employee-android`. Each runs on a push to `main` that touches its own folder, and uploads to
TestFlight or Play internal testing as a draft. Nothing is released to the public from here.

One-time setup, all outside this repo:

1. **Apple:** an App Store Connect app for each `.dev` bundle id, and an App Store
   provisioning profile for each, signed with the team's distribution certificate and fetched
   into Codemagic (Code signing identities, iOS provisioning profiles). Codemagic does not
   create profiles itself; without one the build stops with "No matching profiles found".
   The iOS workflows use the App Store Connect key integration named `JeanieIQ` (the same one
   the MiJeanie build uses) and look the app up by bundle id.
2. **Google Play Console:** the two apps (`com.jeanieiq.locum`, `com.jeanieiq.employee`) in
   the company's Play account, with Play App Signing on.
3. **Codemagic, Team settings:**
   - Android keystore: both apps sign uploads with the existing `mijeanie_upload` keystore
     (the MiJeanie build's). Google allows one upload key across apps; with Play App Signing
     Google holds each app's real signing key, and a lost or leaked upload key is reset
     through Play support.
   - Play access uses the existing environment variable group `google_play` (the MiJeanie
     build's). Its service account holds "Release apps to testing tracks" at account level in
     the Play account, so it covers every app there, and cannot release to production.
4. **First Android upload by hand:** Google accepts API uploads only after the first bundle of
   a new app has been uploaded in Play Console. Download the `.aab` from the first Codemagic
   build and upload it once.
5. **Webhook:** the app is connected to Codemagic by SSH address with a deploy key, so pushes
   reach it only through the GitHub webhook to Codemagic's hook URL for this app.

Both iOS workflows have built and uploaded to TestFlight. The Android workflows have not run.

## Icons and splash screens

Generated from the platform's own icon SVGs (`apps/web/public/locum-icon.svg`,
`employee-icon.svg` in `jeanieiq-web`), rendered to `assets/` and expanded to every launcher
and splash size with `@capacitor/assets`. To change them: re-render `assets/`, then run, in
the app's folder:

```bash
npx @capacitor/assets generate --android --ios
```

Run it with `npx` rather than adding it to `package.json`: its dependency tree carries
`npm audit` findings, and the build machines would install them on every run.

## Local commands

Needs Node 22 or newer.

```bash
cd locum
npm ci
npx cap sync
```

`npx cap open android` needs Android Studio; `npx cap open ios` needs a Mac with Xcode.

## Not done yet

- **Phone push.** The locum app's shift alerts use Web Push today, which an iOS app cannot
  receive. Firebase messaging (as MiJeanie uses) needs a device-token store and a send step in
  the platform repo, plus `@capacitor-firebase/messaging` here.
- **Production address.** See "Address" above.
- **Company Apple account** for the public iOS release.
