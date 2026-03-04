# Hybrid Services → Product Roadmap

> Build revenue and market intelligence through services, then productize what you learn.

---

## Phase 1: Services (Weeks 1-6)

### Goal
Land 2-3 paid consulting engagements helping companies manage their agent infrastructure. Get paid to learn their real pain points.

### Service Offering: "Agent Operations Audit & Setup"

**What you deliver:**
- Audit of current agent deployments (what agents, what wallets, what rails)
- Policy design (spending limits, approval workflows, anomaly triggers)
- Integration setup (connect their agents to Coinbase Agentic, Stripe x402, or existing rails)
- Dashboard setup using existing tools (Grafana, LangSmith, custom scripts)
- Documentation + runbooks for their team
- 30 days of Slack/email support

**Pricing:**
- Starter: $5,000 (up to 10 agents, 1 wallet type, basic policies)
- Growth: $15,000 (up to 50 agents, multiple wallets, custom policies, training)
- Enterprise: $40,000+ (100+ agents, compliance requirements, ongoing advisory)

**Deliverable timeline:** 2-4 weeks per engagement

### Target Customers (First 3)

**1. AI-Native Startups (Series A/B)**
- Running 10-50 agents in production
- Using a mix of LangChain, CrewAI, AutoGPT
- Pain: "We have agents spending money but no visibility"
- Where to find: Y Combinator alumni, AI Twitter, LangChain Discord
- Pitch: "I'll audit your agent fleet and set up proper controls before your next SOC2 audit"

**2. Healthcare Tech Companies**
- Using AI for claims processing, prior auth, patient outreach
- Pain: HIPAA compliance + agent autonomy = terror
- Where to find: Healthcare AI Slack communities, HIMSS attendees, LinkedIn
- Pitch: "I'll make your AI agents HIPAA-audit-ready with proper spending controls and audit trails"

**3. Fintech / Trading Firms**
- AI agents executing trades, managing positions, API calls
- Pain: Regulatory scrutiny, need fiduciary guardrails
- Where to find: Fintech meetups, crypto trading communities, LinkedIn
- Pitch: "I'll set up policy-based controls so your trading agents can't go rogue"

### Outreach Strategy

**Week 1-2: Warm network**
- Post on LinkedIn/Twitter: "Helping companies get control of their AI agent spending. DM me."
- Email 20 founders you know or are 1 degree from
- Offer first engagement at 50% discount for case study rights

**Week 3-4: Cold outreach**
- Identify 50 companies from Crunchbase (AI startups, Series A+, keywords: "agents", "autonomous", "AI automation")
- Personalized emails: "Saw you're building [X]. When your agents start spending money, who's watching? I help companies set up controls before it becomes a problem."

**Week 5-6: Content + inbound**
- Publish 2-3 posts on "Agent Wallet Sprawl" problem
- Share real (anonymized) learnings from first engagements
- Build waitlist for the product

---

## Phase 2: MVP Build (Weeks 7-14)

### Goal
Build minimum viable product based on patterns from consulting. First customers = consulting clients.

### What You Learned in Phase 1 (Hypotheses to Validate)
- Which wallet integrations matter most? (Coinbase? Stripe? Custom?)
- What policies do they actually need? (Spending limits? Time-based? Category?)
- What's the approval workflow? (Slack? Email? Dashboard?)
- What audit format do compliance teams want? (CSV? PDF? API?)

### MVP Feature Set

**Must Have (Week 7-10):**
- [ ] Agent registry (name, type, owner, status)
- [ ] Connect 2 wallet types (Coinbase Agentic + one other)
- [ ] Basic policy engine (daily spend limit, per-transaction limit)
- [ ] Simple dashboard (list agents, see recent transactions, policy violations)
- [ ] Alerts (email/Slack when policy violated)

**Should Have (Week 11-14):**
- [ ] Approval workflows (transactions over $X require human approval)
- [ ] Audit log export (CSV)
- [ ] SDK/API for agent frameworks to register and report
- [ ] Multi-user access (admin, viewer roles)

**Nice to Have (Post-MVP):**
- [ ] ERC-8004 identity integration
- [ ] Anomaly detection (ML-based)
- [ ] Compliance templates (SOC2, HIPAA, FINRA)
- [ ] More wallet integrations

### Tech Stack (Lean)

```
Backend:    Go or Node.js (whatever you ship faster)
Database:   PostgreSQL (Supabase works fine)
Queue:      Redis or Supabase Realtime
Frontend:   React (reuse Loop dashboard patterns)
Infra:      Vercel + Supabase (you know it)
```

Don't over-engineer. Ship something that works for 3 customers.

### Pricing (SaaS)

- **Starter:** $299/mo (up to 20 agents, 2 users, email support)
- **Growth:** $799/mo (up to 100 agents, 10 users, Slack support, custom policies)
- **Enterprise:** $2,500+/mo (unlimited, SSO, dedicated support, compliance add-ons)

---

## Phase 3: Transition (Weeks 15-20)

### Goal
Convert consulting clients to SaaS. Start acquiring new customers on product.

### Consulting → SaaS Conversion

For each consulting client:
1. "Hey, remember that dashboard/setup I built for you? I've productized it."
2. "Instead of paying me for ongoing support, subscribe to the platform — $799/mo."
3. "You get continuous monitoring, automatic updates, and I'm still here for questions."

**Target:** Convert 2 of 3 consulting clients to paid SaaS seats = $1,600-$5,000 MRR baseline

### New Customer Acquisition

- Case studies from consulting clients
- Content marketing: "How [Company X] got control of 50 AI agents"
- Product Hunt launch
- Integrations marketplace (LangChain, CrewAI partner listings)
- LinkedIn/Twitter presence: "The Agent Control Plane"

### Services as Enterprise Tier

Don't kill services — reposition them:
- **Product:** Self-serve, <100 agents, standard integrations
- **Services:** Enterprise implementation, custom integrations, compliance projects
- Services become $50-100k/year enterprise contracts, not $5k one-offs

---

## Timeline Summary

| Week | Focus | Milestone |
|------|-------|-----------|
| 1-2 | Outreach | 10+ conversations, 2 proposals sent |
| 3-4 | Close | 2 consulting deals signed ($10-30k) |
| 5-6 | Deliver | First engagement complete, patterns documented |
| 7-10 | Build MVP | Core features working |
| 11-14 | Polish | Dashboard, SDK, second wallet integration |
| 15-16 | Convert | Consulting clients on SaaS |
| 17-20 | Scale | 5-10 paying customers, $5k+ MRR |

---

## Success Metrics

**Phase 1 (Services):**
- [ ] 3 signed consulting deals
- [ ] $20,000+ revenue
- [ ] Clear pattern of "what they actually need"

**Phase 2 (MVP):**
- [ ] Working product with 2 wallet integrations
- [ ] 3 beta users (consulting clients)
- [ ] <4 second dashboard load time

**Phase 3 (Transition):**
- [ ] 2+ consulting clients converted to SaaS
- [ ] 5+ total paying customers
- [ ] $5,000+ MRR

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Can't find consulting clients | Lower price, offer free audit for case study rights |
| Clients want different things | Find the 80% overlap, build that |
| Tech takes longer than expected | Cut scope ruthlessly, ship ugly but working |
| Market not ready | Services keep you alive while market matures |
| Big player ships competing product | Go vertical (healthcare/finance specialization) |

---

## Decision Points

**After Week 6 (End of Services Phase):**
- Did you get 2+ paying clients? → Continue to MVP
- Did you struggle to find clients? → Pivot messaging or reconsider market

**After Week 14 (End of MVP):**
- Do consulting clients want to use the product? → Continue to scale
- Are they lukewarm? → More customer development, iterate

**After Week 20:**
- $5k+ MRR and growing? → Raise seed or keep bootstrapping
- Flat/declining? → Consider pivoting to pure services or different product

---

## Immediate Next Actions

1. **Today:** Update LinkedIn headline to "Helping companies control their AI agents"
2. **This week:** Draft the service offering one-pager (I can help)
3. **This week:** List 20 warm contacts who might need this or know someone
4. **Next week:** Send first 10 outreach emails
5. **Next week:** Post first content piece on "Agent Wallet Sprawl"

---

*Last updated: 2026-03-04*
