# Bun & Brew — Backend API

[![My Skills](https://skillicons.dev/icons?i=nodejs,typescript,express,postgres,bun)](https://skillicons.dev)

A production-style REST API for a coffee shop and bakery ordering system.  
Built with Node.js, TypeScript, Express 5, and PostgreSQL.

## 🔗 Live API

> `https://your-deployed-url.com/api/v1`  
> *(replace with your deployed URL)*

---

## Features

- JWT authentication with bcrypt password hashing
- Role-based authorization (`customer` / `admin`)
- Full CRUD for categories and menu items
- Order placement with transactional writes
- Order status lifecycle management
- Input validation via VineJS
- Security headers (Helmet), CORS, and request logging

---

## Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js 18+ |
| Language | TypeScript 5 |
| Framework | Express 5 |
| Database | PostgreSQL 18.3 |
| Validation | VineJS |
| Auth | JWT + bcrypt |
| Security | Helmet, CORS |
| Package Manager | Bun |

---

## Project Structure

```
server/
├── config/               # Database and environment setup
├── database/
│   └── migrations/       # SQL schema files
├── middlewares/          # Auth, role, logger, error handling
├── modules/
│   ├── auth/
│   ├── categories/
│   ├── menu/
│   ├── orders/
│   └── users/
└── types/                # Express type augmentation
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL
- Bun *(optional — npm works too)*

### 1. Clone the repository

```bash
git clone https://github.com/mohamedkernel/bun-brew-backend.git
cd bun-brew-backend
```

### 2. Install dependencies

```bash
bun install
# or
npm install
```

### 3. Configure environment

```bash
cp .env.example .env
```

Fill in your `.env`:

```env
NODE_ENV=development
PORT=3000
CORS_ORIGIN=http://localhost:3000

DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=your_password
DB_NAME=coffeeshop_db

JWT_SECRET=your_secret_key
JWT_EXPIRES_IN=7d
```

### 4. Run database migrations

Execute the SQL files in order:

```bash
psql -U postgres -d coffeeshop_db -f server/database/migrations/001_create_users.sql
psql -U postgres -d coffeeshop_db -f server/database/migrations/002_create_categories.sql
psql -U postgres -d coffeeshop_db -f server/database/migrations/003_create_menu_items.sql
psql -U postgres -d coffeeshop_db -f server/database/migrations/004_create_orders.sql
```

This creates the following tables: `users`, `categories`, `menu_items`, `orders`, `order_items`.

### 5. Start the server

```bash
# Development
bun run dev

# Production build
bun run build
bun run start
```

---

## API Reference

### Base URL

```
/api/v1
```

### Response Format

**Success**
```json
{
  "status": "success",
  "data": {}
}
```

**Error**
```json
{
  "status": "error",
  "message": "Something went wrong"
}
```

### Authentication

Protected routes require a Bearer token in the `Authorization` header:

```http
Authorization: Bearer <your_token>
```

### Roles

| Role | Permissions |
|---|---|
| `customer` | Register, login, view menu, place orders, view own orders |
| `admin` | All customer permissions + manage users, categories, menu items, and all orders |

---

## Endpoints

### Auth

| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/auth/register` | Public | Register a new account |
| POST | `/auth/login` | Public | Login and receive a token |
| GET | `/auth/me` | Protected | Get current authenticated user |

### Categories

| Method | Route | Access | Description |
|---|---|---|---|
| GET | `/categories` | Public | Get all categories |
| POST | `/categories` | Admin | Create a category |
| PUT | `/categories/:id` | Admin | Update a category |
| DELETE | `/categories/:id` | Admin | Delete a category |

### Menu

| Method | Route | Access | Description |
|---|---|---|---|
| GET | `/menu` | Public | Get all menu items |
| GET | `/menu/:id` | Public | Get a single menu item |
| POST | `/menu` | Admin | Create a menu item |
| PUT | `/menu/:id` | Admin | Update a menu item |
| DELETE | `/menu/:id` | Admin | Delete a menu item |

### Orders

| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/orders` | Customer | Place a new order |
| GET | `/orders/my-orders` | Customer | Get own order history |
| GET | `/orders` | Admin | Get all orders |
| PUT | `/orders/:id/status` | Admin | Update order status |

**Order status values:** `pending` → `confirmed` → `preparing` → `ready` → `delivered` / `cancelled`

### Users

| Method | Route | Access | Description |
|---|---|---|---|
| GET | `/users` | Admin | Get all users |
| GET | `/users/:id` | Admin | Get user by ID |

---

## Request & Response Examples

### Register

```bash
curl -X POST https://your-deployed-url.com/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Mohamed",
    "email": "mohamed@example.com",
    "password": "password123"
  }'
```

```json
{
  "status": "success",
  "data": {
    "user": {
      "id": "uuid",
      "name": "Mohamed",
      "email": "mohamed@example.com",
      "role": "customer"
    },
    "token": "jwt_token"
  }
}
```

### Place an Order

```bash
curl -X POST https://your-deployed-url.com/api/v1/orders \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "items": [
      { "menu_item_id": 1, "quantity": 2 },
      { "menu_item_id": 4, "quantity": 1 }
    ]
  }'
```

---

## Validation Rules

| Field | Rule |
|---|---|
| `name` | 2–100 characters |
| `email` | Valid email format |
| `password` | 8–64 characters |
| Category `name` | 3–20 characters |
| Category `description` | 20–100 characters |
| Menu item `price` | Number, minimum 0 |
| Menu item `type` | `coffee`, `bun`, or `other` |
| Order `items` | At least 1 item, each quantity ≥ 1 |

---

## HTTP Status Codes

| Code | Meaning |
|---|---|
| `200` | Success |
| `201` | Created |
| `400` | Bad request / validation error |
| `401` | Missing or invalid token |
| `403` | Insufficient role permissions |
| `404` | Resource not found |
| `409` | Conflict (duplicate email, category, or menu item) |

---

## License

MIT
