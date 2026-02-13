# Flavor Lab — Technical Documentation

## Overview
Flavor Lab is a Flutter application targeting Android (with Web preview support). It uses Firebase for authentication, database storage, and user uploads. The architecture is intentionally simple to enable rapid iteration and to keep the MVP maintainable.

## Tech Stack
- **Frontend**: Flutter (Dart)
- **Backend**: Firebase
  - Authentication (Anonymous login)
  - Firestore (recipes, favorites, feedback timeline)
  - Cloud Storage (user photos)
- **Design Direction**: Duolingo-inspired bright, playful UI

## Runtime Architecture
The app initializes Firebase on startup. If Firebase is not configured, it falls back to demo data.

- `AppBootstrap` handles initialization and anonymous sign-in
- `AppScope` provides a shared `AppData` implementation to the widget tree
- `FirestoreData` is the real data layer
- `MockData` provides demo recipes when Firebase is unavailable

## Data Model (Firestore)

### `recipes` (public read, admin write)
Each recipe document contains:
- `title` (string)
- `description` (string)
- `heroImageUrl` (string)
- `videoUrl` (string)
- `externalLinkUrl` (string)
- `prepMinutes` (number)
- `cookMinutes` (number)
- `difficulty` (string)
- `ingredients` (array of `{name, amount}`)
- `tools` (array of strings)
- `steps` (array of `{title, body, timerMinutes?}`)
- `updatedAt` (timestamp)

### `users/{uid}`
- `createdAt` (timestamp)
- `lastActiveAt` (timestamp)
- `tier` (string, default `free`)
- `entitlements` (array of strings)

### `users/{uid}/favorites`
- doc ID = `recipeId`
- `createdAt` (timestamp)

### `cooks`
User feedback timeline:
- `recipeId` (string)
- `recipeTitle` (string)
- `heroImageUrl` (string)
- `userId` (string)
- `createdAt` (timestamp)
- `experience` (string)
- `tweaked` (bool)
- `rating` (number 1–5)
- `photoUrl` (string)

## Firebase Rules (Development)
Firestore rules (dev):
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /recipes/{id} {
      allow read: if true;
      allow write: if false;
    }

    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
      match /favorites/{recipeId} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }
    }

    match /cooks/{cookId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Storage rules (dev):
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /user_uploads/{userId}/{allPaths=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

## Build & Preview

### Web Preview
```
/opt/homebrew/bin/flutter run -d chrome
```

### Android Debug (requires SDK)
```
/opt/homebrew/bin/flutter run
```

### Android Release APK
Requires `android/app/google-services.json`:
```
/opt/homebrew/bin/flutter clean
/opt/homebrew/bin/flutter pub get
/opt/homebrew/bin/flutter build apk --release
```

APK output:
```
build/app/outputs/flutter-apk/app-release.apk
```

### Automated Release Script
```
./scripts/release_apk.sh
```

This script builds the APK and (optionally) publishes a GitHub Release if the `gh` CLI is installed.

## Admin Workflow (MVP)
Admins manage recipes directly in Firebase Console by adding/updating documents in `recipes`. This enables fast content updates without rebuilding the app.

## Payment Hooks (Planned)
The user document includes `tier` and `entitlements`, which serve as the future integration point for payment verification and access gating.

## Limitations
- Inventory tracking is currently local and not persisted.
- Slot machine results are random (not inventory-aware yet).
- No admin dashboard yet (Firebase Console only).

## Next Steps
- Persist fridge items into Firestore
- Add inventory-aware recommendation logic
- Build creator analytics dashboards
- Add subscription billing and entitlement checks
