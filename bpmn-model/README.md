# Problem 3 — BPMN Model of the Symptom-Assessment Process

Team modeling deliverable (max 2 people). This folder contains a **BPMN 2.0**
process model of the **Symptom Assessment & Prediction** component of the
disease-prediction system. All names are in English, as required.

## File

| File | Purpose |
|------|---------|
| `disease-assessment-process.bpmn` | Standard **BPMN 2.0 XML** with diagram interchange (DI) layout. Opens in any BPMN 2.0 tool. |

## What is modeled

The end-to-end collaboration that happens when a patient checks their symptoms:
from starting an assessment, through the system's red-flag/emergency check and
ML prediction, to a doctor reviewing the resulting report. It mirrors the
behavior in `disease-prediction-be` (`Assessment.submit()`,
`Assessment.checkRedFlags()`, `MLModel.predict()`, `Report.send()`,
`Doctor.openReport()` / `markReviewed()`).

### Participants (pools)

1. **Patient** — starts the assessment, enters symptoms, waits for a response,
   reviews predictions, optionally forwards the report to a doctor.
2. **Disease Prediction System** — maps symptoms, checks red-flag rules,
   branches on emergency, runs the ML model, builds the explanation, persists
   the result and sends the report.
3. **Doctor** — receives the report, opens it, reviews it, adds clinical notes
   and marks it reviewed.

### BPMN elements used (for the complexity score)

- **3 pools** (participants) with **5 message flows** between them
  (submit assessment, emergency alert, prediction result, report sent,
  clinical review).
- **Message start events** (System and Doctor pools start on an incoming message).
- **Event-based gateway** in the Patient pool — the patient waits for *either*
  an emergency alert *or* a prediction result.
- **Exclusive gateways** — "Emergency detected?" (System) and
  "Forward to doctor?" (Patient).
- **Intermediate message catch events** — the patient receiving emergency,
  result and review messages.
- **Terminate end event** — the patient's emergency path stops the whole flow.
- **Normal + emergency end events** in each pool.
- Named **sequence flows** on gateway branches (Yes/No).

### Process logic

```
Patient:  start -> Start assessment -> Describe/add symptoms --(msg)-->
          [event-based gateway: await response]
             |-- emergency alert  -> (terminate: seek emergency care)
             |-- prediction result -> Review -> [forward to doctor?]
                                                    |- No  -> done
                                                    |- Yes -> Forward -> (await review) -> done

System:   (msg start) -> Map symptoms -> Check red-flag rules -> [emergency?]
             |- Yes -> Raise emergency alert --(msg)--> patient ; stop
             |- No  -> Run ML model -> Build explanation -> Persist result
                       -> Send report to doctor inbox --(msg)--> doctor ; done

Doctor:   (msg start) -> Open report -> Review -> Add notes & mark reviewed
                       --(msg)--> patient ; done
```

## How to open / view

The file is standard **BPMN 2.0**, so use any of:

- **bpmn.io demo** — go to https://demo.bpmn.io and drag the `.bpmn` file in
  (no install; renders instantly in the browser).
- **Camunda Modeler** (desktop) — https://camunda.com/download/modeler/ →
  `File ▸ Open` the `.bpmn`.
- **Eclipse BPMN2 Modeler** — install from the Eclipse Marketplace, then open
  the file in an Eclipse modeling workspace.
- **Visual Studio Code** — install the "BPMN Editor" / "bpmn.io" extension and
  open the file.

To export an image for a report: in bpmn.io or Camunda Modeler use
`Download / Export ▸ PNG` (or SVG).

## Validation performed

- Well-formed BPMN 2.0 XML.
- Every `sourceRef` / `targetRef` resolves to a declared element.
- Every diagram-interchange `bpmnElement` resolves to a model element.
- `incoming` / `outgoing` wiring is consistent for all sequence flows
  (the check BPMN tools apply on load).

## AI Usage

This BPMN model was produced with the assistance of **Kiro CLI**, an agentic AI
coding assistant.

- **Model/version:** Kiro CLI running in "Auto" mode (the underlying model is
  selected dynamically on the server, so a fixed version string is not exposed
  to the session). The assistant read the backend domain classes in
  `disease-prediction-be` to ground the process in the actual system behavior.

- **Prompt:** Generate a valid BPMN 2.0 XML file (including complete <bpmndi:BPMNDiagram> layout data) for a "Disease Prediction System". The collaboration requires 3 participant pools, sequence flows, and exact node names as follows: 
1. Patient Pool:
  Start ("Decides to check symptoms") -> Task ("Start assessment") -> Task ("Describe and add symptom entries") -> Event Gateway ("Await system response").

  Branch A: Message Catch ("Emergency alert received") -> Terminate End ("Seek emergency care").

  Branch B: Message Catch ("Prediction result received") -> Task ("Review predictions and disclaimer") -> Exclusive Gateway ("Forward to doctor?").

  "No" -> End ("Assessment complete").

  "Yes" -> Task ("Forward report to doctor") -> Message Catch ("Clinical review received") -> End ("Assessment complete").

  2. Disease Prediction System Pool:
  Message Start ("Assessment submitted") -> Task ("Map symptom entries to canonical symptoms") -> Task ("Check red-flag rules") -> Exclusive Gateway ("Emergency detected?").

  "Yes": Task ("Raise emergency alert") -> End ("Flow stopped (emergency)").

  "No": Task ("Run ML model prediction") -> Task ("Build ranked predictions and explanation") -> Task ("Persist prediction result and draft report") -> Task ("Send report to doctor inbox") -> End ("Result delivered").

  3. Doctor Pool:
  Message Start ("Report received in inbox") -> Task ("Open report") -> Task ("Review predictions and patient data") -> Task ("Add clinical notes and mark reviewed") -> End ("Review complete").

  Message Flows (Between Pools):

  Patient ("Describe and add symptom entries") -> System (Start Event)

  System ("Raise emergency alert") -> Patient ("Emergency alert received")

  System ("Persist prediction result...") -> Patient ("Prediction result received")

  System ("Send report to doctor inbox") -> Doctor (Start Event)

  Doctor ("Add clinical notes...") -> Patient ("Clinical review received")

  Ensure all processes are set to isExecutable="false", all elements are properly linked with <bpmn:sequenceFlow>, and complete DI bounding box/waypoint data is provided so it renders correctly in standard BPMN viewers. Output ONLY the raw XML code.

The AI generated the BPMN 2.0 XML, validated its structure. The modeling decisions were reviewed by the team.
