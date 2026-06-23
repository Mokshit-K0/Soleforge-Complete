# Soleforge-Complete
A single-page static site for SoleForge with a real Stripe-powered checkout backend.

```
soleforge-vercel/
├── index.html       ← Main site (Tailwind CDN, fonts, inline JS for routing/cart/checkout)
├── vercel.json      ← Vercel static deploy config
├── README.md        ← This file
└── .gitignore
```

---

## What's wired up

| Feature | Status |
|---|---|
| Static site (Home / Shop / Story / Eco / Cart / Contact) | ✅ |
| Cart with quantity, remove, promo codes | ✅ |
| Server-side pricing & tax (no client-trusted amounts) | ✅ |
| Stripe Checkout (test mode) | ✅ |
| Order tracking + payment polling | ✅ |
| Webhook endpoint for Stripe events | ✅ |

The backend is a FastAPI + MongoDB service living in `/app/backend` (this repo's `../backend` if you cloned the full project). The static HTML calls it via fetch.

---

## Deploy the static site to Vercel

### Configure backend URL (one-line edit)

The static HTML defaults to a preview backend URL. **Before deploying**, edit `index.html` and find this line near the bottom of the `<script>` block:

```js
const API_BASE = (window.SOLEFORGE_API || \"https://vercel-ready-13.preview.emergentagent.com\").replace(/\/$/, \"\");
```

Replace the URL with your deployed FastAPI backend URL (e.g. `https://api.soleforge.com`).

**OR** — add this `<script>` tag in `<head>` before the main script:

```html
<script>window.SOLEFORGE_API = \"https://api.soleforge.com\";</script>
```

### Three ways to deploy

**1. Drag & drop (easiest)** — go to https://vercel.com/new → drop the folder → Framework: *Other* → Deploy.

**2. Vercel CLI**
```bash
npm i -g vercel
cd soleforge-vercel
vercel --prod
```

**3. GitHub + Vercel** — push folder to a repo → Import on vercel.com → Deploy.

No build step, no install. Vercel just serves `index.html`.

### Local preview
```bash
cd soleforge-vercel
python3 -m http.server 3000
# open http://localhost:3000
```

---

## Deploying the backend (FastAPI + MongoDB)

The backend is **not** in this folder. It lives at `/app/backend` in the parent project and is currently running locally on the preview pod. To deploy it standalone:

### Required environment variables
```
MONGO_URL=mongodb+srv://...           # MongoDB Atlas free tier works
DB_NAME=soleforge
CORS_ORIGINS=https://your-vercel-domain.vercel.app
STRIPE_API_KEY=sk_test_emergent       # test key — replace with sk_live_... for prod
```

### Recommended hosts
- **Render** — Free tier, auto-deploys from GitHub. Set start command to `uvicorn server:app --host 0.0.0.0 --port $PORT`
- **Railway** — Similar workflow, $5/month after free credits.
- **Fly.io** — Free shared CPU + always-on.

### Stripe webhook
After deploying the backend, in your Stripe Dashboard add a webhook endpoint:
```
https://<your-backend-domain>/api/webhook/stripe
```
Events: `checkout.session.completed`, `checkout.session.expired`.

---

## Test the checkout flow

1. Open the Vercel site → navigate to **Cart**.
2. Click **SECURE CHECKOUT**.
3. You'll be redirected to Stripe's test checkout.
4. Use card `4242 4242 4242 4242`, any future expiry, any CVC.
5. After success, you'll land back on the site at `?page=success&session_id=cs_test_...` which polls the backend and confirms payment.

---

## API endpoints (backend)

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/` | Health check |
| GET | `/api/products` | List products (server-side prices) |
| GET | `/api/products/{id}` | Get single product |
| POST | `/api/cart/quote` | Get totals (no DB write) |
| POST | `/api/checkout/session` | Create Stripe session + order record |
| GET | `/api/checkout/status/{session_id}` | Poll payment status |
| POST | `/api/webhook/stripe` | Stripe webhook handler |

---

## Notes
- Promo codes: `FORGE2024` (10% off), `BATCH001` (15% off) — defined server-side in `server.py`.
- Tax: 9% (configurable in `server.py` → `TAX_RATE`).
- Shipping: flat $12.
- Stripe is in **test mode** — no real charges happen.
"
