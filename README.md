# Big Fish Sales Ledger — standalone edition

This is a self-contained version of the ledger that runs on any static host
(Netlify, Vercel, GitHub Pages, etc.) instead of inside Claude. It uses:

- **Firebase Authentication** (email/password) as the login system
- **Firebase Firestore** (free tier) as the shared database
- **Netlify** (free tier) to host the page itself

Everything below is free for a small business's usage levels. There is no
build step — it's one plain `index.html` file plus a rules file.

---

## How access works

- Nobody can see or use the ledger without signing in.
- The **first person ever to create an account becomes the owner** and is
  automatically given edit access to procurement (Purchases).
- Everyone who signs up afterwards starts as **view-only**: they can see
  everything and record sales, but the "Record a purchase" form is hidden
  and the server itself rejects any attempt to write a purchase — it's not
  just hidden in the interface.
- The owner has an **Access** tab (only they can see it) to grant or revoke
  edit access for anyone else who has signed in.

This means: before you share the link with your team, sign up first
yourself so you're the owner.

---

## Part 1 — Create your Firebase project (free)

1. Go to **console.firebase.google.com** and sign in with a Google account.
2. Click **Add project**, give it a name (e.g. "big-fish-ledger"), and
   finish the wizard (you can decline Google Analytics — not needed).
3. In the left sidebar, click **Build → Authentication**. Click
   **Get started**, then under "Sign-in method" enable **Email/Password**
   and save.
4. In the left sidebar, click **Build → Firestore Database**. Click
   **Create database**, choose a location close to your users, and start
   in **production mode** (we'll paste in proper rules next, so this is
   safe).
5. In the left sidebar, click the gear icon → **Project settings**. Under
   "Your apps", click the **</>** (web) icon to register a new web app.
   Give it any nickname and click **Register app**. Firebase will show you
   a `firebaseConfig` object — you'll need this in Part 2.

## Part 2 — Connect the app to your project

1. Open `index.html` in a text editor.
2. Find this block near the top of the `<script>` section:
   ```js
   var FIREBASE_CONFIG = {
     apiKey: "PASTE_YOUR_API_KEY",
     authDomain: "PASTE_YOUR_PROJECT.firebaseapp.com",
     projectId: "PASTE_YOUR_PROJECT_ID",
     storageBucket: "PASTE_YOUR_PROJECT.appspot.com",
     messagingSenderId: "PASTE_YOUR_SENDER_ID",
     appId: "PASTE_YOUR_APP_ID"
   };
   ```
3. Replace each `PASTE_...` value with the matching value from the
   `firebaseConfig` object Firebase showed you in step 5 above, then save
   the file.

## Part 3 — Lock down the database with the provided rules

1. Back in the Firebase console, go to **Firestore Database → Rules**.
2. Open `firestore.rules` (included in this folder), select all, copy it.
3. Paste it over whatever is in the Firebase rules editor, replacing the
   default rules entirely.
4. Click **Publish**.

Skipping this step leaves your database wide open (or fully locked,
depending on the mode you picked in Part 1) — the app won't work correctly
without these specific rules in place, since they're what actually enforce
the procurement lock.

## Part 4 — Deploy to Netlify (free)

The quickest way, no account setup beyond Netlify itself, no command line:

1. Go to **app.netlify.com/drop**.
2. Sign up / log in (free — email, or GitHub/Google sign-in).
3. Drag this whole folder (or just `index.html`, if you'd rather leave the
   `.rules` and `.md` files out of the deploy — they're documentation, not
   part of the site) onto the drop zone on that page.
4. Netlify uploads it and gives you a live URL immediately, e.g.
   `https://random-name-1234.netlify.app`.
5. Optional: click **Site settings → Change site name** to pick a nicer
   subdomain, e.g. `big-fish-ledger.netlify.app`. You can also attach a
   custom domain you own under **Domain settings**, still free.

That URL is what you share with anyone who needs to use the ledger.

**Alternative (if you want updates to redeploy automatically):** push this
folder to a GitHub repository, then in Netlify choose **Add new site →
Import an existing project** and connect that repo. Every time you push a
change, Netlify redeploys automatically. Not necessary for a one-off setup.

## Part 5 — First run

1. Open your new Netlify URL.
2. Create an account (any email/password — Firebase just needs a valid-looking
   email format; it doesn't send a confirmation email with this basic
   setup). **You should be the first person to sign up**, so you become
   the owner.
3. The ledger seeds itself with the same starting data your original Excel
   sheet had (matching products, purchase history, and the one sales day
   it recorded), so you can confirm everything's wired up correctly.
4. Go to the **Access** tab to see yourself listed as Owner. When teammates
   sign up later, they'll appear here — click **Grant edit access** for
   anyone who should be able to record purchases.

---

## Notes and limits

- **Cost**: Firebase's free "Spark" plan and Netlify's free tier both have
  generous limits (tens of thousands of reads/writes a day, 100GB
  bandwidth/month on Netlify) — comfortably enough for a small business.
  If it ever needs to scale past that, both offer pay-as-you-go plans.
- **Resetting the seed data**: if you'd rather start with an empty catalog
  instead of the fish/chicken/egg example data, delete the `products`,
  `purchases`, `sales`, `daily`, and `meta` collections directly in
  Firebase console → Firestore Database → Data (click the three-dot menu
  on each collection → Delete collection), then reload the site. The app
  only reseeds when both the `products` collection is empty **and** the
  `meta/seed` doc is gone — deleting `products` alone (but leaving `meta`)
  keeps it empty on reload. To stop it seeding at all, delete the
  `SEED_PRODUCTS`, `SEED_PURCHASES`, `SEED_SALES`, and `SEED_DAILY`
  constants near the top of the script (or just leave the arrays empty).
- **Password resets**: this basic setup doesn't wire up a "forgot
  password" flow. If someone gets locked out, you (with access to the
  Firebase console) can reset their password under Authentication → Users.
- **Each deployment is its own business.** If someone else wants their own
  independent copy of this ledger for their own shop, they repeat Parts 1–4
  with their own Firebase project — they should not reuse your Firebase
  config, or their data would mix with yours in the same database.
