# BoatCheck — Dive Boat Tracker v1.2.0

An installable, offline app for tracking divers on a boat: a trip roster, a simple attendance check (not checked in / showed up / optional no-show), a Return Divers database for repeat customers, a per-trip activity (fun diving or class with instructor and course), a paid vs unpaid check, and QR-based self-registration. It is a Progressive Web App. There is no app-store listing and no server. All data stays on the device.

Works on Android and iPhone/iPad once installed to the home screen. Built for use at sea, including with a weak or missing signal.

## What’s in this folder

- `index.html` — the full app (styles, logic, and QR libraries are already inside this file)
- `manifest.json` and `sw.js` — make it installable and usable offline
- `icons/` — home-screen icons
- `LICENSE-THIRD-PARTY.md` — licenses for the bundled QR libraries

You can email this whole folder (or the release zip) as an attachment. Opening `index.html` from a file or from email is enough to try the roster, payments, and QR *generation*. The camera scanner, Add to Home Screen, and offline cache need the app to be hosted over **https** (see below).

## Install on Android

1. Host this folder at an https address (see “Hosting”).
2. Open that address in **Chrome**.
3. Chrome menu (⋮) → **Add to Home screen** (sometimes “Install app”).
4. Open BoatCheck from the new home-screen icon. It runs full-screen and works offline after that first visit.

## Install on iPhone or iPad

1. Host this folder at an https address.
2. Open that address in **Safari** (not Chrome — iOS only lets Safari add web apps).
3. Tap the **Share** button, then **Add to Home Screen**.
4. Open BoatCheck from the home-screen icon. The camera can be used from this installed app, and it works offline after the first visit.

High-contrast (sun) mode is the sun icon in the header, or the toggle under More → Settings.

## QR registration, offline

There are two different QR codes:

1. **Poster QR** — a link to the self-registration page (`#register`). Divers scan this with their phone’s camera to open the form. This link only works after the app is hosted at a public https URL. Enter that URL under More → Registration poster (or the Poster button). Generating a poster from a file on disk will not give divers a working link.
2. **Diver QR** — created after the diver fills in the form (name, phone, address, emergency contact, cert, notes). The QR itself holds a compact, versioned JSON payload (`BC3:`) with a checksum, including activity (fun dive or class) and an optional instructor and course. Crew phones still read older `BC2:` and `BC1:` codes; rental-gear fields in v1 codes are ignored, and older codes default to Fun diving. The crew taps **Scan QR**, and the diver is added to the current trip and marked **Showed up**. Scanning a QR for someone already on the trip also marks them **Showed up** (it does not add a second copy) and updates their saved record. This works fully offline. No server is involved. Manual adds still start as **Not checked in**. Duplicates from a non-QR add (same name + phone, or the same cert number) are flagged.

Crew can also scan a photo of a QR, or paste the `BC3:…`, older `BC2:…`, or `BC1:…` text, if the camera is unavailable.

### Why the camera needs https

Browsers only allow the camera (`getUserMedia`) in a [secure context](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts): **https**, `localhost`, or an installed PWA that was first opened over https. Opening `index.html` from Files, Downloads, or an email attachment is `file://`, and the camera will be blocked. iOS Safari also needs `playsinline` (already set) and works more reliably from the home-screen icon.

**Recommended hosting:** GitHub Pages (or any static https host) pointing at the publish folder (`index.html` at the site root, plus `manifest.json`, `sw.js`, `icons/`, and `.nojekyll`). A ready-to-publish copy is prepared alongside this release. **This package does not create a repository or push anything.**

Android Chrome and many desktop browsers can use the built-in `BarcodeDetector` API. iOS Safari does not, so BoatCheck falls back to a bundled copy of jsQR. Both paths run on the device.

## Using it on the boat

1. Create a trip: date, boat name, site(s), divemaster.
2. Add divers (manual, from the Divers tab, or by QR). Each diver has a phone number, address, and emergency contact (name + phone). Phone numbers are tap-to-call. Every add or scan is saved to the Return Divers database.
3. On each trip, mark a diver as **Fun diving** or **Class**. For a class, pick (or add) the instructor and optionally the course (Open Water, Advanced, Rescue, or free text). The roster groups by fun dive vs class/instructor, and the banner shows a count such as `Fun 8 / Class: Smith 4, Lee 2`. Divers can also pick fun dive or class (and an instructor) on the self-registration form.
4. On the **Roster** or **Check-in** tab, tap the large **Showed up** button when a diver arrives at the boat. The check-in time is stored and shown in Japan Standard Time (JST, Asia/Tokyo). You can optionally mark **No-show**. Tap the same button again to undo.
5. The live banner shows a count such as `9 of 14 showed up, 2 no-shows`, plus the fun-dive / class split.
6. Filter the list to **Not checked in** to see who has not arrived yet, or filter by Fun diving / Class / instructor. The Check-in tab also keeps a check-in log.
7. On the **Pay** tab, set Paid / Unpaid / Partial, amount, method (cash / card / other), and a note. Filter the list and read collected vs outstanding totals for the trip.


## Return Divers database

Every diver added by the crew or by QR scan is saved (or updated) in a persistent database on the device, matched by name and phone (or cert number) so records are not duplicated. Each record keeps contact info, address, emergency contact, certification, and notes, plus visit history derived from your trips: trips attended, number of visits (times they showed up), first and last visit, no-shows, payment history with any outstanding balance, and whether each trip was fun diving or a class.

The **Divers** tab is searchable by name or phone and can be sorted by name, last visit, or visit count. Tap **Add to trip** to put a saved diver on the current roster in one step. Repeat divers show a **Return** badge on the trip roster and in the database. Open History to see their trips and payments. Edit a record at any time. If two records are the same person, you can merge them; trip history is combined. Deleting a record asks for confirmation and does not remove them from existing trip rosters.

Export or import just this database as JSON or CSV under More → Return Divers database. Import merges by name and phone. The full JSON backup also includes the database and every trip.

## Backup and sharing data

- **JSON backup** (More → Export JSON backup) is a full copy of trips, the Return Divers database, and settings. Import replaces the data on that device. Backups from v1.0.0 and v1.1.0 still import; rental-gear fields are ignored, and old On boat / In water / Back aboard states become **Showed up** (the first recorded entry time, if any, is kept as the check-in time).
- **Return Divers JSON/CSV** (More) exports just the saved divers, including visit and payment history. Import merges into the database already on the device.
- **CSV per trip:** roster/manifest (including activity, instructor, course, attendance, and check-in time) and payments, also under More.
- Data lives in IndexedDB, with a localStorage fallback, **on that one device**. There is no live sync between crew phones. To copy a trip to another phone, export the JSON backup and import it there (or email the file).

## Limitations

- No live sync between devices.
- Camera, install, and offline cache need https (or localhost / an already-installed PWA).
- The registration poster link only works once the app is hosted.
- This is a roster and attendance tool, not a replacement for a physical roll call or a dive computer. It does not track in-water or back-aboard times.
- EXAMPLE data (More → Load EXAMPLE trip) is fictional and clearly labeled. Do not treat it as a real roster.

## Privacy

Nothing is uploaded. There are no analytics and no network calls by the app itself. If you host the files on GitHub Pages or another host, that host will see ordinary page requests when someone first loads the app.

## Licenses

Application code is provided for John’s use with his scuba operation. Bundled third-party libraries are listed in `LICENSE-THIRD-PARTY.md`.
