# 🛡 CFEMS — CyberForensic Evidence Management System

> A full-stack digital forensic evidence management platform built for the investigation of ransomware incidents. Designed to professional DFIR standards.

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Flask](https://img.shields.io/badge/Flask-2.3-000000?style=flat-square&logo=flask&logoColor=white)](https://flask.palletsprojects.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![Standards](https://img.shields.io/badge/ISO%2FIEC-27037%3A2012-blue?style=flat-square)](https://www.iso.org/standard/44381.html)

---

## 📋 Overview

CFEMS is a production-grade forensic evidence management system built as part of a university mid-semester examination in **Database and Programming Concepts** (Data Forensics and Cyber Security program). It demonstrates full-stack development, database design, and cybersecurity principles applied to a real-world digital forensics scenario.

**Case Context:** LockBit 3.0 ransomware attack on fictional firm *FinSecure Pty Ltd* — 47 endpoints compromised, 3 file servers encrypted, investigated by *CipherForensics Pty Ltd*.

---

## ✨ Features

### 🔐 Authentication & Access Control
- **bcrypt password hashing** (cost factor 12) — no plaintext passwords ever stored
- **SHA-256 session tokens** — issued on login, expire after 8 hours
- **Account lockout** — 5 failed attempts triggers a 30-minute lockout
- **Role-Based Access Control (RBAC)** — 4 roles enforced at the PostgreSQL database level
- **Full audit logging** — every login attempt logged with analyst, IP, and timestamp

### 🔍 Evidence Management
- Register new evidence items with automatic **MD5 + SHA-256 hash computation**
- Full **chain-of-custody tracking** — append-only, tamper-evident
- Evidence status workflow: `Unanalysed → Under Review → Analysed → Submitted`
- Search and filter evidence by type, status, or keyword
- 47 pre-loaded evidence items across 9 devices for case FS-2026-0314

### 🧪 Integrity Verification
- Periodic **SHA-256 re-verification** of stored evidence files
- Streaming hash computation (64KB chunks) — handles 500GB+ disk images
- Three result states: `PASS` / `FAIL` (tampered) / `ERROR` (file inaccessible)
- All verification results logged permanently with timestamp and analyst
- **ISO/IEC 27037:2012 compliant** integrity audit trail

### 📊 Forensic Analytics
- Attack timeline reconstruction from custody timestamps
- 4 predefined forensic SQL queries (evidence inventory, device inventory, chain of custody, access violations)
- RBAC security audit panel — violation counts, flagged accounts
- JSON report generation for legal and regulatory submission
- **Court-ready chain-of-custody PDF export**

### 🌐 Web Interface
- Full dark-themed forensic interface — 8 screens
- Real-time dashboard with evidence counts, recent activity, system log
- SQL Console — run predefined forensic queries in-browser
- Evidence intake form wired to live PostgreSQL database
- Visual attack timeline screen

---

## 🗄 Database Schema

```
8 core tables + 3 auth/integrity tables = 11 total

Analyst ──< Analyst_Role >── Role
Analyst ──< Evidence
Analyst ──< Chain_of_Custody
Case_File ──< Device ──< Evidence ──< Chain_of_Custody
Audit_Log
cfems_users ──< cfems_sessions
integrity_check
```

**Design principles:** 3NF normalisation | ISO/IEC 27037:2012 | Sandhu & Feinstein (1994) RBAC | NIST SP 800-53 Rev.5 dual-hash | OWASP Top Ten parameterised queries

---

## 🚀 Quick Start

### Prerequisites
- Python 3.10+
- PostgreSQL 16+
- pip

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/cfems.git
cd cfems
```

### 2. Configure environment
```bash
cp .env.example .env
# Edit .env and set DB_PASSWORD to your PostgreSQL password
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Create and populate the database
```bash
# Create the database
psql -U postgres -c "CREATE DATABASE cfems;"

# Run all SQL files in order
psql -U postgres -d cfems -f database/01_schema.sql
psql -U postgres -d cfems -f database/02_seed_data.sql
psql -U postgres -d cfems -f database/03_rbac.sql
psql -U postgres -d cfems -f database/04_queries.sql
psql -U postgres -d cfems -f database/05_auth.sql
```

### 5. Seed user accounts
```bash
python backend/auth.py seed
```

### 6. Start the server
```bash
python backend/server.py
```

### 7. Open the interface
```
http://localhost:5000
```

---

## 🔑 Default Login Credentials

| Username | Password | Role |
|----------|----------|------|
| `DR.CHEN` | `CFpassw0rd!LFA` | Lead Forensic Analyst |
| `J.PATEL` | `CFpassw0rd!JA` | Junior Analyst |
| `M.TORRES` | `CFpassw0rd!DFS` | Data Forensic Specialist |
| `K.NAKA` | `CFpassw0rd!DFS` | Data Forensic Specialist |
| `R.WALSH` | `CFpassw0rd!CS` | Compliance Supervisor |

> ⚠️ Change all default passwords before any production or public deployment.

---

## 📁 Project Structure

```
cfems/
├── backend/
│   ├── server.py               # Flask REST API — 15 endpoints
│   ├── auth.py                 # Authentication — bcrypt + session tokens
│   ├── evidence_intake.py      # Evidence ingestion with hash verification
│   ├── integrity_checker.py    # Periodic SHA-256 re-verification
│   └── report_generator.py     # JSON + PDF forensic report generation
├── database/
│   ├── 01_schema.sql           # 8 tables, indexes, triggers
│   ├── 02_seed_data.sql        # 47 evidence items, 5 analysts, case data
│   ├── 03_rbac.sql             # GRANT/REVOKE, RLS, audit trigger
│   ├── 04_queries.sql          # 7 forensic SQL queries
│   └── 05_auth.sql             # Auth + integrity tables
├── frontend/
│   └── CFEMS_System.html       # Full web interface
├── screenshots/
│   ├── 01_command_dashboard.png
│   ├── 02_evidence_registry.png
│   ├── 03_sql_console.png
│   └── 04_python_rbac.png
├── docs/
│   └── SETUP.md
└── requirements.txt
```

---

## 🔌 API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/health` | Database connection check |
| `GET` | `/api/dashboard` | Dashboard statistics |
| `GET` | `/api/evidence` | Evidence list with search |
| `POST` | `/api/intake` | Register new evidence item |
| `GET` | `/api/custody` | Chain of custody records |
| `GET` | `/api/analysts` | Analyst list + RBAC info |
| `GET` | `/api/audit` | Audit log + violations |
| `POST` | `/api/query` | Run predefined forensic query |
| `POST` | `/api/login` | Authenticate + get session token |
| `GET` | `/api/session` | Validate session token |
| `POST` | `/api/logout` | Invalidate session |
| `POST` | `/api/integrity/<id>` | Verify single evidence item |
| `POST` | `/api/integrity/verify-all` | Batch integrity verification |
| `GET` | `/api/integrity/history` | Integrity check history |

---

## 👥 RBAC Roles

| Role | Code | Permissions |
|------|------|-------------|
| Lead Forensic Analyst | `LFA` | Full read/write. Cannot DELETE custody or audit records. |
| Data Forensic Specialist | `DFS` | Read all. Insert/update evidence and custody. Cannot modify case records. |
| Junior Analyst | `JA` | Read evidence/devices. Insert evidence and custody entries only. |
| Compliance Supervisor | `CS` | Read-only across all tables including audit logs. |

---

## 🛡 Security Architecture

- **Passwords:** bcrypt with cost factor 12 — industry standard for credential storage
- **Sessions:** Cryptographically random 64-char SHA-256 tokens, 8-hour expiry
- **SQL Injection:** All queries use psycopg2 parameterised statements (OWASP Top Ten 2021)
- **Audit Trail:** Every database action logged — analyst, role, IP, timestamp, result
- **Chain of Custody:** Row-Level Security enforces append-only — no UPDATE or DELETE
- **Evidence Integrity:** Dual hash (MD5 + SHA-256) on every ingest, periodic re-verification
- **RBAC:** PostgreSQL-level GRANT/REVOKE — enforced even if application layer is bypassed

---

## 📚 Academic References

| Reference | Applied In |
|-----------|------------|
| Sandhu & Feinstein (1994) | RBAC three-tier architecture |
| ISO/IEC 27037:2012 | Chain-of-custody design, integrity verification |
| NIST SP 800-53 Rev.5 | Dual hash verification |
| OWASP Top Ten (2021) | Parameterised queries, injection prevention |
| Coronel & Morris (2017) | 3NF database normalisation |
| Poltavtseva et al. (2024) | Heterogeneous forensic data models |

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 🎓 Academic Context

Built for: **Database and Programming Concepts** — Mid-Semester Examination 2026  
Program: **Data Forensics and Cyber Security**  
Organisation: CipherForensics Pty Ltd (fictional)  
Case: FS-2026-0314 — LockBit 3.0 Ransomware Investigation

---

*CFEMS v2.4.1 — CipherForensics Pty Ltd — CONFIDENTIAL FORENSIC USE ONLY*
