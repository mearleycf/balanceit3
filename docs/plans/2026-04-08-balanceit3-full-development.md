# BalanceIt3 Full Development Implementation Plan

> **For Claude:** Use the ai-team-orchestrator agent to implement this plan task-by-task.

**Goal:** Build a private household cash-flow forecasting app that surfaces a trustworthy, explainable `safe to spend` number from account balances, income schedules, and recurring obligations.

**Architecture:** Astro 5.x frontend (with React 19 islands) + FastAPI backend (Python 3.11+). Domain logic isolated in `backend/app/domain/`. Astro pages handle data fetching server-side; React components receive data as props. PostgreSQL via Neon, managed with SQLAlchemy 2.0 + Alembic.

**Tech Stack:**
- Frontend: Astro 5.x, React 19, Tailwind CSS 4, shadcn/ui, Zod, React Hook Form
- Backend: FastAPI, Python 3.11+, SQLAlchemy 2.0 (async), Alembic, Pydantic v2, uv, asyncpg
- Auth: Google OAuth via FastAPI, JWT, allowlisted emails only
- Database: PostgreSQL via Neon (serverless)
- Frontend testing: Vitest + Testing Library + Playwright
- Backend testing: pytest + httpx + pytest-asyncio
- Deploy: Vercel (frontend), Railway or Render (backend)

---

## Phase 1: Project Foundation

### Task 1: Initialize Project Structure

**Files:**
- Create: `web/` (Astro frontend)
- Create: `backend/` (FastAPI backend)
- Create: `.env.example`

**Step 1: Create Astro frontend**

```bash
mkdir web && cd web
npm create astro@latest . -- --template minimal --typescript strict --no-git --install
npx astro add react tailwind
```

**Step 2: Install frontend dependencies**

```bash
cd web
npm install zod react-hook-form @hookform/resolvers
npm install -D vitest @vitest/ui @testing-library/react @testing-library/jest-dom jsdom @vitejs/plugin-react
npx playwright install --with-deps chromium
npx shadcn@latest init
```

**Step 3: Create `web/vitest.config.ts`**

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test/setup.ts"],
    globals: true,
  },
});
```

**Step 4: Create `web/src/test/setup.ts`**

```typescript
import "@testing-library/jest-dom";
```

**Step 5: Initialize FastAPI backend**

```bash
mkdir -p backend/app/{api/v1,domain,models,schemas,core} backend/tests backend/alembic
cd backend
uv init --python 3.11
uv add fastapi uvicorn[standard] sqlalchemy[asyncio] asyncpg alembic pydantic pydantic-settings python-jose[cryptography] httpx google-auth
uv add --dev pytest pytest-asyncio pytest-cov mypy ruff
```

**Step 6: Create `backend/pyproject.toml` tool config**

```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]

[tool.mypy]
python_version = "3.11"
strict = true

[tool.ruff]
line-length = 88
```

**Step 7: Create `.env.example`**

```bash
# Database
DATABASE_URL=postgresql+asyncpg://user:pass@host/balanceit3

# Auth
SECRET_KEY=generate-with-openssl-rand-hex-32
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=10080

# Google OAuth
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Allowlist (comma-separated)
ALLOWLISTED_EMAILS=admin@gmail.com,member@gmail.com

# Frontend
PUBLIC_API_URL=http://localhost:8000
```

**Step 8: Commit**

```bash
git add -A
git commit -m "chore: initialize Astro + FastAPI project structure"
```

---

### Task 2: SQLAlchemy Models — All Domain Entities

**Files:**
- Create: `backend/app/models/base.py`
- Create: `backend/app/models/user.py`
- Create: `backend/app/models/account.py`
- Create: `backend/app/models/transaction.py`
- Create: `backend/app/models/income_schedule.py`
- Create: `backend/app/models/recurring_obligation.py`
- Create: `backend/app/models/forecast.py`
- Create: `backend/app/models/reconciliation.py`
- Create: `backend/app/models/alert.py`
- Create: `backend/app/models/audit.py`
- Create: `backend/app/models/budget.py`

**Step 1: Create `backend/app/models/base.py`**

```python
from sqlalchemy.orm import DeclarativeBase
import uuid
from sqlalchemy import String
from sqlalchemy.orm import mapped_column, MappedColumn


class Base(DeclarativeBase):
    pass


def new_uuid() -> str:
    return str(uuid.uuid4())
```

**Step 2: Create domain models** (one file per entity, matching `balanceit3_docs/02_domain_model.md`)

`backend/app/models/user.py`:
```python
from sqlalchemy import String, Boolean, Enum as SAEnum
from sqlalchemy.orm import Mapped, mapped_column
from .base import Base, new_uuid
import enum


class UserRole(str, enum.Enum):
    ADMIN = "admin"
    MEMBER = "member"


class UserStatus(str, enum.Enum):
    ACTIVE = "active"
    DISABLED = "disabled"


class HouseholdUser(Base):
    __tablename__ = "household_users"

    id: Mapped[str] = mapped_column(String, primary_key=True, default=new_uuid)
    email: Mapped[str] = mapped_column(String, unique=True, nullable=False)
    role: Mapped[UserRole] = mapped_column(SAEnum(UserRole), default=UserRole.MEMBER)
    is_allowlisted: Mapped[bool] = mapped_column(Boolean, default=False)
    status: Mapped[UserStatus] = mapped_column(SAEnum(UserStatus), default=UserStatus.ACTIVE)
```

(Repeat pattern for Account, Transaction, IncomeSchedule, RecurringObligation, ForecastWindow, SafeToSpendResult, ReconciliationRecord, Alert, AuditEvent, BudgetTarget — following the domain model exactly.)

**Step 3: Create initial Alembic migration**

```bash
cd backend
uv run alembic init alembic
uv run alembic revision --autogenerate -m "init: all domain models"
uv run alembic upgrade head
```

**Step 4: Verify**

```bash
uv run python -c "from app.models.user import HouseholdUser; print('models OK')"
```

**Step 5: Commit**

```bash
git add backend/
git commit -m "feat: add SQLAlchemy models for all domain entities"
```

---

### Task 3: Google OAuth Authentication with Allowlist

**Files:**
- Create: `backend/app/core/auth.py`
- Create: `backend/app/core/config.py`
- Create: `backend/app/api/v1/auth.py`
- Create: `backend/tests/test_auth.py`

**Step 1: Write the failing tests**

```python
# backend/tests/test_auth.py
import pytest
from app.core.auth import is_allowlisted, create_access_token, verify_token


def test_allowlisted_email_returns_true():
    assert is_allowlisted("user@gmail.com", "user@gmail.com,other@gmail.com") is True


def test_non_allowlisted_email_returns_false():
    assert is_allowlisted("stranger@gmail.com", "user@gmail.com") is False


def test_allowlist_is_case_insensitive():
    assert is_allowlisted("USER@GMAIL.COM", "user@gmail.com") is True


def test_empty_allowlist_returns_false():
    assert is_allowlisted("user@gmail.com", "") is False


def test_token_roundtrip():
    token = create_access_token({"sub": "user@gmail.com", "role": "admin"})
    payload = verify_token(token)
    assert payload["sub"] == "user@gmail.com"
    assert payload["role"] == "admin"
```

**Step 2: Run to confirm failure**

```bash
cd backend && uv run pytest tests/test_auth.py -v
```

Expected: FAIL — module not found.

**Step 3: Create `backend/app/core/config.py`**

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    database_url: str
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 10080
    google_client_id: str
    google_client_secret: str
    allowlisted_emails: str

    class Config:
        env_file = ".env"


settings = Settings()
```

**Step 4: Create `backend/app/core/auth.py`**

```python
from datetime import datetime, timedelta
from jose import JWTError, jwt
from .config import settings


def is_allowlisted(email: str, allowlist_env: str) -> bool:
    if not allowlist_env:
        return False
    allowed = [e.strip().lower() for e in allowlist_env.split(",")]
    return email.lower() in allowed


def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.secret_key, algorithm=settings.algorithm)


def verify_token(token: str) -> dict:
    return jwt.decode(token, settings.secret_key, algorithms=[settings.algorithm])
```

**Step 5: Run tests to confirm pass**

```bash
uv run pytest tests/test_auth.py -v
```

Expected: PASS (5 tests).

**Step 6: Create `backend/app/api/v1/auth.py`** — Google OAuth callback + allowlist check

```python
from fastapi import APIRouter, HTTPException
from google.oauth2 import id_token
from google.auth.transport import requests as google_requests
from app.core.auth import is_allowlisted, create_access_token
from app.core.config import settings

router = APIRouter(prefix="/auth", tags=["auth"])


@router.post("/google")
async def google_auth(token: str):
    try:
        idinfo = id_token.verify_oauth2_token(
            token, google_requests.Request(), settings.google_client_id
        )
    except ValueError:
        raise HTTPException(status_code=401, detail="Invalid Google token")

    email = idinfo.get("email", "")
    if not is_allowlisted(email, settings.allowlisted_emails):
        raise HTTPException(status_code=403, detail="Not authorized")

    access_token = create_access_token({"sub": email, "role": "member"})
    return {"access_token": access_token, "token_type": "bearer"}
```

**Step 7: Commit**

```bash
git commit -m "feat: add Google OAuth auth with allowlist enforcement (FR-26, FR-27)"
```

---

## Phase 2: Accounts & Transaction Ingestion

### Task 4: Account Management

**Files:**
- Create: `backend/app/domain/accounts.py`
- Create: `backend/app/api/v1/accounts.py`
- Test: `backend/tests/test_accounts.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_accounts.py
from app.domain.accounts import build_account, toggle_forecast_inclusion


def test_build_account_defaults_to_manual_and_included():
    account = build_account(
        display_name="Chase Checking",
        account_type="checking",
        current_balance=5000.0,
    )
    assert account["provider_type"] == "manual"
    assert account["include_in_forecast"] is True


def test_toggle_forecast_inclusion():
    result = toggle_forecast_inclusion(include=False)
    assert result["include_in_forecast"] is False
```

**Step 2: Implement `backend/app/domain/accounts.py`**

```python
def build_account(display_name: str, account_type: str, current_balance: float, **kwargs) -> dict:
    return {
        "display_name": display_name,
        "account_type": account_type,
        "current_balance": current_balance,
        "provider_type": "manual",
        "include_in_forecast": True,
        **kwargs,
    }


def toggle_forecast_inclusion(include: bool) -> dict:
    return {"include_in_forecast": include}
```

**Step 3: Run, verify pass, commit**

```bash
uv run pytest tests/test_accounts.py -v
git commit -m "feat: add account management domain logic and API (FR-7, FR-11)"
```

---

### Task 5: CSV Transaction Import

**Files:**
- Create: `backend/app/domain/ingestion.py`
- Create: `backend/app/api/v1/transactions.py`
- Test: `backend/tests/test_ingestion.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_ingestion.py
import pytest
from app.domain.ingestion import parse_transaction_csv, detect_duplicate


def test_parse_csv_outflow():
    csv = "date,amount,description\n2024-01-15,-50.00,Grocery Store"
    result = parse_transaction_csv(csv, account_id="acc-1")
    assert result[0]["direction"] == "outflow"
    assert result[0]["amount"] == 50.0
    assert result[0]["source_type"] == "csv"


def test_parse_csv_inflow():
    csv = "date,amount,description\n2024-01-15,2500.00,Paycheck"
    result = parse_transaction_csv(csv, account_id="acc-1")
    assert result[0]["direction"] == "inflow"


def test_parse_csv_missing_amount_raises():
    csv = "date,amount,description\n2024-01-15,,Grocery"
    with pytest.raises(ValueError):
        parse_transaction_csv(csv, account_id="acc-1")


def test_detect_duplicate_matches_source_reference():
    existing = [{"source_reference": "csv:2024-01-15:grocery:50.0"}]
    incoming = {"source_reference": "csv:2024-01-15:grocery:50.0"}
    assert detect_duplicate(incoming, existing) is True


def test_detect_duplicate_no_match():
    existing = [{"source_reference": "csv:2024-01-15:grocery:50.0"}]
    incoming = {"source_reference": "csv:2024-01-16:coffee:5.0"}
    assert detect_duplicate(incoming, existing) is False
```

**Step 2: Implement `backend/app/domain/ingestion.py`**

```python
from datetime import date
import csv
import io


def parse_transaction_csv(csv_text: str, account_id: str) -> list[dict]:
    reader = csv.DictReader(io.StringIO(csv_text.strip()))
    results = []
    for row in reader:
        raw_amount = row.get("amount", "").strip()
        if not raw_amount:
            raise ValueError(f"Missing amount in row: {row}")
        numeric = float(raw_amount)
        direction = "inflow" if numeric >= 0 else "outflow"
        amount = abs(numeric)
        description = row["description"].strip()
        posted_at = row["date"].strip()
        source_ref = f"csv:{posted_at}:{description.lower().replace(' ', '-')}:{amount}"
        results.append({
            "account_id": account_id,
            "posted_at": posted_at,
            "amount": amount,
            "direction": direction,
            "description": description,
            "source_type": "csv",
            "source_reference": source_ref,
        })
    return results


def detect_duplicate(incoming: dict, existing: list[dict]) -> bool:
    return any(e["source_reference"] == incoming["source_reference"] for e in existing)
```

**Step 3: Run, verify pass, commit**

```bash
uv run pytest tests/test_ingestion.py -v
git commit -m "feat: add CSV import with deduplication (FR-8, FR-9, FR-10)"
```

---

## Phase 3: Income Schedules & Recurring Obligations

### Task 6: Income Schedule Domain Logic

**Files:**
- Create: `backend/app/domain/income.py`
- Create: `backend/app/api/v1/income_schedules.py`
- Test: `backend/tests/test_income.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_income.py
from datetime import date
from app.domain.income import next_occurrence_after


def test_monthly_advances_one_month():
    base = date(2024, 1, 15)
    after = date(2024, 1, 20)
    result = next_occurrence_after(base, "monthly", after)
    assert result == date(2024, 2, 15)


def test_biweekly_advances_14_days():
    base = date(2024, 1, 1)
    after = date(2024, 1, 8)
    result = next_occurrence_after(base, "biweekly", after)
    assert result == date(2024, 1, 15)


def test_returns_base_when_already_after():
    base = date(2024, 1, 25)
    after = date(2024, 1, 20)
    result = next_occurrence_after(base, "monthly", after)
    assert result == base
```

**Step 2: Implement `backend/app/domain/income.py`**

```python
from datetime import date, timedelta
from dateutil.relativedelta import relativedelta


def next_occurrence_after(base: date, cadence: str, after: date) -> date:
    current = base
    while current <= after:
        if cadence == "weekly":
            current += timedelta(weeks=1)
        elif cadence == "biweekly":
            current += timedelta(weeks=2)
        elif cadence == "semimonthly":
            current += timedelta(days=15)
        elif cadence == "monthly":
            current += relativedelta(months=1)
        elif cadence == "annual":
            current += relativedelta(years=1)
        else:
            break  # custom: caller handles
    return current
```

**Step 3: Run, verify pass, commit**

```bash
uv run pytest tests/test_income.py -v
git commit -m "feat: add income schedule domain logic and API (FR-12, FR-14)"
```

---

### Task 7: Recurring Obligations API

Follow same pattern as Task 6. Obligations use `next_due_date` instead of `next_expected_date`. Test that inactive obligations are excluded from forecast queries.

```bash
git commit -m "feat: add recurring obligations API (FR-13, FR-14, FR-15)"
```

---

## Phase 4: Forecasting Core (North Star)

### Task 8: Cash-Flow Forecast Engine

**Files:**
- Create: `backend/app/domain/forecasting.py`
- Test: `backend/tests/test_forecasting.py`

**Step 1: Write the failing tests — deepest test suite in the project**

```python
# backend/tests/test_forecasting.py
from datetime import date
from app.domain.forecasting import build_forecast_window, ForecastInput, CashFlowEvent


def make_input(**kwargs):
    defaults = dict(
        base_balance=5000.0,
        starts_on=date(2024, 1, 1),
        ends_on=date(2024, 1, 31),
        income_events=[],
        obligation_events=[],
    )
    defaults.update(kwargs)
    return ForecastInput(**defaults)


def test_no_events_low_point_is_base_balance():
    result = build_forecast_window(make_input())
    assert result.projected_low_point == 5000.0
    assert result.projected_inflows == 0.0
    assert result.projected_outflows == 0.0


def test_income_event_in_window_adds_to_inflows():
    result = build_forecast_window(make_input(
        income_events=[CashFlowEvent(date=date(2024, 1, 15), amount=2500.0, label="Paycheck")]
    ))
    assert result.projected_inflows == 2500.0


def test_obligation_in_window_adds_to_outflows():
    result = build_forecast_window(make_input(
        obligation_events=[CashFlowEvent(date=date(2024, 1, 10), amount=1200.0, label="Rent")]
    ))
    assert result.projected_outflows == 1200.0


def test_low_point_is_minimum_running_balance():
    result = build_forecast_window(make_input(
        base_balance=3000.0,
        income_events=[CashFlowEvent(date=date(2024, 1, 20), amount=2000.0, label="Paycheck")],
        obligation_events=[
            CashFlowEvent(date=date(2024, 1, 5), amount=1200.0, label="Rent"),
            CashFlowEvent(date=date(2024, 1, 10), amount=400.0, label="Car"),
        ],
    ))
    # balance: 3000 → -1200 = 1800 → -400 = 1400 (lowest) → +2000 = 3400
    assert result.projected_low_point == 1400.0


def test_events_outside_window_are_excluded():
    result = build_forecast_window(make_input(
        income_events=[CashFlowEvent(date=date(2024, 2, 15), amount=2500.0, label="Paycheck")]
    ))
    assert result.projected_inflows == 0.0
```

**Step 2: Run to confirm failure**

```bash
uv run pytest tests/test_forecasting.py -v
```

Expected: FAIL — module not found.

**Step 3: Create `backend/app/domain/forecasting.py`**

```python
from dataclasses import dataclass
from datetime import date


@dataclass
class CashFlowEvent:
    date: date
    amount: float
    label: str


@dataclass
class ForecastInput:
    base_balance: float
    starts_on: date
    ends_on: date
    income_events: list[CashFlowEvent]
    obligation_events: list[CashFlowEvent]


@dataclass
class ForecastResult:
    starts_on: date
    ends_on: date
    base_balance: float
    projected_inflows: float
    projected_outflows: float
    projected_low_point: float


def build_forecast_window(input: ForecastInput) -> ForecastResult:
    in_window = lambda e: input.starts_on <= e.date <= input.ends_on

    inflows = [e for e in input.income_events if in_window(e)]
    outflows = [e for e in input.obligation_events if in_window(e)]

    total_inflows = sum(e.amount for e in inflows)
    total_outflows = sum(e.amount for e in outflows)

    events = sorted(
        [(e.date, e.amount) for e in inflows] +
        [(e.date, -e.amount) for e in outflows],
        key=lambda x: x[0]
    )

    running = input.base_balance
    low_point = running
    for _, delta in events:
        running += delta
        if running < low_point:
            low_point = running

    return ForecastResult(
        starts_on=input.starts_on,
        ends_on=input.ends_on,
        base_balance=input.base_balance,
        projected_inflows=total_inflows,
        projected_outflows=total_outflows,
        projected_low_point=low_point,
    )
```

**Step 4: Run to confirm pass**

```bash
uv run pytest tests/test_forecasting.py -v
```

Expected: PASS (5 tests).

**Step 5: Commit**

```bash
git commit -m "feat: add cash-flow forecast engine with low-point calculation (FR-1, FR-6)"
```

---

### Task 9: Safe-to-Spend Calculation

**Files:**
- Create: `backend/app/domain/safe_to_spend.py`
- Create: `backend/app/api/v1/forecast.py`
- Test: `backend/tests/test_safe_to_spend.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_safe_to_spend.py
from app.domain.safe_to_spend import calculate_safe_to_spend, SafeToSpendInput


def base_input(**kwargs):
    defaults = dict(
        projected_low_point=3000.0,
        safety_buffer=500.0,
        confidence_level="high",
        unreconciled_obligations=0.0,
    )
    defaults.update(kwargs)
    return SafeToSpendInput(**defaults)


def test_basic_calculation():
    result = calculate_safe_to_spend(base_input())
    assert result.amount == 2500.0


def test_never_negative():
    result = calculate_safe_to_spend(base_input(projected_low_point=400.0))
    assert result.amount == 0.0


def test_unreconciled_obligations_reduce_amount():
    result = calculate_safe_to_spend(base_input(unreconciled_obligations=300.0))
    assert result.amount == 2200.0


def test_low_confidence_preserved_in_output():
    result = calculate_safe_to_spend(base_input(confidence_level="low"))
    assert result.confidence_level == "low"


def test_explanation_includes_key_inputs():
    result = calculate_safe_to_spend(base_input())
    assert "3000" in result.explanation_summary
    assert "500" in result.explanation_summary
```

**Step 2: Implement `backend/app/domain/safe_to_spend.py`**

```python
from dataclasses import dataclass

METHOD_VERSION = "v1.0"


@dataclass
class SafeToSpendInput:
    projected_low_point: float
    safety_buffer: float
    confidence_level: str
    unreconciled_obligations: float


@dataclass
class SafeToSpendOutput:
    amount: float
    confidence_level: str
    method_version: str
    explanation_summary: str


def calculate_safe_to_spend(input: SafeToSpendInput) -> SafeToSpendOutput:
    raw = input.projected_low_point - input.safety_buffer - input.unreconciled_obligations
    amount = max(0.0, raw)

    parts = [
        f"Projected low point: ${input.projected_low_point:.2f}.",
        f"Safety buffer: ${input.safety_buffer:.2f}.",
    ]
    if input.unreconciled_obligations > 0:
        parts.append(f"Unreconciled obligations deducted: ${input.unreconciled_obligations:.2f}.")
    parts.append(f"Safe to spend: ${amount:.2f}.")

    return SafeToSpendOutput(
        amount=amount,
        confidence_level=input.confidence_level,
        method_version=METHOD_VERSION,
        explanation_summary=" ".join(parts),
    )
```

**Step 3: Run, verify pass, commit**

```bash
uv run pytest tests/test_safe_to_spend.py -v
git commit -m "feat: add safe-to-spend calculation (FR-1, FR-2, FR-3, FR-4)"
```

---

## Phase 5: Reconciliation & Alerts

### Task 10: Reconciliation Engine

**Files:**
- Create: `backend/app/domain/reconciliation.py`
- Test: `backend/tests/test_reconciliation.py`

**Step 1: Write failing tests**

```python
# backend/tests/test_reconciliation.py
from datetime import date
from app.domain.reconciliation import score_match, match_transaction_to_expected

tx = dict(posted_at=date(2024, 1, 15), amount=2500.0, direction="inflow")
expected = dict(next_expected_date=date(2024, 1, 15), expected_amount=2500.0, id="sched-1")


def test_exact_match_scores_high():
    assert score_match(tx, expected) > 0.8


def test_amount_mismatch_within_tolerance_scores_lower():
    score = score_match({**tx, "amount": 2400.0}, expected)
    assert 0 < score < 0.8


def test_amount_outside_tolerance_scores_zero():
    assert score_match({**tx, "amount": 500.0}, expected) == 0.0


def test_match_returns_matched_status():
    result = match_transaction_to_expected(tx, [expected])
    assert result["status"] == "matched"
    assert result["matched_id"] == "sched-1"


def test_no_match_returns_review():
    result = match_transaction_to_expected({**tx, "amount": 99.0}, [expected])
    assert result["status"] == "review"
```

**Step 2: Implement, run, verify, commit**

```bash
git commit -m "feat: add reconciliation matching engine (FR-16, FR-17)"
```

---

### Task 11: Alert Generation

**Files:**
- Create: `backend/app/domain/alerts.py`
- Test: `backend/tests/test_alerts.py`

```python
# backend/tests/test_alerts.py
from app.domain.alerts import evaluate_forecast_alerts

def test_negative_low_point_is_critical():
    alerts = evaluate_forecast_alerts(projected_low_point=-200.0, confidence_level="high")
    assert any(a["alert_type"] == "cash_flow_risk" and a["severity"] == "critical" for a in alerts)

def test_low_point_below_buffer_is_warning():
    alerts = evaluate_forecast_alerts(projected_low_point=300.0, confidence_level="high")
    assert any(a["severity"] == "warning" for a in alerts)

def test_low_confidence_raises_warning():
    alerts = evaluate_forecast_alerts(projected_low_point=5000.0, confidence_level="low")
    assert any(a["severity"] == "warning" for a in alerts)

def test_healthy_forecast_no_alerts():
    assert evaluate_forecast_alerts(projected_low_point=3000.0, confidence_level="high") == []
```

```bash
git commit -m "feat: add forecast alert evaluation (FR-21, FR-30)"
```

---

## Phase 6: Household Dashboard UI

### Task 12: Forecast API Endpoint

**Files:**
- Create: `backend/app/api/v1/forecast.py`

Thin route handler that:
1. Queries included accounts for base balance
2. Gets active income schedules and obligations
3. Projects next occurrences within 30-day window
4. Calls `build_forecast_window` → `calculate_safe_to_spend` → `evaluate_forecast_alerts`
5. Persists ForecastWindow + SafeToSpendResult
6. Creates AuditEvent
7. Returns forecast + safe-to-spend + alerts

```bash
git commit -m "feat: add forecast API endpoint (FR-1, FR-2, FR-4, FR-28)"
```

---

### Task 13: Safe-to-Spend Dashboard (Astro + React)

**Files:**
- Create: `web/src/pages/index.astro`
- Create: `web/src/components/SafeToSpendCard.tsx`
- Create: `web/src/components/AlertList.tsx`
- Test: `web/src/components/__tests__/SafeToSpendCard.test.tsx`

**Step 1: Install shadcn/ui components**

```bash
cd web && npx shadcn@latest add card badge alert
```

**Step 2: Create `SafeToSpendCard.tsx`**

```tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";

interface Props {
  amount: number;
  confidenceLevel: "high" | "medium" | "low";
  explanationSummary: string;
}

const badgeVariant = {
  high: "default",
  medium: "secondary",
  low: "destructive",
} as const;

export function SafeToSpendCard({ amount, confidenceLevel, explanationSummary }: Props) {
  return (
    <Card>
      <CardHeader className="flex flex-row items-center justify-between">
        <CardTitle>Safe to Spend</CardTitle>
        <Badge variant={badgeVariant[confidenceLevel]}>
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

**Step 3: Write component test**

```tsx
// web/src/components/__tests__/SafeToSpendCard.test.tsx
import { render, screen } from "@testing-library/react";
import { SafeToSpendCard } from "../SafeToSpendCard";

test("displays amount and confidence level", () => {
  render(<SafeToSpendCard amount={2500} confidenceLevel="high" explanationSummary="Test explanation" />);
  expect(screen.getByText("$2500.00")).toBeInTheDocument();
  expect(screen.getByText("high confidence")).toBeInTheDocument();
});

test("shows low confidence as destructive badge", () => {
  render(<SafeToSpendCard amount={0} confidenceLevel="low" explanationSummary="Low confidence" />);
  expect(screen.getByText("low confidence")).toBeInTheDocument();
});
```

**Step 4: Create `web/src/pages/index.astro`**

```astro
---
import Layout from '../layouts/Layout.astro';
import { SafeToSpendCard } from '../components/SafeToSpendCard';

// Fetch from backend API
const apiUrl = import.meta.env.PUBLIC_API_URL;
const res = await fetch(`${apiUrl}/api/v1/forecast/latest`, {
  headers: { Authorization: `Bearer ${Astro.cookies.get('token')?.value ?? ''}` }
});
const data = res.ok ? await res.json() : null;
---

<Layout title="BalanceIt">
  <main class="container mx-auto max-w-2xl p-6 space-y-6">
    <h1 class="text-2xl font-semibold">BalanceIt</h1>
    {data ? (
      <SafeToSpendCard
        client:load
        amount={data.safe_to_spend.amount}
        confidenceLevel={data.safe_to_spend.confidence_level}
        explanationSummary={data.safe_to_spend.explanation_summary}
      />
    ) : (
      <p class="text-muted-foreground">No forecast yet. Sign in as admin to generate one.</p>
    )}
  </main>
</Layout>
```

**Step 5: Run tests, commit**

```bash
cd web && npx vitest run
git commit -m "feat: add household dashboard with safe-to-spend card (FR-2, FR-20)"
```

---

### Task 14: Admin Layout & Role Guard

**Files:**
- Create: `web/src/layouts/AdminLayout.astro`
- Create: `web/src/pages/admin/index.astro`
- Create: `web/src/components/AdminNav.astro`
- Create: `web/src/middleware.ts`

Astro middleware checks JWT from cookie, decodes role. Non-admin accessing `/admin/*` redirects to `/`.

```bash
git commit -m "feat: add admin layout with role guard (FR-22, FR-29)"
```

---

## Phase 7: Budgeting

### Task 15: Budget Target Management

**Files:**
- Create: `backend/app/domain/budgeting.py`
- Test: `backend/tests/test_budgeting.py`

```python
# backend/tests/test_budgeting.py
from app.domain.budgeting import compare_budget_vs_actual

def test_under_budget():
    result = compare_budget_vs_actual(limit=500.0, actual=420.0)
    assert result["variance"] == -80.0
    assert result["is_over_budget"] is False

def test_over_budget():
    result = compare_budget_vs_actual(limit=500.0, actual=620.0)
    assert result["variance"] == 120.0
    assert result["is_over_budget"] is True
```

```bash
git commit -m "feat: add budget target management (FR-23, FR-24)"
```

---

## Phase 8: E2E & Deployment

### Task 16: Playwright E2E Tests

**Files:**
- Create: `web/e2e/dashboard.spec.ts`

```typescript
import { test, expect } from "@playwright/test";

test("dashboard shows safe to spend", async ({ page }) => {
  await page.goto("/");
  await expect(page.getByText("Safe to Spend")).toBeVisible();
});

test("unauthenticated redirects to sign-in", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveURL(/sign-in/);
});

test("member cannot access admin routes", async ({ page }) => {
  // member session
  await page.goto("/admin");
  await expect(page).not.toHaveURL(/admin/);
});
```

```bash
git commit -m "test: add e2e tests for dashboard and auth flows"
```

---

### Task 17: Deploy Configuration

**Files:**
- Create: `web/vercel.json`
- Create: `backend/railway.toml` (or `render.yaml`)
- Create: `.github/workflows/ci.yml`

**`web/vercel.json`:**
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "astro"
}
```

**`.github/workflows/ci.yml`** — runs pytest + vitest on every PR, blocks merge on failure.

```bash
git commit -m "chore: add Vercel + Railway deploy config and CI pipeline"
```

---

## FR Coverage Summary

| FR | Task |
|----|------|
| FR-1, FR-2, FR-3, FR-4 | Tasks 8, 9, 12 |
| FR-5 (confidence) | Tasks 9, 11 |
| FR-6 (low point) | Task 8 |
| FR-7, FR-11 | Task 4 |
| FR-8, FR-9, FR-10 | Task 5 |
| FR-12, FR-14 | Task 6 |
| FR-13, FR-14, FR-15 | Task 7 |
| FR-16, FR-17, FR-18, FR-19 | Task 10 |
| FR-20, FR-21 | Tasks 13, 11 |
| FR-22, FR-29 | Task 14 |
| FR-23, FR-24, FR-25 | Task 15 |
| FR-26, FR-27 | Task 3 |
| FR-28 | Tasks 5, 12 |
| FR-30, FR-31 | Task 11 |
| FR-32 | Task 12 |

## Open Questions (deferred)

- Plaid account connectivity: manual + CSV only at launch
- Forecast horizon: hardcoded 30 days, make configurable later
- Budget → safe-to-spend: explanatory overlay only at launch (FR-25 "may")
- Multiple calculation modes: single canonical method v1.0 at launch
