# Blush Studio — setup guide

This is a starter project for a premium pink/purple surprise-website creator.

## What is included
- Responsive landing page and creator dashboard
- Firebase email/password login integration
- Multiple websites per signed-in account
- Question editor with add/remove
- Live mobile preview
- Image/GIF and music upload controls
- Public share links using `#site/<id>`
- Public question flow, floating hearts, envelope intro and moving NO button

## Important
Cloud login, saving, uploads and public links require a Firebase project. The project is intentionally not shipped with a fake password or insecure client-side admin login.

## 1. Create a Firebase project
1. Open https://console.firebase.google.com/ and create a project.
2. Add a Web App and copy its Firebase config.
3. In Authentication → Sign-in method, enable **Email/Password**.
4. In Authentication → Users, add your own user/email and password.
5. Create a Firestore database.
6. Create Storage.

## 2. Add Firebase config
Open `app.js` and replace the six `PASTE_...` values in `firebaseConfig` with the values from your Firebase Web App settings.

## 3. Firestore rules
For a first private-owner setup, use these rules. They allow each signed-in user to manage their own drafts, and allow public reading only from published public documents.

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/sites/{siteId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    match /publicSites/{siteId} {
      allow read: if true;
      allow write: if request.auth != null
        && request.resource.data.ownerId == request.auth.uid;
    }
  }
}
```

## 4. Storage rules
```text
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /users/{userId}/assets/{fileName} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.uid == userId
        && request.resource.size < 15 * 1024 * 1024;
    }
  }
}
```

## 5. Run locally
Because this uses JavaScript modules, open it through a local web server (not by double-clicking the HTML). For example, in this folder run:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000`.

## 6. Publish free with GitHub Pages
1. Create a GitHub repository.
2. Upload `index.html`, `style.css`, and `app.js`.
3. In repository Settings → Pages, select the main branch and root folder.
4. Save and wait for GitHub Pages to provide your public URL.
5. Your creator dashboard will be at that URL plus `#admin`.

## Current starter limitations
- The public link is a single-page route (`#site/id`).
- The dashboard has email/password sign-in through Firebase.
- Add a custom domain later if desired.
- For production use, consider Firebase App Check, stricter Storage validation, and limiting who can create public sites.
- Background audio generally starts only after the visitor taps the envelope/button; browsers block autoplay before user interaction.
