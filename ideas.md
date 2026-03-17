# Ideas Shortlist

> Pre-discovery. Goal: pick one to take through the full interview process.

---

## 11. AI Sync Potential Analyzer

**The play:** An indie artist uploads a track, an ML pipeline scores its sync licensing potential and explains why — lyric safety flags, mood/tempo suitability, placement category fit (automotive, drama, lifestyle), similar tracks that have been placed. $9-19/month for indie artists, $199-499/month for music publishers doing catalog analysis.

**Why it works:** No such tool exists publicly. Songtradr tried to build in this direction and was shut down in 2024 — validating the problem, leaving a gap. The underlying tech stack (Whisper for lyrics, Essentia/CLAP for audio features, similarity search) is mature and buildable by one person. Music expertise is directly applicable. Artists share scores on social = organic acquisition. Entirely build-first.

**Open Questions:**
- Is the indie artist segment too price-sensitive to sustain the business, and is the real buyer music publishers?
- What does a credible "sync potential score" need to include to be trusted by industry professionals vs. dismissed as a toy?
- How much audio analysis infrastructure cost per upload at scale?

---

## 12. Covered Call / Options Income Optimizer (revisited)

**The play:** Upload a CSV from any brokerage (Fidelity, Schwab, TD), get back ranked covered call recommendations for the week — optimal strike, expiration, premium yield — accounting for your actual cost basis. Brokerage-agnostic, portfolio-aware. $19-29/month retail, $79/month with tax optimization and earnings-avoidance logic.

**Why it works:** OptionsPlay comes closest but is locked inside specific brokerages. No self-serve, brokerage-agnostic optimizer exists for the $50k-$500k retail investor running a systematic income program. The r/thetagang community is large and actively looking for tools. Pure software — fully buildable before any customers.

**Open Questions:**
- How standardized are CSV exports across brokerages? (Fidelity vs. Schwab vs. TD formats)
- Is there a free-tier hook that generates virality? (e.g., "analyze your first 3 positions free")
- Regulatory: does giving specific options recommendations require any licensing?

---

## 13. SEC EDGAR Vertical Slice API

**The play:** The SEC's public EDGAR API is free but clunky — no full-text search, no normalized financials, no real-time streaming. sec-api.io already charges $55-239/month proving developers will pay for a clean wrapper. The gap is in a specific sub-vertical: e.g., a 13F institutional holdings change API, insider trading alerts API, or normalized XBRL financials endpoint with better latency and cleaner entity resolution than existing options.

**Why it works:** Proven willingness to pay (multiple companies charging $50-300/month). Developer-first, API-key-on-signup, no sales motion. Build-first model. Data engineering background is directly applicable.

**Open Questions:**
- Which sub-vertical has the least competition and clearest buyer? (13F changes for retail investors vs. insider trading for quants vs. financials for fintech)
- Is the moat defensibility real, or does a better-funded competitor just copy it?
- What specific latency/quality gap does sec-api.io leave open?

---

## 8. Lobster Dealer Price Transparency Tool

**The play:** Ex-vessel lobster prices vary significantly between dealers at different ports, but lobstermen have no easy way to compare — they call around or just accept their usual dealer's price. Aggregate dealer buy prices (crowdsourced from lobstermen or scraped from co-op postings) into a simple daily price feed. Charge lobstermen $20-40/month for access, or flip it and charge dealers for a listing/profile.

**Why it works:** The margin between ex-vessel ($4-6/lb) and retail ($15-25/lb) is enormous. Even small price improvements on hundreds of pounds per trip matter significantly. Cousin = immediate credibility and beta users. Community trust is everything in this industry — an outsider couldn't build this.

**Open Questions:**
- How do dealers currently post/communicate prices? Is there any existing aggregation (co-op boards, text chains)?
- Would lobstermen pay for this, or expect it free? What's the price sensitivity?
- Are dealers hostile to price transparency, and could they block adoption?
- How concentrated is the industry geographically? (Maine alone has ~5,000 licensed lobstermen — is that enough?)
- Is there a regulatory/co-op angle where this is already being attempted?

---

## 9. Trap Productivity & Catch Analytics App

**The play:** Lobstermen set 600-800 traps each and track productivity entirely from memory and experience. An app that logs catch weight/count by trap location and date, overlaid on a GPS chart, surfaces which areas and depths are actually producing. Over a season, the patterns become a real competitive edge.

**Why it works:** Pure software + phone GPS — no hardware required. Lobstermen are already on the water with phones. The data compounds: the longer you use it, the more valuable it gets. $30-50/month is trivial compared to the value of optimizing even 10% of trap placement on a 600-trap license.

**Open Questions:**
- What does data entry look like in practice — is logging catch per haul realistic on a working boat, or too much friction?
- Do lobstermen already use any apps for this? (Some use basic GPS chart plotters — is there overlap?)
- Is catch location data considered sensitive/proprietary? (Fishing spots are closely guarded secrets — does sharing concern them even in a private app?)
- What's the realistic paying population? Serious highliners vs. part-timers?

---

## 10. Boat-to-Restaurant Direct Lobster Subscription

**The play:** The provenance story sells at premium prices — restaurants and high-end consumers will pay more for "caught by Captain Mike off Stonington, Maine" than anonymous wholesale. Build a thin logistics layer that connects individual lobstermen to restaurant buyers and subscription consumers directly, handling scheduling, cold chain coordination, and the provenance storytelling. Take 10-15% of transaction value.

**Why it works:** The spread between ex-vessel and restaurant price is $10-20/lb — there's room for everyone to win. AI handles order matching and logistics scheduling. Cousin provides the first supply side. Provenance is a genuine differentiator that commodity distributors can't replicate.

**Open Questions:**
- Cold chain logistics is hard — who handles the shipping leg, and what does that cost per pound?
- Is the direct-to-restaurant model legally straightforward in Maine, or are there dealer licensing requirements?
- Several companies have tried this (Get Maine Lobster, etc.) — what specifically have they gotten wrong that leaves a gap?
- Is the better wedge B2C (subscription boxes) or B2B (restaurant accounts)? Different economics entirely.

---

## 1. Unclaimed Mechanical Royalty Recovery (MLC)

**The play:** Ingest the MLC's public unmatched works database, match against artist catalogs using ML, file claims automatically. Charge 20-25% of recovered royalties. Zero upfront cost to artist.

**Why it works:** Public data + AI matching makes the unit economics viable for one person. "Free money for your catalog" is a frictionless pitch.

**Open Questions:**
- What's the average claim size for a typical indie artist? ($50 vs $500 changes the model significantly)
- What % of MLC unmatched works are actually matchable with existing metadata?
- How long does MLC take to process and pay out a claim?
- Are there existing competitors (Songtrust, DistroKid's royalty tools, dedicated recovery services)?
- Is there any licensing/registration requirement to act as a rights administrator on behalf of artists?
- What's the acquisition channel? (Reddit music communities, indie label networks, DistroKid forums?)

---

## 2. Property Tax Appeal Service

**The play:** Pull public assessment records + recent sales comps. Flag over-assessed properties. Auto-generate appeal documents with AI. Charge 30-40% of first-year savings. Success fee only.

**Why it works:** ~40% of appeals succeed. Average savings $1,000+. Public data is increasingly accessible via county APIs. LLMs write the appeal letter. One person can handle hundreds of cases.

**Open Questions:**
- Which states have the most accessible data + easiest appeal processes?
- Do you need to be a licensed attorney or tax agent to represent homeowners in appeals?
- What's the competitive landscape? (NTPTS, TaxProper exist — why would we win?)

---

## 3. Music Catalog Valuation Tool

**The play:** Music catalog investing exploded (Hipgnosis, Primary Wave, etc.) but mid/small catalogs ($50k–$2M range) have no good pricing tools. Build a valuation model on public royalty data + streaming signals + sync history. Sell reports or a subscription to catalog investors, managers, and artists considering selling.

**Why it works:** This person knows where the data lives. The buyers have real money and real need — a bad valuation costs them six figures.

**Open Questions:**
- Is there enough public streaming + royalty data to build a defensible model, or do you need licensed data feeds?
- Who's the primary buyer — artists wanting to know what their catalog is worth, or investors doing due diligence?
- Competitors: Royalty Exchange, Sound Royalties — do they do valuations or just brokerage?

---

## 4. Sync Licensing Intelligence

**The play:** Track what music is getting placed in TV/film/ads by reconstructing placements from ASCAP/BMI public performance databases + Tunefind + trade press. Sell a subscription to music publishers, composers, and sync agents who want to know what styles/catalogs are actually getting placed and by which supervisors.

**Why it works:** This data exists but is incredibly fragmented. Music supervisors are a small, knowable community. The people who buy this have recurring budget.

**Open Questions:**
- Can public performance databases be reliably parsed to reconstruct sync placements?
- What's the existing landscape? (Tunefind is consumer-facing; is there a B2B equivalent?)
- Willingness to pay: $99/month vs $500/month — who's the right buyer tier?

---

## 5. Building Permit Data → Contractor Lead Gen

**The play:** Scrape/buy building permit data from county/city open data portals. A pulled electrical permit = a homeowner who might need HVAC, solar, or a panel upgrade. Sell warm leads to contractors in the relevant trade. Charge per lead or flat monthly subscription.

**Why it works:** Permit data is public and most lead gen for contractors is either expensive (Angi, HomeAdvisor) or low-quality (shared leads). Fresh permit-based leads are high-intent and exclusive.

**Open Questions:**
- How many counties/cities publish permit data via API vs requiring manual scraping?
- What do contractors actually pay per lead today on Angi/HomeAdvisor? (Sets pricing ceiling)
- Is there a geographic focus that would make this most defensible to start?

---

## 6. HOA Violation Dispute Service

**The play:** Homeowners get HOA violation notices and almost never push back — even when the violation is questionable. Build a service where homeowners upload their notice, you cross-reference their CC&Rs and local ordinances, and generate a formal dispute letter. Charge $49-99 per letter, or a subscription for repeat offenders.

**Why it works:** HOA disputes are emotional, low-stakes legally, and perfect for AI document generation. The CC&Rs + violation notice = structured input → structured output. No attorney needed in most states for written disputes.

**Open Questions:**
- How many HOA-governed homes are there in the US? (~30M — market is there)
- Is the acquisition channel viable? (Facebook HOA groups, Nextdoor, Google "HOA violation help")
- Does $49-99 per letter have enough volume to matter, or does this need a subscription model?

---

## 7. Covered Call / Options Income Optimizer

**The play:** Many retail investors hold dividend stocks and run basic covered call strategies but do it manually or not at all. Build a tool that scans a portfolio, identifies optimal covered call opportunities (strike/expiration/premium) to maximize income without sacrificing position, and tracks performance. Charge $15-30/month.

**Why it works:** The "wheel strategy" / options income crowd is large, engaged, and underserved by existing tools (tastytrade, thinkorswim are execution platforms, not optimizers). This is pure software — no data licensing needed beyond market data APIs (free tiers available).

**Open Questions:**
- Is there a real gap here? (Checking: OptionsPlay, Market Chameleon exist — what do they miss?)
- What's the IB/broker API landscape for reading portfolio positions programmatically?
- Does this need a brokerage integration to be useful, or is manual entry acceptable for v1?
