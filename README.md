# AZMAN HOLDING HR Sistemi

Holding strukturunda çoxsaylı şirkətlərin insan resurslarının idarə edilməsi üçün tam funksional HR sistem.

---

## 📋 Texnologiya Stack

| Təbəqə | Texnologiya |
|--------|------------|
| **Frontend** | Next.js 15 (App Router) + TypeScript + Tailwind CSS v4 + shadcn/ui |
| **Backend** | Next.js API Routes (Route Handlers) |
| **Database** | PostgreSQL 16 + Prisma ORM |
| **Auth** | NextAuth.js v5 (Credentials + JWT) |
| **File Storage** | Local (dev) → S3/MinIO (prod) |
| **Email** | Nodemailer + SMTP |
| **Charts** | Recharts |
| **Testing** | Jest (unit) + Playwright (E2E) |
| **DevOps** | Docker Compose + GitHub Actions |

---

## 🏗️ Arxitektura

```
src/
├── app/
│   ├── (auth)/
│   │   ├── login/page.tsx          # Giriş səhifəsi
│   │   └── layout.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx              # Sidebar + Topbar shell
│   │   ├── page.tsx                # Dashboard
│   │   ├── employees/
│   │   │   └── page.tsx            # Kadr uçotu siyahısı
│   │   ├── structure/
│   │   │   └── page.tsx            # Təşkilat strukturu
│   │   ├── settings/
│   │   │   └── page.tsx            # Ayarlar (Admin)
│   │   └── audit/
│   │       └── page.tsx            # Audit loqu (Admin)
│   └── api/
│       ├── auth/[...nextauth]/route.ts
│       ├── companies/route.ts
│       ├── employees/
│       │   ├── route.ts            # GET (list), POST (create)
│       │   └── [id]/
│       │       ├── route.ts        # GET, PATCH
│       │       ├── hire/route.ts   # POST
│       │       ├── terminate/route.ts # POST
│       │       ├── restore/route.ts # POST
│       │       ├── reject/route.ts  # POST
│       │       ├── transfer/route.ts # POST
│       │       ├── salary/route.ts  # PATCH
│       │       ├── education/route.ts # GET, POST
│       │       ├── experience/route.ts # GET, POST
│       │       ├── family/route.ts    # GET, POST
│       │       ├── certificates/route.ts # GET, POST
│       │       ├── documents/route.ts  # PATCH
│       │       ├── driver/route.ts     # PATCH
│       │       ├── bank/route.ts       # PATCH
│       │       ├── insurance/route.ts  # PATCH
│       │       └── medical/route.ts    # PATCH
│       ├── departments/route.ts
│       ├── positions/route.ts
│       ├── users/route.ts
│       ├── dictionaries/route.ts
│       ├── audit/route.ts
│       ├── dashboard/route.ts
│       ├── notifications/route.ts
│       └── upload/
│           ├── photo/route.ts
│           ├── contract/route.ts
│           └── document/route.ts
├── components/
│   ├── ui/                         # shadcn/ui components
│   ├── layout/
│   │   ├── Sidebar.tsx
│   │   ├── Topbar.tsx
│   │   └── AppShell.tsx
│   ├── employees/
│   │   ├── EmployeeList.tsx
│   │   ├── EmployeeTabs.tsx
│   │   ├── EmployeeCard.tsx
│   │   ├── tabs/
│   │   │   ├── PersonalTab.tsx
│   │   │   ├── DocumentsTab.tsx
│   │   │   ├── DriverTab.tsx
│   │   │   ├── BankTab.tsx
│   │   │   ├── InsuranceTab.tsx
│   │   │   ├── MedicalTab.tsx
│   │   │   ├── EducationTab.tsx
│   │   │   ├── ExperienceTab.tsx
│   │   │   ├── FamilyTab.tsx
│   │   │   ├── CorporateTab.tsx
│   │   │   ├── CertificatesTab.tsx
│   │   │   └── LifecycleTab.tsx
│   │   └── modals/
│   │       ├── NewEmployeeModal.tsx
│   │       ├── HireModal.tsx
│   │       ├── TerminateModal.tsx
│   │       ├── RejectModal.tsx
│   │       ├── TransferModal.tsx
│   │       └── EditModals.tsx
│   ├── structure/
│   │   ├── OrgTree.tsx
│   │   ├── DepartmentList.tsx
│   │   └── PositionList.tsx
│   ├── dashboard/
│   │   ├── StatsCards.tsx
│   │   ├── RecentActivity.tsx
│   │   ├── MonthlyChart.tsx
│   │   └── QuickActions.tsx
│   ├── settings/
│   │   ├── CompanyList.tsx
│   │   ├── UserList.tsx
│   │   └── DictionaryManager.tsx
│   └── shared/
│       ├── StatusPill.tsx
│       ├── Avatar.tsx
│       ├── SearchInput.tsx
│       ├── FilterSelect.tsx
│       ├── BulkActionBar.tsx
│       ├── FunctionalPanel.tsx
│       └── FileUpload.tsx
├── lib/
│   ├── prisma.ts                   # Prisma client singleton
│   ├── auth.ts                     # NextAuth config
│   ├── audit.ts                    # Audit log helper
│   ├── utils.ts                    # Utility functions
│   └── constants.ts                # Enums, labels, etc.
├── types/
│   └── index.ts                    # TypeScript interfaces
├── hooks/
│   ├── useDebounce.ts
│   ├── useBulkSelection.ts
│   └── useCompanyFilter.ts
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
└── middleware.ts                    # Auth + role middleware
```

---

## 🗄️ Database Schema (Əsas Modellər)

### Entity Relationship Diagram

```
Company (Holding/Şirkət)
├── 1:N → Department (parent_id → self-referencing hierarchy)
├── 1:N → Position
├── 1:N → Employee
└── 1:N → User

Employee (Əməkdaş)
├── N:1 → Company
├── N:1 → Department
├── N:1 → Position
├── 1:N → Education
├── 1:N → Experience
├── 1:N → FamilyMember
├── 1:N → IndustryCertificate
├── 1:N → EmployeeLifecycle
├── 1:N → AuditLog
├── 1:N → FileAttachment
└── 1:N → Notification

User (İstifadəçi)
├── N:1 → Company (optional, yalnız SUBSIDIARY_HR)
└── enum role: ADMIN | HOLDING_HR | SUBSIDIARY_HR
```

### Employee Status Lifecycle

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
| Dashboard | ✅ | ✅ | ✅ (öz şirkəti) |
| Kadr Uçotu | ✅ (hamısı) | ✅ (hamısı, filtri var) | ✅ (yalnız öz) |
| Struktur | ✅ | ✅ | ✅ (baxmaq) |
| Ayarlar | ✅ | ❌ | ❌ |
| Audit Loqu | ✅ | ❌ | ❌ |

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

## 🚀 Development

### Prerequisites
- Node.js 20+
- pnpm 9+
- Docker & Docker Compose

### Setup

```bash
# Repoyu klonla
git clone https://github.com/Zam83-AZE/gemini-hr-system.git
cd gemini-hr-system

# Dependencies quraşdır
pnpm install

# Environment faylını yarat
cp .env.example .env.local

# Docker ilə database-i başlat
docker compose up -d

# Database migrasiyalarını işə sal
pnpm prisma migrate dev

# Seed data yüklə
pnpm prisma db seed

# Dev server-i başlat
pnpm dev
```

### Environment Variables

```env
# Database
DATABASE_URL="postgresql://hr_user:hr_password@localhost:5432/hr_system"

# Auth
NEXTAUTH_SECRET="your-secret-key"
NEXTAUTH_URL="http://localhost:3000"

# Email (optional)
SMTP_HOST="smtp.gmail.com"
SMTP_PORT="587"
SMTP_USER="your-email@gmail.com"
SMTP_PASS="your-app-password"
SMTP_FROM="hr@azmanholding.az"

# File Storage
UPLOAD_DIR="./public/uploads"
S3_ENDPOINT="http://localhost:9000"
S3_BUCKET="hr-uploads"
S3_ACCESS_KEY="minioadmin"
S3_SECRET_KEY="minioadmin"
```

---

## 📦 Milestones və Issue-lar

Tam inkişaf planı GitHub Issues-da 12 milestone-da təşkil olunub:

| Milestone | Say | Təsvir |
|-----------|-----|--------|
| **M0** | 8 | Ləvazimat və Arxitektura (Foundation) |
| **M1** | 6 | Autentifikasiya və İstifadəçi İdarəetməsi |
| **M2** | 5 | Şirkət və Təşkilat Strukturu |
| **M3** | 6 | Kadr Uçotu — Əsas Modul |
| **M4** | 9 | Sənədlər, Təhsil, Təcrübə, Ailə |
| **M5** | 6 | Əməkdaş Həyat Dövrü (Lifecycle) |
| **M6** | 5 | Dashboard və Hesabatlar |
| **M7** | 3 | Audit Log və İzləmə |
| **M8** | 2 | Toplu Əməliyyatlar və Export |
| **M9** | 3 | Bildirişlər və Xatırlatmalar |
| **M10** | 4 | UI/UX, Responsive və Polish |
| **M11** | 4 | Testing, Security və Deployment |
| **Cəmi** | **61** | |

> **Qeyd**: Hər issue detalli Qəbul Meyarları (- [ ] checkboxes), Texniki Tələblər və Prototip İstinadı ilə təsvir olunub.

---

## 📐 Prototip İstinadı

HTML prototipi `prototype/` qovluğunda saxlanılır. Bu prototip:
- Bütün UI komponentlərinin vizual dizaynını göstərir
- Bütün data strukturlarını (JS obyektləri) ehtiva edir
- Bütün user interaction-ları (kliklər, modallar, tab switch) simulyasiya edir
- Bütün 3 rol-un görə biləcəyi məzmunu nümayiş etdirir

---

## 📄 License

Private — AZMAN HOLDING
