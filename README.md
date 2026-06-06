# NS Store

Full-stack e-commerce store built with **Next.js 16**, **React 19**, **Prisma** (SQLite), and **NextAuth v5**.

## Tech Stack

| Layer | Tech |
|---|---|
| Framework | Next.js 16.2.7 (Turbopack) |
| UI | React 19.2, Tailwind CSS 4, Framer Motion, Recharts |
| Auth | NextAuth v5 (beta.31) with Prisma adapter |
| Database | SQLite via Prisma (Turso-compatible schema) |
| Icons | Lucide React |

## Prerequisites

- **Node.js** >= 20
- **npm** >= 10

## Local Development

```bash
# 1. Install dependencies
npm install

# 2. Copy environment variables
cp .env.example .env

# 3. Generate Prisma client and run migrations
npx prisma generate
npx prisma db push

# 4. Seed the database with sample data
npm run seed

# 5. Start dev server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### Default Admin Access

After seeding, you can create an admin account via `/auth/signup`, then update your role to `admin` in the database:

```sql
UPDATE User SET role = 'admin' WHERE email = 'your@email.com';
```

## Available Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start dev server (port 3000, configurable via `PORT`) |
| `npm run build` | Production build |
| `npm start` | Start production server |
| `npm run lint` | Run ESLint |
| `npm run seed` | Seed database with sample products/categories |

## Deployment

### Option 1: Vercel (recommended)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fitnoyeem%2Fshop)

1. Push this repo to GitHub
2. Import the repo into [Vercel](https://vercel.com)
3. Add the required **Environment Variables** (see below)
4. Set Build Command: `npm run build`
5. Set Output Directory: `.next`
6. Deploy

### Option 2: Node.js Server (VPS / Railway / Render / Fly.io)

```bash
# 1. Install dependencies
npm install

# 2. Generate Prisma client
npx prisma generate

# 3. Set environment variables (see below)

# 4. Build
npm run build

# 5. Start
npm start
```

### Option 3: Docker

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate && npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

```bash
docker build -t ns-store .
docker run -p 3000:3000 --env-file .env ns-store
```

## Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `DATABASE_URL` | Yes | — | SQLite path (e.g. `file:./dev.db`) or Turso/libSQL connection string |
| `AUTH_SECRET` | Yes | — | NextAuth secret — generate with `npx auth secret` |
| `NEXTAUTH_URL` | For production | `http://localhost:3000` | Canonical URL of your deployment |
| `PORT` | No | `3000` | Server port |

### Production Checklist

- [ ] Generate a strong `AUTH_SECRET`: `npx auth secret`
- [ ] Set `NEXTAUTH_URL` to your production domain
- [ ] Switch to a production database (see below)

## Database

The project uses **SQLite** by default via Prisma. The schema is compatible with [Turso](https://turso.tech) (libSQL) for production.

### Switch to PostgreSQL (for production)

1. Install `@prisma/adapter-pg` and `pg`
2. Update `prisma/schema.prisma`:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

3. Update your `.env`:

```env
DATABASE_URL="postgresql://user:password@host:5432/nsstore"
```

4. Run migrations:

```bash
npx prisma migrate dev --name init
npx prisma generate
npm run seed
```

## Project Structure

```
src/
├── app/              # Next.js App Router pages & API routes
│   ├── account/      # User account page
│   ├── admin/        # Admin dashboard, products, orders
│   ├── api/          # REST API handlers
│   ├── auth/         # Sign in / sign up pages
│   ├── cart/         # Shopping cart page
│   ├── checkout/     # Checkout page
│   └── shop/         # Product listing & detail pages
├── components/       # Shared React components
├── lib/              # Utilities, auth config, store
└── types/            # TypeScript type declarations
```

## API Routes

| Route | Method | Description |
|---|---|---|
| `/api/products` | GET | List all products |
| `/api/products` | POST | Create product (admin) |
| `/api/products/[slug]` | GET | Single product |
| `/api/products/[slug]` | PUT | Update product (admin) |
| `/api/products/[slug]` | DELETE | Delete product (admin) |
| `/api/orders` | POST | Create order |
| `/api/orders` | GET | User's orders |
| `/api/admin/orders` | GET | All orders (admin) |
| `/api/admin/orders` | PATCH | Update order status (admin) |
| `/api/admin/stats` | GET | Dashboard stats (admin) |
| `/api/categories` | GET | List categories |
| `/api/users` | GET | List users (admin) |
| `/api/upload` | POST | Image upload |
| `/api/auth/*` | — | NextAuth endpoints |
| `/api/auth/signup` | POST | Register new user |
