# Zylo Ecommerce

Zylo Ecommerce is a full‑stack demo storefront featuring a static Frontend and a Node/Express Backend API backed by MongoDB. It includes OTP login, cart and wishlist sync, checkout, orders, coupons, ratings, and wallet support.

## Prerequisites

- Node.js 18+
- MongoDB running locally or a cloud instance

## Quick Start

1. Install and start the Backend API:

```bash
cd Backend
npm install
npm start
```

2. Serve the Frontend:

```bash
cd ..
npx --yes http-server -p 8080 Frontend
```

3. Open the app:

- Frontend: http://localhost:8080/pages/index.html
- API base: http://localhost:5000/api

## Environment Configuration

Create a `.env` file in `Backend/` with the following keys:

```env
MONGODB_URI=mongodb://localhost:27017/zylo-ecommerce
JWT_SECRET=replace-with-a-long-random-string
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@example.com
EMAIL_PASS=app-password-or-smtp-password
EMAIL_FROM=no-reply@zylo.com
```

Do not commit secrets. Use environment variables for production.

## Project Structure

```
Zylo Ecommerce/
├─ Frontend/              # Static site (HTML/CSS/JS)
│  ├─ pages/              # HTML pages
│  ├─ styles/             # CSS files
│  ├─ js/                 # Client-side logic
│  └─ img/                # Assets
└─ Backend/               # Node/Express API
   ├─ models/             # Mongoose models
   ├─ routes/             # Express routes
   ├─ middleware/         # Auth/validation
   ├─ utils/              # Helpers and scripts
   ├─ seeders/            # Data population scripts
   ├─ migrations/         # Data migrations
   ├─ server.js           # App entrypoint
   └─ package.json
```

## Common Workflows

- Login via email OTP on `pages/login.html`; upon verification, the app returns to the page stored in `localStorage.zylo_return_to`.
- Cart and wishlist sync with the server after login.
- Checkout requires at least one saved address; if missing, the app redirects to `account.html?tab=addresses` and returns to checkout after saving or setting default.

## Useful Scripts

Run from `Backend/`:

- Seed demo ratings: `node seeders/populate-ratings.js`
- Populate missing ratings: `node seeders/populate-missing-ratings.js`
- Set admin password: `node utils/set-admin-password.js`

## API Highlights

- Auth: `POST /api/auth/request-otp`, `POST /api/auth/verify-otp`
- Products: `GET /api/products`, `GET /api/products/:id`
- Cart: `GET/POST/PUT/DELETE /api/cart`, `POST /api/cart/apply-coupon`
- Addresses: `GET/POST/PUT/DELETE /api/addresses`, `POST /api/addresses/:id/default`
- Orders: `GET/POST /api/orders`
- Coupons: `GET /api/coupons/available`, `GET /api/coupons/validate/:code`

## Development Notes

- Frontend is static and can be served by any static server; paths assume `http://localhost:8080`.
- Backend defaults to `http://localhost:5000` and reads configuration from `.env`.
- Avoid committing `.env` or any secrets.

## License

ISC (demo project)

## Deployment

- Backend
  - Set production env vars (`MONGODB_URI`, `JWT_SECRET`, `EMAIL_*`, `PORT`).
  - Start with `npm start`; ensure the process manager (PM2/systemd) restarts on failure.
  - Example MongoDB Atlas URI: `mongodb+srv://<user>:<pass>@cluster.mongodb.net/zylo-ecommerce`.
- Frontend
  - Serve statically from a CDN or web server (`/pages/*.html`).
  - Update `Frontend/js/api.js` `BASE_URL` to point to your deployed API, e.g. `https://api.yourdomain.com/api`.
  - Optionally use a reverse proxy so Frontend calls `/_api` mapped to your Backend.

## CI/CD

- Suggested checks on every push/PR:
  - `npm ci` in `Backend/`.
  - `node -e "require('fs').accessSync('server.js')"` sanity check.
  - Optional: run `node seeders/populate-ratings.js` against a staging DB.
- Example GitHub Actions workflow (Backend):

```yaml
name: backend-ci
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - name: Install
        working-directory: Backend
        run: npm ci
      - name: Lint/Type (optional)
        working-directory: Backend
        run: |
          echo "No linters configured"
      - name: Start for smoke test
        working-directory: Backend
        run: |
          node -e "require('fs').accessSync('server.js')"
```

## Contribution

- Fork and create feature branches; open PRs with clear descriptions.
- Follow existing code style and patterns in `Frontend/js/*` and `Backend/routes/*`.
- Do not commit secrets; use `.env` locally and CI secrets for pipelines.
- Prefer small, focused changes; include references to files and lines when discussing code.

## Endpoint Examples

- Auth
  - `POST /api/auth/request-otp`
    - Request: `{ "email": "user@example.com" }`
    - Response: `{ "success": true, "previewUrl": "<optional dev preview>" }`
  - `POST /api/auth/verify-otp`
    - Request: `{ "email": "user@example.com", "code": "123456", "name": "User" }`
    - Response: `{ "success": true, "token": "<jwt>", "user": { ... } }`
- Addresses
  - `GET /api/addresses` → list saved addresses
  - `POST /api/addresses` → create `{ firstName, lastName, phone, zip, line, area, city, state, country, label, isDefault }`
  - `POST /api/addresses/:id/default` → mark default
- Cart
  - `GET /api/cart` → current server cart and `appliedCoupon`
  - `POST /api/cart` → `{ productId, size, quantity }`
  - `POST /api/cart/coupon/apply` → `{ couponCode }` → returns `appliedCoupon`
- Coupons (public)
  - `GET /api/coupons/available`
  - `GET /api/coupons/validate/:code?orderValue=1234`

For a broader list, see `Frontend/js/api.js` which mirrors available endpoints.
