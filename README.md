# AZMAN HOLDING HR Sistemi v1

Holding strukturunda çoxsaylı şirkətlərin insan resurslarının idarə edilməsi üçün tam funksional HR sistemi. Go SPA arxitekturası ilə — server-side rendering, HTMX ilə dynamic interaktivlik.

---

## 📋 Texnologiya Stack

| Təbəqə | Texnologiya | Niyə? |
|--------|------------|-------|
| **Backend** | Go + **Chi** router | Sadə, standart library uyğun, middleware asan |
| **Frontend** | **Templ** + **HTMX** + **Tailwind CSS** | Type-safe templates, runtime error yox, server-driven |
| **Database** | **MariaDB 10.11** + **GORM** | 50+ field model üçün ORM vacib, SQL boilerplate az olur |
| **Auth** | Sadə JWT middleware | 3 rol = casbin overkill, 5 middleware funksiyası |
| **File Storage** | **Local filesystem** (`/static/uploads/`) | v1 sadə saxlayırıq |
| **Email** | Go `net/smtp` | Standart library, 3rd-party yoxdur |
| **Testing** | Go testing (unit) + Playwright (E2E) | |
| **DevOps** | Docker Compose + GitHub Actions | |

---

## 🏗️ Layihə Strukturu

```
hr-system/
├── cmd/server/main.go              # Entry point
├── internal/
│   ├── config/config.go            # ENV, DB, JWT config
│   ├── middleware/
│   │   ├── auth.go                 # JWT verify + context
│   │   ├── role.go                 # RequireAdmin, RequireHoldingHR, RequireSubHR
│   │   ├── logging.go              # Request logger
│   │   └── recoverer.go            # Panic recovery
│   ├── models/                     # GORM models
│   │   ├── company.go              # name, taxId, industry, parentId, isHolding
│   │   ├── department.go           # name, companyId, parentId, isActive
│   │   ├── position.go             # name, companyId, status
│   │   ├── employee.go             # 50+ field
│   │   ├── user.go                 # role, name, email, passwordHash, companyId
│   │   ├── education.go            # employee:many
│   │   ├── experience.go           # employee:many
│   │   ├── family_member.go        # employee:many
│   │   ├── industry_certificate.go # employee:many
│   │   ├── employee_lifecycle.go   # employee:many
│   │   ├── audit_log.go            # kim, nə vaxt, hansı sahə, köhnə→yeni
│   │   ├── dictionary.go           # type, name, isActive
│   │   ├── notification.go         # userId, title, message, isRead
│   │   └── file_attachment.go      # entityType, entityId, path, name
│   ├── handlers/                   # HTTP handlers (Chi routes)
│   │   ├── auth_handler.go         # POST /login, POST /logout
│   │   ├── dashboard_handler.go    # GET /dashboard
│   │   ├── company_handler.go      # CRUD /companies, tree
│   │   ├── department_handler.go   # CRUD /departments
│   │   ├── position_handler.go     # CRUD /positions
│   │   ├── employee_handler.go     # CRUD + hire, terminate, restore, reject, transfer
│   │   ├── user_handler.go         # CRUD /users (admin only)
│   │   ├── dictionary_handler.go   # CRUD /dictionaries
│   │   ├── audit_handler.go        # GET /audit
│   │   ├── upload_handler.go       # POST /upload/photo, /contract, /cv, etc.
│   │   └── notification_handler.go # GET/PUT /notifications
│   ├── services/                   # Business logic
│   │   ├── auth_service.go         # Login, token, password hash (bcrypt)
│   │   ├── employee_service.go     # Lifecycle state machine
│   │   ├── audit_service.go        # logAudit() helper
│   │   └── export_service.go       # CSV generation
│   └── repository/                 # Data access (GORM)
│       ├── company_repo.go
│       ├── employee_repo.go
│       ├── user_repo.go
│       └── dictionary_repo.go
├── db/
│   ├── db.go                       # GORM connect + auto-migrate
│   └── seed.go                     # İlkin data (15 şirkət, 9 user, 9 employee...)
├── views/
│   ├── layouts/
│   │   ├── base.templ              # HTML skeleton, Tailwind, HTMX CDN
│   │   ├── app.templ               # Sidebar + Topbar + Content shell
│   │   └── auth.templ              # Login layout (no sidebar)
│   ├── pages/
│   │   ├── login.templ             # Giriş səhifəsi
│   │   ├── dashboard.templ         # Dashboard
│   │   ├── employees.templ         # Kadr uçotu siyahısı
│   │   ├── employee_card.templ     # Profil kartı (12 tab)
│   │   ├── structure.templ         # Təşkilat strukturu
│   │   ├── settings.templ          # Ayarlar (Admin)
│   │   └── audit.templ             # Audit loqu (Admin)
│   ├── components/                 # Reusable Templ components
│   │   ├── sidebar.templ
│   │   ├── topbar.templ
│   │   ├── stats_card.templ
│   │   ├── status_pill.templ
│   │   ├── avatar.templ
│   │   ├── bulk_action_bar.templ
│   │   └── functional_panel.templ
│   └── partials/                   # HTMX partial responses
│       ├── employee_list.templ     # Siyahı yenilənməsi (HTMX swap)
│       ├── company_tree.templ      # Tree node partial
│       ├── tab_personal.templ      # Şəxsi məlumatlar
│       ├── tab_documents.templ     # Sənədlər
│       ├── tab_driver.templ        # Sürücü
│       ├── tab_bank.templ          # Bank
│       ├── tab_insurance.templ     # Sığorta
│       ├── tab_medical.templ       # Tibbi
│       ├── tab_education.templ     # Təhsil
│       ├── tab_experience.templ    # Təcrübə
│       ├── tab_family.templ        # Ailə
│       ├── tab_corporate.templ     # Korporativ
│       ├── tab_certificates.templ  # Sertifikatlar
│       ├── tab_lifecycle.templ     # Tarixçə
│       ├── modal_*.templ           # Hər modal üçün partial
│       └── notification_dropdown.templ
├── static/
│   ├── css/input.css               # Tailwind source (@tailwind directives)
│   ├── js/app.js                   # HTMX config, Alpine.js, custom JS
│   └── uploads/                    # Local file storage
│       ├── photos/
│       ├── contracts/
│       ├── diplomas/
│       ├── certificates/
│       └── cvs/
├── docker-compose.yml              # MariaDB only
├── Dockerfile                      # Multi-stage Go build
├── Makefile                        # dev, build, seed, test, templ, tailwind
├── go.mod / go.sum
├── tailwind.config.js
├── postcss.config.js
├── .env.example
└── README.md
```

---

## 🗄️ GORM Database Schema

### Entity Relationships

```
Company (Holding/Şirkət)
├── 1:N → Department (parent_id → self-referencing hierarchy)
├── 1:N → Position
├── 1:N → Employee
└── 1:N → User

Employee (Əməkdaş) — 50+ field
├── N:1 → Company, Department, Position
├── 1:N → Education, Experience, FamilyMember
├── 1:N → IndustryCertificate, EmployeeLifecycle
├── 1:N → AuditLog, FileAttachment

User (İstifadəçi)
├── N:1 → Company (optional, yalnız SUBSIDIARY_HR)
└── enum role: ADMIN | HOLDING_HR | SUBSIDIARY_HR
```

### Employee Model (Əsas Sahələr)

```
// Əsas                    // Sənədlə
FN, LN, Fan, FIN          IdSerial, IdNumber
BD, Gender, Status        IdIssueDate, IdExpiryDate
                         TaxNumber, PensionNumber

// Əlaqə                  // Sürücü
Phone, Email              DriverLicense (bool)
Addr, RegAddress          DriverCategories (A-E)
FactAddress               DriverIssueDate, DriverExpiryDate

// Ailə                   // Bank
MaritalStatus             BankName, BankIban, BankCard
ChildrenCount
MilitaryService
MilitaryRank

// Sığorta                // Tibbi
InsuranceType             BloodGroup (A+, A-, B+, ...)
InsuranceCompany          ChronicDiseases
InsuranceNumber           Allergies
InsuranceDate             Disability (bool, percent)

// Korporativ              // Status
CompanyId                 Status (ACTIVE/CANDIDATE/TERMINATED/REJECTED)
DepartmentId              HireDate, TermDate, TermReason
PositionId                ProbationEndDate
Salary                    RejectReason, RejectFeedback
WorkSchedule              // Lifecycle connections
WorkLocation              Lifecycle []EmployeeLifecycle
ManagerId, IsReserve      AuditLogs []AuditLog
HasSubordinates
```

### Enums

```go
type UserRole string       // ADMIN, HOLDING_HR, SUBSIDIARY_HR
type EmployeeStatus string // ACTIVE, CANDIDATE, TERMINATED, REJECTED
type Industry string       // CONSTRUCTION, LOGISTICS, HOTEL, RESTAURANT_CHAIN,
                          // RESTAURANT, AGRICULTURE, PRODUCTION, SPORTS
```

---

## 🔄 Employee Lifecycle State Machine

```
        ┌──────────────────────────────────────┐
        │                                      │
   CANDIDATE ──(işə qəbul)──→  ACTIVE ──(xitam)──→  TERMINATED
        │                      ↑                    │
        │                      │                    │
        └──(rədd)──→ REJECTED   └────────(bərpa)────┘
                               │
                               └──(transfer)──→ ACTIVE (başqa şirkət)
```

| Əməliyyat | API Endpoint | Tələblər |
|-----------|-------------|----------|
| İşə qəbul | `POST /api/employees/:id/hire` | positionId, departmentId, hireDate, salary |
| Xitam | `POST /api/employees/:id/terminate` | date, reason (lüğətdən) |
| Bərpa | `POST /api/employees/:id/restore` | — |
| Rədd | `POST /api/employees/:id/reject` | reason, feedback? |
| Transfer | `POST /api/employees/:id/transfer` | targetCompanyId, positionId, departmentId, date |

---

## 🔐 JWT Auth Middleware

```go
// 5 middleware funksiyası:
RequireAuth()          // Token var? Context-ə user yaz
RequireAdmin()         // role == ADMIN
RequireHoldingHR()     // role IN (ADMIN, HOLDING_HR)
RequireSubHR()         // Authenticated kifayətdir (hər rol)
RequireRole(roles...)  // İstənilən rol

// Login flow:
POST /api/auth/login   → bcrypt check → JWT generate → Set-Cookie: token
POST /api/auth/logout  → Cookie-ni clear et
```

---

## 👥 Rollər və Giriş İdarəetməsi

| Rol | Görə biləcəyi əməliyyatlar |
|-----|---------------------------|
| **ADMIN** | Hər şey: bütün şirkətlər, istifadəçi idarəetməsi, audit log, sistem ayarları |
| **HOLDING_HR** | Bütün şirkətlərin HR əməliyyatları, şirkət filtri |
| **SUBSIDIARY_HR** | Yalnız öz şirkəti: əməkdaşlar, profil, lifecycle |

### Route Access Matrix

| Səhifə | ADMIN | HOLDING_HR | SUBSIDIARY_HR |
|--------|-------|------------|---------------|
| `/dashboard` | ✅ | ✅ | ✅ (öz şirkəti) |
| `/employees` | ✅ (hamısı) | ✅ (hamısı, filtri var) | ✅ (yalnız öz) |
| `/structure` | ✅ | ✅ | ✅ (baxmaq) |
| `/settings` | ✅ | ❌ | ❌ |
| `/audit` | ✅ | ❌ | ❌ |

---

## 📊 Təşkilat Strukturu (AZMAN HOLDING)

```
AZMAN HOLDING (Holding)
├── Azman Construction — CONSTRUCTION
├── TEZ Logistics — LOGISTICS
├── Sapphire Hotels Group — HOTEL
├── City Service Company — RESTAURANT_CHAIN (parent)
│   ├── Nar & Sharab — RESTAURANT
│   ├── Senator Hall — RESTAURANT
│   ├── Maxim Hall — RESTAURANT
│   ├── Mangal Steak House Zagulba — RESTAURANT
│   ├── Zuğulba Saray — RESTAURANT
│   ├── Beer Hall — RESTAURANT
│   └── JazzClub by BeerHall — RESTAURANT
├── EcoProd Azerbaijan — AGRICULTURE
├── Mangal MMC — PRODUCTION
└── Judo Club 2012 — SPORTS
```

---

## 🔄 HTMX ilə necə işləyir?

```
İstifadəçi "Cari" tab-a klikləyir
  → HTMX: hx-get="/employees?status=ACTIVE"
           hx-target="#emp-list"
           hx-swap="innerHTML"
  → Go handler render edir: views/partials/employee_list.templ
  → Templ HTML qaytarır
  → HTMX cədvəli yerləşdirir (səhifə yenilenmir)

İstifadəçi profilə klikləyir
  → HTMX: hx-get="/employees/1/card"
           hx-target="#modal-container"
           hx-swap="innerHTML"
  → Go handler: views/pages/employee_card.templ
  → Modal HTML qaytarılır

İstifadəçi tab dəyişir (məs: Təhsil)
  → HTMX: hx-get="/employees/1/tab/education"
           hx-target="#tab-content"
  → Go handler: views/partials/tab_education.templ
  → Sadəcə tab content dəyişir

Şəxsi məlumatları redaktə et
  → HTMX: hx-get="/employees/1/edit/personal"
  → Go: edit form HTML qaytarır
  → İstifadəçi "Yadda saxla" klikləyir
  → HTMX: hx-post="/employees/1/personal"
  → Go: validate → GORM update → audit log
  → Redirect: hx-get="/employees/1/tab/personal" (view mode)
```

---

## 🚀 Development

### Prerequisites
- Go 1.22+
- Templ CLI (`go install github.com/a-h/templ/cmd/templ@latest`)
- Node.js 18+ (yalnız Tailwind build üçün)
- Docker & Docker Compose

### Setup

```bash
# Repoyu klonla
git clone https://github.com/Zam83-AZE/gemini-hr-system.git
cd gemini-hr-system

# Go dependencies
go mod tidy

# Templ CLI quraşdır
go install github.com/a-h/templ/cmd/templ@latest

# Node dependencies (Tailwind üçün)
npm install

# Environment faylını yarat
cp .env.example .env

# Docker ilə MariaDB-i başlat
docker compose up -d

# Database migrate + seed
make seed

# Templ generate + Tailwind build
make build-assets

# Dev server-i başlat
make dev
# → http://localhost:8080
```

### Makefile Əmrləri

```bash
make dev          # Air hot-reload (go run)
make build        # Go binary build
make templ        # Templ generate
make tailwind     # Tailwind CSS build
make build-assets # templ + tailwind
make seed         # Database seed data
make test         # Unit tests
make e2e          # E2E tests (Playwright)
make prod-build   # Docker production build
make prod-up      # Docker compose production
```

### Environment Variables

```env
# Server
SERVER_PORT=8080

# Database (MariaDB)
DB_HOST=localhost
DB_PORT=3306
DB_USER=hr_user
DB_PASSWORD=hr_password
DB_NAME=hr_system

# JWT Auth
JWT_SECRET=your-super-secret-key-change-in-production
JWT_EXPIRY=24h

# File Storage (local)
UPLOAD_DIR=./static/uploads

# Email (optional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_FROM=hr@azmanholding.az
```

---

## 📦 Milestones və Inkişaf Planı

Tam plan GitHub Issues-da 12 milestone-da təşkil olunub:

| Milestone | Issue Sayı | Təsvir |
|-----------|------------|--------|
| **M0** | 8 | Foundation — Go, Chi, GORM, Templ, Tailwind, JWT, Docker, Seed |
| **M1** | 5 | Auth & Users — Login, JWT, User CRUD, Role middleware, Dictionaries |
| **M2** | 5 | Organization — Company Tree, Department, Position CRUD |
| **M3** | 5 | Employee Core — Siyahı, Profil Kartı, Namizəd, Personal tab |
| **M4** | 9 | Employee Details — 9 profil tab (Sənədlər, Təhsil, Təcrübə, Ailə...) |
| **M5** | 6 | Lifecycle — İşə qəbul, Xitam, Bərpa, Rədd, Transfer, Maaş |
| **M6** | 4 | Dashboard — Statistikalar, Fəaliyyətlər, CSV Export |
| **M7** | 3 | Audit Log — Service, Admin UI, Profil tarixçəsi |
| **M8** | 2 | Bulk Actions — Multi-select, Toplu əməliyyatlar |
| **M9** | 2 | Notifications — Email, In-app bildirişlər |
| **M10** | 3 | UI/UX Polish — Functional panel, Responsive, Loading states |
| **M11** | 3 | Testing & Deploy — Unit tests, E2E, Docker prod, CI/CD |
| **Cəmi** | **~55** | |

> **Qeyd**: Hər issue detalli Qəbul Meyarları (- [ ] checkboxes), Texniki Tələblər və Prototip İstinadı ilə təsvir olunub.

---

## 📐 Prototip İstinadı

HTML prototipi sistemdəki bütün funksionallıqların vizual referansıdır:
- Bütün UI komponentlərinin dizaynı
- Bütün data strukturları (JS DB obyekti)
- Bütün user interaction-lar (kliklər, modallar, tab switch)
- Bütün 3 rol-un görə biləcəyi məzmun

---

## 📄 License

Private — AZMAN HOLDING
