# Real Estate Voice Agent (UAE): Research, Plan & Scope

Working document, v0.1 (Nov 2026). Items marked **[verify]** are things we found in secondary sources or that change often. Confirm them against the official source before relying on them.

---

## 1. How voice agents work

A voice agent is a loop that runs every time the caller speaks:

```
Phone/WhatsApp/Web call
   │  (audio stream, 8kHz phone / 16-48kHz web)
   ▼
[Telephony / WebRTC]  ── SIP trunk (du / e&) or WebRTC
   ▼
[VAD + turn detection]  ── detects when the caller started or stopped talking; handles barge-in
   ▼
[STT / ASR]  ── speech → text (+ language ID: Arabic / English / Hindi / Russian)
   ▼
[Agent brain (LLM)]  ── understands intent, decides what to say, calls tools
   │      ├── tools: property search, DLD comps, rent index, mortgage calc,
   │      │          calendar booking, CRM read/write, WhatsApp follow-up
   │      └── knowledge: RAG over listings, area guides, regulations
   ▼
[Guardrails]  ── compliance filter (no guaranteed returns, AI disclosure, etc.)
   ▼
[TTS]  ── text → speech in the caller's language/dialect
   ▼
back to caller   (target: < 800 ms from user stops talking → agent starts talking)
```

There are two architecture styles:

| Style | How it works | Pros | Cons |
|---|---|---|---|
| **Cascaded** (STT → LLM → TTS) | Three separate models | Full control. Best-in-class component for each language. Easy to log, audit and apply guardrails. Cheaper | More latency to engineer away |
| **Speech-to-speech** (e.g. OpenAI Realtime, Gemini Live) | One model hears and speaks | Very natural, low latency | Weaker Gulf Arabic. Harder to audit and add guardrails. Vendor lock-in |

**Recommendation:** cascaded for production, because compliance and auditability matter in a regulated market. Track speech-to-speech models and switch when their Gulf Arabic quality is good enough.

The hard parts are the Arabic dialect, latency, interruptions, Arabic–English code-switching mid-sentence ("ابغى 2 bedroom في Marina"), and phone-quality audio (8 kHz).

---

## 2. What already exists

### Global (mostly US, English)
- **Voice infrastructure platforms:** Vapi, Retell AI, Bland, Synthflow, LiveKit Agents, Pipecat. These are building blocks that most "AI calling" startups are thin wrappers on.
- **Real-estate-specific:** Structurely (AI inside-sales agent), Ylopo (AI voice/text for leads), Lofty, Follow Up Boss integrations. These focus on US speed-to-lead and nurturing.
- **Call analytics / coaching:** Gong, Chorus, Balto (real-time coaching), Convin, Observe.AI.

### UAE / GCC voice-AI vendors that market to real estate (all claims come from their own marketing **[verify]**)
- **Neyox.ai:** multilingual voice agents including Arabic dialects; pilots in Dubai/Abu Dhabi.
- **BIG LAB (Dubai):** Arabic, English, Russian and Hindi voice agents; names Mira Developments as a client.
- **Grow50X.ai:** Arabic/English, published price of about USD 0.25/min.
- **Lumaa AI, MAJ Leads, Orato, Tough Tongue AI:** AI calling or lead qualification for Dubai real estate.
- **xAIa (PRO TECHnology Group):** Arabic-first voice AI for real estate and facilities management.
- **Arabic.AI by Tarjama:** Arabic AI platform with voice agents.
- **Portals:** Property Finder and Bayut/dubizzle are adding AI features. They are a partnership channel and also a potential competitor.

### What this tells us
The "generic AI caller for Dubai real estate" space is already crowded with agencies and wrappers. To stand out we need to win on:
1. **Real Gulf (Khaleeji) Arabic and code-switching quality,** not just Modern Standard Arabic.
2. **Built-in compliance:** RERA/DLD, TDRA telemarketing and PDPL, enforced by the system rather than left to the user.
3. **Real market intelligence:** live DLD transaction comps and rental index inside the conversation, which none of the wrappers do well.
4. **Broker call analytics:** coaching and QA on human brokers' calls, not just a bot.

---

## 3. Public data available

### Land, area and market data (agent side)
| Source | What it has | Access |
|---|---|---|
| **Dubai Pulse – DLD datasets** | Sales, mortgage, gift/inheritance transactions; licences; residential sale price index (with Property Finder) | CSV, plus a Dubai Pulse API (key and secret issued by email) |
| **DLD Open Data portal** (dubailand.gov.ae/en/open-data) | Transactions, rentals (Ejari), projects, valuations, land, buildings, units, brokers, developers | Web tool and downloads |
| **Dubai REST app / DLD rental index** | Rental index by area, which drives the legally allowed rent increases | Web/app **[verify API]** |
| **Abu Dhabi – ADREC / DARI** | Abu Dhabi transactions and market reports | Partly public **[verify]** |
| **Bayut / Property Finder** | Listings, area guides, transaction pages | No open API. Scraping breaks their terms of service, so a partnership is needed |
| **Dubai Statistics Center, FCSC** | Population, demographics by area | Open |
| **OpenStreetMap / Dubai Municipality GIS** | Area boundaries, POIs (schools, metro, malls) | Open |
| **Developer brochures** (Emaar, DAMAC, Sobha, Nakheel, Aldar…) | Off-plan projects, payment plans | Public PDFs; better to get them through partnerships |

### Regulation text (for the compliance knowledge base)
- **uaelegislation.gov.ae:** federal laws and Cabinet resolutions (official PDFs, Arabic and English).
- **dubailand.gov.ae:** RERA rules, broker code of ethics, circulars.
- **Dubai Legislation portal (dlp.dubai.gov.ae):** Dubai laws and decrees.
- **ADREC / DMT:** Abu Dhabi real estate regulations.

### Speech data (voice side)
- **Public Arabic speech corpora:** Mozilla Common Voice (Arabic), MGB-2/3/5 (Arabic broadcast and dialectal, from QCRI), MASC, QASR. A few small Gulf/Emirati dialect sets exist from academia (**[verify]** licences, since many are research-only).
- **No public corpus of real estate sales calls exists.** We have to create it (see §6). This becomes our moat.

---

## 4. What our agent will do (product scope)

### Users
1. **Brokerages (B2B, main customer):** they handle large numbers of portal leads and lose deals by responding slowly.
2. **Developers:** off-plan launches, with thousands of enquiries in days.
3. **Property managers / landlords:** tenant queries, renewals, maintenance.
4. **End buyers and tenants:** they talk to the agent but are not the payer.

### Capabilities (by priority)
| # | Capability | Description |
|---|---|---|
| P0 | **Inbound lead qualification** | Answers portal/website/WhatsApp leads within 60 seconds, 24/7. Asks about budget, area, property type, timeline, cash or mortgage, and residency/visa goal. Scores the lead and pushes it to the CRM |
| P0 | **Viewing booking** | Checks the broker's calendar, books the viewing, sends a WhatsApp confirmation and reminders |
| P0 | **Property Q&A** | Answers about specific listings, areas, service charges, payment plans |
| P1 | **Market intelligence answers** | Example: "What did 2BRs in JVC sell for last quarter?" answered with real DLD transactions; rental index lookups |
| P1 | **Outbound follow-up** | Re-engages cold leads, but only with consent and within TDRA call hours |
| P1 | **Post-call intelligence** | Summary, extracted requirements, sentiment, next action, CRM note |
| P2 | **Broker call analytics** | Records human broker calls (with consent), transcribes them, scores against best practice and compliance, coaches |
| P2 | **Real-time broker assist** | Live prompts during a human call ("client asked about Golden Visa; AED 2M threshold") |
| P3 | **Tenant/landlord service** | Ejari, renewals, rent-increase eligibility from the RERA calculator, maintenance tickets |
| P3 | **Mortgage pre-qualification** | Estimates using UAE Central Bank LTV rules, then hands off to a mortgage broker |

### What the agent must never do
- Give legal, financial or investment advice, or promise returns ("guaranteed 8% ROI" is a red line).
- Quote a price for a listing that doesn't have a valid advertising permit (Trakheesi/QR).
- Pretend to be human. It discloses that it is an AI assistant at the start of the call **[verify current disclosure requirement; it is safe practice regardless]**.
- Call numbers on the Do-Not-Call Register or outside permitted hours.

---

## 5. Agent logic and pipeline

### 5.1 Conversation state machine (the LLM runs inside this; it doesn't replace it)

```
GREETING + AI disclosure + recording notice
   ▼
LANGUAGE DETECT → lock language (allow code-switch)
   ▼
INTENT ROUTING ──► buy | rent | sell/list | off-plan | tenant service | other → human
   ▼
QUALIFICATION (slot filling)
   budget • area(s) • type • beds • timeline • finance (cash/mortgage)
   • purpose (live/invest/visa) • nationality/residency (only if needed)
   ▼
MATCH + INFORM
   tool: search_listings(slots) → top 3
   tool: dld_comps(area, type, beds) → recent sold/rented prices
   tool: rent_index(area, current_rent) → allowed increase
   ▼
OBJECTION HANDLING (learned from broker recordings)
   ▼
CONVERT ──► book_viewing | schedule_callback | send_whatsapp_brochure | handoff_to_human
   ▼
WRAP-UP → summary → CRM write → lead score → follow-up schedule
```

Escalate to a human at any point if: the caller asks for one, is angry, wants legal or financial advice, the deal is above a value threshold, or the model's confidence is low.

### 5.2 System pipeline

```
┌───────────── Real-time path (latency critical) ─────────────┐
│ SIP/WebRTC → VAD → streaming STT → Dialog Orchestrator       │
│                                      │                       │
│                     ┌────────────────┼──────────────┐        │
│                     ▼                ▼              ▼        │
│               LLM (fast)       Tool layer      Regulation    │
│                                (listings,       RAG +        │
│                                 DLD, CRM,       guardrail    │
│                                 calendar)       checker      │
│                     └────────────────┬──────────────┘        │
│                                      ▼                       │
│                              streaming TTS → caller          │
└──────────────────────────────────────────────────────────────┘
┌───────────── Async path (after call) ───────────────────────┐
│ recording → high-accuracy STT → diarization → LLM analysis   │
│ → summary / score / compliance audit → CRM + dashboard       │
│ → anonymised, consented data → training set (flywheel)       │
└──────────────────────────────────────────────────────────────┘
┌───────────── Data pipeline (batch, daily) ──────────────────┐
│ Dubai Pulse / DLD → clean → Postgres + PostGIS (areas)       │
│ Regulations PDFs → chunk (AR+EN) → vector DB, versioned      │
│ Partner listings feed (XML/API) → listings DB                │
└──────────────────────────────────────────────────────────────┘
```

### 5.3 Multi-agent split (when we grow past the MVP)
- **Conversation agent:** talks to the caller, using a small fast model.
- **Research agent:** DLD comps, listings and area info, using tools.
- **Compliance agent:** checks every outgoing statement against the regulation knowledge base (or a fast rules plus classifier layer in real time, with a full LLM audit after the call).
- **Analytics agent:** post-call scoring and coaching.

Start with **one agent plus tools plus a guardrail layer**. Multi-agent adds latency, so add it only where it pays off.

---

## 6. Data we need to collect

### 6.1 Dealer/broker recordings (for the voice part and real-time analysis)
**Purpose:** learn how good Dubai brokers actually talk (objection handling, qualification order, Khaleeji and code-switched speech, phone audio), and build the call-analytics product.

**How to get it legally:**
1. **Design partners:** sign 3–5 brokerages. They already record calls (TDRA requires marketing calls to be recorded with notice). We get access under a data-processing agreement.
2. **Consent:** under PDPL (Federal Decree-Law 45/2021), the recording notice at call start must cover "quality and AI training". Get written consent from brokers (employees).
3. **Anonymise:** strip names, phone numbers, Emirates IDs and unit numbers before the data goes into any training set.
4. **Paid role-play data:** hire Emirati/Gulf speakers and real brokers to role-play scripted plus free-form buyer/tenant calls. This is clean, fully consented, and covers dialects we're missing.
5. **Our own calls:** once live, every consented call grows the dataset.

**Labels to annotate:** speaker turns, language spans, intent, slots (budget, area…), objections, outcome (viewing booked or not), compliance issues, sentiment.

**Target:** 300–500 hours of transcribed domain calls for STT fine-tuning and evaluation. About 50 hours of carefully labelled calls is enough for dialog and coaching logic.

### 6.2 Dealer/broker data
- RERA broker licence registry (public via DLD/Dubai Pulse), used to verify brokers who onboard.
- Partner brokerage CRMs: listings, agents, calendars, lead history (through integrations such as Salesforce, HubSpot, Zoho, Propspace, Goyzer **[verify the CRMs most used by Dubai brokers]**).

### 6.3 Land and area data
- DLD transactions (sales, mortgages), Ejari rentals, rental index, projects, units and buildings: daily sync.
- Area knowledge base: communities, freehold/leasehold status, service charges (from DLD's service charge index), schools, metro, typical yields.
- Off-plan: developer projects, escrow-registered status, payment plans.

### 6.4 Regulation dataset (official alignment)
Build a **versioned, bilingual (AR/EN) regulation corpus**. Each chunk is tagged with its source, article number, emirate, effective date and whether it is superseded.

Core sources to ingest (all **[verify current versions]**):
| Area | Instrument |
|---|---|
| Land registration (Dubai) | Law No. 7 of 2006 |
| Escrow / off-plan (Dubai) | Law No. 8 of 2007; Law No. 13 of 2008 (interim register) and amendments |
| Landlord–tenant (Dubai) | Law No. 26 of 2007, amended by Law No. 33 of 2008; Decree No. 43 of 2013 (rent increases) |
| Brokers (Dubai) | RERA broker regulations, broker code of ethics, Trakheesi advertising permit rules |
| Abu Dhabi | Law No. 3 of 2015 (real estate sector) and ADREC rules |
| Telemarketing (federal) | Cabinet Resolution No. 56 of 2024 (rules) and No. 57 of 2024 (penalties), in force from 27 Aug 2024 |
| Data protection (federal) | Federal Decree-Law No. 45 of 2021 (PDPL) and its executive regulations |
| AML | Real estate brokers are DNFBPs under the federal AML law and must do KYC above cash thresholds |
| Golden Visa | Property-investment visa thresholds (ICP) |

**How the model stays aligned:**
1. **RAG with citations:** every regulatory answer cites article and source. If nothing is retrieved, the agent says "I'll connect you with an agent."
2. **Hard rules layer:** deterministic checks the LLM can't override: call hours, the DNC list, the AI-disclosure line, recording notice, blocked phrases ("guaranteed return").
3. **Eval suite:** hundreds of tricky questions with lawyer-approved answers, run on every model or prompt change.
4. **Legal review:** a UAE real estate lawyer signs off on the corpus and the red-line list every quarter.
5. **Change monitoring:** a scraper checks DLD, uaelegislation and TDRA for new circulars and flags them for re-ingestion.

---

## 7. Tech stack

### Voice layer
| Component | English | Arabic (Gulf) | Notes |
|---|---|---|---|
| **Orchestration** | LiveKit Agents or Pipecat (open source) | same | Avoid Vapi/Retell lock-in long-term; fine for a 2-week prototype |
| **Telephony** | SIP trunk through a **UAE-licensed operator (e& / du)** | same | VoIP is regulated in the UAE. Foreign CPaaS numbers may be blocked or unreliable. Confirm with e&/du and TDRA **[verify]** |
| **WhatsApp** | WhatsApp Business API (Meta) | same | Most UAE leads prefer WhatsApp, including its voice notes |
| **STT** | Deepgram, AssemblyAI, Whisper | Benchmark: Speechmatics, Azure (ar-AE), Google Chirp, ElevenLabs Scribe, Deepgram Arabic, plus UAE/regional Arabic ASR vendors **[verify]**; fine-tuned Whisper-large on our data | **Gulf Arabic STT is the main risk.** Build our own WER benchmark from real calls before choosing |
| **TTS** | ElevenLabs, Cartesia, Azure | ElevenLabs multilingual, Azure ar-AE neural voices; custom Emirati voice later | Native speakers should run a MOS test for "sounds Emirati, not Egyptian or MSA" |
| **VAD / turn-taking** | Silero VAD plus a turn-detection model | same | Arabic pause patterns differ, so tune the thresholds |

### Brain
- **LLM:** a frontier model with strong Arabic and tool use for reasoning. Candidates: Claude (`claude-sonnet-5-5` for quality, `claude-haiku-5-5` for low-latency turns), GPT, Gemini. Evaluate open Arabic models (Jais, Falcon Arabic) for data-residency or on-prem needs.
- **RAG:** Postgres + pgvector (one database to start), multilingual embeddings (e.g. Cohere multilingual or BGE-M3).
- **Data:** Postgres + PostGIS for areas and transactions; Redis for session state.
- **Backend:** Python (FastAPI), because the voice ecosystem is Python-first.
- **Dashboard:** Next.js; Arabic RTL support from day 1.
- **Hosting:** a UAE region (Azure UAE North, AWS me-central-1, Oracle, G42/Core42) for **data residency**. This matters for PDPL and enterprise or government buyers.
- **Observability:** Langfuse or a similar tool for LLM traces; call recordings plus transcripts for QA.

### Doing Arabic well (the core challenge)
1. **Dialects:** Gulf/Emirati and Levantine/Egyptian (many brokers and callers), Modern Standard Arabic for formal speech, and Arabic–English code-switching.
2. **Benchmark first:** record or collect about 10 hours of real Gulf phone audio. Measure WER for 5–6 STT vendors. Choose based on numbers, not marketing.
3. **Fine-tune:** take Whisper-large-v3 (or the best open model), fine-tune on our domain calls, and use domain vocabulary boosting (community names like "Jumeirah Village Circle", "Al Barsha", developer names, "Ejari", "Mulkiya", "oqood").
4. **Normalise:** Arabic text normalisation (alef/ya/ta-marbuta variants, Arabic vs Western numerals, Arabizi).
5. **TTS persona:** pick or clone a warm Gulf-accent voice. Avoid MSA "news-reader" voices, which sound robotic to locals.
6. **Prompting:** reply in the caller's register. If they speak Khaleeji, don't answer in formal MSA.
7. **Human evaluation:** native Emirati reviewers score calls weekly.

**Reality check:** in Dubai most real estate conversations happen in **English, plus Hindi/Urdu, Russian and Arabic**. Arabic is the differentiator and is essential for Abu Dhabi, Sharjah and government or developer clients, but English plus Hindi plus Russian covers a large share of Dubai volume. Plan for multilingual from the start.

---

## 8. Roadmap

### Phase 0: Foundations (weeks 0–4)
- Company setup decision (free zone: DIFC/ADGM/DMCC/Dubai AI Campus, Hub71 in Abu Dhabi), and a lawyer engaged for the regulatory review.
- 10–15 discovery interviews with brokerages and developers: where do leads die? What do they pay today?
- Ingest DLD open data and build the first regulation corpus.
- STT/TTS benchmark on Gulf Arabic plus English phone audio.
- **Exit criteria:** 3 design partners signed (LOI) and a vendor shortlist backed by numbers.

### Phase 1: English MVP (weeks 4–10)
- Inbound lead-qualification agent (English) on web plus phone.
- Tools: listing search on the partner's feed, viewing booking, CRM write, WhatsApp confirmation.
- Hard compliance layer: disclosure, recording notice, call hours, DNC.
- Post-call summary dashboard.
- **Exit criteria:** live with 1 partner, more than 100 real calls, and an under-60-second lead response.

### Phase 2: Arabic + market intelligence (weeks 10–20)
- Gulf Arabic plus code-switching (fine-tuned STT, chosen TTS voice).
- DLD comps and rental index tools in conversation.
- Regulation RAG with citations, plus the eval suite.
- Start collecting broker recordings with partners.
- **Exit criteria:** Arabic calls rated acceptable or better by native reviewers at least 80% of the time; first paying customers.

### Phase 3: Broker analytics + outbound (months 5–9)
- Call analytics and coaching for human brokers (QA scores, compliance flags).
- Consent-based outbound re-engagement.
- Add Hindi and Russian.
- Integrations with major CRMs and portals.

### Phase 4: Scale (months 9–18)
- Real-time broker assist (live prompts during calls).
- Developer off-plan launch mode (high-volume inbound).
- Abu Dhabi (ADREC) and the rest of the GCC: Saudi Arabia (REGA, Arabic-first) is the natural next market.
- Own fine-tuned Gulf Arabic STT and TTS as a defensible asset; possibly license it as an API.

---

## 9. Our view on the startup

### Opportunity
- Dubai is one of the world's most active property markets: a huge lead flow, thousands of brokerages, and high broker churn. **Speed-to-lead** is a real, measurable pain point.
- Call recording is already required by law for marketing calls, so the audio exists. Most of it is never analysed.
- The government pushes AI adoption (UAE AI strategy, DLD digitisation), which makes regulators friendlier than in most markets.

### Risks
| Risk | Mitigation |
|---|---|
| Crowded "AI caller" market | Differentiate on Arabic quality, compliance and DLD intelligence; avoid being a thin wrapper |
| Gulf Arabic STT accuracy | Benchmark early, own the data flywheel, fine-tune |
| Regulation changes and liability | Versioned corpus, lawyer review, human escalation, hard rules layer |
| Portals (Property Finder, Bayut) build it in-house | Partner with them or position as broker-side tooling across all portals |
| Telephony restrictions in the UAE | Local SIP via e&/du; WhatsApp calling as a fallback |
| Unit economics (per-minute model costs) | Cascaded stack, small fast models for most turns, cache common answers |

### Business model
- Per-minute pricing (market reference is about USD 0.25/min) plus a platform fee per brokerage. Analytics priced per broker seat.
- Developer launches: project-based pricing.
- Long-term: license our Gulf Arabic voice stack.

### Suggested positioning
> **"The compliant, Arabic-native AI sales team for UAE real estate,"** which answers every lead in 60 seconds, in the caller's language, with real DLD numbers, and never breaks a RERA/TDRA rule.

### Immediate next steps (this month)
1. Book 10 brokerage discovery calls.
2. Download DLD transactions and rental datasets; load them into Postgres.
3. Collect official PDFs of the regulations in §6.4 into `data/regulations/`.
4. Record 2–3 hours of role-play calls (us plus Arabic-speaking friends) to start the STT benchmark.
5. Build a 2-week English prototype on LiveKit/Pipecat to demo to design partners.

---

## Sources
- [Morgan Lewis: UAE telemarketing regulations (2024)](https://www.morganlewis.com/fr/blogs/sourcingatmorganlewis/2024/07/telemarketing-in-an-evolving-legal-landscape-uae-adopts-regulations-on-telemarketing-activities)
- [Clyde & Co: UAE tightens telemarketing regulations](https://www.clydeco.com/en/insights/2024/07/uae-tightens-telemarketing-regulations-what-you-ne)
- [Pinsent Masons: UAE telemarketing rules](https://pinsentmasons.com/out-law/news/uae-telemarketing-rules-ensure-businesses-operate-transparency-integrity)
- [Chambers: telemarketing fines and enforcement](https://chambers.com/articles/new-telemarketing-rules-and-regulations-in-the-uae-a-comprehensive-guide-to-fines-and-enforcement)
- [UAE Legislation portal](https://uaelegislation.gov.ae/en/legislations/2523/download)
- [Dubai Pulse: DLD transactions](https://www.dubaipulse.gov.ae/organisation/dld/service/dld-transactions)
- [DLD Open Data](https://dubailand.gov.ae/en/open-data/real-estate-data)
- [Arabian Business: DLD makes all real estate data public](https://www.arabianbusiness.com/money/wealth/money-wealth-real-estate/dubai-land-department-makes-all-real-estate-data-publicly-available-as-emirate-strengthens-transparency)
- [Khaleej Times: Neyox.ai launch](https://www.khaleejtimes.com/kt-network/neyoxai-launches-intelligent-voice-ai-agents-in-the-uae)
- [BIG LAB voice agents](https://biglab.ae/services/ai/ai-voice-agents/)
- [Grow50X: voice AI companies in Dubai](https://grow50x.ai/compare/voice-ai-companies-dubai)
- [MAJ Leads: AI voice agents for Dubai real estate](https://www.majleads.com/blog/ai-voice-agents-dubai-real-estate)
- [PRO TECHnology: voice AI for real estate & FM](https://www.protechnology.ae/voice-ai-agents-real-estate-facilities-management-uae)
