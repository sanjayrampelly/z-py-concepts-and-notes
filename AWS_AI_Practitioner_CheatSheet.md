# AWS AI Practitioner — Complete Cheat Sheet
### All 5 Domains | Daily Revision Guide
---

## EXAM OVERVIEW

| Domain | Weight |
|---|---|
| Domain 3 — Applications of Foundation Models | 28% |
| Domain 2 — Fundamentals of Generative AI | 24% |
| Domain 1 — Fundamentals of AI and ML | 20% |
| Domain 4 — Guidelines for Responsible AI | 14% |
| Domain 5 — Security, Compliance & Governance | 14% |

> **Passing Score:** ~70% | **Questions:** 65 | **Time:** 90 minutes

---

# DOMAIN 3 — Applications of Foundation Models (28%)

## What is Amazon Bedrock?
Fully managed AWS service giving access to pre-built Foundation Models via API.
No servers to manage. No ML expertise needed. Pay per token.

**Models on Bedrock:**
| Provider | Model |
|---|---|
| Anthropic | Claude 2, 3, 3.5 |
| Meta | Llama 2, Llama 3 |
| Amazon | Titan Text, Titan Embeddings |
| Mistral | Mistral, Mixtral |
| Stability AI | Stable Diffusion (images) |
| Cohere | Command, Embed |

---

## 3 Ways to Customize a Model

| Method | How it works | Best for | Cost |
|---|---|---|---|
| **Prompt Engineering** | Craft better input text | Quick improvements, simple tasks | Free |
| **RAG** | Connect FM to your documents | Frequently changing data, Q&A bots | Low |
| **Fine-tuning** | Actually retrain model on your data | Specific tone/style/domain language | High |

### Decision Tree:
```
Frequently updated data?         → RAG
Need specific style/behavior?    → Fine-tuning
Just need better outputs?        → Prompt Engineering
Large model + limited resources? → PEFT (fine-tuning variant)
```

---

## Bedrock Key Features

| Feature | Purpose | Exam Trigger |
|---|---|---|
| **Knowledge Bases** | RAG — connect FM to S3 documents | "Answer from company documents" |
| **Agents** | Multi-step task automation with APIs | "Book flight, check calendar, confirm" |
| **Guardrails** | Content filtering + safety controls | "Never discuss X, remove PII" |
| **Model Evaluation** | Test and compare FM performance | "Compare two models" |
| **Provisioned Throughput** | Reserved capacity for high volume | "Cost optimize high-volume usage" |

---

## Bedrock Guardrails — What it Can Do
- **Denied topics** → "Never discuss competitor products"
- **Content filters** → Block violent/sexual/hate content
- **PII redaction** → Automatically mask SSN, phone numbers, account numbers
- **Grounding checks** → Flag hallucinations not based on source documents
- **Word filters** → Block specific words/phrases

---

## RAG Architecture (Retrieval-Augmented Generation)
```
Documents (S3)
    → Chunking (split into smaller pieces)
    → Embedding (convert to vectors)
    → Store in Vector DB (OpenSearch/Pinecone)

At Query Time:
User Question
    → Embed question
    → Similarity search in Vector DB
    → Retrieve top-k relevant chunks
    → Send [question + chunks] to FM
    → FM answers using YOUR data
```

---

## Bedrock Agents — How They Work
```
User: "Book me a flight to Mumbai"
Agent → Understands intent
      → Calls flight API (Action Group = Lambda)
      → Checks calendar API
      → Confirms availability
      → Books flight
      → Responds: "Done! Flight booked"
```
**Components:** Action Groups (Lambda) + Knowledge Bases + Memory

---

## Prompt Engineering Techniques

| Technique | What it is | Example |
|---|---|---|
| **Zero-shot** | No examples, just instructions | "Classify this as positive/negative: ..." |
| **Few-shot** | Give 2-3 examples first | "positive: great. negative: awful. Now: ..." |
| **Chain-of-thought** | Ask to reason step by step | "Let's think step by step..." |
| **System prompt** | Set role/persona/rules | "You are a helpful financial advisor..." |
| **ReAct** | Reason + Act (used in Agents) | Thought → Action → Observation → Answer |

---

## Inference Parameters

| Parameter | Controls | Low value | High value |
|---|---|---|---|
| **Temperature** | Creativity/randomness | Consistent, factual (0 = deterministic) | Creative, varied, unpredictable |
| **Top-P** | Vocabulary diversity | Focused | Diverse |
| **Top-K** | Token selection pool | Very focused | More varied |
| **Max Tokens** | Response length limit | Short responses | Longer responses |

> **Exam rule:** "Consistent/factual" → Temperature = 0 | "Creative/varied" → Temperature high

---

## Bedrock Pricing
- **On-demand:** Pay per token (input + output). Good for unpredictable traffic.
- **Provisioned Throughput:** Reserve capacity. Cheaper at high predictable volume.
- **Primary cost driver:** NUMBER OF TOKENS PROCESSED

### Cost Optimization Strategies:
1. Shorten system prompts (sent on EVERY request)
2. Use smaller/faster models when possible
3. Provisioned throughput for high-volume apps
4. Reduce max output tokens

---

## Model Evaluation Metrics

| Metric | Used for | What it measures |
|---|---|---|
| **ROUGE** | Summarization | Word/phrase overlap with reference text |
| **BERTScore** | Semantic similarity | Meaning-level match (not exact words) |
| **F1 Score** | Classification | Balance of precision and recall |
| **Perplexity** | Language model quality | How confident/fluent the output is |

> **Exam rule:** "Overlap with reference summary" → ROUGE | "Semantic similarity" → BERTScore

---

## PEFT (Parameter-Efficient Fine-Tuning)

| Aspect | PEFT | Full Fine-tuning |
|---|---|---|
| Parameters updated | Small subset only | ALL parameters |
| Compute cost | Low | Very High |
| Multiple use cases | ✅ Ideal | ❌ Expensive |
| Catastrophic forgetting | Low risk | High risk |
| Best for | Large model + limited resources | Full domain adaptation |

> **PEFT is best when:** Large model + limited budget + multiple tenants/use cases

---

# DOMAIN 2 — Fundamentals of Generative AI (24%)

## AI Hierarchy
```
Artificial Intelligence (AI)
    └── Machine Learning (ML)
            └── Deep Learning (DL)
                    └── Generative AI (GenAI)
```

---

## Key GenAI Concepts

### Tokens
- Unit of text the model processes (≈ 0.75 words)
- 100 tokens ≈ 75 words
- Bedrock charges per token (input + output)
- Models have a **context window limit** measured in tokens

### Context Window
- Maximum text the model can process at once
- Includes: system prompt + conversation history + user input + response
- Exceeds limit → must use chunking + RAG
- **Exam trigger:** "Document too long for model" → context window limitation

### Embeddings
- Text converted to numerical vectors capturing semantic meaning
- Similar meaning = vectors close together in space
- Used in: RAG pipeline, semantic search, recommendations
- **AWS service:** Amazon Titan Embeddings

### Hallucination
- FM confidently generates false/made-up information
- **Fixes:** RAG grounding, grounding checks (Guardrails), lower temperature, human review (A2I)

### Grounding
- Anchoring FM responses to verified source documents
- Reduces hallucination by forcing model to use real content
- Implemented via RAG + Bedrock Knowledge Bases

---

## Semantic Search vs Keyword Search

| | Keyword Search | Semantic Search |
|---|---|---|
| How it works | Exact word matching | Embedding-based meaning matching |
| "ways to save money" finds "cost reduction"? | ❌ No — different words | ✅ Yes — same meaning |
| Synonyms | ❌ Missed | ✅ Found |
| Used in RAG | ❌ | ✅ |

---

## GenAI Lifecycle
```
1. Define Use Case
2. Select Foundation Model
3. Choose Customization (Prompt/RAG/Fine-tune)
4. Build & Integrate
5. Evaluate (Bedrock Model Evaluation)
6. Deploy
7. Monitor & Improve (CloudWatch + Human Feedback)
```

---

## GenAI Use Cases

| Use Case | AWS Service |
|---|---|
| Text summarization | Bedrock FM |
| Q&A from documents | Bedrock Knowledge Bases |
| Code generation | Bedrock + Amazon Q Developer |
| Chatbots | Bedrock + Amazon Lex |
| Image generation | Stable Diffusion on Bedrock |
| Document text extraction | Amazon Textract + Bedrock |
| Speech to text | Amazon Transcribe |
| Text to speech | Amazon Polly |

---

## GenAI Limitations

| Limitation | Fix |
|---|---|
| Hallucination | RAG + grounding checks |
| Knowledge cutoff | RAG with current data |
| Context window limit | Chunking |
| Bias | Guardrails + diverse training data |
| No memory across sessions | Bedrock Agent memory |

---

## Chunking
**Purpose:** Break large documents into smaller, semantically meaningful segments for efficient indexing and retrieval in RAG.

> NOT compression. NOT training cost reduction. NOT NLP improvement.
> **= Split for retrieval efficiency.**

---

# DOMAIN 1 — Fundamentals of AI and ML (20%)

## Types of Machine Learning

| Type | Data | How it learns | Examples |
|---|---|---|---|
| **Supervised** | Labeled (input + answer) | From correct examples | Spam detection, price prediction |
| **Unsupervised** | Unlabeled | Finds hidden patterns | Customer segmentation, anomaly detection |
| **Reinforcement** | Rewards/penalties | Trial and error | Game AI, RLHF for LLMs |
| **Semi-supervised** | Mix of labeled + unlabeled | Combination approach | When labeling is expensive |
| **Self-supervised** | Unlabeled (creates own labels) | Predicts masked/next tokens | LLM pre-training |

---

## ML Problem Types

| Problem | Output | Example | AWS Service |
|---|---|---|---|
| **Classification** | Category/label | Spam/not spam, disease/no disease | SageMaker |
| **Regression** | Number | House price, sales forecast | SageMaker |
| **Clustering** | Groups | Customer segments | SageMaker |
| **Anomaly Detection** | Outliers | Fraud detection | Lookout for Metrics |
| **Recommendation** | Suggested items | Netflix, Amazon | Amazon Personalize |
| **Time Series** | Future values | Demand forecasting | Amazon Forecast |
| **NLP** | Text understanding | Sentiment analysis | Amazon Comprehend |
| **Computer Vision** | Image understanding | Object detection | Amazon Rekognition |

---

## Overfitting vs Underfitting vs Drift

| Problem | When it happens | Signal | Fix |
|---|---|---|---|
| **Overfitting** | During training | Train accuracy high, test accuracy low (gap from day 1) | More data, simpler model |
| **Underfitting** | During training | Both train AND test accuracy low | More complex model, more training |
| **Data Drift** | After deployment | Was working → then degraded | SageMaker Model Monitor, retrain |

> **Golden rule:** "Was working for months → then degraded" = DRIFT. "Never worked well on new data" = OVERFITTING.

---

## Precision vs Recall — Most Tested

### The 4 Outcomes:
```
TRUE POSITIVE (TP)  → Predicted positive, actually positive ✅
TRUE NEGATIVE (TN)  → Predicted negative, actually negative ✅
FALSE POSITIVE (FP) → Predicted positive, actually negative ❌ (false alarm)
FALSE NEGATIVE (FN) → Predicted negative, actually positive ❌ (missed case)
```

### Formulas:
```
Recall    = TP / (TP + FN)   → "Did we CATCH all real cases?"
Precision = TP / (TP + FP)   → "Were our predictions CORRECT?"
F1 Score  = 2 × (Precision × Recall) / (Precision + Recall)
Accuracy  = (TP + TN) / Total
```

### When to Use Which:
| Scenario | Metric | Why |
|---|---|---|
| Cancer detection | **Recall** | Missing a case (FN) = patient dies |
| Fraud detection | **Recall** | Missing fraud (FN) = millions lost |
| Spam filter | **Precision** | Blocking legitimate email (FP) = bad |
| Legal evidence | **Precision** | Wrong accusation (FP) = unjust |
| Balanced importance | **F1 Score** | Neither FP nor FN can dominate |

> **Memory trick:** Recall = cast wide net (catch ALL fish) | Precision = catch only RIGHT fish

---

## SageMaker Services — Complete Map

| Service | One job | Keyword trigger |
|---|---|---|
| **Canvas** | No-code ML for business analysts | "No coding", "business analyst", "drag and drop" |
| **Ground Truth** | Data labeling (human + automated) | "Label data", "annotate images" |
| **Data Wrangler** | Data preparation and transformation | "Clean data", "transform", "missing values" |
| **Autopilot** | AutoML — auto selects algorithm | "Automatically find best model", "AutoML" |
| **Clarify** | Bias detection + explainability | "Why did model decide X?", "bias", "feature importance" |
| **Model Monitor** | Detects drift in deployed models | "Model degrading", "data drift", "production monitoring" |
| **Pipelines** | End-to-end ML workflow automation | "Automate entire workflow", "recurring ML pipeline" |
| **Feature Store** | Central store for ML features | "Share features across teams", "feature repository" |
| **JumpStart** | Pre-built models and solutions | "Pre-built", "quick start", "model marketplace" |
| **Studio** | Full ML IDE | "Develop ML models", "notebook environment" |

---

## AWS Pre-built AI Services (No ML expertise needed)

### Computer Vision:
| Service | Does | Use case |
|---|---|---|
| **Rekognition** | Analyzes images/video | Face detection, object labels, content moderation |
| **Textract** | OCR + document extraction | Extract text, tables, forms from PDFs |

### NLP:
| Service | Does | Use case |
|---|---|---|
| **Comprehend** | NLP analysis | Sentiment, entities, key phrases |
| **Translate** | Language translation | Multi-language support |
| **Transcribe** | Speech → Text | Call transcription, subtitles |
| **Polly** | Text → Speech | Voice assistants, audiobooks |
| **Lex** | Conversational AI | Build chatbots (powers Alexa) |

### Specialized:
| Service | Does | Use case |
|---|---|---|
| **Forecast** | Time-series forecasting | Demand planning, inventory |
| **Personalize** | Recommendations | Netflix-style recommendations |
| **Kendra** | Intelligent search | Enterprise document search |
| **Fraud Detector** | Fraud detection | Online payment fraud |
| **A2I** | Human-in-the-loop | Human review for edge cases |

---

## Quick Service Triggers — AI Services

| Exam says... | Answer is... |
|---|---|
| OCR / extract text from documents | **Textract** |
| Sentiment / entities / NLP on text | **Comprehend** |
| Speech to text | **Transcribe** |
| Image / face / object detection | **Rekognition** |
| PII/sensitive data discovery in S3 | **Macie** |
| Human review for ML predictions | **A2I** |
| Recommendations ("you may also like") | **Personalize** |
| Time series forecasting | **Forecast** |
| Conversational chatbot | **Lex** |

---

## ML Lifecycle → AWS Services

| Stage | AWS Service |
|---|---|
| Label data | SageMaker Ground Truth |
| Prepare/clean data | SageMaker Data Wrangler |
| Store features | SageMaker Feature Store |
| Train model | SageMaker Training Jobs |
| AutoML | SageMaker Autopilot |
| Tune hyperparameters | SageMaker Hyperparameter Tuning |
| Evaluate + explain | SageMaker Clarify |
| Deploy | SageMaker Endpoints |
| Monitor production | SageMaker Model Monitor |
| Automate pipeline | SageMaker Pipelines |
| No-code ML | SageMaker Canvas |

---

# DOMAIN 4 — Guidelines for Responsible AI (14%)

## The 8 Pillars of Responsible AI

| Pillar | What it means | AWS Tool |
|---|---|---|
| **Fairness** | Equal treatment across all groups, no discrimination | SageMaker Clarify |
| **Explainability** | Understand WHY AI made a decision | SageMaker Clarify |
| **Privacy** | Protect personal data in AI systems | Amazon Macie, KMS |
| **Transparency** | Open about how AI works and its limitations | Model Cards |
| **Safety** | AI should not cause harm | Bedrock Guardrails, A2I |
| **Controllability** | Humans can monitor, correct, override AI | Amazon A2I |
| **Veracity** | Accurate, reliable, consistent outputs | Guardrails grounding |
| **Governance** | Policies and processes for responsible AI use | CloudTrail, Model Cards |

---

## Types of AI Bias

| Bias Type | What it is | Example |
|---|---|---|
| **Data Bias** | Training data doesn't represent reality fairly | Facial recognition trained on light-skinned faces |
| **Algorithmic Bias** | Model amplifies existing data biases | Hiring model discriminates against women |
| **Automation Bias** | Humans over-trust AI without questioning | Doctor blindly follows AI diagnosis |
| **Confirmation Bias** | Model only learns confirming patterns | — |
| **Reporting Bias** | Some events underrepresented in training data | Positive events reported less |

---

## Responsible AI — Key Rules for Exam

```
❌ ALWAYS WRONG in Responsible AI questions:
→ "Rely solely on AI for final decisions"
→ "Eliminate human involvement"
→ "Fully automate high-stakes decisions"
→ "Maximize automation to remove human errors"

✅ ALWAYS RIGHT in Responsible AI questions:
→ Human review for high-stakes decisions
→ Diverse, representative training data
→ Regular bias audits
→ Continuous evaluation with feedback
→ Human-in-the-loop for edge cases
```

---

## Human Oversight — When Required

```
Low stakes → AI decides automatically
             ("Which product image looks better?")

Medium stakes → AI recommends, human reviews
                (Medical diagnosis suggestions)

High stakes → Human MUST decide, AI only assists
              (Criminal sentencing, surgery, loan decisions)
```

> **Exam rule:** Any scenario where AI makes HIGH-STAKES decisions WITHOUT human oversight = WRONG/IRRESPONSIBLE

---

## Responsible AI Tools

| Challenge | AWS Service |
|---|---|
| Detect bias in data/model | SageMaker Clarify |
| Explain model predictions | SageMaker Clarify |
| Human review for edge cases | Amazon A2I |
| Content filtering + safety | Bedrock Guardrails |
| PII detection in S3 | Amazon Macie |
| Audit trail of AI actions | AWS CloudTrail |
| Monitor model fairness over time | SageMaker Model Monitor |
| Document model details/limitations | SageMaker Model Cards |
| Proactive safety vulnerability testing | Red Teaming |

---

## Shared Responsibility for Responsible AI

| AWS Responsible For | Customer Responsible For |
|---|---|
| Infrastructure security | Prompt design and engineering |
| FM security and availability | Input validation and sanitization |
| Bedrock Guardrails feature itself | Configuring and ENABLING Guardrails |
| Building safety into Titan FMs | Ensuring training data is fair |
| Platform compliance certifications | Industry regulation compliance |

> **Exam rule:** Prompt injection = ALWAYS customer's responsibility

---

## Red Teaming
Adversarial testing BEFORE deployment to find AI vulnerabilities.
- Tries to make model produce harmful outputs
- Tests for jailbreaks, bias, hallucinations, PII leakage
- Done by a dedicated "red team" simulating malicious users
- **NOT** benchmarking, NOT fine-tuning, NOT user testing

---

## Model Cards
Document describing everything about a model:
- Intended use and prohibited uses
- Training data description
- Performance metrics across demographic groups
- Known limitations and biases
- Version history

> Think: **Nutrition label for an AI model**

---

# DOMAIN 5 — Security, Compliance & Governance (14%)

## AWS Shared Responsibility Model

```
AWS RESPONSIBLE FOR (Security OF the cloud):
→ Physical data centers
→ Hardware and infrastructure
→ Bedrock service availability
→ Network infrastructure
→ Managed service patching

CUSTOMER RESPONSIBLE FOR (Security IN the cloud):
→ IAM users, roles, permissions
→ Data encryption configuration
→ Application-level security
→ Prompt injection mitigation
→ Guardrails configuration
→ Training data privacy
→ Compliance with industry regulations
→ What data you send to AI services
```

---

## IAM — Identity and Access Management

**Least Privilege Principle:** Give ONLY the minimum permissions needed.

```
❌ BAD: Give developer AdministratorAccess
✅ GOOD: Give developer only SageMaker + specific S3 bucket access

❌ BAD: SageMaker role has full S3 access to all buckets
✅ GOOD: SageMaker role has s3:GetObject for training bucket only
                              s3:PutObject for artifacts bucket only
```

**IAM Components:**
- **Users** — Individual people
- **Roles** — Temporary permissions for services/applications
- **Policies** — Documents defining allowed/denied actions

---

## Data Encryption

| Type | Protects | Services | Exam trigger |
|---|---|---|---|
| **At Rest** | Stored data | S3, DynamoDB, EBS | "Stored data", "S3 bucket encryption" |
| **In Transit** | Moving data | HTTPS/TLS | "During transmission", "between components" |

**AWS KMS (Key Management Service):**
- Manages encryption keys
- Customer Managed Keys (CMK) = YOU control the key
- Can rotate or revoke keys anytime
- Used for S3, SageMaker, DynamoDB encryption

---

## Security Services — Complete Map

| Service | What it does | Exam trigger |
|---|---|---|
| **GuardDuty** | Intelligent threat detection using ML | "Suspicious activity", "compromised credentials", "unusual API calls" |
| **CloudTrail** | Records every API call (who, what, when, where) | "Audit trail", "who did what", "compliance logging" |
| **CloudTrail Lake** | Query and analyze CloudTrail logs with SQL | "Query audit logs", "compliance + CloudTrail data events" |
| **CloudWatch** | Operational monitoring, metrics, alarms | "Monitor performance", "set alarms", "dashboards" |
| **Macie** | PII/sensitive data discovery in S3 | "Find PII in S3", "sensitive data", "PHI discovery" |
| **Shield** | DDoS protection | "DDoS attack protection" |
| **Shield Advanced** | Advanced DDoS + 24/7 response team | "Advanced DDoS", "dedicated support" |
| **WAF** | Web app protection (SQL injection, XSS, bots) | "Web attacks", "SQL injection", "bot traffic" |
| **Inspector** | Vulnerability scanning (EC2, Lambda, ECR) | "Software vulnerabilities", "unpatched packages" |
| **Config** | Continuous resource configuration compliance monitoring | "Is encryption enabled?", "configuration compliance", "resource history" |
| **Security Hub** | Centralized security findings from multiple services | "Single dashboard", "aggregate security findings" |
| **KMS** | Encryption key management | "Manage encryption keys", "customer managed keys" |
| **Artifact** | Download AWS compliance reports/certificates | "PCI DSS certificate", "SOC report", "compliance documents" |
| **VPC Endpoints** | Private connectivity to AWS services | "Never traverse public internet", "private connectivity" |

---

## Most Confused Service Pairs

```
MACIE vs GUARDRAILS:
→ Macie = finds PII in S3 STORAGE (data at rest)
→ Guardrails = filters PII in real-time MODEL RESPONSES (data in motion)

CLOUDTRAIL vs SECURITY HUB:
→ CloudTrail = raw API activity logs ("who called what API")
→ Security Hub = aggregates findings from GuardDuty + Macie + Inspector + Config

CONFIG vs ARTIFACT:
→ Config = monitors YOUR resource configurations for compliance
→ Artifact = downloads AWS's own compliance certificates/reports

CLOUDWATCH vs CLOUDTRAIL:
→ CloudWatch = operational monitoring (metrics, performance, alarms)
→ CloudTrail = security audit logging (who did what and when)

GUARDDUTY vs INSPECTOR:
→ GuardDuty = detects active threats and suspicious behavior
→ Inspector = scans for software vulnerabilities (passive scanning)

AUTOPILOT vs CANVAS vs PIPELINES:
→ Autopilot = AutoML, auto-selects algorithm (needs some ML knowledge)
→ Canvas = No-code ML for business analysts (zero coding)
→ Pipelines = Automates recurring ML workflows (needs engineering knowledge)
```

---

## AI-Specific Security Threats

| Threat | What it is | Fix |
|---|---|---|
| **Prompt Injection** | User overrides system instructions | Guardrails + input validation (Customer's responsibility) |
| **Data Poisoning** | Malicious data injected into training set | Data validation, source verification |
| **Model Inversion** | Reconstruct training data from model | Rate limiting, differential privacy |
| **Model Theft** | Replicate model via extensive querying | API rate limiting, usage monitoring |
| **Adversarial Examples** | Modified inputs that fool the model | Adversarial training, input validation |

---

## Network Security for AI

```
WITHOUT VPC Endpoints:
Your App → Public Internet → AWS Bedrock

WITH VPC Endpoints:
Your App → Private AWS Network → AWS Bedrock
           (never touches public internet)
```

> **Exam trigger:** "Never traverse public internet" + "Bedrock" = VPC Endpoints

---

## Compliance Frameworks

| Framework | Industry | Key Requirement |
|---|---|---|
| **GDPR** | Europe (all industries) | Right to explanation, data erasure, data minimization |
| **HIPAA** | US Healthcare | Protect PHI, encrypt at rest and transit, audit logs |
| **PCI DSS** | Payment/Finance | Protect cardholder data, regular security testing |
| **SOC 2** | Technology/SaaS | Security, availability, confidentiality controls |

> **AWS Artifact** = where you download compliance documents for auditors

---

## Governance for AI

**Key components:**
1. **AI Usage Policies** — Acceptable/prohibited use cases
2. **Model Documentation** — Model Cards
3. **Audit Trails** — CloudTrail + CloudTrail Lake
4. **Access Controls** — IAM least privilege
5. **Incident Response** — Model rollback procedures

---

# MASTER QUICK REFERENCE

## Service → Use Case (One Line Each)

```
Bedrock          → Access pre-built FMs via API
Knowledge Bases  → RAG — connect FM to your documents
Agents           → Multi-step task automation with APIs
Guardrails       → Content filtering and safety controls
A2I              → Human review for ML edge cases
Textract         → Extract text from documents/PDFs (OCR)
Comprehend       → NLP: sentiment, entities, key phrases
Rekognition      → Analyze images and video
Transcribe       → Speech → Text
Polly            → Text → Speech
Translate        → Language translation
Forecast         → Time-series forecasting
Personalize      → Recommendations ("you may also like")
Lex              → Build conversational chatbots
Kendra           → Intelligent enterprise search
Macie            → Find PII/sensitive data in S3
GuardDuty        → Detect threats and suspicious activity
CloudTrail       → Audit log of all API calls
CloudTrail Lake  → Query audit logs with SQL
CloudWatch       → Monitor metrics, set alarms
Config           → Monitor resource configuration compliance
Security Hub     → Centralized security findings dashboard
Inspector        → Scan for software vulnerabilities
Shield           → DDoS protection
WAF              → Web application firewall
KMS              → Manage encryption keys
Artifact         → Download AWS compliance certificates
VPC Endpoints    → Private connectivity to AWS services
SageMaker Clarify       → Bias detection + explainability
SageMaker Model Monitor → Detect drift in production models
SageMaker Ground Truth  → Data labeling service
SageMaker Data Wrangler → Data preparation and transformation
SageMaker Autopilot     → AutoML (auto-selects algorithm)
SageMaker Canvas        → No-code ML for business analysts
SageMaker Pipelines     → Automate end-to-end ML workflow
SageMaker Feature Store → Central repository for ML features
SageMaker Model Cards   → Document model details/limitations
```

---

## Golden Exam Rules — Never Forget

```
1. "During transmission" → HTTPS/TLS encryption
2. "At rest / stored" → S3 encryption + KMS
3. "PII + S3" → Amazon Macie
4. "PII + real-time model response" → Bedrock Guardrails
5. "Audit trail + who did what" → CloudTrail
6. "Query audit logs + compliance" → CloudTrail Lake
7. "Suspicious activity / threat" → GuardDuty
8. "Software vulnerabilities" → Inspector
9. "Resource configuration compliance" → Config
10. "Download compliance certificates" → Artifact
11. "Central security dashboard" → Security Hub
12. "DDoS protection" → Shield
13. "Web attacks / SQL injection" → WAF
14. "Prompt injection" → Customer's responsibility → Guardrails
15. "No-code ML / business analyst" → SageMaker Canvas
16. "AutoML / auto-select algorithm" → SageMaker Autopilot
17. "Recurring ML pipeline / workflow" → SageMaker Pipelines
18. "Label data" → SageMaker Ground Truth
19. "Clean / prepare data" → SageMaker Data Wrangler
20. "Explain model prediction / bias" → SageMaker Clarify
21. "Model degrading after deployment" → Data drift → Model Monitor
22. "Training accuracy high, test accuracy low" → Overfitting
23. "Frequently changing data" → RAG (not fine-tuning)
24. "Consistent tone/style" → Fine-tuning (not RAG)
25. "Multiple tenants / use cases + limited compute" → PEFT
26. "Overlap with reference summary" → ROUGE metric
27. "Semantic similarity" → BERTScore metric
28. "Miss cancer / fraud" → Maximize RECALL
29. "Block legitimate emails" → Maximize PRECISION
30. "High-stakes + no human oversight" → ALWAYS WRONG (Responsible AI)
31. "Fully automate + eliminate humans" → ALWAYS WRONG (Responsible AI)
32. "Hallucination fix" → RAG + grounding checks
33. "Cost driver for Bedrock" → Number of tokens processed
34. "High-volume predictable Bedrock usage" → Provisioned Throughput
35. "Private connectivity to Bedrock" → VPC Endpoints
```

---

## Recall vs Precision — Quick Calculator

```
Given: TP=90, FP=10, FN=30, TN=870

Recall    = TP/(TP+FN) = 90/(90+30) = 90/120 = 75%
Precision = TP/(TP+FP) = 90/(90+10) = 90/100 = 90%
Accuracy  = (TP+TN)/Total = (90+870)/1000 = 96%
```

---

## Responsible AI — Exam Answer Patterns

```
QUESTION TYPE: "Which is MOST responsible?"
→ Look for: human oversight, diverse data, continuous evaluation, feedback loops

QUESTION TYPE: "Which is LEAST responsible?"
→ Look for: full automation, eliminate humans, no oversight, maximize efficiency at expense of safety

QUESTION TYPE: "Who is responsible for X?"
→ Prompt injection, Guardrails config, training data quality = CUSTOMER
→ Infrastructure, FM security, service availability = AWS
```

---

*Last Updated: Exam Prep 2026 | AWS Certified AI Practitioner (AIF-C01)*
