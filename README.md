# Chess Ladder

A static site (plain HTML, no build step) for running a chess ladder. Players rank the
people they want to challenge, request byes and revoke immunity. Moderators read the ladder
and past rounds from a Google Sheet, run pairings, record results and start the next round.

It runs on GitHub Pages. Shared data (challenges, byes, pairings, results) lives in Firebase
Firestore, and the Google Sheet is read with a Google API key.

## 1. Firebase (free Spark plan is enough)

1. Go to https://console.firebase.google.com and create a project.
2. Build > Firestore Database > Create database (production mode).
3. Build > Authentication > Get started > Sign-in method > enable **Google** (for moderators) and
   **Anonymous** (for players, who never see a login screen).
4. Project settings > General > Your apps > add a **Web app**. Copy the config values
   (`apiKey`, `authDomain`, `projectId`, `appId`) into `config.js`.
5. Firestore > Rules: paste the contents of `firestore.rules` and publish.
6. Authentication > Settings > Authorized domains: add `YOUR-USERNAME.github.io`.

## 2. Make yourself the owner

Moderator tools only appear for emails listed in the `admins` collection, and the rules enforce
this on the server. Create the first one by hand:

1. Firestore > Start collection > Collection ID `admins`.
2. Document ID: your Google email in lower case, for example `you@gmail.com`.
3. Add a field `role` (string) with the value `owner`.

After that, open the site, sign in with that Google account, and add other moderators under
**Moderator tools > Moderators**. Only owners can add or remove moderators.

## 3. Google Sheets access

1. In https://console.cloud.google.com (select the same project), enable the **Google Sheets API**.
2. APIs & Services > Credentials > Create credentials > API key.
3. Restrict the key: Application restrictions > Websites > `https://YOUR-USERNAME.github.io/*`,
   and API restrictions > Google Sheets API only. Put the key in `config.js` as `sheetsApiKey`.
4. Share your ladder spreadsheet as **Anyone with the link can view**.
5. In the site, under Moderator tools > Google Sheet, paste the sheet link and the ladder tab
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
| Any visitor, no login | View the ladder, rank challenges, request a bye, revoke or restore their own immunity |
| Moderator (Google sign-in, listed in `admins`) | Everything above, plus sync the ladder, run pairings, record results and links, start rounds |
| Owner | Moderator powers, plus add and remove moderators |

Players are signed in anonymously by the page, so they just open the link. Moderators use the
small "Moderator sign-in" link at the top.

Keep these in mind:
- Hiding the moderator section in the page is cosmetic. The real protection is `firestore.rules`,
  which Firebase checks on every read and write. Keep those rules published.
- The Firebase config and Sheets API key in `config.js` are meant to be public. Restrict the API
  key as described above.
- With no login, nothing verifies who a player is. Anyone can pick another player's name and change
  that player's choices, byes or immunity. The rules only check the shape of the data. Moderators
  can see everything and "Start next round" clears all challenges and byes. If this becomes a
  problem, add moderator-approved name claims, or require sign-in again.
- Anonymous sign-in is tied to a browser. If a player clears site data or switches device, they pick
  their name again and can still edit their entries.
- Anyone can read challenges, byes and pairings, because the ladder shows them.

## Updating an existing site

Replace `index.html` and republish `firestore.rules` (Firestore > Rules). The rules now include an `inactivity` collection (strikes and removals) and a `cards` collection (yellow and red cards). `config.js` is unchanged, so keep your own.

## Lichess login and ratings

Players log in with Lichess (OAuth, no secret and no app registration needed) so the page knows which ladder player they are. The page asks for no permissions, only learns the username, and throws the token away. Moderators keep a dropdown to act as any player. Ratings shown next to names are Classical ratings from the public Lichess API, cached in the browser for six hours. A `?` means provisional or none.

Ladder names must match Lichess usernames. The page remembers the login in the browser, so players log in once per device. This identifies players in the page, but the Firestore rules are still trust-based: someone who bypasses the page could still write entries. Locking writes to verified players would need a small server (for example a Cloud Function) that checks the Lichess login and issues Firebase sign-in tokens.

Usernames are checked against Lichess whenever a moderator adds a player, loads a ladder from a pasted list or file, or syncs it from the sheet. If any name isn't a valid, open Lichess account, nothing is imported and the message lists which names failed. Names are saved with Lichess's own capitalisation.
