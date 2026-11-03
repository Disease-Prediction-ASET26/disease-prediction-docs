# MDD Iteration Diagrams

Model diagrams for the Model Driven Development iterations (Lab05, Problem 1). Kept in the docs
repository — deliberately **not** mixed into the service code repos. Each `.mmd` is the Mermaid
source; each `.png` is the rendered image (2× scale). See `MDD_ITERATIONS.md` and
`MDD_ITERATIONS_FE_PLAN.md` for the written plans these diagrams belong to.

| Iteration | Folder | Diagrams |
| --- | --- | --- |
| 1 — Free-text symptom parsing (Python) | `iteration1/` | `parser_pipeline_class`, `parser_pipeline_flow` |
| 2 — Assessment API (Java backend) | `iteration2/` | `assessment_api_class`, `assessment_api_sequence`, `baseline_vs_refined` |
| 3 — Patient Web App: prediction feature (React) | `iteration3/` | `assessment_feature_class`, `assessment_feature_flow`, `baseline_vs_refined` |
| 4 — Patient Web App: client-side routing (React) | `iteration4/` | `routing_class`, `routing_flow`, `baseline_vs_refined` |

To re-render a `.mmd` to `.png`: open it in the VS Code *Markdown Preview Mermaid* / IntelliJ
Mermaid plugin, or paste into <https://mermaid.live> and export.
