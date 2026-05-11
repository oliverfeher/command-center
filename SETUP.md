# Command Center — Setup Guide

10-minute setup. Do this once on your Mac, then it works on phone + computer forever, synced.

---

## Part 1 — Firebase Project (5 min)

### 1. Create the project
- Go to https://console.firebase.google.com
- Sign in with your Google account
- Click **"Add project"** → name it `command-center` → continue
- Disable Google Analytics (you don't need it) → Create project
- Wait ~30 seconds, click Continue

### 2. Add a Web App
- On the project home screen, click the **`</>`** (web) icon
- Nickname it `command-center-web` → Register app
- **You'll see a `firebaseConfig` block** — keep this tab open, you'll need it in Part 3
- Click "Continue to console"

### 3. Enable Google Sign-In
- Left sidebar → **Build → Authentication** → Get started
- Click **Google** in the Sign-in providers list → Enable → pick your email as support email → Save

### 4. Enable Firestore Database
- Left sidebar → **Build → Firestore Database** → Create database
- Pick **production mode** (we'll fix rules in next step)
- Pick a location near you (e.g. `us-central` or `us-east1`) → Enable

### 5. Set Security Rules (CRITICAL — locks data to only you)
- In Firestore → **Rules** tab → paste this exactly, replacing what's there:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

- Click **Publish**

### 6. Authorize your domain (for hosting)
- Authentication → **Settings** → **Authorized domains**
- Already there: `localhost`, `your-project.firebaseapp.com`
- You'll add your GitHub Pages domain here in Part 4

---

## Part 2 — Paste Your Config

1. Open `command-center.html` in any text editor (TextEdit, VS Code, whatever)
2. Find this block (around line 540):

```javascript
const firebaseConfig = {
  apiKey: "PASTE_YOUR_API_KEY_HERE",
  authDomain: "PASTE_YOUR_AUTH_DOMAIN_HERE",
  ...
};
```

3. Replace it with the config from Firebase (the one you saw in Part 1, Step 2)
4. If you lost that screen: Firebase Console → ⚙️ Project Settings → scroll to "Your apps" → click the web app → SDK config → copy

It should look like:
```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "command-center-abc123.firebaseapp.com",
  projectId: "command-center-abc123",
  storageBucket: "command-center-abc123.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abc..."
};
```

5. Save the file.

---

## Part 3 — Host It (Free, Public URL)

You need a URL so iPhone can load it. Two options:

### Option A: GitHub Pages (recommended — totally free, permanent URL)

1. Create a free GitHub account at https://github.com if you don't have one
2. Click **+ → New repository** → name it `command-center` → set to **Public** → Create
3. On the new repo page, click **uploading an existing file**
4. Drag `command-center.html` in → rename it to `index.html` before committing → Commit
5. Repo → **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main` → `/root` → Save
6. Wait ~1 minute. Your URL will be: `https://YOUR-USERNAME.github.io/command-center/`
7. **Back to Firebase** → Authentication → Settings → Authorized domains → Add domain → paste `YOUR-USERNAME.github.io` → Add

### Option B: Firebase Hosting (also free, same family)

In a terminal on your Mac:
```bash
npm install -g firebase-tools
firebase login
mkdir command-center && cd command-center
# put your command-center.html in this folder as index.html
firebase init hosting
# pick your project, public dir = current dir (.), single-page app = No
firebase deploy
```

URL: `https://your-project.web.app`

---

## Part 4 — Add to Your Devices

### iPhone (home screen install)
1. Open Safari → go to your URL
2. Sign in with Google
3. Tap the **Share button** → **Add to Home Screen** → name it "Command" → Add
4. Now there's an icon on your home screen that opens the app fullscreen, no browser bar

### Mac (bookmark or app)
1. Open Chrome or Safari → your URL → sign in
2. **Chrome:** ⋮ menu → Save and Share → Install Command Center... → installs as a Mac app
3. **Safari:** File → Add to Dock (Sonoma+) → behaves like a native app
4. Or just bookmark it

---

## How sync works

- You toggle a checkbox on your phone → saves to Firestore within 600ms
- Your Mac is open in the background → sees the change live, updates the UI
- No conflicts because last-write-wins on full state; you won't be editing on both devices at the exact same second

The green "● Synced" indicator top-right confirms it. Goes gold when saving, red if offline.

---

## Cost reality check

Firebase Spark (free) plan limits, for context on how absurdly under-quota you'll be:
- **50,000 reads/day** — you'll do maybe 50
- **20,000 writes/day** — you'll do maybe 100
- **1 GB storage** — your data is ~10 KB

You will never pay a dollar for this. If you somehow do, something is broken.

---

## Troubleshooting

**"Firebase not configured"** → you didn't paste the config, or you missed a field. Open the file again and check.

**Sign-in popup gets blocked** → the code falls back to redirect automatically. If still broken, check Firebase → Authentication → Settings → Authorized domains and make sure your hosting URL is listed.

**Data not syncing** → check the indicator. Red = network issue. Try refresh. If Firestore Rules weren't published correctly in Part 1 Step 5, syncing will silently fail — re-do that step.

**Want to wipe and start over** → Firebase Console → Firestore → find your user document under `users/{uid}` → delete. Refresh the app, it'll recreate.

---

## What you set up

A private, ad-free, syncing personal dashboard that costs $0/month, runs on iPhone + Mac, and that nobody else can access. The only way someone gets your data is by signing into your Google account.
