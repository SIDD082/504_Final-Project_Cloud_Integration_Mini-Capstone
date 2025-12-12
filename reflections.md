# Confidence and Uncertainty

I'm most confident in the architecture's core choice of managed, serverless GCP services. The reliance on Cloud Run, Cloud Functions, and BigQuery elegantly solves the twin concerns of scalability and cost. The event-driven flow (Pub/Sub) for status updates is fundamentally robust and reliable. Crucially, I'm confident in the separation of duties: Cloud SQL for simple application logic, and the Cloud Healthcare API for specialized, compliant medical data.

My area of greatest uncertainty is the direct, real-world enterprise integration with a hospital's on-premises Laboratory Information System (LIS). While our design calls for a secure web upload, a true production system would require a secure, direct interface (likely an HL7v2 or FHIR connection over a dedicated Cloud VPN/Interconnect) to automatically pull manifest data and push results back into the Electronic Health Record (EHR). Implementing this level of complex, secure enterprise communication is a major undertaking that is beyond the scope of a capstone project.

## Alternative Architecture Considered
I initially considered a Monolithic VM Architecture, using a single Compute Engine VM to run the Flask app, a local database, and file storage.
Why I Rejected It:
- Compliance Nightmare: 
  - Placing the burden of configuring the VM and local database to be HIPAA-compliant (auditing, fine-grained access, encryption) on the developer is complex and error-prone.
  - The Cloud Healthcare API offloads much of this responsibility.

- Zero Scalability:
  - A single VM does not scale; increased hospital volume would cause failures and SLA violations.
  - Serverless design (Cloud Run/Functions) scales automatically from zero to many instances.

- DICOM Complexity:
  - Storing massive DICOM files locally or in a generic bucket complicates high-performance viewing.
  - The Cloud Healthcare API's DICOMweb endpoint is designed for integration with standard pathology image viewers.


## Future Steps if given unlimited Resources:
If given unlimited time and budget, I would focus on complete automation and advanced clinical intelligence:
Full LIS/EHR Integration: Implement a Cloud VPN to securely link the hospital network to our GCP environment. Dedicated Cloud Functions would then automatically ingest HL7 messages from the LIS, eliminating manual manifest uploads entirely.
AI-Powered Quality Control (QC): Integrate Vertex AI to deploy a simple AI model. This model would automatically check every newly uploaded DICOM WSI for common artifacts (e.g., out-of-focus areas or staining issues) and flag the case for manual re-review before the pathologist signs it out.
Real-time Performance Dashboard: Build an executive dashboard using Looker (or Looker Studio) connected directly to the BigQuery warehouse. This would provide hospital administration with a real-time, interactive view of metrics like "Current Average Turnaround Time" and "SLA Compliance Rate by Marker."
