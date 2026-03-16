# Meta Pixel + CAPI Demo

A hands-on tool for testing Meta Pixel and Conversions API (CAPI) together — fire browser and server events, verify deduplication, and see results in Events Manager.

**Live demo:** https://abhinav14kr.github.io/meta-pixel-capi-demo/test-lab.html

---

## How It Works

When a user triggers an event (e.g. "Add to Cart"):

1. **Browser Pixel** fires the event via `fbq('track', ...)`
2. **Your backend** sends the same event to Meta's Graph API via CAPI

Both share the same **Event ID**, so Meta deduplicates them. If the Pixel is blocked (ad blocker), CAPI still delivers. Your access token never touches the frontend.

```
User clicks "Add to Cart"
        |
        +---> Browser Pixel ---> Meta (via fbevents.js)
        |         event_id: "ABC123"
        |
        +---> Your Backend ---> Meta Graph API
                  event_id: "ABC123"

Meta matches on event_id, counts as one event.
```

---

## Getting Your Credentials

All three come from [Events Manager](https://business.facebook.com/events_manager2) — select your Pixel, then:

1. **Pixel ID** — shown in the header (a number like `123456789012345`)
2. **Access Token** — go to **Settings > Generate Access Token**. Copy it immediately, it won't be shown again.
3. **Test Event Code** (optional) — go to **Test Events** tab, code is at the top (e.g. `TEST12345`). Routes events to the test view instead of production.

---

## Set Up Your Backend

Each person needs their own backend — it holds your credentials as env vars.

### Option A: Deploy to Render (recommended)

1. Fork this repo, then go to [render.com](https://render.com) and create a **New > Web Service** from your fork
2. Set **Root Directory** to `backend`, **Build Command** to `npm install`, **Start Command** to `npm start`
3. Add env vars: `PIXEL_ID` (required), `FB_ACCESS_TOKEN` (required), `TEST_EVENT_CODE` (optional)
4. Deploy — your backend URL will be `https://your-service-name.onrender.com/api/event`
5. Add your frontend domain to `ALLOWED_ORIGINS` in `backend/server.js`

Works the same on Railway, Heroku, or any Node.js host.

### Option B: Run locally

```bash
git clone https://github.com/abhinav14kr/meta-pixel-capi-demo.git
cd meta-pixel-capi-demo/backend
npm install

export PIXEL_ID="your_pixel_id"
export FB_ACCESS_TOKEN="your_access_token"
export TEST_EVENT_CODE="TEST12345"   # optional

npm start
```

Verify at `http://localhost:3000` — you should see a JSON health check.

---

## Quick Start

1. Open the [live demo](https://abhinav14kr.github.io/meta-pixel-capi-demo/test-lab.html) (or serve locally: `cd docs && npx serve .`)
2. Enter your **Pixel ID**, **CAPI Backend URL**, and optionally a **Test Event Code**, then save
3. Fill in a name and email, click any event button
4. Check the log panel, then head to **Events Manager > Test Events** — events should appear within seconds

---

## Fork & Run Your Own

Want your own instance? No credentials are hardcoded, so it's straightforward:

1. **Fork** this repo on GitHub
2. **Deploy your backend** with your own `PIXEL_ID` and `FB_ACCESS_TOKEN` (see [above](#set-up-your-backend))
3. **Add your GitHub Pages domain** to `ALLOWED_ORIGINS` in `backend/server.js` (e.g. `'https://yourusername.github.io'`)
4. **Enable GitHub Pages** — Settings > Pages, branch `main`, folder `/docs`
5. Open `https://yourusername.github.io/meta-pixel-capi-demo/test-lab.html` and enter your backend URL

---

## Test Lab Scenarios

The Test Lab lets you toggle channels (Pixel, CAPI, Ad Blocker) to simulate:

| Scenario | What happens |
|---|---|
| Both Working | Pixel + CAPI fire together, deduplicated |
| Pixel Only | No server backup — vulnerable to ad blockers |
| CAPI Only | Server-side only — works regardless of browser |
| Ad Blocked | Pixel blocked, CAPI still delivers |

---

## API Reference

**POST /api/event**

```json
{
  "eventName": "AddToCart",
  "eventId": "AddToCart_1710000000_abc123",
  "eventData": {
    "content_name": "Test Product",
    "value": 49.99,
    "currency": "USD"
  },
  "userData": {
    "email": "test@example.com",
    "firstName": "John"
  },
  "eventSourceUrl": "https://yoursite.com/products",
  "testEventCode": "TEST12345"
}
```

The backend hashes PII (email, phone, name) with SHA-256 before sending to Meta.

---

## Production Notes

This is a test tool. For production: obtain user consent before firing Pixel (GDPR/CCPA), add `data_processing_options` for US users (LDU), support `opt_out` in user_data, and keep `API_VERSION` in `server.js` up to date.

---

## Resources

- [Conversions API](https://developers.facebook.com/docs/marketing-api/conversions-api)
- [Meta Pixel](https://developers.facebook.com/docs/meta-pixel)
- [Deduplication Guide](https://developers.facebook.com/docs/marketing-api/conversions-api/deduplicate-pixel-and-server-events)
- [Test Events](https://developers.facebook.com/docs/marketing-api/conversions-api/using-the-api#test-events)
