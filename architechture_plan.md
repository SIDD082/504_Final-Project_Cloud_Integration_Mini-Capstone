# Architecture and Implementation Plan
The system, Pathology Tracking, is designed around serverless and managed services for scalability and HIPAA compliance, leveraging the Cloud Healthcare API as the secure data gateway for medical data standards (DICOM for images, FHIR for metadata).
3. Architecture & Implementation Plan
1. Service Mapping (GCP)
This table outlines the core GCP services and their specific roles in the PathTrack solution.

| Layer | Service (Cloud) | Role in Solution | Related Assignment/Module |
|-------|-----------------|------------------|--------------------------|
| Frontend/Compute | Cloud Run | Runs the containerized Python/Flask application to provide the secure web interface for hospital/lab staff to enter data, check status, and retrieve reports/images. | Containerization/Serverless Module |
| Secure Data Standard Storage | Cloud Healthcare API (FHIR Store & DICOM Store) | HIPAA-compliant storage and API endpoint for structured patient/case metadata (FHIR) and the very large Whole Slide Images (DICOM). This is the secure, auditable gateway for PHI. | AI/Analytics Module (using specialized APIs) |
| Structured/Relational DB | Cloud SQL (PostgreSQL) | Stores application-specific configuration, user authentication data, and simplified, query-friendly tracking status tables for the Flask app's main dashboard. | Managed Database Module |
| Data Orchestration/ETL | Cloud Functions | Serverless compute triggered by events (e.g., new file upload to DICOM store, status change in Cloud SQL). Executes small Python scripts for data transformation, status updates, and notifications. | Serverless Computing Module |
| Analytics | BigQuery | Serverless Data Warehouse for running complex SQL queries on de-identified data exported from the FHIR Store to calculate metrics like average turnaround time and SLA compliance. | Managed Data Warehouse Module |

2. Data Flow Narrative
The end-to-end flow is designed to be asynchronous and event-driven to ensure high performance and minimize "always-on" compute costs.
### Step 1: Manifest Creation: The Hospital Staff uses the Flask App (hosted on Cloud Run) to submit a new shipment manifest. The Flask app writes the basic tracking data (Case ID, Shipment ID, IHC order) to both the Cloud SQL (for quick dashboard lookups) and the Cloud Healthcare API (FHIR Store) for regulatory compliance.
### Step 2: Physical Receipt & Image Upload: When the physical slides arrive at the Reference Lab, the staff updates the status in the Flask App. After staining, the lab's scanner uploads the resulting DICOM WSI file directly to the Cloud Healthcare API's DICOM Store via the DICOMweb API.
### Step 3: Event Trigger: The successful upload of the DICOM file to the DICOM Store generates an event that is published to a Pub/Sub topic.
### Step 4: Status Update & Notification: A Cloud Function is subscribed to the Pub/Sub topic. It triggers, updates the status to "Report Finalized" in Cloud SQL and the FHIR Store, and sends a secure email/push notification to the relevant Hospital Pathologist.
### Step 5: Review & Audit: The Hospital Pathologist logs into the Flask App to securely retrieve the report (served from Cloud Storage) and the WSI (streamed via the Cloud Healthcare API's DICOMweb endpoint).
### Step 6: Analytics Pipeline: A scheduled job (e.g., Cloud Function or Cloud Composer) runs daily to export PHI-de-identified tracking metadata from the FHIR Store into a table in BigQuery for long-term analytics and performance reporting.


# 3. Security, Identity, and Governance Basics

- Managing Credentials
  - Use GCP Service Accounts with the principle of least privilege.
  - Cloud Run and Cloud Functions run as those Service Accounts and obtain credentials via the internal metadata server.
  - Avoid hardcoded environment variables for keys; use Secret Manager for Flask/application secrets.

- Access Controls / RBAC
  - Use IAM for fine-grained access control.
  - Hospital Staff: read access to Cloud Run; read/write access to a specific subset of Cloud SQL (e.g., status, manifest); read-only access to final reports.
  - Reference Lab Staff: read access to manifest data; write access for status updates and to the Cloud Healthcare API's DICOM Store (file uploads).
  - Pathologists: privileged read access to Cloud Healthcare API and Cloud SQL for data retrieval.

- Avoiding PHI in Public Environments
  - Deploy Cloud Run, Cloud SQL, and Cloud Healthcare API within a VPC network.
  - Use Cloud Healthcare API as the primary PHI store (DICOM/FHIR) with encryption at rest and in transit.
  - Tokenize or encrypt patient identifiers at the application layer before writing to Cloud SQL; maintain true PHI linkage only inside the Cloud Healthcare API.


4. Cost and Operational Considerations
  - Cost Drivers
    - Cloud Healthcare API (especially the DICOM Store for large WSI files) — premium, compliance-focused costs.
    - Cloud Run / Cloud Functions — compute costs that scale with traffic.
    - Cloud SQL — predictable "always-on" cost for managed database instances.
    - BigQuery — costs driven by data scanned and query volume.
  - Serverless vs Always-On VMs
    - Cloud Run scales to zero when idle to reduce costs.
    - Cloud Functions replace always-on pollers or dedicated VMs for event-driven processing.
    - BigQuery's pay-per-query pricing avoids large upfront data warehouse provisioning.
  - "Student Budget" Decisions
    - Use Cloud SQL Free Tier when available or start with the smallest instance and restrict runtime to development hours.
    - Aggressively leverage Cloud Run and Cloud Functions free tiers.
    - Prototype trade-off: use a Cloud Storage bucket for WSI/report uploads instead of the Cloud Healthcare API to reduce costs; document the trade-off while demonstrating functionality.
  - Operational Notes
    - Monitor Cloud Healthcare API and storage usage (WSI size/ingress) to control costs.
    - Set budgets and alerts in GCP to avoid unexpected charges.
