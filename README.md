# Anis-2.0

A responsive life management web app (React + Firebase) for notes, tasks, reminders, folders, shopping tracking, and nutrition tracking.

## Firebase setup guide (complete checklist)

If you are new to Firebase, follow these steps in order.

---

## 1) Create your Firebase project

1. Go to https://console.firebase.google.com
2. Click **Create project** → name it (for example `anis-prod`)
3. (Optional) Enable Google Analytics
4. Create a second project for development (for example `anis-dev`)

Recommended environments:
- `anis-dev` (development)
- `anis-staging` (optional)
- `anis-prod` (production)

---

## 2) Register your web app

Inside the Firebase project:
1. Click **</> Web** to add a web app
2. App nickname: `anis-web`
3. Copy the Firebase config values (apiKey, authDomain, projectId, etc.)

Use these values in environment variables (never hardcode secrets):

```env
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
```

---

## 3) Install Firebase in your React app

```bash
npm install firebase
```

Create `src/firebase.ts`:

```ts
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getFirestore } from 'firebase/firestore';
import { getStorage } from 'firebase/storage';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

export const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
export const storage = getStorage(app);
```

---

## 4) Enable Authentication

In Firebase Console → **Authentication**:
1. Click **Get started**
2. Enable providers:
   - Email/Password (MVP)
   - Google (optional)
3. Add your app domain(s) to authorized domains

MVP recommendation:
- Start with Email/Password + password reset
- Add Google login later if needed

---

## 5) Create Firestore database

In Firebase Console → **Firestore Database**:
1. Click **Create database**
2. Start in production mode
3. Choose closest region to users

Suggested top-level collections for Anis:
- `users`
- `notes`
- `folders`
- `tasks`
- `reminders`
- `shoppingItems`
- `nutritionEntries`
- `goals`
- `habits`

Document shape pattern:
- Every user-owned document should include `userId`
- Include timestamps: `createdAt`, `updatedAt`

---

## 6) Firestore security rules (critical)

Use owner-based rules so users only read/write their own data:

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isSignedIn() {
      return request.auth != null;
    }

    function isOwner() {
      return isSignedIn() && request.auth.uid == request.resource.data.userId;
    }

    function isExistingOwner() {
      return isSignedIn() && request.auth.uid == resource.data.userId;
    }

    match /{collection}/{docId} {
      allow create: if isOwner();
      allow read, update, delete: if isExistingOwner();
    }
  }
}
```

Notes:
- Validate required fields per collection as you mature the schema
- Never ship open rules like `allow read, write: if true;`

---

## 7) Firestore indexes

When querying (for example by `userId`, `dueDate`, `priority`), Firebase may ask for indexes.
Create them in **Firestore → Indexes**.

Likely needed composite indexes:
- `tasks`: `userId + dueDate`
- `tasks`: `userId + status + priority`
- `notes`: `userId + updatedAt`
- `reminders`: `userId + remindAt`
- `shoppingItems`: `userId + status`
- `nutritionEntries`: `userId + date`

---

## 8) Cloud Storage setup

In Firebase Console → **Storage**:
1. Click **Get started**
2. Choose region
3. Add owner-only security rules

Example storage rules:

```txt
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /users/{userId}/{allPaths=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

Use this for note attachments, profile images, or future media.

---

## 9) Local development with Firebase Emulator Suite

Install Firebase CLI:

```bash
npm install -g firebase-tools
firebase login
firebase init
```

During `firebase init`, select:
- Firestore
- Authentication emulator
- Storage
- Emulators

Then run:

```bash
firebase emulators:start
```

Why this matters:
- Safe local testing
- No accidental production data writes
- Faster development

---

## 10) Deploy hosting (when ready)

If you want Firebase Hosting:

```bash
firebase init hosting
npm run build
firebase deploy
```

You can also keep frontend hosting elsewhere (Vercel/Netlify) and still use Firebase backend.

---

## 11) Data model recommendation for MVP

Use one of these patterns:

1. **Top-level collections + `userId` field** (simple, scalable for your use case)
2. Per-user subcollections (more nested structure)

For Anis MVP, top-level + `userId` is usually easiest for global search and filtering.

---

## 12) Push notifications (future)

When you add reminders notifications:
- Use Firebase Cloud Messaging (FCM)
- Add service worker for web push
- Store notification token per user
- Trigger notifications from Cloud Functions or scheduled jobs

---

## 13) Production readiness checklist

Before launch:
- [ ] Separate dev/prod Firebase projects
- [ ] Tight Firestore & Storage rules
- [ ] App Check enabled (recommended)
- [ ] Rate limits/abuse controls considered
- [ ] Backups/export strategy for Firestore
- [ ] Error monitoring and analytics
- [ ] Privacy policy and data deletion flow

---

## 14) Minimal order to start quickly

If you want the shortest path to a working MVP:
1. Create project + web app
2. Enable Email/Password auth
3. Create Firestore
4. Add strict owner-only rules
5. Connect React app with `firebase` SDK
6. Build Notes + Tasks first
7. Add Reminders + Shopping + Nutrition
8. Add Storage for attachments
9. Add emulators and improve indexes

---

If you want, the next step can be: generating a concrete Firestore schema (field-by-field) for each Anis feature (notes, tasks, reminders, shopping, nutrition, folders, planning).
