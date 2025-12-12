# Use Case Description: Cloud-Based Pathology Slide Tracking (Pathology Tracking)
# Problem Statement
The problem being solved is the inefficient, non-standardized, and potentially non-compliant tracking of pathology slides and their associated reports when a large hospital sends them to an external reference laboratory for specialized procedures, such as Immunohistochemistry (IHC) staining.

The process is time-critical; the stained slides and the final IHC report must be returned to the hospital's primary pathologist within five business days for timely diagnosis and treatment planning. Manual tracking via phone calls, emails, or paper manifests introduces delays, risk of lost samples, and makes auditing the 5-day Service Level Agreement (SLA) compliance difficult.

Hospital Pathologist/Staff (Users): Need a simple, secure interface to:
### 1. Create a shipment manifest for a batch of slides.
### 2. Track the real-time status of each slide (e.g., "In Transit," "Staining in Progress," "Quality Control," "Report Finalized").
### 3. Receive an immediate notification upon the final report's availability.
## Securely access the final digital IHC report (PDF) and the digital whole slide image (WSI).
### 4. Reference Lab Staff (Users): Need a secure interface to:
### 5. Confirm receipt of the physical slides and digital manifest.
### 6. Update the status of individual slides at key workflow milestones (e.g., staining complete, QC approved).
### 7. Upload the final WSI (DICOM) and report (PDF).


# Data Sources
The solution will handle two primary types of data, both containing Protected Health Information (PHI), mandating a HIPAA-compliant environment.

| Data Type | Format | Source/Origin | Use Case |
|-----------|--------|---------------|----------|
| Tracking/Metadata | FHIR JSON (or structured database tables) | Hospital LIS/EHR extract, User input (Flask App) | Case accession number, Patient ID (encrypted), Specimen ID, Test ordered (IHC marker), Ship date, Current Status, Return Date. |
| Whole Slide Images (WSI) | DICOM (standard for digital pathology imaging) | Reference Lab's Whole Slide Scanner | The raw digital image of the stained tissue. (Large files: Gigabytes per slide) |
| Final Report | PDF/FHIR Document Reference | Reference Lab's Reporting System | The final IHC interpretation report signed out by the pathologist. |
| Audit Logs | Structured Logs (e.g., JSON) | Cloud Logging, Application Events | Timestamped records of all status changes, data access, and file uploads to ensure compliance and traceability. |

Basic Workflow (1 → 2 → 3…)
### 1. Shipment Creation & Manifest Upload: The Hospital Staff uses the Flask Web App to input/upload a manifest (e.g., a CSV or simple form data) detailing the slides being shipped. This action triggers a Cloud Function to generate a unique Shipment ID and create initial status records ("Manifest Created") in the Cloud SQL database and the Cloud Healthcare API (FHIR Store).
### 2. Physical Transport & Slide Digitization: The physical slides are shipped. The digital manifestation is shared with the Reference Lab. Upon receipt, the Reference Lab Staff accesses the system (via the same web app or an API endpoint) and updates the status to "Received at Lab". The lab then performs the IHC staining and digitizes the slide, creating a Whole Slide Image (WSI) in DICOM format.
### 3. WSI and Report Upload: The Reference Lab uploads the large DICOM WSI file to a Cloud Storage bucket that is managed by the Cloud Healthcare API (DICOM Store), and the final IHC Report (PDF) to a secure folder. This upload triggers a Cloud Function.
### 4. Status Update & Notification: The triggered Cloud Function updates the case status to "Report Finalized, Ready for Review" in Cloud SQL and the FHIR Store. This action simultaneously pushes a notification (via Pub/Sub) to the Hospital Pathologist/Staff.
### 5. Access and Review: The Hospital Pathologist logs into the Flask App, receives the notification, and uses a secure link to:
### 6. View the final report (PDF).
### 7. Access the WSI directly from the Cloud Healthcare API's DICOM web endpoint.
### 8.Analytics: All tracking status data in the FHIR store is exported to BigQuery for analysis, such as calculating the average turnaround time and compliance with the 5-day SLA.