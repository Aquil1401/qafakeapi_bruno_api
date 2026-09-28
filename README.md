# QAFakeAPI Bruno API Automation Framework

[![API Automation Tests](https://github.com/Aquil1401/qafakeapi_bruno_api/actions/workflows/api-automation-ci.yml/badge.svg)](https://github.com/Aquil1401/qafakeapi_bruno_api/actions/workflows/api-automation-ci.yml)
![Tests Passed](https://img.shields.io/badge/Tests-25%2F25%20Passed-2ea44f?style=flat-square&logo=bruno&logoColor=white)
![Assertions](https://img.shields.io/badge/Assertions-94%2F94%20Passed-blue?style=flat-square)
![API Client](https://img.shields.io/badge/API%20Client-Bruno%20v4.2.0-yellow?style=flat-square)
![Node.js](https://img.shields.io/badge/Node.js-20+-339933?style=flat-square&logo=node.js&logoColor=white)
![Architecture](https://img.shields.io/badge/Architecture-Token%20Chaining%20%2B%20Idempotent-orange?style=flat-square)
[![Live Sandbox](https://img.shields.io/badge/Live%20Sandbox-qafakeapi.ziaratechqlabs.in-f97316?style=flat-square)](https://qafakeapi.ziaratechqlabs.in/playground)

A scalable, maintainable, and enterprise-grade REST API automation framework built for **[QAFakeAPI](https://qafakeapi.ziaratechqlabs.in/playground)** (The Free Fake REST API Sandbox for QA & SDETs by Ziara TechQ Labs) using **Bruno** (`.bru` plain text format) and `@usebruno/cli`.

Follows strict **Git-friendly `.bru` architecture**, automated JWT token chaining, dynamic entity lifecycle (CRUD with 404 verification), chaos/latency simulations, idempotent pre/post database reset hooks, and automated GitHub Actions CI/CD regression gates with HTML & JUnit reporting.

---

## 🚀 Key Framework Features

- **100% Native Plain-Text `.bru` Format**: Git-friendly, human-readable, and free from monolithic JSON merge conflicts.
- **Dynamic JWT Bearer Token Chaining**: Automatically captures JWT token upon login and propagates it seamlessly to protected routes.
- **Full Entity CRUD Lifecycle & ID Chaining**: Creates products dynamically, stores IDs in memory, updates, deletes, and validates HTTP 404 on subsequent queries.
- **Idempotent Pre & Post Test Suite Reset**: Automatically triggers `POST /api/v1/reset` before and after test runs, guaranteeing zero flakiness.
- **Comprehensive Assertion Layers**: Asserts HTTP status codes, JSON response schemas, mandatory keys, array non-emptiness, headers, and payload integrity (94/94 assertions).
- **QA Chaos & Resilience Testing**: Dedicated validation for 404 Not Found, 500 Server Error, 429 Rate Limiting, and 1500ms network delay.
- **Multi-Environment Support**: Seamless toggle between `Production` (live sandbox) and `Local` environments.
- **Enterprise CI/CD Integration**: Headless execution via `@usebruno/cli` with HTML report and JUnit XML generation.

---

## 📐 Framework Architecture

```mermaid
graph TD
    subgraph Suite_Setup["0. Setup Layer (Idempotency)"]
        S1["00-Setup/01-Reset-Database-Pre.bru<br/>(POST /api/v1/reset)"]
    end

    subgraph Suite_Auth["1. Authentication & Token Chaining"]
        A1["01-Login-Success.bru<br/>(POST /api/v1/auth/login)"]
        A2["02-Login-Invalid-Credentials.bru<br/>(401 Negative Test)"]
        A3["03-Get-Current-Profile.bru<br/>(GET /api/v1/auth/me)"]
        A4["04-Get-Profile-Unauthorized.bru<br/>(401 Negative Test)"]
    end

    subgraph Suite_Products["2. Products Catalog (CRUD & Search)"]
        P1["01-Get-All-Products.bru (Pagination)"]
        P2["02-Search-Products-Keyword.bru (?q=Mouse)"]
        P3["03-Filter-Products-Category.bru (?category=electronics)"]
        P4["04-Get-Product-By-ID.bru (GET /products/1)"]
        P5["05-Create-Product.bru (POST /products -> saves createdProductId)"]
        P6["06-Update-Product.bru (PUT /products/:id)"]
        P7["07-Delete-Product.bru (DELETE /products/:id)"]
        P8["08-Get-Deleted-Product.bru (404 Verification)"]
    end

    subgraph Suite_Users_Orders["3. Users & Relational Orders"]
        U1["01-List-Users.bru"]
        U2["02-Register-User.bru (Dynamic Username)"]
        O1["01-List-Orders.bru"]
        O2["02-Get-Order-By-ID.bru"]
        O3["03-Get-Invalid-Order-404.bru"]
    end

    subgraph Suite_Chaos["4. Chaos & Latency Simulators"]
        C1["01-Simulate-404-Not-Found.bru"]
        C2["02-Simulate-500-Server-Error.bru"]
        C3["03-Simulate-429-Rate-Limit.bru"]
        C4["04-Simulate-Network-Delay.bru (1.5s Latency)"]
    end

    subgraph Suite_Teardown["5. Teardown & Invalidation"]
        T1["01-Health-Check.bru"]
        T2["02-Revoke-Session-Logout.bru"]
        T3["03-Reset-Database-Post.bru"]
    end

    S1 --> A1
    A1 -->|Extracts authToken| A3 & P5 & P6 & P7 & T2
    A1 --> A2 & A4
    P5 -->|Extracts createdProductId| P6 --> P7 --> P8
    A3 --> P1 & P2 & P3 & P4
    P8 --> U1 & U2 & O1 & O2 & O3
    O3 --> C1 & C2 & C3 & C4
    C4 --> T1 --> T2 --> T3
```

---

## 🔄 Automated CI/CD Regression Gate

```mermaid
sequenceDiagram
    autonumber
    actor SDET as QA Engineer / Developer
    participant GitHub as GitHub Repository (qafakeapi_bruno_api)
    participant Runner as GitHub Actions Runner (Ubuntu)
    participant BrunoCLI as @usebruno/cli Engine
    participant TargetAPI as Live QAFakeAPI Sandbox
    
    SDET->>GitHub: Pushes commit or opens Pull Request
    GitHub->>Runner: Triggers api-automation-ci.yml workflow
    Runner->>Runner: Setup Node.js 20 & install dependencies
    Runner->>BrunoCLI: Executes `npx bru run --env Production`
    BrunoCLI->>TargetAPI: Runs 25 automated requests across 7 suites
    TargetAPI-->>BrunoCLI: Returns responses & validates 94 assertions
    alt All Tests Pass (100%)
        BrunoCLI->>Runner: Exports report.html & results.xml
        Runner->>GitHub: Publishes Green Quality Gate ✅ + Step Summary
    else Any Assertion Fails
        BrunoCLI->>Runner: Error exit code
        Runner->>GitHub: Red Pipeline ❌ + Uploads failure artifacts
    end
```

---

## 📁 Repository Structure

```text
qafakeapi_bruno_api/
├── .github/
│   └── workflows/
│       └── api-automation-ci.yml        # Automated GitHub Actions CI/CD Pipeline
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
├── package-lock.json                    # Deterministic build lockfile
└── README.md                            # Comprehensive framework documentation
```

---

## 🧪 Test Scenarios Covered (25 / 25 Passed)

| Suite | File / Request | Method | Endpoint | Verification Focus |
| :--- | :--- | :---: | :--- | :--- |
| **00-Setup** | `01-Reset-Database-Pre` | `POST` | `/api/v1/reset` | Resets sandbox database to defaults before execution |
| **01-Authentication** | `01-Login-Success` | `POST` | `/api/v1/auth/login` | Validates 200, checks role, captures JWT `authToken` |
| | `02-Login-Invalid-Credentials` | `POST` | `/api/v1/auth/login` | Validates 401 Unauthorized for wrong credentials |
| | `03-Get-Current-Profile` | `GET` | `/api/v1/auth/me` | Validates 200 using `Bearer {{authToken}}` |
| | `04-Get-Profile-Unauthorized` | `GET` | `/api/v1/auth/me` | Validates 401 when token header is omitted |
| **02-Products** | `01-Get-All-Products` | `GET` | `/api/v1/products` | Validates paginated list, total items, and page number |
| | `02-Search-Products-Keyword` | `GET` | `/api/v1/products?q=Mouse` | Asserts keyword filtering matches item name |
| | `03-Filter-Products-Category` | `GET` | `/api/v1/products?category=electronics` | Asserts category filtering matches correctly |
| | `04-Get-Product-By-ID` | `GET` | `/api/v1/products/1` | Asserts single product details, pricing, and stock |
| | `05-Create-Product` | `POST` | `/api/v1/products` | Creates product and dynamically captures `createdProductId` |
| | `06-Update-Product` | `PUT` | `/api/v1/products/:id` | Updates price and stock on `{{createdProductId}}` |
| | `07-Delete-Product` | `DELETE` | `/api/v1/products/:id` | Deletes product identified by `{{createdProductId}}` |
| | `08-Get-Deleted-Product` | `GET` | `/api/v1/products/:id` | Negative check: Asserts 404 for deleted product |
| **03-Users** | `01-List-Users` | `GET` | `/api/v1/users` | Validates registered users array and object schemas |
| | `02-Register-User` | `POST` | `/api/v1/users` | Registers dynamic user via pre-request timestamp script |
| **04-Orders** | `01-List-Orders` | `GET` | `/api/v1/orders` | Asserts relational orders list, users, and line items |
| | `02-Get-Order-By-ID` | `GET` | `/api/v1/orders/101` | Validates single order items, status, and total amount |
| | `03-Get-Invalid-Order-404` | `GET` | `/api/v1/orders/999999` | Negative check: Asserts 404 for non-existent order |
| **05-Chaos-Simulators** | `01-Simulate-404-Not-Found` | `GET` | `/api/v1/mock/status/404` | Verifies 404 simulation response and payload |
| | `02-Simulate-500-Server-Error` | `GET` | `/api/v1/mock/status/500` | Verifies 500 internal server error simulation |
| | `03-Simulate-429-Rate-Limit` | `GET` | `/api/v1/mock/status/429` | Verifies 429 rate limit exceeded simulation |
| | `04-Simulate-Network-Delay` | `GET` | `/api/v1/mock/delay/1500` | Validates 1500ms server delay and response latency |
| **06-System** | `01-Health-Check` | `GET` | `/api/v1/meta/health` | Validates online status, service name, and system health |
| | `02-Revoke-Session-Logout` | `POST` | `/api/v1/auth/logout` | Revokes the active session token on the server |
| | `03-Reset-Database-Post` | `POST` | `/api/v1/reset` | Resets database back to clean baseline state |

---

## ⚡ Getting Started

### 1. Prerequisites
- **Node.js**: v18.0.0 or higher
- **Bruno Desktop App** *(Optional, for GUI exploration)*: [Download from usebruno.com](https://www.usebruno.com/downloads)

### 2. Installation
Clone this repository and install dependencies:
```bash
git clone https://github.com/Aquil1401/qafakeapi_bruno_api.git
cd qafakeapi_bruno_api
npm install
```

### 3. Run Tests via CLI

```bash
# Run the entire 25-request suite against Production
npm test

# Run tests with HTML and JUnit report generation
npm run test:report

# Run tests against Local Sandbox (localhost:3000)
npm run test:local

# Run specific functional suites
npm run test:auth
npm run test:products
npm run test:users
npm run test:orders
npm run test:chaos
npm run test:system
```

### 4. View Test Reports
After running `npm run test:report`, open the generated HTML report:
```bash
# Open in browser (Windows)
start reports/report.html
```

---

## 🚢 CI/CD Pipeline (GitHub Actions)

The workflow (`.github/workflows/api-automation-ci.yml`) runs automatically on every `push`, `pull_request`, and daily `cron`:
1. Checks out the repository.
2. Sets up Node.js 20 with npm caching.
3. Installs dependencies (`npm install`).
4. Executes the full Bruno API test suite against the target environment.
5. Generates **HTML Report** (`reports/report.html`) and **JUnit XML** (`reports/results.xml`).
6. Archives test report artifacts for 30 days under GitHub Actions.
7. Publishes high-level test status directly to the **GitHub Step Summary**.

---

## 👨‍💻 Author

**Md Aquil** — QA Automation Engineer & SDET
* 🎯 **Open to**: **Full-Time Opportunities** (Remote / Hybrid) & **Contract QA Consulting**
* 🧪 **Testing Services**: [Ziara QA Labs](https://qa.ziaratechqlabs.in/)
* 🏢 **Company / Studio**: [Ziara TechQ Labs](https://www.ziaratechqlabs.in/)
* 💼 **LinkedIn**: [linkedin.com/in/md-aquil-qa](https://www.linkedin.com/in/md-aquil-qa/)
* 📺 **YouTube**: [Ziara TechQ Labs](https://www.youtube.com/@ZiaraTechQLabs)
* 📧 **Email**: [ziaratechqlabs@gmail.com](mailto:ziaratechqlabs@gmail.com)
