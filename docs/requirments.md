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
# Requirements Analysis Document: Disease Prediction Application

## 1. Introduction
This document outlines the requirements analysis for the Disease Prediction App, an AI-driven healthcare tool designed to bridge the communication gap between patients and medical professionals. The application allows users to input their symptoms and receives a calibrated probability of potential diseases, paired with detailed explanations and precautions[cite: 1]. 

To optimize the patient journey, the app bypasses the traditional family doctor bottleneck for non-subsidized care by acting as an AI triage system. If the system detects a high probability of a specific condition, it directly routes the patient to the relevant specialist. Furthermore, strict adherence to General Data Protection Regulation (GDPR) standards is a core foundational requirement.

## 2. Actors Identification
*   **Patient (Primary Actor):** An end-user experiencing symptoms who utilizes the application for self-assessment, triage, and direct specialist routing.
*   **Specialist Doctor (Secondary Actor):** A specialized medical professional who uses the system to receive patient symptom reports and AI predictions prior to consultation.
*   **Machine Learning System (System Actor):** The backend predictive engine responsible for analyzing inputted symptoms and calculating disease probabilities[cite: 1].
*   **Data Protection Officer / Administrator (Support Actor):** IT and compliance personnel responsible for maintaining system security, updating ML models, and ensuring GDPR compliance.

## 3. Major System Components
1.  **Patient Interface:** A dashboard for symptom input (text and structured checklist)[cite: 1], viewing predictions, and managing data privacy settings.
2.  **Predictive Engine & Triage Module:** The core ML component that predicts diseases, calculates probability scores, provides explanations[cite: 1], and maps the predicted disease to the correct medical specialty.
3.  **Specialist Doctor Interface:** A secure dashboard where specialists receive and review comprehensive symptom reports and ML predictions[cite: 1].
4.  **Security & Privacy Module (GDPR):** The infrastructure managing explicit user consent, end-to-end encryption of health data, and automated handling of "Right to be Forgotten" requests.

## 4. Use Case Scenarios

### Use Case 1: Patient Triage and Direct Specialist Routing
*   **Actor:** Patient
*   **Main Success Scenario:**
    1. The patient explicitly consents to health data processing.
    2. The patient inputs symptoms using a structured checklist and free-text[cite: 1].
    3. The predictive engine outputs a list of the most likely diseases with probability scores[cite: 1].
    4. The system suggests a direct consultation with a specific Specialist Doctor.
    5. The patient forwards their secure report to an available specialist in the network.

### Use Case 2: Emergency Red Flag Detection
*   **Actor:** Patient, Machine Learning System
*   **Main Success Scenario:**
    1. The patient inputs severe symptoms (e.g., radiating chest pain, inability to speak).
    2. The system immediately recognizes these as critical red flags.
    3. The system halts the standard ML prediction output and specialist routing.
    4. The system displays a high-priority alert instructing the patient to contact emergency services (112) immediately.

### Use Case 3: Patient Exercises "Right to be Forgotten"
*   **Actor:** Patient, Privacy Module
*   **Main Success Scenario:**
    1. The patient selects "Delete My Data" in their account settings.
    2. The system permanently erases the patient's personal identifiable information (PII) and health records from all active databases and doctor queues within 30 days.

## 5. Functional and Non-Functional Requirements

### Functional Requirements (FR)
| ID | Description |
| :--- | :--- |
| **FR-P1** | The system shall allow users to describe symptoms via free text or a structured checklist[cite: 1]. |
| **FR-P2** | The system shall output a list of likely diseases alongside a calibrated probability score[cite: 1]. |
| **FR-P3** | Every prediction shall be paired with a disease description, system explanation, and precautions[cite: 1]. |
| **FR-P4** | The system shall map predicted diseases to specific medical specialties for direct routing. |
| **FR-P5** | The system shall detect critical emergency symptoms and redirect the user to emergency services, halting standard triage. |
| **FR-D1** | The system shall provide a secure inbox for doctors to review patient symptom reports[cite: 1]. |
| **FR-G1** | The system shall provide a mechanism for users to grant and revoke explicit consent for health data processing. |
| **FR-G2** | The system shall allow users to permanently delete their account and associated medical data. |

### Non-Functional Requirements (NFR)
| ID | Description |
| :--- | :--- |
| **NFR-1** | **GDPR Compliance:** The application must strictly adhere to the EU GDPR, specifically Article 9 regarding special categories of personal data. |
| **NFR-2** | **Data Minimization:** Health data used to fine-tune future ML models must be stripped of PII prior to inclusion in training datasets. |

## 6. End of Semester Implementation Scope
*   **Symptom Input & Triage Module:** Implementation of the checklist, ML prediction engine, and basic specialty mapping logic[cite: 1].
*   **Emergency Guardrails:** Hardcoded detection of critical symptoms to trigger emergency alerts.
*   **Dual Interface MVP:** A Patient Portal for assessments and a Doctor Portal for reviewing submitted reports[cite: 1].
*   **Basic GDPR Framework:** User consent banners and a functional "Delete Account" database purge endpoint.

## 7. Use of Artificial Intelligence

Artificial Intelligence tools were integrated throughout the requirements engineering and architectural design phases as collaborative co-pilots. To ensure consistency, technical rigour, and regulatory compliance, the team utilized two distinct models for specific deliverables:

### 7.1 Requirements Refinement & Safety Architecture (Google Gemini 3.1 Pro)
Google Gemini was employed during the initial elicitation phase to transform the project rubric into a production-grade specification:
*   **Clinical Safety & Emergency Intercept:** Prompted to evaluate the risks of direct specialist triage, the model formulated the deterministic **Red Flag Emergency Intercept (Use Case 2)**, ensuring that acute conditions (e.g., myocardial infarction, stroke) abort standard probabilistic flows and redirect immediately to emergency services (112).
*   **GDPR Compliance by Design:** The model mapped EU Regulation into explicit functional requirements (`FR-G1`, `FR-G2`) and **Use Case 3**, defining consent management, training data anonymization, and database cascade deletions for the "Right to be Forgotten."

*   **Prompt 1 (Clinical Safety):** *"Please update the requirements document to implement an algorithmic triage and direct-to-specialist routing workflow... additionally, please assess and include any necessary clinical safety guardrails or emergency intercept protocols required when bypassing human primary care."*
    *   **Architectural Result:** The model identified the liability of automated triage and generated **Use Case 2 (Emergency Red Flag Detection)**, establishing a deterministic safety layer to bypass the ML engine during acute medical emergencies.
*   **Prompt 2 (Data Privacy):** *"Please revise the requirements markdown document to ensure strict compliance with the EU GDPR. Specifically, integrate requirements for handling 'special category' health data and detail a comprehensive 'Right to Erasure' "*
    *   **Architectural Result:** The model mapped the legal constraints to explicit system features, generating **Use Case 3** and specific functional requirements (`FR-G1`, `FR-G2`) for auditable consent and cascading database deletion protocols.

### 7.2 UML & C4 Architectural Modeling (Anthropic Claude Opus)
To translate the requirements into formal architectural representations across our 4-person team (delivering 4 UML diagrams and 4 C4 levels), Anthropic Claude Opus was utilized. 

#### Shared Context Strategy
To prevent naming drift and ensure cross-diagram alignment, a strict **Shared Context Block** was prepended to all member prompts. This enforced unified naming for Actors, Domain Entities, Enums, and C4 Containers.

*   **Shared Context Prompt (Pasted by all team members first):**
    > "You are helping a university student with a software engineering lab. The attached document is our team's requirements analysis for a "Disease Prediction Application". Our team has 4 members; each member produces one UML diagram and one C4 diagram. Grading rewards correct identification of elements, correct relationships between them, and use of advanced notation (include/extend, aggregation/composition, generalization, multiplicities, etc.). All names must be in English.
    > 
    > To keep all 8 diagrams consistent, use EXACTLY these names (do not rename or invent synonyms):
    > ACTORS: Patient, Doctor, Administrator, ML Prediction Engine (system actor), Email Service (external system).
    > DOMAIN CLASSES: User (abstract), Patient, Doctor, Administrator, Assessment, SymptomEntry, Symptom, SymptomChecklist, PredictionResult, DiseasePrediction, Disease, Precaution, Explanation, Report, Notification, MLModel, RedFlagRule.
    > ENUMS: ReportStatus {DRAFT, SENT, OPENED, REVIEWED}, Severity {MILD, MODERATE, SEVERE}, UserRole {PATIENT, DOCTOR, ADMIN}.
    > C4 CONTAINERS: Patient Web App (React), Doctor Web App (React), Admin Console (React), API Backend (Python/FastAPI), Prediction Service (Python/scikit-learn), Database (PostgreSQL), Model Store (file storage for trained models). External: Email Service.
    > 
    > AGREED BEHAVIOUR (in addition to the document):
    > - Users must log in; doctors are verified by an Administrator before they can receive reports.
    > - Each Assessment is checked against RedFlagRules (e.g. chest pain, difficulty breathing); if matched, an emergency warning is shown.
    > - Free-text symptoms are mapped to known Symptoms; unmapped text is kept and shown to the doctor.
    > - Predictions show the top 5 diseases with calibrated probabilities, an Explanation (top contributing symptoms), a description and Precautions, plus a "not a medical diagnosis" disclaimer.
    > - The doctor can add clinical notes when reviewing a report.
    > 
    > Rules:
    > - Stay within the document and the agreed behaviour above. If you must assume something, list it under "Assumptions".
    > - Produce diagrams as PlantUML code (for C4, use C4-PlantUML: !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Level.puml) so I can render them at plantuml.com."

#### Team Delegation & Prompts
*   **Alex P (Use-Case & C4 Level 1 Context):** Prompted Claude Opus to generate a 15–20 use-case diagram emphasizing advanced notation (`<<include>>` for mandatory analysis, `<<extend>>` for emergency red flags and free-text descriptions) along with the C4 System Context.
    *   **Prompt 1:** *"My task: (1) UML Use-Case Diagram, (2) C4 Level 1 System Context Diagram. 1. USE-CASE DIAGRAM for the whole system: A system boundary named 'Disease Prediction Application'. All actors from the shared list, with generalization where appropriate. Use cases covering: patient assessment, doctor work, administrator work, and the ML Prediction Engine's participation. Use `<<include>>` where behaviour is always performed and `<<extend>>` with extension points where it is optional/conditional. 2. C4 SYSTEM CONTEXT DIAGRAM: The Disease Prediction Application as one software system, the people and the external Email Service, with labelled relationships. Output for each diagram: a) PlantUML code. b) A table of every element. c) A table of every relationship. d) Assumptions. Finally, give me a short list of things I should double-check myself."*

*   **Daria (Class Diagram & C4 Level 2 Container):** Prompted for a domain model enforcing composition vs. aggregation boundaries, full multiplicities, role names, and container communication protocols.
    *   **Prompt 2:** *"My task: (1) UML Class Diagram, (2) C4 Level 2 Container Diagram. 1. CLASS DIAGRAM (domain model) using the shared class names: Attributes with types and visibility (+, -, #), and key methods. Inheritance: User is abstract; Patient, Doctor, Administrator extend it. Composition where parts cannot exist without the whole. Aggregation where parts are shared or independent. Plain associations with role names and multiplicities on BOTH ends. Dependencies. The enums ReportStatus, Severity, UserRole, used as attribute types. 2. C4 CONTAINER DIAGRAM: All containers from the shared list with technologies, plus the people and the external Email Service. Labelled relationships with protocols. Output for each diagram: a) PlantUML code. b) A table of every class/container with its responsibility. c) A table of every relationship. d) Assumptions."*

*   **Alex R (Sequence Diagram & C4 Level 3 Component - Prediction Service):** Prompted to model the end-to-end assessment lifecycle using combined fragments (`opt`, `alt`, `loop`), alongside component decomposition of the Prediction Service container.
    *   **Prompt 3:** *"My task: (1) UML Sequence Diagram, (2) C4 Level 3 Component Diagram of the Prediction Service. 1. SEQUENCE DIAGRAM for Use Case 1 'Patient Symptom Assessment and Report Generation', from login to the report reaching the doctor's inbox. Lifelines: Patient, Patient Web App, API Backend, Prediction Service, Database, Email Service, Doctor. Synchronous and asynchronous messages, return messages, activation bars. Combined fragments: 'opt', 'alt', 'loop', 'alt'. 2. C4 COMPONENT DIAGRAM of the Prediction Service container, with components such as: Prediction API Controller, Symptom Parser, Feature Vectorizer, Red-Flag Checker, Disease Classifier, Probability Calibrator, Explanation Generator, Disease Knowledge Base, Model Loader. Output for each diagram: PlantUML code, tables for lifelines/messages, and assumptions."*

*   **Carla (Activity Diagram & C4 Level 3 Component - API Backend):** Prompted for swimlane-partitioned workflow tracking (`Patient`, `System`, `Doctor`), explicit report state transitions, and the internal component architecture of the FastAPI backend.
    *   **Prompt 4 :** *"My task: (1) UML Activity Diagram, (2) C4 Level 3 Component Diagram of the API Backend. 1. ACTIVITY DIAGRAM of the end-to-end process, from the patient starting an assessment to the doctor finishing the review. Swimlanes (partitions): Patient, System, Doctor. Initial and final nodes, actions, decision and merge nodes with guards. A fork/join for parallel work. Object nodes where useful. 2. C4 COMPONENT DIAGRAM of the API Backend container, with components such as: Auth Controller, Assessment Controller, Report Controller, Doctor Inbox Controller, Admin Controller, Auth Service, Assessment Service, Report Service, Notification Service, Prediction Client, Repositories. Show connections to the three web apps, Database, Prediction Service and Email Service. Output: PlantUML code, tables, assumptions."*

### 7.3 Critical Evaluation & Model Limitations
While the AI accelerated development, human engineering oversight was required to validate and correct generated artifacts:

*   **Advantages Observed:**
    *   *High Acceleration:* Rapid conversion of textual specifications into directly renderable PlantUML and C4-PlantUML code.
    *   *Structural Consistency:* The shared context block eliminated vocabulary mismatches across independent team modules.
    *   *Notation Breadth:* Recommended edge cases and advanced notation patterns (e.g., specific UML extension points and fragment guards) that improved model expressiveness.
*   **Identified Limitations & Manual Interventions:**
    *   *Inverted Stereotype Semantics:* Claude Opus occasionally inverted UML `<<extend>>` arrow directions (directing from the base use case toward the extension rather than child to base).
    *   *Hallucinated Model Elements:* The model intermittently introduced speculative domain attributes (e.g., external billing and insurance gateways) outside the agreed project scope.
    *   *PlantUML Syntax Collisions:* Generated occasional macro syntax mismatches between standard PlantUML skins and official `C4-PlantUML` include headers, requiring manual script debugging prior to compilation.