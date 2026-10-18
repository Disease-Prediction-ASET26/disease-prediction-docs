# Requirements Analysis Document: Disease Prediction Application

## 1. Introduction
This document outlines the requirements analysis for the Disease Prediction App, an AI-driven healthcare tool designed to bridge the communication gap between patients and medical professionals. The application allows users to input their symptoms and receives a calibrated probability of potential diseases, paired with detailed explanations and precautions. 

To optimize the patient journey, the app is designed to bypass the traditional family doctor bottleneck for non-subsidized (private) care by acting as an AI triage system. If the system detects a high probability of a specific condition, it will directly recommend and route the patient to the relevant specialist. Furthermore, as the application handles sensitive personal health information, strict adherence to General Data Protection Regulation (GDPR) standards is a core foundational requirement.

## 2. Actors Identification
*   **Patient (Primary Actor):** An end-user experiencing symptoms who utilizes the application for self-assessment, triage, and to be routed to the appropriate specialist.
*   **Specialist Doctor (Secondary Actor):** A specialized medical professional (e.g., Cardiologist, Dermatologist) who uses the system to receive patient symptom reports and AI predictions prior to the direct consultation.
*   **Machine Learning System (System Actor):** The backend predictive engine responsible for analyzing inputted symptoms, calculating disease probabilities, and determining the appropriate specialist routing.
*   **Administrator / Data Protection Officer (Support Actor):** IT and compliance personnel responsible for managing user accounts, maintaining system security, updating ML models, and ensuring GDPR compliance (handling data deletion requests, audits).

## 3. Major System Components
1.  **Patient Interface:** A user-friendly dashboard for symptom input (text and structured checklist), viewing predictions, and managing data privacy settings.
2.  **Predictive Engine & Triage Module:** The core ML component that predicts diseases, calculates probability scores, and maps the predicted disease to the correct medical specialty (e.g., high probability of eczema -> Dermatology).
3.  **Specialist Doctor Interface:** A secure dashboard where specialists receive and review comprehensive symptom reports for incoming patients.
4.  **Security & Privacy Module (GDPR):** The infrastructure managing explicit user consent, end-to-end encryption of health data, anonymization for ML training, and automated handling of "Right to be Forgotten" requests.

## 4. Use Case Scenarios

### Use Case 1: Patient Triage and Direct Specialist Routing
*   **Actor:** Patient
*   **Main Success Scenario:**
    1. The patient logs into the Patient Interface and explicitly consents to health data processing (GDPR requirement).
    2. The patient inputs their current symptoms using a structured checklist and free-text descriptions.
    3. The predictive engine analyzes the data and outputs a list of the most likely diseases with probability scores.
    4. The system identifies a high probability for a specific condition requiring specialized care.
    5. The system displays the predictions, explanations, and general precautions.
    6. The system suggests a direct consultation with a specific type of Specialist Doctor (e.g., "Based on your symptoms, we recommend consulting a Neurologist").
    7. The patient agrees and forwards their secure report to an available specialist in the network.

### Use Case 2: Emergency Red Flag Detection
*   **Actor:** Patient, Machine Learning System
*   **Main Success Scenario:**
    1. The patient inputs severe symptoms (e.g., "severe chest pain radiating to the left arm" or "sudden inability to speak").
    2. The system immediately recognizes these as critical red flags.
    3. The system halts the standard prediction output and specialist routing.
    4. The system displays a high-priority, unblockable alert instructing the patient to call emergency services (112) or go to the nearest Emergency Room immediately.

### Use Case 3: Patient Exercises "Right to be Forgotten"
*   **Actor:** Patient, Privacy Module
*   **Main Success Scenario:**
    1. The patient navigates to the Account Settings menu.
    2. The patient selects "Delete My Data / Account."
    3. The system prompts for a final confirmation.
    4. The system permanently erases the patient's personal identifiable information (PII) and health records from all active databases and doctor queues within the mandated 30-day GDPR window.

## 5. Functional and Non-Functional Requirements

### Functional Requirements (FR)
| ID | Description |
| :--- | :--- |
| **FR-P1** | The system shall allow users to describe symptoms via free text or a structured checklist. |
| **FR-P2** | The system shall output a list of the most likely diseases associated with the symptoms alongside a calibrated probability score. |
| **FR-P3** | Every prediction shall be paired with a disease description and general precautions. |
| **FR-P4** | The system shall map predicted diseases to specific medical specialties and recommend direct routing to those specialists. |
| **FR-P5** | The system shall detect critical emergency symptoms and redirect the user to emergency services, halting standard triage. |
| **FR-D1** | The system shall provide a secure inbox for specialist doctors to receive and review patient symptom reports prior to consultation. |
| **FR-G1** | The system shall provide a mechanism for users to grant and revoke explicit consent for the processing of their health data. |
| **FR-G2** | The system shall allow users to download their health data in a portable format or permanently delete their account and associated data. |

### Non-Functional Requirements (NFR) - GDPR & Security
| ID | Description |
| :--- | :--- |
| **NFR-1** | **GDPR Compliance:** The application must strictly adhere to the EU General Data Protection Regulation, specifically Article 9 regarding the processing of special categories of personal data (health data). |
| **NFR-2** | **Data Encryption:** All sensitive patient data must be encrypted both in transit (TLS 1.2+) and at rest (AES-256) in the database. |
| **NFR-3** | **Data Minimization & Anonymization:** The ML system shall only use anonymized or pseudonymized data for future model training, ensuring no Personal Identifiable Information (PII) is linked to training datasets. |

## 6. End of Semester Implementation Scope
To ensure a successful and deliverable project by the end of the current semester, the following components will be fully implemented:

*   **Symptom Input & Triage Module:** Implementation of the structured checklist, ML prediction engine (outputting likely diseases, scores, and explanations), and the basic specialty mapping logic.
*   **Emergency Guardrails:** Hardcoded detection of top-tier critical symptoms to trigger emergency alerts.
*   **Dual Interface MVP:** 
    *   A Patient Portal for running assessments and receiving specialist recommendations.
    *   A Doctor Portal rendering a read-only view of the submitted reports.
*   **Basic GDPR Framework:** Implementation of user consent banners upon registration and a functional "Delete Account" button that removes user data from the database.