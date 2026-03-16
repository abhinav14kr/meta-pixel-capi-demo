# Meta Pixel + CAPI Demo

A demo for integrating Conversions API (CAPI) with browser Pixel events to enable server-side tracking and event deduplication. The interactive front-end allows you to trigger and view both Pixel and CAPI events in real time, making it easy to test and validate your setup.

**My instance:** https://abhinav14kr.github.io/pixel/

You can follow the guide to setup front-end with your pixel and track these events directly in Events Manager, ensuring your implementation is working as expected.

---

## What's Included

- **Browser Pixel:** Sends events from the browser to Facebook.
- **Server CAPI:** Sends the same events from your backend to Facebook's Graph API.
- **Deduplication:** Both use the same Event ID, so Facebook counts them as one event.

**Benefits:**
- Improved data accuracy (server events bypass ad blockers)
- Better event matching (server can send hashed user data)
- Redundancy (if one fails, the other works — test with browser restrictions to observe CAPI fallback)

---

## Project Structure

```
meta-pixel-capi-demo/
├── frontend/          # Static HTML (Pixel + CAPI client)
│   ├── index.html
│   └── test-lab.html
├── backend/           # Node.js Express server (CAPI endpoint)
│   ├── server.js
│   └── package.json
└── README.md
```

---

## Quick Start

1. **Prerequisites:**
   - Node.js 18+
   - Facebook Pixel ID
   - Facebook Access Token (System User token with pixel access, or User token with `ads_management` permission)

2. **Clone & Install:**
   ```bash
   git clone https://github.com/abhinav14kr/meta-pixel-capi-demo.git
   cd meta-pixel-capi-demo/backend
   npm install
   export PIXEL_ID="your_pixel_id_here"
   export FB_ACCESS_TOKEN="your_access_token_here"
   npm start
   ```

3. **Configure Frontend:**
   - Edit `frontend/index.html`:
     - Replace `YOUR_PIXEL_ID` (2 places: Pixel init and noscript fallback)
     - Update `PIXEL_ID` constant in the script section
     - Set `CAPI_URL` to your backend (`http://localhost:3000/api/event` or production URL)

4. **Test Locally:**
   ```bash
   cd frontend
   npx serve .
   ```

---

## Deploy to Production

- **Backend:**
  - Deploy `backend/` to [Render](https://render.com/) or any Node.js hosting service
  - Set `PIXEL_ID` and `FB_ACCESS_TOKEN` env variables

- **Frontend:**
  - Deploy `frontend/` to GitHub Pages or any static host

- **CORS:**
  - Add your frontend domain to `ALLOWED_ORIGINS` in `backend/server.js`

---

## Event Deduplication

Both browser and server send events with the same Event ID. Facebook deduplicates and counts as one event.

---

## API Reference

**POST /api/event**
Send server-side event to Facebook CAPI.

**Request Example:**
```json
{
  "eventName": "AddToCart",
  "eventId": "AddToCart_1234567890_abc123",
  "eventData": { ... },
  "userData": { ... },
  "eventSourceUrl": "https://yoursite.com/products"
}
```

**Response Example:**
```json
{
  "success": true,
  "eventId": "AddToCart_1234567890_abc123",
  "eventTime": 1234567890,
  "result": { "events_received": 1 }
}
```

---

## Testing with Test Events

To route events to Events Manager's **Test Events** tab (without polluting production data):

1. Go to **Events Manager > Your Pixel > Test Events**
2. Copy the **Test Event Code** (e.g., `TEST12345`)
3. Set it as an env var on your backend:
   ```bash
   export TEST_EVENT_CODE="TEST12345"
   ```
   Or enter it in the Test Lab's Configuration section.

---

## Production Considerations

This demo is designed for testing. For production implementations, also consider:

- **Consent Management**: Pixel should only fire after user consent (GDPR/CCPA)
- **Limited Data Use (LDU)**: Add `data_processing_options` to the CAPI payload for US users
- **Opt-Out Handling**: Support `opt_out` field in user_data for users who opt out of tracking
- **Graph API Versioning**: Update `API_VERSION` in `server.js` periodically as Meta deprecates old versions

---

## Resources

- [Conversions API Docs](https://developers.facebook.com/docs/marketing-api/conversions-api)
- [Pixel Docs](https://developers.facebook.com/docs/meta-pixel)
- [Event Deduplication Guide](https://developers.facebook.com/docs/marketing-api/conversions-api/deduplicate-pixel-and-server-events)
- [Test Events](https://developers.facebook.com/docs/marketing-api/conversions-api/using-the-api#test-events)
- [Limited Data Use](https://developers.facebook.com/docs/marketing-apis/data-processing-options)

---
