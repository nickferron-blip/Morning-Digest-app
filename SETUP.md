# Morning Digest App — Setup Guide

Follow these 3 steps in order. Total time: ~10 minutes.

---

## Step 1 — Deploy to Netlify (get your URL first)

1. Go to **netlify.com** and sign up for a free account (use any email).
2. Once in the dashboard, look for the box that says **"Deploy manually"** or drag-and-drop area.
3. Drag the entire **`morning-digest-app`** folder onto that area.
4. Wait ~30 seconds — Netlify gives you a URL like `https://random-name.netlify.app`.
5. **Copy that URL** — you'll need it in Step 2.

---

## Step 2 — Register the app in Azure (5 minutes, free)

1. Go to **portal.azure.com** and sign in with your **nicolas.ferron@mely.ai** account.

2. In the top search bar, type **"App registrations"** and click it.

3. Click **"+ New registration"**.

4. Fill in:
   - **Name:** `Morning Digest App`
   - **Supported account types:** `Accounts in this organizational directory only (Mely.ai only)`
   - **Redirect URI:**
     - Platform: **Single-page application (SPA)**
     - URL: paste your Netlify URL from Step 1 (e.g. `https://random-name.netlify.app`)

5. Click **Register**.

6. You'll land on the app's Overview page. **Copy two values:**
   - **Application (client) ID** — looks like `a1b2c3d4-1234-...`
   - **Directory (tenant) ID** — looks like `abcd1234-5678-...`

7. In the left menu, click **"API permissions"**.

8. Click **"+ Add a permission"** → **Microsoft Graph** → **Delegated permissions**.

9. Search for **`Files.ReadWrite`**, check it, click **Add permissions**.

10. Click **"Grant admin consent for Mely.ai"** → Confirm yes.
    *(If you don't see this button, ask your Microsoft admin to grant consent, or just proceed — it may still work with user consent.)*

> **Note:** `Files.ReadWrite` is the only permission this app needs. The Recaps tab opens emails through an Outlook-on-the-web compose link (just a URL), so no mail permission or admin consent is required.

---

## Step 3 — Paste your IDs into the app

1. Open the file **`index.html`** in any text editor (TextEdit on Mac works).

2. Near the top of the file, find these two lines:
   ```
   const CLIENT_ID = 'YOUR_CLIENT_ID';
   const TENANT_ID = 'YOUR_TENANT_ID';
   ```

3. Replace `YOUR_CLIENT_ID` with your **Application (client) ID** from Step 2.
4. Replace `YOUR_TENANT_ID` with your **Directory (tenant) ID** from Step 2.

5. Save the file.

6. Go back to Netlify → your site → **Deploys** tab → drag the updated folder again to redeploy.

---

## Step 4 — Install on your iPhone

1. Open Safari on your iPhone and go to your Netlify URL.
2. Tap the **Share** button (box with arrow pointing up).
3. Scroll down and tap **"Add to Home Screen"**.
4. Tap **Add** — the Digest app now appears on your home screen like a native app.

---

## How it works day-to-day

- Every morning at 8 AM, Claude generates your digest and saves two files to your **OneDrive → Claude Dashboard** folder:
  - `Morning Action Digest - WEEKDAY, MONTH DATE, YEAR.html` (for reference)
  - `digest-YYYY-MM-DD.json` (what the app reads)
- Open the app on any device → sign in once → your digest loads automatically.
- Check off items as you complete them — state is saved back to OneDrive instantly.
- Unchecked items from yesterday show a **"Yesterday"** badge so nothing falls through the cracks.

### Recaps tab

- Every weekday at ~4 PM ET, the `eod-meeting-recaps` scheduled task pulls that day's external/client meetings from tl;dv, writes a recap for each, and saves `recaps-YYYY-MM-DD.json` to the same OneDrive folder.
- Open the app → **Recaps** tab → review and edit each meeting's recap → tap **Open in Outlook**. An Outlook-on-the-web compose window opens pre-addressed to the attendees with the subject and body filled in, for you to review and send. Nothing sends automatically. (There's also a **Copy** button if you'd rather paste into a new email.)
- The 4 PM run may miss meetings ending at/after 4 PM (tl;dv hasn't finished processing) — re-run the task later to include them.

---

## Troubleshooting

**"Sign-in failed"** — Double-check that the Redirect URI in Azure exactly matches your Netlify URL (no trailing slash).

**"No digest found"** — The app is looking for a `digest-YYYY-MM-DD.json` file in your `Documents/Claude Dashboard` OneDrive folder. This requires the morning digest skill to run once after the skill update.

**Blank page after sign-in** — Open browser console (F12), look for errors. Usually a wrong client/tenant ID.
