# Job Tracker

A single-file job application tracker with a dark, mobile-friendly interface. Log applications by hand, record each interview round, keep the job description, and see charts of how your search is going. Data is stored in Firebase Firestore, with a browser-only fallback.

The whole app is one file, `index.html`. There is no build step and nothing to install.

## Features

**Applications**
- Company, job title, location, work mode (onsite, hybrid, remote), employment type
- Source (LinkedIn, Naukri, Indeed, Company website, Referral, Wellfound, Instahyre, Telegram, Recruiter, Campus, Other)
- Job posting link, status, priority, date applied, follow-up date
- Expected pay range in INR (LPA), USD, EUR or GBP (thousands per year)
- Full job description, notes, recruiter contact details, referral, resume version used
- Status values: Wishlist, Applied, Screening, Interview, Offer, Accepted, Rejected, Withdrawn, Ghosted

**Hiring process**
- Add any number of rounds to an application, each with a type, date, result and notes
- Round types include recruiter screen, online assessment, take-home, technical, system design, managerial, culture fit, HR, final round and offer discussion
- Rounds appear as a timeline on the application, and the next scheduled round shows on its card

**List and search**
- Filter by stage, search across company, role, location, notes and job description
- Sort by date applied, company, priority, follow-up date or last updated
- Change an application's status from its detail view

**Analytics**
- Summary tiles: total applications, active, response rate, interview rate, offers, average expected pay
- Needs attention list: follow-ups due within 2 days or overdue, and applications with no reply for 14 or more days
- Charts: status breakdown, hiring funnel, applications per week (last 12 weeks), source performance, interview round outcomes, work mode split, response rate by priority

**Data**
- Live sync with Firestore across devices
- JSON export and import for backups
- Optional sample data to try the app, removable from the Storage tab

## Run it

Serve the file from a local web server. For example, with the VS Code Live Server extension, open the folder and choose Go Live. Any static host also works.

Opening the file directly with `file://` can work, but the "Download file with config saved" button needs a server.

The page loads Chart.js from jsDelivr and the Firebase libraries from gstatic, so it needs an internet connection.

## Set up Firebase

1. Go to the [Firebase console](https://console.firebase.google.com) and create a project.
2. Add **Firestore Database** (production or test mode).
3. In Project settings, register a **Web app** and copy the `firebaseConfig` object.
4. Open the tracker, go to the **Storage** tab, paste the config and press **Connect**. The page reloads and the badge in the header changes to "Firebase live".
5. In Firestore, open the **Rules** tab, paste the rules below and press **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    match /jobApplications/{id} {
      allow read, write: if true;
    }
  }
}
```

If you see "Missing or insufficient permissions", the rules have not been published yet or do not match the collection name `jobApplications`.

Data is stored in the `jobApplications` collection, one document per application.

### Security note

The tracker has no sign-in, so the rules above allow anyone who has your Firebase config to read and write that collection. Keep the config and the file private, and do not publish them in a public repository.

For stricter access, add Firebase Authentication to the page and change the rule to check `request.auth`, for example `allow read, write: if request.auth.uid == "YOUR_UID";`.

## Keep the connection after clearing browser data

By default the pasted config is stored in your browser, so clearing site data removes it. To keep it permanently:

1. After connecting, open the Storage tab and press **Download file with config saved**.
2. Replace `index.html` with the downloaded file.
3. Reload. The Storage tab will say the config is saved inside this file.

You can also do it by hand. Open `index.html`, find this line near the top of the script, and replace `null` with your config object:

```js
var FIREBASE_CONFIG = null;
```

For example:

```js
var FIREBASE_CONFIG = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

## Storage modes

| Mode | When | Where data lives |
| --- | --- | --- |
| Firebase live | A valid config is present | Firestore, synced live |
| Local only | No config | This browser's local storage |

If you started in local mode and then connect Firebase, use **Copy local data to Firebase** in the Storage tab to upload what you already entered.

## Backup

On the Storage tab, **Export JSON** downloads all applications. **Import JSON** adds applications from a file you exported earlier.

## Customising

All lists are near the top of the script in `index.html`:

- `SOURCES`: where you found the job
- `STATUSES` and `STATUS_COLOR`: application statuses and their colours
- `STEP_TYPES` and `STEP_RESULTS`: interview round types and results
- `MODES`, `JOBTYPES`, `PRIORITIES`, `CUR`: work mode, employment type, priority, currencies

The colour theme is defined by the CSS variables at the top of the `<style>` block.

## Troubleshooting

| Problem | Fix |
| --- | --- |
| Badge says "Firebase error" with "Missing or insufficient permissions" | Publish the Firestore rules shown above |
| Config disappeared after clearing site data | Save it into the file, see the section above |
| Charts do not show | The Chart.js library did not load. Check your connection and reload |
| "Connected" but no data shows | Confirm the config's `projectId` matches the project where you created Firestore |
| Download button says it could not build the file | Run the page from a local server, or paste the config line into the file by hand |

## Tech

Plain HTML, CSS and JavaScript. Chart.js 4.4.1 for charts and Firebase 10.12.2 (Firestore, compat build) for storage. Fonts: Bricolage Grotesque, DM Sans and IBM Plex Mono from Google Fonts.