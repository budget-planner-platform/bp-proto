# budget-planner-proto — Agent Context

> For global architecture, principles, and service overview see `../ARCHITECTURE.md`.

## What this repo is

Single source of truth for all gRPC proto schemas across the Budget Planner project.
No business logic lives here — only `.proto` definitions and Buf configuration.

## Stack

- **Schema registry:** Buf Schema Registry (BSR)
- **Config:** `buf.yaml`, `buf.gen.yaml`
- **CI:** GitHub Actions → pushes new module version to BSR on every merge to `main`

## Repo structure

```
budget-planner-proto/
├── buf.yaml
├── buf.gen.yaml
├── buf.lock
├── .github/
│   └── workflows/
│       └── push-bsr.yml
└── proto/
    └── budgetplanner/
        ├── auth/v1/
        │   └── auth.proto
        ├── user/v1/
        │   └── user.proto
        ├── transaction/v1/
        │   └── transaction.proto
        ├── category/v1/
        │   └── category.proto
        ├── budget/v1/
        │   └── budget.proto
        ├── payment/v1/
        │   └── payment.proto
        ├── ovdp/v1/
        │   └── ovdp.proto
        ├── currency/v1/
        │   └── currency.proto
        └── notification/v1/
            └── notification.proto
```

## Naming conventions

- **Package:** `budgetplanner.<service>.v1`
- **Go package option:** `budgetplanner/<service>/v1;<service>v1`
- **File:** `snake_case.proto`
- **Messages:** `PascalCase`
- **RPCs:** `PascalCase` verbs — `GetUser`, `CreateTransaction`, `ListBudgets`
- **Enums:** `SCREAMING_SNAKE_CASE` values, prefixed with enum name — `TRANSACTION_TYPE_INCOME`
- **Fields:** `snake_case`

## Versioning

- All packages are `v1` until a breaking change is required
- Breaking changes → new directory `v2/`, old version stays until all consumers migrate
- Use `buf breaking` in CI to catch accidental breaking changes

## Key rules

- Every `.proto` file must have `syntax = "proto3"`
- Every service RPC must have a unique `Request` and `Response` message — no reuse
- Use `google.protobuf.Timestamp` for all datetime fields
- Use `string` for IDs (UUIDs)
- Monetary amounts: `int64` in cents (USD) — never `float` or `double`
- Pagination: `page_size` (int32) + `page_token` (string) on list requests

## How consumers use this repo

Each service pulls generated client code directly from BSR — no local proto files:

```bash
# Node.js
npm install @buf/budgetplanner_proto

# Go
go get buf.build/gen/go/budgetplanner/proto

# Java (build.gradle)
# implementation("build.buf.gen:budgetplanner_proto_java:...")

# Python
pip install buf-generated-budgetplanner-proto
```

## Local development

```bash
# Install buf
brew install bufbuild/buf/buf

# Lint
buf lint

# Check for breaking changes against BSR
buf breaking --against "https://buf.build/budgetplanner/proto"

# Format
buf format -w
```

## CI — push-bsr.yml

Triggers on push to `main`. Steps:
1. `buf lint` — fails PR if schema is invalid
2. `buf breaking` — fails PR if breaking change detected
3. `buf push` — pushes new module version to BSR (only on merge to `main`)

## gRPC methods per service (summary)

| Service | RPCs |
|---------|------|
| auth | `ValidateToken`, `GetUserInfo` |
| user | `GetUser`, `CreateUser`, `UpdateUser`, `DeleteUser` |
| transaction | `CreateTransaction`, `GetTransaction`, `ListTransactions`, `UpdateTransaction`, `DeleteTransaction` |
| category | `ListCategories`, `CreateCategory`, `UpdateCategory`, `DeleteCategory` |
| budget | `CreateBudget`, `GetBudget`, `ListBudgets`, `GetBudgetStatus` |
| payment | `CreateSchedule`, `GetSchedule`, `ListSchedules`, `CalculateAmortization` |
| ovdp | `AddBond`, `GetBond`, `ListBonds`, `GetPortfolioReport`, `SimulatePortfolio` |
| currency | `GetRate`, `Convert`, `GetHistoricalRate` |
| notification | `CreateNotification`, `ListNotifications`, `MarkAsRead` |