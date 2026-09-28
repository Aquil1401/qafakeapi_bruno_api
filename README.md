# 🚀 QAFakeAPI — Enterprise Bruno API Automation Framework

[![QAFakeAPI CI/CD](https://github.com/Aquil1401/qafakeapi_bruno_api/actions/workflows/api-automation-ci.yml/badge.svg)](https://github.com/Aquil1401/qafakeapi_bruno_api/actions/workflows/api-automation-ci.yml)
[![Bruno](https://img.shields.io/badge/API%20Client-Bruno-yellow?logo=bruno)](https://usebruno.com)
[![NodeJS](https://img.shields.io/badge/Node-v20%2B-green?logo=node.js)](https://nodejs.org)

Comprehensive, production-grade REST API Test Automation Framework designed for [QAFakeAPI Sandbox](https://qafakeapi.ziaratechqlabs.in/playground) using **Bruno** (`.bru` plain text format) and `@usebruno/cli`.

---

## 📌 Features

- **100% Native Plain-Text `.bru` Format**: Version-controlled, human-readable, and free from JSON merge conflicts.
- **Dynamic Token & ID Chaining**: Automatically extracts JWT Bearer token on login and propagates to protected CRUD operations.
- **Zero-Flakiness Guarantee**: Automated pre-run and post-run database seed reset (`POST /api/v1/reset`).
- **Complete Endpoint Coverage (17+ Tests)**: Covers Auth, Products Catalog, User Management, Relational Orders, Chaos/Negative Simulators, and System Health.
- **Multi-Environment Support**: Seamlessly switch between `Production` (Live) and `Local` sandbox environments.
- **CI/CD Pipeline Ready**: GitHub Actions workflow generating HTML & JUnit test reports with artifact retention.

---

## 📂 Framework Architecture

```text
qafakeapi_bruno_api/
├── .github/
│   └── workflows/
│       └── api-automation-ci.yml        # Automated CI/CD execution pipeline
├── 00-Setup/
│   └── 01-Reset-Database-Pre.bru        # Pre-test hook: Reverts sandbox to seed state
├── 01-Authentication/
│   ├── 01-Login-Success.bru             # Positive login & dynamic JWT capture
│   ├── 02-Login-Invalid-Credentials.bru # Negative auth test (401 verification)
│   ├── 03-Get-Current-Profile.bru       # Protected /auth/me with Bearer token
│   └── 04-Get-Profile-Unauthorized.bru  # Negative auth test (Missing token 401)
├── 02-Products/
│   ├── 01-Get-All-Products.bru          # Pagination & catalog validation
│   ├── 02-Search-Products-Keyword.bru   # Query param search (q=Mouse)
│   ├── 03-Filter-Products-Category.bru  # Category filtering (category=electronics)
│   ├── 04-Get-Product-By-ID.bru         # Single entity lookup (id=1)
│   ├── 05-Create-Product.bru            # POST item + capture createdProductId
│   ├── 06-Update-Product.bru            # PUT mutation on createdProductId
│   ├── 07-Delete-Product.bru            # DELETE on createdProductId
│   └── 08-Get-Deleted-Product.bru       # Negative test (404 on deleted ID)
├── 03-Users/
│   ├── 01-List-Users.bru                # User catalog & schema verification
│   └── 02-Register-User.bru             # Dynamic user registration (pre-request script)
├── 04-Orders/
│   ├── 01-List-Orders.bru               # Relational orders list & line items
│   ├── 02-Get-Order-By-ID.bru           # Single order lookup (id=101)
│   └── 03-Get-Invalid-Order-404.bru     # Negative test (404 on invalid order ID)
├── 05-Chaos-Simulators/
│   ├── 01-Simulate-404-Not-Found.bru    # Error simulation (404)
│   ├── 02-Simulate-500-Server-Error.bru # Chaos simulation (500)
│   ├── 03-Simulate-429-Rate-Limit.bru   # Chaos simulation (429)
│   └── 04-Simulate-Network-Delay.bru    # Latency simulation (1500ms delay)
├── 06-System/
│   ├── 01-Health-Check.bru              # Health and system counters
│   ├── 02-Revoke-Session-Logout.bru     # Session invalidation
│   └── 03-Reset-Database-Post.bru       # Post-test cleanup hook
├── environments/
│   ├── Production.bru                   # Points to live QAFakeAPI server
│   └── Local.bru                        # Points to http://localhost:3000
├── bruno.json                           # Bruno collection descriptor
├── package.json                         # NPM scripts & Bruno CLI dependency
└── .gitignore
```

---

## ⚡ Quick Start

### 1. Prerequisites
- **Node.js**: v18.0.0 or higher
- **Bruno Desktop App** *(Optional, for GUI exploration)*: [Download from usebruno.com](https://www.usebruno.com/downloads)

### 2. Installation
Clone or navigate to this repository and install dependencies:
```bash
npm install
```

### 3. Run Tests via CLI

| Command | Description |
| :--- | :--- |
| `npm test` | Runs the complete suite against **Production** |
| `npm run test:local` | Runs tests against **Local Sandbox** (`localhost:3000`) |
| `npm run test:report` | Runs tests and outputs **HTML** (`reports/report.html`) & **JUnit XML** (`reports/results.xml`) |
| `npm run test:auth` | Runs only the **01-Authentication** folder |
| `npm run test:products` | Runs only the **02-Products** folder |
| `npm run test:users` | Runs only the **03-Users** folder |
| `npm run test:orders` | Runs only the **04-Orders** folder |
| `npm run test:chaos` | Runs only the **05-Chaos-Simulators** folder |

---

## 🔄 Dynamic Variable Chaining

### 1. JWT Bearer Token Chaining
In `01-Authentication/01-Login-Success.bru`:
```javascript
script:post-response {
  const data = res.getBody();
  if (data && data.token) {
    bru.setEnvVar("authToken", data.token);
  }
}
```
Subsequent protected requests use:
```bru
headers {
  Authorization: Bearer {{authToken}}
}
```

### 2. Dynamic Entity ID Chaining
In `02-Products/05-Create-Product.bru`:
```javascript
script:post-response {
  const data = res.getBody();
  if (data && data.id) {
    bru.setEnvVar("createdProductId", data.id);
  }
}
```
Subsequent `PUT`, `DELETE`, and `GET 404` tests dynamically target `{{baseUrl}}/api/v1/products/{{createdProductId}}`.

---

## 🚢 CI/CD Pipeline (GitHub Actions)

The workflow `.github/workflows/api-automation-ci.yml` is pre-configured to:
1. Trigger on every `git push` and `pull_request` to `main`.
2. Run daily synthetic health checks via GitHub cron.
3. Allow manual on-demand execution via `workflow_dispatch` with environment selection (`Production` / `Local`).
4. Generate and archive HTML and JUnit test reports under the GitHub Actions **Artifacts** tab.
5. Publish a high-level test status directly to the job's **Step Summary**.
