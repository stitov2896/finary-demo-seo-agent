# Agentic SEO Growth Loop — Finary Brokerage Accounts (France)

## Vision

A self-improving, multi-agent SEO system that continuously discovers content opportunities, creates optimized content, and refreshes existing pages — specifically for Finary's brokerage/investment products targeting French retail investors.

The system runs on a recurring schedule with human approval gates at critical points (topic selection, content publishing). It accumulates knowledge over time: every published article, every keyword that worked or failed, every competitor move is remembered and shapes future decisions.

**Product focus:** Finary brokerage accounts — PEA, CTO (Compte-Titres Ordinaire), ETF investing, stock trading, portfolio management.

**Market:** French retail investors (EU tax residents). All content in French.


---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SEO GROWTH LOOP                              │
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │  1. INTEL     │───▶│  2. RESEARCH │───▶│  3. CONTENT PLANNER  │  │
│  │  SCOUT        │    │  ANALYST     │    │  (approval gate)     │  │
│  │  every 3 days │    │  after scout │    │  after analyst       │  │
│  └──────────────┘    └──────────────┘    └──────────┬───────────┘  │
│                                                      │              │
│                                          ┌───────────▼───────────┐  │
│                                          │  4. CONTENT WRITER    │  │
│                                          │  after approval       │  │
│                                          └───────────┬───────────┘  │
│                                                      │              │
│                                          ┌───────────▼───────────┐  │
│                                          │  5. SEO VALIDATOR     │  │
│                                          │  before publish       │  │
│                                          └───────────────────────┘  │
│                                                                     │
│  ┌──────────────┐    ┌──────────────────────────────────────────┐  │
│  │  6. CONTENT   │    │  7. PERFORMANCE MONITOR                  │  │
│  │  REFRESHER    │    │  weekly digest + monthly deep review     │  │
│  │  monthly      │    │                                          │  │
│  └──────────────┘    └──────────────────────────────────────────┘  │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    BRAND CONTEXT (read-only)                   │  │
│  │  tone-of-voice.md │ keyword-clusters.json │ segments.json     │  │
│  │  product-facts.md │ content-inventory.json │ style-guide.md   │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow

```
INPUTS (recurring)                    OUTPUTS (accumulating)
─────────────────                     ──────────────────────
Google News (Exa)  ─┐                ┌─ Content briefs (approved)
Reddit /r/vosfinances─┤  ┌────────┐  ├─ Published articles (markdown)
Competitor blogs    ─┤  │ STATE  │  ├─ Keyword performance history
Forum questions     ─┤  │ .json  │  ├─ Content inventory (scored)
GSC rankings       ─┤  │        │  ├─ Weekly/monthly reports
GA4 traffic        ─┤  └────────┘  ├─ Refresh queue
DataForSEO SERP    ─┘                └─ Failed keyword memory
```

---

## Project Structure

```
C:/Users/stito/finary-seo/
├── config/
│   ├── brand-context.md          # Finary brand, tone of voice, positioning
│   ├── product-facts.md          # Brokerage account features, pricing, legal
│   ├── style-guide.md            # Writing style rules for all content
│   ├── keyword-clusters.json     # Master keyword list with clusters
│   ├── segments.json             # Audience segments (from competitor analysis)
│   └── competitors.json          # Competitor URLs and profiles
│
├── state/
│   ├── state.json                # Pipeline state tracking
│   ├── content-inventory.json    # All published content + performance scores
│   ├── keyword-history.json      # Keyword ranking history over time
│   ├── failed-keywords.json      # Keywords that didn't work (never repeat)
│   └── published-topics.json     # Topics already covered (avoid duplication)
│
├── data/
│   ├── intel/                    # Intelligence reports (timestamped)
│   │   └── 2026-04-13_intel.json
│   ├── opportunities/            # Scored opportunity lists
│   │   └── 2026-04-13_opportunities.json
│   ├── briefs/                   # Approved content briefs
│   │   └── 2026-04-13_brief_pea-vs-assurance-vie.json
│   ├── drafts/                   # Written content (pre-validation)
│   │   └── 2026-04-13_pea-vs-assurance-vie.md
│   ├── published/                # Final validated content
│   │   └── 2026-04-13_pea-vs-assurance-vie.md
│   └── refreshes/                # Content refresh recommendations
│       └── 2026-04-monthly-refresh.json
│
├── reports/
│   ├── weekly/                   # Weekly performance digests
│   └── monthly/                  # Monthly strategic reviews
│
└── run_log.md                    # Full execution history
```

---

## Agent 1: Intelligence Scout

**Purpose:** Continuously scan the information landscape for content opportunities relevant to French retail investing and Finary's products.

**Schedule:** Every 3 days (RemoteTrigger cron: `0 6 */3 * *` — 6am every 3rd day)

**Model:** Sonnet (fast, cost-effective for scanning)

**Tools:** Exa API, Playwright, Bash, Read, Write

### What It Scans

#### A. Financial News (France + Europe)
- French market news: CAC 40 movements, ECB rate decisions, French tax law changes
- EU regulatory changes affecting retail investors (MiFID, DORA, etc.)
- New product launches by competitors (new ETFs, fee changes, feature announcements)
- IPOs and major stock events relevant to French investors

**Exa queries (rotate across runs):**
```
"investissement France 2026" (last 3 days)
"PEA actualités" (last 3 days)
"bourse France particuliers" (last 3 days)
"ETF Europe" (last 3 days)
"courtage en ligne France" (last 7 days)
"épargne placement 2026" (last 3 days)
"CAC 40 analyse" (last 3 days)
"taux BCE impact épargnants" (last 7 days)
"réforme fiscale investissement France" (last 7 days)
```

#### B. User Questions & Discussions
- Reddit: /r/vosfinances, /r/financefrance, /r/investissement
- Forums: Boursorama forums, Les Échos community discussions
- Google "People Also Ask" for core keyword clusters (via DataForSEO SERP)
- Trustpilot / App Store reviews of competitors (pain points, feature requests)

**Exa queries:**
```
"site:reddit.com/r/vosfinances" (last 7 days)
"site:reddit.com investir France" (last 7 days)
"avis [competitor]" (last 14 days for each competitor)
"meilleur PEA 2026 avis" (last 14 days)
"problème courtier en ligne France" (last 14 days)
"frais courtage trop cher" (last 14 days)
```

#### C. Competitor Content Monitoring
- New blog posts from: Trade Republic, XTB, BoursoBank, Fortuneo, Bourse Direct, Saxo
- New landing pages or product pages
- Changes to pricing pages
- New comparison or educational content

**Exa queries:**
```
"site:traderepublic.com blog" (last 14 days)
"site:xtb.com/fr actualités" (last 14 days)
"site:boursobank.com bourse guide" (last 14 days)
"site:fortuneo.fr bourse" (last 14 days)
"site:boursedirect.fr apprendre" (last 14 days)
```

#### D. Trending Investment Topics
- Trending tickers on French markets
- Viral finance content on social media
- New financial products (new ETF launches, new savings products)
- Seasonal triggers (tax season in April-May, rentrée in September, year-end tax optimization)

#### E. Regulatory & Tax Changes
- AMF announcements
- French tax law updates (PLF — Projet de Loi de Finances)
- European regulatory changes
- FGDR (deposit guarantee) updates

### Output

`data/intel/YYYY-MM-DD_intel.json`:
```json
{
  "scan_date": "2026-04-13",
  "sources_scanned": 45,
  "signals_found": 23,
  "signals": [
    {
      "id": "sig_001",
      "type": "news|question|competitor|trend|regulatory",
      "source": "reddit.com/r/vosfinances",
      "source_url": "https://...",
      "title": "Thread: PEA chez Trade Republic vs Boursobank?",
      "summary": "User comparing PEA fees, 47 comments discussing...",
      "relevance_score": 8,
      "finary_product_match": ["PEA", "brokerage"],
      "segment_match": ["SEG-02-A: PEA Discoverer", "SEG-03: Fee-Conscious Optimizer"],
      "keyword_opportunities": ["PEA trade republic avis", "PEA boursobank frais", "comparatif PEA 2026"],
      "urgency": "medium",
      "content_angle": "Comparison article: PEA fees across all major brokers in France",
      "timestamp": "2026-04-12T14:30:00Z"
    }
  ],
  "top_signals": ["sig_001", "sig_007", "sig_012"],
  "competitor_moves": [
    {
      "competitor": "Saxo Bank",
      "action": "New blog post about PEA transfer process",
      "url": "https://...",
      "response_needed": true,
      "suggested_response": "Write a better, more detailed PEA transfer guide"
    }
  ],
  "trending_topics": ["ETF obligataire", "PER vs PEA", "dividendes CAC 40"],
  "seasonal_triggers": ["Déclaration fiscale approaching (May deadline)"]
}
```

### Critical Rules
1. NEVER fabricate news or quotes — all signals must have source URLs
2. Deduplicate across scans (check published-topics.json)
3. Score relevance against Finary's product and segments
4. Flag time-sensitive opportunities (regulatory changes, market events)
5. Source `~/.claude/.env` before any API calls

---

## Agent 2: Opportunity Researcher

**Purpose:** Take raw intelligence signals and evaluate them as concrete content opportunities — with keyword data, SERP analysis, competitive gap assessment, and format recommendations.

**Schedule:** Triggered after Agent 1 completes (or on-demand)

**Model:** Opus (complex analysis requiring reasoning)

**Tools:** DataForSEO API, Exa API, Playwright, Read, Write

### Process

#### Step 1: Signal Filtering
- Read latest `data/intel/YYYY-MM-DD_intel.json`
- Read `config/keyword-clusters.json` — match signals to existing clusters
- Read `config/segments.json` — match signals to audience segments
- Read `state/published-topics.json` — filter out already-covered topics
- Read `state/failed-keywords.json` — avoid keywords that previously failed
- Keep only signals scoring 6+ relevance

#### Step 2: Keyword Research (per signal)
For each promising signal, use DataForSEO to:
- Pull search volume for candidate keywords (location: France, language: French)
- Get keyword difficulty scores
- Find related keywords and long-tail variations
- Identify keyword clusters that can be targeted together
- Check current SERP composition (who ranks, what content type)

**DataForSEO calls:**
```python
# Keyword research
get_keyword_research(keyword="ouvrir PEA en ligne", location_code=2250)  # France

# SERP analysis
get_serp_data(keyword="meilleur PEA 2026", location_code=2250, limit=20)

# Competitor rankings
get_rankings(domain="traderepublic.com", keywords=["PEA"], location_code=2250)
```

#### Step 3: SERP Analysis
For each top candidate keyword:
- What currently ranks in top 10? (competitor content? generic sites? forums?)
- What content type dominates? (articles, tools, comparisons, videos)
- Are there SERP features? (Featured snippets, People Also Ask, Knowledge Panel)
- What's the content quality of top results? (word count, depth, freshness)
- What's missing from top results? (angles, data, tools none of them provide)

#### Step 4: Competitive Gap Assessment
Cross-reference with competitor analysis data:
- Do any of the 9 analyzed competitors have content on this topic?
- If yes, how strong is it? (based on their domain authority, content depth, rankings)
- What angle could Finary take that no competitor covers?
- Is there a "10x content" opportunity (dramatically better than existing results)?

#### Step 5: Format Recommendation
Based on SERP analysis and topic type, recommend content format:

| Format | When to Use | Example |
|--------|-------------|---------|
| **Long-form guide** (2000-4000 words) | Informational intent, complex topics | "Guide complet du PEA en 2026" |
| **Comparison article** (1500-2500 words) | Transactional intent, "vs" queries | "PEA Finary vs Trade Republic vs Boursobank" |
| **Calculator / interactive tool** | "Combien" queries, fee calculations | "Calculateur de frais de courtage PEA" |
| **Quick overview** (800-1200 words) | Trending news, time-sensitive | "Impact de la décision BCE sur votre PEA" |
| **Listicle** (1500-2500 words) | "Meilleur" queries, discovery intent | "10 meilleurs ETF PEA en 2026" |
| **Stock/ETF analysis page** (1000-2000 words) | Specific ticker queries | "Analyse ETF Amundi MSCI World (CW8)" |
| **FAQ page** (1000-1500 words) | Question-based queries | "PEA : toutes les questions fréquentes" |
| **Glossary entry** (500-800 words) | Definition queries | "Qu'est-ce qu'un ordre à cours limité ?" |
| **News reaction** (600-1000 words) | Breaking news, market events | "CAC 40 : ce que le rebond signifie pour votre PEA" |
| **Data-driven report** (1500-3000 words) | Original research, statistics | "Étude : combien les Français perdent en frais de courtage" |
| **Template / checklist** (800-1500 words) | Actionable how-to content | "Checklist : ouvrir son PEA en 10 étapes" |

#### Step 6: Opportunity Scoring

Each opportunity gets a composite score (1-10):

```
OPPORTUNITY SCORE = (
  Search Volume Weight    × 0.20  +  # Monthly searches (normalized 1-10)
  Keyword Difficulty Inv  × 0.15  +  # Inverse difficulty (easier = higher)
  Segment Alignment       × 0.20  +  # How well it maps to Finary's target segments
  Competitive Gap         × 0.20  +  # How weak existing SERP results are
  Timeliness             × 0.10  +  # Time-sensitive bonus (trending, seasonal)
  Revenue Potential       × 0.15     # Proximity to conversion (PEA signup, account open)
)
```

**Score interpretation:**
- 8-10: Publish immediately — high volume, low competition, strong product fit
- 6-7: Strong candidate — worth writing, schedule within 2 weeks
- 4-5: Moderate — write if capacity allows, good for content calendar
- 1-3: Low priority — park for later or skip

### Output

`data/opportunities/YYYY-MM-DD_opportunities.json`:
```json
{
  "analysis_date": "2026-04-13",
  "signals_evaluated": 15,
  "opportunities_found": 8,
  "opportunities": [
    {
      "id": "opp_001",
      "source_signal": "sig_001",
      "topic": "Comparatif PEA 2026 : Finary vs Trade Republic vs Boursobank",
      "primary_keyword": "comparatif PEA 2026",
      "keyword_cluster": ["meilleur PEA", "PEA en ligne", "PEA frais comparaison", "ouvrir PEA pas cher"],
      "search_volume_monthly": 2900,
      "keyword_difficulty": 35,
      "current_serp_quality": "weak — top results are outdated (2024) or generic listicles",
      "serp_features": ["People Also Ask", "Featured Snippet (table)"],
      "target_segments": ["SEG-02-A: PEA Discoverer", "SEG-03: Fee-Conscious Optimizer"],
      "recommended_format": "comparison_article",
      "recommended_length": "2000-2500 words",
      "content_angle": "Real fee calculation across 9 brokers with interactive comparison table",
      "differentiator": "Include actual trade cost for 3 investor profiles (€100/mo DCA, €500/mo, €1000/mo)",
      "schema_type": "Article + Product + FAQ",
      "internal_links_to": ["/pea-guide", "/ouvrir-pea", "/frais-courtage"],
      "cta_placement": "After fee comparison table — 'Ouvrir mon PEA Finary'",
      "opportunity_score": 8.5,
      "score_breakdown": {
        "search_volume": 8,
        "difficulty_inverse": 7,
        "segment_alignment": 9,
        "competitive_gap": 9,
        "timeliness": 7,
        "revenue_potential": 9
      },
      "confidence": "HIGH — keyword data from DataForSEO, SERP manually verified",
      "estimated_traffic_potential": "800-1500 organic visits/month at position 3-5"
    }
  ],
  "rejected": [
    {
      "signal": "sig_003",
      "reason": "Keyword too competitive (KD 85), dominated by government sites"
    }
  ]
}
```

### Critical Rules
1. Always check `state/published-topics.json` to avoid writing duplicate content
2. Always check `state/failed-keywords.json` to avoid repeating failed approaches
3. DataForSEO rate limit: max 2000 requests/min, add delays between calls
4. SERP analysis must be for France (location_code=2250, language=fr)
5. Score honestly — don't inflate scores to create artificial urgency

---

## Agent 3: Content Planner

**Purpose:** Review all scored opportunities, create detailed content briefs, and present them for human approval. This is the primary approval gate — no content is written without manager sign-off.

**Schedule:** Triggered after Agent 2 (or weekly review)

**Model:** Opus (strategic decision-making)

**Tools:** Read, Write, AskUserQuestion

### Process

#### Step 1: Portfolio Review
- Read all pending opportunities from `data/opportunities/`
- Read `state/content-inventory.json` — what exists, what's performing
- Read `config/keyword-clusters.json` — ensure coverage across all clusters
- Read current content calendar (if exists)
- Identify gaps: which keyword clusters have NO content yet?

#### Step 2: Prioritization
Rank opportunities considering:
- Raw opportunity score (from Agent 2)
- Cluster coverage gap (uncovered clusters get priority)
- Content calendar balance (mix of formats, topics, segments)
- Seasonal relevance (tax season content before April, year-end before December)
- Resource availability (how many articles can be produced this week?)

#### Step 3: Brief Creation
For each selected topic, create a detailed brief:

```json
{
  "brief_id": "brief_2026-04-13_001",
  "topic": "Comparatif PEA 2026 : quel courtier en ligne choisir ?",
  "status": "pending_approval",
  "format": "comparison_article",
  "target_keyword": "comparatif PEA 2026",
  "secondary_keywords": ["meilleur PEA", "PEA en ligne pas cher", "frais PEA comparaison"],
  "target_length": "2000-2500 words",
  "target_segments": ["PEA Discoverer", "Fee-Conscious Optimizer"],
  "outline": [
    "H1: Comparatif PEA 2026 : quel courtier choisir pour investir en bourse ?",
    "Intro: Why PEA matters + what we'll compare (150 words)",
    "H2: Qu'est-ce qu'un PEA et pourquoi en ouvrir un ?",
    "  - Quick PEA explainer (tax advantages, €150K cap, 5-year rule)",
    "H2: Les critères de comparaison",
    "  - Frais de courtage, frais de garde, produits disponibles, appli mobile, service client",
    "H2: Comparatif détaillé des 9 courtiers PEA en France",
    "  - Comparison table (interactive if possible)",
    "  - Per-broker mini-review (3-4 sentences each)",
    "H2: Simulation : combien payez-vous réellement en frais ?",
    "  - 3 profiles: €100/mo, €500/mo, €1000/mo DCA",
    "  - Table showing actual annual cost per broker per profile",
    "H2: Pour quel profil choisir quel courtier ?",
    "  - Mapping segments to recommended brokers",
    "H2: Comment transférer son PEA (et pourquoi c'est plus simple qu'on croit)",
    "  - Address PEA transfer anxiety",
    "H2: FAQ — Questions fréquentes sur le PEA",
    "  - 5-7 real questions from Reddit/forums",
    "Conclusion + CTA: 'Ouvrir mon PEA Finary'"
  ],
  "schema_markup": ["Article", "FAQ", "Product"],
  "internal_links": ["/pea-guide", "/ouvrir-pea", "/frais-courtage", "/etf-pea"],
  "tone_notes": "Accessible but authoritative. Use 'vous' (formal). Include real numbers. Don't bash competitors — show objective comparison. Finary positioned as modern, transparent alternative.",
  "data_to_include": [
    "Actual fee structures from all 9 competitors (from competitor analysis)",
    "PEA rules: €150K cap, EU stocks, 5-year tax advantage",
    "FGDR protection: €70,000 guarantee"
  ],
  "cta_strategy": "Primary CTA after fee comparison table. Secondary CTA after FAQ. Soft CTA in intro ('découvrir Finary').",
  "estimated_impact": "800-1500 organic visits/month at position 3-5",
  "publish_deadline": "2026-04-20"
}
```

#### Step 4: Present to Manager
Use AskUserQuestion to present top 3-5 briefs for approval:
- Show topic, format, score, estimated impact, deadline
- Manager can approve, modify, reject, or re-prioritize
- Approved briefs move to `data/briefs/` with status "approved"

### Content Ideas by Category (Expanded)

#### Evergreen Content (high search volume, stable demand)
- "Guide complet du PEA : fonctionnement, avantages, fiscalité"
- "Comment ouvrir un PEA en ligne en 2026"
- "Meilleurs ETF éligibles PEA : notre sélection"
- "PEA vs Compte-Titres : lequel choisir ?"
- "PEA vs Assurance-vie : comparaison complète"
- "Investir en bourse pour les débutants : guide pas à pas"
- "DCA (investissement programmé) : le guide pour automatiser"
- "Frais de courtage : comment les réduire"
- "Dividendes : comment les percevoir et les déclarer"
- "Ordre de bourse : les différents types expliqués"

#### Comparison & Alternative Pages (high conversion intent)
- "Comparatif PEA 2026 : [all brokers]"
- "Finary vs Trade Republic" (and vs each competitor)
- "Alternative à [Competitor] : pourquoi choisir Finary"
- "Top 5 courtiers en ligne pour PEA en France"
- "Quel courtier pour investir avec 100€ par mois ?"
- "Meilleur courtier pour ETF en France"
- "PEA Jeunes : quel courtier choisir pour un mineur ?"

#### Calculator & Tool Pages (high engagement, link-worthy)
- "Calculateur de frais de courtage PEA"
- "Simulateur de rendement PEA sur 5, 10, 20 ans"
- "Calculateur : combien coûtent réellement vos frais de courtage"
- "Simulateur DCA : combien investir par mois pour atteindre votre objectif"
- "Calculateur d'avantage fiscal PEA vs CTO"
- "Comparateur de frais : votre banque vs courtier en ligne"

#### Stock & ETF Analysis (programmatic SEO potential)
- "[ETF Name] : analyse, performance, frais" — for top 50 ETFs éligibles PEA
- "[Stock Name] : faut-il investir ? Analyse complète" — for CAC 40 stocks
- "Meilleurs ETF [sector] éligibles PEA" — by sector (tech, santé, dividendes, etc.)
- "ETF Amundi vs Lyxor vs iShares : lequel choisir ?"

#### Seasonal & News-Reactive (time-sensitive, trending)
- "Déclaration fiscale PEA : guide [year]" — before May tax deadline
- "Impact [market event] sur votre PEA" — within 24h of major events
- "Bilan [year] : performance du CAC 40 et perspectives [year+1]"
- "Nouveaux ETF éligibles PEA : ce qui change en [year]"
- "PLF [year] : ce qui change pour les investisseurs"

#### Educational Content (beginner funnel)
- "Investir en bourse : par où commencer ?"
- "Les 5 erreurs des débutants en bourse"
- "Livret A vs bourse : que faire de son épargne ?"
- "Comprendre les ETF en 5 minutes"
- "La fiscalité de la bourse en France : guide simple"
- "Comment lire un graphique boursier"
- "Qu'est-ce que le risque en investissement ?"

#### FAQ & Problem-Solving (long-tail, high intent)
- "PEA bloqué 5 ans : vrai ou faux ?"
- "Peut-on avoir plusieurs PEA ?"
- "Comment transférer son PEA : guide étape par étape"
- "PEA plein : que faire quand on atteint 150 000€ ?"
- "Quelles actions acheter pour un PEA en 2026 ?"
- "Courtier en ligne : est-ce sûr ? (garanties FGDR)"
- "Retirer de l'argent de son PEA : quelles conséquences ?"

#### Data-Driven / Original Research (link-building, authority)
- "Étude : combien les Français perdent en frais de courtage chaque année"
- "Baromètre des courtiers en ligne France [year]"
- "Enquête : le profil type de l'investisseur PEA en France"
- "Performance historique : PEA vs Livret A vs Assurance-vie sur 20 ans"

---

## Agent 4: Content Writer

**Purpose:** Write high-quality, SEO-optimized content following approved briefs, brand context, and technical SEO requirements.

**Schedule:** Triggered after brief approval (on-demand)

**Model:** Opus (quality writing requires strong reasoning)

**Tools:** Read, Write, Bash, Exa API (for fact-checking), existing SEO skills

### Process

#### Step 1: Context Loading
Read all required context files before writing:
- `config/brand-context.md` — Finary positioning, value props, legal constraints
- `config/style-guide.md` — writing rules, tone, vocabulary
- `config/product-facts.md` — accurate product details, pricing, features
- The approved brief from `data/briefs/`
- Relevant competitor data from `trendwatching/projects/` (for comparison articles)
- Existing content on similar topics from `state/content-inventory.json`

#### Step 2: Research & Fact-Gathering
- Use Exa to find the latest data points needed for the article
- Verify all competitor pricing and features are current (visit competitor pages)
- Check for recent regulatory changes that affect the topic
- Find real user questions from Reddit/forums to include in FAQ sections

#### Step 3: Structure
- Follow the outline from the approved brief
- Apply heading hierarchy (H1 > H2 > H3, never skip levels)
- Plan internal link placement (from `config/keyword-clusters.json`)
- Plan schema markup blocks (from seo-schema skill)
- Plan CTA placement per brief instructions

#### Step 4: Writing
Follow these content rules:

**SEO Writing Rules:**
- Primary keyword in H1 (naturally, not forced)
- Primary keyword in first 100 words
- Secondary keywords distributed across H2s
- Short paragraphs (3-4 sentences max)
- Use lists, tables, and callout boxes for scannability
- Include data/numbers in every section (specific, not vague)
- Address one "People Also Ask" question per section where relevant
- Write in the structure that maximizes chances of Featured Snippet capture:
  - Definition queries → clear definition in first paragraph
  - "How to" queries → numbered step list
  - Comparison queries → table format
  - "Best" queries → numbered list with criteria

**Tone Rules** (from brand-context.md):
- "Vous" (formal but approachable)
- Expert but not condescending
- Data-driven — always show numbers
- Transparent — acknowledge trade-offs, don't hide downsides
- Action-oriented — every section ends with a concrete next step
- No hyperbole ("le meilleur" must be backed by data)
- No fear-mongering (market drops are normal, not crises)

**Legal Constraints:**
- Never provide specific investment advice ("achetez ceci")
- Always include "Les performances passées ne préjugent pas des performances futures"
- Mention AMF regulation where relevant
- FGDR guarantee (€70,000) is fact, always cite correctly
- PEA rules must be accurate (€150,000 cap, 5-year fiscal advantage, EU stocks only)

#### Step 5: Metadata Generation
For each article, generate:
- Meta title (50-60 chars, primary keyword front-loaded)
- Meta description (150-160 chars, includes CTA and keyword)
- OG title and description
- Canonical URL suggestion
- Schema markup (JSON-LD) using seo-schema skill templates:
  - Article schema (always)
  - FAQ schema (if FAQ section present — gov/healthcare only per current rules, but prepare for potential future re-enablement)
  - Product schema (if comparing products)
  - BreadcrumbList schema (always)

### Output

`data/drafts/YYYY-MM-DD_[slug].md` — full article in markdown with frontmatter:
```yaml
---
title: "Comparatif PEA 2026 : quel courtier en ligne choisir ?"
meta_title: "Comparatif PEA 2026 : Top 9 Courtiers en Ligne | Finary"
meta_description: "Comparez les frais, produits et services de 9 courtiers PEA en France. Simulation de coûts réels pour 3 profils d'investisseur. Guide mis à jour avril 2026."
slug: comparatif-pea-2026
primary_keyword: "comparatif PEA 2026"
secondary_keywords: ["meilleur PEA", "PEA en ligne", "frais PEA comparaison"]
target_segments: ["PEA Discoverer", "Fee-Conscious Optimizer"]
word_count: 2350
schema_types: ["Article", "FAQ", "BreadcrumbList"]
internal_links: ["/pea-guide", "/ouvrir-pea", "/frais-courtage"]
last_updated: "2026-04-13"
author: "Finary"
brief_id: "brief_2026-04-13_001"
---

[Article content in markdown...]
```

---

## Agent 5: SEO Validator

**Purpose:** Quality-check every piece of content before publishing. Applies technical SEO standards, content quality checks, schema validation, and brand compliance.

**Schedule:** Triggered after Agent 4 writes a draft

**Model:** Sonnet (checklist-based validation, fast)

**Tools:** Read, Write, existing SEO skills (seo-content, seo-schema, seo-technical, seo-geo)

### Validation Checklist

#### A. On-Page SEO
- [ ] Primary keyword in H1
- [ ] Primary keyword in first 100 words
- [ ] Primary keyword in meta title (front-loaded)
- [ ] Primary keyword in meta description
- [ ] Secondary keywords used in H2s (naturally)
- [ ] No keyword stuffing (keyword density < 2.5%)
- [ ] Meta title length: 50-60 characters
- [ ] Meta description length: 150-160 characters
- [ ] Heading hierarchy correct (H1 > H2 > H3, no skips)
- [ ] Only ONE H1 per page
- [ ] At least 3 internal links to relevant pages
- [ ] All internal links use descriptive anchor text
- [ ] At least 1 external link to authoritative source (AMF, BCE, etc.)

#### B. Content Quality (seo-content skill)
- [ ] Word count meets target from brief
- [ ] Flesch Reading Ease score appropriate for audience (target: 40-60 for finance)
- [ ] No thin content sections (every H2 has 150+ words)
- [ ] Original data/analysis present (not just rewording competitors)
- [ ] E-E-A-T signals: author attribution, source citations, factual claims verified
- [ ] Actionable conclusion with clear next step
- [ ] FAQ section answers real user questions (verified from Reddit/forums)

#### C. Schema Markup (seo-schema skill)
- [ ] Article schema present and valid JSON-LD
- [ ] BreadcrumbList schema present
- [ ] Product schema (if comparison article)
- [ ] All schema fields populated correctly
- [ ] No deprecated schema types used (HowTo, SpecialAnnouncement)
- [ ] datePublished and dateModified set

#### D. AI Search Optimization (seo-geo skill)
- [ ] Key claims structured in 134-167 word citeable blocks
- [ ] Clear, quotable definitions for key terms
- [ ] Table/list format for comparison data (AI systems extract these)
- [ ] Brand mention natural and contextual
- [ ] Content answers questions directly (not buried in prose)

#### E. Brand Compliance
- [ ] Uses "vous" throughout (not "tu")
- [ ] No specific investment advice ("achetez X")
- [ ] Legal disclaimers present where required
- [ ] PEA rules stated accurately
- [ ] Competitor comparisons are factual, not derogatory
- [ ] CTAs match brief instructions
- [ ] No emojis in body text (unless brief specifies)

#### F. Technical
- [ ] No broken internal links (verify paths exist in content inventory)
- [ ] Images have alt text (if any)
- [ ] No duplicate H2 text
- [ ] Slug is URL-friendly (lowercase, hyphens, no special chars)
- [ ] Canonical URL set correctly

### Output

Validation report appended to the draft file:
```yaml
# --- SEO VALIDATION REPORT ---
validation_date: "2026-04-13"
status: "PASS" | "FAIL" | "PASS_WITH_WARNINGS"
score: 92/100
issues:
  - severity: warning
    category: content_quality
    detail: "H2 'Qu'est-ce qu'un PEA' section only 120 words (target: 150+)"
    fix: "Expand with PEA history and EU eligibility details"
  - severity: info
    category: schema
    detail: "FAQ schema prepared but currently restricted to gov/healthcare. Ready for future re-enablement."
passed_checks: 28
failed_checks: 0
warning_checks: 2
```

If FAIL: returns to Agent 4 with specific issues to fix.
If PASS: moves draft to `data/published/` and updates `state/content-inventory.json`.

---

## Agent 6: Content Refresher

**Purpose:** Monthly scan of all existing published content. Identifies articles that need updates, finds fresh data to add, and proposes (or executes) refreshes to keep content ranking well.

**Schedule:** Monthly (RemoteTrigger cron: `0 8 1 * *` — 8am on the 1st of each month)

**Model:** Opus (requires judgment about what to update and how)

**Tools:** GSC module, GA4 module, DataForSEO, Exa API, Read, Write

### Process

#### Step 1: Performance Scan
Pull current performance for all published content:

**From GSC (google_search_console.py):**
- `get_keyword_positions()` — all keyword rankings
- `get_quick_wins()` — keywords at positions 11-20 (biggest ROI)
- `get_low_ctr_pages()` — high impressions, low clicks
- `get_position_changes()` — what improved/declined this month
- `get_trending_queries()` — new queries gaining traction

**From GA4 (google_analytics.py):**
- `get_top_pages()` — traffic leaders
- `get_declining_pages()` — pages losing traffic
- `get_page_trends()` — trend direction per page
- `get_conversions()` — which pages drive signups

#### Step 2: Content Audit
For each published article:
- Is it still factually accurate? (check competitor pricing, PEA rules, tax rates)
- Is the data current? (articles referencing "2025" data need "2026" updates)
- Have competitors published better content on the same topic?
- Are there new questions on Reddit/forums that should be added?
- Has the keyword landscape changed? (new keywords to target, old ones declining)

#### Step 3: Refresh Prioritization

Priority matrix:
| Signal | Priority | Action |
|--------|----------|--------|
| Traffic declining >20% month-over-month | URGENT | Deep refresh (rewrite sections, add data, extend) |
| Position 11-20 keywords | HIGH | Targeted optimization (add keyword sections, improve depth) |
| Low CTR (impressions high, clicks low) | HIGH | Rewrite meta title and description |
| Outdated data (>6 months) | MEDIUM | Update numbers, dates, competitor info |
| New PAA questions appeared | MEDIUM | Add FAQ section or expand existing |
| Competitor published better content | HIGH | Major content upgrade to regain position |
| Trending query matches existing content | MEDIUM | Add trending angle to existing article |
| High-converting page declining | URGENT | Full audit and refresh (revenue at risk) |

#### Step 4: Refresh Execution
For URGENT and HIGH priority items:
- Generate a refresh brief (similar to a new content brief but focused on what to change)
- Present to manager for approval via AskUserQuestion
- After approval, Agent 4 (Content Writer) executes the refresh
- Agent 5 (Validator) re-validates the updated content

#### Step 5: Keyword Memory Update
- Add newly failed keywords to `state/failed-keywords.json`
- Update `state/keyword-history.json` with monthly position snapshots
- Update `state/content-inventory.json` with current performance scores

### Output

`data/refreshes/YYYY-MM_monthly-refresh.json`:
```json
{
  "month": "2026-04",
  "total_articles_reviewed": 47,
  "refresh_needed": 12,
  "urgent": 3,
  "high": 5,
  "medium": 4,
  "overall_portfolio_health": "73/100",
  "traffic_trend": "+8% vs previous month",
  "top_performing": [
    {"url": "/comparatif-pea-2026", "traffic": 1450, "trend": "rising", "top_keyword_position": 4}
  ],
  "needs_refresh": [
    {
      "url": "/guide-pea",
      "priority": "urgent",
      "reason": "Traffic down 25% MoM. Competitor (XTB) published comprehensive PEA guide that now outranks us.",
      "current_position": 8,
      "previous_position": 4,
      "actions": [
        "Update fee comparison table with April 2026 data",
        "Add section on PEA transfer (competitor covers this, we don't)",
        "Expand FAQ with 3 new Reddit questions",
        "Update meta title to include '2026'"
      ],
      "estimated_impact": "Recover ~400 visits/month"
    }
  ],
  "new_keyword_opportunities": [
    {
      "keyword": "PEA jeune 2026",
      "volume": 1200,
      "existing_content": "Briefly mentioned in /guide-pea",
      "recommendation": "Create dedicated article or significantly expand existing section"
    }
  ]
}
```

---

## Agent 7: Performance Monitor

**Purpose:** Continuous performance tracking with weekly digests and monthly strategic reports. Provides the data foundation for all other agents' decisions.

**Schedule:**
- Weekly digest: Every Monday 8am (cron: `0 8 * * 1`)
- Monthly deep review: 1st of each month alongside Content Refresher

**Model:** Sonnet (data aggregation, not complex reasoning)

**Tools:** GSC module, GA4 module, DataForSEO, Read, Write, gws CLI (for Google Sheets)

### Weekly Digest Contents

1. **Traffic Overview**: total organic visits, vs previous week, vs 4-week average
2. **Keyword Movements**: keywords that gained/lost 3+ positions
3. **Top 10 Pages**: by traffic, by conversions, by growth rate
4. **New Keywords Discovered**: queries appearing for the first time in GSC
5. **Competitor Alerts**: any new content from monitored competitors (from Exa)
6. **Content Published This Week**: performance of recently published articles
7. **Action Items**: 3-5 specific things to do this week

### Monthly Strategic Report

Full performance analysis + forward-looking strategy:
- Portfolio health score (0-100)
- Keyword position distribution (top 3, top 10, top 20, top 50)
- Content ROI: which articles drive the most signups
- Competitive positioning: are we gaining or losing ground
- Keyword cluster coverage: which clusters need more content
- Recommendations for next month's content calendar

### Output

Weekly: `reports/weekly/YYYY-WXX_digest.json` + optional Google Sheet update
Monthly: `reports/monthly/YYYY-MM_strategic.json` + Google Sheet + Google Doc

---

## Required Config Files

### brand-context.md
```markdown
# Finary Brand Context — Brokerage Accounts

## Company
Finary is a French fintech offering brokerage accounts (PEA, CTO) 
to retail investors in France. Founded in [year]. Regulated by AMF.

## Core Value Proposition
[To be filled: what makes Finary different from Trade Republic, XTB, etc.]

## Products
- PEA (Plan d'Épargne en Actions): [features, pricing, limits]
- CTO (Compte-Titres Ordinaire): [features, pricing]
- Available instruments: [ETFs, stocks, bonds — specifics]
- Pricing: [fee structure, any zero-commission claims]
- Minimum investment: [amount]

## Target Audience
Primary: French retail investors (25-45), beginners to intermediate
Secondary: [other segments]

## Key Differentiators
1. [Differentiator 1]
2. [Differentiator 2]
3. [Differentiator 3]

## Trust Signals
- AMF registration: [number]
- FGDR protection: €70,000
- [Other certifications, awards]

## Positioning Statement
[One-sentence positioning: "For [audience], Finary is the [category] that [benefit] because [reason]"]
```

### style-guide.md
```markdown
# Finary Content Style Guide

## Tone
- Register: formal ("vous"), but accessible and warm
- Expert: show knowledge through data, not jargon
- Transparent: acknowledge trade-offs, never oversell
- Action-oriented: every article helps the reader DO something

## Vocabulary
- Use: "investir", "épargne", "patrimoine", "rendement"
- Avoid: "gagner de l'argent facilement", "devenir riche", "astuce"
- Technical terms: always define on first use
- Competitor mentions: factual, never derogatory

## Structure
- Paragraphs: 3-4 sentences max
- Sentences: 15-25 words average
- Lists: use for 3+ items
- Tables: use for comparisons
- Callout boxes: use for key takeaways, warnings, tips

## Legal Requirements
- No investment advice: never say "achetez ceci" or "vendez cela"
- Performance disclaimer: "Les performances passées ne préjugent pas des performances futures"
- Regulatory mentions: cite AMF, FGDR where relevant
- Accuracy: all numbers must be verifiable and sourced

## CTA Guidelines
- Primary CTA: after the most persuasive section (usually after comparison/proof)
- Secondary CTA: end of article
- CTA text: specific and benefit-oriented ("Ouvrir mon PEA" not "S'inscrire")
- Never: more than 3 CTAs per article
```

### keyword-clusters.json
```json
{
  "clusters": [
    {
      "name": "PEA Core",
      "intent": "mixed (informational + transactional)",
      "keywords": [
        {"keyword": "PEA", "volume": 49500, "difficulty": 65},
        {"keyword": "ouvrir PEA", "volume": 6600, "difficulty": 45},
        {"keyword": "PEA en ligne", "volume": 3600, "difficulty": 40},
        {"keyword": "meilleur PEA", "volume": 2900, "difficulty": 50},
        {"keyword": "comparatif PEA", "volume": 1900, "difficulty": 35},
        {"keyword": "PEA avantage fiscal", "volume": 1300, "difficulty": 30},
        {"keyword": "plafond PEA", "volume": 1000, "difficulty": 25}
      ],
      "content_coverage": "partial",
      "priority": "high"
    },
    {
      "name": "ETF Investing",
      "intent": "informational",
      "keywords": [
        {"keyword": "meilleur ETF PEA", "volume": 5400, "difficulty": 45},
        {"keyword": "ETF bourse", "volume": 3200, "difficulty": 50},
        {"keyword": "investir ETF", "volume": 2700, "difficulty": 40},
        {"keyword": "ETF monde", "volume": 2200, "difficulty": 35},
        {"keyword": "DCA ETF", "volume": 1800, "difficulty": 30}
      ],
      "content_coverage": "none",
      "priority": "high"
    },
    {
      "name": "Brokerage Comparison",
      "intent": "transactional",
      "keywords": [
        {"keyword": "courtier en ligne", "volume": 4400, "difficulty": 55},
        {"keyword": "meilleur courtier bourse", "volume": 2900, "difficulty": 50},
        {"keyword": "frais courtage", "volume": 2200, "difficulty": 35},
        {"keyword": "courtier pas cher", "volume": 1600, "difficulty": 40}
      ],
      "content_coverage": "none",
      "priority": "high"
    },
    {
      "name": "Beginner Investing",
      "intent": "informational",
      "keywords": [
        {"keyword": "investir en bourse débutant", "volume": 3600, "difficulty": 40},
        {"keyword": "comment investir en bourse", "volume": 2900, "difficulty": 45},
        {"keyword": "bourse pour débutant", "volume": 2400, "difficulty": 35},
        {"keyword": "commencer investir", "volume": 1800, "difficulty": 30}
      ],
      "content_coverage": "none",
      "priority": "medium"
    },
    {
      "name": "Tax & Fiscal",
      "intent": "informational",
      "keywords": [
        {"keyword": "fiscalité bourse", "volume": 2200, "difficulty": 40},
        {"keyword": "déclaration PEA", "volume": 1300, "difficulty": 25},
        {"keyword": "flat tax bourse", "volume": 1000, "difficulty": 30},
        {"keyword": "PEA vs assurance vie", "volume": 900, "difficulty": 35}
      ],
      "content_coverage": "none",
      "priority": "medium"
    },
    {
      "name": "Stock Analysis",
      "intent": "informational",
      "keywords": [
        {"keyword": "action bourse", "volume": 6600, "difficulty": 55},
        {"keyword": "action CAC 40", "volume": 3600, "difficulty": 45},
        {"keyword": "dividende action", "volume": 2200, "difficulty": 35},
        {"keyword": "analyse action", "volume": 1300, "difficulty": 40}
      ],
      "content_coverage": "none",
      "priority": "medium"
    }
  ]
}
```
> Note: keyword-clusters.json should be populated with real DataForSEO data during setup. The volumes above are illustrative.

---

## Tools & APIs Required

| Tool | Used By | Purpose | Already Available |
|------|---------|---------|-------------------|
| **Exa API** | Agent 1, 2 | News scanning, competitor monitoring, research | Yes (`$EXA_API_KEY`) |
| **DataForSEO** | Agent 2, 6, 7 | Keyword research, SERP analysis, rankings | Yes (`$DATAFORSEO_LOGIN`) |
| **GSC module** | Agent 6, 7 | Keyword positions, CTR, trending queries | Yes (seomachine) — needs Finary GSC credentials |
| **GA4 module** | Agent 6, 7 | Traffic, conversions, declining pages | Yes (seomachine) — needs Finary GA4 credentials |
| **Playwright** | Agent 1, 2 | Reddit scraping, competitor page screenshots | Yes (trendwatching) |
| **TabStack** | Agent 2 | Extract competitor content to markdown | Yes (`$TABSTACK_API_KEY`) |
| **gws CLI** | Agent 7 | Google Sheets for reports and dashboards | Yes (authenticated) |
| **SEO skills** | Agent 4, 5 | Content quality, schema, technical SEO | Yes (13 skills) |
| **RemoteTrigger** | Scheduling | Cron-based agent scheduling | Yes (built-in) |

### Additional Setup Required
1. **Finary GSC access**: Service account credentials for finary.com's Google Search Console
2. **Finary GA4 access**: Service account credentials for Finary's GA4 property
3. **Brand context files**: Must be created with Finary team input (brand-context.md, style-guide.md, product-facts.md)
4. **Content inventory**: Initial crawl of existing Finary blog/content to build content-inventory.json
5. **Keyword clusters**: Populate keyword-clusters.json with real DataForSEO data for French investment keywords

---

## Schedule Overview

| Agent | Schedule | Trigger | Duration |
|-------|----------|---------|----------|
| **1. Intel Scout** | Every 3 days, 6am | Cron | ~15 min |
| **2. Opportunity Researcher** | After Agent 1 | Chained | ~20 min |
| **3. Content Planner** | After Agent 2 | Chained + human gate | ~10 min |
| **4. Content Writer** | After approval | Manual trigger | ~30 min per article |
| **5. SEO Validator** | After Agent 4 | Chained | ~5 min per article |
| **6. Content Refresher** | 1st of month, 8am | Cron | ~30 min |
| **7. Performance Monitor** | Weekly Monday 8am + monthly | Cron | ~10 min |

**Expected output cadence:** 2-4 new articles per week, 3-5 content refreshes per month.

---

## Programmatic SEO Opportunities

Beyond individual articles, there are high-volume programmatic SEO plays for Finary:

### A. Stock Analysis Pages (template-based)
- Template: "[Stock Name] : cours, analyse et avis [year]"
- Scale: ~200 pages (CAC 40 + SBF 120 + popular EU stocks)
- Data source: market data API for prices, P/E, dividends
- Unique angle: "Available on Finary PEA" badge for eligible stocks
- Quality gate: each page must have 40%+ unique content (analyst commentary, Finary-specific benefits)

### B. ETF Guide Pages (template-based)
- Template: "[ETF Name] : frais, performance, éligibilité PEA"
- Scale: ~100 pages (top ETFs available on Finary)
- Data source: ETF provider data (Amundi, Lyxor, iShares, Vanguard)
- Unique angle: Fee comparison vs holding same ETF at competitor brokers
- Quality gate: original commentary + fee analysis per page

### C. "[Keyword] + [City]" Pages (if Finary has physical presence)
- Only if relevant (Finary is online-only, so likely skip this)

### D. Comparison Matrix Pages
- Template: "[Product A] vs [Product B]"
- Scale: ~45 pages (9 competitors × 5 cross-comparisons)
- Content: auto-generated comparison table + unique editorial summary
- Schema: Product + ItemList

### E. Glossary Pages
- Template: "[Financial term] : définition et exemple"
- Scale: ~150 pages (investment terminology)
- Quality gate: each entry 500+ words with real examples
- Internal linking: every article links to relevant glossary entries

> Apply seo-programmatic skill quality gates: 30% uniqueness warning, 40% hard stop. Never publish thin template pages at scale.

---

## Expected Outcomes

### Month 1-2 (Foundation)
- Set up all config files and state management
- Run initial DataForSEO audit of finary.com current organic performance
- Publish 8-12 cornerstone articles (PEA guide, comparisons, beginner guides)
- Establish baseline: current organic traffic, keyword positions, non-brand traffic share
- Target: index 10+ new pages, rank for 50+ new keywords

### Month 3-4 (Expansion)
- Full agent pipeline running on schedule
- 2-4 new articles per week from intel-to-publish pipeline
- First monthly refresh cycle improves 5-10 existing articles
- Launch programmatic ETF/stock pages (50-100 pages)
- Target: 500+ organic visits/week from non-brand queries

### Month 5-6 (Scale)
- System accumulating keyword memory and performance data
- Content refresher improving existing content based on 3+ months of data
- Comparison and alternative pages ranking for competitor brand queries
- Target: 2000+ organic visits/week, 50+ page-1 keywords

### Month 7-12 (Authority)
- 100+ published articles in content inventory
- Programmatic pages covering major stocks and ETFs
- Original research/data content earning backlinks
- Target: 5000+ organic visits/week, 200+ page-1 keywords

### KPIs to Track
- Non-brand organic traffic (weekly)
- Number of page-1 keywords (weekly)
- Content published per week (weekly)
- Content refreshed per month (monthly)
- Organic conversion rate — visitors to PEA/CTO signups (monthly)
- Average keyword position across target clusters (monthly)
- Featured snippet captures (monthly)
- Domain authority trend (quarterly)

---

## Implementation Roadmap

### Phase 0: Setup (Week 1)
- [ ] Create `C:/Users/stito/finary-seo/` project structure
- [ ] Write brand-context.md, style-guide.md, product-facts.md (with Finary team input)
- [ ] Run DataForSEO audit of finary.com — baseline organic performance
- [ ] Populate keyword-clusters.json with real DataForSEO data
- [ ] Crawl existing Finary content → content-inventory.json
- [ ] Set up GSC and GA4 service account credentials
- [ ] Copy segments.json from competitor analysis output

### Phase 1: Agent Development (Week 2-3)
- [ ] Write agent definition files (Agent 1-7) as .md files in agents/orchestrator/
- [ ] Create orchestrator agent that chains Agents 1→2→3 with approval gate
- [ ] Create content pipeline that chains Agents 4→5 with validation
- [ ] Test each agent individually with sample data
- [ ] Set up RemoteTrigger schedules

### Phase 2: First Content Sprint (Week 3-4)
- [ ] Run Agent 1 (Intel Scout) — first scan
- [ ] Run Agent 2 (Researcher) — first opportunity evaluation
- [ ] Run Agent 3 (Planner) — select first 5 articles
- [ ] Run Agent 4+5 (Write + Validate) — produce first 5 articles
- [ ] Review output quality, adjust brand context and style guide
- [ ] Publish first batch

### Phase 3: Automation (Week 5-6)
- [ ] Activate scheduled triggers for Agents 1 and 7
- [ ] Monitor first automated intel cycle end-to-end
- [ ] Run first Content Refresher cycle (Agent 6)
- [ ] Adjust scoring weights based on early results

### Phase 4: Scale (Month 2+)
- [ ] Begin programmatic SEO templates (ETF pages, stock pages)
- [ ] Expand keyword clusters based on GSC data
- [ ] Build calculator/tool pages for high-engagement keywords
- [ ] Iterate on agent prompts based on content performance data
