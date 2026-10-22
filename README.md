This project is a full-stack online food ordering and restaurant management web application. Customers browse restaurants and menus, place orders and pay online; restaurant owners run their own restaurant page, menu and incoming orders; administrators oversee the whole platform.


---

## Table of Contents

1. [Features](#features)
2. [Tech Stack](#tech-stack)
3. [Project Structure](#project-structure)
4. [Getting Started](#getting-started)
5. [Environment Variables](#environment-variables)
6. [Supabase Setup](#supabase-setup)
7. [VNPAY Payment (Sandbox)](#vnpay-payment-sandbox)
8. [Available Scripts](#available-scripts)
9. [Testing](#testing)
10. [Docker](#docker)
11. [Deployment](#deployment)
12. [Application Routes](#application-routes)
13. [Known Issues](#known-issues)
14. [Troubleshooting](#troubleshooting)
15. [Contributing](#contributing)
16. [Security Notes](#security-notes)
17. [Team](#team)

---

## Features

The app has three user roles, stored in the `role` column of the `profiles` table: `customer`, `restaurant_owner` and `admin`.

**Customers**

- Sign up (choosing the "customer" or "restaurant owner" role), confirm the e-mail address, log in and manage a personal profile, all through Supabase Auth.
- Browse restaurants on the home page and search by name or description.
- Open a restaurant page and view its menu, or browse menus by category: appetizers, main course, desserts and drinks.
- Add dishes to a cart (cart sidebar) and check out to place an order.
- Pay for a pending order online through the VNPAY sandbox gateway.
- Track orders on the "My orders" page, which updates live, and receive notifications when orders change (notification bell, powered by Supabase Realtime).

**Restaurant owners**

- Create a restaurant after signing up (name, description, address, contact details, logo and cover image).
- Edit restaurant details and upload images to Supabase Storage.
- Add, edit and remove menu items with images, prices and categories.
- View incoming orders in real time and update their status.
- Restaurant dashboard showing the restaurant's information with shortcuts to order and menu management.

**Administrators**

- Admin dashboard (access denied for non-admin users) with shortcuts to management pages.
- Manage users and their roles, menu items and orders across the platform.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Framework | [Next.js 14](https://nextjs.org/) (App Router), [React 18](https://react.dev/) |
| Styling | [Tailwind CSS 3](https://tailwindcss.com/), custom CSS, MDB React UI Kit, Font Awesome, Inter font |
| Forms & validation | React Hook Form, Zod |
| UI helpers | React Toastify, React Image Crop |
| Backend / Database | [Supabase](https://supabase.com/) (PostgreSQL, Auth, Storage, Realtime) via `@supabase/ssr` and `@supabase/supabase-js` |
| Payments | [VNPAY](https://sandbox.vnpayment.vn/) sandbox (HMAC-SHA512 signed requests) |
| Testing | [Cypress](https://www.cypress.io/) end-to-end tests |
| Containerisation & hosting | Docker (`node:20-alpine`), Docker Hub, Render |

---

## Project Structure

```text
HexaGrub/
├── app/
│   ├── page.jsx                 # Home page: restaurant list + search
│   ├── layout.jsx               # Root layout (AuthProvider, CartProvider)
│   ├── login/  signup/  logout/ # Authentication pages and server actions
│   ├── auth/confirm/            # Email-confirmation route handler
│   ├── profile/  orders/        # Customer profile and order history
│   ├── menu/                    # Global menu + appetizers/, main-course/, desserts/, drinks/
│   ├── restaurants/             # Restaurant listing
│   ├── restaurant/              # [id]/, create/, edit/, menu/, orders/, dashboard/ (owner area)
│   ├── admin/                   # dashboard/, users/, menu/, orders/ (admin area)
│   ├── Payment/                 # success/ and failed/ result pages
│   ├── api/payment/             # vnpay/ (create payment URL), result/ (VNPAY callback)
│   ├── components/              # Header, Sidebar, CartSidebar, CheckoutForm, NotificationBell, ...
│   ├── context/                 # AuthContext, CartContext
│   ├── services/                # menuService, orderService, restaurantService, vnpayService
│   └── scripts/                 # Sample-menu insert helpers (SQL + JS)
├── utils/supabase/              # Supabase browser client, server client, session middleware helper
├── db/
│   ├── migrations/              # SQL migrations (restaurant support, RLS)
│   └── sample_data.sql          # Sample menu items
├── cypress/                     # E2E tests (e2e/, fixtures/, support/)
├── middleware.js                # Refreshes the Supabase session on every request
├── restaurant.dockerfile        # Docker build for production
├── tailwind.config.js  postcss.config.js  next.config.mjs  jsconfig.json
└── package.json
```

The `@/` import alias points to the project root (see `jsconfig.json`).

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 18 or newer (20 is used in the Docker image) and npm
- [Git](https://git-scm.com/)
- A [Supabase](https://supabase.com/) project (or the credentials of the team's project)
- Optional: [Docker](https://www.docker.com/) for container builds

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repository>.git
cd <your-repository>
```

The team pushes updates straight to the `main` branch, so before running the app always sync with it:

```bash
git checkout main
git pull
```

### 2. Install dependencies

```bash
npm install
```

If you copied the project from someone else, or the install looks broken, delete the `node_modules` folder first and run `npm install` again for a clean install.

### 3. Create `.env.local`

Create a file named `.env.local` in the **project root** (next to `package.json`) and fill it in using the [Environment Variables](#environment-variables) section below. This file is git-ignored and must never be committed.

### 4. Run the development server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). Restart the server whenever you change `.env.local`.

---

## Environment Variables

Create `.env.local` with the following keys. **Ask a teammate or the project owner for the real values. Never paste real keys into the README, issues, commits or screenshots.**

```env
# Supabase (required)
NEXT_PUBLIC_SUPABASE_URL=https://<your-project-ref>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-supabase-anon-key>

# Supabase service role key (required for the payment callback route; server-side only)
SUPABASE_SERVICE_ROLE_KEY=<your-supabase-service-role-key>

# VNPAY sandbox (required for online payment)
VNPAY_TMN_CODE=<your-vnpay-terminal-code>
VNPAY_HASH_SECRET=<your-vnpay-hash-secret>
VNPAY_URL=https://sandbox.vnpayment.vn/paymentv2/vpcpay.html
VNPAY_RETURN_URL=http://localhost:3000/api/payment/result
```

| Variable | Used for | Exposed to browser? |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL (browser, server, middleware) | Yes |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Public (anon) Supabase key | Yes |
| `SUPABASE_SERVICE_ROLE_KEY` | Lets `/api/payment/result` update an order after payment. Bypasses Row Level Security | **No, keep secret** |
| `VNPAY_TMN_CODE` | VNPAY merchant / terminal code | No |
| `VNPAY_HASH_SECRET` | Secret used to sign and verify VNPAY requests | No |
| `VNPAY_URL` | VNPAY payment gateway URL (sandbox by default) | No |
| `VNPAY_RETURN_URL` | Callback URL VNPAY redirects to after payment. Use your deployed domain in production | No |

Without the Supabase variables the app cannot load restaurants or authenticate. Without the VNPAY variables everything except online payment still works.

---

## Supabase Setup

If you are creating your own Supabase project instead of using the shared one:

1. **Create a project** at [supabase.com](https://supabase.com/) and copy the project URL and API keys from *Project Settings → API* into `.env.local`.
2. **Create the base tables** in the Supabase SQL editor or table editor. The code expects these tables:

   | Table | Purpose |
   | --- | --- |
   | `profiles` | One row per auth user: `role` (`customer`, `restaurant_owner`, `admin`), `restaurant_id`, `phone_number`, `address` |
   | `restaurants` | Restaurant details, owned by an auth user through `owner_id` |
   | `menu_items` | Dishes: `restaurant_id`, `name`, `description`, `price`, `category`, `image_urls`, `is_available` |
   | `orders` | Orders with `status` (`pending`, `completed`), `restaurant_id`, `delivery_address`, `delivery_instructions`, `estimated_delivery_time` |
   | `order_items` | Line items of an order |
   | `menu_item_price_history` | Price change history for menu items |

3. **Run the migration** `db/migrations/001_add_restaurant_support.sql` in the SQL editor. It creates the `restaurants` table and adds restaurant-related columns and indexes to `menu_items`, `orders` and `profiles`. It expects the base tables, the `uuid_generate_v4()` extension and a `handle_updated_at()` trigger function to exist already. `002_disable_rls.sql` is currently an empty placeholder.
4. **Create the database function** `delete_order(order_id)`, which the orders pages call through `supabase.rpc('delete_order', ...)`.
5. **Create two public Storage buckets**: `restaurant-images` (logos and covers) and `menu-images` (dish photos).
6. **Enable Realtime** on the `orders` table so notifications and live order lists work.
7. **Configure Auth**: set the site URL and redirect URLs to `http://localhost:3000` (and your deployed URL), and point the confirmation e-mail template link to `{{ .SiteURL }}/auth/confirm?token_hash={{ .TokenHash }}&type=email`.
8. **Optional sample data**: run `db/sample_data.sql` to insert example dishes. `app/scripts/insertSampleMenu.sql` inserts dishes for one specific restaurant; replace the hard-coded restaurant UUID with your own first.

To make an admin, set `role = 'admin'` on the user's row in `profiles`.

---

## VNPAY Payment (Sandbox)

Online payment uses the VNPAY sandbox, so no real money is charged.

1. Put the `VNPAY_*` variables in `.env.local` (see above).
2. Place an order, open **My orders** and click the pay button on a pending order. The app calls `POST /api/payment/vnpay`, which builds a signed VNPAY payment URL and redirects the customer to the gateway.
3. Use the test cards published on the [VNPAY demo page](https://sandbox.vnpayment.vn/apis/vnpay-demo/) to complete the payment.
4. VNPAY redirects back to `GET /api/payment/result`. The route verifies the HMAC-SHA512 signature, and when the response code is `00` it marks the order `completed` and redirects to `/Payment/success`. Otherwise the user lands on `/Payment/failed`.

Notes:

- Prices are stored in USD and converted to VND with a fixed rate of `25,000` VND per USD (`app/services/vnpayService.js`). Update `USD_TO_VND_RATE` when needed.
- In production, set `VNPAY_RETURN_URL` to `https://<your-domain>/api/payment/result`.

---

## Available Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the development server on port 3000 |
| `npm run build` | Create an optimised production build |
| `npm run start` | Start the production server (run `build` first) |
| `npm run lint` | Run Next.js linting |
| `npx cypress open` | Open the Cypress test runner (dev server must be running) |
| `npx cypress run` | Run all Cypress tests headlessly |

---

## Testing

End-to-end tests live in `cypress/e2e/`. Currently `login.cy.js` covers the login form (form is visible, empty-field validation, invalid credentials, successful login).

```bash
# terminal 1
npm run dev

# terminal 2
npx cypress open
```

Before running the "successful login" test, replace the hard-coded e-mail and password in `cypress/e2e/login.cy.js` with a test account that exists in your Supabase project.

---

## Docker

A production image can be built with the included `restaurant.dockerfile` (Node 20 Alpine, runs `npm install`, `npm run build` and `npm start`).

```bash
# Build
docker build -t restaurant-app -f restaurant.dockerfile .

# Run on port 3000
docker run -p 3000:3000 restaurant-app
```

Then open [http://localhost:3000](http://localhost:3000).

Important:

- `NEXT_PUBLIC_*` variables are compiled into the app at **build time**, so `.env.local` must be present in the project folder when you run `docker build`, or the Dockerfile must be extended with `ARG`/`ENV` lines for them.
- The repository has no `.dockerignore`, so `COPY . .` also copies `.env.local` and `node_modules` into the image. **Never push an image built with real secrets to a public registry.** Adding a `.dockerignore` containing `node_modules`, `.next` and `.env*` is recommended.

The published image is available on Docker Hub: [hub.docker.com/r/10422105/hexagrub](https://hub.docker.com/r/10422105/hexagrub).

---

## Deployment

The production site is hosted on [Render](https://render.com/): [hexagrub.onrender.com](https://hexagrub.onrender.com/).

To deploy your own copy on Render as a Web Service connected to your GitHub repository:

- Build command: `npm install && npm run build`
- Start command: `npm start`
- Environment variables: add every variable from [Environment Variables](#environment-variables) in the Render dashboard, with `VNPAY_RETURN_URL` set to `https://<your-render-domain>/api/payment/result`.
- Add the Render URL to the allowed redirect URLs in your Supabase Auth settings.

On the free Render tier the service sleeps when idle, so the first request after a pause can take a while.

---

## Application Routes

| Route | Description | Access |
| --- | --- | --- |
| `/` | Home: restaurants and search | Everyone |
| `/login`, `/signup` | Authentication | Everyone |
| `/restaurants`, `/restaurant/[id]` | Restaurant list and details | Everyone |
| `/menu`, `/menu/appetizers`, `/menu/main-course`, `/menu/desserts`, `/menu/drinks` | Menu by category | Everyone |
| `/profile` | User profile | Redirects to `/login` when logged out |
| `/orders` | Order history and payment | Redirects to `/login` when logged out |
| `/Payment/success`, `/Payment/failed` | Payment result pages | Reached after VNPAY redirect |
| `/restaurant/create`, `/restaurant/edit`, `/restaurant/menu`, `/restaurant/orders`, `/restaurant/dashboard` | Restaurant owner area | Restaurant owners (redirects to `/login` when logged out) |
| `/admin/dashboard`, `/admin/users`, `/admin/menu`, `/admin/orders` | Admin area | Admins (redirects to `/login` when logged out) |
| `POST /api/payment/vnpay` | Create VNPAY payment URL | API |
| `GET /api/payment/result` | VNPAY return/callback handler | API |
| `GET /auth/confirm` | Supabase e-mail confirmation | API |

---

## Known Issues

These are open problems reported by the team while testing:

1. **Deleted accounts can still log in.** When a customer or restaurant-owner account is deleted from the Supabase database, that user can still log in normally.
2. **Deleting an owner does not delete the restaurant.** If a restaurant-owner account is deleted (and the same e-mail is later reused), the restaurant is not removed: customers can still order from it and the owner can still open the restaurant dashboard.
3. **Deleting a restaurant in Supabase does not lock the dashboard.** After the restaurant row is removed in Supabase, the restaurant dashboard is still reachable.
4. **Invisible text in forms.** Occasionally the e-mail and password text on the login and register pages appears invisible (looks like blank space).

Everything else in the core flow works as expected: placing orders, viewing orders, order notifications, viewing menus and editing menus.

Suggested follow-ups: use `ON DELETE CASCADE` (or explicit clean-up) between `auth.users`, `profiles` and `restaurants`; validate the profile and restaurant on the server for each protected page instead of trusting the cached session; and set an explicit text colour on the form inputs.

---

## Troubleshooting

**`Module not found: Can't resolve 'cookie'`**

Run `npm install` again. If the error remains, install the packages with Yarn:

```bash
npm i yarn -g
yarn add cookie-parser
```

**Nothing loads / Supabase errors in the console**

Check that `.env.local` exists in the project root, the variable names are spelled exactly as above, and you restarted `npm run dev` after editing it.

**Port 3000 already in use**

Stop the other process or start on another port: `npm run dev -- -p 3001` (and update `VNPAY_RETURN_URL` accordingly).

**Payment succeeds but the order is not updated**

Make sure `SUPABASE_SERVICE_ROLE_KEY` is set and that `VNPAY_RETURN_URL` matches the address the app is actually running on.

---

## Contributing

1. Make sure you are on an up-to-date branch: `git checkout main && git pull`.
2. Create a feature branch: `git checkout -b feature/<short-name>`.
3. Make your changes and run `npm run lint`.
4. Commit with a clear message and push: `git push origin feature/<short-name>`.
5. Open a Pull Request describing what changed and how you tested it.

For questions or bug reports, open an issue in this repository or contact the maintainers.

---

## Security Notes

- **Never commit** `.env.local` or any file containing real keys. `.gitignore` already excludes `.env*.local` and `node_modules/`.
- `SUPABASE_SERVICE_ROLE_KEY` bypasses Row Level Security. Use it only in server-side code (API routes) and never expose it in client components.
- If a key has ever been posted publicly (README, chat screenshot, shared document that became public), **rotate it** in the Supabase / VNPAY dashboards.
- The browser Supabase client currently logs auth session details to the console (`utils/supabase/client.js`). Remove those `console.log` calls before a real production launch.

