# HealthVault API

## Overview
HealthVault is a secure, scalable healthcare data management system designed to address critical challenges in healthcare, such as data silos, privacy concerns, and emergency access needs. It provides a minimalistic, extensible API for storing and retrieving encrypted medical records, with a hybrid access control model that balances centralized enforcement and decentralized authorization. Built for interoperability, it supports patient and doctor mobility across hospitals while ensuring compliance with regulations like Nigeria's NDPR, GDPR, and HIPAA.

The API uses immutable storage, zero-knowledge client-side encryption, and short-lived tokens for consent and emergency access, making it ideal for hospitals, developers, and regulators aiming to improve healthcare outcomes in Nigeria and beyond.

## Key Features
- **Immutable Storage**: Database with sequential log identifiers; no delete endpoints via API; external deletions create detectable gaps for auditability.
- **Zero-Knowledge Encryption**: Client-side AES-256 encryption/decryption ensures the server stores only encrypted data, reducing liability.
- **Centralized Consent Management**: API endpoints (`/patients/{patientId}/consent`) grant/revoke access, issuing short-lived `consent_token`s for interoperability.
- **Decentralized Authorization**: Developers control who can access data (e.g., doctors, institutions) and how (e.g., OTP, signed forms) before calling the API.
- **Emergency Override**: `/emergency/access` provides short-lived `emergency_access_token`s for minimal critical data (e.g., allergies, blood type), logged immutably.
- **Portability**: NIN-based identities for staff and unique `patient_id`s enable seamless data access across hospitals.
- **Minimal Design**: Six core endpoints for simplicity, extensible for custom workflows.

## Access Management ("Door and Key" Model)
HealthVault uses a hybrid access model, metaphorically a "house with no doors" (secure storage) enhanced with a centralized "door and key" (API endpoints and tokens), while leaving "who" and "how" to developers:
- **Standard Access**: Developers verify consent locally (e.g., signed form, OTP) → Call `/patients/{patientId}/consent` → Receive `consent_token` → Use in `/records/patient/{patientId}` to retrieve encrypted records.
- **Emergency Access**: Developers verify requesters (e.g., paramedic badge) → Call `/emergency/access` → Receive `emergency_access_token` → Access limited `emergency_info` records (e.g., blood type).
- **Key Management**: Developers handle decryption keys via secure vaults (e.g., AWS KMS); tokens prove authorization without exposing data.
- **Revocation**: Update consent via API; tokens expire naturally (e.g., 24-48 hours).
- **Auditability**: All actions logged with sequential `log_id`s, ensuring tamper-evident audits for compliance (e.g., NDPR).

## Real-World Scenarios
1. **Patient Registration**: John Doe registers at General Hospital Lagos (`/patients/register`), storing encrypted `emergency_info` (e.g., blood type: O+, allergies: penicillin). Consent granted via `/patients/{patientId}/consent`.
2. **Patient Moves Hospitals**: John moves to Specialist Center Abuja, shares `patient_id` (e.g., via ID card). Center verifies consent (e.g., OTP) → Grants via API → Accesses records. Old hospital's access revoked.
3. **Doctor Moves Hospitals**: Dr. Adebayo moves to Specialist Center, re-registers NIN (`/staff/register`). Patient grants consent to new hospital/doctor. Old access revoked.
4. **Emergency Access**: John is unconscious in an ambulance. Paramedic (verified locally) calls `/emergency/access` → Receives `emergency_access_token` → Retrieves `emergency_info`. Access logged for review.

## Installation and Setup
### Prerequisites
- **Database**: PostgreSQL or MongoDB (with replicas for redundancy).
- **Server**: Node.js, Python, or similar for API implementation.
- **Client**: Libraries for AES-256 encryption (e.g., Web Crypto API, PyCrypto).
- **Authentication**: JWT provider (e.g., Auth0) or API key system.

### Steps
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/healthvault/healthvault-api.git
   cd healthvault-api
