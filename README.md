# Mystic Lavender Tarot

A beautiful, standalone tarot reading web application with a soft lavender aesthetic, real card artwork, and an AI-powered Mystic Companion.

![Lavender Theme](https://img.shields.io/badge/theme-lavender-purple)
![Cards](https://img.shields.io/badge/cards-78-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## Features

- **Full 78-Card Deck** — Major & Minor Arcana with real artwork from the `Cards-jpg/` folder
- **4 Tarot Spreads** — Single Card, Three Card (Past/Present/Future), Five Card, and Celtic Cross (10 cards)
- **Upright & Reversed Meanings** — Poetic, insightful interpretations for every card
- **3D Card Flip Animations** — Click cards to reveal them with smooth flip effects
- **Shuffle Animation** — Realistic card shuffling before each draw
- **Dark / Light Mode** — Toggle between soft lavender themes
- **Reading Journal** — Save readings to localStorage and revisit them anytime
- **Mystic Companion (AI Chat)** — Gemini-powered companion that remembers your reading history, offers comfort after difficult cards, and provides personalized spiritual guidance
- **Responsive Design** — Works beautifully on mobile and desktop
- **Particle Effects** — Subtle animated background for an ethereal atmosphere
- **Fully Offline Capable** — All interpretations are built-in; no internet required for basic readings

## Tech Stack

- Single HTML file architecture
- Tailwind CSS (via CDN)
- Vanilla JavaScript — no frameworks
- Google Gemini API (for Mystic Companion chat)
- localStorage for persistence (readings & chat memory)

## Folder Structure

```
card reading/
├── tarot.html          # Main application (single file)
├── api-config.js       # Your Gemini API key (gitignored — DO NOT COMMIT)
├── Cards-jpg/          # 78 tarot card images + card back
│   ├── 00-TheFool.jpg
│   ├── 01-TheMagician.jpg
│   ├── ...
│   ├── Cups01.jpg ... Cups14.jpg
│   ├── Wands01.jpg ... Wands14.jpg
│   ├── Swords01.jpg ... Swords14.jpg
│   ├── Pentacles01.jpg ... Pentacles14.jpg
│   └── CardBacks.jpg
├── .gitignore
└── README.md
```

## Local Setup

No build step or server is required for local use.

1. **Clone or download** this repository.
2. **Add your Gemini API key** to `api-config.js`:
   ```javascript
   window.GEMINI_API_KEY = 'your-key-here';
   ```
3. **Open `tarot.html`** in any modern browser (Chrome, Firefox, Edge, Safari).
4. Click **Begin Reading** and enjoy.

> **Note:** The `api-config.js` file is already listed in `.gitignore` to prevent accidental commits of your API key.

## Why Do I Need a Server?

**Short answer:** You don't need one for local, personal use. You **do** need one if you want to share this app publicly.

### The Problem: API Key Exposure

The Mystic Companion uses the Google Gemini API. That requires an API key. Right now, the key lives in `api-config.js`, which is loaded by the browser. 

- **On your own computer:** This is perfectly safe. Only you can see the key.
- **On a public website (GitHub Pages, Netlify, etc.):** Anyone who visits your site can open the browser's Developer Tools, look at the Network tab, and steal your API key. They could then use it to make requests on your behalf and drain your quota or billing.

### The Solution: A Proxy Server

A proxy server sits between your users and the Gemini API. The flow looks like this:

```
User Browser → Your Server → Gemini API
                (hides key)
```

Your server holds the API key secretly. The browser only talks to your server. This is the standard, secure pattern for every production web app that uses paid or rate-limited APIs.

---

## How to Deploy to a Server

### Option 1: Deploy the Frontend Only (No AI Companion)

If you don't need the chat feature, you can host the static files anywhere safely because there is no API key to expose.

**Free static hosts:**
- [GitHub Pages](https://pages.github.com/)
- [Netlify Drop](https://app.netlify.com/drop)
- [Vercel](https://vercel.com/)
- [Surge.sh](https://surge.sh/)

**Steps:**
1. Remove or comment out the `<script src="api-config.js"></script>` line in `tarot.html`.
2. Upload `tarot.html` and the `Cards-jpg/` folder to your host.
3. Done. The tarot readings work fully offline.

### Option 2: Full Deployment with AI Companion (Recommended)

To safely use the Mystic Companion online, you need a tiny backend to proxy the Gemini API calls.

Below is a minimal **Node.js/Express** proxy you can add to the project.

#### Step 1: Create a `server.js` file

```javascript
const express = require('express');
const path = require('path');
const app = express();
const PORT = process.env.PORT || 3000;

// Serve static files (tarot.html, Cards-jpg, css, etc.)
app.use(express.static(path.join(__dirname)));
app.use(express.json());

// Proxy endpoint for Gemini
app.post('/api/chat', async (req, res) => {
    const GEMINI_API_KEY = process.env.GEMINI_API_KEY; // loaded from server environment
    const GEMINI_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${GEMINI_API_KEY}`;

    try {
        const response = await fetch(GEMINI_URL, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(req.body)
        });
        const data = await response.json();
        res.json(data);
    } catch (err) {
        res.status(500).json({ error: 'Proxy error' });
    }
});

app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));
```

#### Step 2: Update `tarot.html` to use your proxy

Find the `sendChatMessage` function and replace the `fetch(GEMINI_URL, ...)` call with:

```javascript
const response = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        systemInstruction: { parts: [{ text: buildSystemPrompt() }] },
        contents: apiMessages,
        generationConfig: { temperature: 0.8, maxOutputTokens: 800 }
    })
});
```

Also remove or comment out the `api-config.js` script tag from the HTML head.

#### Step 3: Set your API key on the server (never in code)

**Local testing:**
```bash
export GEMINI_API_KEY="your-key-here"
node server.js
```

**Production (Render, Railway, Heroku, etc.):**
Set `GEMINI_API_KEY` in the platform's **Environment Variables** dashboard.

#### Step 4: Deploy

1. Push your code to GitHub.
2. Connect your repo to [Render](https://render.com/), [Railway](https://railway.app/), or [Heroku](https://heroku.com/).
3. Add `GEMINI_API_KEY` in the platform's environment variables.
4. Deploy.

---

## Security Checklist

- [ ] `api-config.js` is in `.gitignore` and never committed.
- [ ] If deploying publicly, move the Gemini key to a server-side environment variable.
- [ ] If sharing the repo, replace `api-config.js` with `api-config.example.js` (containing a placeholder) so others know where to put their key.

## Customization

- **Change card meanings:** Edit the `tarotDeck` array in `tarot.html`.
- **Change spreads:** Edit the `spreads` object in `tarot.html`.
- **Change colors:** Modify the Tailwind config inside the `<script>` tag in `tarot.html`.
- **Replace card images:** Drop new JPGs into `Cards-jpg/` and update the `image` paths in the deck data.

## Credits

- Tarot meanings based on traditional Rider-Waite-Smith symbolism.
- AI companion powered by [Google Gemini](https://ai.google.dev/).
- Built with [Tailwind CSS](https://tailwindcss.com/).

## License

MIT — feel free to use, modify, and share. Please keep the `api-config.js` file private.
