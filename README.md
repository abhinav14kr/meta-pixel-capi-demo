# Meta Pixel + CAPI Demo

Test your Meta Pixel and Conversions API (CAPI) integration. Send browser and server events simultaneously, verify deduplication, and confirm results in Events Manager.

**Live demo:** https://abhinav14kr.github.io/meta-pixel-capi-demo/

---

## How It Works

When a user triggers an event (e.g. "Add to Cart"), two things happen:

1. **Browser Pixel** sends the event to Meta via `fbq('track', ...)`
2. **Your backend** sends the same event to Meta's Graph API via CAPI

Both share the same **Event ID**, so Meta deduplicates and counts once. If the Pixel is blocked (ad blocker), CAPI still delivers.

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

The frontend never holds your access token — it sends raw event data to your backend, which hashes PII with SHA-256, attaches the token, and forwards to Meta.

---

## Getting Your Credentials

You need three things from Meta. Here's where to find each:

1. **Pixel ID** — Go to [Events Manager](https://business.facebook.com/events_manager2), select your Pixel. The ID is in the header (a number like `123456789012345`).

2. **Access Token** — Go to [Business Settings > System Users](https://business.facebook.com/settings/system-users). Create a System User (Admin role), click **Generate New Token**, select your app, and grant the `ads_management` permission. Copy the token immediately — it won't be shown again.

3. **Test Event Code** (optional) — In Events Manager, select your Pixel, go to the **Test Events** tab. The code is shown at the top (e.g. `TEST12345`). Using this routes events to the test view instead of production data.

---

## Quick Start

### 1. Start the backend

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

### 2. Open the frontend

Serve locally with `cd ../docs && npx serve .`, or use the [live demo](https://abhinav14kr.github.io/meta-pixel-capi-demo/).

### 3. Configure and send events

1. Enter your Pixel ID, CAPI Backend URL (`http://localhost:3000/api/event`), and optionally a Test Event Code
2. Click **Save Configuration**
3. Enter a name and email, click any event button
4. Check the log panel for results

### 4. Verify in Events Manager

Go to Events Manager > your Pixel > Test Events tab. Events should appear within seconds.

---

## Deploy Your Own Backend (Render)

Each person needs their own backend — it holds your Pixel ID and access token. Here's how to set one up on [Render](https://render.com) (free tier works):

1. Fork this repo on GitHub
2. Go to [render.com](https://render.com), sign up, click **New > Web Service**
3. Connect your GitHub account, select your fork
4. Set **Root Directory** to `backend`, **Build Command** to `npm install`, **Start Command** to `npm start`
5. Add environment variables:

| Variable | Required | Description |
|---|---|---|
| `PIXEL_ID` | Yes | Your Facebook Pixel ID |
| `FB_ACCESS_TOKEN` | Yes | Access token with pixel permissions |
| `TEST_EVENT_CODE` | No | Routes events to Test Events tab |

6. Click **Deploy**. Your backend URL will be `https://your-service-name.onrender.com/api/event`
7. Add your frontend domain to `ALLOWED_ORIGINS` in `backend/server.js` for CORS

You can also deploy to Railway, Heroku, or any Node.js host — the setup is the same.

### Frontend hosting

Fork the repo, go to Settings > Pages, set branch to `main` and folder to `/docs`. Your site will be at `https://YOUR_USERNAME.github.io/meta-pixel-capi-demo/`.

---

## Test Lab

The [Test Lab](https://abhinav14kr.github.io/meta-pixel-capi-demo/test-lab.html) page lets you toggle channels and simulate scenarios:

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
    "content_ids": ["PROD-1234"],
    "value": 49.99,
    "currency": "USD"
  },
  "userData": {
    "email": "test@example.com",
    "firstName": "John",
    "lastName": "Doe"
  },
  "eventSourceUrl": "https://yoursite.com/products",
  "testEventCode": "TEST12345"
}
```

The backend hashes user data (email, phone, name) with SHA-256 before sending to Meta. `fbc` and `fbp` cookies are passed as-is.

---

## Production Notes

This is a test tool. For production: obtain user consent before firing Pixel (GDPR/CCPA), add `data_processing_options` for US users (LDU), support `opt_out` in user_data, and keep `API_VERSION` in `server.js` up to date.

---

## Resources

- [Conversions API](https://developers.facebook.com/docs/marketing-api/conversions-api)
- [Meta Pixel](https://developers.facebook.com/docs/meta-pixel)
- [Deduplication Guide](https://developers.facebook.com/docs/marketing-api/conversions-api/deduplicate-pixel-and-server-events)
- [Test Events](https://developers.facebook.com/docs/marketing-api/conversions-api/using-the-api#test-events)
- [Limited Data Use](https://developers.facebook.com/docs/marketing-apis/data-processing-options)
