# Meta Pixel + CAPI Demo

Test and validate your Meta Pixel and Conversions API (CAPI) integration. Send events from the browser and server simultaneously, verify deduplication, and see results in Events Manager.

**Live demo:** https://abhinav14kr.github.io/meta-pixel-capi-demo/

---

## How It Works

When a user takes an action (e.g. "Add to Cart"), two things happen at the same time:

1. **Browser Pixel** sends the event directly to Meta via `fbq('track', ...)`
2. **Server CAPI** sends the same event from your backend to Meta's Graph API

Both use the same **Event ID**, so Meta counts them as one event (deduplication). If the browser event is blocked (e.g. ad blocker), the server event still gets through.

```
User clicks "Add to Cart"
        |
        +---> Browser Pixel ---> Meta (via fbevents.js)
        |         event_id: "ABC123"
        |
        +---> Your Backend ---> Meta Graph API
                  event_id: "ABC123"

Meta sees both, matches on event_id, counts once.
```

---

## Project Structure

```
meta-pixel-capi-demo/
  docs/                 <- Frontend (GitHub Pages)
    index.html          <- Simple mode: configure and send events
    test-lab.html       <- Test Lab: toggle Pixel/CAPI, simulate ad blockers
  backend/              <- Node.js server (CAPI endpoint)
    server.js
    package.json
```

---

## Setup Guide

### What You Need

- Node.js 18+
- A Facebook **Pixel ID** (from Events Manager)
- A Facebook **Access Token** (System User token with pixel access, or User token with `ads_management` permission)
- Optionally, a **Test Event Code** (from Events Manager > Test Events tab)

### Step 1: Start the Backend

```bash
git clone https://github.com/abhinav14kr/meta-pixel-capi-demo.git
cd meta-pixel-capi-demo/backend
npm install

export PIXEL_ID="your_pixel_id"
export FB_ACCESS_TOKEN="your_access_token"
export TEST_EVENT_CODE="TEST12345"   # optional

npm start
```

The server starts at `http://localhost:3000`. Verify by opening that URL in your browser — you should see a JSON health check response.

### Step 2: Open the Frontend

Option A — serve locally:
```bash
cd ../docs
npx serve .
```

Option B — use the live demo at https://abhinav14kr.github.io/meta-pixel-capi-demo/

### Step 3: Configure and Test

1. Enter your **Pixel ID** in the Configuration section
2. Enter your **CAPI Backend URL** (e.g. `http://localhost:3000/api/event` or your deployed URL)
3. Optionally enter a **Test Event Code** to route events to the Test Events tab instead of production
4. Click **Save Configuration**
5. Enter a name and email, then click any event button (Add to Cart, Purchase, etc.)
6. Watch the log panel — it shows whether Pixel and CAPI succeeded

### Step 4: Verify in Events Manager

1. Go to **Events Manager** in your Meta Business Suite
2. Click on your Pixel
3. Go to the **Test Events** tab (if you used a Test Event Code)
4. You should see your events appear within seconds

---

## Deploying Your Own Instance

### Backend

Deploy the `backend/` folder to any Node.js host (Render, Railway, Heroku, etc.). Set these environment variables:

| Variable | Required | Description |
|---|---|---|
| `PIXEL_ID` | Yes | Your Facebook Pixel ID |
| `FB_ACCESS_TOKEN` | Yes | Access token with pixel permissions |
| `TEST_EVENT_CODE` | No | Routes events to Test Events tab |

After deploying, add your frontend domain to `ALLOWED_ORIGINS` in `server.js` for CORS.

### Frontend

Option A — **GitHub Pages**: Fork the repo, go to Settings > Pages, set branch to `main` and folder to `/docs`. Your site will be at `https://YOUR_USERNAME.github.io/meta-pixel-capi-demo/`.

Option B — Deploy the `docs/` folder to any static host.

---

## Test Lab

The Test Lab page (`test-lab.html`) lets you experiment with different scenarios:

| Scenario | What it does |
|---|---|
| Both Working | Pixel + CAPI fire together. Events are deduplicated. |
| Pixel Only | CAPI disabled. Shows browser-only tracking. |
| CAPI Only | Pixel disabled. Shows server-only tracking. |
| Ad Blocked | Pixel is simulated as blocked. CAPI still works. |

It also tracks per-session stats: how many events went through each channel and average CAPI latency.

---

## API Reference

**POST /api/event**

Send a server-side event to Meta's Conversions API.

Request:
```json
{
  "eventName": "AddToCart",
  "eventId": "AddToCart_1710000000_abc123",
  "eventData": {
    "content_name": "Test Product",
    "content_ids": ["PROD-1234"],
    "content_type": "product",
    "value": 49.99,
    "currency": "USD"
  },
  "userData": {
    "email": "test@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "phone": "+1234567890"
  },
  "eventSourceUrl": "https://yoursite.com/products",
  "testEventCode": "TEST12345"
}
```

Response:
```json
{
  "success": true,
  "eventId": "AddToCart_1710000000_abc123",
  "eventTime": 1710000000,
  "result": { "events_received": 1 }
}
```

The backend hashes all user data (email, phone, name) with SHA-256 before sending to Meta. Facebook click ID (`fbc`) and browser ID (`fbp`) are passed as-is.

---

## Resources

- [Conversions API Documentation](https://developers.facebook.com/docs/marketing-api/conversions-api)
- [Meta Pixel Documentation](https://developers.facebook.com/docs/meta-pixel)
- [Event Deduplication Guide](https://developers.facebook.com/docs/marketing-api/conversions-api/deduplicate-pixel-and-server-events)
- [Test Events](https://developers.facebook.com/docs/marketing-api/conversions-api/using-the-api#test-events)
