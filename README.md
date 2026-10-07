# E-Commerce App using C++ Crow — Web Edition

A minimalist, black-and-white e-commerce storefront for everyday accessories. It has product browsing, search and category filters, user accounts, and a persistent shopping cart.

> **This is the Web edition.** The project also has a Mobile edition (React mobile-style UI, C++ Crow backend and a Kotlin Android client) in a companion repository: [E-Commerce-APP-using-CPP-CROW](https://github.com/gamerpixel387-crypto/E-Commerce-APP-using-CPP-CROW).

The repository contains two backends for the same API:

| Backend | Location | Stack | Status |
|---|---|---|---|
| **C++ backend** | [`cpp_backend/`](cpp_backend) | [Crow](https://github.com/CrowCpp/Crow) + MySQL Connector/C++ | Core REST endpoints (products, register, login, add to cart) |
| **Node.js backend** (powers the live preview) | [`server.ts`](server.ts) | Express + SQLite (`better-sqlite3`) + JWT | Full feature set used by the React frontend |

> **Note:** The React frontend talks to the Node.js server by default, because the preview environment this project was generated in runs Node. The C++ backend implements the same `/api` routes (a subset) and is the "Crow" part of this project. See [Running the C++ backend](#running-the-c-backend).

---

## Features

- Product grid with images, prices (₹) and categories
- Live search and category filtering (All, Accessories, Essentials, Bags, Tech, Stationery, Home)
- Register / login / logout
- Cart drawer: add items, change quantities, remove items, see the subtotal
- Cart is stored per user in the database
- Responsive UI with animations (Tailwind CSS + Motion)
- Seeded with 8 sample products on first run

> The **Checkout** button in the cart is UI only. There is no order or payment flow yet.

## Tech Stack

**Frontend:** React 19, TypeScript, Vite 6, Tailwind CSS 4, Motion, Lucide icons

**Node.js backend:** Express, better-sqlite3, bcryptjs, jsonwebtoken, cookie-parser

**C++ backend:** C++17, Crow, MySQL Connector/C++, MySQL, CMake

## Project Structure

```
.
├── cpp_backend/          # C++ (Crow + MySQL) backend
│   ├── main.cpp          # Routes and DB access
│   ├── CMakeLists.txt
│   └── README.md         # C++ setup notes and SQL schema
├── src/
│   ├── App.tsx           # Navbar, product grid, auth modal, cart drawer
│   ├── main.tsx
│   └── index.css
├── server.ts             # Express + SQLite server (also serves the Vite app)
├── ecommerce.db          # SQLite database (created and seeded automatically)
├── index.html
├── vite.config.ts
├── package.json
└── .env.example
```

## Getting Started (React + Node.js)

### Prerequisites

- [Node.js](https://nodejs.org/) 18 or newer
- npm

### Install and run

```bash
# 1. Clone the repository
git clone https://github.com/gamerpixel387-crypto/E-Commerce-WEBAPP-using-CPP-CROW.git
cd E-Commerce-WEBAPP-using-CPP-CROW

# 2. Install dependencies
npm install

# 3. (Optional) Set up environment variables
cp .env.example .env

# 4. Start the dev server (Express API + Vite)
npm run dev
```

Open **http://localhost:3000**.

The SQLite database (`ecommerce.db`) is created and seeded with sample products on first launch.

### Available scripts

| Command | Description |
|---|---|
| `npm run dev` | Start the Express server with Vite middleware (port 3000) |
| `npm run build` | Build the frontend into `dist/` |
| `npm run preview` | Preview the production frontend build |
| `npm run lint` | Type-check with `tsc --noEmit` |
| `npm run clean` | Remove `dist/` |

For production, run `npm run build`, then start the server with `NODE_ENV=production`. It will serve the files in `dist/`.

### Environment variables

| Variable | Purpose |
|---|---|
| `JWT_SECRET` | Secret used to sign auth tokens. Defaults to an insecure placeholder, so **set your own value outside local development.** |
| `GEMINI_API_KEY`, `APP_URL` | Listed in `.env.example` (they come from the AI Studio template). The current app code doesn't use them. |

> **Cookie note:** Auth cookies are set with `secure: true` and `sameSite: "none"`. Browsers generally accept `Secure` cookies on `http://localhost`, but if login doesn't stick on another host, serve the app over HTTPS.

## Running the C++ backend

The C++ server exposes the same API on **port 8080**, backed by MySQL.

### Prerequisites

- A C++17 compiler (GCC or Clang)
- [Crow](https://github.com/CrowCpp/Crow)
- [MySQL Connector/C++](https://dev.mysql.com/downloads/connector/cpp/)
- A running MySQL server

### 1. Create the database

```sql
CREATE DATABASE ecommerce_db;
USE ecommerce_db;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL
);

CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    image VARCHAR(255) NOT NULL
);

CREATE TABLE cart (
    user_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT DEFAULT 1,
    PRIMARY KEY (user_id, product_id),
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

INSERT INTO products (name, price, image) VALUES
('Minimalist Watch', 2499.00, 'https://images.unsplash.com/photo-1523275335684-37898b6baf30'),
('Leather Wallet', 1299.00, 'https://images.unsplash.com/photo-1627123424574-724758594e93');
```

### 2. Configure the connection

Database credentials are constants at the top of [`cpp_backend/main.cpp`](cpp_backend/main.cpp). Edit them to match your MySQL setup:

```cpp
const string DB_HOST = "tcp://127.0.0.1:3306";
const string DB_USER = "root";
const string DB_PASS = "password";
const string DB_NAME = "ecommerce_db";
```

### 3. Build and run

```bash
cd cpp_backend

# Direct compile
g++ main.cpp -o server -std=c++17 -lmysqlcppconn -lpthread
./server
```

Or with CMake (you may need to adjust `CMakeLists.txt` so it can find Crow and the MySQL connector on your system):

```bash
mkdir build && cd build
cmake ..
cmake --build .
./EcommerceBackend
```

The server listens on **http://localhost:8080**.

## API Reference

| Method | Endpoint | Auth | Description | Node | C++ |
|---|---|---|---|:-:|:-:|
| `GET` | `/api/products` | – | List all products | ✅ | ✅ |
| `POST` | `/api/register` | – | Create an account (`name`, `email`, `password`) | ✅ | ✅ |
| `POST` | `/api/login` | – | Log in (`email`, `password`) | ✅ | ✅ |
| `POST` | `/api/logout` | – | Clear the session cookie | ✅ | – |
| `GET` | `/api/me` | JWT cookie | Current user | ✅ | – |
| `GET` | `/api/cart` | JWT cookie | Items in the user's cart | ✅ | – |
| `POST` | `/api/cart` | JWT cookie | Add an item (`productId`, `quantity`) | ✅ | ✅ (takes `userId` in the body) |
| `DELETE` | `/api/cart/:productId` | JWT cookie | Remove an item from the cart | ✅ | – |

### Differences between the two backends

The C++ backend is a simpler, learning-oriented implementation:

- Passwords are stored and compared as plain text (the source has a `// In real app, hash this!` note). The Node server hashes them with bcrypt.
- It has no JWT or cookie sessions. Cart requests pass a `userId` directly.
- It has no `category` column on products.
- It has no cart read/remove, `/me` or `/logout` routes.

To use the C++ server with the React app, you would need to add the missing routes and a session strategy, and point the Vite dev server's `/api` proxy at port 8080.

## Roadmap / Ideas

- Hash passwords and add real auth to the C++ backend
- Add cart read/remove, `/me` and `/logout` routes to the C++ backend
- Order and checkout flow with payment integration
- Product detail pages and admin product management
- Read DB credentials from environment variables instead of hard-coding them

## License

No license file is included yet. Add one (for example MIT) before reusing or distributing this code.
