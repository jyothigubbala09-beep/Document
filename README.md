# ✈️ DocuPilot — AI Intelligent Document Processing (IDP) Platform

> **Autonomous Multimodal Document Extraction, Deterministic Financial Audit & Natural Language Knowledge Discovery for Finance Teams.**

[![Stack](https://img.shields.io/badge/Stack-React%20%7C%20Node.js%20%7C%20MongoDB%20%7C%20Gemini%202.5%20Flash-indigo.svg)](https://github.com)
[![Status](https://img.shields.io/badge/Status-Hackathon%20Ready%20%E2%9C%94-emerald.svg)](https://github.com)

---

## 📌 Problem Statement
Finance and accounts payable teams receive invoices, purchase orders (POs), and receipts in scattered, heterogeneous formats (scanned PDFs, high-res photos, receipts).
- **Manual Data Entry Bottlenecks**: High labor costs and slow processing cycles.
- **Human Math Errors**: Missed tax or subtotal mismatches.
- **Duplicate Payments**: Identical bills submitted multiple times causing corporate cash leakage.
- **PO Mismatches**: Unapproved line items, inflated quantities, and unit price discrepancies slip through without deterministic cross-checks.

## 💡 Solution Overview
**DocuPilot** is an end-to-end Intelligent Document Processing platform:
1. **Multimodal AI Extraction**: Direct ingestion of PDFs and images using **Google Gemini 2.5 Flash** with strict JSON schemas — eliminating brittle, legacy OCR pipelines.
2. **Deterministic Financial Audit Engine**: 7 plain-code rules (zero LLM hallucinations) that enforce exact line-item math, tax balance, PO cross-referencing, duplicate detection, and chronological date logic.
3. **Interactive Side-by-Side Review**: Side-by-side original document viewer with editable fields, real-time recalculation, and 1-click approval.
4. **Executive Analytics & AI Discovery Copilot**: Real-time spending distributions via Recharts, and a natural language Chat Copilot that answers questions strictly from the user's documents with verified citations.

---

## 🏗️ System Architecture

```text
+-----------------------------------------------------------------------------------+
|                                  CLIENT (Vite + React)                            |
|                                                                                   |
|  [ AuthContext ] <---> [ ProtectedRoute ]                                         |
|  [ Dashboard ]        - Recharts (Spend by Vendor & Classification Breakdown)     |
|  [ Upload Page ]      - Multi-File Drag-and-Drop Ingestion                        |
|  [ Documents View ]   - Filterable / Searchable Ledger with Issue Pills           |
|  [ Document Detail ]  - Side-by-Side File Preview + Editable Correction Grid      |
|  [ AI Copilot ]       - Chat Interface with Quick Suggested Prompts               |
+------------------------------------------+----------------------------------------+
                                           | Axios (JWT Bearer Interceptor)
                                           v
+-----------------------------------------------------------------------------------+
|                             BACKEND (Node.js + Express)                           |
|                                                                                   |
|  [ Middleware ]  - JWT Auth (7d expiry), Multer (10MB memory), Zod Validation     |
|  [ Controllers ] - Auth, Documents, Analytics Aggregation, AI Chat                |
|  [ Validator ]   - 7 Deterministic Audit Rules (Pure Code Engine)                 |
|  [ Gemini AI ]   - Multimodal Inline Base64 Data -> gemini-2.5-flash             |
+------------------------------------------+----------------------------------------+
                                           | Mongoose
                                           v
+-----------------------------------------------------------------------------------+
|                             DATABASE (MongoDB / Atlas)                            |
|                                                                                   |
|  - Users Collection     : { name, email, passwordHash, timestamps }               |
|  - Documents Collection : { userId, originalName, mimeType, fileData,             |
|                           extractedData, validationIssues, reviewStatus }         |
+-----------------------------------------------------------------------------------+
```

---

## ⚡ Tech Stack (Strictly Implemented)

- **Frontend**:
  - React.js 18 + Vite
  - React Router DOM v6
  - Tailwind CSS + Lucide React Icons
  - Axios (with centralized request/response auth interceptors)
  - Recharts (Spend by Vendor Bar Chart & Type Pie Chart)
- **Backend**:
  - Node.js (v20+) + Express.js
  - JWT Authentication (7-day signed tokens)
  - `bcryptjs` password hashing
  - `zod` schema validation for all endpoints
  - `multer` in-memory multipart upload
  - `mongoose` ODM with automatic in-memory fallback
- **AI Engine**:
  - Google Gemini API (`@google/generative-ai`)
  - Model: `gemini-2.5-flash`
  - Inline multimodal payload (PDFs, PNG, JPG, JPEG)
  - Schema-enforced JSON generation (`responseMimeType: "application/json"`)
- **Deployment**:
  - Client: Vercel (`vercel.json` SPA rewrite configuration included)
  - Server: Render (Production `start` script, `process.env.PORT`)

---

## 🔍 The 7 Deterministic Validation Rules

Every uploaded or edited document runs through `/server/src/services/validator.js` (plain JavaScript, zero LLM guesswork):

| # | Rule Code | Severity | Description |
|---|---|---|---|
| 1 | `MISSING_FIELDS` | **Error** | Flags missing required fields: `vendor_name`, `document_number`, `total`, or `issue_date`. |
| 2 | `LOW_CONFIDENCE` | **Warning** | Flags extractions where AI confidence score is below 0.70. |
| 3 | `LINE_ITEM_MATH` | **Error** | Verifies `quantity × unit_price = amount` (tolerance $0.01) and checks that sum of lines equals `subtotal`. |
| 4 | `TOTAL_MISMATCH` | **Error** | Flags discrepancy if `subtotal + tax !== total` (tolerance $0.01). |
| 5 | `DUPLICATE` | **Error** | Detects potential double-payment by matching same vendor + document #, or same vendor + total + date. |
| 6 | `PO_MISMATCH` | **Error / Warning** | Cross-references against matching Purchase Order (`po_reference`). Flags price variances, quantity overages, unauthorized items, or `PO_NOT_FOUND`. |
| 7 | `DATE_LOGIC` | **Error / Warning** | Flags errors if `due_date < issue_date`, or warnings if `issue_date` is in the future. |

*Note: Uploading or updating a Purchase Order triggers automatic re-validation across all related invoices referencing that PO.*

---

## 🚀 Setup & Local Installation

### Prerequisites
- Node.js v18+ (tested on Node v20/v24)
- npm v9+

### 1. Server Setup
```bash
cd server
npm install

# Copy environment variables
cp .env.example .env
```

Configure your `server/.env`:
```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/docupilot
JWT_SECRET=super_secret_jwt_key_docupilot_2026_change_in_production
CLIENT_URL=http://localhost:5173
GEMINI_API_KEY=your_gemini_api_key_here
```
*(If no local MongoDB daemon is running, DocuPilot automatically starts an in-memory database instance for zero-friction local development!)*

Start the backend:
```bash
npm run dev
# Server runs on http://localhost:5000
```

### 2. Client Setup
```bash
cd client
npm install

# Copy environment variables
cp .env.example .env
```

Start the frontend:
```bash
npm run dev
# Frontend runs on http://localhost:5173
```

---

## 🔑 Environment Variables Reference

### Backend (`/server/.env`)
| Variable | Required | Description | Example |
|---|---|---|---|
| `PORT` | Optional | Server port (default: 5000) | `5000` |
| `MONGODB_URI` | Optional | MongoDB connection string (Atlas or local) | `mongodb://localhost:27017/docupilot` |
| `JWT_SECRET` | **Required** | Secret key for signing JWT tokens | `super_secret_jwt_key...` |
| `CLIENT_URL` | **Required** | Frontend client origin for CORS | `http://localhost:5173` |
| `GEMINI_API_KEY` | Optional* | Google Gemini API key (*has mock fallback) | `AIzaSy...` |

### Frontend (`/client/.env`)
| Variable | Required | Description | Example |
|---|---|---|---|
| `VITE_API_URL` | Optional | API base URL for deployment | `http://localhost:5000/api` |

---

## 📡 API Endpoint Catalog

### Authentication (`/api/auth`)
- `POST /api/auth/register` — Register new user (`name`, `email`, `password`)
- `POST /api/auth/login` — Login user (`email`, `password`)
- `GET /api/auth/me` — Get current user profile (requires `Bearer <token>`)

### Documents (`/api/documents`)
- `POST /api/documents/upload` — Multi-file upload and Gemini multimodal extraction (max 10MB, PDF/images)
- `POST /api/documents/seed-sample` — Seeds 5 realistic demo documents with intentional audit discrepancies
- `GET /api/documents` — Search & filter documents (`?type=...&hasIssues=...&search=...`)
- `GET /api/documents/:id` — Retrieve full document details and line items
- `GET /api/documents/:id/file` — Inline binary stream of original file (PDF/image)
- `PATCH /api/documents/:id` — Save corrected fields and re-run deterministic validation
- `POST /api/documents/:id/approve` — Approve document workflow status
- `DELETE /api/documents/:id` — Delete document record

### Analytics (`/api/analytics`)
- `GET /api/analytics/summary` — Aggregated KPI metrics, spend by vendor, document type distribution, and recent audit issues

### AI Discovery Chat (`/api/chat`)
- `POST /api/chat` — Natural language question answering with document citations (`{ question }`)

### System (`/api/health`)
- `GET /api/health` — Returns server status, database connection, uptime, and timestamp

---

## 🌐 Deployment Instructions

### Deploy Frontend to Vercel
1. Set the root directory to `client`.
2. Build command: `npm run build`, output directory: `dist`.
3. Set environment variable: `VITE_API_URL=https://your-backend.onrender.com/api`.
4. The included `client/vercel.json` automatically handles client-side SPA routing rewrites.

### Deploy Backend to Render
1. Set the root directory to `server`.
2. Build command: `npm install`, start command: `npm start`.
3. Add environment variables in Render Dashboard:
   - `MONGODB_URI` (from MongoDB Atlas)
   - `JWT_SECRET` (secure random string)
   - `CLIENT_URL` (your Vercel frontend URL)
   - `GEMINI_API_KEY` (from Google AI Studio)

---

## 🎬 3-Minute Hackathon Demo Script

1. **Scaffold & Auth (0:00 - 0:30)**:
   - Open [http://localhost:5173](http://localhost:5173). Register a new account (`finance@docupilot.io`).
   - Notice instant JWT session issuance and protected dashboard routing.
2. **Instant Demo Seeder (0:30 - 1:00)**:
   - Click the prominent amber **"Load Sample Data"** button on the Dashboard.
   - 5 realistic financial documents are seeded into MongoDB with live validation execution.
3. **Analytics Dashboard (1:00 - 1:30)**:
   - Show the KPI cards: Total Documents, Total Spend, Flagged Discrepancies.
   - Highlight the **Spend by Vendor** bar chart and **Documents by Type** pie chart.
4. **Deterministic Audit & Side-by-Side Review (1:30 - 2:15)**:
   - Click into `INV_2055_Acme_MathMismatch.pdf` from the alerts list.
   - Point out the side-by-side view: original file preview on the left, deterministic rules on the right.
   - Show the red `TOTAL_MISMATCH` badge: Subtotal ($1,100) + Tax ($100) != Total ($1,450).
   - Change Total to `$1,200.00`, click **Save Corrections**: rules re-evaluate to green **"All Rules Passed"**.
   - Click **Approve Document**.
5. **PO Cross-Referencing & Duplicate Detection (2:15 - 2:40)**:
   - Open `INV_3088_Apex_POMismatch.pdf`: show how it caught a $1,700 price variance and unauthorized delivery item against `PO-2026-089`.
   - Open `INV_1041_CloudScale_Duplicate.pdf`: show how it stopped a duplicate invoice before payout.
6. **Natural Language Discovery Copilot (2:40 - 3:00)**:
   - Navigate to `/chat`. Click *"Which vendor did we spend the most on?"*.
   - Watch the copilot cite `CloudScale Technologies` with exact dollar figures audited from your files.
   - Ask an off-topic question: notice the AI strictly says `"not found in your documents"` with zero hallucinations.

---

*Built with ❤️ for the Hackathon by the DocuPilot Team.*
#   D o c u m e n t  
 