# Lintel Space · Supply Chain Dashboard

Procure-to-pay dashboard for Lintel Space Atelier — vendor database, KYC, purchase orders, advances, invoices, payments, project-wise P&L, site expenses & client receipts.

Backed by **Firebase** (Firestore for data, Firebase Storage for KYC files, Firebase Auth with Google sign-in restricted to `@lintelspace.com`).

Hosted on **GitHub Pages** as a single `index.html`.

---

## Setup (one-time, ~10 minutes)

### 1. Create a Firebase project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add project** → name it `lintelspace-scm` (or anything you like)
3. Disable Google Analytics if prompted (not needed) → **Create project**

### 2. Enable Authentication

1. In the Firebase console, go to **Build → Authentication → Get started**
2. Click **Sign-in method** tab → enable **Google**
3. Set support email to your @lintelspace.com admin email → **Save**

### 3. Enable Firestore Database

1. Go to **Build → Firestore Database → Create database**
2. Choose **Start in production mode** → pick the **asia-south1 (Mumbai)** region → **Create**
3. Go to the **Rules** tab and replace the default rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /lintelspace/{document=**} {
      allow read, write: if request.auth != null
                         && request.auth.token.email.matches('.*@lintelspace\\.com$');
    }
  }
}
```

4. Click **Publish**

### 4. Enable Firebase Storage

1. Go to **Build → Storage → Get started**
2. Choose **Start in production mode** → same region (asia-south1) → **Done**
3. Go to the **Rules** tab and replace with:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /kyc/{allPaths=**} {
      allow read, write: if request.auth != null
                         && request.auth.token.email.matches('.*@lintelspace\\.com$');
    }
  }
}
```

4. Click **Publish**

### 5. Register a Web App & get config

1. Go to **Project Settings** (gear icon) → scroll down to **Your apps**
2. Click the **Web** icon (`</>`) → name it `Supply Chain Dashboard`
3. **Do NOT** check "Firebase Hosting" — we're using GitHub Pages
4. Click **Register app** → copy the `firebaseConfig` object shown
5. Open `index.html` and replace the placeholder config at the top of the `<script>` section:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",            // ← paste your real values
  authDomain: "lintelspace-scm.firebaseapp.com",
  projectId: "lintelspace-scm",
  storageBucket: "lintelspace-scm.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef..."
};
```

### 6. Authorize your GitHub Pages domain

1. In Firebase Console → **Authentication → Settings → Authorized domains**
2. Add your GitHub Pages domain: `your-org.github.io`
   (or `lintelspace.github.io` if that's your GitHub org)

### 7. Deploy to GitHub Pages

1. Create a new GitHub repository (e.g. `supply-chain`)
2. Push `index.html` and `README.md` to the `main` branch
3. Go to **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main`, folder: `/ (root)` → **Save**
4. After a minute, your dashboard is live at `https://your-org.github.io/supply-chain/`

---

## How it works

- **Login**: Only `@lintelspace.com` Google accounts can sign in. Others are rejected.
- **Shared data**: All data lives in a single Firestore document. Every team member sees the same vendors, POs, invoices, payments etc.
- **KYC files**: Uploaded to Firebase Storage under `kyc/{vendorId}/`, up to 5 MB per file.
- **Offline backup**: The **Export** button downloads a 10-sheet Excel workbook covering all registers.
- **Refresh**: Press **Refresh** to pull the latest data if a teammate just made changes.

## Security

- Firestore and Storage rules restrict all reads/writes to authenticated `@lintelspace.com` users only.
- No data is publicly accessible even though the HTML is on GitHub Pages.
- The Firebase config in `index.html` is safe to commit — it only identifies the project; the security rules enforce access control server-side.

---

## File structure

```
supply-chain/
├── index.html    ← the entire dashboard (single file)
└── README.md     ← this file
```

That's it. No build step, no npm, no server. Just a static HTML file + Firebase cloud backend.
