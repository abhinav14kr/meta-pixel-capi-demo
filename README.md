# Meta Conversions API (CAPI) + Pixel Integration Demo

A complete working example demonstrating how to set up **Facebook Conversions API (CAPI)** alongside browser **Pixel events** for server-side tracking with event deduplication.

## 🎯 What This Demo Does

1. **Browser Pixel**: Sends events directly from the user's browser to Facebook
2. **Server CAPI**: Sends the same events from your backend server to Facebook's Graph API
3. **Deduplication**: Both events share the same Event ID, so Facebook counts them as one event

This dual-tracking approach provides:
- Better data accuracy (server events aren't blocked by ad blockers)
- Improved event matching (server can send hashed user data)
- Redundancy (if one fails, the other still works)

## 📁 Project Structure

```
meta-capi-demo/
├── frontend/                 # Static HTML (deploy to GitHub Pages)
│   └── index.html           # Pixel + CAPI client code
├── backend/                  # Node.js Express server (deploy to Render)
│   ├── server.js            # CAPI endpoint + Facebook Graph API calls
│   └── package.json         # Dependencies
└── README.md                # This file
```

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ installed
- A Facebook Pixel ID (get from [Meta Events Manager](https://business.facebook.com/events_manager))
- A Facebook Access Token with `ads_management` permission (generate from Events Manager → Settings → Generate Access Token)

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/meta-capi-demo.git
cd meta-capi-demo
```

### 2. Set Up Backend (Local Testing)

```bash
cd backend
npm install

# Set your Facebook Access Token
export FB_ACCESS_TOKEN="your_access_token_here"

# Start the server
npm start
```

The server will run at `http://localhost:3000`

### 3. Update Frontend Configuration

Edit `frontend/index.html`:

1. Replace all instances of `YOUR_PIXEL_ID` with your actual Facebook Pixel ID (appears in 3 places)
2. Update `CAPI_URL` to your backend URL:
   - Local: `http://localhost:3000/api/event`
   - Production: `https://your-backend.onrender.com/api/event`

### 4. Test Locally

Open `frontend/index.html` in a browser, or use a local server:

```bash
cd frontend
npx serve .
# Opens at http://localhost:3000
```

## ☁️ Production Deployment

### Deploy Backend to Render.com

1. Create a new **Web Service** on [Render](https://render.com)
2. Connect your GitHub repo
3. Configure:
   - **Root Directory**: `backend`
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
4. Add Environment Variable:
   - `FB_ACCESS_TOKEN` = your Facebook access token
5. Deploy and note your URL (e.g., `https://your-app.onrender.com`)

### Deploy Frontend to GitHub Pages

1. Go to your repo Settings → Pages
2. Source: Deploy from branch `main`
3. Folder: `/frontend` (or root if you move index.html)
4. Your site will be at `https://username.github.io/meta-capi-demo/frontend/`

### Update CORS and URLs

In `backend/server.js`, add your GitHub Pages domain to CORS:

```javascript
app.use(cors({
    origin: [
        'https://YOUR_USERNAME.github.io',
        'http://localhost:3000'
    ]
}));
```

In `frontend/index.html`, update the CAPI_URL:

```javascript
const CAPI_URL = 'https://your-app.onrender.com/api/event';
```

## 🔧 Configuration Reference

### Backend Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `FB_ACCESS_TOKEN` | Yes | Facebook System User Access Token |
| `PIXEL_ID` | Yes | Your Facebook Pixel ID |
| `PORT` | No | Server port (default: 3000) |

### Frontend Configuration (index.html)

| Variable | Description |
|----------|-------------|
| `YOUR_PIXEL_ID` | Replace with your Facebook Pixel ID (3 places) |
| `CAPI_URL` | Your backend API endpoint |
| `PIXEL_ID` | JavaScript variable for logging |

## 📊 How Event Deduplication Works

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Clicks "Add to Cart"                │
└─────────────────────────────────────────────────────────────────┘
                                  │
                    Generate Event ID: "AddToCart_1234567890_abc123"
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
        ┌───────────────────┐       ┌───────────────────┐
        │   Browser Pixel   │       │   CAPI Backend    │
        │                   │       │                   │
        │ fbq('track',      │       │ POST /api/event   │
        │   'AddToCart',    │       │ {eventId: "..."}  │
        │   {eventID:"..."})│       │                   │
        └─────────┬─────────┘       └─────────┬─────────┘
                  │                           │
                  ▼                           ▼
        ┌───────────────────────────────────────────────┐
        │              Facebook Events Manager          │
        │                                               │
        │  Event ID: "AddToCart_1234567890_abc123"      │
        │  Sources: Browser ✓  Server ✓                 │
        │  Status: DEDUPLICATED (counted as 1 event)    │
        └───────────────────────────────────────────────┘
```

## 🔍 Testing & Debugging

### Verify Pixel Installation

1. Install [Meta Pixel Helper](https://chrome.google.com/webstore/detail/meta-pixel-helper/fdgfkebogiimcoedlicjlajpkdmockpc) Chrome extension
2. Visit your frontend page
3. Click the extension icon - it should show your Pixel ID and events

### Verify CAPI Backend

```bash
# Check if server is running
curl https://your-backend.onrender.com/

# Check configuration
curl https://your-backend.onrender.com/api/test

# Send a test event
curl -X POST https://your-backend.onrender.com/api/event \
  -H "Content-Type: application/json" \
  -d '{
    "eventName": "TestEvent",
    "eventId": "test_123",
    "userData": {"email": "test@example.com"},
    "eventSourceUrl": "https://example.com"
  }'
```

### Check Meta Events Manager

1. Go to [Events Manager](https://business.facebook.com/events_manager)
2. Select your Pixel
3. Click **Test Events** tab
4. Trigger events from your frontend
5. Events should appear with both "Browser" and "Server" labels

## 🛠️ API Reference

### POST /api/event

Send a server-side event to Facebook CAPI.

**Request Body:**

```json
{
  "eventName": "AddToCart",
  "eventId": "AddToCart_1234567890_abc123",
  "eventData": {
    "content_name": "Product Name",
    "content_ids": ["SKU123"],
    "content_type": "product",
    "value": 49.99,
    "currency": "USD"
  },
  "userData": {
    "email": "user@example.com",
    "phone": "+1234567890",
    "firstName": "John",
    "lastName": "Doe",
    "fbc": "fb.1.1234567890.abcdef",
    "fbp": "fb.1.1234567890.123456789"
  },
  "eventSourceUrl": "https://yoursite.com/products",
  "clientUserAgent": "Mozilla/5.0..."
}
```

**Response:**

```json
{
  "success": true,
  "eventId": "AddToCart_1234567890_abc123",
  "eventTime": 1234567890,
  "result": {
    "events_received": 1,
    "messages": []
  }
}
```

## 📚 Resources

- [Conversions API Documentation](https://developers.facebook.com/docs/marketing-api/conversions-api)
- [Pixel Documentation](https://developers.facebook.com/docs/meta-pixel)
- [Event Deduplication Guide](https://developers.facebook.com/docs/marketing-api/conversions-api/deduplicate-pixel-and-server-events)
- [Meta Events Manager](https://business.facebook.com/events_manager)

## 🤝 Contributing

Feel free to submit issues and pull requests!

## 📄 License

MIT License - feel free to use this code for your own projects.
