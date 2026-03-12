# Swiss Re Final Round — Interview Prep Schedule
## Interview: March 16, 11:30 AM
## Today: March 11 — 5 days to go

---

## What They Are Testing (Keep This in Mind Every Day)

Based on your feedback from Round 1:
- Architecture skills: STRONG (don't over-prepare here)
- Cloud Governance: WEAK (your #1 priority)
- Architectural Depth: They want to verify it's real
- Java skills + programs: Expected
- Microservices patterns: Expected
- Whiteboarding: Will happen — you must be ready to draw and explain live

---

## The 5-Day Battle Plan

---

## DAY 1 — March 11 (Today) — GOVERNANCE DAY
### Total: 4-5 hours

**Your biggest gap. Fix it today.**

### Morning Block: 9:00 AM – 11:00 AM (2 hours)
**Read: `17-cloud-governance.md` — full read, no skipping**

Focus on:
- Azure Management Group hierarchy (draw it on paper)
- Azure Policy — effects, common policies, how to assign
- Azure RBAC — roles, custom roles, PIM
- Azure Conditional Access
- Cost governance — tagging strategy, budgets

After reading: close the doc and write from memory:
- The Azure governance hierarchy (5 levels)
- 5 common Azure Policy examples
- How PIM works (the workflow)

---

### Afternoon Block: 2:00 PM – 4:00 PM (2 hours)
**Continue `17-cloud-governance.md` — AWS + Alibaba sections**

Focus on:
- AWS Organizations structure
- SCPs — what they do, 3 examples
- AWS Control Tower
- Alibaba RAM + Resource Directory
- The comparison table (AWS vs Azure vs Alibaba)

After reading: practice answering these out loud (say it, don't just think it):
1. "How do you enforce that all Azure resources have required tags?"
2. "How do you prevent developers from creating public resources in production?"
3. "How do you manage costs across multiple teams?"
4. "What is a Landing Zone and why does it matter?"
5. "How do you handle privileged access in Azure?"

---

### Evening Block: 7:00 PM – 8:00 PM (1 hour)
**Draw the Azure governance hierarchy from memory on paper**
- Management Groups → Subscriptions → Resource Groups → Resources
- Where policies apply
- Where RBAC applies
- Where costs are tracked

**This is likely to appear on the whiteboard.**

---

## DAY 2 — March 12 — MICROSERVICES + JAVA DAY
### Total: 4-5 hours

### Morning Block: 9:00 AM – 11:00 AM (2 hours)
**Read: `19-microservices-design-patterns.md`**

Focus on (in this order):
1. Saga pattern — choreography vs orchestration (draw both)
2. Circuit Breaker — 3 states, when it trips
3. CQRS — draw the split model
4. Outbox pattern — why it exists, how it works
5. Shared Database Service design — this is Swiss Re specific, read carefully

After reading: draw the Shared Database Service architecture from memory.
This is very likely to come up given you're joining a Shared Services team.

---

### Afternoon Block: 2:00 PM – 4:30 PM (2.5 hours)
**Read: `18-java-for-architects.md`**

Focus on:
1. Design patterns — Builder, Factory, Singleton, Strategy, Circuit Breaker proxy
2. Concurrency — thread safety, AtomicInteger, CompletableFuture
3. Spring Boot structure — annotations, Resilience4j
4. Programs — write these by hand (not copy-paste):
   - Retry with exponential backoff
   - LRU Cache
   - Rate Limiter (token bucket)
   - Thread-safe Singleton

**Write the programs on paper. This is how you prepare for live coding.**

---

### Evening Block: 7:30 PM – 8:30 PM (1 hour)
**Practice answering Java questions out loud:**
1. HashMap vs ConcurrentHashMap
2. How does @Transactional work?
3. What is CompletableFuture and when would you use it?
4. How do you handle connection pooling in Spring Boot?
5. What is the difference between Saga and 2PC?

---

## DAY 3 — March 13 — WHITEBOARDING + SYSTEM DESIGN DAY
### Total: 4-5 hours

### Morning Block: 9:00 AM – 10:00 AM (1 hour)
**Read: `20-whiteboarding-guide.md` — full read**

Memorize:
- The 5-step framework (Clarify → Estimate → High-level → Deep dive → Trade-offs)
- The 30-minute session structure with timings
- The 10 phrases that show architect thinking
- The 10 mistakes to avoid

---

### Morning Block: 10:00 AM – 12:00 PM (2 hours)
**Whiteboard Practice — get a physical whiteboard or large paper**

Practice Problem 1: Design a Shared Database Service (30 min)
```
Step 1 (5 min): Write requirements on board
Step 2 (3 min): Estimate scale
Step 3 (10 min): Draw high-level architecture
Step 4 (8 min): Deep dive on access management + governance
Step 5 (4 min): Trade-offs
```

Practice Problem 2: Design a Multi-Cloud Data Platform (Azure + Alibaba) (30 min)
```
Same 5-step approach
Focus on: data residency, governance consistency, connectivity
```

Practice Problem 3: Design a Microservices Order System (30 min)
```
Focus on: Saga pattern, Circuit Breaker, API Gateway
Draw the failure scenarios
```

---

### Afternoon Block: 2:00 PM – 4:00 PM (2 hours)
**Read: `13-25-interview-designs.md` — focus on these 5:**
1. Design a Payment System (most relevant for financial services)
2. Design a Notification System
3. Design a Multi-Region Active-Active System
4. Design a CI/CD Pipeline
5. Design a Banking System

For each: read the solution, then close the doc and redraw from memory.

---

### Evening Block: 7:30 PM – 8:30 PM (1 hour)
**Read: `11-system-design-frameworks.md`**
- RESHADED framework — memorize the acronym
- Trade-off articulation framework — practice 3 examples
- Database selection framework — know the decision tree

---

## DAY 4 — March 14 — ARCHITECTURE DEPTH + AZURE DAY
### Total: 4 hours

### Morning Block: 9:00 AM – 11:00 AM (2 hours)
**Read: `08-azure-architecture.md`**

Focus on:
- AKS production architecture (draw it)
- Azure networking: Hub-Spoke, Private Link, APIM
- Azure vs AWS service mapping table (memorize 10 key pairs)
- Cosmos DB consistency levels
- Azure DR strategies

---

### Afternoon Block: 2:00 PM – 4:00 PM (2 hours)
**Read: `06-networking-deep-dive.md`**

Focus on:
- Complete request flow (DNS → CDN → LB → Ingress → Service → Pod)
- Azure request flow specifically (Front Door → App Gateway → AKS)
- Istio mTLS flow
- SSL/TLS termination options

Practice: trace a request from a user's browser to a pod in AKS, naming every component.

---

### Evening Block: 7:30 PM – 8:00 PM (30 min)
**Review `14-visual-architecture-guide.md`**
- Look at diagrams 21-25 (Azure specific)
- Look at diagrams 26-33 (microservices patterns)
- Redraw 3 of them from memory

---

## DAY 5 — March 15 — FINAL REVIEW + MOCK DAY
### Total: 3-4 hours (don't overdo it — rest matters)

### Morning Block: 9:00 AM – 10:30 AM (1.5 hours)
**Speed review — all governance interview Q&A from `17-cloud-governance.md`**

Read each question, cover the answer, answer out loud, then check.
Target: answer each in under 2 minutes, confidently.

Questions to nail:
1. How do you enforce tags on Azure resources?
2. How do you prevent public resources in production?
3. How do you manage costs across teams?
4. What is a Landing Zone?
5. How do you handle privileged access?
6. How do you ensure GDPR compliance in Azure?
7. What are SCPs and how do they differ from IAM policies?
8. How does Alibaba RAM differ from Azure RBAC?

---

### Mid-Morning Block: 10:30 AM – 12:00 PM (1.5 hours)
**Full Mock Whiteboard Session (simulate the real thing)**

Set a timer. Stand up. Use a whiteboard or large paper.

Problem: "Design a Shared Database Service for a financial services company. The platform team supports multiple application teams on Azure and Alibaba Cloud."

Run the full 30-minute session:
- 5 min: requirements (ask yourself the questions)
- 3 min: estimation
- 10 min: draw high-level
- 8 min: deep dive on governance + access management
- 4 min: trade-offs

Then do a second problem: "Design a microservices-based order processing system"

---

### Afternoon Block: 2:00 PM – 3:00 PM (1 hour)
**Java programs — write from memory, no looking:**
1. Retry with exponential backoff (10 min)
2. Thread-safe Singleton (5 min)
3. LRU Cache (10 min)
4. Circuit Breaker state machine (10 min)

---

### Evening Block: 6:00 PM – 7:00 PM (1 hour)
**Light review only — no new material**

Read through:
- `16-fundamentals-first.md` — the Swiss Re focus areas section
- Your hand-drawn diagrams from the week
- The 10 whiteboard phrases from `20-whiteboarding-guide.md`

**After 7 PM: stop studying. Rest. Sleep early.**

---

## DAY 6 — March 16 — INTERVIEW DAY
### Interview: 11:30 AM

### 7:00 AM – 8:00 AM: Light warm-up
Read only these (no new material):
- The 5-step whiteboard framework
- The governance interview Q&A (just skim)
- The architect phrases list

### 8:00 AM – 9:00 AM: Breakfast, walk, clear your head

### 9:00 AM – 10:30 AM: Final mental prep
- Review your hand-drawn diagrams
- Say out loud: "I am a Cloud Solution Architect. I think in systems, trade-offs, and business outcomes."
- Review the Shared Database Service design one more time

### 10:30 AM – 11:00 AM: Travel / setup
- Arrive early if in-person
- Test audio/video if remote

### 11:30 AM: Interview

---

## Priority Matrix — What to Focus On

### MUST KNOW (non-negotiable)
```
✓ Azure governance hierarchy (draw from memory)
✓ Azure Policy — effects and 5 common examples
✓ PIM workflow
✓ Shared Database Service design
✓ Saga pattern (choreography + orchestration)
✓ Circuit Breaker (3 states)
✓ CQRS diagram
✓ Whiteboard 5-step framework
✓ Java: Singleton, Builder, Strategy, Circuit Breaker patterns
✓ Java: Retry with backoff program
✓ LRU Cache program
✓ Request flow: browser → AKS pod (every component)
```

### SHOULD KNOW (high probability)
```
✓ Azure RBAC custom roles
✓ Conditional Access policies
✓ Outbox pattern
✓ CompletableFuture usage
✓ Resilience4j annotations
✓ Hub-Spoke network topology
✓ Alibaba RAM vs Azure RBAC comparison
✓ Multi-region active-active design
✓ Cost governance (tagging + budgets)
```

### GOOD TO KNOW (if time permits)
```
✓ Event Sourcing
✓ AWS SCPs examples
✓ Alibaba Cloud Config rules
✓ Kubernetes RBAC deep dive
✓ Istio mTLS internals
```

---

## Daily Time Budget Summary

| Day | Date | Focus | Hours |
|-----|------|-------|-------|
| Day 1 | Mar 11 | Cloud Governance (Azure + AWS + Alibaba) | 5h |
| Day 2 | Mar 12 | Microservices Patterns + Java | 5h |
| Day 3 | Mar 13 | Whiteboarding + System Design Practice | 5h |
| Day 4 | Mar 14 | Azure Architecture Depth + Networking | 4h |
| Day 5 | Mar 15 | Mock sessions + Final review | 4h |
| Day 6 | Mar 16 | Light warm-up only | 1.5h |
| **Total** | | | **24.5h** |

---

## The Night Before Checklist (March 15 Evening)

- [ ] Hand-drawn Azure governance hierarchy ready
- [ ] Hand-drawn Shared Database Service architecture ready
- [ ] Hand-drawn Saga pattern (both variants) ready
- [ ] Java programs written on paper
- [ ] Whiteboard phrases memorized
- [ ] Governance Q&A practiced out loud
- [ ] Good night's sleep — target 8 hours

---

## Confidence Anchors

When you feel nervous, remember:
- Round 1 feedback: "architecture answers very good" — your foundation is solid
- You are not starting from zero — you are filling specific gaps
- They already like you — this round is about confirming depth
- You have 5 days of focused preparation — that is enough
- Governance is learnable in 1 day — it's not complex, just unfamiliar

**You are ready. Execute the plan.**
