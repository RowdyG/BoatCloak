# BoatCloak — Dive Boat Tracker v1.4.3

BoatCloak is an installable, offline-first app for dive boat trips. Crew keep a diver roster and a simple show-up check-in (not checked in / showed up / optional no-show), a paid vs unpaid check, and a Return Divers database for repeat customers, including a **Trusted** tag for experienced divers. Divers can open a public sign-up link — or a per-trip poster QR — and request a spot. Those requests arrive in the crew **Requests** inbox, where you can approve, waitlist, or decline them.

Optional shared sync uses a Google Sheet and Apps Script so several crew phones share the same data. The app works first on the device and queues changes in an outbox until they can be sent. Sun mode is a high-contrast display for direct sunlight. You can export and import the Return Divers list as CSV or JSON, and export a full backup. It is a Progressive Web App, with no app-store listing.

Works on Android and iPhone/iPad once installed to the home screen. Built for use at sea, including with a weak or missing signal.

This release is a rename of the app formerly called BoatCheck. Existing data is carried over automatically (see “Updating from BoatCheck”).

v1.4.3 fixes public sign-up when Google Sheets has turned trip dates and times into Date values (the live list was showing raw ISO strings such as `2026-11-14T15:00:00.000Z` and `1899-12-29T23:00:00.000Z`). Dates now display as `yyyy-MM-dd` and times as `HH:mm` in Japan Standard Time, on both the diver page and crew phones. The app also stops duplicate sign-up requests and roster/Return Divers records for the same person, and it auto-formats Japan and US phone numbers as you type.

## What’s in this folder

- `index.html` — the full app (styles, logic, and the poster QR generator are already inside this file)
- `manifest.json` and `sw.js` — make it installable and usable offline
- `icons/` — home-screen icons, cropped from the BoatCloak circular emblem (no wordmark)
- `backend/` — optional Google Sheet sync: `Code.gs` and `SETUP.md`
- `sync-config.example.json` — template for public self-registration only (never put the crew key in this file)
- `LICENSE-THIRD-PARTY.md` — license for the bundled poster QR generator (qrcode-generator 1.4.4)

You can email this whole folder (or the release zip) as an attachment. Opening `index.html` from a file or from email is enough to try the roster and payments. Add to Home Screen, offline cache, and working poster links need the app to be hosted over **https** (see below).

## Install on Android

1. Host this folder at an https address (see “Hosting”).
2. Open that address in **Chrome**.
3. Chrome menu (⋮) → **Add to Home screen** (sometimes “Install app”).
4. Open BoatCloak from the new home-screen icon. It runs full-screen and works offline after that first visit.

## Install on iPhone or iPad

1. Host this folder at an https address.
2. Open that address in **Safari** (not Chrome — iOS only lets Safari add web apps).
3. Tap the **Share** button, then **Add to Home Screen**.
4. Open BoatCloak from the home-screen icon. It works offline after that first visit.

High-contrast (sun) mode is the sun icon in the header, or the toggle under More → Settings.

## Updating from BoatCheck

If a phone already has BoatCheck data, BoatCloak reads the old IndexedDB (`boatcheck`) and localStorage (`boatcheck-state`) keys and copies that data into the new BoatCloak keys. Nothing is deleted from the old store. Backups exported as BoatCheck JSON still import. The service worker cache name is new (`boatcloak-v1.4.3`), so an installed phone will pick up this version on the next visit over https.

## Registration posters

Print a poster and put it on the dock or the boat. Divers scan the QR with their phone’s camera (or type the link) to open the sign-up form, fill in their details, and request a spot. The crew reviews those requests on the **Requests** tab.

There are two poster types:

1. **General poster** — the **Poster** button (or More → Registration poster) makes a QR and printed link for `#register`, for example `https://rowdyg.github.io/BoatCloak/#register`.
2. **Trip poster** — each trip has its own **Poster** button. That poster shows the trip’s public details (date, time, boat or trip name, sites, and meeting note) and a QR plus link for a trip-specific address such as `https://rowdyg.github.io/BoatCloak/#register?trip=<tripId>`. The trip id is the same public id the open-trips list uses. No diver names, phones, or other personal or crew-only data appear on the poster.

Opening a trip link preselects that trip after the diver fills in their details, so they can request it directly. If the trip is closed, full, or not found, the page shows a clear message and the normal list of open trips.

These links only work after the app is hosted at a public https URL. Enter that URL on the poster screen if it is not already filled. Generating a poster from a file on disk will not give divers a working link. **Recommended hosting:** GitHub Pages (or any static https host) pointing at the publish folder (`index.html` at the site root, plus `manifest.json`, `sw.js`, `icons/`, and `.nojekyll`). A ready-to-publish copy is prepared alongside this release. **This package does not create or push a repository.**

If shared sync is set up on the hosted site (`sync-config.json` next to `index.html`, with the web app URL and the limited register key only), `#register` lists open trips and sends a sign-up request. That public file must never contain the crew key. If `sync-config.json` is missing, the page tells the diver that online sign-up is not set up.

The app no longer generates a personal QR of a diver’s details, and crew phones no longer include a camera scanner for those codes.

## Online trip sign-up (optional)

Crew can mark a trip as open for sign-up when creating or editing it: date, boat, dive sites, departure time, capacity, and optional public price or notes. Those fields are published as a limited public list — trip info and seats left only. No diver names, phones, addresses, or emergency contacts are included. Internal crew notes and the divemaster name stay off the public list.

When `#register` is hosted with `sync-config.json`, a diver fills in their details, then sees the open upcoming trips and can request a spot. They choose Fun diving or Class (with an instructor or course if known) and can add a note. Submitting sends a sign-up request with the register-only key. The confirmation screen shows a reference code. The diver can later check that request by reference code; the public reply is status and trip time only, not their personal details.

Returning divers are recognized by phone and name on the crew side only. That match is never shown on the public page.

On crew phones, a **Requests** tab (with a badge) lists pending requests per trip. **Approve** adds the diver to the roster as Unpaid and Not checked in, and creates or merges their Return Divers record. **Decline** or **Waitlist** sets that status instead. Capacity is enforced; approving a full trip asks for confirmation before overbooking. Status syncs to every crew phone that has Shared sync turned on.

Without a backend, there is no online request. The public page tells the diver that online sign-up is not set up.

## Using it on the boat

1. Create a trip: date, time, boat or trip name, sites, divemaster, price, meeting note, capacity, and whether it is open for online sign-up. You can edit any trip later from the **Trips** tab (each card has an **Edit** button), or by tapping the trip name or the pencil on the banner at the top of the roster. Save or Cancel in the editor. Delete asks for confirmation and keeps the trip id so existing poster links still work; the roster and requests stay linked. If you lower capacity below the number of divers already approved, the app warns you and asks you to confirm — it does not automatically decline anyone.
2. Add divers by hand, from the Divers tab, or with **Returning**. A name and address are required. Each diver also has a phone number and emergency contact (name + phone). Phone numbers are tap-to-call. Every add is saved to the Return Divers database. Crew can mark a diver as **Trusted** and add a short note (for example, “Deep cert, 200+ dives”). That tag is crew-only and never appears on the public page or posters.
3. To check in a repeat customer, tap **Returning** on the Roster or Check-in tab. Search by name or phone (type-ahead, including close spellings). Pick a diver to review their saved phone, address, emergency contact, and last activity (fun diving or class with instructor and course), then add them. Payment starts as **Unpaid**. You can add and mark **Showed up** in the same step. If they are already on this trip, the app will not add a second copy. Turn on **Select several** to add more than one returning diver at once.
4. On each trip, mark a diver as **Fun diving** or **Class**. For a class, pick (or add) the instructor and optionally the course (Open Water, Advanced, Rescue, or free text). The roster groups by fun dive vs class/instructor, and the banner shows a count such as `Fun 8 / Class: Smith 4, Lee 2`. Divers can also pick fun dive or class (and an instructor) on the self-registration form.
5. On the **Roster** or **Check-in** tab, tap the large **Showed up** button when a diver arrives at the boat. The check-in time is stored and shown in Japan Standard Time (JST, Asia/Tokyo). You can optionally mark **No-show**. Tap the same button again to undo.
6. The live banner shows a count such as `9 of 14 showed up, 2 no-shows`, plus the fun-dive / class split.
7. Filter the list to **Not checked in** to see who has not arrived yet, or filter by Fun diving / Class / instructor. The Check-in tab also keeps a check-in log.
8. On the **Pay** tab, set Paid / Unpaid / Partial, amount, method (cash / card / other), and a note. Filter the list and read collected vs outstanding totals for the trip.
9. Open the **Requests** tab to approve, waitlist, or decline sign-up requests from the public link. Approved divers land on the roster unpaid and not checked in.

## Return Divers database

Every diver added by the crew is saved (or updated) in a persistent database on the device, matched by name and phone (or cert number) so records are not duplicated. Each record keeps contact info, address, emergency contact, certification, notes, and an optional Trusted tag, plus visit history derived from your trips: trips attended, number of visits (times they showed up), first and last visit, no-shows, payment history with any outstanding balance, and whether each trip was fun diving or a class.

The **Divers** tab is searchable by name or phone and can be sorted by name, last visit, or visit count. Tap **Add to trip** to put a saved diver on the current roster in one step. Repeat divers show a **Return** badge on the trip roster and in the database. Open History to see their trips and payments. Edit a record at any time. If two records are the same person, you can merge them; trip history is combined. Deleting a record asks for confirmation and does not remove them from existing trip rosters.

Export or import just this database as JSON or CSV under More → Return Divers database. Import merges by name and phone. The full JSON backup also includes the database and every trip.

## Shared sync (optional)

Several crew phones can share the same roster and Return Divers database through a Google Sheet and an Apps Script web app. This is optional. With no backend configured, BoatCloak stays local-only, as before.

Setup is in `backend/SETUP.md`: create the Sheet, paste `backend/Code.gs`, set a `CREW_KEY` and a `REGISTER_KEY`, deploy the web app (Execute as: Me, Who has access: Anyone), and host `sync-config.json` (web app URL + register key only — never the crew key) next to `index.html`. On a crew phone, **More → Shared sync** fills the web app URL from that site config automatically if the field is empty (it will not overwrite a URL you already entered). Crew only type the `CREW_KEY` and a crew name. Once the crew key is entered, the phone pings the script and starts syncing. If you already deployed an earlier version, paste the new `Code.gs` and deploy a **new version** of the same web app so Signups and OpenTrips tabs, and the public sign-up endpoints, go live. Optional crew email notices are off until you set `CREW_NOTIFY_EMAIL` yourself (see SETUP.md).

The app stays offline-first. The local copy is the source while the phone has no signal. Changes go into a queue and are sent when the app is opened, after every change, every 30 seconds while the app is visible, or when you tap **Sync now**. A status chip in the header shows Local, syncing, waiting, or last synced. If two phones edit the same record, the later edit wins. Deleted records are kept as tombstones so other phones can drop them.

Phones talk to the script with a `text/plain` POST (the body is still JSON) so the browser does not send a CORS preflight. See SETUP.md for details.

The Sheet holds divers’ personal information (phone, address, emergency contact). Keep it in the shop owner’s Google account. Do not share the Sheet publicly, and do not post the crew key or the web app URL in a public place. This package cannot deploy the script to a Google account; that step is done once in the owner’s account, following SETUP.md.

## Backup and sharing data

- **JSON backup** (More → Export JSON backup) is a full copy of trips, the Return Divers database, and settings. Import replaces the data on that device. Backups from earlier BoatCheck versions still import; unused older fields are ignored, and older attendance values become **Showed up** (the first recorded entry time, if any, is kept as the check-in time). Export files are named `boatcloak-backup-…`, `boatcloak-divers-…`, and so on.
- **Return Divers JSON/CSV** (More) exports just the saved divers, including visit and payment history. Import merges into the database already on the device.
- **CSV per trip:** roster/manifest (including activity, instructor, course, attendance, and check-in time) and payments, also under More.
- Data lives in IndexedDB, with a localStorage fallback, **on that one device**, and in the optional shop-owned Sheet if Shared sync is turned on.

## Limitations

- Without Shared sync, there is no live sync between devices and no online trip sign-up; use a JSON backup (or turn on Shared sync) to share a roster. Until `sync-config.json` and a backend are in place, the public `#register` page tells the diver that online sign-up is not set up.
- Shared sync is last-write-wins per record. The script is not deployed by this package; the shop owner must set it up.
- Camera, install, and offline cache need https (or localhost / an already-installed PWA).
- Registration and trip poster links only work once the app is hosted at a public https URL.
- The public `#register` page cannot request a trip until `sync-config.json` and a backend are in place. Crew phones auto-fill the web app URL from that file; they still must enter the crew key. Never put the crew key in a public file, and never use the register key as the crew key.
- This is a roster and attendance tool, not a replacement for a physical roll call or a dive computer.
- EXAMPLE data (More → Load EXAMPLE trip) is fictional and clearly labeled. It is not loaded by default. Do not treat it as a real roster.

## Privacy

The app does not include analytics. With no backend configured, nothing is uploaded. If you host the files on GitHub Pages or another host, that host will see ordinary page requests when someone first loads the app. If Shared sync is turned on, trip and diver records — including phone, address, and emergency contact — are stored in the shop owner’s Google Sheet. Keep that Sheet in the owner’s account and do not share it publicly. The public open-trip list and the status-by-reference reply do not include diver personal information. Optional request-notification email is off by default.

## Licenses

Application code is provided for John’s use with his scuba operation. Bundled third-party libraries are listed in `LICENSE-THIRD-PARTY.md`.
