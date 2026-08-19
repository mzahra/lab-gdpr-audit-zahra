# GDPR audit: AI Entrepreneur Coach

GDPR audit of my Week 5 project, [AI Entrepreneur Coach](https://github.com/mzahra/ai-entrepreneur-coach). Follows the phases in `instructions.md`. I checked the actual code, not just my own notes, before writing this.

---

## Data processing brief

**Does it process personal data?** Yes. It takes a CV, a personality quiz, and budget and time details, and produces a report about that specific person.

**What personal data:**
- Identity and career data from the uploaded CV or LinkedIn PDF: name, location, headline, industry, experience, skills, education.
- TIPI personality quiz answers and the five Big Five trait scores calculated from them.
- Budget (EUR) and time available (hours per week).
- Free text feedback the user types to revise the report (up to 3 rounds).
- System outputs: fit score per idea, AI written summary, reasons, and 90 day roadmap. All personal data about the user.
- Accidental inference risk: the full CV text goes into the AI prompt, not a filtered field list, so age, nationality, or similar could be picked up even though the app never asks for them.

No special category data (Article 9) is asked for on purpose.

**Where it comes from:** entirely from the user, in the same session. No company database, no third party source.

**What it's used for:**
1. Building a structured profile from the CV (OpenAI call).
2. Scoring the TIPI quiz (local, no external call).
3. Finding and ranking business ideas by similarity (Cohere embeddings, Pinecone search).
4. Calculating a fit score per idea (local).
5. Writing the AI text: summary, reasons, roadmap (OpenAI call).
6. Saving the report as PDF and HTML in a local `output` folder, filename includes the real name.
7. Rewriting the text, not the scores, on user feedback (OpenAI call, up to 3 rounds).

**Who processes it:** my own code (runs locally on whoever's machine); OpenAI (profile extraction, report text); Cohere (embeddings for matching); Pinecone (stores only the 463 business idea entries, gets a temporary search vector, does not save user data).

**Where it's stored:** locally, and on disk forever, since there is no delete step anywhere in the app. OpenAI and Cohere are US companies, so purposes 1, 3, and 5 send data outside the EEA.

**Live finding:** two real report files, with my real name, career data, and personality scores, are committed to the public `ai-entrepreneur-coach` GitHub repo (`output/report_Zahra_Moghaddasi_*.pdf/html`). Data made for testing ended up public with no time limit. Needs fixing on its own, outside this audit.

**Automated decisions:** the fit score and idea ranking is automated scoring, but today it only affects the user themselves. They see the ranked ideas, pick one, and can request wording changes before acting. No employer, bank, or other party ever sees or acts on the output. No separate human reviewer exists because the user is both subject and reader.

---

## Phase 1: Personal data inventory

| Data category | Source | Purpose(s) | Retention | Crosses EU border? |
|---|---|---|---|---|
| Name, location, headline | User (CV) | Structured profile | Forever, no delete step | Yes, OpenAI |
| Industry, experience, skills, education | User (CV) | (a) Structured profile | Forever | Yes, OpenAI |
| Same | User (CV) | (b) Idea matching | Not saved, query only | Yes, Cohere, briefly Pinecone |
| TIPI answers | User (quiz) | Big Five scoring | Session + exported report | No, local |
| Big Five scores | Calculated | (a) Trait fit scoring, local | Session | No |
| Same | Calculated | (b) Working style summary prompt | Exported report | Yes, OpenAI |
| Budget, time available | User | Fit score, roadmap | Session + exported report | Yes, OpenAI |
| Feedback text | User | Rewriting report (up to 3x, cumulative) | In memory for session | Yes, OpenAI each round |
| Fit score, AI text (outputs) | System, about user | Delivering the report | Forever, no delete step | N/A, but sourced from OpenAI |
| Exported report file | Generated | Final deliverable | **Forever. No retention rule anywhere.** | N/A |

**Purpose incompatibility flag:** nothing inside the app repurposes data. But the two real report files committed to the public repo are a genuine case: data made for local testing (name, career, personality scores) now sits public with no time limit and no stated purpose. Article 5(1)(b) and (e) failure, needs fixing separately from this audit.

**Retention, generally:** no code path anywhere deletes an uploaded PDF, a report, or session data. Everything persists until a human deletes it by hand.

---

## Phase 2: Role map

| Entity | Role | Activity | DPA in place? |
|---|---|---|---|
| Client / end user | No separate client exists. No accounts, no hosted version. Each user runs their own copy, so **each user is controller of their own data.** | Uploads own data, keeps own report | N/A |
| Zahra Moghaddasi | **Controller**: designed the processing, and separately controller of my own test data (the exposed files above) | Designing the pipeline, running own data through it | N/A with myself, but see vendor rows |
| OpenAI | **Processor**, acts on instructions in the code/prompts | Profile extraction, report text generation | **Not in place.** No DPA reviewed, no retention setting configured |
| Cohere | **Processor** | Embeddings for idea matching | **Not in place.** Same gap |
| Pinecone | **Processor**, only for the temporary query vector; the idea catalogue is not user data | Stores idea index, gets query vector | **Not in place.** `TBD`, Pinecone's own query logging policy not checked |

**International transfers:** OpenAI, Cohere, Pinecone are all US. Transfer mechanism (Data Privacy Framework, SCCs, or other) is `TBD, legal review`, needs checking vendor by vendor before real data flows again.

**Note:** OpenAI and Cohere are processors, not joint controllers, since they don't decide the purpose or means, just execute API calls. This sits alongside the vendor roles in my [AI Act audit](https://github.com/mzahra/audit-project-zahra) of the same project. Biggest gap: **no DPA exists for any processor relationship**, not optional under Article 28, and applies even to a single test run.

---

## Phase 3: Lawful basis assessment

| Purpose | Basis | Why | Flag? |
|---|---|---|---|
| CV reading, profile extraction | Consent, Art. 6(1)(a) | Optional feature, no account or contract, so consent fits better than "necessary for a contract" | Yes, no consent step exists |
| TIPI scoring | Consent, Art. 6(1)(a) | Optional quiz, only powers a feature the user chose | Yes, same gap |
| Idea matching (Cohere/Pinecone) | Consent, Art. 6(1)(a) | Bundled with the consent above, no new purpose added | No, once consent exists |
| AI text generation | Consent, Art. 6(1)(a) | The actual deliverable the user asked for | No |
| Saving the report file | Consent for making it; retention needs its own limit | Making the file is part of the service; keeping it forever is a separate storage limitation issue (Art. 5(1)(e)) | Yes, retention period needed |
| Rewriting from feedback | Consent, Art. 6(1)(a) | Fresh, explicit user action each time | No |

**No legitimate interests used anywhere.** This is profiling (CV plus personality test, combined and scored), exactly the case where GDPR expects an opt in choice, not a default with a buried opt out. Since legitimate interests is not proposed, no LIA is needed.

**No "contract" basis used either:** no account, no terms, no ongoing relationship. Consent is the honest fit for someone choosing to hand over sensitive data for an optional AI feature.

**The real gap:** consent is the right basis on paper, but the app has zero consent mechanism today, no checkbox, no notice. This is not lawful processing yet, just processing with no valid basis behind it.

---

## Phase 4: Risk and rights analysis

**Special category data (Article 9):** not asked for on purpose. But the full CV text goes into the AI prompt unfiltered, so age, nationality, health related gaps, or religious/political affiliations could be picked up by accident. No Article 9 condition applies today since nothing is collected deliberately, but there's no technical safeguard against accidental inference either.

**Automated decision making (Article 22):** the fit score and ranking is automated scoring, but the decision affects only the user, who makes it themselves. No third party ever acts on the output. Article 22's ban likely does not apply today, since the human in the loop is the same person the data is about. Safeguard: computed numbers and AI text are clearly separated in the report. `TBD, legal review` if "similarly significant effect" is read broadly, and this needs a full redo if any organization ever runs the tool for other people.

**DPIA trigger, EDPB's nine criteria:**

| Criterion | Applies? |
|---|---|
| Evaluation or scoring of people | **Yes** |
| Automated decision with significant effect | Probably not today, `TBD` if a deployer scenario emerges |
| Systematic monitoring | No |
| Special category data at scale | No |
| Large scale processing | No, single user, local tool |
| Matching/combining datasets | **Yes**, CV + TIPI + O*NET based idea dataset |
| Vulnerable people | No |
| Innovative technology | **Yes**, LLM based CV and personality profiling |
| Cross border transfer blocking rights | Partly, `TBD` |

Three criteria clearly apply, so a DPIA is recommended (2+ usually means it's needed). Not treating it as strictly mandatory at this small, single user scale, but it becomes required the moment a hosted or multi user version exists.

**Data subject rights friction:**
- **Access:** high friction, no way to tell a user what OpenAI/Cohere/Pinecone hold on them.
- **Erasure:** high friction, no delete function for uploaded files, reports, or upstream vendor data.
- **Object to profiling:** medium friction, no partial opt out, all or nothing flow.

---

## Phase 5: Law stacking check

- **AI Act:** classified as limited risk under Article 50 in my [AI Act audit](https://github.com/mzahra/audit-project-zahra) of this project (interacts with users, generates AI text, no Annex III area). Article 50's duty to label AI content and disclose AI interaction is not the same as GDPR's transparency rule, so it adds a real obligation on top.
- **ePrivacy:** no cookies, tracking pixels, analytics, or device level access found in `app.py`. Not applicable today. Recheck if a hosted version with analytics is ever built.
- **Data Act:** not applicable, no connected product, IoT, or cloud switching data involved.

---

## Phase 6: Compliance memo

**To:** Data Protection Officer (none currently in place, routed to legal counsel)
**From:** Zahra Moghaddasi
**Subject:** GDPR, first pass compliance review, AI Entrepreneur Coach

**Bottom line: proceed with conditions.** The system processes real personal data (CV, personality answers, budget and time) and sends it to three US based processors, with no consent step, no privacy notice, no DPAs, and no retention controls anywhere in the code. None of this needs a redesign, but the system should not process another real person's data until the first two actions below are done.

**Top three actions:**

1. Remove the personal data currently in the public GitHub repo (`output/report_Zahra_Moghaddasi_*.pdf/html`) and clean it from git history. This is a real exposure happening now, not something to plan for later.
2. Add a real consent step and privacy notice before upload, covering what's collected, that OpenAI/Cohere/Pinecone (US based) process it, retention, and how to request deletion.
3. Set up Data Processing Agreements and confirm a valid transfer method (SCCs or the EU-US Data Privacy Framework) with all three vendors, and set a real retention rule for saved reports.

**Residual risks, even after those actions:**
- OpenAI and Cohere may still retain API inputs under their own default settings, outside this project's control, unless retention is explicitly configured with each vendor.
- The full CV text goes to the AI model, not a filtered field list, so sensitive signals like age or nationality can slip into AI generated output by accident, even though the app never asks for them.
- If the tool is ever run by an organization for other people, rather than by individuals for themselves, the lawful basis, the Article 22 analysis, and the AI Act risk tier all need reworking from scratch.

**What this memo is not:** a legal opinion, a DPIA, or a certification of compliance. Points marked `TBD, legal review` should be confirmed by a qualified specialist before this system processes any real person's data beyond my own testing.

---

## Reinforce

**Accountability test, could I show a regulator I'm compliant today?** No.

| Documentation | Exists? |
|---|---|
| DPA (OpenAI, Cohere, Pinecone) | **Missing** |
| Legitimate Interests Assessment | Not applicable, LI not used anywhere |
| DPIA | **Missing** |
| Privacy notice | **Drafted, not live in the app** |
| Retention schedule | **Missing entirely** |
| Records of processing activities | **Missing**, this audit is the closest thing to one |

**Revisiting one TBD, the transfer mechanism:** a lawyer would need, per vendor: current Data Privacy Framework certification status, whether SCCs are already in their standard terms and which module applies, what data actually flows (already answered in Phase 1), and whether a short or zero retention API setting is available. What I can prepare myself, without legal input: pull each vendor's current DPF and DPA/SCC status from their own published pages, and bring that alongside this audit. Turns an open research question into a narrower one a lawyer can answer faster.

---

## Stretch: data protection by design checklist

**Highest risk activity:** sending the full CV text and Big Five scores to OpenAI for profile extraction and report writing. Most sensitive data mix, least control once it leaves.

| Design principle | Current state | Pass / Fail / Unknown |
|---|---|---|
| Data minimisation | Full CV text sent, not just the fields actually needed (skills, industry, experience, education) | **Fail** |
| Purpose binding | Depends entirely on OpenAI's own API terms, not reviewed | **Unknown** |
| Access controls | Fine for local single use, but failed in practice, real reports ended up in a public repo | **Fail** |
| Retention enforcement | No automatic deletion anywhere | **Fail** |
| Subject rights workflow (30 days) | No workflow exists | **Fail** |
| Incident response (72 hour) | Nothing documented, and the exposed files are effectively a live incident this audit found | **Fail** |

**What would need to change:** send only the fields actually needed instead of the full CV text; add a retention rule and automatic cleanup; write a simple subject rights process, even a manual one; document what happens if data is found exposed, since that already happened once.

---

## Draft privacy notice

Starting point only, needs GDPR review before going live.

**What we collect:** your CV or LinkedIn PDF, TIPI quiz answers, budget and time, and any feedback text you type in.

**Why:** to score your traits, match you to business ideas, and write your report and roadmap.

**Who else sees it:** OpenAI (reads your CV, writes your report) and Cohere (searches for matching ideas), both US based. We don't sell your data.

**How long we keep it:** your report is saved locally as PDF and HTML. Nothing deletes it automatically yet, a known gap. Until it's fixed, treat anything you upload or generate as data you're responsible for deleting yourself.

**Your rights:** ask what we hold, ask us to delete it, or ask questions, at [contact email]. Deletion today means removing your local files by hand; we don't yet have a process for requesting deletion from OpenAI or Cohere.

**One more thing:** your report is AI written, based on what you give us. Check it yourself before acting on it.
