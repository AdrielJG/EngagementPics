# Wedding Photo Booth — Setup Guide

You have two files:
- **capture.html** — the QR-code page guests use to take & send photos
- **admin.html** — your private page to view, select, download, and share the best ones

Both need a free **Firebase** project as the backend (this is what lets photos flow from guests' phones to your gallery, live). Setup takes about 15 minutes, no coding required.

## 1. Create a Firebase project
1. Go to https://console.firebase.google.com and sign in with a Google account.
2. Click **Add project**, give it a name (e.g. "our-wedding-photos"), and finish the wizard (you can skip Google Analytics).

## 2. Turn on the pieces you need
In the left sidebar of your new project:
- **Build → Authentication** → click **Get started**.
  - Enable **Anonymous** sign-in (this lets guests send photos without an account).
  - Enable **Email/Password** sign-in (this is how *you* log into the admin page).
  - Still under Authentication, go to the **Users** tab → **Add user** → enter the email and password you (the admin) will use to log in.
- **Build → Firestore Database** → **Create database** → start in **production mode** → pick a location close to you.

That's it for Firebase — we're using **Cloudinary** (a separate free service) to actually store the photo files, so you never need to touch Firebase Storage or upgrade Firebase's billing plan. Firestore only stores the small text record for each photo (its Cloudinary link and timestamp), which stays comfortably on Firebase's free "Spark" plan.

## 3. Set up Cloudinary (free, no card required) for storing the photos
1. Go to https://cloudinary.com and sign up for a free account.
2. On your Cloudinary dashboard, note your **Cloud name** (shown near the top).
3. Go to **Settings** (gear icon) → **Upload** tab → scroll to **Upload presets** → click **Add upload preset**.
   - Set **Signing Mode** to **Unsigned** (this is what lets guests upload straight from their phone with no login).
   - Give it a name you'll remember, e.g. `wedding_guest_upload`.
   - Save.
4. You now have two values to use in step 5: your **Cloud name** and this **upload preset name**.

## 4. Set Firestore security rules
Go to Firestore Database → **Rules** tab, delete what's there, and paste this, replacing `YOUR_ADMIN_EMAIL`:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /photos/{photoId} {
      allow create: if request.auth != null;
      allow read, update, delete: if request.auth != null
        && request.auth.token.email == 'YOUR_ADMIN_EMAIL';
    }
  }
}
```

Click **Publish**. This means anyone (guests, anonymously signed in) can add a new photo record, but only you can read, edit, or delete them.

## 5. Get your config keys and paste them into the files
1. In Firebase, click the gear icon → **Project settings**.
2. Under "Your apps", click the **</>** (web) icon, register an app (any nickname), skip hosting setup if asked.
3. Firebase shows a `firebaseConfig` object with your `apiKey`, `authDomain`, etc.
4. Open **both** `capture.html` and  `admin.html` in a text editor, find the `firebaseConfig` block near the top, and replace the placeholder values with your real ones. It's the same config in both files.
5. In `capture.html` only, find `CLOUDINARY_CLOUD_NAME` and `CLOUDINARY_UPLOAD_PRESET` near the top and fill in the values from step 3.

## 6. Put the site online
Easiest free option — **Firebase Hosting**:
1. Install Node.js if you don't have it, then in a terminal:
   ```
   npm install -g firebase-tools
   firebase login
   ```
2. In a folder containing `capture.html` and `admin.html`:
   ```
   firebase init hosting
   ```
   - Choose your project.
   - Public directory: `.` (current folder)
   - Single-page app: **No**
   - Don't overwrite existing files if asked.
3. Rename `capture.html` to `index.html` (so it's the default page guests land on), or keep both names and just link to `capture.html` directly in your QR code.
4. Deploy:
   ```
   firebase deploy
   ```
5. Firebase gives you a live URL like `https://our-wedding-photos.web.app`. That's your guest link — put `capture.html` (or the root URL if you renamed it) into a QR code generator (e.g. qr-code-generator.com).
6. Your admin page is at `https://our-wedding-photos.web.app/admin.html` — keep this link private, just for yourself.

*(Any static host works too — Netlify, Vercel, GitHub Pages — just upload both files.)*

## 7. Test before the big day
- Open the guest link on a phone, allow camera access, take a photo, tap the heart to send it.
- Open the admin link, log in with the email/password you created, confirm the photo shows up.
- Try selecting a photo and both the Download and WhatsApp share buttons.

## Notes
- WhatsApp sharing uses your phone's native share sheet (via the browser's Web Share API) — works well on modern Android and iOS. If it's not available, the button falls back to downloading the photos so you can attach them manually in WhatsApp.
- Guests never see each other's photos or the gallery — only you can, after logging in.
- Firebase's free "Spark" plan (Auth + Firestore only, no billing needed) and Cloudinary's free tier (25GB storage, no card required) both comfortably cover a wedding's worth of photos and traffic.
