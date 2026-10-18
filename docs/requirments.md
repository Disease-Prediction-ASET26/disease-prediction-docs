# Requirements Analysis Document: Disease Prediction Application

## 1. Introduction
This document outlines the requirements analysis for the Disease Prediction App, an AI-driven healthcare tool designed to bridge the communication gap between patients and medical professionals. The application will allow users to input their symptoms and receive a calibrated probability of potential diseases, paired with detailed explanations and precautions. To enhance clinical utility, the system will feature a dual interface: a Patient Portal for symptom input and self-assessment, and a Doctor Portal for reviewing patient-generated symptom reports and algorithmic predictions. 

## 2. Actors Identification
A critical step in determining the system's scope is identifying the primary actors who will interact with the application:

*   **Patient (Primary Actor):** An end-user experiencing symptoms who utilizes the application to gain insights into their potential condition and securely forward these insights to a medical professional.
*   **Doctor / Healthcare Provider (Secondary Actor):** A medical professional who uses the system to receive, review, and analyze the symptom reports and AI predictions submitted by the patient.
*   **Machine Learning System (System Actor):** The backend predictive engine responsible for analyzing inputted symptoms and calculating disease probabilities based on trained healthcare datasets.
*   **Administrator (Support Actor):** IT personnel responsible for managing user accounts, maintaining system security, and updating the machine learning models.

## 3. Major System Components
The application is structured around three major components to serve the dual-interface requirement and the core predictive functionality:

1.  **Patient Interface:** A user-friendly dashboard where users can describe their symptoms using either free-text input or by selecting from a structured checklist. 
2.  **Predictive Engine (ML Backend):** The core component that processes symptom data to predict the most likely diseases, calculates a probability score for each disease, and pairs every prediction with a disease description and general precautions.
3.  **Doctor Interface:** A secure dashboard where medical professionals can receive and review the comprehensive symptom reports, including the system-calculated scores and explanations.

## 4. Use Case Scenarios

### Use Case 1: Patient Symptom Assessment and Report Generation
*   **Actor:** Patient
*   **Precondition:** The patient is logged into the Patient Interface.
*   **Main Success Scenario:**
    1. The patient navigates to the "New Assessment" module.
    2. The patient inputs their current symptoms using a structured checklist and supplementary text descriptions.
    3. The system's predictive engine analyzes the data.
    4. The system outputs a list of the most likely diseases associated with the inputted symptoms.
    5. The system displays a calculated probability score for each potential disease.
    6. The patient views the predictions, which include system explanations, disease descriptions, and recommended general precautions.
    7. The patient clicks "Send Report to Doctor."
    8. The system securely routes the report to the designated doctor's queue.

### Use Case 2: Doctor Reviews Patient Symptom Report
*   **Actor:** Doctor
*   **Precondition:** The doctor is logged into the Doctor Interface and a patient has submitted a report.
*   **Main Success Scenario:**
    1. The doctor receives a notification of a new patient report.
    2. The doctor opens the report within their secure dashboard.
    3. The doctor reviews the raw symptoms inputted by the patient (text and checklist data).
    4. The doctor reviews the system-generated output, including the most likely diseases, probability scores, and the AI's explanation for the prediction.
    5. The doctor acknowledges the report, using the AI's calibrated probabilities as a supplementary tool for their upcoming clinical consultation.

## 5. Functional Requirements
| Category | Requirement ID | Description |
| :--- | :--- | :--- |
| **Patient Features** | FR-P1 | The system shall allow users to describe symptoms via free text or a structured checklist. |
| | FR-P2 | The system shall output a list of the most likely diseases associated with the symptoms. |
| | FR-P3 | Every prediction shall be paired with a disease description and general precautions. |
| | FR-P4 | The system shall allow the patient to forward the completed assessment report to a registered doctor. |
| **Doctor Features** | FR-D1 | The system shall provide a secure inbox for doctors to receive patient symptom reports. |
| | FR-D2 | The system shall display the patient's raw symptom input alongside the system-calculated probability scores and explanations. |
| **Core/ML System** | FR-S1 | The system shall calculate a score indicating the probability that a patient with the inputted symptoms has disease X. |
| | FR-S2 | The system shall provide explanations for why a specific prediction was made based on the symptoms. |

## 6. End of Semester Implementation Scope
To ensure a successful and deliverable project by the end of the current semester, the following components will be fully implemented:

*   **Symptom Input Module:** Full implementation of the structured checklist and basic text input for patients.
*   **Machine Learning Predictive Engine:** A functional ML model integrated into the backend capable of outputting a list of likely diseases with calibrated probability scores.
*   **Prediction Results View:** The UI component displaying the disease descriptions, system explanations, and general precautions to the patient.
*   **Basic Dual Interface:** 
    *   A functional Patient Portal for running assessments.
    *   A functional Doctor Portal capable of receiving and rendering a read-only view of the patient's submitted report and the system's predictions.

*Advanced features such as EHR (Electronic Health Records) integration, real-time doctor-patient chat, and continuous active learning for the ML model will be deferred to future development phases.*