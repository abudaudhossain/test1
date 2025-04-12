
# 🧱 Project: Idealo Admin & Price Comparison Portal

## 🚀 Overview
This platform combines a public price comparison website (similar to Idealo) with a secure admin portal for clients. Clients can submit their product feeds, view their clicks, and receive billing based on real user activity. The public frontend is SEO-optimized and protected against scrapers.

---

## ⚙️ Tech Stack

| Area              | Technology       |
|-------------------|------------------|
| Frontend (SSR)    | Next.js + Tailwind CSS |
| Backend/API       | Node.js + Express or Fastify |
| ORM / DB Access   | Prisma |
| Database          | PostgreSQL |
| Email             | Nodemailer |
| Captcha           | reCAPTCHA v2 or hCaptcha |
| Charts            | Recharts / Chart.js |
| PDF Export        | Puppeteer or PDFKit |
| Hosting           | Any VPS with Node + PostgreSQL |
| API Security      | JWT + API Key system |

---

## 🗃️ Prisma Database Schema (v5 compatible)

```prisma
model Client {
  id           String   @id @default(uuid())
  name         String
  email        String   @unique
  clickPrice   Float    @default(0.25)
  apiKey       String   @unique @default(uuid())
  feed         Feed?
  user         User?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}

model Feed {
  id            String     @id @default(uuid())
  clientId      String     @unique
  url           String
  format        FeedFormat @default(CSV)
  status        FeedStatus @default(PENDING)
  errorMessage  String?
  validatedAt   DateTime?
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt

  client        Client     @relation(fields: [clientId], references: [id])
  logs          FeedLog[]
}

enum FeedFormat {
  CSV
  XML
  JSON
}

enum FeedStatus {
  PENDING
  VALID
  ERROR
  ACTIVE
}

model FeedLog {
  id         String   @id @default(uuid())
  feedId     String
  timestamp  DateTime @default(now())
  status     String
  message    String?
  feed       Feed     @relation(fields: [feedId], references: [id])
}

model Product {
  id           String     @id @default(uuid())
  gtin         String     @unique
  title        String
  description  String?
  category     String?
  imageUrl     String?
  lastImported DateTime?
  visible      Boolean    @default(true)
  active       Boolean    @default(false)
  offers       Offer[]
  priceHistory PriceHistory[]
}

model Offer {
  id           String   @id @default(uuid())
  productId    String
  clientId     String
  price        Float
  shippingCost Float
  deliveryTime String
  link         String
  availability String?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  product      Product  @relation(fields: [productId], references: [id])
  client       Client   @relation(fields: [clientId], references: [id])
}

model PriceHistory {
  id         String   @id @default(uuid())
  productId  String
  date       DateTime
  price      Float
  offerCount Int
  product    Product  @relation(fields: [productId], references: [id])
}

model Click {
  id           String   @id @default(uuid())
  offerId      String
  productId    String
  clientId     String
  ipAddress    String
  userAgent    String?
  fingerprint  String?
  clickedAt    DateTime @default(now())
  counted      Boolean  @default(true)
  wasCaptcha   Boolean  @default(false)

  offer        Offer    @relation(fields: [offerId], references: [id])
}

model ClickPriceChange {
  id         String   @id @default(uuid())
  clientId   String
  oldPrice   Float
  newPrice   Float
  changedBy  String
  reason     String?
  changedAt  DateTime @default(now())
}

model User {
  id         String   @id @default(uuid())
  email      String   @unique
  password   String
  role       UserRole
  client     Client?
}

enum UserRole {
  SUPERADMIN
  ADMIN
  CUSTOMER
}
```

---

## 📚 Modules

1. **Feed Onboarding & Validation**  
   Clients provide CSV feed URLs → Validated manually or via cron job → Errors logged and notified

2. **Public Product Pages**  
   Products aggregated via GTIN → Offers sorted by total price → Includes structured data (schema.org)

3. **Click Tracking**  
   1 real click per visitor/IP per product → CAPTCHA on repeat clicks → Full transparency in dashboard

4. **Client Dashboard**  
   Product list, click stats, feed status, filtering → Real vs blocked clicks overview

5. **Admin Panel**  
   View all clients, simulate login, manage feeds, set custom click price, monitor billing

6. **Billing API**  
   Endpoint: `/api/billing/client/:id?from=YYYY-MM-DD&to=YYYY-MM-DD`  
   Auth: API Key → Used for external tools like wFirma

7. **SEO & Anti-Scraper**  
   - SSR with Next.js  
   - schema.org metadata  
   - Obfuscated prices and links  
   - Honeypot links  
   - Token-based redirections

---

## 🧪 Setup Checklist

```bash
npm install
npx prisma migrate dev --name init
npm run dev
```

✅ Ready to develop.

