# Problem 2 — EMF Model of the Assessment Component

Team modeling deliverable (2 people). This folder contains an **Eclipse Modeling
Framework (EMF / Ecore)** metamodel of the **Assessment component** — the core
domain of the disease-prediction system — plus supporting artifacts.

All names are in English, as required.

## Files

| File | Purpose |
|------|---------|
| `model/diseaseprediction.ecore` | The Ecore **metamodel** (the model itself). |
| `model/diseaseprediction.genmodel` | EMF generator model — lets Eclipse generate Java model code, an edit plugin and an editor from the `.ecore`. |
| `model/sample-instance.xmi` | A **dynamic instance** conforming to the metamodel (one patient, one assessment with two symptom entries, a prediction result, a report routed to a doctor). Demonstrates the model is valid and usable. |

## What is modeled

The component modeled is the **Assessment aggregate** and everything it needs to
produce a prediction and a clinical report. It mirrors the Java domain in
`disease-prediction-be/src/main/java/com/diseaseprediction/domain`.

### Package
- `nsURI`: `http://www.diseaseprediction.com/model/1.0`
- `nsPrefix`: `dp`

### Classes (19) and 3 enumerations

**Users (inheritance hierarchy)**
- `User` *(abstract)* → `Patient`, `Doctor`, `Administrator`
- Enum `UserRole { PATIENT, DOCTOR, ADMIN }`

**Assessment aggregate (the focus)**
- `Assessment` — owns (composition) its `SymptomEntry` objects and one `PredictionResult`.
- `SymptomEntry` — a raw symptom report, optionally mapped to a canonical `Symptom`.
  Enum `Severity { MILD, MODERATE, SEVERE }`.

**Symptom taxonomy — Composite pattern**
- `SymptomComponent` *(interface)* with `Symptom` (leaf) and `SymptomGroup` (composite,
  recursively contains `SymptomComponent` children).
- `SymptomChecklist` holds the root `SymptomGroup`.

**Prediction result aggregate**
- `PredictionResult` — composes an ordered list of `DiseasePrediction`.
- `DiseasePrediction` — references a `Disease`, composes an `Explanation`.
- `Disease` — composes `Precaution` objects.

**Safety, reporting, ML**
- `RedFlagRule` — emergency detection against an `Assessment`.
- `Report` — references the `Assessment` and its recipient `Doctor`.
  Enum `ReportStatus { DRAFT, SENT, OPENED, REVIEWED }`.
- `MLModel` — produces a `PredictionResult` from symptom entries.

**Root container**
- `DiseasePredictionSystem` — document root that contains all top-level objects
  (needed so instance models have a single serializable root).

### Relationship kinds used (model complexity)

- **Inheritance**: `User` ← `Patient` / `Doctor` / `Administrator`;
  `SymptomComponent` ← `Symptom` / `SymptomGroup`.
- **Composition (containment)**: `Patient→Assessment`, `Assessment→SymptomEntry`,
  `Assessment→PredictionResult`, `PredictionResult→DiseasePrediction`,
  `Disease→Precaution`, `SymptomGroup→children` (recursive).
- **Association (non-containment reference)**: `SymptomEntry→Symptom`,
  `DiseasePrediction→Disease`, `Report→Assessment`.
- **Bidirectional references (eOpposite)**: `Patient↔Assessment`,
  `Assessment↔SymptomEntry`, `Assessment↔PredictionResult`, `Doctor↔Report`.
- **Multiplicities**: `1`, `0..1`, `0..*`, `1..*` are used throughout
  (e.g. `SymptomEntry.assessment` is `1..1`, `Assessment.symptomEntries` is `0..*`).
- **Operations**: key behaviors (`submit`, `checkRedFlags`, `predict`,
  `mapToSymptom`, `getTopPredictions`, …) are modeled as `EOperation`s with
  typed parameters and return types.

## How to open / regenerate in Eclipse

1. Install **Eclipse Modeling Tools** (or add the EMF SDK to any Eclipse).
2. `File ▸ Import ▸ Existing Projects` or just copy the `model/` folder into a project.
3. Double-click `diseaseprediction.ecore` to inspect the metamodel in the tree editor.
   - Right-click the package ▸ *Initialize Ecore Diagram* to get a class diagram.
4. Open `diseaseprediction.genmodel`, right-click the root ▸ **Generate Model Code**
   (and optionally *Edit Code* / *Editor Code*) to produce Java EMF classes.
5. To test instances: with the generated editor, `File ▸ New ▸ Other ▸ Example EMF
   Model Creation Wizards`, pick `DiseasePredictionSystem` as the root. The provided
   `sample-instance.xmi` can be opened directly to validate the metamodel.

### Validating the sample instance
Right-click `sample-instance.xmi` ▸ **Validate** in the generated editor — it should
report no errors, confirming the instance conforms to the metamodel.

## Mapping to the implementation

The metamodel is a faithful abstraction of the backend Java domain classes
(same class names, attributes, enums and the Composite pattern for symptoms),
so the EMF model and the running system stay aligned.
