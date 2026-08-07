# Setting up shared App Reviews (Firebase + GitHub + Vercel)

The app now has two kinds of reviews:

- **My Meal Ratings** — private, stored in each visitor's own browser (`localStorage`). No setup needed, works already.
- **App Reviews** — public, stored in Firebase Firestore so every visitor sees the same list. Needs the setup below.

## 1. Create a Firebase project

1. Go to https://console.firebase.google.com and click **Add project**.
2. Name it anything (e.g. `food-finder-ai`) and finish the wizard.
3. In the left sidebar, go to **Build > Firestore Database > Create database**.
   - Choose **Start in production mode**.
   - Pick a region close to your users.

## 2. Set Firestore security rules

Still in Firestore, go to the **Rules** tab and replace the contents with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /appReviews/{reviewId} {
      allow read: if true;
      allow create: if request.resource.data.stars is int
                    && request.resource.data.stars >= 1
                    && request.resource.data.stars <= 5
                    && request.resource.data.comment is string
                    && request.resource.data.comment.size() < 500
                    && request.resource.data.name is string
                    && request.resource.data.name.size() < 50;
      allow update, delete: if false;
    }
  }
}
```

This lets anyone read reviews and post a new one, but nobody can edit or delete someone else's review, and it caps field sizes to block spam/abuse. Click **Publish**.

## 3. Get your web app config

1. In Firebase, go to **Project settings** (gear icon) > **General**.
2. Under "Your apps," click the **</>** (web) icon to register a new web app. Any nickname is fine — you don't need Firebase Hosting.
3. Firebase will show a `firebaseConfig` object. Copy it.
4. In `food-finder-ai.html`, find this block near the top of the `<script>` tag and paste your values in:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

Note: this key is safe to expose in client-side code — Firestore security is enforced by the rules from step 2, not by hiding the key.

## 4. Push to GitHub

```bash
git init
git add .
git commit -m "Food Finder AI with shared app reviews"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

## 5. Deploy on Vercel

1. Go to https://vercel.com and sign in with GitHub.
2. Click **Add New > Project**, then import your repo.
3. It's a static HTML file, so no framework preset or build command is needed — Vercel will serve it as-is. Click **Deploy**.
4. Once deployed, visit your Vercel URL — post a review, then open the same URL in a different browser/device to confirm it shows up there too.

## Testing locally before deploying

You can't just double-click the HTML file for the Firebase part to work reliably (some browsers block Firestore requests from `file://` URLs). Instead, run a quick local server from the project folder:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000/food-finder-ai.html`.
