# Movie Tracker

A React Native (Expo) app for browsing movies and TV shows and tracking what
you've watched, backed by a small PHP/MySQL API.

## Why not IMDb directly?

IMDb doesn't publish a public API and scraping the site violates their
Terms of Service. Instead this app uses **TMDb (The Movie Database)** —
free, well-documented, and it covers everything you asked for: full cast
and crew, every season and episode of a show, ratings, images, release
dates, recommendations, etc. IMDb ratings/IDs are also cross-referenced
inside TMDb's data if you ever want them.

## Project layout

```
movie-tracker/
  backend/          PHP + MySQL API (accounts + watched-tracking)
  mobile-app/        Expo/React Native app
```

## 1. Backend setup

Deploy `backend/` the same way as your other atwebpages.com projects:

1. Create a MySQL database and import `backend/schema.sql`.
2. Copy `backend/config.example.php` to `backend/config.php` and fill in your
   real DB credentials and ElevenLabs API key. **`config.php` is gitignored
   — never commit it.**
3. Upload the `backend/` folder to your hosting, e.g. as `/api/`.
4. Test it's alive by opening `https://yourdomain.atwebpages.com/api/login.php`
   in a browser — you should get a JSON error (expected, since it's a POST
   endpoint), not a PHP error page.

Endpoints:
- `POST /register.php` — `{ email, password }` → `{ token, user }`
- `POST /login.php` — `{ email, password }` → `{ token, user }`
- `GET /watched_list.php` — (Bearer token) → `{ movies: [...], episodes: [...] }`
- `POST /watched_toggle.php` — (Bearer token) mark/unmark a movie or episode

## 2. Mobile app setup

1. Get a free TMDb API key: https://www.themoviedb.org/settings/api
2. Copy `mobile-app/src/config.example.js` to `mobile-app/src/config.js` and
   fill in `TMDB_API_KEY` and `BACKEND_URL` (your deployed backend from
   step 1). **`config.js` is gitignored — never commit it.**
3. Install dependencies and run:
   ```bash
   cd mobile-app
   npm install
   npx expo start
   ```
   Scan the QR code with Expo Go on your phone, or press `i`/`a` for a
   simulator.

## 3. What's already built

- Email/password accounts (register, login, persisted session)
- Home: trending movies & TV today / this week
- Search across movies and TV shows
- Movie detail: overview, runtime, genres, rating, full cast, mark-as-watched
- TV detail: overview, cast, full season list
- Season screen: every episode with air date, overview, per-episode
  mark-as-watched
- Watchlist tab: watched movies + shows in progress with episode counts,
  synced to your account via the PHP backend
- Profile tab: quick stats, logout

## 4. Running it in a browser / GitHub Pages

The app is built with Expo, which can compile the same React Native code to
a web bundle (`react-native-web`). Everything works in a browser except the
ElevenLabs voice assistant — its SDK is native/WebRTC-only, so the web build
automatically shows a "not available in the browser" message there instead
(see `AssistantScreen.web.js`) while the real one still runs in the mobile
app (`AssistantScreen.native.js`).

**Try it locally:**
```bash
cd mobile-app
npm install
npm run web
```

**Deploy to GitHub Pages automatically:** `.github/workflows/deploy-web.yml`
is already set up to build and publish on every push to `main`. To turn it on:

1. Push this repo to GitHub (see the git steps below).
2. In the repo, go to **Settings → Pages** and set **Source** to
   **GitHub Actions** (not "Deploy from a branch").
3. Go to **Settings → Secrets and variables → Actions** and add two
   repository secrets:
   - `TMDB_API_KEY` — your TMDb key
   - `BACKEND_URL` — your deployed PHP backend URL
   (The workflow writes these into `src/config.js` at build time, since that
   file is gitignored.)
4. In `mobile-app/app.json`, `experiments.baseUrl` is set to `/movietracker`
   — that has to match your actual repo name exactly (GitHub Pages project
   sites are served at `username.github.io/repo-name/`). Update it if your
   repo is named differently.
5. Push to `main`. Check the **Actions** tab for the run; once it's green,
   your site is live at `https://your-username.github.io/movietracker/`.

## 5. The ElevenLabs voice agent

The app is wired up for a **private** agent — your ElevenLabs API key lives
only in `backend/config.php`, and the app asks your own backend
(`backend/elevenlabs_token.php`) for a short-lived conversation token
instead. This is the pattern ElevenLabs documents for client apps; the
secret key never ships inside the app.

To finish setup:

1. Create an agent at https://elevenlabs.io/app/conversational-ai. In the
   agent's settings, set it to require authorization (**private**, not
   public) — that's what makes the token-based flow apply.
2. Paste the agent's ID into `backend/config.php` as `ELEVENLABS_AGENT_ID`.
3. Install the React Native SDK and its WebRTC dependencies, then build a
   custom dev client (this SDK does **not** run in plain Expo Go):
   ```bash
   cd mobile-app
   npm install
   npx expo run:ios   # or: npx expo run:android
   ```
4. Open the Profile tab → "Talk to Assistant" in the app. It fetches a
   token from your backend and starts a WebRTC session with the agent.

`mobile-app/src/tools/agentTools.js` defines client-side functions the agent
can call — search titles, check if an episode's watched, mark a movie or
episode watched, get a show's watched progress. Mirror these names and
parameters as "Client tools" in the ElevenLabs agent dashboard so the agent
can act on your watch data by voice (e.g. "mark The Bear season 1 episode 3
as watched").

Full ElevenLabs React Native docs:
https://elevenlabs.io/docs/eleven-agents/guides/integrations/expo-react-native

## 6. Ideas for later

- Pull IMDb-style "known for" and full filmography via `/person/{id}` (already
  exposed in `services/tmdb.js` as `getPersonDetails`)
- Push notifications when a new episode of a tracked show airs
- Offline caching of watched status
