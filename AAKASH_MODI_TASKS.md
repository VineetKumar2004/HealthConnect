# 📋 Aakash Modi — Team Leader Task Sheet

**Project:** HealthConnect  
**Role:** Team Leader — UI/UX, Frontend Features & Integration  
**Tech Stack:** React 18.x, Vite, React Router v6, Axios, Lucide Icons, CSS  
**Dev Server:** `http://localhost:5173`  
**Backend API:** `http://localhost:5000/api`

---

## 📌 Table of Contents

1. [Project Setup](#-1-project-setup)
2. [Architecture Overview](#-2-architecture-overview)
3. [Reusable Components](#-3-reusable-components)
4. [All Pages — Complete List](#-4-all-pages--complete-list)
5. [API Integration Guide](#-5-api-integration-guide)
6. [AI Prompts for Frontend Pages](#-6-ai-prompts-for-frontend-pages)
7. [Integration Tasks (Frontend + Backend + Database)](#-7-integration-tasks)

---

## 🛠 1. Project Setup

### How to run:
```bash
cd medTech-frontend
npm install
npm run dev                    # Starts Vite dev server on port 5173
```

### Key Dependencies (`package.json`):
- `react`, `react-dom` — UI library
- `react-router-dom` — Client-side routing
- `axios` — HTTP client for API calls
- `lucide-react` — Icon library
- `vite` — Build tool & dev server

### Environment Variables (`.env`):
```
VITE_API_URL=http://localhost:5000/api
```

---

## 🏗 2. Architecture Overview

### Folder Structure:
```
src/
├── App.jsx                   ← Main app + all routes
├── main.jsx                  ← React entry point
├── components/
│   └── common/               ← 10 reusable UI components
│       ├── Button.jsx
│       ├── Card.jsx
│       ├── Footer.jsx
│       ├── Input.jsx
│       ├── LoadingSpinner.jsx
│       ├── Modal.jsx
│       ├── Navbar.jsx
│       ├── Sidebar.jsx
│       ├── Table.jsx
│       ├── ThemeToggle.jsx
│       └── index.js          ← Barrel export
├── contexts/
│   ├── AuthContext.jsx        ← Auth state (login/register/logout)
│   └── ThemeContext.jsx       ← Theme state (Light/Dark mode)
├── pages/
│   ├── public/                ← 4 public pages (no auth)
│   ├── patient/               ← 4 patient pages
│   ├── doctor/                ← 5 doctor pages
│   ├── pharmacy/              ← 2 pharmacy pages
│   └── admin/                 ← 2 admin pages
├── services/
│   └── api.js                 ← Axios instance + all API endpoints
├── styles/
│   └── index.css              ← Global CSS design system
└── utils/
    └── validation.js          ← Form validation helpers
```

### Routing Map (`App.jsx`):

| Route                         | Component            | Auth Required | Role        |
|-------------------------------|----------------------|:------------:|:-----------:|
| `/`                           | `LandingPage`        | ❌           | —           |
| `/login`                      | `LoginPage`          | ❌           | —           |
| `/register`                   | `PatientRegister`    | ❌           | —           |
| `/about`                      | `AboutPage`          | ❌           | —           |
| `/patient/dashboard`          | `PatientDashboard`   | ✅           | `patient`   |
| `/patient/records`            | `MyRecords`          | ✅           | `patient`   |
| `/patient/prescriptions`      | `MyPrescriptions`    | ✅           | `patient`   |
| `/patient/profile`            | `PatientProfile`     | ✅           | `patient`   |
| `/doctor/dashboard`           | `DoctorDashboard`    | ✅           | `doctor`    |
| `/doctor/search`              | `SearchPatient`      | ✅           | `doctor`    |
| `/doctor/patient/:id`         | `PatientDetails`     | ✅           | `doctor`    |
| `/doctor/visits/create`       | `CreateVisit`        | ✅           | `doctor`    |
| `/doctor/prescriptions/create`| `WritePrescription`  | ✅           | `doctor`    |
| `/pharmacy/dashboard`         | `PharmacyDashboard`  | ✅           | `pharmacy`  |
| `/pharmacy/verify`            | `VerifyPrescription` | ✅           | `pharmacy`  |
| `/admin/dashboard`            | `AdminDashboard`     | ✅           | `admin`     |
| `/admin/hospitals`            | `ManageHospitals`    | ✅           | `admin`     |
| `*`                           | Redirect to `/`      | —            | —           |

### Protected Route Pattern:
```jsx
<ProtectedRoute allowedRoles={['patient']}>
  <PatientDashboard />
</ProtectedRoute>
```
- Shows loading spinner while checking auth
- Redirects to `/login` if not authenticated
- Redirects to `/` if role doesn't match

---

## 🧩 3. Reusable Components

Located in `src/components/common/`:

| Component        | Purpose                                          | Key Props                                      |
|------------------|--------------------------------------------------|------------------------------------------------|
| `Button`         | Styled button with variants                      | `variant` (primary/secondary/ghost/danger), `icon`, `size`, `loading`, `disabled` |
| `Card`/`CardBody`| Container card with optional hover               | `hover`, `className`, `children`               |
| `Input`          | Form input with label and error                  | `label`, `type`, `error`, `icon`, `placeholder`|
| `Modal`          | Overlay modal dialog                             | `isOpen`, `onClose`, `title`, `children`       |
| `Navbar`         | Top navigation bar with auth state               | `onMenuClick`, `showMenu`                      |
| `Sidebar`        | Collapsible side navigation                      | `links[]`, `isOpen`, `onClose`                 |
| `Footer`         | Page footer                                      | —                                              |
| `Table`          | Data table component                             | `columns`, `data`, `onRowClick`                |
| `LoadingSpinner` | Loading animation                                | `size`, `color`                                |
| `ThemeToggle`    | Light/dark mode switch                           | Uses `ThemeContext`                             |

---

## 📄 4. All Pages — Complete List

### 4.1 Public Pages (No Auth Required)

---

#### Page 1: `LandingPage.jsx` — Route: `/`
**File:** `src/pages/public/LandingPage.jsx` (397 lines)  
**Purpose:** Main marketing/landing page for HealthConnect.

**Sections:**
- Hero section with headline, CTA buttons (Login / Register)
- Features grid (6 cards): Patients, Doctors, Hospitals, Pharmacy, Security, Mobile-ready
- How It Works section (3-step flow)
- Statistics section (users, hospitals, prescriptions)
- Footer

**UI Elements:** Navbar, Footer, Button, Card, CardBody  
**Icons:** Heart, Shield, Users, Hospital, Pill, FileText, ArrowRight, CheckCircle, Smartphone, Lock, Globe

---

#### Page 2: `LoginPage.jsx` — Route: `/login`
**File:** `src/pages/public/LoginPage.jsx` (336 lines)  
**Purpose:** Multi-role login page with role selector.

**Features:**
- Role selector tabs: Patient, Doctor, Pharmacy, Admin (each with unique color)
- Email + Password form with validation
- "Forgot Password?" link
- "Register" link for patients
- Error display, loading state
- Post-login redirect based on role (`/patient/dashboard`, `/doctor/dashboard`, etc.)

**State:** `formData`, `errors`, `isLoading`, `selectedRole`  
**API:** `authAPI.login(credentials)`

---

#### Page 3: `PatientRegister.jsx` — Route: `/register`
**File:** `src/pages/public/PatientRegister.jsx`  
**Purpose:** New patient registration form.

**Fields:**
- Full Name, Email, Phone, Password, Confirm Password
- Date of Birth, Gender (M/F/O)
- Government ID (optional)
- Terms & Conditions checkbox

**Post-registration:** Auto-login → redirect to `/patient/dashboard`  
**API:** `authAPI.register(userData)`

---

#### Page 4: `AboutPage.jsx` — Route: `/about`
**File:** `src/pages/public/AboutPage.jsx`  
**Purpose:** Information page about the platform, team, mission.

---

### 4.2 Patient Pages (Role: `patient`)

---

#### Page 5: `PatientDashboard.jsx` — Route: `/patient/dashboard`
**File:** `src/pages/patient/PatientDashboard.jsx` (209 lines)  
**Purpose:** Patient's main dashboard after login.

**Sections:**
- Welcome header with patient name and date
- Stats cards: Upcoming Appointments, Active Prescriptions, Medical Records, Health Score
- Recent activity timeline
- Quick action links: View Records, View Prescriptions, Edit Profile

**Sidebar Links:** Dashboard, My Records, Prescriptions, Profile  
**Layout:** Sidebar + Navbar + Main Content  
**API:** `patientAPI.getProfile()`, `patientAPI.getRecords()`, `patientAPI.getPrescriptions()`

---

#### Page 6: `MyRecords.jsx` — Route: `/patient/records`
**File:** `src/pages/patient/MyRecords.jsx`  
**Purpose:** View all medical records with filters.

**Features:**
- Filter bar: Date range (From / To), Record type dropdown
- Record type filter: All, Lab, Scan, Discharge, Prescription, Vaccination, Surgery, Other
- Records list with: title, date, hospital name, record type badge, summary
- Empty state when no records found

**API:** `patientAPI.getRecords({ dateFrom, dateTo, type })`

---

#### Page 7: `MyPrescriptions.jsx` — Route: `/patient/prescriptions`
**File:** `src/pages/patient/MyPrescriptions.jsx`  
**Purpose:** View all prescriptions with status filter.

**Features:**
- Status filter tabs: All, Pending, Dispensed, Expired
- Prescription cards with: RX code, doctor name, diagnosis, medicines list, status badge, valid until date
- Medicine details: name, dosage, frequency, duration

**API:** `patientAPI.getPrescriptions({ status })`

---

#### Page 8: `PatientProfile.jsx` — Route: `/patient/profile`
**File:** `src/pages/patient/PatientProfile.jsx`  
**Purpose:** View and edit patient profile.

**Sections:**
- Profile info display: UHID, Name, Email, Phone, DOB, Gender, Age
- Address section: Address, City, State, Pincode
- Medical info: Blood Group, Allergies, Chronic Conditions, Emergency Contact
- Edit mode toggle with save/cancel

**API:** `patientAPI.getProfile()`, `patientAPI.updateProfile(data)`

---

### 4.3 Doctor Pages (Role: `doctor`)

---

#### Page 9: `DoctorDashboard.jsx` — Route: `/doctor/dashboard`
**File:** `src/pages/doctor/DoctorDashboard.jsx` (235 lines)  
**Purpose:** Doctor's main dashboard.

**Sections:**
- Welcome header with doctor name and specialty
- Stats cards: Patients Today, Prescriptions Written, Pending Visits, Appointments
- Today's appointments list with patient name, UHID, time, status
- Quick actions: Search Patient, Create Visit, Write Prescription

**Sidebar Links:** Dashboard, Search Patient, Appointments, Create Visit, Prescriptions  
**API:** `doctorAPI.getAppointments()`

---

#### Page 10: `SearchPatient.jsx` — Route: `/doctor/search`
**File:** `src/pages/doctor/SearchPatient.jsx`  
**Purpose:** Search for patients by Health ID, phone, or govt ID.

**Features:**
- Search type selector: Health ID, Phone Number, Government ID
- Search input with icon
- Results list: Patient name, UHID, age, gender
- Click result → navigate to `/doctor/patient/:id`
- No results state

**API:** `doctorAPI.searchPatient({ type, value })`

---

#### Page 11: `PatientDetails.jsx` — Route: `/doctor/patient/:id`
**File:** `src/pages/doctor/PatientDetails.jsx`  
**Purpose:** Full patient details view for doctors.

**Sections:**
- Patient info header: Name, UHID, Age, Gender, Blood Group
- Medical info: Allergies, Chronic Conditions
- Visit history (recent visits with symptoms, diagnosis, vitals)
- Prescription history (with RX codes, status, medicines)
- Action buttons: Create Visit, Write Prescription

**API:** `doctorAPI.getPatientDetails(patientId)`

---

#### Page 12: `CreateVisit.jsx` — Route: `/doctor/visits/create`
**File:** `src/pages/doctor/CreateVisit.jsx`  
**Purpose:** Form to record a new patient visit/consultation.

**Form Fields:**
- Patient ID / UHID (with search/lookup)
- Symptoms (textarea)
- Diagnosis (textarea)
- Notes (textarea, optional)
- Vitals section (all optional):
  - Blood Pressure (Systolic / Diastolic)
  - Pulse (BPM)
  - Temperature (°C)
  - Weight (kg), Height (cm)
  - Oxygen Saturation (SpO2 %)

**API:** `doctorAPI.createVisit(visitData)`

---

#### Page 13: `WritePrescription.jsx` — Route: `/doctor/prescriptions/create`
**File:** `src/pages/doctor/WritePrescription.jsx`  
**Purpose:** Create an e-prescription with medicines.

**Form Fields:**
- Patient ID / UHID
- Visit ID (optional, link to existing visit)
- Diagnosis
- Notes (optional)
- Validity period (days, default 30)
- Medicines (dynamic list, add/remove):
  - Medicine Name
  - Dosage (e.g., 500mg)
  - Frequency (e.g., 3x daily)
  - Duration (e.g., 5 days)
  - Quantity
  - Instructions (e.g., After meals)

**Post-creation:** Shows generated RX code  
**API:** `doctorAPI.createPrescription(rxData)`

---

### 4.4 Pharmacy Pages (Role: `pharmacy`)

---

#### Page 14: `PharmacyDashboard.jsx` — Route: `/pharmacy/dashboard`
**File:** `src/pages/pharmacy/PharmacyDashboard.jsx` (82 lines)  
**Purpose:** Pharmacy dashboard with daily stats.

**Sections:**
- Welcome header with pharmacy name and date
- Stats cards: Verified Today, Pending, Rejected
- Recent prescriptions list: RX code, patient name, doctor, medicine count, time

**Sidebar Links:** Dashboard, Verify Prescription, Dispensed Today  
**API:** `pharmacyAPI.getHistory()`

---

#### Page 15: `VerifyPrescription.jsx` — Route: `/pharmacy/verify`
**File:** `src/pages/pharmacy/VerifyPrescription.jsx`  
**Purpose:** Verify and dispense prescriptions by RX code.

**Features:**
- RX Code input field with search button
- Prescription details display:
  - Validity status (Valid ✅ / Invalid ❌)
  - Patient name, UHID
  - Doctor name, Hospital
  - Diagnosis
  - Medicines list with dosage, frequency, duration
  - Expiry date
- "Dispense" button (with optional notes)
- Status messages: Already dispensed, Expired, Cancelled

**API:** `pharmacyAPI.verifyPrescription(rxCode)`, `pharmacyAPI.markDispensed(rxCode, data)`

---

### 4.5 Admin Pages (Role: `admin`)

---

#### Page 16: `AdminDashboard.jsx` — Route: `/admin/dashboard`
**File:** `src/pages/admin/AdminDashboard.jsx` (112 lines)  
**Purpose:** System-wide admin dashboard.

**Sections:**
- Stats grid: Hospitals, Doctors, Pharmacies, Patients (with trend indicators)
- Quick Actions grid: Add Hospital, Add Doctor, Add Pharmacy, View Logs
- Recent Activity timeline
- Today's Statistics: Prescriptions, Visits, New Patients, Uptime

**Sidebar Links:** Dashboard, Hospitals, Doctors, Pharmacies, System Logs  
**API:** `adminAPI.getStats()`

---

#### Page 17: `ManageHospitals.jsx` — Route: `/admin/hospitals`
**File:** `src/pages/admin/ManageHospitals.jsx`  
**Purpose:** CRUD management for hospitals.

**Features:**
- Hospitals table: Name, Type, Registration No., City, Status
- Filters: Status dropdown, Type dropdown
- Add Hospital button → opens modal form
- Edit/Delete actions per row
- Hospital form modal: Name, Type, Registration Number, Address, City, State, Pincode, Phone, Email, Status

**API:** `adminAPI.hospitals.list()`, `adminAPI.hospitals.create()`, `adminAPI.hospitals.update()`, `adminAPI.hospitals.delete()`

---

## 🔌 5. API Integration Guide

### API Service (`src/services/api.js`)

The API layer is already structured with Axios. Currently uses **mock auth** in `AuthContext.jsx` — needs to be connected to the Flask backend.

#### Auth Flow (Integration Steps):
1. **Login:** Call `authAPI.login()` → store `access` token in `localStorage` → set user state
2. **Register:** Call `authAPI.register()` → auto-login on success → redirect to dashboard
3. **JWT Token:** Auto-attached by Axios interceptor (`Authorization: Bearer <token>`)
4. **401 Handling:** Auto-redirects to `/login` and clears stored auth
5. **Token Refresh:** Call `api.post('/auth/token/refresh/', { refresh })` when access token expires

#### API Modules (already defined in `api.js`):

| Module          | Functions                                                     |
|-----------------|---------------------------------------------------------------|
| `authAPI`       | `login()`, `register()`, `forgotPassword()`, `requestOTP()`  |
| `patientAPI`    | `getProfile()`, `updateProfile()`, `getRecords()`, `getPrescriptions()` |
| `doctorAPI`     | `searchPatient()`, `getPatientDetails()`, `createVisit()`, `createPrescription()`, `getAppointments()` |
| `pharmacyAPI`   | `verifyPrescription()`, `markDispensed()`, `getHistory()`     |
| `adminAPI`      | `getStats()`, `hospitals.list/create/update/delete()`, `doctors.list/create/update/delete()`, `pharmacies.list/create/update/delete()`, `getLogs()` |

---

## 🤖 6. AI Prompts for Frontend Pages

Below are copy-paste-ready **AI prompts** for building each page:

---

### Prompt 1 — Landing Page
```
Create a React landing page component for HealthConnect (a digital healthcare platform).
Use React Router Link, Lucide icons (Heart, Shield, Users, Hospital, Pill, FileText, ArrowRight, CheckCircle, Smartphone, Lock, Globe).
Import shared components: Navbar, Footer, Button, Card, CardBody from components/common.

Sections:
1. Hero: Headline "Your Health, Connected" with tagline. Two CTA Buttons: "Get Started" (Link to /register) and "Login" (Link to /login).
2. Features Grid (6 cards): Patients, Doctors, Hospitals, Pharmacy, Security, Mobile-Ready — each with icon, title, and short description.
3. How It Works: 3-step flow (Register → Connect → Manage).
4. Stats banner: Total Users, Hospitals, Prescriptions, cities covered.
5. Footer.

Use CSS classes: page-header, hero-section, features-grid, stats-grid, animate-fade-in.
Make it responsive for mobile.
```

---

### Prompt 2 — Login Page
```
Create a React login page with multi-role selection.
Use useState for formData (email, password), errors, isLoading, selectedRole.
Use React Router (Link to /register, useNavigate for post-login redirect).
Import useAuth for login function.

Features:
1. Role selector with 4 tabs: Patient (blue), Doctor (green), Pharmacy (orange), Admin (red). Each shows different icon.
2. Login form: Email input, Password input (with show/hide toggle), Submit button with loading state.
3. Validation: email required + format check, password required + min 6 chars.
4. On success: redirect to /{role}/dashboard.
5. Error handling: show API error messages.
6. Links: "Forgot Password?" and "Register as patient".

Use shared Input and Button components.
```

---

### Prompt 3 — Patient Registration
```
Create a React patient registration page.
Use useState for multi-step form or single form.
Import useAuth for register function, useNavigate for redirect.

Form Fields:
- Full Name (required)
- Email (required, unique validation)
- Phone (required, 10+ digits)
- Password (required, min 6 chars) + Confirm Password (must match)
- Date of Birth (date picker)
- Gender (dropdown: Male/Female/Other)
- Government ID (optional, for UHID verification)
- Terms checkbox

On success: Show generated Health ID (UHID), auto-login, redirect to /patient/dashboard.
Use shared Input, Button, Card components. Add form validation with error messages.
```

---

### Prompt 4 — Patient Dashboard
```
Create a React patient dashboard with sidebar layout.
Import: Sidebar, Navbar, Card, CardBody, Button from shared components.
Import: useAuth for user info, patientAPI for data fetching.

Layout: Sidebar (left) + Main content (right).
Sidebar links: Dashboard, My Records, Prescriptions, Profile.

Main content sections:
1. Welcome header: "Welcome, {patient.name}!" with current date.
2. Stats grid (4 cards): Upcoming Appointments, Active Prescriptions, Medical Records count, Health Score.
3. Recent activity timeline (last 5 events).
4. Quick action buttons: View Records, View Prescriptions, Edit Profile.

Use useEffect to fetch data on mount. Show LoadingSpinner while loading.
```

---

### Prompt 5 — My Records Page
```
Create a React medical records page for patients.
Use sidebar layout with Sidebar, Navbar components.
Import patientAPI.getRecords() for data fetching.

Features:
1. Filter bar: Date From (date input), Date To (date input), Record Type dropdown (All, Lab, Scan, Discharge, Prescription, Vaccination, Surgery, Other).
2. Records list: Each record card shows — title, date, hospital name, type badge (colored), summary text.
3. Filter onChange triggers API call with params: { dateFrom, dateTo, type }.
4. Empty state: "No medical records found" message when list is empty.
5. Loading state with spinner.

Use Card components for each record. Make responsive.
```

---

### Prompt 6 — My Prescriptions Page
```
Create a React prescriptions page for patients.
Use sidebar layout. Import patientAPI.getPrescriptions().

Features:
1. Status filter tabs: All, Pending (yellow), Dispensed (green), Expired (red).
2. Prescription cards showing: RX code badge, doctor name, diagnosis, status badge, valid_until date.
3. Expandable medicine list: medicine name, dosage, frequency, duration, instructions.
4. Tab click filters by status.
5. Empty state per tab.

Use Card, Button components. Use status colors: pending=warning, dispensed=success, expired=danger.
```

---

### Prompt 7 — Patient Profile Page
```
Create a React patient profile page with view/edit modes.
Use sidebar layout. Import patientAPI.getProfile() and patientAPI.updateProfile().

View Mode:
- Display all patient info in organized sections: Personal (UHID, Name, Email, Phone, DOB, Gender, Age), Address (Address, City, State, Pincode), Medical (Blood Group, Allergies, Chronic Conditions, Emergency Contact).
- "Edit Profile" button.

Edit Mode:
- All fields become editable Input components.
- Save and Cancel buttons.
- On save: call patientAPI.updateProfile(data), show success toast.

UHID and Email are read-only (not editable).
```

---

### Prompt 8 — Doctor Dashboard
```
Create a React doctor dashboard with sidebar layout.
Import doctorAPI.getAppointments() for today's schedule.

Sidebar Links: Dashboard, Search Patient, Appointments, Create Visit, Prescriptions.

Sections:
1. Welcome header with doctor name and specialty.
2. Stats grid (4 cards): Patients Today, Prescriptions Written, Pending Visits, Total Appointments.
3. Today's Appointments table: Patient Name, UHID, Time, Status (scheduled/confirmed/completed), Action button.
4. Quick Actions: Search Patient, Create Visit, Write Prescription (Link buttons).

Status badges: scheduled=info, confirmed=primary, completed=success, cancelled=danger.
```

---

### Prompt 9 — Search Patient Page
```
Create a React search patient page for doctors.
Use sidebar layout. Import doctorAPI.searchPatient().

Features:
1. Search type selector (3 tabs/buttons): Health ID, Phone Number, Government ID.
2. Search input with appropriate placeholder per type.
3. Search button with loading state.
4. Results section: List of matching patients showing — Name, UHID, Age, Gender.
5. Each result is clickable → navigates to /doctor/patient/{id}.
6. No results message: "No patient found with the given criteria".
7. Empty state before first search.

Use Card for results, Input for search, Button for submit.
```

---

### Prompt 10 — Patient Details Page (Doctor View)
```
Create a React patient details page for doctors.
Route: /doctor/patient/:id (use useParams to get id).
Import doctorAPI.getPatientDetails(patientId).

Sections:
1. Patient header: Name, UHID, Age, Gender, Blood Group with colored badges.
2. Medical info cards: Allergies list, Chronic Conditions list.
3. Visit History tab: Table with date, symptoms, diagnosis, vitals summary, doctor name.
4. Prescription History tab: Table with RX code, date, diagnosis, status, medicine count.
5. Action buttons: "Create Visit" (Link to /doctor/visits/create?patient={id}), "Write Prescription" (Link to /doctor/prescriptions/create?patient={id}).

Use tabbed layout for Visit/Prescription history. Show LoadingSpinner while loading.
```

---

### Prompt 11 — Create Visit Page
```
Create a React form page for doctors to record patient visits.
Import doctorAPI.createVisit(). Use useNavigate for redirect.

Form:
- Patient ID/UHID input (pre-filled if coming from patient details via query param)
- Symptoms (textarea, required)
- Diagnosis (textarea, required)
- Notes (textarea, optional)
- Vitals section (collapsible, all optional):
  - Blood Pressure: Systolic + Diastolic (two number inputs side by side)
  - Pulse (number, BPM)
  - Temperature (number, °C)
  - Weight (number, kg)
  - Height (number, cm)
  - Oxygen Saturation (number, SpO2 %)

On submit: validate, call API, show success message with visit ID, option to Write Prescription.
```

---

### Prompt 12 — Write Prescription Page
```
Create a React e-prescription form page for doctors.
Import doctorAPI.createPrescription(). Use useNavigate.

Form:
- Patient ID (pre-filled from query param)
- Visit ID (optional, dropdown or input)
- Diagnosis (textarea, required)
- Notes (textarea, optional)
- Validity period (number input, default 30 days)
- Medicines List (dynamic add/remove):
  - Each medicine row has: Medicine Name, Dosage, Frequency, Duration, Quantity, Instructions
  - "Add Medicine" button (minimum 1 medicine required)
  - Remove button per row

On submit: call API, show success modal with generated RX code.
Style RX code with monospace font and highlighted badge.
```

---

### Prompt 13 — Pharmacy Dashboard
```
Create a React pharmacy dashboard.
Sidebar links: Dashboard, Verify Prescription, Dispensed Today.
Import pharmacyAPI.getHistory().

Sections:
1. Welcome header with pharmacy name and date.
2. Stats cards (3): Verified Today (green), Pending (yellow), Rejected (red).
3. Recent Prescriptions list: RX code badge, patient name, doctor, medicine count, time.
4. "View All" link to history page.

Use Card, Button components. Keep it clean and compact.
```

---

### Prompt 14 — Verify & Dispense Prescription
```
Create a React prescription verification page for pharmacy.
Import pharmacyAPI.verifyPrescription() and pharmacyAPI.markDispensed().

Two-section layout:
Section 1 — Search: RX code input + Verify button.
Section 2 — Result (shown after verify):
- Validity badge: "Valid ✅" (green) or "Invalid ❌" (red) with reason.
- If valid, show: Patient name, UHID, Doctor name, Hospital, Diagnosis, Valid until date.
- Medicines table: Name, Dosage, Frequency, Duration, Quantity.
- "Dispense" button with optional notes textarea.
- If invalid: show reason (Already dispensed / Expired / Cancelled).

On dispense: call markDispensed(), show success toast, clear form.
```

---

### Prompt 15 — Admin Dashboard
```
Create a React admin dashboard.
Sidebar links: Dashboard, Hospitals, Doctors, Pharmacies, System Logs.
Import adminAPI.getStats().

Sections:
1. Stats grid (4 cards with trend): Hospitals, Doctors, Pharmacies, Patients.
2. Quick Actions grid (2x2): Add Hospital, Add Doctor, Add Pharmacy, View Logs.
3. Recent Activity timeline (4-5 items): action, entity name, time.
4. Today's Statistics row: Prescriptions, Visits, New Patients, Uptime.

Use Card, Button, Link components. Show trend indicators (e.g., "+2 this month").
```

---

### Prompt 16 — Manage Hospitals
```
Create a React CRUD page for hospital management (admin).
Import adminAPI.hospitals (list, create, update, delete).

Features:
1. Header with "Add Hospital" button.
2. Filter bar: Status dropdown (All/Active/Inactive/Suspended), Type dropdown (All/Government/Private/Clinic/Nursing Home).
3. Hospitals table (use Table component): Name, Type, Registration No., City, Phone, Status badge.
4. Row actions: Edit (opens modal), Delete (confirmation dialog).
5. Add/Edit Modal form: Name, Type (dropdown), Registration Number, Address, City, State, Pincode, Phone, Email, Status (dropdown).
6. Form validation and error handling.

On save: refresh table. On delete: confirmation Modal before deleting.
```

---

### Prompt 17 — About Page
```
Create a React about page for HealthConnect.
Import Navbar, Footer from shared components.

Sections:
1. Hero: "About HealthConnect" headline with mission statement.
2. Our Mission: Paragraph about digitizing healthcare for India.
3. Key Features: Grid of 4-6 feature highlights with icons.
4. Team section: Team name "ATOM TEAM" with member cards (Aakash Modi - Leader, Rajesh Kumar Mishra - Backend, Vineet Kumar - Database).
5. Technology stack overview.
6. Footer.

Use clean, modern design with gradient accents.
```

---

## 🔗 7. Integration Tasks

### 7.1 Connect AuthContext to Backend API
- [ ] Replace mock `login()` in `AuthContext.jsx` with `authAPI.login()` call
- [ ] Replace mock `register()` with `authAPI.register()` call
- [ ] Store JWT `access` token in `localStorage` as `'token'`
- [ ] Store `refresh` token separately for token refresh
- [ ] Add token refresh logic when access token expires (401 → try refresh → retry request)

### 7.2 Connect Pages to Real API Data
- [ ] Patient Dashboard → Fetch real stats from `patientAPI.getProfile()`
- [ ] My Records → Fetch from `patientAPI.getRecords()` with filters
- [ ] My Prescriptions → Fetch from `patientAPI.getPrescriptions()` with status filter
- [ ] Patient Profile → GET/PUT from `patientAPI`
- [ ] Doctor Dashboard → Fetch from `doctorAPI.getAppointments()`
- [ ] Search Patient → Call `doctorAPI.searchPatient()`
- [ ] Patient Details → Call `doctorAPI.getPatientDetails()`
- [ ] Create Visit → Submit to `doctorAPI.createVisit()`
- [ ] Write Prescription → Submit to `doctorAPI.createPrescription()`
- [ ] Pharmacy Dashboard → Fetch from `pharmacyAPI.getHistory()`
- [ ] Verify Prescription → Call `pharmacyAPI.verifyPrescription()` + `markDispensed()`
- [ ] Admin Dashboard → Fetch from `adminAPI.getStats()`
- [ ] Manage Hospitals → CRUD via `adminAPI.hospitals`

### 7.3 Update API Base URL
- [ ] Change `VITE_API_URL` from `http://localhost:8000/api` to `http://localhost:5000/api` (Flask backend)

### 7.4 Error Handling
- [ ] Add toast notifications for success/error messages
- [ ] Handle network errors gracefully
- [ ] Show user-friendly error messages from API responses

### 7.5 UI/UX Polish
- [ ] Ensure all pages are mobile responsive
- [ ] Add page transitions and micro-animations
- [ ] Implement dark mode toggle consistency across all pages
- [ ] Add proper loading skeletons instead of plain spinners

---

## ✅ Page Checklist Summary

| #  | Page                 | Route                          | Module     | Status |
|----|----------------------|--------------------------------|------------|--------|
| 1  | Landing Page         | `/`                            | public     | ✅ Done |
| 2  | Login Page           | `/login`                       | public     | ✅ Done |
| 3  | Patient Register     | `/register`                    | public     | ✅ Done |
| 4  | About Page           | `/about`                       | public     | ✅ Done |
| 5  | Patient Dashboard    | `/patient/dashboard`           | patient    | ✅ Done |
| 6  | My Records           | `/patient/records`             | patient    | ✅ Done |
| 7  | My Prescriptions     | `/patient/prescriptions`       | patient    | ✅ Done |
| 8  | Patient Profile      | `/patient/profile`             | patient    | ✅ Done |
| 9  | Doctor Dashboard     | `/doctor/dashboard`            | doctor     | ✅ Done |
| 10 | Search Patient       | `/doctor/search`               | doctor     | ✅ Done |
| 11 | Patient Details      | `/doctor/patient/:id`          | doctor     | ✅ Done |
| 12 | Create Visit         | `/doctor/visits/create`        | doctor     | ✅ Done |
| 13 | Write Prescription   | `/doctor/prescriptions/create` | doctor     | ✅ Done |
| 14 | Pharmacy Dashboard   | `/pharmacy/dashboard`          | pharmacy   | ✅ Done |
| 15 | Verify Prescription  | `/pharmacy/verify`             | pharmacy   | ✅ Done |
| 16 | Admin Dashboard      | `/admin/dashboard`             | admin      | ✅ Done |
| 17 | Manage Hospitals     | `/admin/hospitals`             | admin      | ✅ Done |

---

> **Note:** The frontend runs on Vite dev server at `http://localhost:5173`. The API service in `api.js` handles JWT token injection, 401 auto-redirect, and 10s request timeout. Currently using mock auth in `AuthContext.jsx` — needs to be replaced with real API calls for production.
