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

1. **Apple (App Store Connect):** create an app for `com.jeanieiq.locum.dev` and one for
   `com.jeanieiq.employee.dev`. The iOS workflows use the App Store Connect key integration
   named `JeanieIQ` (the same one the MiJeanie build uses) and look the app up by bundle id.
2. **Google Play Console:** create the two apps (`com.jeanieiq.locum`,
   `com.jeanieiq.employee`) with Play App Signing on.
3. **Codemagic, Team settings:**
   - Android keystores: an upload keystore for each app, reference names
     `jeanieiq_locum_upload` and `jeanieiq_employee_upload`. Keep a safe copy; Google holds
     the real app-signing key, these only sign uploads.
   - Environment variable group `jeanieiq_google_play` with
     `GCLOUD_SERVICE_ACCOUNT_CREDENTIALS` = the base64 of a Google Cloud service account's
     JSON key, invited in Play Console with release access to both apps (testing tracks only).
4. **First Android upload by hand:** Google accepts API uploads only after the first bundle of
   a new app has been uploaded in Play Console. Download the `.aab` from the first Codemagic
   build and upload it once.

None of the workflows has run yet. The first run of each proves the config, including the
App Store Connect lookup by bundle id.

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
