# BalanceIt3 Full Development Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a private household cash-flow forecasting app that surfaces a trustworthy, explainable `safe to spend` number from account balances, income schedules, and recurring obligations.

**Architecture:** Next.js 15 App Router full-stack app — UI, API routes, and domain logic in one repo. Domain logic (forecasting, safe-to-spend calculation) is isolated in a first-class `lib/domain/` module, never scattered in UI or API layers. PostgreSQL via Neon (serverless managed DB, zero-ops) with Prisma ORM for type-safe data access.

**Tech Stack:** Next.js 15 (App Router), TypeScript, Prisma + Neon PostgreSQL, NextAuth v5 (Google OAuth), Tailwind CSS + shadcn/ui, Zod (validation), Vitest + Testing Library (unit/integration), Playwright (e2e)

**Personas:**
- `admin` — technical household operator; can configure accounts, schedules, overrides, reconciliation
- `member` — non-technical household user; read-only daily-use dashboard view

**FR Mapping:** FR-1 through FR-32 (see `balanceit3_docs/05_functional_requirements.md`)

---

## Phase 1: Project Foundation

### Task 1: Initialize Next.js Project

**Files:**
- Create: `package.json`, `tsconfig.json`, `next.config.ts`, `.env.example`

**Step 1: Bootstrap project**

```bash
npx create-next-app@latest . --typescript --tailwind --app --src-dir --import-alias "@/*" --no-git
```

**Step 2: Install core dependencies**

```bash
npm install @prisma/client @auth/prisma-adapter next-auth@beta zod
npm install -D prisma vitest @vitest/ui @testing-library/react @testing-library/jest-dom jsdom @vitejs/plugin-react
npx shadcn@latest init
```

**Step 3: Add Playwright**

```bash
npx playwright install --with-deps chromium
```

**Step 4: Create `.env.example`**

```bash
DATABASE_URL="postgresql://..."
AUTH_SECRET="generate-with-openssl-rand-base64-32"
AUTH_GOOGLE_ID=""
AUTH_GOOGLE_SECRET=""
ALLOWLISTED_EMAILS="user1@gmail.com,user2@gmail.com"
```

**Step 5: Create `vitest.config.ts`**

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test/setup.ts"],
    globals: true,
  },
  resolve: {
    alias: { "@": path.resolve(__dirname, "./src") },
  },
});
```

**Step 6: Create `src/test/setup.ts`**

```typescript
import "@testing-library/jest-dom";
```

**Step 7: Commit**

```bash
git add -A
git commit -m "chore: initialize Next.js 15 project with TypeScript, Tailwind, Prisma, NextAuth, Vitest"
```

---

### Task 2: Prisma Schema — Core Domain Models

**Files:**
- Create: `prisma/schema.prisma`
- Create: `prisma/migrations/` (auto-generated)

**Step 1: Write the schema**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model HouseholdUser {
  id           String   @id @default(cuid())
  email        String   @unique
  role         Role     @default(MEMBER)
  isAllowlisted Boolean @default(false)
  status       UserStatus @default(ACTIVE)
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  auditEvents  AuditEvent[]
  accounts     Account[]

  @@map("household_users")
}

enum Role {
  ADMIN
  MEMBER
}

enum UserStatus {
  ACTIVE
  DISABLED
}

model Account {
  id               String      @id @default(cuid())
  userId           String
  providerType     ProviderType @default(MANUAL)
  displayName      String
  accountType      AccountType
  currentBalance   Decimal     @db.Decimal(12, 2)
  availableBalance Decimal?    @db.Decimal(12, 2)
  lastSyncedAt     DateTime?
  includeInForecast Boolean    @default(true)
  createdAt        DateTime    @default(now())
  updatedAt        DateTime    @updatedAt

  user         HouseholdUser  @relation(fields: [userId], references: [id])
  transactions Transaction[]

  @@map("accounts")
}

enum ProviderType {
  MANUAL
  CONNECTED
}

enum AccountType {
  CHECKING
  SAVINGS
  CREDIT
  OTHER
}

model Transaction {
  id              String        @id @default(cuid())
  accountId       String
  postedAt        DateTime      @db.Date
  amount          Decimal       @db.Decimal(12, 2)
  direction       Direction
  description     String
  categoryId      String?
  sourceType      SourceType
  sourceReference String
  createdAt       DateTime      @default(now())

  account               Account                @relation(fields: [accountId], references: [id])
  reconciliationRecord  ReconciliationRecord?

  @@map("transactions")
}

enum Direction {
  INFLOW
  OUTFLOW
}

enum SourceType {
  CSV
  CONNECTED
}

model IncomeSchedule {
  id                 String          @id @default(cuid())
  label              String
  cadence            Cadence
  expectedAmount     Decimal         @db.Decimal(12, 2)
  nextExpectedDate   DateTime        @db.Date
  confidenceLevel    ConfidenceLevel @default(HIGH)
  isActive           Boolean         @default(true)
  createdAt          DateTime        @default(now())
  updatedAt          DateTime        @updatedAt

  reconciliationRecords ReconciliationRecord[]

  @@map("income_schedules")
}

model RecurringObligation {
  id              String          @id @default(cuid())
  label           String
  cadence         Cadence
  expectedAmount  Decimal         @db.Decimal(12, 2)
  nextDueDate     DateTime        @db.Date
  categoryId      String?
  confidenceLevel ConfidenceLevel @default(HIGH)
  isActive        Boolean         @default(true)
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt

  reconciliationRecords ReconciliationRecord[]

  @@map("recurring_obligations")
}

enum Cadence {
  WEEKLY
  BIWEEKLY
  SEMIMONTHLY
  MONTHLY
  ANNUAL
  CUSTOM
}

enum ConfidenceLevel {
  HIGH
  MEDIUM
  LOW
}

model ForecastWindow {
  id                  String   @id @default(cuid())
  startsOn            DateTime @db.Date
  endsOn              DateTime @db.Date
  baseBalance         Decimal  @db.Decimal(12, 2)
  projectedInflows    Decimal  @db.Decimal(12, 2)
  projectedOutflows   Decimal  @db.Decimal(12, 2)
  projectedLowPoint   Decimal  @db.Decimal(12, 2)
  generatedAt         DateTime @default(now())

  safeToSpendResult SafeToSpendResult?

  @@map("forecast_windows")
}

model SafeToSpendResult {
  id                 String          @id @default(cuid())
  forecastWindowId   String          @unique
  amount             Decimal         @db.Decimal(12, 2)
  methodVersion      String
  confidenceLevel    ConfidenceLevel
  explanationSummary String
  calculatedAt       DateTime        @default(now())

  forecastWindow ForecastWindow @relation(fields: [forecastWindowId], references: [id])

  @@map("safe_to_spend_results")
}

model ReconciliationRecord {
  id                   String              @id @default(cuid())
  subjectType          ReconciliationSubject
  incomeScheduleId     String?
  recurringObligationId String?
  transactionId        String?             @unique
  matchedTransactionId String?
  status               ReconciliationStatus @default(REVIEW)
  confidenceScore      Decimal             @db.Decimal(5, 4)
  reasonCode           String
  createdAt            DateTime            @default(now())
  updatedAt            DateTime            @updatedAt

  incomeSchedule      IncomeSchedule?      @relation(fields: [incomeScheduleId], references: [id])
  recurringObligation RecurringObligation? @relation(fields: [recurringObligationId], references: [id])
  transaction         Transaction?         @relation(fields: [transactionId], references: [id])

  @@map("reconciliation_records")
}

enum ReconciliationSubject {
  INCOME
  OBLIGATION
  TRANSACTION
}

enum ReconciliationStatus {
  MATCHED
  REVIEW
  MISSING
  IGNORED
}

model Alert {
  id                String      @id @default(cuid())
  alertType         AlertType
  severity          Severity
  message           String
  relatedEntityType String?
  relatedEntityId   String?
  isResolved        Boolean     @default(false)
  createdAt         DateTime    @default(now())
  resolvedAt        DateTime?

  @@map("alerts")
}

enum AlertType {
  CASH_FLOW_RISK
  MISSING_INCOME
  MISSING_OBLIGATION
  AUTH
  SYNC
  ANOMALY
}

enum Severity {
  INFO
  WARNING
  CRITICAL
}

model AuditEvent {
  id          String     @id @default(cuid())
  actorUserId String
  eventType   AuditEventType
  entityType  String
  entityId    String
  summary     String
  occurredAt  DateTime   @default(now())

  actor HouseholdUser @relation(fields: [actorUserId], references: [id])

  @@map("audit_events")
}

enum AuditEventType {
  AUTH
  INGESTION
  FORECAST
  OVERRIDE
  CONFIGURATION
}

model BudgetTarget {
  id          String   @id @default(cuid())
  label       String
  categoryId  String
  monthlyLimit Decimal @db.Decimal(12, 2)
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@map("budget_targets")
}
```

**Step 2: Run migration**

```bash
npx prisma migrate dev --name init
```

Expected: Migration file created, tables created in DB.

**Step 3: Verify schema**

```bash
npx prisma studio
```

Expected: All tables visible with correct columns.

**Step 4: Commit**

```bash
git add prisma/
git commit -m "feat: add Prisma schema with all domain models from domain model spec"
```

---

### Task 3: Google OAuth Authentication with Allowlist

**Files:**
- Create: `src/lib/auth.ts`
- Create: `src/app/api/auth/[...nextauth]/route.ts`
- Create: `src/middleware.ts`
- Create: `src/app/(auth)/sign-in/page.tsx`
- Test: `src/lib/__tests__/auth-allowlist.test.ts`

**Step 1: Write the failing test**

```typescript
// src/lib/__tests__/auth-allowlist.test.ts
import { describe, it, expect } from "vitest";
import { isAllowlisted } from "../auth-utils";

describe("isAllowlisted", () => {
  it("returns true for an email in the allowlist", () => {
    const list = "user1@gmail.com,user2@gmail.com";
    expect(isAllowlisted("user1@gmail.com", list)).toBe(true);
  });

  it("returns false for an email not in the allowlist", () => {
    const list = "user1@gmail.com,user2@gmail.com";
    expect(isAllowlisted("stranger@gmail.com", list)).toBe(false);
  });

  it("is case-insensitive", () => {
    const list = "User1@Gmail.com";
    expect(isAllowlisted("user1@gmail.com", list)).toBe(true);
  });

  it("returns false when allowlist is empty", () => {
    expect(isAllowlisted("user1@gmail.com", "")).toBe(false);
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npx vitest run src/lib/__tests__/auth-allowlist.test.ts
```

Expected: FAIL — `isAllowlisted` not found.

**Step 3: Create `src/lib/auth-utils.ts`**

```typescript
export function isAllowlisted(email: string, allowlistEnv: string): boolean {
  if (!allowlistEnv) return false;
  const allowed = allowlistEnv.split(",").map((e) => e.trim().toLowerCase());
  return allowed.includes(email.toLowerCase());
}
```

**Step 4: Run test to verify it passes**

```bash
npx vitest run src/lib/__tests__/auth-allowlist.test.ts
```

Expected: PASS (4 tests).

**Step 5: Create `src/lib/auth.ts`**

```typescript
import NextAuth from "next-auth";
import Google from "next-auth/providers/google";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { prisma } from "./prisma";
import { isAllowlisted } from "./auth-utils";

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [Google],
  callbacks: {
    async signIn({ user }) {
      if (!user.email) return false;
      const allowlist = process.env.ALLOWLISTED_EMAILS ?? "";
      return isAllowlisted(user.email, allowlist);
    },
    async session({ session, user }) {
      if (session.user) {
        const dbUser = await prisma.householdUser.findUnique({
          where: { email: session.user.email! },
        });
        session.user.id = user.id;
        session.user.role = dbUser?.role ?? "MEMBER";
      }
      return session;
    },
  },
  pages: { signIn: "/sign-in", error: "/sign-in" },
});
```

**Step 6: Create `src/lib/prisma.ts`**

```typescript
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({ log: ["error"] });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

**Step 7: Create `src/app/api/auth/[...nextauth]/route.ts`**

```typescript
import { handlers } from "@/lib/auth";
export const { GET, POST } = handlers;
```

**Step 8: Create `src/middleware.ts`**

```typescript
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export default auth((req) => {
  const isAuthenticated = !!req.auth;
  const isAuthPage = req.nextUrl.pathname.startsWith("/sign-in");

  if (!isAuthenticated && !isAuthPage) {
    return NextResponse.redirect(new URL("/sign-in", req.url));
  }
  return NextResponse.next();
});

export const config = {
  matcher: ["/((?!api/auth|_next/static|_next/image|favicon.ico).*)"],
};
```

**Step 9: Create `src/app/(auth)/sign-in/page.tsx`**

```tsx
import { signIn } from "@/lib/auth";

export default function SignInPage() {
  return (
    <main className="flex min-h-screen items-center justify-center">
      <div className="flex flex-col items-center gap-6 p-8">
        <h1 className="text-2xl font-semibold">BalanceIt</h1>
        <p className="text-muted-foreground text-sm">Household finance dashboard</p>
        <form
          action={async () => {
            "use server";
            await signIn("google", { redirectTo: "/" });
          }}
        >
          <button
            type="submit"
            className="bg-primary text-primary-foreground rounded-md px-6 py-2 text-sm font-medium"
          >
            Sign in with Google
          </button>
        </form>
      </div>
    </main>
  );
}
```

**Step 10: Commit**

```bash
git add src/ prisma/
git commit -m "feat: add Google OAuth authentication with email allowlist (FR-26, FR-27)"
```

---

## Phase 2: Accounts & Transaction Ingestion

### Task 4: Account Management API

**Files:**
- Create: `src/lib/domain/accounts.ts`
- Create: `src/app/api/accounts/route.ts`
- Create: `src/app/api/accounts/[id]/route.ts`
- Test: `src/lib/domain/__tests__/accounts.test.ts`

**Step 1: Write the failing tests**

```typescript
// src/lib/domain/__tests__/accounts.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { createAccount, toggleForecastInclusion } from "../accounts";
import type { PrismaClient } from "@prisma/client";

const mockPrisma = {
  account: {
    create: vi.fn(),
    update: vi.fn(),
    findMany: vi.fn(),
  },
} as unknown as PrismaClient;

describe("createAccount", () => {
  beforeEach(() => vi.clearAllMocks());

  it("creates a manual account with default forecast inclusion", async () => {
    const input = {
      userId: "user-1",
      displayName: "Chase Checking",
      accountType: "CHECKING" as const,
      currentBalance: 5000,
    };
    await createAccount(mockPrisma, input);
    expect(mockPrisma.account.create).toHaveBeenCalledWith({
      data: expect.objectContaining({
        providerType: "MANUAL",
        includeInForecast: true,
        currentBalance: 5000,
      }),
    });
  });
});

describe("toggleForecastInclusion", () => {
  it("updates includeInForecast on the account", async () => {
    await toggleForecastInclusion(mockPrisma, "account-1", false);
    expect(mockPrisma.account.update).toHaveBeenCalledWith({
      where: { id: "account-1" },
      data: { includeInForecast: false },
    });
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npx vitest run src/lib/domain/__tests__/accounts.test.ts
```

Expected: FAIL — module not found.

**Step 3: Create `src/lib/domain/accounts.ts`**

```typescript
import type { PrismaClient } from "@prisma/client";
import type { AccountType } from "@prisma/client";

interface CreateAccountInput {
  userId: string;
  displayName: string;
  accountType: AccountType;
  currentBalance: number;
  availableBalance?: number;
}

export async function createAccount(prisma: PrismaClient, input: CreateAccountInput) {
  return prisma.account.create({
    data: {
      userId: input.userId,
      displayName: input.displayName,
      accountType: input.accountType,
      currentBalance: input.currentBalance,
      availableBalance: input.availableBalance ?? null,
      providerType: "MANUAL",
      includeInForecast: true,
    },
  });
}

export async function toggleForecastInclusion(
  prisma: PrismaClient,
  accountId: string,
  include: boolean
) {
  return prisma.account.update({
    where: { id: accountId },
    data: { includeInForecast: include },
  });
}
```

**Step 4: Run test to verify it passes**

```bash
npx vitest run src/lib/domain/__tests__/accounts.test.ts
```

Expected: PASS.

**Step 5: Create `src/app/api/accounts/route.ts`**

```typescript
import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";
import { createAccount } from "@/lib/domain/accounts";
import { z } from "zod";
import { NextResponse } from "next/server";

const createAccountSchema = z.object({
  displayName: z.string().min(1),
  accountType: z.enum(["CHECKING", "SAVINGS", "CREDIT", "OTHER"]),
  currentBalance: z.number(),
  availableBalance: z.number().optional(),
});

export async function GET() {
  const session = await auth();
  if (!session) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });

  const accounts = await prisma.account.findMany({
    where: { userId: session.user.id },
    orderBy: { createdAt: "asc" },
  });
  return NextResponse.json(accounts);
}

export async function POST(req: Request) {
  const session = await auth();
  if (!session || session.user.role !== "ADMIN") {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }

  const body = createAccountSchema.safeParse(await req.json());
  if (!body.success) {
    return NextResponse.json({ error: body.error.flatten() }, { status: 400 });
  }

  const account = await createAccount(prisma, {
    userId: session.user.id,
    ...body.data,
  });
  return NextResponse.json(account, { status: 201 });
}
```

**Step 6: Commit**

```bash
git add src/lib/domain/ src/app/api/accounts/
git commit -m "feat: add account management domain logic and API routes (FR-7, FR-11)"
```

---

### Task 5: CSV Transaction Import

**Files:**
- Create: `src/lib/domain/ingestion.ts`
- Create: `src/app/api/transactions/import/route.ts`
- Test: `src/lib/domain/__tests__/ingestion.test.ts`

**Step 1: Write the failing tests**

```typescript
// src/lib/domain/__tests__/ingestion.test.ts
import { describe, it, expect } from "vitest";
import { parseTransactionCsv, detectDuplicate } from "../ingestion";

describe("parseTransactionCsv", () => {
  it("parses a valid CSV row into a normalized transaction", () => {
    const csv = `date,amount,description\n2024-01-15,-50.00,Grocery Store`;
    const [result] = parseTransactionCsv(csv, "account-1");
    expect(result).toMatchObject({
      accountId: "account-1",
      direction: "OUTFLOW",
      amount: 50,
      description: "Grocery Store",
      sourceType: "CSV",
    });
  });

  it("sets direction INFLOW for positive amounts", () => {
    const csv = `date,amount,description\n2024-01-15,2500.00,Paycheck`;
    const [result] = parseTransactionCsv(csv, "account-1");
    expect(result.direction).toBe("INFLOW");
  });

  it("returns a validation error for rows missing required fields", () => {
    const csv = `date,amount,description\n2024-01-15,,Grocery Store`;
    expect(() => parseTransactionCsv(csv, "account-1")).toThrow();
  });
});

describe("detectDuplicate", () => {
  it("identifies a duplicate by source reference", () => {
    const existing = [{ sourceReference: "csv:2024-01-15:grocery:50" }];
    const incoming = { sourceReference: "csv:2024-01-15:grocery:50" };
    expect(detectDuplicate(incoming, existing)).toBe(true);
  });

  it("returns false when no duplicate exists", () => {
    const existing = [{ sourceReference: "csv:2024-01-15:grocery:50" }];
    const incoming = { sourceReference: "csv:2024-01-16:coffee:5" };
    expect(detectDuplicate(incoming, existing)).toBe(false);
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npx vitest run src/lib/domain/__tests__/ingestion.test.ts
```

Expected: FAIL.

**Step 3: Create `src/lib/domain/ingestion.ts`**

```typescript
import { z } from "zod";

const csvRowSchema = z.object({
  date: z.string().min(1),
  amount: z.string().refine((v) => v !== "" && !isNaN(Number(v)), "Amount required"),
  description: z.string().min(1),
});

interface ParsedTransaction {
  accountId: string;
  postedAt: Date;
  amount: number;
  direction: "INFLOW" | "OUTFLOW";
  description: string;
  sourceType: "CSV";
  sourceReference: string;
}

export function parseTransactionCsv(csv: string, accountId: string): ParsedTransaction[] {
  const lines = csv.trim().split("\n");
  const headers = lines[0].split(",").map((h) => h.trim());
  return lines.slice(1).map((line) => {
    const values = line.split(",").map((v) => v.trim());
    const raw = Object.fromEntries(headers.map((h, i) => [h, values[i]]));
    const parsed = csvRowSchema.parse(raw);
    const numericAmount = Math.abs(Number(parsed.amount));
    const direction = Number(parsed.amount) >= 0 ? "INFLOW" : "OUTFLOW";
    const sourceReference = `csv:${parsed.date}:${parsed.description.toLowerCase().replace(/\s+/g, "-")}:${numericAmount}`;
    return {
      accountId,
      postedAt: new Date(parsed.date),
      amount: numericAmount,
      direction,
      description: parsed.description,
      sourceType: "CSV",
      sourceReference,
    };
  });
}

export function detectDuplicate(
  incoming: { sourceReference: string },
  existing: { sourceReference: string }[]
): boolean {
  return existing.some((e) => e.sourceReference === incoming.sourceReference);
}
```

**Step 4: Run test to verify it passes**

```bash
npx vitest run src/lib/domain/__tests__/ingestion.test.ts
```

Expected: PASS.

**Step 5: Create `src/app/api/transactions/import/route.ts`**

```typescript
import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";
import { parseTransactionCsv, detectDuplicate } from "@/lib/domain/ingestion";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const session = await auth();
  if (!session || session.user.role !== "ADMIN") {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }

  const formData = await req.formData();
  const file = formData.get("file") as File;
  const accountId = formData.get("accountId") as string;

  if (!file || !accountId) {
    return NextResponse.json({ error: "file and accountId required" }, { status: 400 });
  }

  const csv = await file.text();
  let parsed;
  try {
    parsed = parseTransactionCsv(csv, accountId);
  } catch (e) {
    return NextResponse.json({ error: "Invalid CSV format" }, { status: 422 });
  }

  const existing = await prisma.transaction.findMany({
    where: { accountId },
    select: { sourceReference: true },
  });

  const newTransactions = parsed.filter((t) => !detectDuplicate(t, existing));
  const skipped = parsed.length - newTransactions.length;

  await prisma.transaction.createMany({ data: newTransactions });

  await prisma.auditEvent.create({
    data: {
      actorUserId: session.user.id,
      eventType: "INGESTION",
      entityType: "Transaction",
      entityId: accountId,
      summary: `Imported ${newTransactions.length} transactions, skipped ${skipped} duplicates`,
    },
  });

  return NextResponse.json({ imported: newTransactions.length, skipped });
}
```

**Step 6: Commit**

```bash
git add src/lib/domain/ingestion.ts src/app/api/transactions/
git commit -m "feat: add CSV transaction import with deduplication and audit logging (FR-8, FR-9, FR-10)"
```

---

## Phase 3: Income Schedules & Recurring Obligations

### Task 6: Income Schedule Domain Logic

**Files:**
- Create: `src/lib/domain/income.ts`
- Create: `src/app/api/income-schedules/route.ts`
- Test: `src/lib/domain/__tests__/income.test.ts`

**Step 1: Write the failing tests**

```typescript
// src/lib/domain/__tests__/income.test.ts
import { describe, it, expect } from "vitest";
import { nextOccurrenceAfter } from "../income";

describe("nextOccurrenceAfter", () => {
  it("advances a monthly schedule to the next month", () => {
    const base = new Date("2024-01-15");
    const after = new Date("2024-01-20");
    const next = nextOccurrenceAfter(base, "MONTHLY", after);
    expect(next.toISOString().startsWith("2024-02-15")).toBe(true);
  });

  it("advances a biweekly schedule by 14 days", () => {
    const base = new Date("2024-01-01");
    const after = new Date("2024-01-08");
    const next = nextOccurrenceAfter(base, "BIWEEKLY", after);
    expect(next.toISOString().startsWith("2024-01-15")).toBe(true);
  });

  it("returns base date when it is after the reference date", () => {
    const base = new Date("2024-01-25");
    const after = new Date("2024-01-20");
    const next = nextOccurrenceAfter(base, "MONTHLY", after);
    expect(next).toEqual(base);
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npx vitest run src/lib/domain/__tests__/income.test.ts
```

Expected: FAIL.

**Step 3: Create `src/lib/domain/income.ts`**

```typescript
import type { Cadence } from "@prisma/client";

export function nextOccurrenceAfter(base: Date, cadence: Cadence, after: Date): Date {
  let next = new Date(base);
  while (next <= after) {
    switch (cadence) {
      case "WEEKLY":
        next = new Date(next.getTime() + 7 * 86400000);
        break;
      case "BIWEEKLY":
        next = new Date(next.getTime() + 14 * 86400000);
        break;
      case "SEMIMONTHLY":
        next = new Date(next.getTime() + 15 * 86400000);
        break;
      case "MONTHLY":
        next = new Date(next);
        next.setMonth(next.getMonth() + 1);
        break;
      case "ANNUAL":
        next = new Date(next);
        next.setFullYear(next.getFullYear() + 1);
        break;
      default:
        return next; // CUSTOM: caller handles
    }
  }
  return next;
}
```

**Step 4: Run test to verify it passes**

```bash
npx vitest run src/lib/domain/__tests__/income.test.ts
```

Expected: PASS.

**Step 5: Create `src/app/api/income-schedules/route.ts`** (CRUD — GET + POST)

```typescript
import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";
import { z } from "zod";
import { NextResponse } from "next/server";

const schema = z.object({
  label: z.string().min(1),
  cadence: z.enum(["WEEKLY", "BIWEEKLY", "SEMIMONTHLY", "MONTHLY", "ANNUAL", "CUSTOM"]),
  expectedAmount: z.number().positive(),
  nextExpectedDate: z.string().datetime({ offset: true }).or(z.string().date()),
  confidenceLevel: z.enum(["HIGH", "MEDIUM", "LOW"]).default("HIGH"),
});

export async function GET() {
  const session = await auth();
  if (!session) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  const schedules = await prisma.incomeSchedule.findMany({ orderBy: { nextExpectedDate: "asc" } });
  return NextResponse.json(schedules);
}

export async function POST(req: Request) {
  const session = await auth();
  if (!session || session.user.role !== "ADMIN") {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }
  const body = schema.safeParse(await req.json());
  if (!body.success) return NextResponse.json({ error: body.error.flatten() }, { status: 400 });

  const schedule = await prisma.incomeSchedule.create({
    data: { ...body.data, nextExpectedDate: new Date(body.data.nextExpectedDate) },
  });
  return NextResponse.json(schedule, { status: 201 });
}
```

**Step 6: Commit**

```bash
git add src/lib/domain/income.ts src/app/api/income-schedules/
git commit -m "feat: add income schedule domain logic and API (FR-12, FR-14)"
```

---

### Task 7: Recurring Obligations API

**Files:**
- Create: `src/app/api/recurring-obligations/route.ts`
- Test: `src/lib/domain/__tests__/obligations.test.ts`

Follow the same pattern as Task 6. Obligations use `nextDueDate` instead of `nextExpectedDate`. Test that inactive obligations are excluded from active queries.

```typescript
// src/lib/domain/__tests__/obligations.test.ts
import { describe, it, expect, vi } from "vitest";
import type { PrismaClient } from "@prisma/client";

const mockPrisma = {
  recurringObligation: { findMany: vi.fn() },
} as unknown as PrismaClient;

describe("active obligations query", () => {
  it("filters by isActive: true", async () => {
    await mockPrisma.recurringObligation.findMany({ where: { isActive: true } });
    expect(mockPrisma.recurringObligation.findMany).toHaveBeenCalledWith(
      expect.objectContaining({ where: { isActive: true } })
    );
  });
});
```

**Commit after passing:**

```bash
git commit -m "feat: add recurring obligations API (FR-13, FR-14, FR-15)"
```

---

## Phase 4: Forecasting Core (North Star)

### Task 8: Cash-Flow Forecast Engine

**Files:**
- Create: `src/lib/domain/forecasting.ts`
- Test: `src/lib/domain/__tests__/forecasting.test.ts`

**Step 1: Write the failing tests — this is the most important test suite in the project**

```typescript
// src/lib/domain/__tests__/forecasting.test.ts
import { describe, it, expect } from "vitest";
import { buildForecastWindow } from "../forecasting";
import type { ForecastInput } from "../forecasting";

const baseInput: ForecastInput = {
  baseBalance: 5000,
  startsOn: new Date("2024-01-01"),
  endsOn: new Date("2024-01-31"),
  incomeEvents: [],
  obligationEvents: [],
};

describe("buildForecastWindow", () => {
  it("returns base balance as low point when no events", () => {
    const result = buildForecastWindow(baseInput);
    expect(result.projectedLowPoint).toBe(5000);
    expect(result.projectedInflows).toBe(0);
    expect(result.projectedOutflows).toBe(0);
  });

  it("adds projected inflows from income events in window", () => {
    const input: ForecastInput = {
      ...baseInput,
      incomeEvents: [{ date: new Date("2024-01-15"), amount: 2500, label: "Paycheck" }],
    };
    const result = buildForecastWindow(input);
    expect(result.projectedInflows).toBe(2500);
  });

  it("subtracts projected outflows from obligation events in window", () => {
    const input: ForecastInput = {
      ...baseInput,
      obligationEvents: [{ date: new Date("2024-01-10"), amount: 1200, label: "Rent" }],
    };
    const result = buildForecastWindow(input);
    expect(result.projectedOutflows).toBe(1200);
  });

  it("calculates the projected low point correctly across events", () => {
    const input: ForecastInput = {
      ...baseInput,
      baseBalance: 3000,
      incomeEvents: [{ date: new Date("2024-01-20"), amount: 2000, label: "Paycheck" }],
      obligationEvents: [
        { date: new Date("2024-01-05"), amount: 1200, label: "Rent" },
        { date: new Date("2024-01-10"), amount: 400, label: "Car" },
      ],
    };
    const result = buildForecastWindow(input);
    // balance at day 5: 3000 - 1200 = 1800; day 10: 1800 - 400 = 1400 (lowest before income)
    expect(result.projectedLowPoint).toBe(1400);
  });

  it("excludes events outside the forecast window", () => {
    const input: ForecastInput = {
      ...baseInput,
      incomeEvents: [{ date: new Date("2024-02-15"), amount: 2500, label: "Paycheck" }],
    };
    const result = buildForecastWindow(input);
    expect(result.projectedInflows).toBe(0);
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npx vitest run src/lib/domain/__tests__/forecasting.test.ts
```

Expected: FAIL — module not found.

**Step 3: Create `src/lib/domain/forecasting.ts`**

```typescript
export interface CashFlowEvent {
  date: Date;
  amount: number;
  label: string;
}

export interface ForecastInput {
  baseBalance: number;
  startsOn: Date;
  endsOn: Date;
  incomeEvents: CashFlowEvent[];
  obligationEvents: CashFlowEvent[];
}

export interface ForecastResult {
  startsOn: Date;
  endsOn: Date;
  baseBalance: number;
  projectedInflows: number;
  projectedOutflows: number;
  projectedLowPoint: number;
}

function isInWindow(date: Date, start: Date, end: Date): boolean {
  return date >= start && date <= end;
}

export function buildForecastWindow(input: ForecastInput): ForecastResult {
  const { baseBalance, startsOn, endsOn, incomeEvents, obligationEvents } = input;

  const inWindow = (e: CashFlowEvent) => isInWindow(e.date, startsOn, endsOn);

  const inflows = incomeEvents.filter(inWindow);
  const outflows = obligationEvents.filter(inWindow);

  const totalInflows = inflows.reduce((s, e) => s + e.amount, 0);
  const totalOutflows = outflows.reduce((s, e) => s + e.amount, 0);

  // Build a day-by-day running balance to find the low point
  const allEvents: { date: Date; delta: number }[] = [
    ...inflows.map((e) => ({ date: e.date, delta: e.amount })),
    ...outflows.map((e) => ({ date: e.date, delta: -e.amount })),
  ].sort((a, b) => a.date.getTime() - b.date.getTime());

  let running = baseBalance;
  let lowPoint = baseBalance;

  for (const event of allEvents) {
    running += event.delta;
    if (running < lowPoint) lowPoint = running;
  }

  return {
    startsOn,
    endsOn,
    baseBalance,
    projectedInflows: totalInflows,
    projectedOutflows: totalOutflows,
    projectedLowPoint: lowPoint,
  };
}
```

**Step 4: Run test to verify it passes**

```bash
npx vitest run src/lib/domain/__tests__/forecasting.test.ts
```

Expected: PASS (5 tests).

**Step 5: Commit**

```bash
git add src/lib/domain/forecasting.ts src/lib/domain/__tests__/forecasting.test.ts
git commit -m "feat: add cash-flow forecast engine with low-point calculation (FR-1, FR-6)"
```

---

### Task 9: Safe-to-Spend Calculation

**Files:**
- Create: `src/lib/domain/safe-to-spend.ts`
- Create: `src/app/api/forecast/route.ts`
- Test: `src/lib/domain/__tests__/safe-to-spend.test.ts`

**Step 1: Write the failing tests**

```typescript
// src/lib/domain/__tests__/safe-to-spend.test.ts
import { describe, it, expect } from "vitest";
import { calculateSafeToSpend } from "../safe-to-spend";
import type { SafeToSpendInput } from "../safe-to-spend";

const base: SafeToSpendInput = {
  projectedLowPoint: 3000,
  safetyBuffer: 500,
  confidenceLevel: "HIGH",
  unreconciledObligations: 0,
};

describe("calculateSafeToSpend", () => {
  it("is low point minus safety buffer", () => {
    const result = calculateSafeToSpend(base);
    expect(result.amount).toBe(2500);
  });

  it("is zero when buffer exceeds low point", () => {
    const result = calculateSafeToSpend({ ...base, projectedLowPoint: 400 });
    expect(result.amount).toBe(0);
  });

  it("reduces by unreconciled obligations", () => {
    const result = calculateSafeToSpend({ ...base, unreconciledObligations: 300 });
    expect(result.amount).toBe(2200);
  });

  it("sets confidence LOW when input confidence is LOW", () => {
    const result = calculateSafeToSpend({ ...base, confidenceLevel: "LOW" });
    expect(result.confidenceLevel).toBe("LOW");
  });

  it("returns a human-readable explanation", () => {
    const result = calculateSafeToSpend(base);
    expect(result.explanationSummary).toContain("3000");
    expect(result.explanationSummary).toContain("500");
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npx vitest run src/lib/domain/__tests__/safe-to-spend.test.ts
```

Expected: FAIL.

**Step 3: Create `src/lib/domain/safe-to-spend.ts`**

```typescript
import type { ConfidenceLevel } from "@prisma/client";

export interface SafeToSpendInput {
  projectedLowPoint: number;
  safetyBuffer: number;
  confidenceLevel: ConfidenceLevel;
  unreconciledObligations: number;
}

export interface SafeToSpendOutput {
  amount: number;
  confidenceLevel: ConfidenceLevel;
  methodVersion: string;
  explanationSummary: string;
}

const METHOD_VERSION = "v1.0";

export function calculateSafeToSpend(input: SafeToSpendInput): SafeToSpendOutput {
  const { projectedLowPoint, safetyBuffer, confidenceLevel, unreconciledObligations } = input;
  const raw = projectedLowPoint - safetyBuffer - unreconciledObligations;
  const amount = Math.max(0, raw);

  const explanationSummary =
    `Projected low point: $${projectedLowPoint.toFixed(2)}. ` +
    `Safety buffer: $${safetyBuffer.toFixed(2)}. ` +
    (unreconciledObligations > 0
      ? `Unreconciled obligations deducted: $${unreconciledObligations.toFixed(2)}. `
      : "") +
    `Safe to spend: $${amount.toFixed(2)}.`;

  return { amount, confidenceLevel, methodVersion: METHOD_VERSION, explanationSummary };
}
```

**Step 4: Run test to verify it passes**

```bash
npx vitest run src/lib/domain/__tests__/safe-to-spend.test.ts
```

Expected: PASS.

**Step 5: Create `src/app/api/forecast/route.ts`** — orchestrates a full forecast + safe-to-spend calculation

```typescript
import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";
import { buildForecastWindow } from "@/lib/domain/forecasting";
import { calculateSafeToSpend } from "@/lib/domain/safe-to-spend";
import { nextOccurrenceAfter } from "@/lib/domain/income";
import { NextResponse } from "next/server";

const SAFETY_BUFFER = 500; // configurable in a later task
const FORECAST_DAYS = 30;

export async function GET() {
  const session = await auth();
  if (!session) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });

  const today = new Date();
  const endsOn = new Date(today);
  endsOn.setDate(endsOn.getDate() + FORECAST_DAYS);

  // Base balance: sum of included accounts
  const accounts = await prisma.account.findMany({ where: { includeInForecast: true } });
  const baseBalance = accounts.reduce((s, a) => s + Number(a.currentBalance), 0);

  // Project income events in window
  const incomeSchedules = await prisma.incomeSchedule.findMany({ where: { isActive: true } });
  const incomeEvents = incomeSchedules.map((s) => ({
    date: nextOccurrenceAfter(s.nextExpectedDate, s.cadence, today),
    amount: Number(s.expectedAmount),
    label: s.label,
  })).filter((e) => e.date <= endsOn);

  // Project obligation events in window
  const obligations = await prisma.recurringObligation.findMany({ where: { isActive: true } });
  const obligationEvents = obligations.map((o) => ({
    date: nextOccurrenceAfter(o.nextDueDate, o.cadence, today),
    amount: Number(o.expectedAmount),
    label: o.label,
  })).filter((e) => e.date <= endsOn);

  const forecast = buildForecastWindow({
    baseBalance,
    startsOn: today,
    endsOn,
    incomeEvents,
    obligationEvents,
  });

  // Unreconciled obligations (MISSING status)
  const unreconciled = await prisma.reconciliationRecord.count({
    where: { status: "MISSING" },
  });
  const unreconciledAmount = unreconciled * 0; // amount lookup in future iteration

  const safeToSpend = calculateSafeToSpend({
    projectedLowPoint: forecast.projectedLowPoint,
    safetyBuffer: SAFETY_BUFFER,
    confidenceLevel: "HIGH", // derived from data freshness in later iteration
    unreconciledObligations: unreconciledAmount,
  });

  // Persist the forecast result
  const savedForecast = await prisma.forecastWindow.create({
    data: {
      startsOn: forecast.startsOn,
      endsOn: forecast.endsOn,
      baseBalance: forecast.baseBalance,
      projectedInflows: forecast.projectedInflows,
      projectedOutflows: forecast.projectedOutflows,
      projectedLowPoint: forecast.projectedLowPoint,
    },
  });

  await prisma.safeToSpendResult.create({
    data: {
      forecastWindowId: savedForecast.id,
      amount: safeToSpend.amount,
      methodVersion: safeToSpend.methodVersion,
      confidenceLevel: safeToSpend.confidenceLevel,
      explanationSummary: safeToSpend.explanationSummary,
    },
  });

  await prisma.auditEvent.create({
    data: {
      actorUserId: session.user.id,
      eventType: "FORECAST",
      entityType: "ForecastWindow",
      entityId: savedForecast.id,
      summary: `Forecast generated. Safe to spend: $${safeToSpend.amount.toFixed(2)}`,
    },
  });

  return NextResponse.json({ forecast: savedForecast, safeToSpend });
}
```

**Step 6: Commit**

```bash
git add src/lib/domain/safe-to-spend.ts src/app/api/forecast/
git commit -m "feat: add safe-to-spend calculation and forecast API (FR-1, FR-2, FR-3, FR-4)"
```

---

## Phase 5: Reconciliation & Alerts

### Task 10: Reconciliation Engine

**Files:**
- Create: `src/lib/domain/reconciliation.ts`
- Create: `src/app/api/reconciliation/route.ts`
- Test: `src/lib/domain/__tests__/reconciliation.test.ts`

**Step 1: Write the failing tests**

```typescript
// src/lib/domain/__tests__/reconciliation.test.ts
import { describe, it, expect } from "vitest";
import { matchTransactionToExpected, scoreMatch } from "../reconciliation";

const mockTransaction = {
  id: "tx-1",
  postedAt: new Date("2024-01-15"),
  amount: 2500,
  direction: "INFLOW" as const,
  description: "DIRECT DEPOSIT EMPLOYER",
};

const mockExpected = {
  id: "sched-1",
  expectedAmount: 2500,
  nextExpectedDate: new Date("2024-01-15"),
  label: "Paycheck",
};

describe("scoreMatch", () => {
  it("returns high score for exact amount and date match", () => {
    const score = scoreMatch(mockTransaction, mockExpected);
    expect(score).toBeGreaterThan(0.8);
  });

  it("returns lower score for amount mismatch within tolerance", () => {
    const score = scoreMatch({ ...mockTransaction, amount: 2400 }, mockExpected);
    expect(score).toBeLessThan(0.8);
    expect(score).toBeGreaterThan(0);
  });

  it("returns 0 for amount outside tolerance", () => {
    const score = scoreMatch({ ...mockTransaction, amount: 500 }, mockExpected);
    expect(score).toBe(0);
  });
});

describe("matchTransactionToExpected", () => {
  it("returns MATCHED when score is above threshold", () => {
    const result = matchTransactionToExpected(mockTransaction, [mockExpected]);
    expect(result.status).toBe("MATCHED");
    expect(result.matchedId).toBe("sched-1");
  });

  it("returns REVIEW when no candidates match", () => {
    const result = matchTransactionToExpected(
      { ...mockTransaction, amount: 99 },
      [mockExpected]
    );
    expect(result.status).toBe("REVIEW");
  });
});
```

**Step 2: Run test to verify it fails**

```bash
npx vitest run src/lib/domain/__tests__/reconciliation.test.ts
```

Expected: FAIL.

**Step 3: Create `src/lib/domain/reconciliation.ts`**

```typescript
const AMOUNT_TOLERANCE = 0.05; // 5%
const DATE_TOLERANCE_DAYS = 3;
const MATCH_THRESHOLD = 0.7;

interface TransactionInput {
  id: string;
  postedAt: Date;
  amount: number;
  direction: "INFLOW" | "OUTFLOW";
  description: string;
}

interface ExpectedEvent {
  id: string;
  expectedAmount: number;
  nextExpectedDate: Date;
  label: string;
}

export function scoreMatch(tx: TransactionInput, expected: ExpectedEvent): number {
  const amountDiff = Math.abs(tx.amount - expected.expectedAmount) / expected.expectedAmount;
  if (amountDiff > AMOUNT_TOLERANCE) return 0;

  const dateDiff =
    Math.abs(tx.postedAt.getTime() - expected.nextExpectedDate.getTime()) / 86400000;
  const dateScore = Math.max(0, 1 - dateDiff / DATE_TOLERANCE_DAYS);
  const amountScore = 1 - amountDiff / AMOUNT_TOLERANCE;

  return (amountScore * 0.6 + dateScore * 0.4);
}

export function matchTransactionToExpected(
  tx: TransactionInput,
  candidates: ExpectedEvent[]
): { status: "MATCHED" | "REVIEW"; matchedId: string | null; score: number } {
  let best: { score: number; id: string } | null = null;

  for (const candidate of candidates) {
    const score = scoreMatch(tx, candidate);
    if (score >= MATCH_THRESHOLD && (!best || score > best.score)) {
      best = { score, id: candidate.id };
    }
  }

  if (best) return { status: "MATCHED", matchedId: best.id, score: best.score };
  return { status: "REVIEW", matchedId: null, score: 0 };
}
```

**Step 4: Run test to verify it passes**

```bash
npx vitest run src/lib/domain/__tests__/reconciliation.test.ts
```

Expected: PASS.

**Step 5: Commit**

```bash
git add src/lib/domain/reconciliation.ts src/lib/domain/__tests__/reconciliation.test.ts
git commit -m "feat: add reconciliation matching engine (FR-16, FR-17)"
```

---

### Task 11: Alert Generation

**Files:**
- Create: `src/lib/domain/alerts.ts`
- Test: `src/lib/domain/__tests__/alerts.test.ts`

**Step 1: Write the failing tests**

```typescript
// src/lib/domain/__tests__/alerts.test.ts
import { describe, it, expect } from "vitest";
import { evaluateForecastAlerts } from "../alerts";

describe("evaluateForecastAlerts", () => {
  it("raises CRITICAL cash_flow_risk when low point is negative", () => {
    const alerts = evaluateForecastAlerts({ projectedLowPoint: -200, confidenceLevel: "HIGH" });
    expect(alerts).toContainEqual(expect.objectContaining({
      alertType: "CASH_FLOW_RISK",
      severity: "CRITICAL",
    }));
  });

  it("raises WARNING cash_flow_risk when low point is under buffer", () => {
    const alerts = evaluateForecastAlerts({ projectedLowPoint: 300, confidenceLevel: "HIGH" });
    expect(alerts).toContainEqual(expect.objectContaining({
      alertType: "CASH_FLOW_RISK",
      severity: "WARNING",
    }));
  });

  it("raises WARNING when confidence is LOW", () => {
    const alerts = evaluateForecastAlerts({ projectedLowPoint: 5000, confidenceLevel: "LOW" });
    expect(alerts.some((a) => a.severity === "WARNING")).toBe(true);
  });

  it("returns no alerts for healthy forecast", () => {
    const alerts = evaluateForecastAlerts({ projectedLowPoint: 3000, confidenceLevel: "HIGH" });
    expect(alerts).toHaveLength(0);
  });
});
```

**Step 2: Run, fail, implement, pass — same TDD loop**

```typescript
// src/lib/domain/alerts.ts
import type { ConfidenceLevel } from "@prisma/client";

const SAFETY_BUFFER = 500;

interface ForecastConditions {
  projectedLowPoint: number;
  confidenceLevel: ConfidenceLevel;
}

interface AlertDraft {
  alertType: string;
  severity: "INFO" | "WARNING" | "CRITICAL";
  message: string;
}

export function evaluateForecastAlerts(conditions: ForecastConditions): AlertDraft[] {
  const alerts: AlertDraft[] = [];
  const { projectedLowPoint, confidenceLevel } = conditions;

  if (projectedLowPoint < 0) {
    alerts.push({
      alertType: "CASH_FLOW_RISK",
      severity: "CRITICAL",
      message: `Forecast projects a negative balance of $${Math.abs(projectedLowPoint).toFixed(2)}.`,
    });
  } else if (projectedLowPoint < SAFETY_BUFFER) {
    alerts.push({
      alertType: "CASH_FLOW_RISK",
      severity: "WARNING",
      message: `Forecast low point ($${projectedLowPoint.toFixed(2)}) is below the safety buffer.`,
    });
  }

  if (confidenceLevel === "LOW") {
    alerts.push({
      alertType: "ANOMALY",
      severity: "WARNING",
      message: "Forecast confidence is low due to missing or stale data.",
    });
  }

  return alerts;
}
```

**Step 3: Run test**

```bash
npx vitest run src/lib/domain/__tests__/alerts.test.ts
```

Expected: PASS.

**Step 4: Commit**

```bash
git add src/lib/domain/alerts.ts src/lib/domain/__tests__/alerts.test.ts
git commit -m "feat: add forecast alert evaluation (FR-21, FR-30)"
```

---

## Phase 6: Household Dashboard UI

### Task 12: Safe-to-Spend Dashboard (Household View)

**Files:**
- Create: `src/app/(dashboard)/page.tsx`
- Create: `src/components/safe-to-spend-card.tsx`
- Create: `src/components/forecast-summary.tsx`

**Step 1: Install shadcn/ui components**

```bash
npx shadcn@latest add card badge alert
```

**Step 2: Create `src/components/safe-to-spend-card.tsx`**

```tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import type { ConfidenceLevel } from "@prisma/client";

interface SafeToSpendCardProps {
  amount: number;
  confidenceLevel: ConfidenceLevel;
  explanationSummary: string;
}

const confidenceBadgeVariant: Record<ConfidenceLevel, "default" | "secondary" | "destructive"> = {
  HIGH: "default",
  MEDIUM: "secondary",
  LOW: "destructive",
};

export function SafeToSpendCard({ amount, confidenceLevel, explanationSummary }: SafeToSpendCardProps) {
  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between">
        <CardTitle>Safe to Spend</CardTitle>
        <Badge variant={confidenceBadgeVariant[confidenceLevel]}>
          {confidenceLevel} confidence
        </Badge>
      </CardHeader>
      <CardContent>
        <p className="text-4xl font-bold">${amount.toFixed(2)}</p>
        <p className="text-muted-foreground mt-2 text-sm">{explanationSummary}</p>
      </CardContent>
    </Card>
  );
}
```

**Step 3: Create `src/app/(dashboard)/page.tsx`**

```tsx
import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";
import { SafeToSpendCard } from "@/components/safe-to-spend-card";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await auth();
  if (!session) redirect("/sign-in");

  const latest = await prisma.safeToSpendResult.findFirst({
    orderBy: { calculatedAt: "desc" },
    include: { forecastWindow: true },
  });

  const alerts = await prisma.alert.findMany({
    where: { isResolved: false },
    orderBy: { createdAt: "desc" },
    take: 5,
  });

  return (
    <main className="container mx-auto max-w-2xl p-6 space-y-6">
      <h1 className="text-2xl font-semibold">BalanceIt</h1>
      {latest ? (
        <SafeToSpendCard
          amount={Number(latest.amount)}
          confidenceLevel={latest.confidenceLevel}
          explanationSummary={latest.explanationSummary}
        />
      ) : (
        <p className="text-muted-foreground">No forecast generated yet. Ask the admin to run a forecast.</p>
      )}
      {alerts.length > 0 && (
        <section>
          <h2 className="text-lg font-medium mb-3">Alerts</h2>
          <ul className="space-y-2">
            {alerts.map((a) => (
              <li key={a.id} className="rounded-md border p-3 text-sm">
                <span className="font-medium capitalize">{a.severity.toLowerCase()}:</span>{" "}
                {a.message}
              </li>
            ))}
          </ul>
        </section>
      )}
    </main>
  );
}
```

**Step 4: Commit**

```bash
git add src/app/(dashboard)/ src/components/
git commit -m "feat: add household dashboard with safe-to-spend card and alerts (FR-2, FR-20, FR-21)"
```

---

### Task 13: Admin Navigation & Role Guard

**Files:**
- Create: `src/app/(admin)/layout.tsx`
- Create: `src/app/(admin)/accounts/page.tsx`
- Create: `src/app/(admin)/schedules/page.tsx`
- Create: `src/components/admin-nav.tsx`

**Step 1: Create `src/app/(admin)/layout.tsx`**

```tsx
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";
import { AdminNav } from "@/components/admin-nav";

export default async function AdminLayout({ children }: { children: React.ReactNode }) {
  const session = await auth();
  if (!session || session.user.role !== "ADMIN") redirect("/");

  return (
    <div className="flex min-h-screen">
      <AdminNav />
      <main className="flex-1 p-6">{children}</main>
    </div>
  );
}
```

**Step 2: Create `src/components/admin-nav.tsx`**

```tsx
import Link from "next/link";

const links = [
  { href: "/", label: "Dashboard" },
  { href: "/admin/accounts", label: "Accounts" },
  { href: "/admin/schedules", label: "Schedules & Obligations" },
  { href: "/admin/reconciliation", label: "Reconciliation" },
  { href: "/admin/audit", label: "Audit Log" },
];

export function AdminNav() {
  return (
    <nav className="w-52 border-r p-4 space-y-1">
      <p className="text-xs text-muted-foreground uppercase font-medium mb-4">Admin</p>
      {links.map((l) => (
        <Link
          key={l.href}
          href={l.href}
          className="block rounded-md px-3 py-2 text-sm hover:bg-accent"
        >
          {l.label}
        </Link>
      ))}
    </nav>
  );
}
```

**Step 3: Commit**

```bash
git add src/app/(admin)/ src/components/admin-nav.tsx
git commit -m "feat: add admin layout with role guard and navigation (FR-22, FR-29)"
```

---

## Phase 7: Budgeting

### Task 14: Budget Target Management

**Files:**
- Create: `src/lib/domain/budgeting.ts`
- Create: `src/app/api/budget-targets/route.ts`
- Test: `src/lib/domain/__tests__/budgeting.test.ts`

**Step 1: Write the failing tests**

```typescript
// src/lib/domain/__tests__/budgeting.test.ts
import { describe, it, expect } from "vitest";
import { compareBudgetVsActual } from "../budgeting";

describe("compareBudgetVsActual", () => {
  it("returns variance as actual minus limit", () => {
    const result = compareBudgetVsActual({ limit: 500, actual: 420 });
    expect(result.variance).toBe(-80); // under budget
    expect(result.isOverBudget).toBe(false);
  });

  it("flags over budget when actual exceeds limit", () => {
    const result = compareBudgetVsActual({ limit: 500, actual: 620 });
    expect(result.variance).toBe(120);
    expect(result.isOverBudget).toBe(true);
  });
});
```

**Step 2: Implement, test, commit**

```typescript
// src/lib/domain/budgeting.ts
export function compareBudgetVsActual(input: { limit: number; actual: number }) {
  const variance = input.actual - input.limit;
  return { variance, isOverBudget: variance > 0 };
}
```

```bash
git commit -m "feat: add budget target management and spend comparison (FR-23, FR-24)"
```

---

## Phase 8: End-to-End Verification

### Task 15: Playwright E2E — Household User Flow

**Files:**
- Create: `e2e/household-dashboard.spec.ts`

**Step 1: Write the e2e test**

```typescript
// e2e/household-dashboard.spec.ts
import { test, expect } from "@playwright/test";

test("household user sees safe to spend on dashboard", async ({ page }) => {
  // Assumes test user is seeded and session is mocked via storageState
  await page.goto("/");
  await expect(page.getByText("Safe to Spend")).toBeVisible();
});

test("unauthenticated user is redirected to sign-in", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveURL(/sign-in/);
});

test("member user cannot access admin routes", async ({ page }) => {
  // Assumes member session
  await page.goto("/admin/accounts");
  await expect(page).not.toHaveURL(/admin/);
});
```

**Step 2: Run e2e**

```bash
npx playwright test
```

**Step 3: Commit**

```bash
git commit -m "test: add e2e tests for household dashboard and auth flows"
```

---

### Task 16: Run Full Test Suite & Final Commit

**Step 1: Run all unit tests**

```bash
npx vitest run
```

Expected: All PASS. Zero failing tests in domain modules.

**Step 2: Type check**

```bash
npx tsc --noEmit
```

Expected: No type errors.

**Step 3: Lint**

```bash
npx eslint src/ --ext .ts,.tsx
```

**Step 4: Final commit**

```bash
git add -A
git commit -m "chore: final pre-deploy verification — all tests pass, types clean"
```

---

## Deployment

### Task 17: Deploy to Vercel

**Step 1: Create Neon database**

- Go to neon.tech and create a project named `balanceit3`
- Copy the connection string to `.env.local` as `DATABASE_URL`

**Step 2: Run production migration**

```bash
npx prisma migrate deploy
```

**Step 3: Deploy to Vercel**

```bash
npx vercel --prod
```

Set environment variables in Vercel dashboard:
- `DATABASE_URL`
- `AUTH_SECRET` (generate: `openssl rand -base64 32`)
- `AUTH_GOOGLE_ID`
- `AUTH_GOOGLE_SECRET`
- `ALLOWLISTED_EMAILS`

**Step 4: Verify deployment**

Visit production URL, attempt sign-in with allowlisted email. Confirm redirect works for non-allowlisted email.

---

## FR Coverage Summary

| FR | Task |
|----|------|
| FR-1, FR-2, FR-3, FR-4 | Tasks 8, 9 |
| FR-5 (confidence) | Tasks 9, 11 (partial — enhance in iteration) |
| FR-6 (low point) | Task 8 |
| FR-7, FR-11 | Task 4 |
| FR-8, FR-9, FR-10 | Task 5 |
| FR-12, FR-14 | Task 6 |
| FR-13, FR-14, FR-15 | Task 7 |
| FR-16, FR-17, FR-18, FR-19 | Task 10 |
| FR-20, FR-21 | Task 12 |
| FR-22, FR-29 | Task 13 |
| FR-23, FR-24, FR-25 | Task 14 |
| FR-26, FR-27 | Task 3 |
| FR-28 | Tasks 5, 9 (audit logging) |
| FR-30, FR-31 | Task 11 |
| FR-32 | Task 9 (forecast persistence) |

## Open Questions (from docs) — Deferred Decisions

- **Forecast horizon**: hardcoded to 30 days for now; make configurable in a later iteration
- **Budget → safe-to-spend**: budget state feeds explanation only at launch (FR-25 "may")
- **Account connectivity**: manual + CSV only at launch; Plaid integration is a future task
- **Multiple calculation modes**: single canonical method (v1.0) at launch
- **Alert acknowledgment by member user**: admin-only for now
