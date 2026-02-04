A demo showing how to set up **Facebook Conversions API (CAPI)** with browser **Pixel events** for server-side tracking and event deduplication.

---

## What’s Included

- **Browser Pixel:** Sends events from the browser to Facebook.
- **Server CAPI:** Sends the same events from your backend to Facebook’s Graph API.
- **Deduplication:** Both use the same Event ID, so Facebook counts them as one event.

**Benefits:**  
- Improved data accuracy (server events bypass ad blockers)  
- Better event matching (server can send hashed user data)  
- Redundancy (if one fails, the other works)

---

## Project Structure

```
meta-capi-demo/
├── frontend/      # Static HTML (Pixel + CAPI client)
├── backend/       # Node.js Express server (CAPI endpoint)
└── README.md
```

---

## Quick Start

1. **Prerequisites:**  
   - Node.js 18+  
   - Facebook Pixel ID  
   - Facebook Access Token (`ads_management` permission)

2. **Clone & Install:**
   ```bash
   git clone https://github.com/YOUR_USERNAME/meta-capi-demo.git
   cd meta-capi-demo/backend
   npm install
   export FB_ACCESS_TOKEN="your_access_token_here"
   npm start
   ```

3. **Configure Frontend:**  
   - Edit `frontend/index.html`:  
     - Replace `YOUR_PIXEL_ID` (3 places)  
     - Set `CAPI_URL` to your backend (`http://localhost:3000/api/event` or production URL)

4. **Test Locally:**  
   ```bash
   cd frontend
   npx serve .
   ```

---

## Deploy to Production

- **Backend:**  
  - Deploy `backend/` to [Render](https://render.com/)  
  - Set `FB_ACCESS_TOKEN` env variable

- **Frontend:**  
  - Deploy `frontend/` to GitHub Pages

- **CORS:**  
  - Add your GitHub Pages domain to CORS in `backend/server.js`

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

## Resources

- [Conversions API Docs](https://developers.facebook.com/docs/marketing-api/conversions-api)
- [Pixel Docs](https://developers.facebook.com/docs/meta-pixel)
- [Event Deduplication Guide](https://developers.facebook.com/docs/marketing-api/conversions-api/deduplicate-pixel-and-server-events)
- [Meta Events Manager](https://business.facebook.com/events_manager)

--- 
