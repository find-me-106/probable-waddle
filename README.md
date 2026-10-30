# Handle — username chat app

Frontend (static HTML/CSS/JS) and backend (Flask API) are now **separate** and can be hosted independently.

```
backend/    app.py, config.py, requirements.txt, Procfile, .env.example   -> Python API
frontend/   *.html, css/, js/ (js/config.js holds the API URL)            -> static site
```

## 1. Run locally
```
cd backend
pip install -r requirements.txt
cp .env.example .env            # fill in MONGO_URI, etc.
# put serviceAccountKey.json in backend/
python app.py                   # API on http://localhost:5000

cd ../frontend
python -m http.server 3000      # site on http://localhost:3000
```
`frontend/js/config.js` -> `API_URL = "http://localhost:5000"`.
Camera/mic need localhost or HTTPS.

## 2. Deploy the backend (Render / Railway / any Python host)
- Root directory: `backend`
- Build: `pip install -r requirements.txt`
- Start: `gunicorn app:app --bind 0.0.0.0:$PORT --workers 2 --threads 4 --timeout 60` (see `Procfile`)
- Environment variables: `MONGO_URI`, `DB_NAME`, `FIREBASE_KEY_JSON` (whole serviceAccountKey.json), `CORS_ORIGINS` (your frontend URL), `FLASK_DEBUG=0`
- MongoDB Atlas -> Network Access: allow your host (or `0.0.0.0/0`).

## 3. Deploy the frontend (Netlify / Vercel / GitHub Pages / Cloudflare Pages)
- Set `API_URL` in `frontend/js/config.js` to your deployed API URL (e.g. `https://your-api.onrender.com`).
- Publish the `frontend` folder (no build step).
- Firebase console -> Authentication -> Settings -> **Authorized domains**: add your frontend domain.
- Then set `CORS_ORIGINS` on the backend to that frontend URL.

## 4. AI assistant (Groq)
- Pinned as **Nova** at the top of every chat list. Chat with it, tap the 🎧 button to **talk out loud**, and it points out your spelling/grammar mistakes (✏️ line before its answer).
- The ✨ button next to the message box fixes mistakes in **any** chat before you send (tap again to undo).
- The Groq key is in `frontend/js/config.js` -> `AI_CONFIG.apiKey` (model: `AI_CONFIG.model`); the browser calls Groq directly through `frontend/js/ai.js`. The file is public, so set a usage limit for the key in the Groq console.
- Default name / voice language / history size: `AI_CONFIG` in `frontend/js/config.js`.

## 5. Settings - every person customises their own app
`settings.html` (gear icon in the sidebar): theme (dark/light/black/auto), accent colours or your own two colours, background, text size, font, bubble shape, reduce motion, show/hide online + last seen, message sound, 24h clock, AI name / personality / language / instructions, call quality, join muted / camera off. Stored per user in MongoDB. Defaults for new users: `DEFAULT_SETTINGS` in `config.js`.

## 6. Last seen
Header of a direct chat shows **online** or *last seen today at 14:05*. If you hide yours in Settings you will not see other people's either.

## 7. Calls
`ICE_SERVERS` in `config.js` holds STUN + TURN. Replace the free Open Relay TURN with your own (metered.ca / Twilio / coturn) for real use - without TURN calls often work on one side only on mobile data.

## Notes
- Never commit `backend/.env` or `serviceAccountKey.json`.
- Chat uses polling (2.5 s); calls poll every 1-2 s. Video calls are 1-to-1 (STUN only; add TURN for strict networks).
