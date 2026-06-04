# EtharaCafe — Complete Functional Blueprint & Production Engineering Specification (Next.js App Router Edition)

*System Architecture, Database Schemas, Design Tokens, Security Implementation, Component Contracts, Test Protocols, and Technical Specification*

---

## 1. Executive Product Vision & Core Workflow

EtharaCafe is an architectural, minimalist loyalty web application designed to replace physical punch cards. The application operates on a transactional loop that rewards user loyalty while offering administrators absolute control over customer point balances through an intuitive, data-dense management interface.

### Core User Journeys & State Machines

#### Customer Ecosystem
The customer journey is strictly confined to tracking loyalty and redeeming rewards.
* **Authentication:** Access accounts securely via a minimalist portal (`/portal`). The portal dynamically routes users to `/dashboard` upon successful JWT validation.
* **Loyalty Monitoring:** View a real-time visual point tracker showing progress toward a rigid reward threshold of **10 points**. The UI must reflect real-time hydration of server-fetched point balances into a client-side Zustand store.
* **Reward Selection & Redemption:** Upon reaching 10 points, a sliding action drawer reveals a premium selection of **3 specific menu items**:
  1. *Signature Double Espresso*
  2. *Mint Cream Cold Brew*
  3. *Artisanal Pistachio Croissant*
* **Point Reset Logic:** Selecting a reward deducts exactly 10 points. If a user has an overflow balance (e.g., 12 points), the remaining points (2 points) are preserved after the reset. This mutation is handled via Next.js Server Actions, triggering optimistic UI updates.

#### Administrator Ecosystem
The administrator journey prioritizes high-throughput data management and point issuance.
* **User Management Matrix (`/admin`):** View a unified, tabular ledger displaying all registered customers, their contact details, point histories, and account creation dates sorted chronologically.
* **Transactional Controls:** Safely increment customer point balances following verified in-store visits. The action requires clicking a `+1 Point` button that manages local loading states to prevent double-submissions.
* **System Overrides:** Manually adjust point balances or update customer records directly from an administrative dashboard using Server Actions.

---

## 2. Technical Stack Specification

The application must use a modern Next.js ecosystem engineered for serverless deployment:

* **Framework:** Next.js (App Router, v15+) utilizing Server Actions for all data mutations to eliminate traditional REST API boilerplate.
* **Language:** TypeScript with strict type checking enabled. No `any` types unless absolutely unavoidable in global type declarations.
* **Styling:** Tailwind CSS v4 utilizing the `@theme` directive inside `src/app/globals.css`.
* **State Architecture:** Zustand for lightweight, centralized global client state management where server-state syncing is required (specifically for point balances).
* **Database Engine:** SQLite + Prisma ORM, utilizing a local file-based database (`dev.db`) for seamless environment setup and rapid prototyping.
* **Security Stack:** JWT signing/verification using `jose`. Next.js built-in cookie handling (`cookies().set()`) for `httpOnly`, `Secure`, `SameSite=Strict` session tokens, combined with middleware-level route protection to bounce unauthenticated users.
* **Iconography:** `lucide-react` for minimalist, scalable vector icons.

---

## 3. Design System & Frontend Architecture

### Visual Direction
The design follows a high-end, editorial aesthetic inspired by minimalist architecture magazines. The layout prioritizes ample whitespace, sharp lines, and crisp typography over decorative elements.

### The 60-30-10 Color System
The user interface implements a strict **60-30-10 color rule** mapped across a light, warm palette:

```css
/* Placed inside src/app/globals.css using Tailwind v4 @theme syntax */
@theme {
  /* 60% Dominant Base: Organic Cream/Warm Alabaster */
  --color-bg-base: #FDFBF7;
  --color-bg-surface: #F5F2EB;
  --color-bg-raised: #FFFFFF;

  /* 30% Structural/Textual: Dark Roast Espresso Shades */
  --color-text-primary: #1C120C;
  --color-text-muted: #5C524A;
  --color-border-clean: rgba(28, 18, 12, 0.08);
  --color-border-strong: rgba(28, 18, 12, 0.18);

  /* 10% Intentional Accent: Clean Mint Green */
  --color-accent-mint: #4E8770;
  --color-accent-mint-muted: rgba(78, 135, 112, 0.08);

  /* System Indicators */
  --color-sys-danger: #A94442;
  --color-sys-success: #3C763D;
}
```

### Design Boundaries (The "Never" List)
1. **No Heavy Drop Shadows:** Depth must be achieved strictly through background shifts and `1px` solid borders (`var(--color-border-clean)`). Only very subtle, expansive shadows (`shadow-[0_-8px_30px_rgb(0,0,0,0.04)]`) are allowed for bottom drawers.
2. **Sharp Geometry:** Maximum border-radius for buttons and containers is fixed at `2px` to `4px`. Navigation markers must use square corners.
3. **Minimal Gradients:** Surfaces should primarily use flat, solid hex colors. Very subtle, architectural gradients (`bg-gradient-to-br`) are permitted *only* on the main portal canvas to add depth without breaking the aesthetic.
4. **No Native Browser Modals:** Absolutely no usage of `window.alert()` or `window.confirm()`. All dialogs must be custom React components (Modals/Toasts).

### Typography Scale
* **Headings & Accent Displays:** `Playfair Display` or `DM Serif Display` (Weight: `400` only. Never bold serif typography).
* **Body & Administrative Interfaces:** `Instrument Sans` or `Inter` (Weights: `400`, `500`, `600`).
* **Body Copy Tracking:** Fixed line-height of `1.75` for optimal readability and breathing room.

---

## 4. Comprehensive File System Architecture

```text
etharacafe/
├── src/
│   ├── app/
│   │   ├── admin/
│   │   │   └── page.tsx        # Server Component: Admin ledger layout
│   │   ├── dashboard/
│   │   │   ├── page.tsx        # Server Component: Fetches user points
│   │   │   └── DashboardClient.tsx # Client Component: Mounts Zustand and components
│   │   ├── portal/
│   │   │   └── page.tsx        # Client Component: Login form and UI
│   │   ├── globals.css         # Tailwind tokens
│   │   └── layout.tsx          # Root layout with fonts & suppressHydrationWarning
│   ├── components/
│   │   ├── AdminLedger.tsx     # Client Component: User table and increment logic
│   │   ├── ConfirmModal.tsx    # Custom modal for redemption confirmations
│   │   ├── Navbar.tsx          # Universal navigation header
│   │   ├── PointTracker.tsx    # 10-slot visual point grid
│   │   ├── RewardDrawer.tsx    # Bottom sheet for reward selection
│   │   └── Toast.tsx           # Success/Error toast notifications
│   ├── lib/
│   │   ├── actions.ts          # All Next.js Server Actions
│   │   ├── auth.ts             # JWT signing and verification
│   │   └── db.ts               # Prisma global instantiation
│   └── store/
│       └── useLoyaltyStore.ts  # Zustand store definition
├── prisma/
│   └── schema.prisma           # SQLite Database models
├── scripts/
│   └── seed.ts                 # Database seeding script
├── middleware.ts               # Edge middleware for route protection
├── package.json
└── tsconfig.json
```

---

## 5. Database Schema & Data Models

### Prisma Schema (`prisma/schema.prisma`)

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL") // "file:./dev.db"
}

model User {
  id            String   @id @default(uuid())
  name          String
  email         String   @unique
  password      String
  phoneNumber   String
  role          String   @default("customer") // 'customer' or 'admin'
  pointsBalance Int      @default(0)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  transactionsAsCustomer Transaction[] @relation("CustomerTransactions")
  transactionsAsAdmin    Transaction[] @relation("AdminTransactions")
}

model Transaction {
  id              String   @id @default(uuid())
  customerId      String
  adminOperatorId String
  type            String   // 'increment' or 'redemption'
  pointsExchanged Int
  rewardSelected  String   @default("none")
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  customer      User @relation("CustomerTransactions", fields: [customerId], references: [id])
  adminOperator User @relation("AdminTransactions", fields: [adminOperatorId], references: [id])
}
```

---

## 6. Server Initialization & Authentication Contracts

### Prisma Initialization (`src/lib/db.ts`)
Next.js Server Actions and Route Handlers run in serverless contexts. You must cache the Prisma Client instance on the `global` object to prevent connection exhaustion and memory leaks during development hot-reloads.

### JWT Authentication & Middleware (`middleware.ts`)
The application implements stateless authentication. 
* Passwords must be hashed using `bcryptjs` (salt rounds: 10).
* JSON Web Tokens (JWT) are signed and verified using `jose` to ensure edge-compatibility in Next.js middleware.
* The `middleware.ts` file intercepts all requests to `/dashboard/*` and `/admin/*`. If the `session` cookie is missing or invalid, the request is immediately redirected to `/portal`.
* If a `customer` attempts to access `/admin`, the middleware or page layout must bounce them back to `/dashboard`.

---

## 7. Component Level Specifications & UI Contracts

### 7.1 Access Portal (`/portal/page.tsx`)
* **Layout Structure:** Split Screen (60% Left, 40% Right).
* **Left Canvas:** Minimalist hero featuring an architectural gradient (`from-[#EAE1D0] via-[var(--color-bg-base)] to-[#D9CDB3]`) layered with a white `backdrop-blur`. A `lucide-react` Coffee icon sits elegantly above the brand title.
* **Right Canvas:** A clean, vertical form with transparent inputs relying entirely on bottom borders. Icons (Mail, Lock) are absolutely positioned within the input wrappers to indicate focus states.

### 7.2 Point Tracker (`PointTracker.tsx`)
* **Logic:** Maps over an array of 10 slots. Displays a `Check` icon if earned, or a faded `Star` icon if unearned.
* **Animations:** Earned slots scale slightly (`scale-105`) and use a linear gradient background. A progress bar underneath visually maps the `(points/10) * 100` percentage.

### 7.3 Reward Drawer & Modals (`RewardDrawer.tsx`, `ConfirmModal.tsx`, `Toast.tsx`)
* **The Action Drawer:** When the customer's point balance reaches 10, a bottom slide-out panel reveals the 3 reward options. 
* **Custom Confirmations:** Selecting a reward item triggers a custom `ConfirmModal` component (dimmed backdrop, centered card). It handles asynchronous redemption and maps to a loading state.
* **Toasts:** Upon success or failure, a sleek toast notification slides in from the top of the viewport, displays the status via `CheckCircle` or `AlertCircle`, and automatically dismisses after 4 seconds.

### 7.4 Admin Ledger (`AdminLedger.tsx`)
* **Tabular Layout:** Displays a responsive HTML `<table>`. 
* **Micro-interactions:** The `+1 Point` button disables itself and swaps its icon for a spinning `Loader2` while the Server Action executes, preventing accidental double-clicks from impatient administrators.

---

## 8. Development Seed Manifest

The application environment must initialize with predefined state records to allow immediate manual testing.

* **System Administrator Account:** `admin@etharacafe.com` / `CafeAdmin2026!`
* **Standard Customer Profile:** `customer@etharacafe.com` / `Loyalty2026!`
* **Mock Dataset Configuration:** The `scripts/seed.ts` script must truncate the database and populate 5 additional mock customer accounts with varying point balances (ranging from 0 to 12 points). This ensures the point-overflow and threshold logic can be validated instantly without manual setup.

---

## 9. Code Quality, SEO, & Accessibility (A11y) Rules

### Accessibility Standard
* **Semantics:** Use proper HTML5 semantic tags (`<nav>`, `<main>`, `<section>`).
* **Attributes:** Forms must use `required` attributes and proper `<label>` or `placeholder` pairing.
* **Contrast:** The color palette provided natively passes WCAG AA contrast ratios. Do not deviate from the text colors provided.

### SEO & Metadata
* Every page must define a proper `<title>` via Next.js `metadata` exports (e.g., `title: 'EtharaCafe - Admin'`).

---

## 10. Test Section & Zero-Tolerance Execution

To ensure production readiness, reliability, and project longevity, the generated application must adhere to a zero-tolerance error standard:

### Lint Verification
* Run code linting verification via ESLint configured for Next.js (`npm run lint`).
* The entire codebase must pass with **zero errors and zero warnings**. Exceptions using `eslint-disable` must be extremely rare and explicitly justified (e.g., disabling `no-var` for global TypeScript declarations).

### Compilation & Build Protocol
* Verify compilation stability by executing a production build (`npm run build`).
* The build output must succeed cleanly without throwing type mismatches, missing export errors, module resolution failures, or hydration mismatches (ensure `suppressHydrationWarning` is utilized on `<html>` and `<body>` tags).

### Debugging & Server Actions
* Implement robust `try/catch` blocks inside all Server Actions. Database errors must *never* crash the Next.js process or throw raw stack traces to the client. They must return serializable objects: `{ error: 'Message' }`.

---

## 11. Comprehensive Execution Directive

> **Instructions for Generation:** Build this Next.js App Router system following a strict, modular structure. Generate clean, complete code files without placeholders, ellipses (`// ...`), or omitted blocks. Implement robust `try/catch` architectures across all Route Handlers and Server Actions, hook the frontend components to the Zustand store, and enforce the design tokens consistently across all views. Ensure the entire project structure is fully realized, fully test-compliant as per Section 10, and completely documented.