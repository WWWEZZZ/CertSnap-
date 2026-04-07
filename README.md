# CertSnap
The source code for this application is currently undergoing a security and IP audit prior to public release, as it is actively being tested in a live commercial MEP environment. A full architectural breakdown is provided below, and a live demonstration is available upon request.

**Automated MEP Asset Capture & Compliance Pipeline**

CertSnap is a multi-tenant, offline-first compliance and asset tracking platform designed for the Mechanical, Electrical, and Plumbing (MEP) industry. It bridges the gap between field engineers and office-based project managers, automating the extraction and collation of Operations & Maintenance (O&M) manuals, datasheets, and regulatory certificates.
## The Problem: The "Practical Completion" Bottleneck
In the commercial MEP sector, reaching "Practical Completion" on a project triggers final payment from the client. However, this payment is frequently delayed by weeks because Project Managers are waiting on engineers to manually compile and submit asset logs, equipment datasheets, and signed compliance certificates (e.g., Gas Safety CP12s).
Field engineers historically resist complex, bloated CAFM (Computer-Aided Facility Management) apps, leading to missing data, transcription errors, and severe cash-flow bottlenecks for the contractor.
## The Solution: Frictionless Field Data
CertSnap eliminates data-entry friction by turning the engineer's smartphone camera into the primary data-entry tool, supported by an asynchronous AI backend.
 1. **Snap & Extract:** An engineer snaps a photo of a boiler, HVAC or other piece of mechanical equipment's dataplate. The backend instantly extracts the make, model, and serial number. It also stores the image for reference.
 2. **Automated Enrichment:** A background worker searches the internet for the exact equipment model and automatically attaches the manufacturer's PDF datasheet to the project file.
 3. **Smart Certifications:** Assets are linked to regulatory certificates (e.g., CP12, CP17). Forms are auto-filled with extracted asset data and location APIs.
 4. **Real-Time PM Dashboard:** Project Managers receive a live, collated O&M package, enabling them to hand over the project and invoice the client the moment physical works are complete.
## Architecture & Tech Stack
CertSnap is built with a lean, highly opinionated stack prioritising speed, offline capabilities, and secure multi-tenancy.
### Frontend: Zero-Install PWA
 * **Vanilla HTML/CSS/JS:** Chosen over heavy SPA frameworks (React/Vue) to ensure micro-payloads and instant load times in low-connectivity boiler rooms.
 * **Offline-First (IndexedDB & Service Workers):** A robust local queue handles asset logging and certificate updates even with zero cellular signal, syncing automatically when connectivity returns.
 * **Passwordless Auth:** SMS OTP onboarding combined with a custom **AES-256-GCM PIN Vault**. JWTs are encrypted via the Web Crypto API using a PBKDF2-derived key, allowing engineers to unlock the app instantly with a 4-digit PIN while keeping tokens secure at rest.
### Backend: FastAPI & AI Integration
 * **Python / FastAPI:** High-performance, asynchronous routing handling mobile endpoints and the admin dashboard.
 * **Inline AI Pipeline:** Integrates **Google Gemini 2.5 Flash** for highly accurate, inline OCR extraction. Processed synchronously so the user UI updates instantly without polling.
 * **Background Workers:** FastAPI BackgroundTasks handle asynchronous internet searches to locate and download PDF datasheets without blocking the user response thread.
 * **Pixel-Perfect Document Generation:** Uses headless **Playwright (Chromium)** to render dynamic Jinja2 HTML templates into strictly formatted, regulatory-compliant PDF certificates.
### Database & Security: Supabase (PostgreSQL)
 * **True Multi-Tenancy:** Strict **Row Level Security (RLS)** policies enforce isolation. Every query validates the company_id against the decoded JWT, ensuring data cannot cross-pollinate between contractors.
 * **Append-Only Patterns:** Database architecture leans heavily on append-only logs and SQL Views for state resolution, preventing race conditions and maintaining a perfect audit trail.
## Key Engineering Decisions
 * **Playwright over HTML-to-PDF Libraries:** Swapped traditional Python PDF libraries (like WeasyPrint) for Playwright. While heavier, browser-based rendering was the only way to guarantee pixel-perfect output for strict regulatory forms.
 * **Dropping Redis/Celery for Native Background Tasks:** Reduced infrastructure bloat and points of failure by moving asynchronous datasheet enrichment directly into FastAPI's native BackgroundTasks.
 * **State Machine Database:** Moved business logic down to the PostgreSQL layer wherever possible. By utilising atomic SQL RPCs (e.g., incrementing sequential asset tags), the system safely handles highly concurrent mobile uploads.
