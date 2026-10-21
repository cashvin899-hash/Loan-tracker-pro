# LoanTrack Pro: Supabase + Android build

> Status: written but NOT yet run against a live Supabase project, and no APK has been built or installed.
> Follow the steps in order and test each one. Use fictional data until you have verified the access rules.

## 1. Supabase project
1. Create a project at supabase.com. Note the **Project URL** and **anon public key** (Settings > API).
2. SQL Editor: paste and run `supabase/migrations/001_core.sql`.
3. Authentication > Providers > Email: keep enabled, **turn OFF "Confirm email"** and **turn OFF public sign-ups**
   (Authentication > Sign In / Providers > "Allow new users to sign up" = off). Users are created only by the admin function.
4. Create the first admin: follow the comments in `supabase/bootstrap-admin.sql`.
5. Deploy the admin function:
   ```bash
   npm i -g supabase
   supabase login
   supabase link --project-ref YOUR_REF
   supabase functions deploy admin-users
   ```
   (`SUPABASE_SERVICE_ROLE_KEY` is injected by Supabase; never put it in the app or in .env.)

## 2. Run the web app
```bash
cp .env.example .env      # fill URL and anon key
npm install
npm run dev               # open the shown URL, sign in with the admin mobile + passcode
npm test                  # EMI engine tests (43 checks)
```

## 3. Verify security before real use
Create one Admin, one Manager, two Employees (Profile tab > User management). Then check:
- Employee A cannot see Employee B's customers, clients, follow-ups or documents.
- Manager sees everyone's; Employee cannot open Profile > User management actions.
- A deactivated user cannot read data (RLS checks `active`).
- Audit rows cannot be edited or deleted (try it from the SQL editor as an authenticated user).
- Open a document link: it should work for permitted users only (signed URL, 5 minutes).

## 4. Android APK (Capacitor)
Prerequisites: Android Studio, JDK 17.
```bash
npm run build
npx cap add android
npx cap sync android
npx cap open android       # Build > Build APK(s)  -> android/app/build/outputs/apk/debug/app-debug.apk
```
Release build: create a keystore (`keytool -genkey -v -keystore loantrack.jks -keyalg RSA -keysize 2048 -validity 10000 -alias loantrack`),
configure signing in `android/app/build.gradle`, then `cd android && ./gradlew assembleRelease` (APK) or `bundleRelease` (Play Store AAB).
Install on a phone: `adb install app-debug.apk`.
Test on a real phone: login, add a customer, upload a document, open it, export CSV/PDF (opens the share sheet).

## 5. Known limits
- Data lives in a generic `docs` table (JSONB) with RLS per collection. This is secure only as far as `001_core.sql` is correct; review it, and consider typed tables later.
- Dashboard cards use the default stage names; renaming stages in settings affects those counts.
- Document links open in the system browser/WebView via `window.open`; on Android you may want the Capacitor Browser plugin.
- Fonts and the PDF library load from CDNs, so the first launch needs internet.
- Use a strong admin passcode. Back up the database (Supabase backups) and define a retention policy before storing real customer data.

## 6. Build the APK online (no Android Studio needed)
1. Create a free GitHub account and a new **private** repository; upload all files of this project (keep the `.github` folder).
2. Repo > Settings > Secrets and variables > Actions > add two secrets: `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`.
3. Repo > Actions > "Build Android APK" > Run workflow. After a few minutes download the artifact `LoanTrackPro-debug-apk`, unzip, and install `app-debug.apk` on your phone (allow "install unknown apps").
This workflow is untested; if a step fails, open the failed step's log and send it to me.
