# Graph Briefs For Presentation

Generate the following diagrams in a consistent clinical, minimal style: white background, navy/teal accents, red only for safety risks or failures, clear sans-serif labels, and no decorative medical imagery. Each graph must be readable on a 16:9 presentation slide.

## G1. Safety-Bounded Triage Workflow

**Purpose:** Explain that the system prioritizes reading order, not diagnosis or replacement of radiologists.

```text
De-identified frontal chest X-ray
            |
            v
      AI triage flag
       /           \
      v             v
Priority worklist  Routine worklist
       \           /
        v         v
     Radiologist reads every case
```

**Required callouts:**

- "Not a diagnosis"
- "No image is suppressed"
- Red safety callout: "False negative costs more -> prioritize sensitivity"

## G2. Data Governance Funnel

**Purpose:** Show data provenance, privacy protection, and quarantine instead of silent deletion.

```text
Public de-identified dataset
            |
            v
PHI / burned-in text / format check
          /                 \
         v                   v
Eligible + ledger      Quarantine + privacy review
         |
         v
Controlled annotation workspace
```

**Required fields on eligible-data ledger:** `image_id`, `patient_id`, `study_id`, source/version, view, eligibility, split, label version.

## G3. Patient-Level Split Prevents Leakage

**Purpose:** Contrast the dangerous image-level split with the correct patient-level split.

**Left panel, red:** One patient icon has three serial chest X-rays. Place one in Train, one in Validation, one in Test. Label: "WRONG: image-level split -> leakage".

**Right panel, green:** The same patient and all three X-rays enter only Train. Other patients enter Validation or Test. Label: "CORRECT: patient_id is the group key".

**Footer:** "Gate: 0 patient IDs appear in more than one split."

## G4. Annotation Decision Tree

**Purpose:** Make label decisions operational and expose the mandatory abstain path.

```text
Start: frontal chest X-ray
            |
            v
Technically readable for reliable triage?
     / yes                     \ no
    v                           v
Possible finding requiring    Unreadable / manual technical review
earlier radiologist read?
  / yes       | uncertain       \ no
 v             v                 v
Suspected      Abstain /          Routine: no suspected
abnormality    adjudication       abnormality
```

**Add small examples:** subtle opacity -> suspected abnormality; severe motion / missing anatomy -> unreadable.

## G5. Pilot Calibration Loop

**Purpose:** Show why the team runs a small pilot before expensive bulk annotation.

```text
30 diverse images
        |
        v
2 radiologists label independently and blindly
        |
        v
Agreement score + disagreement clusters
        |
        v
Senior-radiologist adjudication
        |
        v
Guideline v2 + reference set
        |
        v
Gate: weighted kappa >= 0.80?
   / pass                   \ fail
  v                          v
Bulk annotation        revise rules and rerun pilot
```

**Safety callout:** "Every false-routine disagreement is reviewed."

## G6. Bulk Annotation Swimlane

**Purpose:** Show distinct responsibilities and efficient use of scarce radiologist time.

Create five horizontal swimlanes:

1. **Data steward:** controlled batch + ledger, grouped by patient and source.
2. **Radiologist annotator:** independent first label, no report/AI suggestion shown.
3. **Radiologist reviewer:** mandatory second review for suspected abnormality, unreadable, abstain, unusual source, and risk-flagged cases.
4. **Senior adjudicator:** resolves disagreements/abstentions and approves rule changes.
5. **Data owner:** accepts evidence for release or sets HOLD.

**Required anti-anchoring callout:** "Reveal AI/report suggestion only after independent first decision is saved."

## G7. QA/QC Safety Dashboard

**Purpose:** Demonstrate why overall accuracy is unsafe for an imbalanced dataset.

Layout with three blocks:

1. **Random audit:** seeded sample, `max(10% of batch, 100 images)`, estimates whole-batch quality.
2. **Safety audit:** independent review of at least 100 `routine` labels, searches for dangerous false-routine cases.
3. **Risk queue:** all abstentions, disagreements, new sources, low confidence, and poor quality; finds errors but is not a population estimate.

Include a 3x3 confusion matrix. Highlight the cell:

```text
True: suspected abnormality -> Labeled: routine
```

in red, labelled "dangerous miss".

**Release gates shown beside matrix:**

- Random-audit macro agreement `>= 0.90`
- Safety-audit false-routine rate `<= 2%`, always with numerator/denominator
- No unresolved systematic error cluster

## G8. Release Or HOLD Gate

**Purpose:** Show that data releases are evidence-based and reproducible.

Draw a checklist entering a diamond decision:

```text
Final labels + patient-level split + guideline/decision log
+ provenance + QC report + dataset card + data owner signature
                            |
                            v
                  All evidence and gates pass?
                     / yes              \ no
                    v                    v
               RELEASE v1.0       HOLD: missing item,
                                  named owner, due time
```

## G9. Monitoring Feedback Loop

**Purpose:** Establish annotation as a repeating lifecycle, not a one-time delivery.

Center: "Deployed triage workflow". Surround it with four weekly signals:

- Data shift: new site/scanner, AP/PA mix, image quality
- Operations: priority volume, alert burden, read-time difference
- Safety: independently reviewed routine cases and missed abnormalities
- Error clusters: by source, projection, quality, abnormality type

Draw arrows back to lifecycle stages:

- New source/scanner -> Collection and Preparation
- Ambiguous cases -> Guideline and Pilot
- Annotator error cluster -> Annotation and QA/QC
- Operational objective mismatch -> Objective and Scope

**Footer:** "Each finding has one owner and one lifecycle step."
