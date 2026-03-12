# HealthConnect Patient Portal Database Schema (MySQL)

This document defines the MySQL 8.0+ schema for the Patient Portal.

## Database Core Prompt
> Design a MySQL database for a medical patient portal. It must handle secure patient records, authentication, doctors, facilities, prescriptions (with JSON medication lists), and visit histories. Ensure relational integrity with foreign keys and use InnoDB engine for reliability.

## SQL Schema Definition

```sql
-- Users Table (Patients)
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    dob DATE,
    gender VARCHAR(20),
    blood_group VARCHAR(5),
    govt_id VARCHAR(50), -- Masked or encrypted
    health_id VARCHAR(20) UNIQUE NOT NULL, -- UHID (HID-XXXX-XXXX)
    address TEXT,
    city VARCHAR(100),
    state VARCHAR(100),
    pincode VARCHAR(10),
    emergency_contact TEXT,
    allergies JSON,
    chronic_conditions JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Doctors Table (Common across portals)
CREATE TABLE doctors (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    specialization VARCHAR(255),
    email VARCHAR(255) UNIQUE
);

-- Medical Records
CREATE TABLE medical_records (
    id INT AUTO_INCREMENT PRIMARY KEY,
    patient_id INTEGER,
    date DATE NOT NULL,
    doctor_name VARCHAR(255),
    facility VARCHAR(255),
    type VARCHAR(50), -- LAB, SCAN, DISCHARGE, VACCINE
    title VARCHAR(255) NOT NULL,
    diagnosis TEXT,
    file_url TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Prescriptions
CREATE TABLE prescriptions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    patient_id INTEGER,
    rx_code VARCHAR(20) UNIQUE NOT NULL,
    doctor_name VARCHAR(255),
    facility VARCHAR(255),
    issue_date DATE NOT NULL,
    expiry_date DATE,
    diagnosis TEXT,
    status VARCHAR(20) DEFAULT 'pending', -- pending, dispensed, expired
    medicines JSON NOT NULL, -- Array of objects {name, dosage}
    pharmacy_note TEXT,
    transaction_id VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Visits / Appointments
CREATE TABLE visits (
    id INT AUTO_INCREMENT PRIMARY KEY,
    patient_id INTEGER,
    doctor_name VARCHAR(255),
    facility VARCHAR(255),
    date DATE NOT NULL,
    time TIME NOT NULL,
    diagnosis TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES users(id) ON DELETE CASCADE
);
```

## Schema Logic
- **UHID**: The `health_id` (UHID) is the primary identifier for patients across the ecosystem (Patient, Doctor, and Pharmacy portals).
- **JSONB**: Used for `allergies`, `chronic_conditions`, and `medicines` to allow flexible, semi-structured data within established medical domains.
- **Relational Integrity**: Foreign keys link records, prescriptions, and visits directly to the `users` table via `patient_id`.

## Shared Database Architecture (ASCII)

```text
  +-------------------+       +-------------------+       +-------------------+
  |  PATIENT PORTAL   |       |   DOCTOR PORTAL   |       |  PHARMACY PORTAL  |
  | (Flask Backend)   |       | (Flask Backend)   |       | (Flask Backend)   |
  +---------+---------+       +---------+---------+       +---------+---------+
            |                           |                           |
            |           SHARED          |           UHID-BASED      |
            +---------------------------+---------------------------+
                                        |
                          +-------------v-------------+
                          |      MySQL DATABASE       |
                          |   (HealthConnect_Live)    |
                          +-------------+-------------+
                                        |
                          +-------------v-------------+
                          | TABLES:                   |
                          | - users (Patients)        |
                          | - doctors                 |
                          | - medical_records         |
                          | - prescriptions           |
                          | - visits                  |
                          +---------------------------+
```
