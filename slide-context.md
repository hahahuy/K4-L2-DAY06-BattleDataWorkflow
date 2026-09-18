# Slide Generation Brief: Chest X-Ray Rapid Queue Triage

## Presentation Goal

Create a 10-slide, 10-minute presentation for a data-annotation workflow battle. The team proposes a complete nine-step annotation lifecycle for a Vinmec tool that prioritizes suspected-abnormal frontal chest X-rays for radiologist review.

The presentation must demonstrate operational detail, safety, data governance, edge-case handling, scarce-expert-time planning, measurable quality gates, and honest limitations.

## Non-Negotiable Narrative

- This is **rapid queue triage**, not diagnosis.
- Every image is still read by a radiologist; no study is suppressed.
- A false negative is costlier than a false positive.
- Labels are created or adjudicated by qualified radiologists, not students.
- Use only public de-identified medical data. Do not use Vinmec patient data.
- Group all serial images by `patient_id` to prevent leakage across splits.
- The workflow is a loop; monitoring sends lessons back to named lifecycle stages.

## Visual Style

- Format: 16:9.
- Tone: clinical, trustworthy, precise, operational.
- Palette: white or very light gray base; navy/teal for normal flow; amber for attention; red only for risks, unsafe errors, or failed gates.
- Typography: clean sans serif, high contrast, large enough for a classroom screen.
- Avoid decorative X-ray imagery, generic AI/brain icons, and dense paragraphs.
- Use one main visual per slide, with short supporting text.
- Use the graph briefs in `graph-to-gen.md` for diagram requirements.

## Slide 1: The Decision And Safety Boundary

**Title:** `Nhanh hon, khong thay the bac si`

**Core message:** From a de-identified frontal chest X-ray, the system chooses priority or routine reading order so suspected-abnormal cases are read earlier.

**Show:** G1 Safety-Bounded Triage Workflow.

**Include three short statements:**

- `Triage, not diagnosis`
- `Radiologist reads every image`
- `False negative costs more -> prioritize sensitivity`

**Speaker emphasis:** We never allow the model to suppress an image or make treatment decisions.

## Slide 2: Scope, Provenance, And Privacy

**Title:** `Chi dung du lieu du dieu kien`

**Core message:** Privacy and provenance are gates before annotation begins.

**Show:** G2 Data Governance Funnel.

**Include:**

- Adult AP/PA frontal chest X-rays only.
- Public de-identified source under license/DUA; example: MIMIC-CXR.
- Each image has a ledger record: IDs, source/version, eligibility, split, label version, and responsible people.
- Suspected PHI or bad files go to quarantine, never silent deletion.

## Slide 3: The Leakage Trap

**Title:** `Khong chia theo anh: chia theo benh nhan`

**Core message:** Serial X-rays from the same patient must never cross train, validation, and test splits.

**Show:** G3 Patient-Level Split Prevents Leakage.

**Include:**

- `Group key = patient_id`
- `70% train / 10% validation / 20% test`
- Gate: `0 patient IDs in more than one split`

**Speaker emphasis:** Image-level random split produces falsely optimistic performance.

## Slide 4: Label Rules And Edge Cases

**Title:** `Nhan co the hanh dong, khong phai chan doan`

**Core message:** Observable rules drive a clear triage action and provide an abstain route for uncertainty.

**Show:** G4 Annotation Decision Tree.

**Include a compact label table:**

- `Suspected abnormality -> priority queue`
- `Routine: no suspected abnormality -> normal queue`
- `Unreadable -> technical/manual review`
- `Abstain -> adjudication`

**Mention two edge cases:** subtle opacity is not automatically routine; inadequate positioning/exposure may be unreadable.

## Slide 5: Pilot Before Scale

**Title:** `Dung gio bac si de sua rule truoc, khong sua hang nghin anh sau`

**Core message:** A 30-image pilot catches broken guidelines before expensive bulk annotation.

**Show:** G5 Pilot Calibration Loop.

**Include:**

- Two radiologists label independently and blindly.
- No AI/report suggestion before first decision, to prevent anchoring.
- Adjudicate every disagreement; convert recurring ambiguity into guideline v2.
- Pilot gate: weighted Cohen's kappa `>= 0.80`.

## Slide 6: Annotation Operations Under Expert Scarcity

**Title:** `Hai gio bac si moi tuan: review dung noi de vo`

**Core message:** The process protects limited radiologist capacity while retaining safety.

**Show:** G6 Bulk Annotation Swimlane.

**Include the five roles:** Data steward, radiologist annotator, radiologist reviewer, senior adjudicator, data owner.

**Required operational points:**

- Single independent primary label for routine cases.
- Mandatory second review for suspected abnormality, unreadable, abstain, unfamiliar source/scanner, and risk-flagged cases.
- AI/report can make a risk queue only, never become ground truth.

## Slide 7: QC Is About Dangerous Misses

**Title:** `Accuracy cao van co the khong an toan`

**Core message:** Overall accuracy hides poor performance on the clinically important minority class.

**Show:** G7 QA/QC Safety Dashboard.

**Include:**

- Three separate streams: random audit, safety audit, risk queue.
- Highlight red error: `True suspected abnormality -> labeled routine`.
- Formula: `False-routine rate = dangerous misses / audited suspected-abnormal cases`.
- Always report numerator and denominator, macro metric, worst-class error, and results by source/projection/quality.

**Release gates:**

- Random-audit macro agreement `>= 0.90`
- Safety-audit false-routine rate `<= 2%`
- No unresolved systematic error cluster

**Footer:** `2% is a design target; clinical governance must approve any real deployment threshold.`

## Slide 8: Release Only With Evidence

**Title:** `Khong du bang chung: HOLD`

**Core message:** A dataset release must be reproducible, signed, and evidence-backed.

**Show:** G8 Release Or HOLD Gate.

**Required package:** final labels, patient-level split, guideline and decision log, annotation/adjudication provenance, QC report, dataset card, named data-owner signature.

**Speaker emphasis:** If any artifact or gate is missing, the correct decision is HOLD, with missing item, owner, and deadline stated.

## Slide 9: Monitoring Closes The Loop

**Title:** `Release khong phai la diem ket thuc`

**Core message:** Monitor source shift and operational outcomes, then route each finding to the lifecycle step that can fix it.

**Show:** G9 Monitoring Feedback Loop.

**Monitor weekly:** new scanner/site, AP/PA mix, image quality, priority volume, read-time difference, missed abnormalities, and error clusters.

**Speaker emphasis:** Every issue has one named owner and one destination step: collection/preparation, guideline/pilot, annotation/QC, or objective review.

## Slide 10: Honest Risks And Defense Answers

**Title:** `Chung toi biet quy trinh de vo o dau`

**Core message:** A credible workflow states its limitations and mitigation plan.

**Use a two-column table:**

| Risk | Mitigation |
|---|---|
| Public data differ from Vinmec scanners/population | Local validation before clinical use; monitor source shift |
| Radiologists are scarce | Risk-based second review, targeted audits, senior adjudication |
| Broad suspected-abnormal label | Versioned guideline, pilot, abstain, decision log |
| Alert fatigue | Track priority volume and false alarms; governance-approved threshold changes |
| Dataset QC is not model validation | Locked test evaluation, then prospective local validation |

**Add a bottom strip labelled `Ready for TA questions`:** objective, false-negative cost, difficult-case rule, `patient_id` group key, QC measurement with denominator, roles and weakest point.

## Required Team Contribution Slide Or Footer

Add a final small slide or visible footer that explicitly maps every team member to deliverables. Do not use "everyone did everything." Suggested work packages:

- Objective, scope, and clinical safety boundary
- Data governance, provenance, and patient-level split
- Guideline, labels, and difficult cases
- Pilot and radiologist capacity plan
- Bulk annotation roles and anti-anchoring controls
- QC metrics, gates, release, monitoring
- Slide design, diagrams, timing, and battle preparation

## Delivery Checks

- Export to PDF before battle.
- Keep total talk time under 10 minutes.
- Every member must understand the six TA questions because the TA may select any person.
- Prefer specific gates, owners, artifacts, and edge cases over generic lifecycle descriptions.
