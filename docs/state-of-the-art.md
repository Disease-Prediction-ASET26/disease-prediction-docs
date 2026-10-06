# State of the Art: Disease Prediction Apps

## Found evidence quick summary

| Study and design                                                           | Reported result                                                                                  | Main limit                                                                                       |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Semigran et al. (2015), 23 public checkers and 45 vignettes [2](#ref-2)    | Correct diagnosis ranked first in 34%; triage matched in 57%.                                    | Standard cases and one inputter; no user outcome measure.                                        |
| Wallace et al. (2022), systematic review of 10 studies [3](#ref-3)         | Top diagnosis accuracy ranged from 19% to 37.9%; triage from 48.8% to 90.1%.                     | Different cases and reference standards prevented a pooled estimate.                             |
| Chan et al. (2021), prospective study of 581 adults [10](#ref-10)          | Triage accuracy was 73%, compared with 58% for patients' own triage.                             | Only 10 emergency cases; participants were already seeking care.                                 |
| Hassan et al. (2024), text classification on Symptom2Disease [19](#ref-19) | Best model reached 99.58% accuracy on 1,200 descriptions.                                        | Small, balanced dataset; no external patient cohort.                                             |
| DxFormer (2023), dialogue model on three datasets [27](#ref-27)            | Accuracy was 81.7%, 74.3%, and 63.3%, with 6.32, 8.74, and 7.58 questions.                       | Narrow disease sets and historical dialogue data.                                                |
| AMIE (2025), simulated consultations with 159 patient actors [29](#ref-29) | Rated higher on 30/32 specialist axes and 25/26 patient-actor axes.                              | Text-based simulated cases, not routine clinical consultations.                                  |
| SymptomAI (2026), randomized study with 13,917 participants [45](#ref-45)  | Reported odds ratio for higher expert accuracy ratings was 2.47.                                 | Provider diagnoses were self-reported; only 517 cases received expert review; research preprint. |
| Kenya primary-care trial (2026), 9,691 patients [33](#ref-33)              | Treatment failure was 2.2% with LLM support and 2.0% in control; difference was not significant. | Clinician support in one health system, not a public symptom checker.                            |

## 1. Patient-facing symptom checkers and clinical evaluations

### From diagnostic knowledge bases to public apps

Clinical decision-support systems have used medical knowledge bases for decades.
DXplain, developed by G. O. Barnett and colleagues at Massachusetts General
Hospital and Harvard, accepts clinical findings and returns a ranked
differential diagnosis with explanations [13](#ref-13). It was designed for
clinicians. Current public symptom checkers guide users through questions, then
list possible conditions, care advice, or both. Studies have tested Ada,
Babylon, Buoy, K Health, Mediktor, Symptomate, WebMD, Your.MD, and NHS 111
Online [2](#ref-2), [5](#ref-5), [7](#ref-7), [8](#ref-8). Since commercial
tools change, published results apply to the version tested.

Studies usually measure diagnostic ranking, triage, or both. Ranking records
whether the correct condition appears first or near the top of the list. Triage
records whether the tool directs the user to the right level of care. A correct
disease ranking does not guarantee safe care advice. Some studies require an
exact disease match; others also count related or plausible diagnoses. These
measures are not interchangeable.

### Accuracy in vignette audits

Semigran, Linder, Gidengil, and Mehrotra tested 23 online checkers with 45
cases: 15 emergency, 15 non-emergency, and 15 self-care. Across 770 diagnostic
evaluations, the correct diagnosis ranked first in 34% and appeared in the first
20 results in 58%. Triage advice matched the expected level of care in 57%. It
was more accurate for emergency than self-care cases (80% versus 33%)
[2](#ref-2). A research assistant entered each standard vignette, so the audit
did not measure how a broad sample of users would describe symptoms.

Two reviews found wide variation. Wallace and colleagues included ten studies;
first-choice diagnostic accuracy ranged from 19% to 37.9%, and triage accuracy
from 48.8% to 90.1% [3](#ref-3). Riboli-Sasco and colleagues included 14
studies: seven used vignettes and seven used patient data. Nine of 13 studies
that reported triage found suboptimal results. All 14 had at least one unclear
risk-of-bias area, and six had a high-risk area [4](#ref-4). Different
populations, reference standards, urgency categories, and measures prevented a
pooled analysis.

Schmieding and colleagues repeated the 45-case audit with 22 checkers in 2020.
Median triage accuracy was 55.8%, compared with 59.1% in the 2015 study. The
2020 tools were less cautious overall and missed more than 40% of emergencies
[5](#ref-5). In Australia, Hill and colleagues tested 36 tools with 48 cases.
The correct diagnosis ranked first in 36% and appeared in the top three in 52%;
triage accuracy was 49%. Tools did better on urgent cases than on non-urgent
cases, while many directed users to more care than needed [6](#ref-6). Across
both audits, checkers sometimes missed emergencies and sometimes recommended
unnecessary care.

Ceney and colleagues evaluated 12 tools over 548 completed vignette sessions.
First-choice diagnosis accuracy was 37.7%, triage accuracy was 57.7%, and the
authors’ safety measure was 82.6%. More than half of self-care cases received
advice to use more services than NICE guidance recommended. More questions were
associated with better first-choice diagnosis accuracy, but not better triage
accuracy [8](#ref-8). In another audit, seven primary-care doctors reviewed 400
cases and six checkers were tested. Avey reached 67.5% top-one and 87.3%
top-three accuracy; three doctors averaged 61.2% and 72.5%. Doctors had higher
precision and similar F1. All six paper authors were affiliated with Avey, the
company that developed the tool [9](#ref-9). The results come from vignettes and
do not measure routine app use.

### Patient use and health outcomes

Prospective studies with app users are less common. Chan and colleagues
evaluated a decision-tree prototype with 581 adults in emergency and
family-practice waiting rooms. Its triage recommendation matched the physician
reference in 73% of cases, compared with 58% for patients’ own triage. The study
included only ten emergency cases and excluded some cases with major
disagreement [10](#ref-10). Fraser and colleagues tested Ada with patients in a
US emergency department. Of 40 participants, 30 diagnosis cases and 37 triage
cases were available for analysis. Ada’s top-five diagnosis sensitivity was 70%,
close to the 68.9% mean for three physicians reviewing the same symptom reports.
At least two physician reviewers judged 14% of Ada’s triage outputs unsafe
[11](#ref-11).

A 2025 randomized trial tested Ada before clinician contact at acute-care sites
in Berlin. Of 442 randomized patients, 363 contributed to the primary analysis.
Satisfaction with the physician interaction was 78.5 out of 100 in the app group
and 80.8 in the control group. The difference was not statistically significant.
Anxiety, care satisfaction, and physician outcomes also showed no significant
difference [12](#ref-12). The trial measured the app’s effect on the clinical
visit, not its diagnostic accuracy.

The 2026 SafeMom study assessed an Ada symptom tool in South Africa’s MomConnect
maternal-health service. Among 968 women, a blinded physician panel judged 98.4%
of reviewed advice safe. The urgency level matched the panel in 48.9% and was
more cautious in 45.1%. Reported safe care-seeking behavior rose from 56.4% to
74.5% [14](#ref-14). With no control group, the study cannot attribute this
change to the checker; the findings apply to the women and service studied.

Kopka and colleagues’ 2025 review found self-triage accuracy from 11.5% to 90%
for symptom apps, 57.8% to 76% for LLMs, and 47.3% to 62.4% for laypeople.
Seventeen of 19 studies had at least one high or unclear risk-of-bias area, and
most used fictional cases [15](#ref-15). These ranges come from different tools
and test designs, so they do not support a single accuracy estimate for symptom
checkers.

## 2. Structured symptom models, text classification, and probability evidence

### Checklist models and public datasets

In structured studies, disease prediction is often a classification task: coded
symptoms are the inputs and disease labels are the outputs. Common models
include decision trees, nearest-neighbour methods, support vector machines,
random forests, and neural networks. These tests start with coded symptoms; they
do not measure symptom extraction from user text or the choice of follow-up
questions.

The project brief links the Disease Symptom Prediction dataset, a table of
symptom indicators and disease names [16](#ref-16). Fuster-Palà and colleagues
compared classifiers on a version with 4,920 cases and 41 disease labels.
K-nearest neighbours with k=2 reached 99.19% accuracy and 99.03% F1 on the
held-out split. The authors describe the data as synthetic or non-clinical and
report no clinical validation [17](#ref-17). The score applies to this dataset;
it does not estimate accuracy for new patients.

Symptom2Disease contains 1,200 short text descriptions, with 50 examples for
each of 24 disease labels [18](#ref-18). Hassan and colleagues compared
transformer models with a bidirectional long short-term memory network. Their
best model reached 99.58% accuracy and 99.13% F1 [19](#ref-19). The balanced set
has fixed labels and wording, and the study reports no external patient test.

Both datasets test closed-set classification after symptoms have been supplied.
They do not test missing details, co-occurring conditions, unusual wording, or a
different disease distribution. Benchmark papers usually report accuracy or F1,
while symptom-checker reviews report diagnosis ranking and triage. The reviews
do not report external calibration of individual disease probabilities
[3](#ref-3), [15](#ref-15). A classifier score is not automatically a
probability that matches disease frequency in a patient population.

### Bayesian systems and diagnostic knowledge

Bayesian diagnostic systems represent links between diseases and findings, then
update disease probabilities as evidence arrives. Guan and Baral used expected
information gain to select the next symptom question, with an option to plan
several questions ahead. They tested the method on small dialogue datasets and
larger simulated sets [20](#ref-20). Its probability updates are tied to the
knowledge base; results depend on that base’s coverage and on how simulated
patients answer.

Jones and Jones tested MidasMed on 30 sparse-history vignettes. It ranked the
correct diagnosis first in 28 cases and in the top three in 29. The paper
compared its score with Ada and with physician results from an earlier study.
MidasMed covered the 30 conditions in the test set, and the sample was small.
The physician comparison was not conducted head-to-head. Both authors were
affiliated with the company that developed MidasMed [21](#ref-21).

### Ranked differentials and synthetic scale

DDxT uses a transformer to generate an ordered differential diagnosis and a
separate classifier to predict the ground-truth disease. On DDXPlus, it reached
a mean differential F1 score of 94.72% and mean differential accuracy of 99.82%
[22](#ref-22). The model receives complete case evidence rather than
interviewing a patient, and DDXPlus is synthetic.

DDXPlus, released by Tchango and colleagues, contains about 1.3 million
synthetic cases, 49 conditions, and 223 symptom or medical-history items. Each
case includes binary, categorical, or multiple-choice evidence, a differential
list, and a ground-truth condition [23](#ref-23). A proprietary knowledge base
and diagnostic engine generated the cases. The dataset supports large,
repeatable comparisons, but its disease list and evidence patterns come from
that simulation.

High classification scores on these benchmarks do not show how a model’s
probabilities behave in an open patient population. Most papers report
classification or ranking results, not whether cases assigned a 20% disease
probability have that disease about 20% of the time.

Guo and colleagues found that modern neural networks can be overconfident
despite high classification accuracy. On image-classification benchmarks,
temperature scaling reduced calibration error while retaining accuracy
[44](#ref-44). Symptom-checker studies rarely test whether predicted disease
probabilities match observed disease frequencies.

## 3. Interactive diagnosis and question-selection methods

### Dialogue as a sequential task

A dialogue model selects questions as it updates its disease ranking, then
decides when it has enough information to return a differential. A standard
classifier receives all symptoms at once.

Wei and colleagues introduced a task-oriented diagnosis dialogue system in 2018.
It learned a policy for asking about symptoms and predicting a disease from
online patient-doctor conversations. Their MuZhi dataset contained 710 pediatric
dialogues, four diseases, and 66 symptoms. In simulation, a deep Q-network
reached a task-success score of 0.65, compared with 0.23 for a rule-based policy
and 0.06 for a random policy. It completed the task in about five turns; the
other systems used about 17 [24](#ref-24). The result applies to a small
pediatric disease set and a simulated dialogue.

BR-Agent combined a disease-symptom Bayesian network with reinforcement
learning. It ranked questions by how typical a symptom was for a disease and how
well it separated competing diagnoses. On Dxy, it reported 84.6% disease
accuracy and 48.6% recall of implicit symptoms; on MuZhi, 76% accuracy and 67%
recall; on a 12-disease hospital dataset, 82% accuracy and 50% recall
[25](#ref-25). Human raters also scored question and diagnosis rationality.
These evaluations used structured symptom evidence; they did not test open-ended
patient language.

Diaformer generates a sequence of symptoms followed by a diagnosis. Its
transformer uses symptom attention and orderless training to handle the fact
that patients can report symptoms in any order. On three public dialogue
datasets, it improved diagnosis accuracy over its baselines by 1, 6, and 11.5
percentage points, with lower training cost than the reinforcement-learning
models in the comparison [26](#ref-26). The gain ranged from 1 to 11.5 points
across datasets.

DxFormer separates symptom inquiry from disease diagnosis. Its inquiry component
generates questions, while its diagnosis component ranks diseases from the
answers collected so far. On Dxy, MZ-4, and MZ-10, it reported diagnostic
accuracies of 81.7%, 74.3%, and 63.3%, with averages of 6.32, 8.74, and 7.58
questions. The best baseline scored 80.2%, 73.1%, and 50.5% [27](#ref-27). The
accuracy gains were 1.5, 1.2, and 12.8 percentage points. The datasets contain
527, 1,733, and 4,116 dialogues and cover a limited number of conditions.

He and Chen studied how reinforcement-learning diagnosis systems scale as the
number of symptoms and diseases grows. Their MMF-AC model uses a
variational-autoencoder actor to choose questions and a diagnosis model as its
critic. On MuZhi it reached 73.9% accuracy with 4.47 questions, compared with
73.1% and 8.80 questions for the strongest baseline. On Dxy it reached 82.7%
accuracy with 6.08 questions, compared with 80.2% and 7.49 questions. On a
synthetic 1,000-disease task, MMF-AC had lower top-one accuracy than the
baseline (63.04% versus 67.03%), though its top-three and top-five scores were
slightly higher [47](#ref-47).

### Differential diagnosis and the cost of questions

On DDXPlus, the paper compared systems trained on the single correct condition
with systems trained on a full differential list. For one model, differential F1
rose from 31.28% to 78.24%, while the average number of dialogue turns rose from
6.73 to 25.75. The paper reports a similar F1 increase for a second method
[23](#ref-23). These results use synthetic cases; the benchmark does not weight
severe and mild diseases by urgency.

Dialogue studies report final disease accuracy, recall of unreported symptoms,
number of turns, and differential precision or recall. The measures capture
different outcomes: asking more questions can improve accuracy, while shorter
dialogues can miss evidence. Disease sets, patient simulators, reward functions,
and stopping rules also vary, so scores across these papers are not directly
comparable.

Dxy and MuZhi draw on Chinese online health conversations; MZ-4 and MZ-10 cover
small pediatric disease groups. Even when a dataset contains historical
dialogues, model-user exchanges are often simulated from recorded symptoms.
DDXPlus offers more cases and evidence types, but all are generated. Current
benchmarks provide little evidence about unrestricted patient language or real
consultations.

## 4. Large language models and clinical-use evaluations

### Static question answering and simulated conversations

Large language models can generate a ranked differential from a written case. A
static test supplies the full patient history; it does not test history-taking.
CRAFT-MD compared these tasks across 12 specialties using simulated
doctor-patient dialogues. On free-response questions, GPT-4 scored 48.6% with a
full vignette and 26.4% when it had to collect details over multiple turns.
GPT-3.5 scored 37.5% and 16.9%, respectively [28](#ref-28). The benchmark tests
simulated cases, and results vary by answer format.

AMIE, developed by Tu, Natarajan, and colleagues, is an LLM system optimized for
diagnostic dialogue. It used real and simulated medical dialogues, automated
self-play feedback, and inference-time reasoning. In a randomized, blinded
crossover study, 20 primary-care physicians and AMIE completed text
consultations with trained patient actors across 159 scenarios from Canada, the
United Kingdom, and India. Specialists rated AMIE higher on 30 of 32 axes, and
patient actors rated it higher on 25 of 26 axes. AMIE also had higher top-k
differential accuracy [29](#ref-29). The actors followed prepared scenarios, and
synchronous text chat was unfamiliar to the physicians. The study tests
controlled history-taking, not consultations with real patients in routine care.

McDuff and colleagues evaluated a model optimized for differential diagnosis on
302 challenging cases from the New England Journal of Medicine. The model’s
top-ten differential included the correct diagnosis in 59.1% of cases, compared
with 33.6% for clinicians who received the case without the model. In a separate
experiment, model assistance raised clinicians’ top-ten accuracy from 36.1% to
51.7% [30](#ref-30). These selected retrospective cases often involve rare or
complex conditions. Top-ten inclusion does not measure performance on short
symptom reports or the safety of advice to patients.

### Real records and prospective use

Hager and colleagues tested several LLMs on 2,400 cases derived from MIMIC-IV.
The task required models to make decisions across a clinical workflow for four
abdominal conditions, including test selection, follow-up steps, and treatment
decisions. The authors found performance gaps, weak robustness, and failures to
follow instructions. They concluded that the tested models were not suitable for
autonomous clinical decision-making [31](#ref-31). The study replays a sequence
of decisions on clinical-record cases; it does not observe prospective use.

In 2026, Brodeur and colleagues reported a prospective, single-centre
feasibility study of AMIE in urgent care. One hundred adults used a text chat up
to five days before their appointment. Human safety supervisors monitored all
sessions and did not stop any consultation. At eight-week chart review, AMIE’s
differential included the final diagnosis in 90% of cases and ranked it in the
top three in 75%. Blinded reviewers rated AMIE and primary-care physicians
similarly on differential quality and management safety. Physicians’ plans
scored better for practicality and cost [32](#ref-32). The preprint reports
feasibility in 100 users; it has no randomized control group.

A larger randomized trial tested an LLM-based clinical decision-support system
in 16 primary-care facilities in Kenya. The system supported clinical officers
inside an electronic medical record. Among 9,691 patients, treatment failure
within 14 days occurred in 2.2% of the LLM group and 2.0% of the control group;
the difference was not statistically significant. The study found no serious
intervention-related safety signal [33](#ref-33). A secondary review found
better documentation and recorded diagnosis quality in the assisted group. The
trial measured clinician support in routine care, not public use of a symptom
app.

SymptomAI is a 2026 investigational system for patient interviews and
differential diagnosis. In a randomized Fitbit-app study, 13,917 participants
interacted with one of five interview agents. Of these participants, 1,228 later
reported a clinician-provided diagnosis; a clinician panel reviewed 517 cases.
SymptomAI differentials received higher accuracy ratings than differentials from
clinicians who read the same dialogue (odds ratio 2.47, p<0.001). Agents that
actively asked follow-up questions also outperformed a user-led chat baseline
[45](#ref-45). The preprint relies on self-reported diagnoses; clinicians rated
fixed transcripts rather than conducting interviews, and app outputs were for
research only.

DxDirector-7B plans a clinical work-up and requests physician input for tests or
examinations. Its Nature Communications study evaluated 26,018 cases and
reported higher full-process diagnostic accuracy than larger language models,
including GPT-4o and o1-preview [46](#ref-46). The evaluation was retrospective
and clinician-in-the-loop; it did not test a patient-facing symptom checker.

### What clinical-use studies measure

Study design affects what each result can show. CRAFT-MD and actor-based AMIE
tested history-taking in simulated dialogue; the urgent-care AMIE study tested
feasibility and diagnostic ratings with 100 real users. The Berlin randomized
trial found no significant effect on patient-physician interaction. The Kenya
trial found no significant difference in treatment failure, while SafeMom
reported increased care-seeking without a control group.

## Synthesis, research groups, and available resources

Diagnosis ranking and triage measure different outcomes: a correct disease in a
list does not guarantee suitable care advice. Results also depend on the test
data. Synthetic and balanced vignette sets support repeatable comparisons, but
do not represent the range or frequency of disease in a patient population.
Dialogue performance depends on how systems collect information, yet many
studies start with a complete case. Most papers report ranking or task accuracy;
few test calibrated disease probabilities or effects on health outcomes.
Patient-facing outcome studies remain rare.

The teams represented here include the Massachusetts General Hospital and
Harvard group behind DXplain; Harvard and Boston researchers who studied public
checkers; Imperial College London researchers who led symptom-checker reviews;
Charité and TU Berlin researchers who studied app triage and developed SCARF;
and Brown University researchers who evaluated checkers with
emergency-department patients. Dialogue-model research includes teams at Harbin
Institute of Technology and Peng Cheng Laboratory. Mila researchers released
DDXPlus, and Google teams developed CRAFT-MD, AMIE, and SymptomAI. Authors whose
work appears across these studies include G. O. Barnett, Hannah Semigran, Ateev
Mehrotra, William Wallace, Marvin Kopka, Markus Feufel, Hamish Fraser, Megan
Ranney, Junying Chen, Arsene Fansi Tchango, Pranav Rajpurkar, Tao Tu, Vivek
Natarajan, Joseph Breda, and Daniel McDuff.

Available datasets and resources include:

- **Disease Symptom Prediction** — the structured Kaggle dataset named in the
  project brief [16](#ref-16).
- **Symptom2Disease** — 1,200 labeled text descriptions from Kaggle
  [18](#ref-18).
- **DDXPlus** — the dataset, task description, and evaluation code for synthetic
  differential diagnosis [23](#ref-23).
- **MIMIC-IV-ED** — emergency-department records for approved research. Access
  requires credentialing and training [34](#ref-34).
- **RepVig** — a method for creating use-case-specific clinical vignettes
  [35](#ref-35).
- **SCARF** — a framework and checklist for reporting symptom-checker
  evaluations [36](#ref-36).
- **symptomcheckR** — an R package for calculating and plotting symptom-checker
  diagnostic and triage measures [37](#ref-37).
- **MedlinePlus Web Service** — National Library of Medicine health-topic
  information and links [38](#ref-38).
- **scikit-learn probability calibration guide** — descriptions of calibration
  curves and methods for machine-learning outputs [39](#ref-39).
- **PyTorch** and **Hugging Face Transformers** — commonly used software for
  machine-learning and language-model research [40](#ref-40), [41](#ref-41).

Stern, Cifu, and Altkorn’s _Symptom to Diagnosis: An Evidence-Based Guide_
covers clinical reasoning from symptoms [42](#ref-42). Greenes’s _Clinical
Decision Support_ covers knowledge-based and data-driven support systems
[43](#ref-43).

## AI disclosure

Codex GPT-6 Luna Extra High with research subagents was used in order to create
a rough initial draft of this state of the art. It was used in order to help
with the initial literature search of relevant papers/projects and their
results.

The initial prompt was:

```text
Make an initial state of the art for the disease prediction app. You can find basic infos in [project-info.txt](docs/organizatorial/project-info.txt). For context, the task of this lab is: "On the theme set by the scientific coordinator you make the state-of-the-art (a document(1/2 pages per team member), which to put on GoogleDocs/GitHub/Bitbucket/etc., in Englishwith):• What did the others, techniques and methodologies used, evaluation methods,results;• Important names in the field, research teams;• Related Articles and books;• Relevant links;• Resources and tools available." Update [state-of-the-art.md](docs/state-of-the-art.md). Keep the text profesional, simple and clean (use ASD-STE100 Simplified Technical English ). Search with multiple subagents on the web for relevant papers/articles/datasets/applications etc and make a good initial state of the art which we will then review.
```

This prompt created a initial draft which than needed some refinemants, as the
AI introduced unrelevant information and projects which were out of scope. After
cleaning up, refining the text and reviewing the references, we introduced some
more papers which we found by searching for relevant papers using **Google
Scholar**, by using the keywords like "symptom checker" and "disease
prediction".

Some examples of found papers include:

- _The diagnostic and triage accuracy of digital and online symptom checker
  tools: a systematic review (2022)_ [3](#ref-3)
- _Diagnostic and triage accuracy and usability of a symptom checker in an
  emergency department: observational study (2022)_ [11](#ref-11)
- _Evaluation of symptom checkers for self diagnosis and triage: audit study
  (2015)_ [2](#ref-2)
- _How to Evaluate the Accuracy of Symptom Checkers and Diagnostic Decision
  Support Systems: Symptom Checker Accuracy Reporting Framework (SCARF)_
  [36](#ref-36)
- etc.

We also added citations to sites like:

- [MedlinePlus Web Service](https://www.nlm.nih.gov/medlineplus/webservices.html)
  [38](#ref-38)
- [scikit-learn probability calibration guide](https://scikit-learn.org/stable/auto_examples/calibration/plot_calibration_curve.html)
  [39](#ref-39)
- [PyTorch](https://pytorch.org/) [40](#ref-40)
- etc.

````text
We then followed up with:

```text
Make the refferences clickable (to send you to the section bellow).
````

We then wanted to ensure that the structure of the state of the art was clear
and easy to read, so we asked the AI to read the current document and suggest
improvements to the structure of the state of the art, seeking from examples of
well known state of the art documents. The prompt was:

```text
Search the web on how to structure a good state of the art document. Suggest improvements to the structure of the current document. Do not edit the file, return here
```

With the advice we restructured the document, breaking up the found papers into
grouped sections. We also did one final proof read to ensure all is consistent and clear.

## References

<a id="ref-1"></a> 1\. Barry ES, Merkebu J, Varpio L. “State-of-the-art
literature review methodology: A six-step approach for knowledge synthesis.”
_Perspectives on Medical Education_. 2022;11:281–288.
[Article](https://link.springer.com/article/10.1007/s40037-022-00725-9).

<a id="ref-2"></a> 2\. Semigran HL, Linder JA, Gidengil C, Mehrotra A.
“Evaluation of symptom checkers for self diagnosis and triage: audit study.”
_BMJ_. 2015;351:h3480. [Article](https://www.bmj.com/content/351/bmj.h3480).

<a id="ref-3"></a> 3\. Wallace W, Chan C, Chidambaram S, et al. “The diagnostic
and triage accuracy of digital and online symptom checker tools: a systematic
review.” _npj Digital Medicine_. 2022;5:118.
[Article](https://www.nature.com/articles/s41746-022-00667-w).

<a id="ref-4"></a> 4\. Riboli-Sasco E, Gerges C, de Bont EGPM, et al. “Triage
and Diagnostic Accuracy of Online Symptom Checkers: Systematic Review.” _Journal
of Medical Internet Research_. 2023;25:e43803.
[Article](https://www.jmir.org/2023/1/e43803/).

<a id="ref-5"></a> 5\. Schmieding M, Kopka M, Schmidt K, Schulz-Niethammer S,
Balzer F, Feufel MA. “Triage Accuracy of Symptom Checker Apps: 5-Year Follow-up
Evaluation.” _Journal of Medical Internet Research_. 2022;24(5):e31810.
[Article](https://www.jmir.org/2022/5/e31810/).

<a id="ref-6"></a> 6\. Hill MG, Sim M, Mills B. “The quality of diagnosis and
triage advice provided by free online symptom checkers and apps in Australia.”
_Medical Journal of Australia_. 2020;212(11):514–519.
[Article](https://pubmed.ncbi.nlm.nih.gov/32391611/).

<a id="ref-7"></a> 7\. Gilbert S, Mehl A, Baluch A, et al. “How accurate are
digital symptom assessment apps for suggesting conditions and urgency advice? A
clinical vignette comparison to GPs.” _BMJ Open_. 2020;10:e040269.
[Article](https://bmjopen.bmj.com/content/10/12/e040269).

<a id="ref-8"></a> 8\. Ceney A, Tolond S, Glowinski A, Marks B, Swift S, Palser
T. “Accuracy of online symptom checkers and the potential impact on service
utilisation.” _PLOS ONE_. 2021;16(7):e0254088.
[Article](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0254088).

<a id="ref-9"></a> 9\. Hammoud M, Douglas S, Darmach M, Alawneh S, Sanyal S,
Kanbour Y. “Evaluating the Diagnostic Performance of Symptom Checkers: Clinical
Vignette Study.” _JMIR AI_. 2024;3:e46875.
[Article](https://ai.jmir.org/2024/1/e46875).

<a id="ref-10"></a> 10\. Chan F, Lai S, Pieterman M, Richardson L, Singh A,
Peters J, et al. “Performance of a new symptom checker in patient triage:
Canadian cohort study.” _PLOS ONE_. 2021;16(12):e0260696.
[Article](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0260696).

<a id="ref-11"></a> 11\. Fraser H, Coiera E, Wong D, et al. “Evaluation of
Diagnostic and Triage Accuracy and Usability of a Symptom Checker in an
Emergency Department: Observational Study.” _Journal of Medical Internet
Research_. 2022. [Article](https://pubmed.ncbi.nlm.nih.gov/36121688/).

<a id="ref-12"></a> 12\. Schmieding ML, et al. “Impact of a Symptom Checker App
on Patient-Physician Interaction Among Self-Referred Walk-In Patients in the
Emergency Department: Multicenter, Parallel-Group, Randomized, Controlled
Trial.” _Journal of Medical Internet Research_. 2025;27:e64028.
[Article](https://www.jmir.org/2025/1/e64028/).

<a id="ref-13"></a> 13\. Barnett GO, Hoffer EP, Packer MS, et al.
“DXplain—demonstration and discussion of a diagnostic decision support system.”
_Proceedings of the Annual Symposium on Computer Application in Medical Care_.
1992:822. [PubMed record](https://pubmed.ncbi.nlm.nih.gov/1483000/).

<a id="ref-14"></a> 14\. “An AI-based symptom checker integrated into MomConnect
maternal healthcare platform: a prospective, observational mixed-methods study.”
_Nature Health_. 2026.
[Article](https://www.nature.com/articles/s44360-026-00125-x).

<a id="ref-15"></a> 15\. Kopka M, von Kalckreuth N, Feufel MA. “Accuracy of
online symptom assessment applications, large language models, and laypeople for
self-triage decisions.” _npj Digital Medicine_. 2025;8:178.
[Article](https://www.nature.com/articles/s41746-025-01566-6).

<a id="ref-16"></a> 16\. Kaggle.
[Disease Symptom Prediction dataset](https://www.kaggle.com/datasets/itachi9604/disease-symptom-description-dataset).

<a id="ref-17"></a> 17\. Fuster-Palà A, Luna-Perejón F, Miró-Amarante L,
Domínguez-Morales M. “Optimized Machine Learning Classifiers for Symptom-Based
Disease Screening.” _Computers_. 2024;13(9):233.
[Article](https://www.mdpi.com/2073-431X/13/9/233).

<a id="ref-18"></a> 18\. Kaggle.
[Symptom2Disease dataset](https://www.kaggle.com/datasets/niyarrbarman/symptom2disease).

<a id="ref-19"></a> 19\. Hassan E, Abd El-Hafeez T, Shams MY. “Optimizing
classification of diseases through language model analysis of symptoms.”
_Scientific Reports_. 2024;14:1507.
[Article](https://www.nature.com/articles/s41598-024-51615-5).

<a id="ref-20"></a> 20\. Guan X, Baral C. “A Bayesian Approach for Medical
Inquiry and Disease Inference in Automated Differential Diagnosis.” 2021.
[Preprint](https://arxiv.org/abs/2110.08393).

<a id="ref-21"></a> 21\. Jones AM, Jones DR. “A Novel Bayesian General Medical
Diagnostic Assistant Achieves Superior Accuracy With Sparse History: A
Performance Comparison of 7 Online Diagnostic Aids and Physicians.” _Frontiers
in Artificial Intelligence_. 2022;5:727486.
[Article](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2022.727486/full).

<a id="ref-22"></a> 22\. Alam N, et al. “DDxT: Deep Generative Transformer
Models for Differential Diagnosis.” 2023.
[Preprint](https://arxiv.org/abs/2312.01242).

<a id="ref-23"></a> 23\. Tchango AF, Goel R, Wen Z, Martel J, Ghosn J. “DDXPlus:
A New Dataset For Automatic Medical Diagnosis.” _Advances in Neural Information
Processing Systems_. 2022.
[Paper](https://proceedings.neurips.cc/paper_files/paper/2022/hash/cae73a974390c0edd95ae7aeae09139c-Abstract.html).
[Dataset site](https://mila-iqia.github.io/ddxplus/).

<a id="ref-24"></a> 24\. Wei Z, Liu Q, Peng B, et al. “Task-Oriented Dialogue
System for Automatic Diagnosis.” _Proceedings of ACL_. 2018.
[Paper](https://aclanthology.org/P18-2033/).

<a id="ref-25"></a> 25\. Guan X, et al. “BR-Agent: A Bayesian Network-Based
Reinforcement Learning Agent for Medical Diagnosis.” _Proceedings of
IJCAI_. 2022. [Paper](https://www.ijcai.org/proceedings/2022/0592.pdf).

<a id="ref-26"></a> 26\. Chen J, Li D, Chen Q, Zhou W, Liu X. “Diaformer:
Automatic Diagnosis via Symptoms Sequence Generation.” _Proceedings of the AAAI
Conference on Artificial Intelligence_. 2022;36(4):4432–4440.
[Paper](https://ojs.aaai.org/index.php/AAAI/article/view/20365).

<a id="ref-27"></a> 27\. Chen J, et al. “DxFormer: A Decoupled Automatic
Diagnostic System Based on Decoder-Encoder Transformer with Dense Symptom
Representations.” _Bioinformatics_. 2023;39(1):btac744.
[Article](https://academic.oup.com/bioinformatics/article/39/1/btac744/6835407).

<a id="ref-28"></a> 28\. Johri S, Jeong J, Tran BA, et al. “An evaluation
framework for clinical use of large language models in patient interaction
tasks.” _Nature Medicine_. 2025;31:77–86.
[Article](https://www.nature.com/articles/s41591-024-03328-5).
[Code and benchmark](https://github.com/rajpurkarlab/craft-md).

<a id="ref-29"></a> 29\. Tu T, Schaekermann M, Palepu A, et al. “Towards
conversational diagnostic artificial intelligence.” _Nature_. 2025;642:442–450.
[Article](https://www.nature.com/articles/s41586-025-08866-7).

<a id="ref-30"></a> 30\. McDuff D, Schaekermann M, Tu T, et al. “Towards
accurate differential diagnosis with large language models.” _Nature_. 2025.
[Article](https://www.nature.com/articles/s41586-025-08869-4).

<a id="ref-31"></a> 31\. Hager P, Jungmann F, Holland R, et al. “Evaluation and
mitigation of the limitations of large language models in clinical
decision-making.” _Nature Medicine_. 2024;30:2613–2622.
[Article](https://www.nature.com/articles/s41591-024-03097-1).

<a id="ref-32"></a> 32\. Brodeur P, Koshy JM, Palepu A, et al. “A prospective
clinical feasibility study of a conversational diagnostic AI in an ambulatory
primary care clinic.” 2026. [Preprint](https://arxiv.org/abs/2603.08448).

<a id="ref-33"></a> 33\. “Generative AI-enabled clinical decision support system
in primary care: a pragmatic, cluster-randomized trial.” _Nature Medicine_.
2026;32:3032–3039.
[Article](https://www.nature.com/articles/s41591-026-04503-6).

<a id="ref-34"></a> 34\. Johnson AEW, Bulgarelli L, Shen L, et al. “MIMIC-IV, a
freely accessible electronic health record dataset.” _Scientific Data_.
2023;10:1. [Dataset paper](https://www.nature.com/articles/s41597-022-01899-x).
[MIMIC-IV-ED data](https://physionet.org/content/mimic-iv-ed/).

<a id="ref-35"></a> 35\. Kopka M, von Kalckreuth N, Feufel MA. “The RepVig
framework for designing use-case specific representative vignettes and
evaluating triage accuracy of laypeople and symptom assessment applications.”
_Scientific Reports_. 2025.
[Article](https://www.nature.com/articles/s41598-024-83844-z).

<a id="ref-36"></a> 36\. Kopka M, Feufel MA. “How to Evaluate the Accuracy of
Symptom Checkers and Diagnostic Decision Support Systems: Symptom Checker
Accuracy Reporting Framework (SCARF).” _JMIR Human Factors_. 2026;13:e76168.
[Article](https://humanfactors.jmir.org/2026/1/e76168).

<a id="ref-37"></a> 37\. CRAN.
[symptomcheckR package](https://cran.r-project.org/package=symptomcheckR).

<a id="ref-38"></a> 38\. U.S. National Library of Medicine.
[MedlinePlus Web Service documentation](https://medlineplus.gov/about/developers/webservices/).

<a id="ref-39"></a> 39\. scikit-learn developers.
[Probability calibration documentation](https://scikit-learn.org/stable/modules/calibration.html).

<a id="ref-40"></a> 40\. PyTorch.
[Official documentation](https://pytorch.org/docs/stable/index.html).

<a id="ref-41"></a> 41\. Hugging Face.
[Transformers documentation](https://huggingface.co/docs/transformers/index).

<a id="ref-42"></a> 42\. Stern SDC, Cifu AS, Altkorn D. _Symptom to Diagnosis:
An Evidence-Based Guide_. 4th ed. McGraw Hill; 2020.
[Publisher page](https://www.mheducation.com/highered/mhp/product/symptom-diagnosis-evidence-based-guide-fourth-edition.html).

<a id="ref-43"></a> 43\. Greenes RA, ed. _Clinical Decision Support: The Road to
Broad Adoption_. 2nd ed. Elsevier; 2014.
[Publisher page](https://shop.elsevier.com/books/clinical-decision-support/greenes/978-0-12-398476-0).

<a id="ref-44"></a> 44\. Guo C, Pleiss G, Sun Y, Weinberger KQ. “On Calibration
of Modern Neural Networks.” _Proceedings of Machine Learning Research_.
2017;70:1321–1330. [Paper](https://proceedings.mlr.press/v70/guo17a.html).

<a id="ref-45"></a> 45\. Breda J, Yousif F, Hawkins B, et al. “SymptomAI:
Towards a Conversational AI Agent for Everyday Symptom Assessment.” 2026.
[Preprint](https://arxiv.org/abs/2605.04012).

<a id="ref-46"></a> 46\. Xu S, Huang X, Wei Z, et al. “DxDirector: an agentic
large language model driving the full-process clinical diagnosis.” _Nature
Communications_. 2026;17:5614.
[Article](https://www.nature.com/articles/s41467-026-71928-5).

<a id="ref-47"></a> 47\. He W, Chen T. “Scalable Online Disease Diagnosis via
Multi-Model-Fused Actor-Critic Reinforcement Learning.” _Proceedings of the ACM
SIGKDD Conference on Knowledge Discovery and Data Mining_. 2022.
[Paper](https://arxiv.org/abs/2206.03659).
