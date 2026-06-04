# EtharaCafe - Curated Loyalty Platform

A modern, serverless Next.js web application for tracking customer loyalty points and redemptions, built with a premium minimalist aesthetic.

## Tech Stack
- **Framework**: Next.js 16 (App Router)
- **Styling**: Tailwind CSS v4
- **Database**: SQLite
- **ORM**: Prisma
- **State Management**: Zustand
- **Icons**: Lucide React
- **Authentication**: JWT & bcryptjs

---

## 🚀 Getting Started

### 1. Install Dependencies
```bash
pnpm install
```

### 2. Setup the Database
Push the Prisma schema to generate the local SQLite database (`dev.db`).
```bash
npx prisma db push
```

### 3. Seed the Database
Populate the database with mock administrators and customers.
```bash
pnpm dlx tsx scripts/seed.ts
```

### 4. Start the Development Server
```bash
pnpm run dev
```

The application will be running at [http://localhost:3000](http://localhost:3000).

---

## 🔐 Mock Credentials

Use the following credentials to access the different portals.

### **Administrator Portal**
Used for managing customer accounts and incrementing points.
- **Email**: `admin@etharacafe.com`
- **Password**: `CafeAdmin2026!`

### **Customer Portal**
Used for viewing point balances and redeeming rewards (automatically unlocking at 10 points).
- **Email**: `customer@etharacafe.com`
- **Password**: `Loyalty2026!`

*(Note: There are also additional mock customers populated by the seed script with varying point balances that share the same customer password `Loyalty2026!`).*

---

## Architecture Notes
- **Server Actions**: Form submissions and database mutations are handled entirely via Next.js Server Actions.
- **Prisma**: Configured to use a local `dev.db` file, requiring zero external database configuration.
- **Zustand**: Used on the Dashboard to handle client-side optimistic UI updates when redeeming points.
