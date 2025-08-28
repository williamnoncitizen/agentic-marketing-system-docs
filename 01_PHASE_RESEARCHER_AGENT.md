# Phase 1: Researcher Agent (Intel Gathering)

**Core Function:** To gather and structure raw intelligence from the external world. It is the system's eyes and ears, responsible for turning the unstructured web into clean, machine-readable data.

#### Core Agent Attributes (The Detective Persona)
*   **Core Identity:** You are a skeptical, evidence-obsessed market intelligence analyst. You are a detective, not a storyteller. Your sole purpose is to find and structure objective, verifiable facts.
*   **Cognitive Bias:** You are biased towards verifiable evidence. A claim without a source is just a rumor. You treat a company's self-stated claims with professional skepticism until they are validated by third-party sources.
*   **Core Directives:**
    1.  **Extract, Do Not Interpret:** Your job is to report what you find, not what you think it means. Avoid all subjective, loaded, or interpretive language.
    2.  **Verifiability is Paramount:** Every key piece of data you report must be linked to a specific source URL.
    3.  **Embrace Contradictions:** Do not try to resolve conflicts between what a company claims and what the market says. Your job is to highlight these contradictions as objective facts.
    4.  **Be Meticulous:** Precision is key. Capture specific numbers, names, and quotes.
*   **Negative Constraints:** You are not a strategist. You are not creative. You do not have opinions. You must not provide recommendations, draw conclusions, or offer any form of strategic synthesis. Your sole function is to find and report objective, verifiable facts.

**Architectural Note:** The Researcher Agent's function is broken down into multiple, specialized workflows. This prevents the dilution of quality that can occur with a single, monolithic prompt and ensures each type of intelligence is gathered with maximum depth and precision.

---

## Workflow 1: `ExecuteClientSelfScan` (Foundational)

*   **Purpose:** To validate and enrich the human-provided `Client Onboarding Document` and `ICP Definition` with objective, public-facing evidence from the client's own digital assets. This workflow takes the client's internal truth and compares it against their external presentation, creating a rich, hybrid "Ground Truth" document.
*   **Function Signature:** `ExecuteClientSelfScan(client_onboarding_path: str, icp_definition_path: str) -> str`
*   **KB Destination:** `kb/layer1_client_ground_truth/`
*   **Tools:** `Firecrawl`
*   **Output Schema (JSON):**
    ```json
    {
      "client_name": "string",
      "analysis_date": "YYYY-MM-DD",
      "source_onboarding_document": "string_path",
      "source_icp_document": "string_path",
      "internal_identity": {
        "core_purpose": "string",
        "market_void": "string",
        "ten_year_vision": "string",
        "core_values": ["string"],
        "audience_profile": {
          "core_driver": "string",
          "jobs_to_be_done": {
            "functional": "string",
            "emotional": "string"
          },
          "pains_and_stupid_taxes": ["string"]
        },
        "positioning": {
          "defined_enemy": "string",
          "contrarian_truth": "string",
          "table_stakes": ["string"],
          "proof_points": ["string"]
        },
        "personality": {
          "brand_feeling": "string",
          "personality_adjectives": [
            {
              "we_are": "string",
              "but_not": "string"
            }
          ]
        }
      },
      "ideal_customer_profile": {
        "company_characteristics": {
          "description": "string",
          "industry": ["string"],
          "company_size": "string",
          "technology_stack": ["string"]
        },
        "pain_points": ["string"],
        "buying_triggers": ["string"],
        "key_differentiators_for_icp": ["string"],
        "buyer_personas": {
          "job_titles": ["string"],
          "common_objections": ["string"]
        }
      },
      "public_facing_validation": {
        "stated_mission": "string",
        "stated_vision": "string",
        "core_product_description": "string",
        "proprietary_technologies": ["string"],
        "key_product_features": ["string"],
        "third_party_integrations": ["string"]
      },
      "synthesis": {
        "alignment_score": "string (High | Medium | Low)",
        "key_misalignments": ["string (e.g., 'Internal belief is 'simplicity', but website features are highly technical and complex.')"],
        "icp_alignment_notes": ["string (e.g., 'Website messaging strongly aligns with stated ICP pain points around cost.')"]
      }
    }
    ```

---

## Workflow 2: `ExecuteWebsiteScan` (L1 Equivalent)

*   **Purpose:** To perform a comprehensive analysis of a competitor's own website and documentation. This workflow focuses exclusively on the competitor's self-stated claims, features, and positioning.
*   **Function Signature:** `ExecuteWebsiteScan(competitor_name: str, competitor_url: str) -> str`
*   **KB Destination:** `kb/layer2A_website_intelligence/`
*   **Tools:** `Firecrawl`, `Web Search`
*   **Output Schema (JSON):**
    ```json
    {
      "competitor_name": "string",
      "analysis_date": "YYYY-MM-DD",
      "official_domain": "string_url",
      "scanned_pages": ["string_url"],
      "core_positioning_statement": "string",
      "stated_value_propositions": [
        {
          "proposition": "string",
          "source_url": "string_url"
        }
      ],
      "claimed_features": [
        {
          "feature": "string",
          "source_url": "string_url"
        }
      ],
      "performance_claims": [
        {
          "claim": "string",
          "source_url": "string_url"
        }
      ],
      "pricing_model": {
        "model_type": "string (e.g., Pay as You Go, Reserved Capacity)",
        "public_claim": "string (e.g., 'No fees for egress or API requests')",
        "starting_price_point": "string (e.g., '$6.99 per TB/month')",
        "source_url": "string_url"
      },
      "trust_signals": {
        "certifications": ["string (e.g., 'SOC-2', 'ISO 27001')"],
        "security_features": ["string (e.g., 'Object Lock', 'SSE-C')"]
      },
      "social_proof": {
        "customer_logos": ["string"],
        "case_study_count": "integer",
        "key_testimonials": ["string"]
      },
      "developer_experience": {
        "api_compatibility": ["string (e.g., '100% AWS S3 and IAM API compatible')"],
        "documentation_portal_url": "string_url",
        "partner_integrations_count": "integer"
      },
      "observed_coverage_gaps": ["string (e.g., 'Specific uptime guarantees not accessible in scraped content')"],
      "key_evidence_urls": ["string_url"]
    }
    ```
#### Key Implementation Requirements
*   **Scope & Cost Management:** The agent must be configured with a "scan depth" limit. It should start at the homepage, crawl all links found in the primary navigation elements (e.g., header, footer), but should not go more than one level deeper into the site architecture. It must be explicitly instructed to avoid crawling entire blog archives, case study libraries, or other high-volume content sections to ensure focus and manage operational costs.
*   **Dynamic Content Handling:** The agent must be able to gracefully handle websites that rely heavily on JavaScript. If a tool like Firecrawl fails to extract meaningful text content from a crucial page (e.g., a pricing or features page), the agent should not fail silently. It must log the URL of the problematic page into the `observed_coverage_gaps` array with a note, such as *"Failed to render content from URL, may require advanced browser automation."*

---

## Workflow 3: `ExecuteMarketScan` (L2 Equivalent)

*   **Purpose:** To gather third-party market intelligence to validate, contradict, or enrich the claims made on the competitor's website. This workflow focuses on external news, analyst reports, and user reviews.
*   **Function Signature:** `ExecuteMarketScan(competitor_name: str, market_keywords: list, website_scan_path: str) -> str`
*   **KB Destination:** `kb/layer2B_market_intelligence/`
*   **Tools:** `Web Search`
*   **Output Schema (JSON):**
    ```json
    {
      "competitor_name": "string",
      "analysis_date": "YYYY-MM-DD",
      "lookback_period_months": "integer",
      "context_file": "string_path_to_layer2A_file",
      "market_positioning_summary": "string",
      "third_party_validations": [
        {
          "validation": "string (e.g., 'GigaOm Radar Outperformer designation')",
          "source_url": "string_url"
        }
      ],
      "critical_assessments": [
        {
          "assessment": "string (e.g., 'Users report instances of prolonged outages')",
          "source_url": "string_url"
        }
      ],
      "adoption_signals": ["string (e.g., 'Over 143 companies using Wasabi')"],
      "competitive_comparisons": [
        {
          "competitor": "string (e.g., 'AWS S3')",
          "comparison_summary": "string"
        }
      ],
      "pricing_discussions": ["string (e.g., 'Minimum storage commitment fees can be confusing')"],
      "technical_assessments": [
        {
          "assessment": "string (e.g., 'Read performance advantage in benchmarks')",
          "source_url": "string_url"
        }
      ],
      "overall_market_sentiment": "string (e.g., 'Positive', 'Mixed', 'Negative')",
      "key_sentiment_trends": ["string"],
      "l1_vs_l2_contradictions": [
        {
          "contradiction": "string (e.g., 'Website claims performance parity, but benchmarks show write performance deficit')",
          "source_url": "string_url"
        }
      ],
      "observed_coverage_gaps": ["string (e.g., 'Limited analyst coverage from major firms')"],
      "key_source_urls": ["string_url"]
    }
    ```
#### Key Implementation Requirements
*   **Intelligent Query Generation:** The agent's master prompt must include a sub-task to generate a diverse and strategic set of search queries. Instead of only searching for the competitor's name, it must be instructed to probe for specific types of information. Example query patterns include: `"[competitor] review"`, `"[competitor] vs [client] pricing"`, `"[competitor] performance benchmark"`, `"[competitor] user complaints [forum name]"`, and `"[competitor] outage history"`. This ensures a balanced and less biased collection of market intelligence.

---

## Workflow 4: `ExecuteSocialScan` (L3 Equivalent)

*   **Purpose:** To perform a deep analysis of a competitor's social media presence (initially focusing on Twitter) to understand their communication strategy, content themes, and audience engagement.
*   **Function Signature:** `ExecuteSocialScan(competitor_name: str, social_handles: dict) -> str`
*   **KB Destination:** `kb/layer2C_social_intelligence/`
*   **Tools:** `Twitter API`
*   **Output Schema (JSON):**
    ```json
    {
      "competitor_name": "string",
      "analysis_date": "YYYY-MM-DD",
      "social_handle": "string (e.g., '@competitor_handle')",
      "cadence_summary": "string",
      "tone_and_style_summary": "string",
      "key_messaging_themes": ["string"],
      "high_level_engagement_analysis": "string",
      "competitive_mention_strategy": "string (e.g., 'Indirect competitive positioning')",
      "behavioral_contradictions": [
        {
          "contradiction": "string (e.g., 'Heavy investment in culture content despite low engagement')",
          "example_post_urls": ["string_url"]
        }
      ],
      "strategic_takeaways": [
        {
          "takeaway": "string",
          "example_post_urls": ["string_url"]
        }
      ]
    }
    ```
#### Key Implementation Requirements
*   **API Rate Limiting:** The implementation must be mindful of the Twitter API's (or any social media API's) rate limits. The code should include logic to handle these limits gracefully, such as implementing appropriate delays or backoff strategies to avoid being blocked.
*   **Data Sampling:** For high-volume accounts, the agent should not attempt to pull thousands of posts. It should be configured to retrieve a statistically relevant and recent sample size (e.g., the last 100-200 tweets) for its analysis.

---

## Validator Agent Sub-Workflow

The quality and structural integrity of the `Researcher Agent`'s outputs are evaluated by the `Validator Agent`. The specific criteria and self-correction process are defined in the master **[Validator Subsystem Architecture document](./VALIDATOR_SUBSYSTEM.md)**. The rubric used for this agent is **[RESEARCHER_AGENT_EVALUATION.json](./_EVALUATION_TEMPLATES/RESEARCHER_AGENT_EVALUATION.json)**.
