# MERN Shop — Full-Featured eCommerce Platform

> A production-ready eCommerce platform built with the **MERN stack** (MongoDB, Express.js, React, Node.js). Ships with a polished storefront, JWT cookie authentication, admin dashboard, product management, order lifecycle, reviews, search, pagination, and a pure **Cash on Delivery** checkout.

[![Node.js](https://img.shields.io/badge/Node.js-22.x-339933?style=flat&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=flat&logo=react&logoColor=black)](https://react.dev/)
[![Express](https://img.shields.io/badge/Express-4.x-000000?style=flat&logo=express&logoColor=white)](https://expressjs.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose_8-47A248?style=flat&logo=mongodb&logoColor=white)](https://mongoosejs.com/)
[![Redux Toolkit](https://img.shields.io/badge/Redux_Toolkit-RTK_Query-764ABC?style=flat&logo=redux&logoColor=white)](https://redux-toolkit.js.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Live Demo:** [https://mern-shop-abxs.onrender.com/](https://mern-shop-abxs.onrender.com/)

> **Note on hosting:** The demo runs on Render's free tier. It sleeps after 15 minutes of inactivity — the first request after wake-up may take ~30–50s. Subsequent requests are fast.

---

## Table of Contents

- [Overview](#overview)
- [Highlights](#highlights)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Data Models](#data-models)
- [API Reference](#api-reference)
- [Frontend — Pages & Routing](#frontend--pages--routing)
- [State Management](#state-management)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Available Scripts](#available-scripts)
- [Database Seeding](#database-seeding)
- [Authentication & Authorization](#authentication--authorization)
- [Key Features Deep Dive](#key-features-deep-dive)
- [Admin Dashboard](#admin-dashboard)
- [Deployment](#deployment)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

MERN Shop is a complete online shopping experience with separate **customer** and **admin** surfaces. Customers can browse, search, review, cart, and place COD orders. Admins manage catalog, users, orders, and platform analytics from a dedicated dashboard.

It is designed to be:

- **Real-world** — pagination, search, auth, image upload, order lifecycle, role guards
- **Secure** — HTTP-only JWT cookies, `express-validator`, bcrypt hashing, protected routes
- **Maintainable** — clear `backend/` + `frontend/` separation, RTK Query, consistent error handling, reusable components
- **Deployable** — single-service production build (`frontend/build` served by Express) and `render.yaml` included

---

## Highlights

### Shopper Experience
- Full shopping cart (add / update qty / remove, persisted in `localStorage` via Redux)
- Product search with debounced query (`?search=`) and server-side regex filtering
- Pagination (`?limit=` & `?skip=`) with `PAGINATION_MAX_LIMIT` guard
- Top-products carousel (top 3 by rating)
- Product detail page with stock indicator, rating component, and review submission (one review per user)
- Shipping address step + COD-only checkout
- Order confirmation and order history (`My Orders`)
- User profile with editable name/email/password and order list

### Admin Experience
- Dedicated admin auth surface at `/admin/login` and guarded `/admin/*` layout (`AdminDashboard.js`)
- Dashboard with analytics: order price charts, product price charts, summary cards (Recharts)
- Product CRUD — create, edit, delete, upload image via Multer (`/api/v1/upload`)
- User management — list, view, update (including `isAdmin` toggle), delete
- Admin management — list and manage admin accounts
- Order management — list all orders, view details, mark as **Delivered**

### Platform
- Cash on Delivery only — no payment gateway complexity
- Image upload serving from `/uploads` (static) with file cleanup on update/delete
- Centralized error handling (`notFound`, `errorHandler`)
- Request validation with `express-validator` + `validateRequest` middleware
- Response compression and CORS with credentials

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Frontend** | React 18, React Router 6 | SPA, client-side routing |
| **State** | Redux Toolkit + RTK Query, `cartSlice`/`authSlice` | Global state, caching, mutations |
| **UI** | React-Bootstrap 5, Bootstrap 5, React Icons, React Toastify | Layout, notifications |
| **Charts** | Recharts | Admin analytics |
| **HTTP** | Axios (via RTK Query `fetchBaseQuery`) | API communication |
| **Backend** | Node.js 22, Express 4 | REST API |
| **Database** | MongoDB + Mongoose 8 | Persistence, schemas |
| **Auth** | JWT (`jsonwebtoken`), bcrypt 5, HTTP-only cookies | Authentication |
| **Validation** | express-validator 7 | Request validation |
| **Upload** | Multer | Product image uploads |
| **Infra** | dotenv, cookie-parser, cors, compression | Config & middleware |
| **Deploy** | Render (Node runtime), `render.yaml` | Hosting |

---

## Architecture

```
                ┌─────────────────────────────────┐
                │         React 18 (SPA)           │
                │  React Router 6 + RTK Query      │
                │  Redux: cart / auth / search      │
                └──────────────┬──────────────────┘
                               │  /api/v1/*
                               ▼
                ┌─────────────────────────────────┐
                │        Express 4 (API)           │
                │  routes → validator → protect    │
                │  → admin → controller → Mongoose │
                └──────────────┬──────────────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
           MongoDB          /uploads      JWT Cookie
          (Atlas)        (Multer static) (httpOnly)
```

**Request lifecycle:**

1. Frontend dispatches RTK Query hook (`productsApiSlice`, `ordersApiSlice`, etc.) → `fetchBaseQuery` with `credentials: include`.
2. Express validates input via `express-validator` → `validateRequest`.
3. `protect` middleware verifies `req.cookies.jwt`; `admin` checks `req.user.isAdmin`.
4. Controller interacts with Mongoose models; errors bubble to `errorHandler`.
5. In production, Express serves `frontend/build` and falls back to `index.html` for SPA routing (`backend/server.js:43-50`).

---

## Project Structure

```
week-4/
├── backend/
│   ├── config/
│   │   └── db.js                 # Mongoose connection
│   ├── controllers/
│   │   ├── productController.js  # getProducts, getTopProducts, CRUD, reviews
│   │   ├── userController.js     # login, register, profile, admin user mgmt
│   │   └── orderController.js    # addOrderItems, getOrders, deliver
│   ├── data/
│   │   ├── products.js           # 11 sample gaming laptops
│   │   └── users.js              # 3 seed users (1 admin + 2 customers)
│   ├── middleware/
│   │   ├── authMiddleware.js     # protect (JWT cookie), admin
│   │   ├── errorMiddleware.js    # notFound, errorHandler
│   │   └── validator.js          # validateRequest (express-validator)
│   ├── models/
│   │   ├── productModel.js       # Product + embedded reviewSchema
│   │   ├── userModel.js          # User (isAdmin, timestamps)
│   │   └── orderModel.js         # Order (items, shipping, COD, delivery)
│   ├── routes/
│   │   ├── productRoutes.js
│   │   ├── userRoutes.js
│   │   ├── orderRoutes.js
│   │   └── uploadRoutes.js       # Multer single-file upload
│   ├── utils/
│   │   ├── generateToken.js      # JWT + httpOnly cookie (remember-me aware)
│   │   └── file.js               # deleteFile helper
│   ├── seeder.js                 # import / destroy seed data
│   └── server.js                 # App entry: middleware, routes, static serve
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Header.jsx, Footer.jsx, Loader.jsx, Message.jsx
│   │   │   ├── Product.jsx, Paginate.jsx, SearchBox.jsx
│   │   │   ├── ProductCarousel.jsx, Rating.jsx, Reviews.jsx
│   │   │   ├── CheckoutSteps.jsx, PrivateRoute.jsx, AdminRoute.jsx
│   │   │   └── Admin/            # AdminHeader, AdminSidebar, DashboardCard, charts
│   │   ├── pages/
│   │   │   ├── HomePage.jsx, ProductPage.jsx, CartPage.jsx
│   │   │   ├── LoginPage.jsx, RegisterPage.jsx
│   │   │   ├── ShippingPage.jsx, PlaceOrderPage.jsx, OrderDetailsPage.jsx
│   │   │   ├── ProfilePage.jsx, NotFoundPage.jsx
│   │   │   └── admin/            # Dashboard, ProductList/Form, UserList, OrderList, etc.
│   │   ├── slices/
│   │   │   ├── apiSlice.js       # baseApi (fetchBaseQuery, BASE_URL="")
│   │   │   ├── authSlice.js      # persisted auth state (localStorage)
│   │   │   ├── cartSlice.js      # cart items, shipping, prices
│   │   │   ├── productsApiSlice.js
│   │   │   ├── ordersApiSlice.js
│   │   │   ├── usersApiSlice.js
│   │   │   └── searchProductSlice.js
│   │   ├── routes/
│   │   │   └── Routes.jsx        # createBrowserRouter tree
│   │   ├── utils/
│   │   │   ├── cartUtils.js
│   │   │   └── addCurrency.js
│   │   ├── App.js                # Header + Outlet + Footer + ToastContainer
│   │   ├── AdminDashboard.js     # Admin layout shell
│   │   ├── store.js              # configureStore: apiSlice + cart + auth + search
│   │   └── constants.js          # BASE_URL, PRODUCTS_URL, USERS_URL, etc.
│   └── package.json              # react-scripts 5, proxy → localhost:5000
├── uploads/                      # Served statically at /uploads
├── .env.example
├── render.yaml                   # Render service definition
└── package.json                  # Root scripts: dev, server, client, data:*
```

---

## Data Models

### User — `backend/models/userModel.js:4-30`

| Field | Type | Notes |
|-------|------|-------|
| `name` | String | required |
| `email` | String | required, unique |
| `password` | String | required, bcrypt-hashed in controller |
| `isAdmin` | Boolean | default `false` |
| `timestamps` | Date | `createdAt`, `updatedAt` |

### Product — `backend/models/productModel.js:31-93`

| Field | Type | Notes |
|-------|------|-------|
| `user` | ObjectId → User | creator (admin) |
| `name` | String | required |
| `image` | String | required, path under `/uploads` |
| `description` | String | required |
| `brand` | String | required |
| `category` | String | required |
| `price` | Number | required, default 0 |
| `countInStock` | Number | required, default 0 |
| `rating` | Number | default 0, recomputed on review |
| `numReviews` | Number | default 0 |
| `reviews` | Array | embedded `reviewSchema` (user, name, rating, comment, timestamps) |

### Order — `backend/models/orderModel.js:4-52`

| Field | Type | Notes |
|-------|------|-------|
| `user` | ObjectId → User | required |
| `orderItems` | Array | `{ name, qty, image, price, product: ObjectId→Product }` |
| `shippingAddress` | Object | `{ address, city, postalCode, country }` |
| `paymentMethod` | String | default `Cash on Delivery` |
| `itemsPrice`, `taxPrice`, `shippingPrice`, `totalPrice` | Number | required |
| `isDelivered` | Boolean | default `false` |
| `deliveredAt` | Date | set when `isDelivered → true` |
| `timestamps` | Date | `createdAt`, `updatedAt` |

---

## API Reference

Base prefix: `/api/v1` — defined in `backend/server.js:38-41` and `frontend/src/constants.js:3-7`

### Products — `backend/routes/productRoutes.js`

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `GET` | `/api/v1/products?limit=&skip=&search=` | Public | List products with pagination + search (regex, case-insensitive). Enforces `PAGINATION_MAX_LIMIT`. |
| `GET` | `/api/v1/products/top` | Public | Top 3 products by `rating` desc |
| `GET` | `/api/v1/products/:id` | Public | Single product by ID |
| `POST` | `/api/v1/products` | Private/Admin | Create product |
| `PUT` | `/api/v1/products/:id` | Private/Admin | Update product (deletes old image if changed) |
| `DELETE` | `/api/v1/products/:id` | Private/Admin | Delete product + uploaded file |
| `POST` | `/api/v1/products/reviews/:id` | Private | Add review (one per user; recomputes `rating`/`numReviews`) |

Validation is applied to all mutating and ID routes via `express-validator` in `backend/routes/productRoutes.js:17-83`.

### Users — `backend/routes/userRoutes.js`

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `POST` | `/api/v1/users` | Public | Register new user |
| `POST` | `/api/v1/users/login` | Public | Login (sets `jwt` httpOnly cookie) |
| `POST` | `/api/v1/users/logout` | Private | Logout (clears cookie) |
| `GET` | `/api/v1/users/profile` | Private | Get own profile |
| `PUT` | `/api/v1/users/profile` | Private | Update own profile |
| `GET` | `/api/v1/users` | Private/Admin | List all users |
| `GET` | `/api/v1/users/admins` | Private/Admin | List admin users |
| `GET` | `/api/v1/users/:id` | Private/Admin | Get user by ID |
| `PUT` | `/api/v1/users/:id` | Private/Admin | Update user (name, email, isAdmin) |
| `DELETE` | `/api/v1/users/:id` | Private/Admin | Delete user |

### Orders — `backend/routes/orderRoutes.js`

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `POST` | `/api/v1/orders` | Private | Create order (validates `cartItems`, `shippingAddress`, price fields) |
| `GET` | `/api/v1/orders` | Private/Admin | List all orders |
| `GET` | `/api/v1/orders/my-orders` | Private | List current user's orders |
| `GET` | `/api/v1/orders/:id` | Private | Get order by ID (owner or admin) |
| `PUT` | `/api/v1/orders/:id/deliver` | Private/Admin | Mark order as delivered |

### Upload — `backend/routes/uploadRoutes.js`

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `POST` | `/api/v1/upload` | Private/Admin | Upload single image via Multer; returns stored path; served from `/uploads` |

> All protected routes use `protect` (`backend/middleware/authMiddleware.js:5-27`) which reads `req.cookies.jwt` and verifies with `JWT_SECRET`. Admin routes additionally use `admin` (`backend/middleware/authMiddleware.js:30-40`).

---

## Frontend — Pages & Routing

Router is defined in `frontend/src/routes/Routes.jsx` using `createBrowserRouter`.

### Public Routes (inside `<App />`)

| Path | Component | Description |
|------|-----------|-------------|
| `/` | `HomePage` | Product grid + search + pagination + featured carousel |
| `/product/:id` | `ProductPage` | Details, stock, reviews, add-to-cart |
| `/cart` | `CartPage` | Cart management |
| `/login` | `LoginPage` | Customer login |
| `/register` | `RegisterPage` | Customer registration |

### Private Routes (guarded by `PrivateRoute.jsx`)

| Path | Component | Description |
|------|-----------|-------------|
| `/shipping` | `ShippingPage` | Shipping address form (persists to cart slice) |
| `/place-order` | `PlaceOrderPage` | Order summary + place order (COD) |
| `/order/:id` | `OrderDetailsPage` | Order status, items, delivery info |
| `/profile` | `ProfilePage` | Edit profile + my orders |

### Admin Routes (guarded by `AdminRoute.jsx` inside `AdminDashboard` layout)

| Path | Component | Description |
|------|-----------|-------------|
| `/admin/login` | `AdminLoginPage` | Separate admin login (same API, admin check on redirect) |
| `/admin/dashboard` | `Dashboard` | Analytics, summary cards, charts |
| `/admin/product-list` | `ProductListPage` | Table + create/edit/delete |
| `/admin/product/create` | `ProductFormPage` | Create form + image upload |
| `/admin/product/update/:id` | `ProductFormPage` | Edit form (reuses same component) |
| `/admin/order-list` | `OrderListPage` | All orders |
| `/admin/order/:id` | `OrderDetailsPage` | Reused with admin deliver action |
| `/admin/user-list` | `UserListPage` | All users |
| `/admin/admin-list` | `AdminListPage` | Admin-only list |
| `/admin/user/update/:id` | `UpdateUserFormPage` | Edit user / toggle admin |
| `/admin/profile` | `AdminProfilePage` | Admin profile |

Catch-all `*` renders `NotFoundPage`.

**Layout shells:**

- `frontend/src/App.js:9-22` — public shell: `Header` + `Outlet` + `Footer` + `ToastContainer`
- `frontend/src/AdminDashboard.js` — admin shell: `AdminHeader` + `AdminSidebar` + `Outlet`

---

## State Management

Store is configured in `frontend/src/store.js:9-22`:

```js
configureStore({
  reducer: {
    [apiSlice.reducerPath]: apiSlice.reducer,
    cart: cartSliceReducer,
    auth: authSliceReducer,
    search: searchProductSliceReducer,
  },
  middleware: (getDefault) => getDefault().concat(apiSlice.middleware),
});
```

| Slice | File | Responsibility |
|-------|------|----------------|
| `apiSlice` | `slices/apiSlice.js` | RTK Query base (`fetchBaseQuery` with `BASE_URL=""`, `credentials: include`) |
| `productsApiSlice` | `slices/productsApiSlice.js` | `getProducts`, `getProduct`, `getTopProducts`, `create/update/delete`, `createReview` |
| `ordersApiSlice` | `slices/ordersApiSlice.js` | `createOrder`, `getOrder`, `getMyOrders`, `getOrders`, `deliverOrder` |
| `usersApiSlice` | `slices/usersApiSlice.js` | `login`, `register`, `logout`, `profile`, admin user ops |
| `auth` | `slices/authSlice.js` | Persisted userInfo in `localStorage`, login/logout reducers |
| `cart` | `slices/cartSlice.js` | Items, quantities, shippingAddress, price calculations (`cartUtils.js`) |
| `search` | `slices/searchProductSlice.js` | Global search query state (consumed by `SearchBox.jsx` + `HomePage`) |

---

## Getting Started

### Prerequisites

- **Node.js** `22.x` (see `package.json:18-20` engines field)
- **npm** `>= 9`
- **MongoDB** — local instance or [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) cluster
- **Git**

Verify:

```bash
node -v   # should be v22.x
npm -v
```

### 1. Clone & Install

```bash
# fork first on GitHub, then:
git clone https://github.com/<your-username>/MERN-eCommerce.git
cd MERN-eCommerce

# install root + backend deps
npm install

# install frontend deps
cd frontend
npm install
cd ..
```

> The project keeps a `frontend/package.json:proxy` pointing to `http://localhost:5000` for dev CORS-free API calls.

### 2. Configure Environment

Create `.env` in the project root (see `.env.example`):

```bash
cp .env.example .env
```

Edit `.env`:

```dotenv
NODE_ENV=development
PORT=5000
JWT_SECRET=your_strong_random_secret_here   # e.g. openssl rand -base64 32
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/mern-shop?retryWrites=true&w=majority
PAGINATION_MAX_LIMIT=12
```

| Variable | Required | Description |
|----------|----------|-------------|
| `NODE_ENV` | yes | `development` or `production` — controls static serving & cookie `secure` flag |
| `PORT` | no | Defaults to `5000` (`backend/server.js:16`) |
| `JWT_SECRET` | **yes** | Secret for `jsonwebtoken` sign/verify. Keep private. |
| `MONGO_URI` | **yes** | MongoDB connection string |
| `PAGINATION_MAX_LIMIT` | no | Max products per page; defaults enforced in `productController.js:11` |

Generate a strong secret:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

### 3. Seed the Database (optional but recommended)

```bash
# import sample users + 11 products
npm run data:import

# to wipe everything
npm run data:destroy
```

Seeded users are defined in `backend/data/users.js`; products in `backend/data/products.js`.

### 4. Run

**Concurrent (recommended for development):**

```bash
npm run dev
# runs: nodemon backend/server.js  +  react-scripts start
# frontend → http://localhost:3000
# backend  → http://localhost:5000
```

**Backend only:**

```bash
npm run server
```

**Frontend only:**

```bash
npm run client
# or
cd frontend && npm start
```

---

## Environment Variables

Reference `.env.example:1-5`:

```dotenv
NODE_ENV=development
PORT=5000
JWT_SECRET=ADD_YOUR_JWT_SECRET_HERE
MONGO_URI=ADD_YOUR_MONGO_URI_HERE
PAGINATION_MAX_LIMIT=12
```

---

## Available Scripts

### Root `package.json:7-14`

| Script | Command | Description |
|--------|---------|-------------|
| `npm start` | `node backend/server.js` | Production start (serves `frontend/build` if `NODE_ENV=production`) |
| `npm run server` | `nodemon backend/server.js` | Dev server with auto-reload |
| `npm run client` | `npm start --prefix frontend` | Frontend dev server only |
| `npm run dev` | `concurrently "npm run server" "npm run client"` | Full-stack concurrent dev |
| `npm run data:import` | `node backend/seeder.js` | Seed DB |
| `npm run data:destroy` | `node backend/seeder.js -d` | Destroy all data |
| `npm run build` | `npm install && npm install --prefix frontend && npm run build --prefix frontend` | Production build (used by Render) |

### Frontend `frontend/package.json:28-32`

| Script | Description |
|--------|-------------|
| `npm start` | `react-scripts start` on port 3000 |
| `npm run build` | Production build to `frontend/build` |
| `npm test` | `react-scripts test` |
| `npm run eject` | Eject CRA config |

---

## Database Seeding

`backend/seeder.js` handles both paths:

- **`npm run data:import`** — deletes `Order`, `User`, `Product`; inserts 3 users (first is admin); attaches `adminUser` as `user` on all 11 products; logs `Data Imported!`.
- **`npm run data:destroy`** (`-d` flag) — deletes all three collections; logs `Data Destroyed!`.

Re-seed anytime after wiping or schema changes.

---

## Authentication & Authorization

- **Registration/Login** — `backend/controllers/userController.js` hashes passwords with `bcrypt`, calls `generateToken` (`backend/utils/generateToken.js:3-16`) which signs `{ userId }` with `JWT_SECRET` and sets an HTTP-only cookie:

  ```js
  res.cookie('jwt', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV !== 'development',
    sameSite: 'strict',
    maxAge: remember ? 365*24*60*60*1000 : 24*60*60*1000
  });
  ```

- **Remember me** — if `req.body.remember` is truthy, token and cookie TTL extend to 1 year; otherwise 24h.

- **Protect** — `backend/middleware/authMiddleware.js:5-27` reads `req.cookies.jwt`, verifies via `jsonwebtoken`, loads `req.user` (without password). Returns `401` if missing/invalid.

- **Admin** — `backend/middleware/authMiddleware.js:30-40` checks `req.user.isAdmin`; returns `401` if not admin.

- **Frontend guards** — `PrivateRoute.jsx` redirects unauthenticated users to `/login`; `AdminRoute.jsx` redirects non-admins away from `/admin/*`.

- **Logout** — `POST /api/v1/users/logout` clears the `jwt` cookie.

---

## Key Features Deep Dive

### Search & Pagination

- `GET /api/v1/products?search=&limit=&skip=` — `productController.js:8-37` uses `$regex` with `i` flag on `name`, clamps `limit` to `PAGINATION_MAX_LIMIT`, clamps `skip` to `maxSkip`. Returns `{ products, total, maxLimit, maxSkip }`. Frontend `Paginate.jsx` and `SearchBox.jsx` consume this. The global search term lives in `searchProductSlice`, so navigation preserves the query.

### Reviews

- `POST /api/v1/products/reviews/:id` — `productController.js:171-212` rejects duplicate reviews (lookup by `review.user._id`), appends `{ user, name, rating, comment }`, recomputes `rating` as average and `numReviews`. Frontend `Reviews.jsx` + `ProductPage.jsx` handle submission via `productsApiSlice`.

### Cart & Prices

- `cartSlice.js` + `utils/cartUtils.js` persist cart in `localStorage`, compute `itemsPrice`, `taxPrice` (if applicable), `shippingPrice`, `totalPrice`. `CheckoutSteps.jsx` visualizes the flow: Cart → Shipping → Place Order.

### Image Upload

- `uploadRoutes.js` uses `multer` (disk storage to `uploads/`). Returned path is stored as `product.image` and served statically by `server.js:36` (`/uploads` → `express.static`). On update/delete, `utils/file.js:deleteFile` cleans up the old file (`productController.js:122-137`, `159`).

### Order Lifecycle

- Customer: `ShippingPage` → `PlaceOrderPage` → `POST /api/v1/orders` → redirect to `/order/:id`. Status shows `isDelivered` / `deliveredAt`. `ProfilePage` lists `my-orders`.
- Admin: `OrderListPage` → `OrderDetailsPage` → **Mark as Delivered** (`PUT /api/v1/orders/:id/deliver`) sets `isDelivered=true`, `deliveredAt=Date.now()`.

---

## Admin Dashboard

Access at `/admin/login` (separate route tree in `Routes.jsx:79-134` under `AdminDashboard` layout).

| Capability | Where | Notes |
|------------|-------|-------|
| View analytics | `/admin/dashboard` | `Dashboard.jsx` + `DashboardCard` + `OrderPriceChart` + `ProductPriceChart` (Recharts) |
| Manage products | `/admin/product-list` | Table with edit/delete; create via `/admin/product/create` |
| Create / edit product | `ProductFormPage.jsx` | Validated fields + image upload; reuses same component for create vs update |
| Manage orders | `/admin/order-list` | List + detail + mark delivered |
| Manage users | `/admin/user-list` | List, update (`UpdateUserFormPage`), delete |
| Manage admins | `/admin/admin-list` | Admin-only listing (`GET /api/v1/users/admins`) |
| Admin profile | `/admin/profile` | Self-service profile |

All admin pages are wrapped in `<AdminRoute />` which checks Redux `auth.userInfo.isAdmin`.

---

## Deployment

### Render (included)

`render.yaml:1-22` defines a single `web` service:

```yaml
services:
  - type: web
    name: mern-shop
    runtime: node
    plan: free
    buildCommand: npm install && npm install --prefix frontend && npm run build --prefix frontend
    startCommand: npm start
    envVars:
      - { key: NODE_ENV, value: production }
      - { key: NODE_VERSION, value: 22.15.0 }
      - { key: PORT, value: 5000 }
      - { key: JWT_SECRET, sync: false }
      - { key: MONGO_URI, sync: false }
      - { key: PAGINATION_MAX_LIMIT, value: 12 }
```

**Manual deploy steps:**

```bash
cd frontend && npm run build   # or npm run build at root (runs both installs + build)
NODE_ENV=production npm start  # Express serves frontend/build + API on same port
```

Set `JWT_SECRET`, `MONGO_URI` as secret env vars in the Render dashboard; `PAGINATION_MAX_LIMIT` can be tuned per environment.

### Any Node host / Docker

The app is a standard Express server. Ensure:

1. `frontend/build` exists (run build step).
2. Env vars are set.
3. Reverse proxy forwards to `PORT`.
4. `uploads/` is writable and persistent (or migrate to S3/Cloudinary for ephemeral filesystems).

---

## Security

- Passwords hashed with `bcrypt` (10+ salt rounds) before save; never returned (`select('-password')` in `authMiddleware.js:21`).
- JWT stored in **HTTP-only, SameSite=strict** cookie — not in `localStorage` — mitigating XSS exfiltration.
- `secure: true` in production (`generateToken.js:12`).
- Input validated with `express-validator` + centralized `validateRequest` on every mutating route.
- CORS configured with `credentials: true` (`server.js:24-29`); in production same-origin rendering avoids cross-origin issues entirely.
- `compression` enabled; generic `errorHandler` avoids leaking stack traces in production.
- **Hardening ideas (contributions welcome):** rate limiting on `/login`, helmet headers, CSRF double-submit for cookie auth, upload MIME/size validation, move uploads to object storage.

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `MongoServerError` / `MongooseServerSelectionError` | Wrong `MONGO_URI` or Atlas IP not allowlisted | Verify URI, allow `0.0.0.0/0` in Atlas Network Access for dev, check DB user password |
| `Authentication failed: Token not provided` (401) | Missing cookie / `credentials: include` | Ensure frontend `fetchBaseQuery` uses `credentials: include` and cookies aren't blocked |
| `Validation failed` on product create | Missing required fields (`image`, `price`, etc.) | Check request body matches `productRoutes.js:29-48` validator |
| Products `404 Products not found!` | Empty collection | Run `npm run data:import` |
| Images 404 | `uploads/` not served or file deleted | Confirm `server.js:36` static mount and that file exists on disk |
| Port conflict `EADDRINUSE` | Two servers on same `PORT` | Change `PORT` in `.env` or kill process `lsof -i :5000` |
| Frontend blank after build | `NODE_ENV !== production` or missing build | Run `npm run build` and start with `NODE_ENV=production` |
| Render cold start delay | Free tier sleep | Wait ~30–50s for first request; consider upgrading plan for always-on |

---

## Sample Credentials

Seeded via `backend/data/users.js` — available after `npm run data:import`:

**Admin**

- URL: `/admin/login` (also works on `/login` — dashboard redirect checks `isAdmin`)
- Email: `admin@admin.com`
- Password: `admin123`

**Customers**

- `john@email.com` / `john123` (John Doe)
- `alice@email.com` / `alice123` (Alice Smith)

> Change these immediately if you deploy publicly and re-seed with your own data.

---

## Contributing

Contributions are welcome — whether code, design, docs, or bug reports.

1. **Fork** the repository.
2. **Clone** your fork:

   ```bash
   git clone https://github.com/<your-username>/MERN-eCommerce.git
   cd MERN-eCommerce
   ```

3. **Branch**:

   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/issue-name
   ```

4. **Make changes** — follow existing style (ES modules, `async/await`, `express-validator` patterns, RTK Query conventions).
5. **Test** thoroughly — run `npm run dev`, exercise cart → checkout → admin flows; add tests if possible (`frontend/src/setupTests.js`).
6. **Commit** with a clear message:

   ```bash
   git add .
   git commit -m "feat: add wishlist support"
   ```

7. **Push** and open a **Pull Request** against `main` — include a clear title, description, and screenshots for UI changes.

**Code review** — maintainers will review for correctness, security, and consistency. Please address feedback promptly.

See also: GitHub Issues and Discussions for questions and feature requests.

---

## License

MIT — see `LICENSE` (author: Ajay Boro). You are free to use, modify, and distribute this project with attribution.

---

<p align="center">
  Built with ❤️ using the MERN stack &nbsp;·&nbsp; Happy coding!
</p>
