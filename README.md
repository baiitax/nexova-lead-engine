# Nexova Lead Engine

A production-grade B2B SaaS platform that automates lead generation, CRM tracking, and AI-powered outbound email drafting.

## Architecture Overview

```
Nexova Lead Engine
│
├── Next.js App (Frontend + API)
│   ├── Dark-mode CRM Dashboard
│   ├── Leads Data Table (sortable, filterable)
│   ├── Lead Detail Modal with AI Email Drafting
│   └── API Routes (REST)
│
├── Supabase (Backend)
│   ├── PostgreSQL Database (via Prisma)
│   ├── Supabase Auth (OAuth + Magic Link)
│   └── Row Level Security
│
├── Standalone Scraper Engine (VPS)
│   ├── Google Places API Integration
│   ├── Website Health Validation
│   ├── Lead Qualification Pipeline
│   └── Email Reporting (Nodemailer)
│
└── Google Gemini AI
    └── Professional Email Draft Generation
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 16 (App Router), React 19, Tailwind CSS 4, Lucide React |
| Backend | Next.js Route Handlers (REST API) |
| Database | Supabase PostgreSQL + Prisma ORM 6 |
| Auth | Supabase Auth (Email/Password + Magic Link) |
| AI | Google Gemini 1.5 Flash (@google/genai SDK) |
| Scraper | Node.js, Axios, node-cron, Nodemailer |
| APIs | Google Places API (New) |

## Project Structure

```
nexova-lead-engine/
├── prisma/
│   └── schema.prisma          # Database schema
├── scraper/
│   ├── scraper.js             # Standalone scraper engine
│   ├── package.json           # Scraper dependencies
│   └── .env.scraper.example   # Scraper env template
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── auth/user/route.ts      # Get current user
│   │   │   ├── leads/route.ts          # List & create leads
│   │   │   ├── leads/[id]/route.ts     # Get, update, delete lead
│   │   │   └── generate-draft/route.ts # Gemini email generation
│   │   ├── dashboard/
│   │   │   ├── page.tsx                # Dashboard home
│   │   │   ├── leads/page.tsx          # Leads page
│   │   │   ├── outreach/page.tsx       # Outreach tracking
│   │   │   ├── analytics/page.tsx      # Analytics & insights
│   │   │   ├── settings/page.tsx       # App settings
│   │   │   └── layout.tsx              # Dashboard layout
│   │   ├── auth/page.tsx               # Login/signup page
│   │   ├── page.tsx                    # Landing page
│   │   ├── layout.tsx                  # Root layout
│   │   └── globals.css                 # Global styles
│   ├── components/
│   │   ├── sidebar.tsx                 # Dashboard sidebar
│   │   ├── topbar.tsx                  # Dashboard top bar
│   │   └── lead-detail-modal.tsx       # Lead detail + AI draft
│   ├── hooks/
│   │   └── use-auth.ts                 # Auth state hook
│   └── lib/
│       ├── prisma.ts                   # Prisma client singleton
│       ├── supabase.ts                 # Supabase clients
│       ├── supabase-server.ts          # Server-side Supabase
│       ├── supabase-browser.ts         # Browser Supabase
│       ├── auth.ts                     # Auth helpers
│       └── utils.ts                    # Utility functions
├── .env.local                          # Environment variables
├── next.config.ts                      # Next.js config
├── package.json                        # Dependencies
└── tsconfig.json                       # TypeScript config
```

## Database Schema

### Tables

#### User
| Field | Type | Description |
|-------|------|-------------|
| id | String (CUID) | Primary key |
| email | String (unique) | User email |
| name | String? | Display name |
| avatarUrl | String? | Profile image |
| role | UserRole | USER or ADMIN |
| supabaseUid | String? (unique) | Supabase auth ID |
| createdAt | DateTime | Account creation |
| updatedAt | DateTime | Last update |

#### Lead
| Field | Type | Description |
|-------|------|-------------|
| id | String (CUID) | Primary key |
| companyName | String | Business name |
| category | String | Business category |
| location | String | Geographic location |
| phone | String? | Phone number |
| rating | Float? | Google rating (1-5) |
| reviews | Int? | Number of reviews |
| websiteStatus | WebsiteStatus | MISSING, EXPIRED_SSL, BROKEN_LINK, ACTIVE, UNKNOWN |
| websiteUri | String? | Website URL |
| servicePitch | ServicePitch | Web or Automation |
| salesAngle | String? | Custom sales pitch |
| status | LeadStatus | New, Emailed, Follow_Up, Closed |
| placeId | String? (unique) | Google Place ID |
| createdAt | DateTime | Discovery date |
| updatedAt | DateTime | Last update |

#### OutreachLog
| Field | Type | Description |
|-------|------|-------------|
| id | String (CUID) | Primary key |
| leadId | String (FK) | Related lead |
| emailDraft | Text | AI-generated email |
| followUpDate | DateTime? | When to follow up |
| sentAt | DateTime | Creation timestamp |

## Quick Start

### 1. Prerequisites

- Node.js 20+
- Supabase account (free tier)
- Google Cloud account (for Places API & Gemini)

### 2. Supabase Setup

1. Create a new Supabase project at https://supabase.com
2. Go to Project Settings > API and copy:
   - Project URL (`NEXT_PUBLIC_SUPABASE_URL`)
   - Anon Key (`NEXT_PUBLIC_SUPABASE_ANON_KEY`)
   - Service Role Key (`SUPABASE_SERVICE_ROLE_KEY`)
3. Go to Database > Connection string and copy the PostgreSQL URL

### 3. Google Cloud Setup

1. Enable APIs at https://console.cloud.google.com:
   - Places API (New)
   - Gemini API
2. Create an API key for each service

### 4. Environment Configuration

```bash
# Copy the example file
cp .env.local .env.local

# Edit with your credentials
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
DATABASE_URL=postgresql://postgres:[password]@db.your-project.supabase.co:5432/postgres
DIRECT_URL=postgresql://postgres:[password]@db.your-project.supabase.co:5432/postgres
GOOGLE_PLACES_API_KEY=your_places_api_key
GOOGLE_GEMINI_API_KEY=your_gemini_api_key
```

### 5. Database Setup

```bash
# Install dependencies
npm install

# Generate Prisma client
npx prisma generate

# Push schema to database
npx prisma db push

# (Optional) Open Prisma Studio
npx prisma studio
```

### 6. Run Development Server

```bash
npm run dev
```

Visit http://localhost:3000

### 7. Deploy Scraper to VPS

The scraper runs independently of the Next.js app on a cheap VPS (Hetzner ~$5/month or DigitalOcean ~$6/month).

```bash
# On your VPS:
cd scraper
cp .env.scraper.example .env.scraper
# Edit .env.scraper with your credentials
npm install
node scraper.js
```

Use PM2 for process management:
```bash
npm install -g pm2
pm2 start scraper.js --name "nexova-scraper"
pm2 save
pm2 startup
```

## API Reference

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/auth/user` | Get current user |

### Leads
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/leads?status=&servicePitch=&search=&sortBy=&sortOrder=&page=&limit=` | List leads |
| POST | `/api/leads` | Create a lead |
| GET | `/api/leads/:id` | Get lead with outreach logs |
| PATCH | `/api/leads/:id` | Update lead |
| DELETE | `/api/leads/:id` | Delete lead |

### AI Email Drafting
| Method | Endpoint | Body | Description |
|--------|----------|------|-------------|
| POST | `/api/generate-draft` | `{ leadId }` | Generate email via Gemini |

## Gemini Prompt Engineering

The email generation system uses sophisticated prompt engineering:

1. **Context Gathering**: Lead data (company name, rating, reviews, website status, location)
2. **Dynamic Angles**: Sales angle adapts based on website status (missing → web pitch, active → optimization)
3. **Tone Calibration**: Professional yet warm, human-sounding, not pushy
4. **Structure**: Acknowledge reputation → Identify bottleneck → Offer solution → Soft CTA
5. **Constraints**: Under 200 words, plain text, no HTML

## Scraper Engine

### Features
- Runs every 5 hours via node-cron
- Queries Google Places API with field masks for efficiency
- Filters: rating > 4.0, reviews > 30
- Website validation: HTTP GET with timeout, SSL check, 404 detection
- Only saves leads with actionable website issues (missing, broken, expired SSL)
- Email summary report via Nodemailer after each run
- Duplicate prevention via Google Place ID

### Configuration
Edit the `CONFIG` object in `scraper/scraper.js`:
- `searchQueries`: Array of niche + location queries
- `minRating`: Minimum star rating (default: 4.0)
- `minReviews`: Minimum review count (default: 30)
- `cronSchedule`: Cron expression (default: every 5 hours)

## Deployment Options

### Option A: Vercel (Frontend) + VPS (Scraper)
- Deploy Next.js app to Vercel
- Run scraper on Hetzner/DigitalOcean VPS
- Connect both to same Supabase database

### Option B: Full VPS Deployment
- Run both Next.js and scraper on same VPS
- Use PM2 for process management
- Configure Nginx reverse proxy

### Option C: Supabase Edge Functions (Scraper)
- Migrate scraper to Supabase Edge Functions
- Schedule via pg_cron in Supabase
- Eliminates separate VPS cost

## License

MIT License - Built for B2B sales teams.
