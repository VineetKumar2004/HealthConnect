# 📋 Vineet Kumar — Database & Documentation Task Sheet

**Project:** HealthConnect
**Role:** Database Architecture & Project Documentation  
**Database:** MySQL  
**ORM (if used):** SQLAlchemy (Flask) / Raw SQL

---

## 📌 Table of Contents

1. [Database Overview & ER Diagram](#-1-database-overview--er-diagram)
2. [Table Schemas (MySQL)](#-2-table-schemas-mysql)
3. [Relationships & Foreign Keys](#-3-relationships--foreign-keys)
4. [Indexes & Constraints](#-4-indexes--constraints)
5. [Documentation Tasks](#-5-documentation-tasks)
6. [AI Prompts for Database Creation](#-6-ai-prompts-for-database-creation)

---

## 🗂 1. Database Overview & ER Diagram

### Tables Summary

| #  | Table Name          | Description                              | Primary Key |
|----|---------------------|------------------------------------------|-------------|
| 1  | `users`             | All user accounts (patient/doctor/pharmacy/admin) | `id`   |
| 2  | `patients`          | Patient profiles with UHID               | `id`        |
| 3  | `doctors`           | Doctor profiles with specialties          | `id`        |
| 4  | `hospitals`         | Healthcare facilities                    | `id`        |
| 5  | `visits`            | Patient consultation/visit records        | `id`        |
| 6  | `appointments`      | Scheduled doctor-patient appointments     | `id`        |
| 7  | `prescriptions`     | E-prescriptions issued by doctors         | `id`        |
| 8  | `prescription_items`| Individual medicines in a prescription    | `id`        |
| 9  | `pharmacies`        | Pharmacy profiles                        | `id`        |
| 10 | `dispense_logs`     | Prescription dispensing history           | `id`        |
| 11 | `medical_records`   | Lab reports, scans, discharge summaries   | `id`        |
| 12 | `audit_logs`        | Security & activity tracking              | `id`        |

### ER Diagram (Mermaid)

```
erDiagram
    users ||--o| patients : "has"
    users ||--o| doctors : "has"
    users ||--o| pharmacies : "has"
    users ||--o{ audit_logs : "generates"

    patients ||--o{ visits : "has"
    patients ||--o{ appointments : "has"
    patients ||--o{ prescriptions : "receives"
    patients ||--o{ medical_records : "has"

    doctors ||--o{ visits : "conducts"
    doctors ||--o{ appointments : "schedules"
    doctors ||--o{ prescriptions : "writes"
    doctors }o--|| hospitals : "works_at"

    prescriptions ||--o{ prescription_items : "contains"
    prescriptions ||--o{ dispense_logs : "tracked_by"
    prescriptions o|--o| visits : "linked_to"
    prescriptions }o--o| pharmacies : "dispensed_by"

    pharmacies ||--o{ dispense_logs : "records"

    hospitals ||--o{ medical_records : "issued_by"
```

---

## 🏗 2. Table Schemas (MySQL)

### 2.1 `users` — User Accounts

```sql
CREATE TABLE users (
    id              BIGINT AUTO_INCREMENT PRIMARY KEY,
    email           VARCHAR(255) NOT NULL UNIQUE,
    phone           VARCHAR(15) DEFAULT '',
    password        VARCHAR(255) NOT NULL,
    role            ENUM('patient', 'doctor', 'pharmacy', 'admin') NOT NULL DEFAULT 'patient',
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    is_staff        BOOLEAN NOT NULL DEFAULT FALSE,
    is_verified     BOOLEAN NOT NULL DEFAULT FALSE,
    is_superuser    BOOLEAN NOT NULL DEFAULT FALSE,
    created_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    last_login      DATETIME NULL,

    INDEX idx_users_email (email),
    INDEX idx_users_role (role)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Notes:**
- `password` stores hashed password (werkzeug or bcrypt)
- `role` determines access level across the platform
- `email` is the login username (unique)

---

### 2.2 `patients` — Patient Profiles

```sql
CREATE TABLE patients (
    id                  BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id             BIGINT NOT NULL UNIQUE,
    uhid                VARCHAR(20) NOT NULL UNIQUE,
    govt_id_hash        VARCHAR(128) NULL,
    name                VARCHAR(255) NOT NULL,
    dob                 DATE NULL,
    gender              ENUM('M', 'F', 'O') NULL,
    address             TEXT DEFAULT '',
    city                VARCHAR(100) DEFAULT '',
    state               VARCHAR(100) DEFAULT '',
    pincode             VARCHAR(10) DEFAULT '',
    emergency_contact   VARCHAR(15) DEFAULT '',
    blood_group         ENUM('A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-') NULL,
    allergies           TEXT DEFAULT '',
    chronic_conditions  TEXT DEFAULT '',
    created_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_patients_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE INDEX idx_patients_uhid (uhid),
    INDEX idx_patients_govt_id_hash (govt_id_hash)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Notes:**
- `uhid` = Unique Health ID, auto-generated in format `HID-XXXX-XXXX`
- `govt_id_hash` = SHA-256 hashed government ID (one-way, for lookup only)
- `allergies` and `chronic_conditions` are comma-separated text fields

---

### 2.3 `doctors` — Doctor Profiles

```sql
CREATE TABLE doctors (
    id                  BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id             BIGINT NOT NULL UNIQUE,
    hospital_id         BIGINT NULL,
    name                VARCHAR(255) NOT NULL,
    specialty           ENUM(
                            'general', 'cardiology', 'dermatology', 'endocrinology',
                            'gastroenterology', 'neurology', 'oncology', 'orthopedics',
                            'pediatrics', 'psychiatry', 'pulmonology', 'radiology',
                            'surgery', 'urology', 'other'
                        ) NOT NULL DEFAULT 'general',
    license_number      VARCHAR(50) NOT NULL UNIQUE,
    qualification       VARCHAR(255) DEFAULT '',
    experience_years    INT UNSIGNED NOT NULL DEFAULT 0,
    available_days      VARCHAR(100) DEFAULT 'Mon,Tue,Wed,Thu,Fri',
    consultation_hours  VARCHAR(50) DEFAULT '09:00-17:00',
    created_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_doctors_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT fk_doctors_hospital FOREIGN KEY (hospital_id) REFERENCES hospitals(id) ON DELETE SET NULL,
    INDEX idx_doctors_specialty (specialty),
    UNIQUE INDEX idx_doctors_license (license_number)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

### 2.4 `hospitals` — Healthcare Facilities

```sql
CREATE TABLE hospitals (
    id                      BIGINT AUTO_INCREMENT PRIMARY KEY,
    name                    VARCHAR(255) NOT NULL,
    type                    ENUM('government', 'private', 'clinic', 'nursing_home') NOT NULL DEFAULT 'private',
    registration_number     VARCHAR(100) NOT NULL UNIQUE,
    address                 TEXT NOT NULL,
    city                    VARCHAR(100) NOT NULL,
    state                   VARCHAR(100) NOT NULL,
    pincode                 VARCHAR(10) NOT NULL,
    phone                   VARCHAR(15) NOT NULL,
    email                   VARCHAR(255) NOT NULL,
    status                  ENUM('active', 'inactive', 'suspended') NOT NULL DEFAULT 'active',
    created_at              DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at              DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    UNIQUE INDEX idx_hospitals_reg (registration_number),
    INDEX idx_hospitals_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

### 2.5 `visits` — Patient Consultation Records

```sql
CREATE TABLE visits (
    id                          BIGINT AUTO_INCREMENT PRIMARY KEY,
    patient_id                  BIGINT NOT NULL,
    doctor_id                   BIGINT NOT NULL,
    date                        DATE NOT NULL,
    symptoms                    TEXT NOT NULL,
    diagnosis                   TEXT NOT NULL,
    notes                       TEXT DEFAULT '',
    blood_pressure_systolic     INT UNSIGNED NULL,
    blood_pressure_diastolic    INT UNSIGNED NULL,
    pulse                       INT UNSIGNED NULL COMMENT 'BPM',
    temperature                 DECIMAL(4,1) NULL COMMENT 'Celsius',
    weight                      DECIMAL(5,2) NULL COMMENT 'kg',
    height                      DECIMAL(5,2) NULL COMMENT 'cm',
    oxygen_saturation           INT UNSIGNED NULL COMMENT 'SpO2 %',
    created_at                  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at                  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_visits_patient FOREIGN KEY (patient_id) REFERENCES patients(id) ON DELETE CASCADE,
    CONSTRAINT fk_visits_doctor FOREIGN KEY (doctor_id) REFERENCES doctors(id) ON DELETE CASCADE,
    INDEX idx_visits_patient (patient_id),
    INDEX idx_visits_doctor (doctor_id),
    INDEX idx_visits_date (date)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

### 2.6 `appointments` — Scheduled Appointments

```sql
CREATE TABLE appointments (
    id              BIGINT AUTO_INCREMENT PRIMARY KEY,
    patient_id      BIGINT NOT NULL,
    doctor_id       BIGINT NOT NULL,
    date            DATE NOT NULL,
    time            TIME NOT NULL,
    status          ENUM('scheduled', 'confirmed', 'completed', 'cancelled', 'no_show') NOT NULL DEFAULT 'scheduled',
    notes           TEXT DEFAULT '',
    created_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_appointments_patient FOREIGN KEY (patient_id) REFERENCES patients(id) ON DELETE CASCADE,
    CONSTRAINT fk_appointments_doctor FOREIGN KEY (doctor_id) REFERENCES doctors(id) ON DELETE CASCADE,
    INDEX idx_appointments_date (date, time),
    INDEX idx_appointments_doctor_date (doctor_id, date)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

### 2.7 `prescriptions` — E-Prescriptions

```sql
CREATE TABLE prescriptions (
    id                  BIGINT AUTO_INCREMENT PRIMARY KEY,
    patient_id          BIGINT NOT NULL,
    doctor_id           BIGINT NOT NULL,
    visit_id            BIGINT NULL,
    rx_code             VARCHAR(20) NOT NULL UNIQUE,
    diagnosis           TEXT NOT NULL,
    notes               TEXT DEFAULT '',
    status              ENUM('pending', 'dispensed', 'partially_dispensed', 'expired', 'cancelled') NOT NULL DEFAULT 'pending',
    dispensed_at        DATETIME NULL,
    dispensed_by        BIGINT NULL,
    dispense_notes      TEXT DEFAULT '',
    valid_until         DATE NULL,
    created_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_prescriptions_patient FOREIGN KEY (patient_id) REFERENCES patients(id) ON DELETE CASCADE,
    CONSTRAINT fk_prescriptions_doctor FOREIGN KEY (doctor_id) REFERENCES doctors(id) ON DELETE CASCADE,
    CONSTRAINT fk_prescriptions_visit FOREIGN KEY (visit_id) REFERENCES visits(id) ON DELETE SET NULL,
    CONSTRAINT fk_prescriptions_pharmacy FOREIGN KEY (dispensed_by) REFERENCES pharmacies(id) ON DELETE SET NULL,
    UNIQUE INDEX idx_prescriptions_rx_code (rx_code),
    INDEX idx_prescriptions_patient (patient_id),
    INDEX idx_prescriptions_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**Notes:**
- `rx_code` = Auto-generated unique prescription code, format `RX-XXXXXXXX`
- `valid_until` = Expiry date (default 30 days from creation)
- `dispensed_by` references a pharmacy that dispensed the prescription

---

### 2.8 `prescription_items` — Medicines in a Prescription

```sql
CREATE TABLE prescription_items (
    id                  BIGINT AUTO_INCREMENT PRIMARY KEY,
    prescription_id     BIGINT NOT NULL,
    medicine_name       VARCHAR(255) NOT NULL,
    dosage              VARCHAR(100) NOT NULL COMMENT 'e.g., 500mg',
    frequency           VARCHAR(100) NOT NULL COMMENT 'e.g., 2x day',
    duration            VARCHAR(100) NOT NULL COMMENT 'e.g., 30 days',
    quantity            INT UNSIGNED NOT NULL DEFAULT 0,
    instructions        VARCHAR(255) DEFAULT '' COMMENT 'e.g., After meals',

    CONSTRAINT fk_items_prescription FOREIGN KEY (prescription_id) REFERENCES prescriptions(id) ON DELETE CASCADE,
    INDEX idx_items_prescription (prescription_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

### 2.9 `pharmacies` — Pharmacy Profiles

```sql
CREATE TABLE pharmacies (
    id                  BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id             BIGINT NOT NULL UNIQUE,
    name                VARCHAR(255) NOT NULL,
    license_number      VARCHAR(50) NOT NULL UNIQUE,
    address             TEXT NOT NULL,
    city                VARCHAR(100) NOT NULL,
    state               VARCHAR(100) NOT NULL,
    pincode             VARCHAR(10) NOT NULL,
    phone               VARCHAR(15) NOT NULL,
    status              ENUM('active', 'inactive', 'suspended') NOT NULL DEFAULT 'active',
    opening_time        TIME NOT NULL DEFAULT '09:00:00',
    closing_time        TIME NOT NULL DEFAULT '21:00:00',
    created_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_pharmacies_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE INDEX idx_pharmacies_license (license_number)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

### 2.10 `dispense_logs` — Dispensing History

```sql
CREATE TABLE dispense_logs (
    id                  BIGINT AUTO_INCREMENT PRIMARY KEY,
    pharmacy_id         BIGINT NOT NULL,
    prescription_id     BIGINT NOT NULL,
    dispensed_by        BIGINT NULL,
    notes               TEXT DEFAULT '',
    dispensed_at        DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_dlogs_pharmacy FOREIGN KEY (pharmacy_id) REFERENCES pharmacies(id) ON DELETE CASCADE,
    CONSTRAINT fk_dlogs_prescription FOREIGN KEY (prescription_id) REFERENCES prescriptions(id) ON DELETE CASCADE,
    CONSTRAINT fk_dlogs_user FOREIGN KEY (dispensed_by) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_dlogs_pharmacy (pharmacy_id),
    INDEX idx_dlogs_dispensed_at (dispensed_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

### 2.11 `medical_records` — Medical Records

```sql
CREATE TABLE medical_records (
    id                  BIGINT AUTO_INCREMENT PRIMARY KEY,
    patient_id          BIGINT NOT NULL,
    hospital_id         BIGINT NULL,
    type                ENUM('lab', 'scan', 'discharge', 'prescription', 'vaccination', 'surgery', 'other') NOT NULL,
    title               VARCHAR(255) NOT NULL,
    summary             TEXT DEFAULT '',
    date                DATE NOT NULL,
    file                VARCHAR(500) NULL COMMENT 'File path: records/YYYY/MM/filename',
    file_name           VARCHAR(255) DEFAULT '',
    uploaded_by         BIGINT NULL,
    is_shared           BOOLEAN NOT NULL DEFAULT TRUE COMMENT 'Visible to doctors',
    created_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_records_patient FOREIGN KEY (patient_id) REFERENCES patients(id) ON DELETE CASCADE,
    CONSTRAINT fk_records_hospital FOREIGN KEY (hospital_id) REFERENCES hospitals(id) ON DELETE SET NULL,
    CONSTRAINT fk_records_uploader FOREIGN KEY (uploaded_by) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_records_patient (patient_id),
    INDEX idx_records_date (date),
    INDEX idx_records_type (type)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

### 2.12 `audit_logs` — System Audit Trail

```sql
CREATE TABLE audit_logs (
    id              BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id         BIGINT NULL,
    action          ENUM(
                        'login', 'logout', 'register', 'view_record',
                        'create_record', 'create_prescription', 'dispense',
                        'search_patient', 'update_profile', 'admin_action'
                    ) NOT NULL,
    ip_address      VARCHAR(45) NULL,
    user_agent      VARCHAR(500) DEFAULT '',
    details         JSON DEFAULT NULL,
    timestamp       DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_audit_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_audit_user (user_id),
    INDEX idx_audit_action (action),
    INDEX idx_audit_timestamp (timestamp)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

## 🔗 3. Relationships & Foreign Keys

| Parent Table     | Child Table          | FK Column         | On Delete    | Relationship |
|------------------|----------------------|-------------------|--------------|--------------|
| `users`          | `patients`           | `user_id`         | CASCADE      | One-to-One   |
| `users`          | `doctors`            | `user_id`         | CASCADE      | One-to-One   |
| `users`          | `pharmacies`         | `user_id`         | CASCADE      | One-to-One   |
| `users`          | `audit_logs`         | `user_id`         | SET NULL     | One-to-Many  |
| `users`          | `medical_records`    | `uploaded_by`     | SET NULL     | One-to-Many  |
| `users`          | `dispense_logs`      | `dispensed_by`    | SET NULL     | One-to-Many  |
| `hospitals`      | `doctors`            | `hospital_id`     | SET NULL     | One-to-Many  |
| `hospitals`      | `medical_records`    | `hospital_id`     | SET NULL     | One-to-Many  |
| `patients`       | `visits`             | `patient_id`      | CASCADE      | One-to-Many  |
| `patients`       | `appointments`       | `patient_id`      | CASCADE      | One-to-Many  |
| `patients`       | `prescriptions`      | `patient_id`      | CASCADE      | One-to-Many  |
| `patients`       | `medical_records`    | `patient_id`      | CASCADE      | One-to-Many  |
| `doctors`        | `visits`             | `doctor_id`       | CASCADE      | One-to-Many  |
| `doctors`        | `appointments`       | `doctor_id`       | CASCADE      | One-to-Many  |
| `doctors`        | `prescriptions`      | `doctor_id`       | CASCADE      | One-to-Many  |
| `visits`         | `prescriptions`      | `visit_id`        | SET NULL     | One-to-Many  |
| `prescriptions`  | `prescription_items` | `prescription_id` | CASCADE      | One-to-Many  |
| `prescriptions`  | `dispense_logs`      | `prescription_id` | CASCADE      | One-to-Many  |
| `pharmacies`     | `prescriptions`      | `dispensed_by`    | SET NULL     | Many-to-One  |
| `pharmacies`     | `dispense_logs`      | `pharmacy_id`     | CASCADE      | One-to-Many  |

---

## 📐 4. Indexes & Constraints

### Unique Constraints
| Table               | Column(s)             | Purpose                          |
|---------------------|-----------------------|----------------------------------|
| `users`             | `email`               | One account per email            |
| `patients`          | `user_id`             | One patient profile per user     |
| `patients`          | `uhid`                | Unique Health ID                 |
| `doctors`           | `user_id`             | One doctor profile per user      |
| `doctors`           | `license_number`      | Unique medical license           |
| `pharmacies`        | `user_id`             | One pharmacy profile per user    |
| `pharmacies`        | `license_number`      | Unique pharmacy license          |
| `hospitals`         | `registration_number` | Unique registration              |
| `prescriptions`     | `rx_code`             | Unique prescription code         |

### Performance Indexes
| Table               | Column(s)              | Purpose                       |
|---------------------|------------------------|-------------------------------|
| `patients`          | `govt_id_hash`         | Fast govt ID lookup           |
| `visits`            | `patient_id`, `date`   | Patient visit history         |
| `appointments`      | `doctor_id`, `date`    | Doctor's daily schedule       |
| `prescriptions`     | `rx_code`              | Quick prescription lookup     |
| `prescriptions`     | `status`               | Filter by status              |
| `medical_records`   | `patient_id`, `date`   | Patient records by date       |
| `audit_logs`        | `timestamp`, `action`  | Log filtering and search      |

---

## 📖 5. Documentation Tasks

### 5.1 Database Documentation
- [ ] Create ER diagram (visual) using a tool like dbdiagram.io or MySQL Workbench
- [ ] Document all table schemas with column descriptions
- [ ] Document all foreign key relationships and cascade rules
- [ ] List all ENUM values and their meanings
- [ ] Document indexes and their performance impact

### 5.2 MySQL Setup Guide
- [ ] MySQL installation instructions (local & server)
- [ ] Database creation commands
- [ ] User and permissions setup
- [ ] Connection configuration for Flask (`.env` variables)
- [ ] Migration strategy documentation

### 5.3 General Project Documentation
- [ ] Project README with architecture overview
- [ ] Deployment guide (local + production)
- [ ] Data flow diagrams for key features
- [ ] Security documentation (hashing, JWT, role-based access)

---

## 🤖 6. AI Prompts for Database Creation

Below are copy-paste-ready **AI prompts** to create each table:

---

### Prompt 1 — Create MySQL Database & Users Table
```
Create a MySQL database called 'medtech_db' with utf8mb4 charset.
Then create a 'users' table with:
- id (BIGINT, auto increment, primary key)
- email (VARCHAR 255, unique, not null) — used as login username
- phone (VARCHAR 15, default empty)
- password (VARCHAR 255, not null) — stores hashed password
- role (ENUM: patient, doctor, pharmacy, admin; default 'patient')
- is_active (BOOLEAN, default true)
- is_staff (BOOLEAN, default false)
- is_verified (BOOLEAN, default false)
- is_superuser (BOOLEAN, default false)
- created_at (DATETIME, auto-set on creation)
- updated_at (DATETIME, auto-update on modification)
- last_login (DATETIME, nullable)
Add indexes on email and role. Use InnoDB engine.
```

---

### Prompt 2 — Patients Table
```
Create a MySQL 'patients' table for storing patient profiles:
- id (BIGINT, auto increment, primary key)
- user_id (BIGINT, unique, not null, FK to users.id ON DELETE CASCADE)
- uhid (VARCHAR 20, unique, not null) — Unique Health ID format HID-XXXX-XXXX
- govt_id_hash (VARCHAR 128, nullable, indexed) — SHA-256 hashed government ID
- name (VARCHAR 255, not null)
- dob (DATE, nullable)
- gender (ENUM: M, F, O; nullable)
- address, city, state, pincode (text/varchar fields)
- emergency_contact (VARCHAR 15)
- blood_group (ENUM: A+, A-, B+, B-, AB+, AB-, O+, O-; nullable)
- allergies (TEXT) — comma-separated list
- chronic_conditions (TEXT) — comma-separated list
- created_at, updated_at (auto timestamps)
Add unique index on uhid and index on govt_id_hash for fast lookup.
```

---

### Prompt 3 — Hospitals Table
```
Create a MySQL 'hospitals' table:
- id (BIGINT, auto increment, primary key)
- name (VARCHAR 255, not null)
- type (ENUM: government, private, clinic, nursing_home; default 'private')
- registration_number (VARCHAR 100, unique, not null)
- address (TEXT, not null)
- city, state, pincode (VARCHAR)
- phone (VARCHAR 15, not null)
- email (VARCHAR 255, not null)
- status (ENUM: active, inactive, suspended; default 'active')
- created_at, updated_at (auto timestamps)
Note: This table must be created BEFORE the doctors table since doctors has a FK to hospitals.
```

---

### Prompt 4 — Doctors Table
```
Create a MySQL 'doctors' table:
- id (BIGINT, auto increment, primary key)
- user_id (BIGINT, unique, FK to users.id ON DELETE CASCADE)
- hospital_id (BIGINT, nullable, FK to hospitals.id ON DELETE SET NULL)
- name (VARCHAR 255, not null)
- specialty (ENUM with 15 choices: general, cardiology, dermatology, endocrinology, gastroenterology, neurology, oncology, orthopedics, pediatrics, psychiatry, pulmonology, radiology, surgery, urology, other; default 'general')
- license_number (VARCHAR 50, unique, not null)
- qualification (VARCHAR 255)
- experience_years (INT UNSIGNED, default 0)
- available_days (VARCHAR 100, default 'Mon,Tue,Wed,Thu,Fri')
- consultation_hours (VARCHAR 50, default '09:00-17:00')
- created_at, updated_at (auto timestamps)
Add index on specialty.
```

---

### Prompt 5 — Visits Table
```
Create a MySQL 'visits' table for patient consultations:
- id (BIGINT, auto increment, primary key)
- patient_id (BIGINT, FK to patients.id ON DELETE CASCADE)
- doctor_id (BIGINT, FK to doctors.id ON DELETE CASCADE)
- date (DATE, not null)
- symptoms (TEXT, not null)
- diagnosis (TEXT, not null)
- notes (TEXT, default empty)
- Vitals columns (all nullable): blood_pressure_systolic (INT), blood_pressure_diastolic (INT), pulse (INT, comment 'BPM'), temperature (DECIMAL 4,1, comment 'Celsius'), weight (DECIMAL 5,2, comment 'kg'), height (DECIMAL 5,2, comment 'cm'), oxygen_saturation (INT, comment 'SpO2 %')
- created_at, updated_at (auto timestamps)
Add indexes on patient_id, doctor_id, and date.
```

---

### Prompt 6 — Appointments Table
```
Create a MySQL 'appointments' table:
- id (BIGINT, auto increment, primary key)
- patient_id (BIGINT, FK to patients.id ON DELETE CASCADE)
- doctor_id (BIGINT, FK to doctors.id ON DELETE CASCADE)
- date (DATE, not null)
- time (TIME, not null)
- status (ENUM: scheduled, confirmed, completed, cancelled, no_show; default 'scheduled')
- notes (TEXT, default empty)
- created_at, updated_at (auto timestamps)
Add composite index on (doctor_id, date) for querying a doctor's daily schedule.
```

---

### Prompt 7 — Pharmacies Table
```
Create a MySQL 'pharmacies' table:
- id (BIGINT, auto increment, primary key)
- user_id (BIGINT, unique, FK to users.id ON DELETE CASCADE)
- name (VARCHAR 255, not null)
- license_number (VARCHAR 50, unique, not null)
- address (TEXT, not null)
- city, state, pincode (VARCHAR)
- phone (VARCHAR 15, not null)
- status (ENUM: active, inactive, suspended; default 'active')
- opening_time (TIME, default '09:00:00')
- closing_time (TIME, default '21:00:00')
- created_at, updated_at (auto timestamps)
Note: This table must be created BEFORE prescriptions table.
```

---

### Prompt 8 — Prescriptions & Prescription Items Tables
```
Create two MySQL tables for prescriptions:

Table 1 — 'prescriptions':
- id (BIGINT, primary key)
- patient_id (BIGINT, FK to patients.id ON DELETE CASCADE)
- doctor_id (BIGINT, FK to doctors.id ON DELETE CASCADE)
- visit_id (BIGINT, nullable, FK to visits.id ON DELETE SET NULL)
- rx_code (VARCHAR 20, unique, not null) — format RX-XXXXXXXX
- diagnosis (TEXT, not null)
- notes (TEXT)
- status (ENUM: pending, dispensed, partially_dispensed, expired, cancelled; default 'pending')
- dispensed_at (DATETIME, nullable)
- dispensed_by (BIGINT, nullable, FK to pharmacies.id ON DELETE SET NULL)
- dispense_notes (TEXT)
- valid_until (DATE, nullable) — typically 30 days from creation
- created_at, updated_at

Table 2 — 'prescription_items':
- id (BIGINT, primary key)
- prescription_id (BIGINT, FK to prescriptions.id ON DELETE CASCADE)
- medicine_name (VARCHAR 255, not null)
- dosage (VARCHAR 100, not null) — e.g., '500mg'
- frequency (VARCHAR 100, not null) — e.g., '2x day'
- duration (VARCHAR 100, not null) — e.g., '30 days'
- quantity (INT UNSIGNED, default 0)
- instructions (VARCHAR 255) — e.g., 'After meals'
```

---

### Prompt 9 — Dispense Logs Table
```
Create a MySQL 'dispense_logs' table to track prescription dispensing:
- id (BIGINT, primary key)
- pharmacy_id (BIGINT, FK to pharmacies.id ON DELETE CASCADE)
- prescription_id (BIGINT, FK to prescriptions.id ON DELETE CASCADE)
- dispensed_by (BIGINT, nullable, FK to users.id ON DELETE SET NULL) — the actual pharmacist user
- notes (TEXT)
- dispensed_at (DATETIME, default CURRENT_TIMESTAMP)
Add indexes on pharmacy_id and dispensed_at for history queries.
```

---

### Prompt 10 — Medical Records Table
```
Create a MySQL 'medical_records' table:
- id (BIGINT, primary key)
- patient_id (BIGINT, FK to patients.id ON DELETE CASCADE)
- hospital_id (BIGINT, nullable, FK to hospitals.id ON DELETE SET NULL)
- type (ENUM: lab, scan, discharge, prescription, vaccination, surgery, other; not null)
- title (VARCHAR 255, not null)
- summary (TEXT)
- date (DATE, not null)
- file (VARCHAR 500, nullable) — file path for uploads
- file_name (VARCHAR 255)
- uploaded_by (BIGINT, nullable, FK to users.id ON DELETE SET NULL)
- is_shared (BOOLEAN, default true) — controls doctor visibility
- created_at, updated_at
Add indexes on patient_id, date, and type.
```

---

### Prompt 11 — Audit Logs Table
```
Create a MySQL 'audit_logs' table for security tracking:
- id (BIGINT, primary key)
- user_id (BIGINT, nullable, FK to users.id ON DELETE SET NULL)
- action (ENUM: login, logout, register, view_record, create_record, create_prescription, dispense, search_patient, update_profile, admin_action; not null)
- ip_address (VARCHAR 45, nullable) — supports IPv6
- user_agent (VARCHAR 500)
- details (JSON, nullable) — stores additional context as JSON
- timestamp (DATETIME, default CURRENT_TIMESTAMP)
Add indexes on user_id, action, and timestamp for efficient log queries.
```

---

### Prompt 12 — Full Database Setup Script
```
Write a complete MySQL setup script for the MedTech application that:
1. Creates database 'medtech_db' with utf8mb4 charset.
2. Creates a MySQL user 'medtech_user' with a strong password and grants all privileges on medtech_db.
3. Creates all 12 tables in the correct order (respecting foreign key dependencies):
   Order: users → patients → hospitals → doctors → pharmacies → visits → appointments → prescriptions → prescription_items → dispense_logs → medical_records → audit_logs
4. Adds all foreign keys, indexes, and unique constraints.
5. Inserts a default admin user (email: admin@medtech.com, role: admin).
6. Inserts a sample hospital record for testing.
Use InnoDB engine and utf8mb4 charset for all tables.
```

---

## ✅ Table Creation Checklist

| #  | Table                | FK Dependencies              | Status |
|----|----------------------|------------------------------|--------|
| 1  | `users`              | None                         | [ ]    |
| 2  | `hospitals`          | None                         | [ ]    |
| 3  | `patients`           | `users`                      | [ ]    |
| 4  | `doctors`            | `users`, `hospitals`         | [ ]    |
| 5  | `pharmacies`         | `users`                      | [ ]    |
| 6  | `visits`             | `patients`, `doctors`        | [ ]    |
| 7  | `appointments`       | `patients`, `doctors`        | [ ]    |
| 8  | `prescriptions`      | `patients`, `doctors`, `visits`, `pharmacies` | [ ] |
| 9  | `prescription_items` | `prescriptions`              | [ ]    |
| 10 | `dispense_logs`      | `pharmacies`, `prescriptions`, `users` | [ ] |
| 11 | `medical_records`    | `patients`, `hospitals`, `users` | [ ] |
| 12 | `audit_logs`         | `users`                      | [ ]    |

> **Important:** Tables must be created in the order listed above to satisfy foreign key dependencies.
