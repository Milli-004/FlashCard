# 🚀 CardVault — Student Management Suite

> A production-ready **MERN stack** web application built for managing student records — complete with secure JWT authentication, cloud image storage via Cloudinary, lightning-fast search, smart filtering, and seamless pagination.

---

## 💡 What Is This Project?

**CardVault** is a modern **Student Directory & Profile Manager** designed for educational institutions, training academies, and coaching centers that need a clean, centralized place to store and explore student data.

### The Problem It Solves
Managing student records in spreadsheets is messy. CardVault gives you a beautiful, searchable card-based interface where every student gets their own profile — with a photo, course info, contact details, and a short bio — all stored securely in the cloud.

### Who Is It For?
- 🎓 **College departments** keeping track of enrolled students semester by semester
- 🏋️ **Coaching institutes** organizing batches by year and subject
- 🏫 **School administrators** who need a fast, filterable people directory
- 👨‍💻 **Developers** looking for a polished MERN portfolio project to study or extend

### What Makes It Useful?
- Anyone can **browse and search** — no login needed
- Logged-in admins can **add, edit, or delete** student records
- A **stats bar** shows live counts of students, courses, and cities
- Photos are uploaded to **Cloudinary** with automatic face-detection cropping
- Everything works beautifully on **mobile, tablet, and desktop**

---

## 🗂️ Codebase Structure

```
CardVault/
├── README.md
│
├── backend/                          # Node.js + Express API server
│   ├── server.js                     # App entry — middleware, routes, error handler
│   ├── .env.example                  # Template for environment variables
│   ├── .gitignore
│   ├── package.json
│   └── src/
│       ├── config/
│       │   ├── db.js                 # Connects to MongoDB Atlas via Mongoose
│       │   └── cloudinary.js         # Cloudinary SDK config + Multer storage engine
│       ├── models/
│       │   ├── User.js               # User schema — bcrypt password hashing on save
│       │   └── Student.js            # Student schema — full-text index + field indexes
│       ├── controllers/
│       │   ├── authController.js     # Handles register, login, getMe
│       │   └── studentController.js  # Handles all CRUD, search, pagination, stats
│       ├── routes/
│       │   ├── authRoutes.js         # /api/auth/*
│       │   └── studentRoutes.js      # /api/students/*
│       └── middleware/
│           ├── auth.js               # JWT verification — protect + adminOnly
│           ├── errorMiddleware.js    # Global error catcher + 404 handler
│           └── validate.js           # Input validation rules via express-validator
│
└── frontend/                         # React + Vite client app
    ├── index.html
    ├── vite.config.js
    ├── .env.example
    ├── .gitignore
    ├── package.json
    └── src/
        ├── App.jsx                   # Route definitions
        ├── main.jsx                  # ReactDOM entry + Toaster setup
        ├── index.css                 # CSS variables, global resets, utility classes
        ├── utils/
        │   └── api.js                # Axios instance with JWT header injection + 401 handler
        ├── context/
        │   └── AuthContext.jsx       # Auth state — login, register, logout exposed via hook
        ├── components/
        │   ├── Navbar.jsx            # Sticky top bar with user pill and nav links
        │   ├── StudentCard.jsx       # Individual student card with initials fallback
        │   ├── StudentModal.jsx      # Add / Edit form modal with image preview
        │   └── Pagination.jsx        # Page controls with smart ellipsis logic
        └── pages/
            ├── Home.jsx              # Main view — stats, filters, card grid, pagination
            ├── Login.jsx             # Sign-in page
            └── Register.jsx          # Account creation page
```

---

## ⚙️ Getting Started

### Step 1 — Install Dependencies

```bash
# Install backend packages
cd backend
npm install

# Install frontend packages
cd ../frontend
npm install
```

### Step 2 — Set Up Backend Environment

```bash
cd backend
cp .env.example .env
```

Open `.env` and fill in your credentials:

```env
PORT=5000
NODE_ENV=development

# Your MongoDB Atlas connection string
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/cardvault_db

# A long random string used to sign JWT tokens
JWT_SECRET=replace_with_a_strong_random_secret
JWT_EXPIRES_IN=7d

# From your Cloudinary dashboard → Settings → API Keys
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Where your frontend is running (for CORS)
CLIENT_URL=http://localhost:5173
```

### Step 3 — Set Up Frontend Environment

```bash
cd frontend
cp .env.example .env
```

```env
VITE_API_URL=http://localhost:5000/api
```

### Step 4 — Start Both Servers

```bash
# Terminal 1 — API server on port 5000
cd backend
npm run dev

# Terminal 2 — React app on port 5173
cd frontend
npm run dev
```

Then open **http://localhost:5173** in your browser.

---

## 📡 API Documentation

### Base URL
```
http://localhost:5000/api
```

---

### 🔐 Authentication Endpoints

| Method | Route | What It Does | Login Required |
|--------|-------|-------------|:--------------:|
| `POST` | `/auth/register` | Creates a new account and returns a JWT | ✗ |
| `POST` | `/auth/login` | Validates credentials and returns a JWT | ✗ |
| `GET` | `/auth/me` | Returns the currently authenticated user | ✓ |

---

### 🎓 Student Endpoints

| Method | Route | What It Does | Login Required |
|--------|-------|-------------|:--------------:|
| `GET` | `/students` | Fetch all students with optional filters | ✗ |
| `GET` | `/students/stats` | Get total students, courses, and city counts | ✗ |
| `GET` | `/students/:id` | Fetch a single student record | ✗ |
| `POST` | `/students` | Create a new student (accepts photo upload) | ✓ |
| `PUT` | `/students/:id` | Update an existing student record | ✓ |
| `DELETE` | `/students/:id` | Delete a student and remove their photo from Cloudinary | ✓ |

---

### 🔍 Search & Filter Parameters

Append these to `GET /students` as query strings:

| Parameter | Type | Default | Example | Notes |
|-----------|------|---------|---------|-------|
| `search` | string | — | `?search=riya` | Searches name, course, and city |
| `course` | string | — | `?course=mba` | Case-insensitive regex match |
| `city` | string | — | `?city=lucknow` | Case-insensitive regex match |
| `year` | number | — | `?year=3` | Exact match, values 1–6 |
| `page` | number | `1` | `?page=2` | Pagination page number |
| `limit` | number | `9` | `?limit=12` | Results per page (max 50) |
| `sort` | string | `-createdAt` | `?sort=name` | Prefix `-` for descending order |

---

## 🔒 How Authentication Works

```
1. User registers  →  password is hashed (bcrypt, 12 rounds)  →  JWT returned
2. User logs in    →  password compared to hash  →  JWT returned
3. JWT stored in localStorage under the key  fc_token
4. All protected requests include the header:  Authorization: Bearer <token>
5. Tokens expire after 7 days
6. If the server returns 401  →  Axios interceptor clears storage and redirects to /login
```

---

## ✨ Full Feature List

| Feature | Description |
|---------|-------------|
| **JWT Authentication** | Stateless auth with 7-day expiry and auto-redirect on expiry |
| **Password Hashing** | bcryptjs with 12 salt rounds on every save |
| **MongoDB Atlas** | Cloud-hosted database with Mongoose schemas and text indexes |
| **Cloudinary Uploads** | Profile photos stored in the cloud with 400×400 face-crop |
| **Debounced Search** | 350ms debounce on full-text search — no request spam |
| **Multi-field Filtering** | Filter by course, city, and year independently or together |
| **Sort Options** | Sort by name, year, or creation date — ascending or descending |
| **Pagination** | Smart ellipsis page controls, configurable page size |
| **Stats Dashboard** | Live counts of total students, unique courses, and cities |
| **Rate Limiting** | 100 req / 15 min globally · 10 req / 15 min on auth routes |
| **Security Headers** | Helmet.js applied to every response |
| **Input Validation** | express-validator checks every field before hitting the database |
| **Global Error Handler** | Catches Mongoose, JWT, and validation errors with clean JSON output |
| **Skeleton Loaders** | Shimmer placeholders shown while data is fetching |
| **Toast Notifications** | Success and error toasts via react-hot-toast |
| **Responsive Design** | Fluid grid layout works on all screen sizes |
| **Auto Logout** | Axios response interceptor handles token expiry silently |

---

## 🐛 Bug Fixes — v1.1

The following issues were identified and resolved after the initial release:

| # | File | Issue Found | Resolution Applied |
|---|------|-------------|-------------------|
| 1 | `AuthContext.jsx` | `api` was imported without the `.js` extension, which breaks strict ESM module resolution | Updated import path to `'../utils/api.js'` |
| 2 | `AuthContext.jsx` | `useEffect` was listed in the React imports but never actually used in the file | Removed it from the import statement |
| 3 | `Navbar.jsx` | `useState` was listed in the React imports but never used in the component | Removed it from the import statement |
| 4 | `Home.jsx` | **Critical — Double API call on page load.** Both the filter `useEffect` and the page `useEffect` were firing at the same time on mount, causing two simultaneous network requests | Introduced a `filterChangedRef` boolean flag; when filters trigger a fetch and reset the page, the flag prevents the page `useEffect` from firing a second time |
| 5 | `Home.jsx` | `fetchStudents` was wrapped in `useCallback` with all filter values as dependencies, which meant the page `useEffect` (which depended on `fetchStudents`) re-ran on every filter change — doubling every filter-triggered fetch | Removed `useCallback` entirely; `fetchStudents` now accepts an explicit `overridePage` argument instead of reading stale closure values |
| 6 | `Home.jsx` | `handleSaved` called `setPage(1)` after `fetchStudents(1)`, which caused the page `useEffect` to trigger a redundant second API request | `handleSaved` now sets `filterChangedRef.current = true` before calling `setPage(1)` to suppress the follow-up effect |
| 7 | Both | Neither `backend/` nor `frontend/` had a `.gitignore` file, meaning `node_modules/` and `.env` secrets could be accidentally pushed to version control | Added a `.gitignore` to both directories |

---

## 👩‍💻 Developer

**Milli Srivastava** — Full-Stack MERN Developer

---

*Built with passion and precision — CardVault by Milli Srivastava*
