# Tamper-Evident Logging System for Secure Forensic Analysis

A lightweight Python-based logging system that uses **SHA-256 cryptographic hash chaining** to detect unauthorized modification, deletion, or injection of log entries. Includes both a CLI and a Flask web interface with a live audit dashboard.

---

## How It Works

Every log entry is cryptographically linked to the one before it. If anyone tampers with even a single character in any log — the entire chain breaks and the system flags it instantly.

```
Log #1  →  SHA-256(log_id | timestamp | event | message | GENESIS)  =  hash_1
Log #2  →  SHA-256(log_id | timestamp | event | message | hash_1)   =  hash_2
Log #3  →  SHA-256(log_id | timestamp | event | message | hash_2)   =  hash_3
```

During verification, hashes are recomputed from scratch and compared against stored values. Any mismatch = tampering detected, exact position reported.

---

## Project Structure

```
├── app.py               # Flask web server + REST API
├── main.py              # CLI interface
├── logfiles.py          # Log entry generation
├── hash_chain.py        # SHA-256 hash chaining logic
├── verify_logs.py       # Chain verification + audit report generation
├── Front_end_tamper.html  # Web dashboard (Secure Log Console)
├── logs.jsonl           # Log storage (one JSON object per line)
└── audit_report.txt     # Generated after each verification run
```

---

## File Breakdown

### `logfiles.py` — Log Generation
Creates a new log entry with a sequential `log_id`, ISO timestamp, `event_type`, and `message`. Appends it to `logs.jsonl` with placeholder hashes (`NULL`) until hash chaining is applied.

```python
generate_log("LOGIN", "User admin logged in")
```

Each entry looks like:
```json
{
  "log_id": 1,
  "timestamp": "2026-03-03 20:10:59",
  "event_type": "LOGIN",
  "message": "User admin logged in",
  "previous_hash": "GENESIS",
  "current_hash": "9132cbf2..."
}
```

---

### `hash_chain.py` — Hash Chaining
Reads all entries in `logs.jsonl`, computes SHA-256 for each using:
```
hash_input = log_id | timestamp | event_type | message | previous_hash
```
The first entry uses `"GENESIS"` as its previous hash. Overwrites the file with updated hash fields.

---

### `verify_logs.py` — Integrity Verification
Walks through every log entry and recomputes hashes. Detects two types of tampering:

| Tampering Type | How It's Detected |
|---|---|
| **Modification** | Recomputed hash ≠ stored `current_hash` |
| **Deletion / Insertion** | `log_id` sequence breaks (e.g. jumps from 3 to 5) |

Writes result to `audit_report.txt` with timestamp, status, tampered log ID, and reason.

---

### `app.py` — Flask API
Serves the web frontend and exposes these endpoints:

| Method | Endpoint | What it does |
|---|---|---|
| `GET` | `/` | Serves the HTML dashboard |
| `GET` | `/logs` | Returns all logs as a JSON array |
| `POST` | `/add` | Adds a new log entry |
| `POST` | `/hash` | Applies hash chaining across all logs |
| `GET` | `/verify` | Verifies chain + returns per-entry validity |
| `GET` | `/report` | Downloads `audit_report.txt` |

---

### `main.py` — CLI
```bash
python main.py add LOGIN "User admin logged in"   # Add a log
python main.py hash                                # Apply hash chain
python main.py verify                              # Verify integrity
python main.py report                              # View audit report
```

---

## Getting Started

### Prerequisites
```bash
pip install flask flask-cors
```

### Run the web interface
```bash
python app.py
```
Opens automatically at `http://localhost:5000`

### Or use the CLI
```bash
# 1. Add some logs
python main.py add LOGIN "User admin logged in"
python main.py add FILE_ACCESS "Accessed secret.pdf"
python main.py add LOGOUT "User admin logged out"

# 2. Apply hash chaining
python main.py hash

# 3. Verify integrity
python main.py verify

# 4. View audit report
python main.py report
```

---

## Sample Audit Report

```
===== AUDIT REPORT =====
Verification Time: 2026-04-21 07:17:53
Status: FAILED
Tampering Detected At Log ID: 6
Reason: Hash mismatch (log modification detected)
========================
```

---

## Limitations

- Detects tampering but does not prevent it
- Verification is not real-time — must be triggered manually
- Requires manual re-ingestion for continuous chaining
- Performance may degrade with very large log volumes

---

## Future Work

- Real-time log monitoring with automated verification
- Integration with OS-level logs (Windows Event Viewer, Linux syslog)
- DBMS-backed storage for scalable log management
- Web-based alert system for instant tamper notifications

---

## Team

| Name | USN |
|---|---|
| Nimai R S | 1DS23CB026 |
| Ujjwal M | 1DS23CB059 |
| Shreya M | 1DS23CB050 |
| Shreesha Gowda | 1DS23CB049 |

Under the guidance of **Mrs. Sanjana Shankar**, Assistant Professor,
Department of CSBS, Dayananda Sagar College of Engineering, Bengaluru.
