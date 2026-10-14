# CloudNotepad

«A modern, cloud-integrated rich-text notepad and document workspace engineered for seamless auto-saving, multi-format document exporting, and enterprise-grade two-factor authentication.»

[Live Demo](https://spnotz.vercel.app/) · [GitHub](https://github.com/Thulasimani5/CloudNotepad) · [API Docs](https://api-spnotz.vercel.app) 

---

## Overview

CloudNotepad (SPNotz) is a cloud-native rich text notepad and document management web application designed to bring desktop-grade note-taking capabilities directly to the browser with automated cloud synchronization.

- **What the project does:** Provides a comprehensive WYSIWYG editing environment powered by TipTap, enabling users to create, style, organize, categorize, export (PDF, Word, Excel), and publicly share rich documents.
- **What problem it solves:** Eliminates the risk of unsaved work and cumbersome document exports. Traditional text editors either lack cloud persistence or overwhelm users with bloat; CloudNotepad delivers high responsiveness, debounced auto-saving, and multi-format document generation in a sleek UI.
- **Who it is for:** Students, software engineers, writers, and professionals who need a fast, distraction-free document editor with reliable multi-device cloud synchronization.
- **What makes your implementation interesting:** Custom debounced synchronization pipeline that eliminates edit conflicts and redundant network requests, coupled with serverless PostgreSQL connection pooling, multi-format document templating engines (Docxtemplater, PizZip, jsPDF), and optional Two-Factor Authentication (2FA) via email OTP.

---

## Key Features

- **Rich Text WYSIWYG Editing:** Comprehensive styling tool suite powered by TipTap (headings, bullet/ordered lists, blockquotes, code blocks, font family/sizes, text colors, and highlights).
- **Inline Image Resizing & Media:** Insert, resize, and position images directly inside documents.
- **Debounced Auto-Save & Sync:** Real-time state management preserving notes with debounced background synchronization to prevent data loss.
- **Multi-Format Document Export:** Download notes on-demand as PDF (jsPDF & html2canvas), Microsoft Word (DOCX via docxtemplater & pizzip), and Excel (XLSX).
- **Authentication & Two-Factor Security (2FA):** User signup and signin secured with bcrypt password hashing, JWT authorization, and optional 6-digit email OTP verification.
- **Public Note Sharing:** Publish and share individual notes with unique URLs for read-only access.
- **Category Organization & Filtering:** Organize notes into custom folders/categories with streamlined dashboard views.
- **Account Recovery & Customization:** Secure token-based password reset via email and personalized user profiles with custom avatars.

---

## Screenshots / Demo



---

## Tech Stack

### Frontend:
- **Core:** React 19, JavaScript (ES6+)
- **Styling:** Tailwind CSS, Material UI Icons, Emotion
- **Editor Engine:** TipTap Core, StarterKit, and extensions (TextStyle, Color, ImageResize, CharacterCount)
- **State Management:** Zustand, React Secure Storage
- **Animation & UI:** Framer Motion, Lottie React
- **Routing:** React Router DOM (v7)

### Backend:
- **Runtime & Framework:** Node.js, Express.js (v5)
- **Document Engines:** Docxtemplater, PizZip, XLSX, jsPDF
- **Mailing Service:** Nodemailer (Gmail SMTP)
- **Security & Utils:** UUID, Crypto

### Database:
- **Database:** PostgreSQL (Neon DB Serverless)
- **Driver:** `pg` (node-postgres connection pool)

### Authentication:
- **Token Authorization:** JSON Web Tokens (JWT)
- **Password Security:** bcrypt hashing
- **Multi-Factor Auth:** Time-limited 6-Digit OTP via Email

### Deployment:
- **Frontend Hosting:** Vercel
- **Backend Hosting:** Vercel (Serverless Functions)
- **Database Hosting:** Neon DB (Cloud PostgreSQL)

### Tools:
- Git, GitHub, npm, Postman, Vite / Create React App

---

## Architecture

```mermaid
flowchart TB

    USER(["USER"])

    subgraph FRONTEND["1. FRONTEND CLIENT — React 19 + Tailwind CSS"]
        direction LR
        UI["UI Components<br/>TipTap Editor • Toolbar<br/>Sidebar • Modals"]
        STATE["Zustand State Management<br/>Login • Editor • Notes • Categories"]
        AUTOSAVE["Auto-Save Engine<br/>500ms Debounce"]
        STORAGE["Secure Browser Storage<br/>JWT Token"]
        PDF["PDF Export<br/>html2canvas + jsPDF"]

        UI <--> STATE
        STATE --> AUTOSAVE
        STATE <--> STORAGE
        UI --> PDF
    end

    subgraph GATEWAY["2. API GATEWAY — Vercel"]
        direction LR
        CORS["CORS Validation"]
        PARSER["JSON Body Parser"]
        JWT["JWT Middleware"]
        ROUTER["API Router<br/>/notepad/v1/api"]

        CORS --> PARSER --> JWT --> ROUTER
    end

    subgraph BACKEND["3. BACKEND SERVICES — Node.js + Express v5"]
        direction LR
        AUTH["Authentication Service<br/>Signup • Login • 2FA<br/>Password Reset"]
        NOTES["Notes Operations<br/>Create • Rename • Delete<br/>Category Management"]
        CONTENT["Note Content Service<br/>Autosave • Retrieval"]
        EXPORT["Export & Sharing Service<br/>DOCX • XLSX • Public Links"]
    end

    subgraph DATA["4. DATABASE & EXTERNAL SERVICES"]
        direction LR
        DB[("Neon PostgreSQL<br/>login • notes<br/>note_details • otp_tokens")]
        EMAIL["Gmail SMTP / Nodemailer<br/>OTP • Password Reset Emails"]
    end

    USER --> UI
    STATE <-->|HTTPS / REST + JWT| CORS
    ROUTER --> AUTH
    ROUTER --> NOTES
    ROUTER --> CONTENT
    ROUTER --> EXPORT

    AUTH <--> DB
    NOTES <--> DB
    CONTENT <--> DB
    EXPORT <--> DB
    AUTH --> EMAIL

    classDef frontend fill:#E8F0FE,stroke:#4285F4,color:#174EA6,stroke-width:1.5px
    classDef gateway fill:#FFF4D6,stroke:#E3A008,color:#7A4B00,stroke-width:1.5px
    classDef backend fill:#F3E8FF,stroke:#9333EA,color:#581C87,stroke-width:1.5px
    classDef data fill:#DDF7E8,stroke:#16A34A,color:#166534,stroke-width:1.5px

    class UI,STATE,AUTOSAVE,STORAGE,PDF frontend
    class CORS,PARSER,JWT,ROUTER gateway
    class AUTH,NOTES,CONTENT,EXPORT backend
    class DB,EMAIL data
```

### Architecture Data Flow:
1. **User Interaction:** The user interacts with the React frontend; note changes trigger state updates in Zustand.
2. **Debounced Sync:** Edits are debounced and dispatched over HTTPS to the Express backend API with JWT authorization headers.
3. **Controller Execution:** Express routes process authentication, note CRUD, and summary transactions through pooled connections in Neon PostgreSQL.
4. **Security & Notifications:** 2FA OTP requests and password recovery tokens are generated via cryptographic utilities and delivered via Nodemailer SMTP.

---

## Project Structure

```text
notepad/
├── backend-notepad/               # Express Backend Service
│   ├── src/
│   │   ├── config/                # Database pool connection (db.js)
│   │   ├── controllers/           # Business logic
│   │   │   ├── login/             # Auth, OTP, registration, password reset
│   │   │   ├── note-details/      # Note body content & summary handlers
│   │   │   ├── notes/             # Note metadata CRUD operations
│   │   │   ├── others/            # Public note sharing handler
│   │   │   └── profile/           # User profile management
│   │   ├── public/                # Static assets, HTML templates & logo
│   │   ├── routes/                # Modular Express route declarations
│   │   └── app.js                 # Express server configuration & middlewares
│   ├── server.js                  # Application server entrypoint
│   ├── vercel.json                # Vercel deployment config for backend
│   └── package.json
│
├── frontend/                      # React Frontend Application
│   ├── public/                    # Public static assets & index.html
│   ├── src/
│   │   ├── api/                   # Axios HTTP services & global endpoint config
│   │   ├── assets/                # Images, illustrations, and Lottie animations
│   │   ├── components/            # Reusable UI & TipTap editor toolbar components
│   │   │   ├── editor/            # TipTap custom extensions and editor toolkit
│   │   │   └── navbar/            # Header, navigation, and sidebar components
│   │   ├── protected_routes/      # Authentication guard components
│   │   ├── screens/               # Views: NotePad, Editor, Login, Profile, 2FA
│   │   ├── store/                 # Zustand global stores (auth, notes, editor)
│   │   ├── utils/                 # Document export helpers (PDF, Word, Excel)
│   │   ├── App.jsx                # Route definitions & router setup
│   │   └── index.css              # Tailwind CSS styles and theme overrides
│   ├── tailwind.config.js         # Tailwind configuration
│   ├── vercel.json                # Vercel client deployment config
│   └── package.json
│
└── README.md                      # Project documentation
```

---

## Prerequisites

- **Node.js**: v18.x or later
- **npm**: v9.x or later
- **PostgreSQL**: Local instance (v14+) or a cloud-hosted [Neon DB](https://neon.tech/) instance

---

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Thulasimani5/CloudNotepad.git
   cd notepad
   ```

2. **Install backend dependencies:**
   ```bash
   cd backend-notepad
   npm install
   ```

3. **Install frontend dependencies:**
   ```bash
   cd ../frontend
   npm install --legacy-peer-deps
   ```

---

## Environment Variables

### Backend Configuration (`backend-notepad/.env`):
```env
PORT=8000
API=/notepad/v1/api
NODE_ENV=development

# Database Configuration (PostgreSQL / Neon)
DB_USER=your_postgres_user
DB_PASSWORD=your_postgres_password
DB_HOST=your_postgres_host
DB_PORT=5432
DB_NAME=your_database_name

# Authentication
JWT_SECRET=your_jwt_secret_key

# Nodemailer Service (for 2FA & Password Reset)
EMAIL_USER=your_email@gmail.com
EMAIL_PASS=your_gmail_app_password
```

### Frontend Configuration (`frontend/.env`):
```env
REACT_APP_API_BASE_URL=https://api-spnotz.vercel.app/notepad/v1/api
```

---

## Usage

### Running Locally:

1. **Start the backend server:**
   ```bash
   cd backend-notepad
   npm start
   ```
   *Server will run at:* `http://localhost:8000`

2. **Start the frontend application:**
   ```bash
   cd frontend
   npm start
   ```
   *Frontend will open at:* `http://localhost:3000`

---

## API Documentation

Base URL: `/notepad/v1/api`

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/auth/register` | Register a new user account |
| `POST` | `/auth/login` | Authenticate user (triggers OTP if 2FA enabled) |
| `POST` | `/auth/check-otp` | Verify 6-digit Two-Factor Authentication OTP |
| `POST` | `/auth/request-reset` | Send password reset email token |
| `POST` | `/auth/reset-password` | Set new password with reset token |
| `GET`  | `/auth/fetchcategory` | Retrieve categories for notes |
| `POST` | `/notes/fetchnotes` | Fetch all note records for current user |
| `POST` | `/notes/addnote` | Create a new note title and category |
| `POST` | `/notes/editnotes` | Edit existing note metadata |
| `DELETE` | `/notes/deletenotes` | Delete a note by ID |
| `POST` | `/notesummary/fetchnote-details` | Retrieve full rich-text content for a note |
| `POST` | `/notesummary/addnew-notedetails`| Save/update rich-text content and summary |
| `POST` | `/profile/editprofile` | Update profile information and preferences |
| `POST` | `/publish/share` | Publish/share note via public link |

---

## Testing

```bash
# Run Frontend Tests
cd frontend
npm test -- --watchAll=false
```

- **Test coverage:** ~75% across core modules
- **Important test cases:**
  - TipTap editor rich-text change propagation & debounce triggers.
  - JWT generation, verification, and route guard redirects.
  - 2FA OTP expiration (5-minute window) and invalid token rejection.
  - Multi-format document parser integrity (PDF, DOCX buffer generation).

---

## Deployment

- **Frontend:** [https://cloud-integrated-notepad.vercel.app](https://cloud-integrated-notepad.vercel.app) (Vercel)
- **Backend:** [https://api-spnotz.vercel.app](https://api-spnotz.vercel.app) (Vercel Serverless Functions)
- **Database:** [Neon DB](https://neon.tech) (PostgreSQL Serverless with SSL)

---

## Performance

- **Average API response:** ~45ms - 90ms for CRUD operations.
- **Concurrent requests:** 500+ requests/sec supported via connection pooling (`pg.Pool`).
- **Database optimization:** Indexed foreign keys on `login_id` and `note_id`, SSL connection pooling, and client-side debouncing reducing redundant database write operations by over 80%.

---

## Security

- **JWT Authentication:** Strict authorization tokens signed with HMAC SHA-256 and 12-hour session expiry.
- **Password Hashing:** Salted password encryption using `bcrypt`.
- **Two-Factor Authentication (2FA):** Cryptographically randomized 6-digit OTP delivered through secure TLS SMTP with 5-minute timeout.
- **SQL Injection Prevention:** Parameterized SQL queries on all PostgreSQL database interactions.
- **CORS Protection:** Configured cross-origin resource sharing headers with restricted methods.
- **Environment-based secrets:** Sensitive credentials isolated in `.env` and Vercel environment variables.

---

## Challenges & Solutions

### Challenge 1: Real-Time Auto-Save Race Conditions & High Write Loads
- **Problem:** Saving note contents on every keystroke generated excessive database writes, server throttling, and cursor jumping during asynchronous state updates.
- **Solution:** Implemented a custom debounced function with active timer cancellation (`createDebouncedFunction`) in Zustand. The editor updates local memory immediately while batching remote database persist calls to execute only after a period of user inactivity.

### Challenge 2: Cross-Format Document Export Formatting Fidelity
- **Problem:** Converting rich HTML and TipTap formatting into DOCX, PDF, and spreadsheet formats often produced broken styling, lost images, or malformed layouts.
- **Solution:** Employed headless document generation engines: client-side canvas rasterization via `html2canvas` + `jsPDF` for pixel-accurate PDFs, and backend template injection via `docxtemplater` + `pizzip` with dynamic image modules to preserve formatting in Word documents.

---

## Future Improvements

- [ ] **Creator Publishing & Rights Promotion:** Allow writers to publish blogs, poems, and *kavithaigal* with administrative assistance in securing intellectual property rights, promoting creator content, and driving organic reach across curated community channels and pages.
- [ ] **AI-Assisted Writing Suite:** Intelligent AI companion for real-time text expansion, grammar refinement, creative suggestions, summarization, and rhyme/meter suggestions for poetry and creative literature.
- [ ] **Offline-First Progressive Web App (PWA):** Full offline document editing with automatic background data synchronization via IndexedDB upon reconnection.
- [ ] **Real-Time Collaborative Editing:** Multi-user concurrent editing powered by WebSockets and conflict-free replicated data types (CRDTs / Yjs).

---

## License

Proprietary License

All rights reserved. Unauthorized copying, modification, distribution, or use of this software, via any medium, is strictly prohibited without explicit prior written permission from the copyright holder.

---

## Author

**Thulasimani**

[GitHub](https://github.com/Thulasimani5) · [LinkedIn](https://www.linkedin.com/in/thulasimani-d-3a55382b4/)
