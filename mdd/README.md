# MDD Iteration Diagrams

Model Driven Development iterations (Lab05, Problem 1). The iteration READMEs and
`MDD_ITERATIONS.md` embed the Mermaid diagrams so they display with the text. Each `.mmd` file
keeps the editable source. See `MDD_ITERATIONS.md` and `MDD_ITERATIONS_FE_PLAN.md` for the plans.

| Iteration | Folder | Diagrams |
| --- | --- | --- |
| 1 — Free-text symptom parsing (Python) | `iteration1/` | `parser_pipeline_class`, `parser_pipeline_flow` |
| 2 — Assessment API (Java backend) | `iteration2/` | `assessment_api_class`, `assessment_api_sequence`, `baseline_vs_refined` |
| 3 — Patient Web App: prediction feature (React) | `iteration3/` | `assessment_feature_class`, `assessment_feature_flow`, `baseline_vs_refined` |
| 4 — Patient Web App: client-side routing (React) | `iteration4/` | `routing_class`, `routing_flow`, `baseline_vs_refined` |
| 5 — User and consent persistence (Java backend) | `iteration5/` | `user_consent_er`, `user_persistence_class`, `baseline_vs_refined` |
| 6 — Identity and login with Hanko (Java backend + React), plus the AOP access guard (Part B) | `iteration6/` | `login_sequence`, `identity_class`, `baseline_vs_refined`, `access_guard_class`, `access_guard_sequence` |
| 8 — Prediction Service: web layer (Python) | `iteration8/` | `web_layer_class`, `baseline_vs_refined` |
| 9 — ML pipeline: trained classifier (Python) | `iteration9/` | `classifier_class`, `training_flow`, `baseline_vs_refined` |
| 9b — Adaptive follow-up questions (AI + backend + React) | `iteration9b/` | `followup_class`, `followup_sequence`, `baseline_vs_refined` |
| 9c — Red-flag urgency: ambulance vs doctor (AI + backend + React), final iteration | `iteration9c/` | `urgency_class`, `urgency_sequence`, `baseline_vs_refined` |
| Roadmap — planned iterations 5–13 | `roadmap/` | `roadmap.png` |

Iteration 7 (Health-data persistence, Java backend) remains a planned roadmap item. Iterations 8,
9 and 9b are documented in full (text and embedded Mermaid diagrams) in `MDD_ITERATIONS.md` and in
their own folders. Iteration 9b is a cross-service feature placed after Iteration 9 so it does not
take Iteration 7's number. Iteration 9c (red-flag urgency) is the final iteration and follows the
same rule; its README ends with a check of every roadmap item against the code on `develop`.

To edit a diagram, update its `.mmd` source and the matching Mermaid block in the related page.
