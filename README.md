# panjlingu Digital Dictionary — Setup Guide

This is a static HTML/CSS/JavaScript website using Firebase Realtime Database and Firebase Authentication. It does **not** use PHP or MySQL.

## 1. Add your Firebase web configuration
1. Open Firebase Console and select your `digital-dictionary` project.
2. Open the gear icon → Project settings → General.
3. Under **Your apps**, register a Web app if you have not already done so.
4. Copy the Firebase config object.
5. Open `js/firebase-config.js` and replace `PASTE_YOUR_API_KEY`, `PASTE_YOUR_MESSAGING_SENDER_ID`, and `PASTE_YOUR_APP_ID`. Verify the other values match your project exactly, especially `databaseURL`.

## 2. Enable Firebase services
1. Build → Realtime Database → create/confirm your database.
2. Build → Authentication → Get started → Sign-in method → enable **Email/Password**.
3. Authentication → Users → Add user. Create your administrator email and password.

## 3. Register the admin UID
1. In Authentication → Users, copy the administrator user's **UID**.
2. Realtime Database → Data, add this structure (replace `YOUR_ADMIN_UID` with the real UID):
   `admins / YOUR_ADMIN_UID = true`
3. Do not use an email address as the key. Use the Firebase Auth UID.

## 4. Use secure Realtime Database rules
The permissive rules `{".read": true, ".write": true}` are only for a short local test and are unsafe for a public website. Replace them with the following rules, then Publish:

```json
{
  "rules": {
    "words": {
      ".read": true,
      ".write": "auth != null && root.child('admins').child(auth.uid).val() === true"
    },
    "languages": {
      ".read": true,
      ".write": "auth != null && root.child('admins').child(auth.uid).val() === true"
    },
    "team": {
      ".read": true,
      ".write": "auth != null && root.child('admins').child(auth.uid).val() === true"
    },
    "feedback": {
      ".read": "auth != null && root.child('admins').child(auth.uid).val() === true",
      ".write": true
    },
    "admins": {
      ".read": "auth != null && root.child('admins').child(auth.uid).val() === true",
      ".write": false
    }
  }
}
```

The rules above allow visitors to read public dictionary content and submit feedback, while only a signed-in UID listed under `admins` can manage words, languages, and team members. Admin access is also checked by the page JavaScript, but **Realtime Database rules are the actual security boundary**.

## 5. Run the website
- Recommended: use VS Code with the Live Server extension and open `index.html` through Live Server.
- Or upload the whole `digital-dictionary` folder to static hosting such as Firebase Hosting or GitHub Pages.
- A plain `file://` open may cause browser restrictions; use a local web server.

## 6. First use
1. Open `admin/login.html` and sign in with the admin email/password you created.
2. Open **Languages** and confirm the five default languages: English, Urdu, Sindhi, Arabic, Spanish.
3. Open **Words** to add entries. Add the main word, meaning, and translations for each language.
4. Open the public **Dictionary** page to test searching.
5. Send a message from the public Feedback page and view it in the admin Feedback inbox.

## Database structure
- `admins/{uid}` → `true`
- `words/{autoId}` → `{ word, meaning, translations: { en, ur, sd, ar, es }, createdAt, updatedAt }`
- `languages/{code}` → `{ name, code }`
- `feedback/{autoId}` → `{ name, email, type, message, status, createdAt }`
- `team/{autoId}` → `{ name, role, bio, createdAt }`

## Important notes
- Firebase Web API config is intended to be present in browser code. Never put a Firebase service-account private key in this project.
- Feedback is public-write in the rules above; for a production public site, add App Check, validation/rate limiting through a trusted backend or Cloud Functions, and spam protection.
- Admin account creation is done in Firebase Console, not from a public registration page.
