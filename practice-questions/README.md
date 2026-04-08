# Practice Questions — How to Use This Guide
> Google Professional Cloud Architect Exam — 2025 Edition

---

## About This Question Bank

This collection of **77 practice questions** is organized by the six exam domains. Questions are written at PCA difficulty — scenario-based, requiring you to weigh tradeoffs rather than recall facts.

---

## Exam Domain Breakdown

| Domain | Weight | Questions Here |
|---|---|---|
| 1. Designing and planning a cloud solution architecture | ~24% | 12 |
| 2. Managing and provisioning a solution infrastructure | ~15% | 10 |
| 3. Designing for security and compliance | ~18% | 12 |
| 4. Analyzing and optimizing technical and business processes | ~18% | 10 |
| 5. Managing implementation | ~11% | 8 |
| 6. Ensuring solution and operations reliability | ~14% | 10 |
| Case Study Questions (cross-domain) | — | 15 |

---

## How to Approach PCA Exam Questions

### 1. Read the Scenario, Not Just the Question
The PCA exam is scenario-heavy. Key details hidden in the scenario often determine the correct answer:
- "The company has 5 engineers" → operational simplicity matters
- "Regulatory compliance" → think VPC SC, audit logs, data residency
- "Minimize cost" → Spot VMs, Coldline storage, committed use discounts
- "High availability across regions" → global LBs, multi-region configs, Spanner

### 2. Identify the Core Constraint
Every question has a primary driver. Find it:
- **Performance** vs **Cost** vs **Security** vs **Operational Simplicity** vs **Speed to Market**
- Often one answer optimizes for the right constraint while others optimize for the wrong one

### 3. Eliminate Clearly Wrong Answers First
Common wrong answer patterns:
- ❌ Using **Owner/Editor** primitive roles (always wrong for "least privilege" questions)
- ❌ Suggesting **public IPs** when the scenario mentions private/secure networking
- ❌ **Spanner** for small regional workloads (overkill + expensive)
- ❌ **Bigtable** for datasets < 1 TB (inefficient at small scale)
- ❌ Creating **SA key files** when Workload Identity Federation is available
- ❌ Choosing **re-architect** when "minimal disruption" or "fastest migration" is required

### 4. Watch for Qualifiers
These words change the correct answer:
- **"most cost-effective"** → eliminate premium services unless required
- **"least operational overhead"** → prefer fully managed services (Cloud Run > GKE > GCE)
- **"near-real-time"** → streaming pipeline, not batch
- **"global users"** → Global Load Balancer, multi-region config
- **"strongly consistent"** → Spanner, not Bigtable or Firestore with eventual consistency
- **"scale to zero"** → Cloud Run or Cloud Functions
- **"existing Hadoop workloads"** → Dataproc, not Dataflow

### 5. Multi-Select Questions ("Select TWO")
- Both answers must be correct individually AND work together
- Don't pick two answers that contradict each other
- Common pattern: one answer addresses security, another addresses architecture

### 6. Case Study Questions
- Memorize the four official case studies: **EHR Healthcare**, **Altostrat Media**, **Cymbal Retail**, **KnightMotives Automotive**
- Read each company's requirements: technical requirements, business requirements, existing environment
- Questions often test whether you can map requirements to the correct GCP service

---

## Scoring Guide

| Score | Interpretation |
|---|---|
| < 60% | Review the relevant cheat sheets; revisit fundamentals |
| 60–70% | On the borderline; focus on weak domains |
| 70–80% | Good progress; review explanations for any missed questions |
| > 80% | Ready for the exam; take a full-length practice test |

---

## Official Exam Resources

- **Exam Guide**: https://cloud.google.com/certification/guides/professional-cloud-architect
- **Case Studies**: https://cloud.google.com/certification/guides/professional-cloud-architect#case-studies
- **Sample Questions**: https://docs.google.com/forms/d/e/1FAIpQLSf54f7FbtSJcXUY6-DqLoLajm5tl7nTEHoBQaJtMlHqK3SMIA/viewform
- **Practice Exam (paid)**: Google Cloud Skills Boost

---

## Study Tips

1. **Understand, don't memorize** — the exam tests judgment, not recall
2. **Build hands-on labs** — at least one lab per major service
3. **Read the case studies multiple times** — print them out if needed
4. **Time yourself** — 50 questions in 2 hours = ~2.4 minutes/question
5. **Flag and return** — don't spend more than 3 minutes on any one question
6. **Read all four options** before selecting — often two options look similar
