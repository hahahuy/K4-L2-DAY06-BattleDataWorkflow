# Chest X-Ray Rapid Queue Triage Workflow

## 1. Decision And Safety Boundary

### Objective

> From a de-identified frontal chest X-ray, decide whether to move the study to the top of the radiologist worklist or retain normal queue order, so potentially abnormal cases are read sooner.

- **User:** Radiologist or imaging-worklist coordinator at Vinmec.
- **Unit:** One frontal chest X-ray from one imaging encounter.
- **Output labels:** `priority: suspected abnormality`, `routine: no suspected abnormality`, `unreadable / insufficient technical quality`.
- **Not a diagnosis:** The system does not name a disease, replace the report, suppress an image, or make a treatment decision.
- **Safety choice:** False negative is more costly than false positive.
- **Design consequence:** Set high sensitivity for `suspected abnormality`; accept additional false-positive priority flags; every image still receives a radiologist read.

### Graph To Generate

```text
De-identified frontal CXR -> triage flag -> priority or routine worklist -> radiologist reads every case
```

## 2. Scope And Data Contract

### Include

- Adult frontal chest X-rays only: PA or AP view.
- Public, de-identified datasets only, accessed under their license or data-use agreement.
- Example source: MIMIC-CXR contains de-identified DICOM chest radiographs, reports, patient identifiers, and study identifiers.
- One ledger record per image: `image_id`, `patient_id`, `study_id`, source/dataset version, view position, acquisition period, de-identification check, eligible/quarantine status, split, label version, and annotator/reviewer IDs.
- Exclude lateral-only images, non-chest studies, unreadable/corrupt files, images with suspected residual PHI, and images outside the agreed population.
- Quarantine, never silently delete, any corrupted file or possible PHI leak.

### Privacy Controls

- Verify DICOM metadata are de-identified.
- Scan for burned-in text; quarantine uncertain images for privacy review.
- Use controlled access only: named users, access log, no personal downloads or sharing.
- The dataset card states source license, permitted use, and known limitations.

### Graph To Generate

```text
Public de-identified source -> PHI / format check -> Eligible dataset or Quarantine -> Controlled annotation workspace
```

## 3. Split Before Annotation Leakage

- **Group key:** `patient_id`, not image or study.
- A patient can have repeated X-rays over time. All of that patient's images must be in one split only.
- Create and lock a patient-level `70% train / 10% validation / 20% test` split before bulk annotation.
- Stratify approximately by source, projection, and later by final triage label, but never break a patient group.
- Keep the test set inaccessible to guideline writers and model developers after the split is locked.

### Gate

Zero patient IDs appear in more than one split.

### Graph To Generate

Patient leakage comparison:

- Left: One patient with three X-rays distributed across train, validation, and test, marked `LEAKAGE` in red.
- Right: All three images enter one split, marked `PATIENT-LEVEL SPLIT` in green.

## 4. Guideline: Observable Rules, Not Diagnosis

Use a short, versioned guideline written and approved by radiologists.

| Final label | Operational definition | Action |
|---|---|---|
| `Suspected abnormality` | A qualified radiologist sees one or more image findings that make earlier reading appropriate, even if the exact diagnosis is uncertain. | Move to priority queue |
| `Routine: no suspected abnormality` | No finding visible on the image warrants earlier reading under the approved triage criteria. This does not mean "healthy" or "diagnostically normal." | Keep routine order |
| `Unreadable / insufficient quality` | Projection, exposure, motion, positioning, missing anatomy, or artifact prevents a reliable triage judgment. | Send to technical-quality/manual review |
| `Abstain / escalate` | The annotator cannot safely choose a label under the guideline. | Adjudication queue |

### Explicit Difficult Cases

- Subtle or uncertain opacity: label `suspected abnormality`, not `routine` because it is uncertain.
- Chronic abnormality versus acute abnormality: if earlier reading may be warranted from image alone, label `suspected abnormality`; do not infer patient history.
- Poor inspiration, rotation, under/over-exposure, motion, or missing apices/costophrenic angles: use `unreadable` if reliable triage cannot be made.
- Devices, tubes, or lines: only prioritize when an image-visible concern meets the agreed triage rule; otherwise do not infer clinical urgency.
- A report or AI score may assist sampling, but the primary label comes from independent radiologist image review.

### Gate

A radiologist not involved in writing the guideline applies the rules correctly to a held-out reference set.

### Graph To Generate

```text
Technically readable?
  -> no: Unreadable / manual review
  -> yes: Possible finding requiring earlier read?
      -> yes: Suspected abnormality
      -> no: Routine
```

## 5. Pilot: Spend Expert Time Before Scaling

### Constraint

Each radiologist has only two hours per week. Expert time must be used to calibrate the system, resolve ambiguity, and audit quality, not to duplicate every easy image.

### Pilot Design

- Select 30 images: normal-looking, clear abnormal, subtle abnormal, technically poor, and different sources/projections.
- Two radiologists label independently and blindly.
- Do not show the other radiologist's label, report-derived candidate label, or AI pre-label during the initial read.
- Measure agreement for three labels and inspect every disagreement.
- A senior thoracic radiologist, or pre-agreed adjudication panel, resolves disagreements.
- Convert recurring disagreement into clearer written rules and examples.
- Create a gold/reference set for later QC.

### Pilot Gate

- Agreement threshold set before pilot: weighted Cohen's kappa `>= 0.80`.
- No unresolved recurring disagreement category.
- Every false-`routine` disagreement is reviewed by the adjudicator.
- If the gate fails: revise guideline version, rerun pilot on new cases, and do not begin bulk labeling.

### Graph To Generate

```text
30 diverse cases -> 2 blinded radiologists -> agreement + disagreement clusters
-> adjudication -> guideline v2 + reference set -> gate pass/fail
```

## 6. Bulk Annotation: Efficient And Defensible

### Roles

- **Data steward:** Prepares controlled batches and ledger; never makes clinical labels.
- **Radiologist annotator:** Labels an image independently using the current guideline.
- **Radiologist reviewer:** Audits cases; cannot be the original annotator for that case.
- **Senior radiologist / adjudicator:** Resolves abstentions and disagreements and approves rule changes.
- **Data owner:** Signs a release or puts it on HOLD.

### Preserving Scarce Radiologist Time

- One primary radiologist labels routine cases.
- Mandatory second review for `abstain`, `unreadable`, suspected abnormality, unusual source/scanner, and risk-flagged cases.
- Use existing de-identified reports or an AI model only to create a **sampling/risk queue**, never as unquestioned truth.
- To prevent anchoring, keep suggestions hidden until the radiologist's first independent decision is saved.
- Batch by patient group and source; log guideline version and person responsible for every record.

### Artifact

Candidate labels, abstention/escalation list, batch ledger, and annotation log.

### Graph To Generate

Swimlane diagram with Data Steward, Annotator, Reviewer, Adjudicator, and Data Owner.

## 7. QA/QC: Measure The Right Failure

Do not report only overall accuracy. Normal images dominate; a high average can hide dangerous errors.

### Three Separate QC Streams

| QC stream | Purpose | Sample / denominator | Never use it for |
|---|---|---|---|
| Random audit | Estimate quality of the whole batch | Seeded random sample: `max(10% of batch, 100 images)` | Finding rare failure modes efficiently |
| Safety audit | Detect dangerous false-routine labels | Stratified sample with at least 100 `routine` labels, reviewed independently | Estimating overall batch error |
| Risk queue | Find likely errors | All abstentions, disagreements, low-confidence cases, new sources, poor-quality images | Claiming a population error rate |

### Metrics

- Label error rate by true/adjudicated class, always with denominator.
- `False-routine rate = adjudicated suspected-abnormal images labeled routine / all adjudicated suspected-abnormal images audited`.
- Macro agreement across the three labels.
- Worst-class error rate.
- Confusion matrix, especially `suspected abnormality -> routine`.
- Report all metrics by source, projection, technical quality, and label version.

### Initial Release Gates

- Random-audit macro agreement with adjudicated reference: `>= 0.90`.
- Safety-audit false-routine rate: `<= 2%`, reported with numerator and denominator.
- No systematic error cluster remains unresolved.
- If a gate fails: identify affected rule/source/batch, update the guideline, relabel the defined scope, then re-audit.

The `<= 2%` threshold is a design target, not a claim that the team has clinically validated a product. Vinmec clinical governance must review and approve any production threshold.

### Graph To Generate

A confusion matrix with the red cell `True suspected abnormality -> Labeled routine`; beside it, a bar chart comparing macro, micro, and worst-class performance.

## 8. Release: Evidence Or HOLD

Release only a reproducible versioned packet:

1. Images and final labels.
2. Locked patient-level split manifest.
3. Guideline version and decision log.
4. Annotation and adjudication provenance.
5. QC report: samples, seeds, denominators, confusion matrices, and corrective actions.
6. One-page dataset card: intended triage use, prohibited uses, source/license, coverage gaps, known error patterns, and privacy controls.
7. Named data-owner signature.

### Release Decision

- `RELEASE v1.0` only when all gates pass and evidence is in the packet.
- Otherwise `HOLD`: state what is missing, the named owner, and expected remediation time.

### Graph To Generate

Release checklist with five required packet items and a final `RELEASE / HOLD` decision gate.

## 9. Monitoring And Feedback Loop

After release and model deployment, monitor both data shift and operational safety.

### Monitor Weekly

- Input distribution: source hospital, scanner/vendor, AP/PA projection, image quality, and patient-population proxy where permitted.
- Queue behavior: percentage flagged priority, false-alarm workload, and time-to-radiologist-read for priority versus routine cases.
- Label/model safety sample: independent radiologist review of sampled routine predictions, especially cases later found abnormal.
- Error concentration: source, scanner, projection, technical quality, and abnormality type.

### Trigger Investigation

- New scanner/site or a meaningful source-distribution shift.
- Increase in unreadable images.
- Drop in safety-audit performance.
- Cluster of missed suspected abnormalities.
- Priority queue volume becoming operationally unmanageable.

### Close The Lifecycle Loop

- New scanner distribution issue -> Step 2 collection / Step 3 preparation.
- Repeated ambiguous interpretation -> Step 4 guideline / Step 5 pilot.
- Annotator error cluster -> Step 6 retraining / Step 7 re-audit.
- Prioritization objective no longer matches operations -> Step 1 scope review.

### Graph To Generate

Monitoring feedback loop from deployed triage back to Steps 1, 2, 4, 6, and 7.

## Recommended 10-Minute Slide Structure

| Slide | Content | Time |
|---|---|---:|
| 1 | Clinical problem, objective, safety boundary | 0:45 |
| 2 | Scope, data source, privacy and provenance | 1:00 |
| 3 | Patient-level split and leakage prevention | 0:45 |
| 4 | Label schema and difficult-case decision tree | 1:15 |
| 5 | Radiologist-time constraint and pilot | 1:00 |
| 6 | Bulk workflow and anti-anchoring design | 1:00 |
| 7 | QC streams, metrics, denominators, gates | 1:30 |
| 8 | Release packet and HOLD rule | 0:45 |
| 9 | Monitoring feedback loop | 0:45 |
| 10 | Known weaknesses, mitigations, and six TA answers | 1:15 |

## Known Weaknesses And Mitigations

| Weakness | Mitigation |
|---|---|
| Public data may not represent Vinmec's population or scanners. | Treat the release as research data; conduct site-specific validation before clinical use and monitor source shift. |
| Radiologists are scarce, so full double annotation is expensive. | Double-review high-risk cases, use blinded random and safety audits, and reserve adjudicator time. |
| `Suspected abnormality` is intentionally broad. | Use guideline examples, an abstain path, pilot disagreement analysis, and versioned rule updates. |
| A triage model can create alert fatigue. | Measure priority volume and false alarms; adjust operational thresholds only after clinical-governance review. |
| A high dataset-QC score does not prove model safety. | Separately validate the trained model on a locked patient-level test set, then perform prospective local validation. |

## Direct Answers To TA Questions

1. **Decision:** Prioritize suspected-abnormal frontal chest X-rays for earlier radiologist reading; never diagnose or remove a study from review.
2. **Costlier error:** False negative. Therefore, use conservative abnormal flagging, an abstain route, a safety audit focused on false-routine labels, and a high-sensitivity model acceptance target.
3. **Hard case:** Uncertain subtle opacity. Do not call it routine because of uncertainty; use suspected abnormality or abstain/escalate under the guideline.
4. **Group key:** Patient ID. Lock patient-level 70/10/20 splits before bulk annotation and verify no patient is present across splits.
5. **Quality:** Independent seeded random audit plus stratified safety audit. Report macro/worst-class error and false-routine rate with explicit denominator; reviewer is independent and adjudicator resolves disputes.
6. **People and failure point:** Data steward, radiologist annotator, independent radiologist reviewer, senior adjudicator, and data owner. The weakest point is scarce specialist time; address it with risk-based double review, pilot, and targeted relabeling.

## Research Note

MIMIC-CXR is a relevant example because it provides de-identified DICOM chest radiographs with report, study, and patient identifiers; access remains credentialed and governed by a data-use agreement.

Source: [PhysioNet, MIMIC-CXR v2.1.0](https://physionet.org/content/mimic-cxr/2.1.0/).
