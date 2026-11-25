# Chess Ladder

A static site (plain HTML, no build step) for running a chess ladder. Players rank the
people they want to challenge, request byes and revoke immunity. Organizers read the ladder
and past rounds from a Google Sheet, run pairings, record results and start the next round.

It runs on GitHub Pages. Shared data (challenges, byes, pairings, results) lives in Firebase
Firestore, and the Google Sheet is read with a Google API key.

## 1. Firebase (free Spark plan is enough)

1. Go to https://console.firebase.google.com and create a project.
2. Build > Firestore Database > Create database (production mode).
3. Build > Authentication > Get started > Sign-in method > enable **Google**.
4. Project settings > General > Your apps > add a **Web app**. Copy the config values
   (`apiKey`, `authDomain`, `projectId`, `appId`) into `config.js`.
5. Firestore > Rules: paste the contents of `firestore.rules` and publish.
6. Authentication > Settings > Authorized domains: add `YOUR-USERNAME.github.io`.

## 2. Make yourself the owner

Organizer tools only appear for emails listed in the `admins` collection, and the rules enforce
this on the server. Create the first one by hand:

1. Firestore > Start collection > Collection ID `admins`.
2. Document ID: your Google email in lower case, for example `you@gmail.com`.
3. Add a field `role` (string) with the value `owner`.

After that, open the site, sign in with that Google account, and add other organizers under
**Organizer tools > Organizers**. Only owners can add or remove organizers.

## 3. Google Sheets access

1. In https://console.cloud.google.com (select the same project), enable the **Google Sheets API**.
2. APIs & Services > Credentials > Create credentials > API key.
3. Restrict the key: Application restrictions > Websites > `https://YOUR-USERNAME.github.io/*`,
   and API restrictions > Google Sheets API only. Put the key in `config.js` as `sheetsApiKey`.
4. Share your ladder spreadsheet as **Anyone with the link can view**.
5. In the site, under Organizer tools > Google Sheet, paste the sheet link and the ladder tab
   name, then Save sheet. Pick any sheet here later without changing code.

Sheet layout the page expects:
- Ladder tab: columns `Ranking`, `Username`, and optionally `Immunity` (Y marks immune).
- Round tabs named `Round 1`, `Round 2`, and so on: games in columns B to G
  (white, vs, black, result, link), and the ladder at the start of the round in columns J to L.

## 4. Publish on GitHub

1. Create a repository and upload `index.html`, `config.js` (and these other files if you like).
2. Settings > Pages > Deploy from a branch > `main` / root.
3. Open `https://YOUR-USERNAME.github.io/REPO/`.

## Who can do what

| Person | Can do |
| --- | --- |
| Anyone | View the ladder |
| Signed in with Google | Rank challenges, request a bye, revoke their own immunity, and change or remove only entries they created |
| Organizer | Everything above, plus sync the ladder, run pairings, record results and links, start rounds |
| Owner | Organizer powers, plus add and remove organizers |

Keep these in mind:
- Hiding the organizer section in the page is cosmetic. The real protection is `firestore.rules`,
  which Firebase checks on every read and write. Keep those rules published.
- The Firebase config and Sheets API key in `config.js` are meant to be public. Restrict the API
  key as described above.
- Players choose their own name from a list. Sign-in proves a Google account, not that the person
  is that player. A player can only edit their own entries, but nothing stops someone from picking
  another player's name. Say so to your players, or ask me to add organizer-approved name claims.
- Anyone can read challenges, byes and pairings, because the ladder shows them.
