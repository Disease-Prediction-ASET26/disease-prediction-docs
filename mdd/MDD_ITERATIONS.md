# Model Driven Development — Iterations (Lab05, Problem 1)

This document records our use of Model Driven Development (AMDD) on the Disease Prediction
project. Lab05 requires at least two MDD/TDD iterations, graded on the correct identification of
the steps and their execution. We use MDD because the project has been model-first since Lab02
(requirements → UML/C4 → design patterns → code), so our code is genuinely derived from models.

Repositories (SOA — separate services):

- `disease-prediction-be` — Java / Spring Boot backend
- `disease-prediction-ai` — Python prediction service (the ML part)

## The MDD iteration cycle (six steps)

Each iteration below executes the same six steps:

1. **Requirements envisioning** — pick the functional slice, anchor it to a real requirement
   (FR / use case).
2. **Architecture / platform envisioning** — decide where the slice lives in the SOA architecture
   (which service, which layer, how it communicates).
3. **Model the slice** — evolve the model (class and/or behavioural diagram) for just this slice;
   record what changed from the baseline model and why. This is the artifact the iteration is
   driven from.
4. **Model-to-code transformation** — derive code from the model, element by element, keeping
   traceability (model element → code element).
5. **Validate / test** — compile and run; confirm the code matches the model and behaves as
   modelled.
6. **Review & refine the model** — fold back what implementation taught us; the refined model
   becomes the input to the next iteration.

## Iteration 0 — Baseline (not counted)

Lab02–Lab04 produced the baseline model: the shared class diagram
([class diagram](../design/diagrams/uml/class_diagram_daria.png), [AI pattern diagram](../design/diagrams/patterns/patterns_class_diagram_ai.png)), the C4 diagrams, and
the declaration-only domain + design-pattern skeletons (Strategy + Factory in the Python service).
We treat this as Iteration 0: a model-first foundation, but not framed with explicit MDD steps.
The two iterations below build on it.

## Iteration 1 — Free-Text Symptom Parsing (Python prediction service)

Turns a patient's free-text symptom description into structured, mapped symptom data (a
severity-weighted feature vector + `SymptomEntryDTO` list).

### Step 1 — Requirements envisioning

- Slice: free text → structured, mapped symptoms.
- Anchored to: FR-P1 ("describe symptoms via free text or a structured checklist") and the agreed
  behaviour "Free-text symptoms are mapped to known Symptoms; unmapped text is kept and shown to
  the doctor." (see `../requirments.md`).

### Step 2 — Architecture / platform envisioning (SOA)

The slice lives entirely in the Prediction Service (Python), not the Java backend, honouring the
SOA split. It is the front stage of the service's pipeline: raw text enters, a severity-weighted
feature vector and a list of `SymptomEntryDTO` leave, ready for the classifier stage (Iteration 2).
NLP libraries (transformers, spaCy/negspacy, sentence-transformers, rapidfuzz) sit behind
interfaces so they can be swapped.

### Step 3 — Model the slice (core MDD step)

Baseline model: `SymptomEntry`, `Symptom`, `Severity` existed only as passive data. There was no
model for how free text becomes a `SymptomEntry`.

Model evolution this iteration: introduce a parsing pipeline — a chain of four stages, each
modelled as a role (interface) with a concrete realization. This is a direct application of the
Strategy pattern from our design model.

```mermaid
classDiagram
    class SymptomParser {
        +parse(text) ParseResult
    }
    class EntityExtractor {
        <<interface>>
        +extract(text) ExtractedEntity[]
    }
    class NegationDetector {
        <<interface>>
        +annotate(text, entities) ExtractedEntity[]
    }
    class EntityLinker {
        <<interface>>
        +link(entities) LinkedSymptom[]
    }
    class ParseResult {
        +raw_text
        +feature_vector int[]
        +binary_vector() int[]
        +to_symptom_entries() SymptomEntryDTO[]
    }
    class ExtractedEntity {
        +text
        +is_negated bool
    }
    class LinkedSymptom {
        +canonical_symptom
        +similarity
        +severity_weight int
    }

    SymptomParser --> EntityExtractor : uses
    SymptomParser --> NegationDetector : uses
    SymptomParser --> EntityLinker : uses
    SymptomParser --> ParseResult : produces
    EntityExtractor ..> ExtractedEntity : creates
    EntityLinker ..> LinkedSymptom : creates
    ParseResult ..> SymptomEntryDTO : maps to
```

Behavioural model (pipeline flow):

```mermaid
flowchart LR
    A[raw free text] --> B[Extractor: find problem entities]
    B --> C[NegationDetector: mark negated]
    C --> D{affirmed?}
    D -->|yes| E[Linker: map to canonical symptom]
    D -->|no| F[kept as negated, excluded from vector]
    E --> G[severity-weighted feature vector]
    G --> H[SymptomEntryDTO list]
```

What changed from baseline and why: the baseline had no transformation from text → `SymptomEntry`.
We added the pipeline abstraction (Extractor → NegationDetector → Linker → vectorization) because
the requirement demands mapping free text to known symptoms while keeping unmapped / negated text.
Modelling each stage as an interface lets us swap NLP implementations without touching the pipeline.

### Step 4 — Model-to-code transformation

Each model element maps one-to-one to code on branch `feature/nlp-pipeline`:

| Model element | Code element | File |
| --- | --- | --- |
| `SymptomParser` (orchestrator) | `class SymptomParser.parse()` | `prediction_service/parser.py` |
| `EntityExtractor` (interface) | `Protocol EntityExtractor` + `ClinicalBertExtractor` | `prediction_service/parser.py` |
| `NegationDetector` (interface) | `Protocol NegationDetector` + `NegspacyNegationDetector`, `KeywordNegationDetector` | `prediction_service/parser.py` |
| `EntityLinker` (interface) | `Protocol EntityLinker` + `SemanticEntityLinker` (semantic → fuzzy fallback) | `prediction_service/parser.py` |
| `ExtractedEntity`, `LinkedSymptom`, `ParseResult` | dataclasses | `prediction_service/parser.py` |
| canonical symptom vocabulary | `CANONICAL_SYMPTOMS` + `normalise_symptom()` | `prediction_service/vocabulary.py` |
| severity weighting | `weight_for()`, `SEVERITY_WEIGHTS` | `prediction_service/severity.py` |
| output DTO + `Severity` | `ParseResult.to_symptom_entries()` → `SymptomEntryDTO` | `prediction_service/models.py` |

Every interface and dataclass in the model has a named counterpart in the code.

### Step 5 — Validate / test

The slice is validated by the parser test suite in `tests/test_parser.py`, which checks exactly
the modelled behaviour: happy-path weighted vector, negation filtered before linking, empty /
no-entity / only-negated → zero vector, severity bucketing, semantic-above-threshold vs fuzzy
fallback, and dedup keeping the best match.

Verified run (Python 3.12.10, Windows):

```
40 passed in 0.26s
```

All 40 tests pass locally — including the fuzzy-fallback linking path (`rapidfuzz`). There are no
environment failures. The same pinned stack runs in CI on Python 3.12 (see
`.github/workflows/ci.yml`, the `lint` / `build` jobs with `python-version: '3.12'`), so the local
and CI results agree. Because the heavy ML libraries are lazy-imported behind the interfaces, the
suite also runs with only pytest, numpy and the pure-Python modules present.

> Correction note: an earlier draft of this document reported "38 passed, 2 failed in 0.08s" on
> "Python 3.15, macOS", attributing the two failures to a missing `rapidfuzz` wheel on Python 3.15,
> and said CI ran on Python 3.13. That result was not reproducible: the suite is 40 tests and all
> pass on Python 3.12.10, and CI is pinned to Python 3.12. The numbers above are the verified result.

### Step 6 — Review & refine the model

Implementation surfaced three things the baseline class diagram did not capture; they are folded
back into the model (and carried into Iteration 2):

1. **Negation is a dedicated stage, not a flag on a symptom.** Affirmed and negated entities must
   be split before linking, so negated symptoms never reach the feature vector.
2. **Linking needs a two-tier strategy** — semantic match first, fuzzy match when the semantic
   model is absent or below threshold. Modelled now as two strategies behind `EntityLinker`, not a
   single `link()`.
3. **Severity is a weight (int), not only the enum.** The feature vector carries severity-weighted
   values; the `Severity` enum is derived from the weight at the end. `LinkedSymptom.severity_weight`
   is now explicit in the model.

The refined model's output (`ParseResult.feature_vector` / `to_symptom_entries()`) defines what the
prediction service consumes, which the Iteration 2 API contract (Java backend → prediction service)
is modelled against.

## Iteration 2 — Assessment API (Java backend)

Status: implemented. Builds directly on Iteration 1 across the SOA boundary: the Java backend
exposes the assessment endpoint and (per the design) delegates prediction to the Python prediction
service, which consumes the parser's output. The backend web layer was net-new this iteration.

### Step 1 — Requirements envisioning

- Slice: a patient submits an assessment (free text + symptom entries) through the backend and
  receives a prediction result.
- Anchored to: UC1 (Patient Triage and Direct Specialist Routing), FR-P1 (describe symptoms via
  free text / checklist) and FR-P2 (list of likely diseases with a calibrated probability score).
  See `../requirments.md`.

### Step 2 — Architecture / platform envisioning (SOA)

The slice lives in the API Backend (Java / Spring Boot) — the web layer. The backend does not
predict; it orchestrates. It is the client of the Python prediction service across the SOA boundary:

```
Patient → POST /api/assessments (Java backend) → calls → POST /predict (Python service)
       → PredictionResult → backend → Patient
```

The API contract (request/response schema) is the shared model that both sides are derived from.

### Step 3 — Model the slice (core MDD step)

Baseline model: the domain exists as declaration-only skeletons (`Assessment`, `SymptomEntry`,
`PredictionResult`, `DiseasePrediction`, `Patient`) with empty methods; there is no web layer and no
API contract modelling how an assessment is submitted or how the backend reaches the prediction
service.

Model evolution this iteration: introduce the Assessment API contract and the controller/client
structure that realises it. (Rendered diagrams: `mdd/iteration2/` in this repository.)

API contract (the platform-independent model) — `POST /api/assessments`:

Request:

```json
{
  "patientId": "uuid",
  "freeText": "I have a bad cough and high fever for 3 days",
  "symptomEntries": [
    { "rawText": "cough", "severity": "MODERATE", "durationDays": 3, "isMapped": true }
  ]
}
```

Response (200):

```json
{
  "assessmentId": "uuid",
  "emergencyDetected": false,
  "result": {
    "resultId": "uuid",
    "generatedAt": "2026-10-31T12:00:00Z",
    "disclaimer": "This is not a medical diagnosis",
    "predictions": [
      {
        "rank": 1,
        "probability": 0.82,
        "diseaseName": "Common Cold",
        "explanation": { "summary": "...", "contributionWeights": { "cough": 0.5 } }
      }
    ]
  }
}
```

400 — invalid / empty symptom payload.

Structural model (web layer derived from the contract):

```mermaid
classDiagram
    class AssessmentController {
        +submit(AssessmentRequest) AssessmentResponse
    }
    class AssessmentApiService {
        +runAssessment(AssessmentRequest) AssessmentResponse
    }
    class PredictionClient {
        <<interface>>
        +predict(freeText, symptomEntries) PredictionResultDto
    }
    class AssessmentRequest {
        +patientId
        +freeText
        +symptomEntries
    }
    class AssessmentResponse {
        +assessmentId
        +emergencyDetected
        +result
    }

    AssessmentController --> AssessmentApiService : uses
    AssessmentController ..> AssessmentRequest : receives
    AssessmentController ..> AssessmentResponse : returns
    AssessmentApiService --> PredictionClient : delegates prediction
    PredictionClient ..> PredictionResultDto : returns
```

What changed from baseline and why: the baseline had domain objects but no way to invoke them over
HTTP. We add the API contract (the model of the service boundary) plus the controller → service →
client structure. Modelling `PredictionClient` as an interface keeps the SOA call behind an
abstraction, so the backend can be built and tested against the contract whether or not the live
Python endpoint exists yet.

### Step 4 — Model-to-code transformation

Each model element maps to Java code under `src/main/java/com/diseaseprediction/`:

| Model element | Code element | Location |
| --- | --- | --- |
| `POST /api/assessments` + `AssessmentController` | `@RestController AssessmentController` | `controller/` |
| `AssessmentRequest` / `AssessmentResponse` | request/response DTO records | `controller/dto/` |
| `PredictionResultDto`, `DiseasePredictionDto`, `ExplanationDto`, `SymptomEntryDto` | DTO records | `controller/dto/` |
| `AssessmentService.runAssessment()` | `@Service AssessmentApiService` | `service/assessment/` |
| `PredictionClient` (interface) | `PredictionClient` + `StubPredictionClient` | `service/prediction/` |

Naming note: the orchestrator is `AssessmentApiService` (not `AssessmentService`) because a
pre-existing domain-coupled `AssessmentService` already exists; Iteration 2 is net-new web-layer
code and deliberately does not rewrite it. DTOs are immutable records so the web slice serialises
cleanly without touching the declaration-only domain classes.

### Step 5 — Validate / test

`@WebMvcTest AssessmentController` with a mocked `PredictionClient`
(`src/test/java/com/diseaseprediction/controller/AssessmentControllerTest.java`):

- `validRequestReturns200WithPrediction` — valid payload → 200 with a `PredictionResultDto` body.
- `emptyRequestReturns400` — blank free text + empty entries → 400.
- `chestPainIsFlaggedAsEmergency` — red-flag text → `emergencyDetected = true`.

Verified run:

```
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
BUILD SUCCESS
```

(3 controller tests + the existing `contextLoads`.) Verified on JDK 21 (JetBrains Runtime) with
`-Dmaven.compiler.release=21`; the `pom.xml` pins `java.version=25` and CI uses that toolchain.

### Step 6 — Review & refine the model

- **Emergency check runs before prediction** — `AssessmentApiService.runAssessment()` evaluates the
  red-flag check before calling the prediction client, so an emergency surfaces even if prediction
  fails.
- **Prediction behind an interface** — `PredictionClient` + `StubPredictionClient`; the Step 6
  follow-up is an `HttpPredictionClient` (Spring `RestClient`) that calls the Python `POST /predict`.
- Open items for the next refinement: prediction-service timeout/fallback handling, and whether
  `GET /api/assessments/{id}` is needed to retrieve a stored result.

## Iteration 3 — Patient Web App: Symptom Prediction Feature (Frontend construction)

Status: **planned** (not yet implemented). Builds on Iteration 2 across the SOA boundary: a Patient
Web App (React) that accepts a free-text NLP symptom description **and** a structured checklist,
submits to the Java backend's `POST /api/assessments`, and renders the prediction result (or the
FR-P5 emergency alert). This iteration constructs the app scaffold and the single assessment feature
(one page); it does **not** add navigation between pages — that is Iteration 4.

- Slice anchored to FR-P1 (free text + checklist), FR-P2/FR-P3 (ranked predictions + explanation +
  disclaimer), FR-P5 / UC2 (emergency intercept), UC1 (patient triage flow).
- Lives entirely in the "Patient Web App (React)" C4 container; a pure HTTP/JSON client of the
  backend, with the view → hook → `AssessmentApiClient` (port/adapter) → types layering deliberately
  mirroring the backend's controller → service → `PredictionClient` structure.

The full six-step plan — context the implementing agent must load, the authoritative API contract,
structural/behavioural models, the model-to-code file map, CI/GitFlow constraints, and the
definition of done — is in **`MDD_ITERATIONS_FE_PLAN.md`, Part 1**.

## Iteration 4 — Patient Web App: Client-side Routing

Status: **planned** (not yet implemented, depends on Iteration 3). Adds client-side **routing** to
the Patient Web App built in Iteration 3: distinct URL-addressable views (home/landing, the
assessment page, a dedicated result view, an emergency view, and a not-found fallback) with React
Router, so the single-page feature becomes a navigable multi-route app and the result/emergency
states become shareable/bookmarkable routes.

- Anchored to UC1 (navigable triage flow) and the structural need to add FR-P4 (specialty routing)
  and later portals (FR-D1) as new routes without reworking the feature components.
- Still entirely within the "Patient Web App (React)" container; introduces a routing/navigation
  layer **above** the Iteration 3 feature components, which are reused unchanged.

The full six-step plan — route model, the router/layout structure derived from it, the
model-to-code map, and routing-specific tests (navigation, deep-linking, guards) — is in
**`MDD_ITERATIONS_FE_PLAN.md`, Part 2**.

## Iteration 5 — User and Consent Persistence (Java backend)

Status: **implemented** (branch `feat/user-persistence`, PR into `develop`; 21 tests pass). First
step of the "secured before stored" plan: users and consent are persisted before any health data,
so that assessments (Iteration 7) land in a schema that already has identity, consent, cascade
deletion and encrypted personal data.

### Step 1 — Requirements envisioning

- Slice: persist patients, doctors, administrators and the patient's consent history.
- Anchored to: NFR-1 (GDPR Article 9, health data is a special category), FR-G1 (grant and revoke
  consent), FR-G2 / UC3 (delete the account and all data), UC1 step 1 (consent before symptom
  input).

### Step 2 — Architecture / platform envisioning

- **Where:** the API Backend (Java / Spring Boot), new persistence layer. The Python prediction
  service is not involved.
- **Platform:** PostgreSQL 18, run locally with Docker Compose (`compose.yaml`, port 5433);
  Spring Data JPA; Flyway for versioned migrations.
- **Schema ownership:** Flyway migrations are the source of truth for the schema. Hibernate runs
  with `ddl-auto=validate`, so it checks the entities against the tables but never changes them.
  Letting Hibernate generate the schema was rejected because it is not versioned or reviewable.
- **Relational, not MongoDB:** an assessment would fit a document store, but the wider domain is
  relational (users, doctors, reports, consent), and the GDPR requirements need foreign keys with
  cascade deletes and auditable history. MongoDB also has no Flyway equivalent of the same maturity.
- **Identity:** login and passwords are delegated to Keycloak in Iteration 6. This iteration only
  adds the `identity_id` column that links a user row to the Keycloak account.
- **Domain classes are the JPA entities:** a separate persistence model with mappers (hexagonal
  style) was considered and rejected, because it doubles the classes for little benefit at this
  size and weakens the one-to-one model-to-code traceability.
- **Encryption:** personal data (`full_name`, `date_of_birth`) is encrypted by the application
  with AES-256-GCM (field-level encryption). Disk-level encryption at rest and TLS to the database
  need a deployed database, so they belong to the deployment work with the GitOps and infra
  owners.

The model (ER, class and baseline-vs-refined diagrams), the reasons for each change, the
model-to-code mapping, the test results and the review (Steps 3–6) are in
**`iteration5/README.md`**.

## Iteration 6 — Identity and Login with Hanko (Java backend + React)

Status: **designed** (implementation on one branch per repository). Second step of the "secured
before stored" plan: every API request carries a Hanko session token, the backend verifies it
and maps it to the Iteration 5 user, and a first-login onboarding step creates the patient
profile and records consent.

### Step 1 — Requirements envisioning

- Slice: register, log in, log out, first-login onboarding, consent grant and revoke, server-side
  protection of the assessment API. Every `/api/**` endpoint requires login.
- Anchored to: NFR-1 (GDPR Article 9), FR-G1 (consent), UC1 step 1 (consent before symptom
  input), the domain rule "users must log in".

### Step 2 — Architecture / platform envisioning

- **Identity provider:** Hanko v3.1.0, already deployed by DevOps on K3s (dev and production
  instances), replaces the Keycloak plan from Iteration 5 and the roadmap. `identity_id` holds the
  Hanko user id (the token's `sub`); the schema does not change.
- **Session transport:** `Authorization: Bearer <Hanko session JWT>`. No refresh tokens; the
  session lifespan goes from 15 minutes to 1 hour.
- **Backend:** Spring Security OAuth2 resource server (RS256, JWKS, issuer, audience) plus a
  cached `POST /sessions/validate` check for logout and revocation, failing closed.
- **Roles:** stay in the app database (the Iteration 5 discriminator). New users are patients.
- **Frontend:** Hanko Elements (`<hanko-auth>`), an auth context, route guards and an onboarding
  page.
- **Local development:** Hanko, its database and a mail catcher in the backend's Docker Compose.

The model (sequence, class and baseline-vs-refined diagrams), the API contract changes, the
model-to-code plan, the test plan, and the design of the AOP component (Part B, Personal Data
Access Guard) are in **`iteration6/README.md`**.

> The four iterations below are written in ASD Simplified Technical English (STE): short
> sentences, active voice, present tense, and one instruction in each sentence. Iteration 7
> (Health-data persistence, Java backend) stays a planned roadmap item. It is not yet done.
> Iteration 9b comes after Iteration 9 so that it does not use the Iteration 7 number.
> Iteration 9c (red-flag urgency, the final iteration) follows the same rule.

## Iteration 8 — Prediction Service: Web Layer (Python)

Status: **done**. This iteration puts an HTTP layer in front of the Iteration 1 pipeline. The
Java backend can then call a real service. Before this iteration, the backend used a stub.

### Step 1 — Requirements envisioning

- Slice: show the pipeline on HTTP. The service predicts diseases. It maps free text to symptoms.
  It checks red flags. It gives disease information.
- Anchored to FR-P1 and FR-P2 (predict diseases from symptoms) and the SOA rule. The backend must
  not run ML code. The backend calls the Python service.

### Step 2 — Architecture / platform envisioning (SOA)

- The web layer stays in the Prediction Service (Python). It does not go into the backend.
- Use FastAPI. FastAPI gives input validation, dependency injection and an OpenAPI contract.
- Inject the pipeline, the red-flag engine, the knowledge base and the specialty mapper with
  `Depends`. Cache each one with `lru_cache`. The model loads one time for each process.

### Endpoints

| Operation | Function | Success | Error |
| --- | --- | --- | --- |
| `POST /predict` | Free text to ranked diseases, with descriptions, precautions, red flags and specialty | `200` | `422` if the text is empty |
| `POST /map-symptoms` | Free text to structured symptom entries | `200` | `422` |
| `POST /red-flags/check` | Active symptoms to red-flag rules and urgency | `200` | `422` |
| `GET /diseases/{name}` | Description and precautions for one disease | `200` | `404` if the disease is unknown |
| `GET /health`, `GET /ready` | Liveness and readiness for the container probes | `200` / `503` | — |

### Step 3 — Model the slice

**Structural model**

```mermaid
classDiagram
    class FastAPIApp {
        +POST /predict
        +POST /predict/followup
        +POST /map-symptoms
        +POST /red-flags/check
        +GET /diseases/{name}
        +GET /health
        +GET /ready
    }
    class PredictRequest {
        +str text
    }
    class PredictResponse {
        +PredictionItem[] predictions
        +RedFlag[] redFlags
        +Urgency urgency
        +bool needs_follow_up
        +FollowUp follow_up
    }
    class PredictionPipeline {
        +predict(text) PredictionResult
        +classifier
    }
    class RedFlagEngine {
        +evaluate(active) RedFlag[]
    }
    class DiseaseKnowledgeBase {
        +describe(disease)
        +precautions(disease)
    }
    class SpecialtyMapper {
        +specialty_for(disease)
    }

    FastAPIApp --> PredictRequest : validates
    FastAPIApp --> PredictResponse : returns
    FastAPIApp --> PredictionPipeline : get_pipeline (Depends)
    FastAPIApp --> RedFlagEngine : get_red_flag_engine (Depends)
    FastAPIApp --> DiseaseKnowledgeBase : get_knowledge (Depends)
    FastAPIApp --> SpecialtyMapper : get_specialty_mapper (Depends)
```

**Baseline vs refined**

```mermaid
flowchart TB
    subgraph BEFORE["Baseline (after Iteration 1)"]
        direction TB
        B1["SymptomParser and pipeline are Python classes"]
        B2["No HTTP surface. The backend uses a stub"]:::problem
    end
    subgraph AFTER["Refined (Iteration 8)"]
        direction TB
        A1["FastAPI app wraps the pipeline"]
        A2["POST /predict, /map-symptoms,<br/>/red-flags/check, GET /diseases/{name}"]
        A3["pydantic validation, 422 on bad input"]
        A4["Dependencies injected with Depends,<br/>cached with lru_cache"]
        A1 --> A2 --> A3 --> A4
    end
    BEFORE ==>|"show the pipeline on HTTP so the backend calls a real service"| AFTER
    classDef problem fill:#ffe0e0,stroke:#c0392b,color:#7b1e14;
```

The baseline from Iteration 1 was a pipeline of Python classes. It had no network surface. The
backend could use only a stub. Iteration 8 adds a FastAPI app. The app wraps the pipeline and does
not change it. The app adds validation and container probes.

### Step 4 — Model-to-code transformation

| Model element | Code element | File |
| --- | --- | --- |
| FastAPI app and routes | `app`, `predict`, `map_symptoms`, `red_flags_check`, `disease_info`, `health`, `ready` | `prediction_service/api.py` |
| Request and response models | `PredictRequest`, `PredictResponse`, `MapSymptomsResponse`, `RedFlagCheckResponse`, `DiseaseInfoResponse` | `prediction_service/api.py` |
| Pipeline dependency | `get_pipeline()`, env `DISEASE_MODEL_PATH` | `prediction_service/api.py`, `pipeline.py` |
| Red-flag engine | `RedFlagEngine`, `RedFlagRule`, `Urgency` | `prediction_service/red_flags.py` |
| Disease knowledge | `DiseaseKnowledgeBase` | `prediction_service/knowledge.py` |
| Specialty mapping (FR-P4) | `SpecialtyMapper` | `prediction_service/specialty.py` |

### Step 5 — Validate / test

The test suite passes. It has 166 tests on Python 3.12.10. The API tests (`test_api.py`) have 23
tests. The red-flag tests (`test_red_flags.py`) have 7 tests. The knowledge tests
(`test_knowledge.py`) have 6 tests. The API tests call each endpoint with the FastAPI `TestClient`.

### Step 6 — Review & refine the model

1. The dependencies are cached, not global. `lru_cache` loads the model one time for each process.
   `Depends` keeps the endpoints easy to test.
2. There are two probes. `/health` is liveness. `/ready` reports if the model is loaded.
3. The red-flag check is deterministic. It runs next to the prediction. It is not a diagnosis.
4. The follow-up endpoint is added later (Iteration 9b). It reuses this app. For this reason,
   `PredictResponse` already carries `needs_follow_up` and `follow_up`.

## Iteration 9 — ML Pipeline: Trained Classifier (Python)

Status: **done**. This iteration fills the Iteration 0 Strategy and Factory skeleton. It adds
trained, calibrated classifier models. The `/predict` endpoint now gives real ranked diseases.
Before this iteration, the pipeline gave a stub result.

### Step 1 — Requirements envisioning

- Slice: train classifiers on the Kaggle dataset. Calibrate the best model. Save it. Serve ranked
  diseases with explanations and a specialty.
- Anchored to FR-P2 (ranked probabilities), FR-P4 (specialty routing) and NFR-2 (anonymise the
  training data).

### Step 2 — Architecture / platform envisioning (SOA)

- All of this stays in the Prediction Service (Python).
- Keep the Iteration 0 patterns. Strategy holds the interchangeable classifier algorithms.
  Factory builds each strategy.
- Train the models offline in `train.py`. The API loads a saved model at request time. The heavy
  training does not run in the request path.

### Classifier strategies

| Strategy | Estimator |
| --- | --- |
| `RandomForestStrategy` | random forest |
| `KNNStrategy` | k-nearest neighbours |
| `DecisionTreeStrategy` | decision tree |
| `NaiveBayesStrategy` | naive Bayes |
| `CalibratedStrategy` | wraps the best strategy in `CalibratedClassifierCV` |

### Step 3 — Model the slice

**Structural model**

```mermaid
classDiagram
    class ClassifierStrategy {
        +classify(entries) DiseasePrediction[]
    }
    class SklearnStrategy {
        +fit(features, labels)
        +with_explanations(gen) SklearnStrategy
    }
    class RandomForestStrategy
    class KNNStrategy
    class DecisionTreeStrategy
    class NaiveBayesStrategy
    class CalibratedStrategy
    class DiseaseClassifier {
        +predict(entries) DiseasePrediction[]
    }
    class ModelFactory {
        +create_classifier() ClassifierStrategy
    }
    class ExplanationGenerator {
        +explain(vector, estimator, class) Explanation
    }
    class SpecialtyMapper {
        +specialty_for(disease) str
    }
    class ModelStore {
        +save(strategy, metadata) version
        +load_active() SklearnStrategy
    }

    SklearnStrategy ..|> ClassifierStrategy
    RandomForestStrategy --|> SklearnStrategy
    KNNStrategy --|> SklearnStrategy
    DecisionTreeStrategy --|> SklearnStrategy
    NaiveBayesStrategy --|> SklearnStrategy
    CalibratedStrategy --|> SklearnStrategy
    DiseaseClassifier --> ClassifierStrategy : wraps
    ModelFactory ..> ClassifierStrategy : creates
    SklearnStrategy --> ExplanationGenerator : optional
    ModelStore ..> SklearnStrategy : persists (joblib)
```

**Training flow**

```mermaid
flowchart TD
    DS["dataset.csv (Kaggle)"] --> AN["anonymize: scrub ids and free text (NFR-2)"]
    AN --> FE["encode_entries: severity-weighted vectors"]
    FE --> TR["train.py: fit every strategy"]
    TR --> BEST["pick the best strategy"]
    BEST --> CAL["CalibratedStrategy: CalibratedClassifierCV"]
    CAL --> SAVE["save best.joblib and a manifest"]
    SAVE --> STORE["ModelStore: version, metadata, mark active"]
    STORE --> LOAD["API loads the active model"]
    LOAD --> CLF["DiseaseClassifier.predict(entries)"]
    CLF --> EXP["ExplanationGenerator: per-symptom weights"]
```

**Baseline vs refined**

```mermaid
flowchart TB
    subgraph BEFORE["Baseline (Iteration 0 skeleton)"]
        direction TB
        B1["Strategy and Factory are declarations"]:::missing
        B2["The pipeline returns a stub result"]:::problem
    end
    subgraph AFTER["Refined (Iteration 9)"]
        direction TB
        A1["Concrete strategies: RandomForest, KNN,<br/>DecisionTree, NaiveBayes"]
        A2["ModelFactory for each strategy"]
        A3["CalibratedStrategy for correct probabilities"]
        A4["ModelStore, joblib persistence, manifest"]
        A5["ExplanationGenerator, SpecialtyMapper, anonymize"]
        A1 --> A2 --> A3 --> A4 --> A5
    end
    BEFORE ==>|"fill the skeleton with trained, calibrated, explainable models"| AFTER
    classDef missing stroke-dasharray: 5 5,color:#999,stroke:#999;
    classDef problem fill:#ffe0e0,stroke:#c0392b,color:#7b1e14;
```

Iteration 0 left Strategy and Factory as declarations with a stub. Iteration 9 makes them real. It
adds concrete strategies, a factory for each strategy, a calibration wrapper, a model registry with
provenance, explanations, and the FR-P4 specialty mapper. It anonymises the training data first.

### Step 4 — Model-to-code transformation

| Model element | Code element | File |
| --- | --- | --- |
| Strategy interface and base | `ClassifierStrategy`, `SklearnStrategy` | `prediction_service/classifier.py` |
| Concrete strategies | `RandomForestStrategy`, `KNNStrategy`, `DecisionTreeStrategy`, `NaiveBayesStrategy` | `prediction_service/classifier.py` |
| Calibration | `CalibratedStrategy` (`CalibratedClassifierCV`) | `prediction_service/classifier.py` |
| Factory | `ModelFactory` and the four concrete factories | `prediction_service/factory.py` |
| Classifier wrapper | `DiseaseClassifier` | `prediction_service/classifier.py` |
| Feature encoding | `encode_entries` | `prediction_service/features.py` |
| Explanations | `ExplanationGenerator`, `Explanation` | `prediction_service/explanation.py` |
| Specialty mapping (FR-P4) | `SpecialtyMapper` | `prediction_service/specialty.py` |
| Model registry and persistence | `ModelStore`, `save_strategy`, `load_strategy` | `prediction_service/persistence.py` |
| Provenance manifest | `write_model_manifest`, `verify_model_artifact` | `prediction_service/artifact.py` |
| Training script | `train.py` | `prediction_service/train.py` |
| Anonymisation (NFR-2) | `anonymize_entries`, `scrub_text`, `pseudonymize_id` | `prediction_service/anonymize.py` |

### Step 5 — Validate / test

The test suite passes. It has 166 tests. The classifier tests have 20 tests. The calibration tests
have 8 tests. The explanation tests have 8 tests. The specialty tests have 6 tests. The persistence
tests have 11 tests. The training tests have 6 tests. The anonymise tests have 12 tests.

### Step 6 — Review & refine the model

1. Calibration is a strategy, not a special case. `CalibratedStrategy` subclasses `SklearnStrategy`.
   The API loads it and calls it like any other strategy.
2. The project uses `joblib`, not `pickle`. scikit-learn models hold large NumPy arrays. `joblib`
   saves them well. The loader still reads the old `.pkl` files.
3. The manifest travels with the model. It records the artifact hash, the data hash, the strategy,
   the scores and the runtime.
4. The explanation code unwraps the calibration wrapper. It then reads the feature importances or
   the coefficients. Only the active symptoms can contribute.
5. The project anonymises the data before training (NFR-2). It hashes the `entry_id`. It scrubs the
   free text.

## Iteration 9b — Adaptive Follow-up Questions (AI + backend + React, FR-P3)

Status: **done**. This iteration adds a follow-up loop across the three services. When a
prediction is not confident, the service asks the patient one yes/no question. The service adds
the answer and predicts again. This iteration comes after Iteration 9. It does not use the
Iteration 7 number.

### Step 1 — Requirements envisioning

- Slice: for a prediction that is not confident, ask one yes/no question. The question is about the
  symptom that best separates the top disease from the next disease. Add the answer. Predict again.
  Stop when the prediction is confident.
- Anchored to FR-P3 (follow-up questions to refine a prediction).
- Out of scope: free-text answers. The service accepts only yes or no. The server keeps no session
  state. The client carries the symptom sets between turns.

### Step 2 — Architecture / platform envisioning (SOA)

The feature uses all three services. Each service keeps its role:

- **AI (Python):** selects the question. Decides when to stop. It keeps no state. The caller sends
  the confirmed and denied symptoms for each turn.
- **Backend (Java):** adds one endpoint with the consent check. It relays the loop to the AI
  service. It adds no ML logic.
- **Frontend (React):** runs the loop in the UI. It collects the symptom sets across the turns. It
  gives only the final confident result to the page.

### How the loop works

1. A normal prediction runs (`POST /predict`).
2. The top disease is below the confidence threshold (default 0.80). The AI selects the symptom
   that is frequent in the top disease and rare in the next disease. It returns this symptom as a
   yes/no question.
3. The patient answers. The service adds the symptom to the confirmed set or the denied set.
4. `POST /predict/followup` adds the answer and predicts again. It reuses the trained model. It
   does not run the NLP stage again.
5. The loop stops for one of three reasons: the confidence reaches the threshold; no useful
   question remains; the loop reaches the round limit. The round limit makes sure that the loop
   always stops.

**Structural model**

```mermaid
classDiagram
    namespace PredictionService_Python {
        class SymptomAssociationIndex {
            +most_discriminating(top, runner_up, asked) str
        }
        class FollowUpService {
            +continue_loop(confirmed, denied, answer) PredictResponse
        }
        class FastAPIApp {
            +POST /predict
            +POST /predict/followup
        }
    }
    namespace Backend_Java {
        class AssessmentController {
            +submitFollowup(FollowupRequest) AssessmentResponse
        }
        class AssessmentApiService {
            +continueAssessment(FollowupRequest) AssessmentResponse
        }
        class PredictionClient {
            +predictFollowUp(confirmed, denied, rounds) PredictionResult
        }
        class ProfileService {
            +requireConsentingPatient() Patient
        }
    }
    namespace Frontend_React {
        class AssessmentPage
        class FollowUpQuestion {
            +yes / no
        }
        class useAssessment {
            +answerFollowUp(present)
        }
        class HttpAssessmentApiClient {
            +submitFollowUp(FollowUpRequest) PredictionResult
        }
    }

    FastAPIApp --> FollowUpService : /predict/followup
    FollowUpService --> SymptomAssociationIndex : pick question
    AssessmentController --> AssessmentApiService
    AssessmentApiService --> ProfileService : consent gate
    AssessmentApiService --> PredictionClient
    PredictionClient --> FastAPIApp : POST /predict/followup
    AssessmentPage --> FollowUpQuestion
    AssessmentPage --> useAssessment
    useAssessment --> HttpAssessmentApiClient
    HttpAssessmentApiClient --> AssessmentController : POST /api/assessments/followup
```

**Sequence**

```mermaid
sequenceDiagram
    autonumber
    actor Patient
    participant FE as Web App (React)
    participant BE as Backend (Spring Boot)
    participant AI as Prediction Service (Python)

    Patient->>FE: Describe symptoms, submit
    FE->>BE: POST /api/assessments
    BE->>BE: requireConsentingPatient()
    BE->>AI: POST /predict
    AI->>AI: Predict. Top disease below threshold?
    AI->>AI: Pick the most discriminating symptom
    AI-->>BE: result + needs_follow_up=true + follow_up
    BE-->>FE: result + needsFollowUp + followUp
    FE-->>Patient: Show the yes/no question

    loop until needsFollowUp = false
        Patient->>FE: Answer yes or no
        FE->>FE: Add the symptom, rounds++
        FE->>BE: POST /api/assessments/followup
        BE->>BE: requireConsentingPatient()
        BE->>AI: POST /predict/followup
        AI->>AI: Add the answer, predict again
        alt confident, or no question, or round limit
            AI-->>BE: result + needs_follow_up=false
            BE-->>FE: final result
            FE-->>Patient: Show the final result
        else still below threshold
            AI-->>BE: result + next follow_up
            BE-->>FE: next question
            FE-->>Patient: Show the next question
        end
    end
```

**Baseline vs refined**

```mermaid
flowchart TB
    subgraph BEFORE["Baseline (after Iterations 8 and 9)"]
        direction TB
        B1["One-shot /predict: symptoms in, diseases out"]
        B2["A vague description gives low confidence.<br/>There is no way to ask for more"]:::problem
    end
    subgraph AFTER["Refined (Iteration 9b, FR-P3)"]
        direction TB
        A1["The AI asks the most discriminating<br/>symptom as a yes/no question"]
        A2["POST /predict/followup adds the answer<br/>and predicts again (stateless, round limit)"]
        A3["The backend adds a consent-gated endpoint"]
        A4["React runs the loop and shows only<br/>the final result"]
        A1 --> A2 --> A3 --> A4
    end
    BEFORE ==>|"add a yes/no loop across AI, backend and frontend"| AFTER
    classDef problem fill:#ffe0e0,stroke:#c0392b,color:#7b1e14;
```

### Step 4 — Model-to-code transformation (by repository)

**AI (`disease-prediction-ai`, branch `feat/adaptive-followup-questions`)**

- `associations.py` — `SymptomAssociationIndex` learns the symptom frequency for each disease. It
  selects the most discriminating symptom that the patient has not answered.
- `followup.py` — `FollowUpService` runs the loop. It uses the confidence threshold and the round
  limit.
- `api.py` — `/predict` returns `needs_follow_up` and `follow_up`. The new `POST /predict/followup`
  continues the loop from the carried symptoms and the latest answer.
- `pipeline.py` — exposes the trained classifier so the loop re-ranks without the NLP stage.
- Configuration: `DISEASE_FOLLOWUP_THRESHOLD`, `DISEASE_FOLLOWUP_MAX_ROUNDS`, `DISEASE_DATASET_PATH`.

**Backend (`disease-prediction-be`, branch `feat/adaptive-followup`)**

- `assessment-api.yaml` — adds `POST /api/assessments/followup`. Adds the `FollowupRequest`,
  `FollowupAnswer` and `FollowUp` schemas. Adds `activeSymptoms`, `needsFollowUp` and `followUp`
  to `PredictionResult`.
- `PredictionClient` / `HttpPredictionClient` — add `predictFollowUp(...)`. It posts to the AI
  `/predict/followup`.
- `AssessmentApiService.continueAssessment(...)` — puts the yes/no answer into the confirmed set or
  the denied set. It calls `requireConsentingPatient()` first, like `runAssessment`.
- `AssessmentController.submitFollowup(...)` — a thin delegate.

**Frontend (`disease-prediction-fe`, branch `feat/adaptive-followup`)**

- `domain/assessment.ts` — adds the `FollowUp`, `FollowUpAnswer` and `FollowUpRequest` types. Adds
  `activeSymptoms`, `needsFollowUp` and `followUp` to `PredictionResult`. These fields are optional.
- `HttpAssessmentApiClient` — adds `submitFollowUp(...)`. It posts to `/api/assessments/followup`.
- `useAssessment.answerFollowUp(present)` — collects the confirmed and denied symptoms and the
  round count across the turns.
- `FollowUpQuestion` and `AssessmentPage` — show the yes/no question and run the loop in the page.
  The page gives only the final confident result to the host.
- Same branch: the symptom checklist now covers the full canonical vocabulary (131 symptoms) in
  clinical groups. It replaces the earlier subset.

### Later refinement — top condition with precautions (branch `feat/top-disease-with-precautions`)

A follow-up change on top of this iteration narrows what the result view shows and surfaces the
disease precautions, which were already produced by the AI but dropped before reaching the patient.

- **AI (`disease-prediction-ai`):** no change. The `/predict` response already carries a
  `description` and `precautions` for each disease, loaded by `DiseaseKnowledgeBase` from
  `data/symptom_Description.csv` and `data/symptom_precaution.csv`.
- **Backend (`disease-prediction-be`):** `assessment-api.yaml` adds `description` and `precautions`
  to the `DiseasePrediction` schema; `HttpPredictionClient` now maps both fields from the AI
  response instead of discarding them.
- **Frontend (`disease-prediction-fe`):** `domain/assessment.ts` adds optional `description` and
  `precautions` to `DiseasePrediction`. `PredictionResultView` shows only the single most likely
  condition — its probability, description, contributing symptoms and recommended precautions —
  rather than the full ranked list. The backend still requests the top five.

### Step 5 — Validate / test

| Service | Scope | Result |
| --- | --- | --- |
| AI (Python 3.12.10) | association index (6), follow-up loop (9), endpoints in `test_api.py` (23) | full suite 166 tests pass |
| Backend (JDK 25) | the client maps the new fields and posts to `/predict/followup`; the service merges answers and counts rounds; the controller returns 200 / 400 | full suite 16 tests pass |
| Frontend | client method and error mapping, hook loop, `FollowUpQuestion` component, one integration test | 157 tests pass (113 unit + 44 integration) |

The AI tests confirm that the loop stops: the confidence rises past the threshold; the service does
not ask a denied symptom again; the round limit stops the loop.

### Step 6 — Review & refine the model

1. The loop keeps no state on the server. The client carries the confirmed symptoms, the denied
   symptoms and the round count. Both services stay stateless. Both services scale horizontally.
2. The service selects a discriminating question, not only a frequent symptom. Each answer gives
   more information. The loop needs fewer questions.
3. The loop always stops. There are three reasons: the confidence reaches the threshold; no useful
   question remains; the loop reaches the round limit.
4. The service checks consent for each turn. `continueAssessment` calls `requireConsentingPatient()`,
   like `runAssessment`. A follow-up answer cannot pass the Iteration 6 consent check.
5. The UI shows only the final result. `AssessmentPage` runs the loop in the page. It does not show
   the intermediate results.
6. The same branch fixes an authentication race. A protected call could run before the Hanko token
   was ready. This caused a wrong 401. The fetch now waits for the token.
7. The result view shows only the top condition, with precautions. The ranked list was replaced by
   the single most likely disease plus its description, contributing symptoms and precautions. The
   precautions already existed in the AI response; the backend had been dropping them, so the fix
   was to relay the field (backend) and render it (frontend), not to produce new data.

## Iteration 9c — Red-flag Urgency: Ambulance vs Doctor (AI + backend + React, FR-P5)

Design and model-to-code record for Iteration 9c of the Model Driven Development cycle (Lab05,
Problem 1). This is the **final iteration** of the cycle. The slice: make the AI prediction service
the single source of truth for red flags, relay the urgency level through the backend, and show the
patient a clear instruction: **call an ambulance (112)** or **see a doctor promptly**. The same
iteration closes a data gap: every one of the 41 diseases now has a description and precautions.
Like Iteration 9b, it comes after Iteration 9 and does not use a planned roadmap number.

- **Anchored to:** FR-P5 / UC2 (emergency alert), FR-P3 (description and precautions).
- **Lives in:** `disease-prediction-ai`, `disease-prediction-be` and `disease-prediction-fe`, each
  on branch `feat/red-flag-urgency` (from `develop`, PR into `develop`).
- **Status:** done. Steps 5 and 6 are filled in after implementation.
- **Out of scope:** the unused backend chain `RedFlagHandler` / `ChestPainHandler` /
  `RedFlagRule.java` (left for a separate cleanup PR); urgency from free-text answers in the
  follow-up loop (the loop accepts only yes or no).

> Render the `.mmd` files with the VS Code *Markdown Preview Mermaid* extension, IntelliJ's
> Mermaid plugin, or by pasting into <https://mermaid.live>. GitHub renders the fenced `mermaid`
> blocks below automatically.

---

### Step 1 — Requirements envisioning

- Slice: detect dangerous symptom patterns once, in the AI service, and tell the patient what to
  do. The instruction depends on the urgency level:

  | Urgency | Patient instruction | Basis |
  | --- | --- | --- |
  | `EMERGENCY` | "Call emergency services (112) or go to the nearest emergency department now." | A short list of dangerous symptoms or symptom combinations. It does not depend on the predicted disease |
  | `URGENT` | "See a doctor promptly today; seek emergency care if it gets worse." | Single serious symptoms |
  | `ADVISORY` / none | The normal result and the disclaimer | Default |

- Decision rule: the urgency comes from the **symptoms**, never from the precaution text. The
  precaution text is noisy. For example, "consult nearest hospital" is a precaution for
  Osteoarthritis, which is not an emergency.
- Data gap: `symptom_Description.csv` spells one disease "Dimorphic hemorrhoids(piles)". The
  dataset and the precaution file spell it "Dimorphic hemmorhoids(piles)". The lookup did not match,
  so this disease had no description.

### Step 2 — Architecture / platform envisioning (SOA)

Before this iteration, three red-flag mechanisms existed and they were not connected:

| Layer | Before | Problem |
| --- | --- | --- |
| AI `red_flags.py` | `RedFlagEngine`, 6 rules, 3 urgency levels | Only on `/red-flags/check`. `/predict` returns no red flags |
| Backend `AssessmentApiService` | A keyword list (`RED_FLAG_PHRASES`) on the free text and the symptom text | Parallel logic, yes/no only, different from the AI rules |
| Frontend `EmergencyBanner` | One banner, keyed on `emergencyDetected` | Cannot tell "ambulance" from "doctor" |

After this iteration each service keeps its role:

- **AI (Python):** runs the red-flag rules on every prediction. It is the single source of truth.
- **Backend (Java):** relays the red flags and the highest urgency. It keeps `emergencyDetected`
  for older clients and derives it from `highestUrgency == EMERGENCY`. It has no keyword list now.
- **Frontend (React):** shows the right banner. For `EMERGENCY` it leads with the ambulance banner
  and does not show the prediction.

Each layer degrades gracefully when the layer below is older: the new fields are optional, and the
frontend falls back to `emergencyDetected`.

#### Red-flag rules (AI)

| Rule | Urgency | Triggers |
| --- | --- | --- |
| `possible_cardiac_event` | EMERGENCY | any 2 of `chest_pain`, `breathlessness`, `sweating` |
| `possible_stroke` | EMERGENCY | `weakness_of_one_body_side`, `altered_sensorium` or `slurred_speech` |
| `loss_of_consciousness` (new) | EMERGENCY | `coma` |
| `coughing_blood_with_breathlessness` (new) | EMERGENCY | `blood_in_sputum` and `breathlessness` |
| `severe_dehydration` | URGENT | `vomiting` and `dehydration` |
| `high_fever` | URGENT | `high_fever` |
| `acute_breathing_difficulty` | URGENT | `breathlessness` |
| `gi_bleeding` | URGENT | `bloody_stool` |
| `stomach_bleeding` (new) | URGENT | `stomach_bleeding` |
| `acute_liver_failure` (new) | URGENT | `acute_liver_failure` |
| `fluid_overload` (new) | URGENT | `fluid_overload` |

Every EMERGENCY message tells the patient to call 112. Every URGENT message tells the patient to
see a doctor promptly.

### Step 3 — Model the slice

#### Structural model (`urgency_class.mmd`)

```mermaid
classDiagram
    namespace PredictionService_Python {
        class RedFlagEngine {
            +check(active_symptoms) RedFlag[]
            +highest_urgency(active_symptoms) Urgency
        }
        class RedFlagRule {
            +name
            +urgency Urgency
            +message
            +triggers
            +min_matches
        }
        class FastAPIApp {
            +POST /predict
            +POST /predict/followup
            +POST /red-flags/check
        }
        class PredictResponse {
            +red_flags RedFlagItem[]
            +highest_urgency Urgency
        }
        class DiseaseKnowledgeBase {
            +describe(disease)
            +precautions_for(disease)
        }
    }
    namespace Backend_Java {
        class HttpPredictionClient {
            +predict(freeText, entries) PredictionResult
            +predictFollowUp(confirmed, denied, rounds) PredictionResult
        }
        class AssessmentApiService {
            +runAssessment(AssessmentRequest) AssessmentResponse
            +continueAssessment(FollowupRequest) AssessmentResponse
            -isEmergency(result) boolean
        }
        class PredictionResult {
            +redFlags RedFlag[]
            +highestUrgency Urgency
        }
    }
    namespace Frontend_React {
        class AssessmentPage
        class UrgencyBanner {
            +urgency Urgency
            +redFlags RedFlag[]
        }
        class urgencyOf {
            +response AssessmentResponse
        }
        class EmergencyRoute
        class ResultRoute
    }

    RedFlagEngine --> RedFlagRule : evaluates
    FastAPIApp --> RedFlagEngine : every prediction
    FastAPIApp --> DiseaseKnowledgeBase : description, precautions
    FastAPIApp ..> PredictResponse
    HttpPredictionClient --> FastAPIApp : POST /predict
    HttpPredictionClient ..> PredictionResult : relays red flags
    AssessmentApiService --> HttpPredictionClient
    AssessmentApiService ..> PredictionResult : emergencyDetected = EMERGENCY
    AssessmentPage --> UrgencyBanner
    EmergencyRoute --> UrgencyBanner : EMERGENCY
    ResultRoute --> UrgencyBanner : URGENT
    AssessmentPage ..> urgencyOf
```

#### Sequence (`urgency_sequence.mmd`)

```mermaid
sequenceDiagram
    autonumber
    actor Patient
    participant FE as Web App (React)
    participant BE as Backend (Spring Boot)
    participant AI as Prediction Service (Python)

    Patient->>FE: Describe or tick symptoms, submit
    FE->>BE: POST /api/assessments
    BE->>BE: requireConsentingPatient()
    BE->>AI: POST /predict
    AI->>AI: Parse symptoms, predict
    AI->>AI: RedFlagEngine.check(active symptoms)
    AI-->>BE: predictions + red_flags + highest_urgency
    BE->>BE: emergencyDetected = (highestUrgency == EMERGENCY)
    BE-->>FE: result.redFlags + result.highestUrgency + emergencyDetected

    alt highestUrgency = EMERGENCY
        FE-->>Patient: Red alert "Call an ambulance (112) now". No prediction is shown
    else highestUrgency = URGENT
        FE-->>Patient: Amber banner "See a doctor promptly", then the result
    else no urgency (or ADVISORY)
        FE-->>Patient: The result and the disclaimer only
    end
```

#### Baseline vs refined (`baseline_vs_refined.mmd`)

```mermaid
flowchart TB
    subgraph BEFORE["Baseline (after Iteration 9b)"]
        direction TB
        B1["AI: RedFlagEngine only on /red-flags/check.<br/>/predict returns no red flags"]:::problem
        B2["Backend: its own keyword list (RED_FLAG_PHRASES)<br/>gives a yes/no emergencyDetected"]:::problem
        B3["Frontend: one banner for emergencyDetected.<br/>No difference between ambulance and doctor"]:::problem
        B4["One disease loses its description<br/>(hemorrhoids spelling mismatch)"]:::problem
    end
    subgraph AFTER["Refined (Iteration 9c, FR-P5)"]
        direction TB
        A1["AI: /predict and /predict/followup attach<br/>red_flags + highest_urgency (single source of truth)"]
        A2["Backend: relays redFlags + highestUrgency.<br/>emergencyDetected = highestUrgency == EMERGENCY"]
        A3["Frontend: UrgencyBanner. EMERGENCY = call 112,<br/>URGENT = see a doctor promptly"]
        A4["Knowledge base: 41 / 41 diseases have a<br/>description and precautions (test)"]
        A1 --> A2 --> A3
    end
    BEFORE ==>|"move red-flag detection to the AI and relay the urgency level"| AFTER
    classDef problem fill:#ffe0e0,stroke:#c0392b,color:#7b1e14;
```

### Step 4 — Model-to-code transformation (by repository)

**AI (`disease-prediction-ai`, branch `feat/red-flag-urgency`)**

- `red_flags.py` — adds `loss_of_consciousness` and `coughing_blood_with_breathlessness`
  (EMERGENCY), and `stomach_bleeding`, `acute_liver_failure` and `fluid_overload` (URGENT). The
  messages now say "call 112" or "see a doctor promptly".
- `api.py` — `PredictResponse` gets `red_flags` and `highest_urgency`. `/predict` and
  `/predict/followup` run `RedFlagEngine.check(...)` on the active symptoms. `RedFlagItem` is reused
  from `/red-flags/check`.
- `knowledge.py` — `_key()` maps the description spelling "dimorphic hemorrhoids(piles)" to the
  dataset spelling. The Kaggle CSVs are downloaded, not versioned in the repository, so the fix is in
  the lookup and not in the CSV file.

**Backend (`disease-prediction-be`, branch `feat/red-flag-urgency`, contract-first)**

- `assessment-api.yaml` — adds the `Urgency` enum (`EMERGENCY`, `URGENT`, `ADVISORY`) and the
  `RedFlag` schema (`name`, `urgency`, `message`, `matchedSymptoms`). Adds `redFlags` and
  `highestUrgency` (nullable) to `PredictionResult`. Keeps `emergencyDetected` on
  `AssessmentResponse`. The Java models are regenerated with `./mvnw generate-sources`; no
  generated code is edited by hand.
- `HttpPredictionClient` — maps `red_flags` and `highest_urgency` from the AI response into
  `PredictionResult`. An unknown urgency value maps to "no urgency".
- `AssessmentApiService` — removes `RED_FLAG_PHRASES`, `detectEmergency()` and `containsRedFlag()`.
  It always calls the prediction service and sets `emergencyDetected` from the relayed urgency, on
  the initial path and on the follow-up path.

**Frontend (`disease-prediction-fe`, branch `feat/red-flag-urgency`)**

- `domain/assessment.ts` — adds the `Urgency` type, the `RedFlag` interface, and optional
  `redFlags` / `highestUrgency` on `PredictionResult`. `urgencyOf(response)` returns the relayed
  urgency, or `EMERGENCY` from `emergencyDetected` for older backends.
- `UrgencyBanner` (new) — `EMERGENCY`: red alert "Call an ambulance (112) now" (`role="alert"`,
  `tel:112` link). `URGENT`: amber notice "See a doctor promptly" (`role="status"`). Otherwise
  nothing. Both list the red-flag messages as the reasons.
- `AssessmentPage`, `AssessmentRoute`, `EmergencyRoute`, `ResultRoute` — choose the banner from
  `urgencyOf(...)`. `EMERGENCY` goes to `/assessment/emergency` and shows no prediction (FR-P5, as
  in Iterations 3 and 4). `URGENT` shows the result with the banner above it.

### Step 5 — Validate / test

| Service | Scope | Result |
| --- | --- | --- |
| AI (Python 3.12) | new rules fire and do not fire; urgency order; `/predict` returns `EMERGENCY` for `{chest_pain, breathlessness}`, `URGENT` for `high_fever`, nothing for benign input; `/predict/followup` returns `EMERGENCY`; spelling alias; all 41 diseases have a description and precautions | 179 tests pass, 12 new. The 41-disease test runs when the Kaggle CSVs are in `data/` and is skipped otherwise. 12 tests in `test_publish_model_runtime.py` (image publishing script) also fail on `develop` on Windows; they are not related to this change |
| Backend (JDK 25) | the client maps `red_flags` / `highest_urgency` and missing fields; the service derives `emergencyDetected` from `EMERGENCY` only (not from `URGENT`, not from keywords) on both paths; the controller returns the urgency in JSON | 36 tests without a database pass (19 in the changed classes). The 42 Testcontainers tests need Docker, which was not available on the test machine; run `./mvnw clean test` with Docker before merging |
| Frontend | `UrgencyBanner` for each urgency; `urgencyOf` and its fallback; routed flow: emergency leads with the ambulance banner and hides the prediction, urgent shows "see a doctor" above the result, benign shows no banner; inline page: the alert comes before the form | 169 tests pass (123 unit + 46 integration); `tsc`, ESLint, Prettier and the build pass |

Acceptance criteria:

- `{chest_pain, breathlessness}` gives `highestUrgency = EMERGENCY` and the UI leads with "Call an
  ambulance (112) now".
- A single `high_fever` gives `URGENT` and the UI says "See a doctor promptly".
- Benign input shows no urgency banner, only the result and the disclaimer.
- The backend has no keyword list; `emergencyDetected` comes from the relayed urgency.
- All 41 diseases have a description and precautions; a test checks it.

### Step 6 — Review & refine the model

1. **One source of truth.** The red-flag rules exist only in the AI service. The backend keyword
   list is gone, so the backend and the AI can no longer disagree about an emergency.
2. **Symptoms, not precautions.** The urgency comes from the symptoms only. The precaution text is
   not used, because it marks non-urgent diseases (for example Osteoarthritis) with "consult
   nearest hospital".
3. **Two levels for the patient.** The patient now sees two different instructions: ambulance
   (112) for `EMERGENCY` and doctor for `URGENT`. Before, there was only one yes/no alert.
4. **Emergency still halts the prediction.** For `EMERGENCY` the prediction is relayed (so the red
   flags reach the client) but the frontend does not show it, as FR-P5 and the UC1 sequence
   require.
5. **Trade-off: emergency now needs the AI service.** Before, the backend keyword check could flag
   an emergency even when the AI service was down. Now, if the AI service cannot be reached, the
   request fails (503) and no urgency is computed. The frontend error message keeps the 112 advice.
   A circuit breaker with a fallback (roadmap Iteration 10, Resilience4j) is the place to restore an
   offline emergency check.
6. **Backward compatible.** `emergencyDetected` stays in the contract. The new fields are optional.
   The frontend falls back to `emergencyDetected` when `highestUrgency` is absent.
7. **Data fix in the lookup.** The Kaggle CSVs are not in the repository, so the spelling fix is an
   alias in `DiseaseKnowledgeBase._key()`. The coverage test protects the 41 / 41 result.

### Roadmap check (planned iterations 5–13)

Status of the roadmap items (`roadmap/roadmap.png`) at the end of the cycle, checked against
`develop` in each repository:

| Roadmap iteration | Status |
| --- | --- |
| 5 — User and consent store (Java BE) | Done (`iteration5/`) |
| 6 — Identity (BE + React) | Done with Hanko instead of Keycloak (`iteration6/`) |
| 7 — Health-data persistence (Java BE) | Not done (no assessment tables; migrations stop at V2) |
| 8 — Python prediction service web layer | Done (`iteration8/`) |
| 9 — Python ML pipeline (classifier) | Done (`iteration9/`); follow-up loop in `iteration9b/` |
| 10 — Real SOA boundary (BE to Python) | Partly done: `HttpPredictionClient` calls the AI service; no Resilience4j circuit breaker yet |
| 11 — Event-driven communication (Kafka) | Not done |
| 12 — Doctor and admin apps (React) | Not done |
| 13 — Observability | Not done |
| 9c — Red-flag urgency (this iteration, not on the roadmap) | Done |

The diagram sources are in `iteration9c/`.

## Coordinator & team notes

- Coordinator: `<fill in name>` — tracks who did what and negotiates per-member points (Lab05
  Problem 1 requirement).
- Iteration ownership:
  - Iteration 1 — NLP parser (Python) — `<teammate(s), feature/nlp-pipeline>`
  - Iteration 2 — Assessment API (Java backend) — `<owner>`
  - Iteration 3 — Patient Web App: prediction feature (React frontend) — `<owner>` (planned; see `MDD_ITERATIONS_FE_PLAN.md`, Part 1)
  - Iteration 4 — Patient Web App: client-side routing (React Router) — `<owner>` (planned; see `MDD_ITERATIONS_FE_PLAN.md`, Part 2)
  - Iteration 5 — User and consent persistence (Java backend) — `<owner>` (implemented; see `iteration5/README.md`)
  - Iteration 6 — Identity and login with Hanko (Java backend + React) — `<owner>` (designed; see `iteration6/README.md`)
  - AOP component — Personal Data Access Guard (Java backend) — Daria (designed; see `iteration6/README.md`, Part B)
- Repository / process (Lab05 Theme 2 — GitHub): GitFlow branching (main ← develop ← feature
  branches), CI pipeline (lint, build, secret scan, OWASP dependency scan, SonarQube gate,
  containerize), and pre-commit hooks (ruff + hygiene). Process/DevOps evidence for the GitHub
  theme — not counted as an MDD iteration (no model is transformed into the pipeline).

## Use of Artificial Intelligence

Per the Lab05 AI-use policy:

- Tool: Kiro IDE agent (server-selected model, "Auto").
- How used: AI acted as a modelling/documentation partner. The team made all decisions (MDD over
  TDD, which slices to iterate on, the SOA split). The AI read the existing code, derived the model
  fragments that the code already realises, built the model→code traceability tables, implemented
  Iteration 2's web layer from the API contract, ran both test suites to produce the Step 5
  evidence, and rendered the diagrams. Diagrams and tables were reviewed by the team.

---

# Appendix — Implementation Verification Record (Iterations 1 & 2)

This appendix verifies the iterations above against the code that is actually implemented, step by
step. For every one of the six MDD steps it records whether the step's artefact exists, whether the
implementation is the *same as* / *better than* / *weaker than* the description, and the evidence.

Verification was run on **2026-11-02** on Windows.
Repositories: `disease-prediction-ai` (Iteration 1, branch `feature/nlp-pipeline`) and
`disease-prediction-be` (Iteration 2, branch `main`).

Legend: ✅ same as described · ⬆️ better than described · ⚠️ discrepancy (corrected above).

## B.1 Summary

| Iteration | Step | Implemented? | Verdict |
| --- | --- | --- | --- |
| 1 — Free-text symptom parsing | 1 Requirements envisioning | Yes (anchored) | ✅ same |
| 1 | 2 Architecture / platform envisioning | Yes | ✅ same |
| 1 | 3 Model the slice | Yes | ✅ same |
| 1 | 4 Model-to-code transformation | Yes | ⬆️ better (every element present + extras) |
| 1 | 5 Validate / test | Yes | ⬆️ better (40 pass, 0 fail — no failures to excuse) |
| 1 | 6 Review & refine the model | Yes | ✅ same (all three refinements in code) |
| 2 — Assessment API | 1 Requirements envisioning | Yes (anchored) | ✅ same |
| 2 | 2 Architecture / platform envisioning | Yes | ✅ same |
| 2 | 3 Model the slice | Yes | ✅ same |
| 2 | 4 Model-to-code transformation | Yes | ⬆️ better (naming + records; domain untouched) |
| 2 | 5 Validate / test | Yes | ⬆️ better (adds emergency test; 4 pass) |
| 2 | 6 Review & refine the model | Yes | ✅ same (decisions encoded in code) |

Overall: **every step of both iterations is implemented.** Iteration 1 is fully realised code;
Iteration 2 — originally described as "planned / in progress" — is now implemented and green.

## B.2 Iteration 1 — Free-Text Symptom Parsing (Python)

- **Step 1 — ✅ same.** Slice anchored to FR-P1 and the free-text-mapping behaviour. Mapped symptoms
  become a weighted vector; negated / unmapped text is preserved (`ParseResult.negated`,
  `ParseResult.entities`).
- **Step 2 — ✅ same.** Lives entirely in the Python service; heavy NLP libs are **lazy-imported**
  inside each concrete class (`ClinicalBertExtractor._get_pipeline`, `NegspacyNegationDetector._get_nlp`,
  `SemanticEntityLinker._get_model` / `_fuzzy_match`), so the pure-Python pipeline runs without the
  ML stack installed.
- **Step 3 — ✅ same.** The four-stage pipeline (Extractor → NegationDetector → Linker →
  vectorization) is the structure of `prediction_service/parser.py`; each role is a `Protocol`, each
  data element a dataclass.
- **Step 4 — ⬆️ better.** Every model element maps one-to-one to code in `parser.py`
  (`SymptomParser`, the three `Protocol`s + realizations, `ExtractedEntity`/`LinkedSymptom`/`ParseResult`),
  with `CANONICAL_SYMPTOMS`+`normalise_symptom()` in `vocabulary.py`, `weight_for()`+`SEVERITY_WEIGHTS`
  in `severity.py`, and `to_symptom_entries()`→`SymptomEntryDTO` in `models.py`. Extras beyond the model:
  `binary_vector()`, `numpy_vector(dtype=...)`, `active_symptoms()`, a second `KeywordNegationDetector`,
  `@runtime_checkable` Protocols, and injectable implementations (`pipeline=`/`nlp=`/`model=`).
- **Step 5 — ⬆️ better (with the ⚠️ now fixed in the main doc).** `tests/test_parser.py` covers the
  modelled behaviour. Verified: **40 passed in 0.26s on Python 3.12.10**; the fuzzy-fallback
  (`rapidfuzz`) path runs and passes. CI is pinned to Python 3.12, so local and CI agree. (An earlier
  draft wrongly said "38 passed, 2 failed / Python 3.15" with CI on 3.13 — corrected in Iteration 1,
  Step 5 above.) `test_parser_default_vocabulary_is_canonical` asserts the vocabulary length is 131,
  matching `vocabulary.py`.
- **Step 6 — ✅ same.** All three refinements are in code: negation is a dedicated stage
  (`parse()` splits affirmed/negated before linking); two-tier linking (`_link_one()` semantic →
  `_fuzzy_match()`); severity is an int weight (`LinkedSymptom.severity_weight`, enum derived at the end).

## B.3 Iteration 2 — Assessment API (Java backend)

The pre-existing baseline was richer than "declaration-only": the backend already had an
`AssessmentService`, `AssessmentProcessor` / `StandardAssessmentProcessor` (Template Method),
red-flag handlers (Chain of Responsibility) and report observers. Iteration 2 adds the missing HTTP
boundary without rewriting that code.

- **Step 1 — ✅ same.** Slice "submit assessment → receive prediction result", anchored to UC1, FR-P1,
  FR-P2.
- **Step 2 — ✅ same.** Web layer delegates prediction across the SOA boundary via `PredictionClient`
  (port); `StubPredictionClient` is the current adapter; `HttpPredictionClient` is the Step 6 follow-up.
- **Step 3 — ✅ same.** API contract + controller → service → client structure modelled and rendered
  (`mdd/iteration2/` in this repository: `assessment_api_class`, `assessment_api_sequence`,
  `baseline_vs_refined`, with Mermaid embedded in the iteration README and editable `.mmd` sources).
- **Step 4 — ⬆️ better.** Every model element maps to Java under
  `src/main/java/com/diseaseprediction/`: `@RestController AssessmentController`; request/response and
  result DTOs as **records** in `controller/dto/`; `@Service AssessmentApiService.runAssessment()`;
  `interface PredictionClient` + `@Component StubPredictionClient`. Deliberate, non-destructive
  deviations: orchestrator named `AssessmentApiService` (to coexist with the pre-existing
  `AssessmentService`); immutable records instead of the getter-less domain skeletons; validation
  modelled into `AssessmentRequest.hasAnySymptomSignal()`.
- **Step 5 — ⬆️ better.** `@WebMvcTest AssessmentControllerTest` with a mocked `PredictionClient`,
  adding a third (emergency) test beyond the planned two: `validRequestReturns200WithPrediction`,
  `emptyRequestReturns400`, `chestPainIsFlaggedAsEmergency`. Verified: **Tests run: 4, Failures: 0,
  Errors: 0 — BUILD SUCCESS** (3 controller + existing `contextLoads`). Caveat: built on JDK 21
  (JetBrains Runtime) with `-Dmaven.compiler.release=21` because no JDK 25 is installed (pom pins
  `java.version=25`; CI uses that toolchain); Spotless couldn't run offline so AOSP formatting is
  unverified — run `./mvnw spotless:apply` online before committing.
- **Step 6 — ✅ same.** Emergency check runs before prediction in `runAssessment()`; prediction is
  behind the `PredictionClient` interface; open items (timeout/fallback, `GET /api/assessments/{id}`)
  carried forward.

## B.4 Open action items

1. ✅ Done — Iteration 1 Step 5 numbers corrected to "40 passed, 0 failed" on Python 3.12.10 (CI 3.12).
2. Before committing the backend: run `./mvnw spotless:apply` (online) and, ideally, a build on the
   pinned JDK 25 to confirm the `java.version=25` toolchain.
3. Iteration 2 Step 6 follow-up: implement `HttpPredictionClient` calling the Python `POST /predict`,
   replacing `StubPredictionClient`.
