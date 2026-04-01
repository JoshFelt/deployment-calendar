# deployment-calendar

A single-file release planning app with Google login, shared Firestore storage, and writer-managed permissions.

## What this repository contains

- `index.html`: the entire app UI and client logic.
- `firebase-config.js`: your Firebase web app config values.
- `firestore.rules`: Firestore Security Rules for reader and writer access.
- `README.md`: setup and operating notes.

## Access model

- Any user who signs in with Google gets read access to the shared calendar.
- A signed-in user is recorded in the `users` collection the first time they log in.
- Users whose `role` is `writer` can:
  - edit the shared calendar,
  - import/export/reset timeline data,
  - open the permissions panel,
  - promote or demote other signed-in users between `reader` and `writer`.
- Writers cannot change their own role in the in-app permissions table, which helps avoid accidental lockout.

## Data model

- `calendars/default`
  - shared calendar document containing `title`, `window`, `events`, and `ranges`
- `users/{uid}`
  - signed-in user profile with `email`, `displayName`, `photoURL`, `lastLoginAt`, and `role`

## Firebase setup

### 1. Firebase project

This repo is now connected to:

- Firebase project: `deployment-calendar-260325`
- Web app: `Deployment Calendar Web`

### 2. Enable Google sign-in

In Firebase Authentication:

- enable the `Google` provider
- add your deployment domain to `Authorized domains` if needed

Direct URL:

- <https://console.firebase.google.com/project/deployment-calendar-260325/authentication/providers>

### 3. Create Firestore

Create a Firestore database in production mode.

### 4. Web config

[firebase-config.js](/Users/joshuatolman/Documents/GitHub/deployment-calendar/firebase-config.js) is already populated with the Firebase web config for this project:

```js
window.DEPLOYMENT_CALENDAR_FIREBASE_CONFIG = {
  apiKey: "your-api-key",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  appId: "your-app-id",
  storageBucket: "your-project.firebasestorage.app",
  messagingSenderId: "1234567890",
};
```

These values are safe to expose in a web app. They identify your Firebase project; they are not admin secrets.

### 5. Firestore rules

[firestore.rules](/Users/joshuatolman/Documents/GitHub/deployment-calendar/firestore.rules) have already been deployed to the project.

### 6. Bootstrap your first writer account

Because new sign-ins default to `reader`, there is one manual first-time step:

1. Enable Google sign-in in the Firebase console.
2. Start the app and sign in with your Google account once.
3. In Firestore, open your `users/{uid}` document.
4. Change `role` from `reader` to `writer`.
5. Refresh the app.

After that, you can manage other users from the in-app permissions panel.

### 7. Initialize the shared calendar

The first writer who opens the app can initialize Firestore with the current local calendar state. After that, all signed-in users will see the shared timeline.

Direct URL:

- <https://console.firebase.google.com/project/deployment-calendar-260325/firestore>

## Local development

Because this is static HTML, you can run it with a tiny local web server:

```bash
python3 -m http.server 8000
```

Then open:

- <http://localhost:8000/index.html>

## Behavior notes

- The app still keeps a local browser copy of the calendar as a convenience cache.
- Firestore is the shared source of truth after sign-in.
- Readers can page through the calendar and export data, but they cannot edit timeline data.
- Writers see the event/range editors and can drag items directly on the calendar.

## Recommended next steps

- Add a small audit trail for role changes and major calendar edits.
- Add a dedicated admin list or filter if your user list grows.
- Optionally move from a single shared calendar doc to per-calendar documents if you want multiple teams or projects.
