# secdoc skills

![License](https://img.shields.io/badge/license-Apache%202.0-blue)
![Skills](https://img.shields.io/badge/skills-7-blue)
![Format](https://img.shields.io/badge/format-Agent%20Skills%20(open%20standard)-informational)

Practitioner-built skills for AI assistants — instruction sets that hold an assistant's output to the same standards I hold my own work to: framework claims with identifiers, versions verified before citing, vendor neutrality, and no invented data. I'm a working security architect; these were built for my own use, tested in real architecture and reporting work, and published here for anyone to install. Each one is an Agent Skill in the open skill format — a `SKILL.md` with YAML frontmatter plus optional reference files that load on demand — so they run in Claude and in any agent product that supports the format.

## Contents

- [Skill catalog](#skill-catalog)
  - [cybersecurity-architecture](#cybersecurity-architecture)
  - [ai-ml-engineering](#ai-ml-engineering)
  - [application-architecture](#application-architecture)
  - [code-security-analysis](#code-security-analysis)
  - [networking-architecture](#networking-architecture)
  - [network-engineering](#network-engineering)
  - [executive-reporting](#executive-reporting)
- [Repository layout](#repository-layout)
- [Installation](#installation)
- [Design principles](#design-principles)
- [Roadmap](#roadmap)
- [About the author](#about-the-author)
- [License and disclaimer](#license-and-disclaimer)
- [Contributing](#contributing)

## Skill catalog

| Skill | Purpose | Domains | Package |
|---|---|---|---|
| [cybersecurity-architecture](#cybersecurity-architecture) | Architecture-grade security work anchored to authoritative frameworks | MITRE ATT&CK/ATLAS, NIST SPs, CVSS, SOX/SOC 2/PCI DSS/GDPR/CMMC, eight practice domains from secops to governance | [`/cybersecurity-architecture.skill`](/cybersecurity-architecture.skill) |
| [ai-ml-engineering](#ai-ml-engineering) | ML, deep learning, and LLM systems with the hype stripped out | Classical ML, DL, LLM/genAI, MLOps, AI security/governance, math foundations | [`/ai-ml-engineering.skill`](/ai-ml-engineering.skill) |
| [application-architecture](#application-architecture) | Application design and language engineering with trade-offs stated | Architecture patterns, C++/Java/.NET/Python/shell, dev practices, data structures | [`/application-architecture.skill`](/application-architecture.skill) |
| [code-security-analysis](#code-security-analysis) | Hands-on security review of code and artifacts — findings that get fixed | Vulnerability classes, secrets, dependencies/supply chain, config/IaC, data handling, reporting | [`/code-security-analysis.skill`](/code-security-analysis.skill) |
| [networking-architecture](#networking-architecture) | Vendor-neutral enterprise network architecture guidance | WAN/SD-WAN, SASE/SSE, zero trust, segmentation, cloud/edge | [`/networking-architecture.skill`](/networking-architecture.skill) |
| [network-engineering](#network-engineering) | Mechanically precise networking fundamentals | Routing protocols, OSI/TCP-IP models, troubleshooting, topology design | [`/network-engineering.skill`](/network-engineering.skill) |
| [executive-reporting](#executive-reporting) | Turns messy mixed sources into traceable executive deliverables | Email briefs, Word reports, PowerPoint decks | [`/executive-reporting.skill`](/executive-reporting.skill) |

### cybersecurity-architecture

Security architecture work that a senior practitioner could defend in front of an audit committee, a CISO, or an assessor. It grounds analysis in MITRE ATT&CK and ATLAS, the NIST Special Publications (800-53, 800-37, 800-30, 800-171, 800-207, and the CSF), CVSS scoring, and the compliance regimes I work with most — SOX, SOC 2, PCI DSS, GDPR, and DoD CMMC.

Two principles govern everything the skill produces, and they're stated in the skill itself:

- **Cite specifically, never vaguely.** "NIST recommends MFA" is worthless in architecture work. Every framework claim carries an identifier — a control ID (AC-2(1)), a technique ID (T1078.004), a requirement number (PCI DSS 8.3.6), a Trust Services criterion (CC6.1), a GDPR article (Art. 32(1)), a CMMC practice tied to its 800-171 requirement. If the identifier can't be named, the skill says so and verifies before asserting.
- **Verify versions before citing.** These frameworks move — ATT&CK ships two releases a year, CVSS v4.0 coexists with v3.1 in the wild, PCI DSS v4.0.1 superseded v4.0, and CMMC program status carries dates because the rulemaking timeline shifts. When a citation carries version-sensitive weight, the skill confirms currency against the canonical source before presenting it, and flags the version when it can't.

The workflow maps to the adversary view first and controls second — compliance-first analysis produces checkbox architectures that pass audits and fail incidents — and every recommendation set ends with residual risk stated in writing.

Beyond the framework layer, the skill now carries eight practice-domain files — security operations, cloud and hybrid, endpoint, application security, data and cryptography, IAM, network and perimeter, and governance. Each follows the same shape (scope, architecture patterns, control mapping, decision criteria, assessment questions), each cross-references the framework files instead of restating them, and each draws its boundary toward its neighbors — including the sibling skills in this library — so a question routes to one owner instead of getting half-answers from three.

**Triggers when** you ask for security architecture reviews, threat modeling, control selection or mapping, risk assessment, vulnerability prioritization, compliance gap analysis, zero trust design, AI/ML system security, or audit prep — or a domain question in SOC/SIEM/detection engineering, incident response, cloud security, endpoint/EDR, secure SDLC and supply chain, DLP/encryption/key management, IAM/PAM/federation, segmentation/ZTNA, governance and risk programs, or CMMC/CUI scoping — or when you hand it an artifact like a system design, a finding to score, an SoA, or a controls matrix, even without saying "architecture."

**What it will not do:** it does not certify compliance — gap analyses inform, but only a QSA, CPA firm, C3PAO, or authorized assessor can attest, and GDPR interpretation questions route to counsel or a DPO, not to the skill. It does not resolve genuinely contestable scoping questions (PCI scope, SOX materiality, CUI boundaries) unilaterally; it flags them as assessor conversations. And it does not produce exploit code or offensive tooling — ATT&CK mapping stays on the defensive side of that line.

**Reference files:**

- `mitre-attack.md` — ATT&CK structure and how to apply it in architecture work without the common mapping pitfalls
- `mitre-atlas.md` — AI/ML threat modeling organized by system lifecycle
- `nist-sp.md` — which SP answers which question, working with 800-53 Rev. 5 and 800-30, and crosswalking
- `cvss.md` — scoring conventions (full vector strings, always), v3.1 vs v4.0, and prioritization beyond the number
- `compliance.md` — SOX, SOC 2, PCI DSS, GDPR, and CMMC, the crosswalk pattern, and boundary discipline
- `architecture-best-practices.md` — principles, reference models, review method, and the recommendation quality bar
- `domain-secops.md` — SOC design, SIEM and telemetry architecture, detection engineering, SOAR, threat intel and hunting, incident response
- `domain-cloud-hybrid.md` — shared responsibility, landing zones, IaC security, workload identity, hybrid trust boundaries, container/Kubernetes
- `domain-endpoint.md` — the layered endpoint stack, EPP/EDR/XDR, mobile and BYOD, privileged workstation patterns, OT/IoT exceptions
- `domain-appsec.md` — secure-SDLC gate placement, threat modeling as a program, supply chain, API security, secrets management
- `domain-data-crypto.md` — classification and lifecycle, encryption across states, key management, DLP, tokenization, post-quantum readiness
- `domain-iam.md` — directory and IdP design, authentication, authorization models, privileged access, identity governance, non-human identity
- `domain-network-perimeter.md` — trust boundaries and segmentation design, the policy layer above the devices, network detection, TLS inspection
- `domain-governance.md` — operating model, policy hierarchy, risk machinery, metrics, third-party risk, the compliance program as integration layer

```
cybersecurity-architecture/
├── SKILL.md
└── references/
    ├── architecture-best-practices.md
    ├── compliance.md
    ├── cvss.md
    ├── domain-appsec.md
    ├── domain-cloud-hybrid.md
    ├── domain-data-crypto.md
    ├── domain-endpoint.md
    ├── domain-governance.md
    ├── domain-iam.md
    ├── domain-network-perimeter.md
    ├── domain-secops.md
    ├── mitre-atlas.md
    ├── mitre-attack.md
    └── nist-sp.md
```

### ai-ml-engineering

Machine learning, deep learning, and LLM/generative-AI systems, plus the capability stack that makes them survive production — data and MLOps, AI security and governance, and the mathematical foundations underneath. Its opening premise, stated in the skill: the mechanics of this field are real and durable, the marketing wrapped around them is mostly noise, and the practical failure modes — leakage, evaluation theater, unmonitored drift, prompt injection treated as a curiosity — are where projects actually die.

The standards it enforces on itself:

- **Mechanics before magic.** Every capability claim traces to how the mechanism produces it. What can't be explained mechanically gets flagged as an empirical observation, not understanding — "scaling laws are measured regularities, not derived theory" is the honest register.
- **The perishability rule — the strictest in this library.** Model names, benchmark standings, tool recommendations, pricing, context-window sizes, and regulatory status are verified at use time and dated in the answer, never asserted from training memory. The durable layer — how attention works, why leakage invalidates evaluation, what drift is — is taught as durable, and the reference files are built on that split.
- **The anti-hype standard.** Capabilities and limitations get equal prominence, adoption recommendations state what a technique costs next to what it promises, and "you may not need ML for this" is a first-class answer.
- **Evaluation honesty.** No metric is reported without its failure mode — accuracy on imbalanced classes, ROC-AUC with a rare positive class, potentially contaminated LLM benchmarks all carry their caveat in the same breath. Leakage is named as the field's number-one practical failure and checked for in any workflow the skill reviews or produces.

It also carries a teaching mode — intuition first, mechanism second, math third, misconception check last — with the recurring misconceptions named: "the model understands," "more data always helps," "RAG eliminates hallucination."

**Triggers when** you mention machine learning, deep learning, neural networks, training or fine-tuning, named algorithms, LLMs, RAG, embeddings, prompt engineering, AI agents, hallucination, model evaluation, MLOps, drift, adversarial ML, AI governance, or ask "should we use AI for X" — and in teaching contexts, from ML coursework to "explain how neural networks learn."

**What it will not do alone:** ATLAS mapping, AI threat modeling, and security architecture reviews compose with `cybersecurity-architecture` — this skill supplies the ML mechanics those attacks exploit, that one supplies the framework mapping and review method. General Python and architecture questions compose with `application-architecture`; executive deliverables about AI initiatives compose with `executive-reporting`.

**Reference files:**

- `ml-fundamentals.md` — classical ML: algorithm selection, feature engineering, evaluation design, cross-validation, leakage, metric choice
- `deep-learning.md` — how networks learn, optimizers, regularization, CNN/RNN/transformer architectures, fine-tuning vs. LoRA, quantization and distillation
- `llm-genai.md` — prompting, RAG design and evaluation, agents and tool use, hallucination and mitigation, the build-choice ladder, generative evaluation
- `mlops-data.md` — pipelines, versioning, experiment tracking, deployment patterns, monitoring and drift, retraining, cost engineering, why ML projects fail
- `ai-security-governance.md` — adversarial ML mechanics, prompt injection and defenses, NIST AI RMF, regulation (verify-current), fairness, explainability, model cards
- `math-stats-foundations.md` — probability, statistics for experiments, linear algebra, gradients and the chain rule, entropy and cross-entropy

```
ai-ml-engineering/
├── SKILL.md
└── references/
    ├── ai-security-governance.md
    ├── deep-learning.md
    ├── llm-genai.md
    ├── math-stats-foundations.md
    ├── ml-fundamentals.md
    └── mlops-data.md
```

### application-architecture

Application and service design, language-level engineering in five ecosystems — C++, Java, C#/.NET, Python, and Bash/PowerShell — the development practices around the code, and data-structure selection. Eight reference files carry the depth; the SKILL.md body carries the standards.

The rules it states for itself:

- **Trade-offs are mandatory.** Every architectural recommendation states its cost and the scale at which it stops being right. A pattern proposed without its cost is marketing, not architecture — "use a circuit breaker" is incomplete; "use a circuit breaker, it adds state you must tune, and below ~3 downstream dependencies a plain timeout is simpler and sufficient" is an answer.
- **Code is idiomatic to its language.** Never Java-shaped Python or C-shaped C++. When a practice doesn't translate across ecosystems — checked exceptions, header hygiene, LINQ habits — it says so rather than porting silently.
- **Version-sensitive facts get verified at use time.** Current language standards, LTS lines, runtime support windows: searched and dated, because training memory is not a source for anything with an end-of-life date.
- **Complexity claims are honest.** Big-O first, then the constant-factor and cache-behavior caveats that make the textbook answer wrong at real sizes.
- **Code compiles and runs as written** for a named language version, with comments that explain why, never narrate what.

Like its AI/ML sibling, it carries a teaching mode — concept before syntax, one worked example before the general rule, misconception check after, with the classics named: "microservices are more scalable" as a reflex, mutable default arguments in Python, `==` versus `.equals()` in Java, unquoted variables in Bash.

**Triggers when** you ask how to structure an application, service, or API; name a design pattern; weigh microservices vs monolith; ask a language-specific question in any of the five ecosystems; request a code review; ask "which data structure"; or raise refactoring, technical debt, testing strategy, or CI/CD. REST design, CQRS, async/await, the GIL, RAII, ShellCheck, or hash tables all qualify — the word "architecture" isn't required. Teaching contexts, from coursework to certification study, trigger it too.

**What it will not do alone:** security architecture, threat modeling, and compliance mapping compose with `cybersecurity-architecture` — this skill supplies secure-coding practice, that one supplies the framework layer. Hands-on vulnerability review of a specific file, PR, or repo composes with `code-security-analysis` — this skill owns quality and design, that one owns the security finding. Network behavior questions compose with `network-engineering`; reporting deliverables about development work compose with `executive-reporting`.

**Reference files:**

- `architecture-patterns.md` — system/service/API structure, microservices vs monolith, resilience, caching, observability
- `cpp.md` — RAII, ownership, move semantics, undefined behavior, sanitizers, build reality
- `java.md` — collections, streams, concurrency, records, GC, build tooling, LTS lines
- `dotnet.md` — async/await, LINQ, nullable references, DI, EF Core, release cadence
- `python.md` — idioms, typing, packaging, GIL and concurrency, performance, traps
- `shell-scripting.md` — Bash and PowerShell safety discipline, pipeline models, and the promotion rule
- `development-practices.md` — version control, testing strategy, code review, CI/CD, dependencies, secure coding, debugging, performance
- `data-structures.md` — selection, complexity honesty, and per-language stdlib mapping

```
application-architecture/
├── SKILL.md
└── references/
    ├── architecture-patterns.md
    ├── cpp.md
    ├── data-structures.md
    ├── development-practices.md
    ├── dotnet.md
    ├── java.md
    ├── python.md
    └── shell-scripting.md
```

### code-security-analysis

Hands-on security review of real code and application artifacts — source files, repositories, diffs, dependency manifests, configuration, and IaC — producing classified, prioritized, remediation-ready findings. It runs a review the way a review actually finds bugs: not by grepping for dangerous function names, which finds the shallow 40% and misses every authorization flaw, but by tracing untrusted input to where it does damage. Two failure modes govern its standards — noise that trains developers to ignore findings, and confidence that misses the auth bug sitting in plain sight — and the skill is built to avoid both.

The standards it enforces on itself:

- **Every finding carries its fix.** A finding without a fix is homework, not help — remediation is working code in the codebase's own language and style, and it explains *why* the fix closes the flow, so the finding doubles as a teaching artifact rather than a lint hit to paste and forget.
- **False-positive discipline.** Uncertainty is stated as uncertainty — "injectable if `user_id` reaches this untrusted, trace shown" is a different finding from "injectable, here's the request" — and missing context (framework version, whether an input is trusted, deployment model) gets asked for, not silently assumed worst. A report padded with guesses loses the credibility that gets the real findings fixed.
- **Severity honesty.** No inflating an internal-tool XSS to critical because XSS sounds bad, no deflating an auth bypass to medium because the fix is expensive. Best-practice items are labeled hardening and kept in their own section, cleanly apart from exploitable findings, so the severity gradient — the report's most valuable signal — stays credible.
- **Coverage honesty.** The review states what it analyzed and what it couldn't — runtime behavior, environment config it can't see, code outside the provided scope — because a review that implies a completeness it doesn't have is worse than one that names its blind spots.
- **Defensive purpose, stated in the skill.** It explains the attack path in prose to the depth a fix needs, but does not produce weaponized exploit code, bypass tooling, or working payloads beyond the minimal illustrative case a fix must defend against. Found secrets are treated as live and reported for immediate rotation, never tested for validity — using the credential is the breach, not the check for it. Reviewing unfamiliar or third-party code for backdoors, obfuscated exfiltration, and malicious install scripts is in scope as defensive work.

The methodology is ordered on purpose — scope first, then entry points, then trust boundaries, then source-to-sink reasoning — because authorization flaws are invisible to pattern matching (the vulnerability is a *missing* check, which looks like nothing) and only surface when you read the logic and ask "where is the check that this caller may touch this object?" AI-generated code gets checked harder, not softer: plausible-looking security code is read as wrong until proven right, and every imported package is verified to exist, because hallucinated and slopsquatted dependencies are a live entry vector.

**Triggers when** you say "review this code for vulnerabilities," "is this secure" or "is this exploitable," "scan this repo for secrets," ask for a "security review" of a file, PR, or repo, ask "is this query injection-safe," want to "audit this Dockerfile/pipeline/Terraform/Kubernetes manifest," "check these dependencies" for known CVEs, or "harden this code" — and whenever you paste code with any security question attached, even a one-liner. It also fires on inspecting third-party code for backdoors, exfiltration, or malicious install scripts.

**What it will not do alone:** code quality, idioms, and design review with no security question compose with `application-architecture` — that skill owns quality and design, this one owns the security finding. AppSec program architecture, security-tool-category strategy, framework and compliance mapping, and formal CVSS scoring compose with `cybersecurity-architecture` (including its `cvss.md` reference for vector strings and its appsec/data-crypto domains for vaulting and encryption architecture). ML-model-specific analysis — poisoning, adversarial input, model supply chain — composes with `ai-ml-engineering`, while this skill still reviews the code around the model. Leadership-facing report deliverables compose with `executive-reporting`. And it does not run code or confirm exploitability at runtime, does not certify a codebase as secure (absence of found bugs in scope is not proof of security, and the coverage section says so), and does not produce offensive tooling or working exploits beyond the minimal proof a fix must defend against.

**Reference files:**

- `methodology.md` — how a review runs: depth tiers (triage, deep, diff-focused PR, each with what it can't conclude), entry-point enumeration, trust-boundary mapping before pattern matching, taint reasoning and the sanitizer-that-doesn't rule, the manual/tool split, validation before reporting, and reviewing AI-generated code as its own discipline
- `vulnerability-classes.md` — the CWE-anchored, OWASP-mapped taxonomy with per-language notes across C++, Java, .NET, Python, shell, and JS/TS: the injection family, authn/session flaws, authorization logic as a first-class section (IDOR, missing function-level checks, confused deputy), deserialization, SSRF, path traversal, XXE, races/TOCTOU, memory safety, cryptographic misuse, and unsafe reflection
- `secrets-detection.md` — secret classes and their shapes, the pattern/entropy/context detection layers, where secrets actually hide (git history included), assume-live triage, and the rotate-then-purge-then-prevent order
- `dependencies-supplychain.md` — known-CVE matching against per-ecosystem manifests and lockfiles, transitive reality, reachability honesty, patch/pin/replace guidance, typosquat and hallucinated-package checks, install-script risk, and lockfile hygiene
- `config-iac-analysis.md` — Dockerfiles, Kubernetes manifests, Terraform and cloud IaC, CI/CD pipeline definitions, and application config, each with the misconfiguration, why it's exploitable, and the corrected stanza
- `data-handling.md` — sensitive data into logs, errors, and traces; PII in URLs; analytics and crash reporters; client-side storage; over-broad serialization; and the retention patterns worth flagging for the compliance conversation
- `reporting.md` — the finding format (impact-first title, evidence and flow, CWE, severity with reasoning, exploitability, remediation code, verification step), risk-ordered report assembly, the false-positive log as a deliverable, re-review discipline, and the executive rollup

```
code-security-analysis/
├── SKILL.md
└── references/
    ├── config-iac-analysis.md
    ├── data-handling.md
    ├── dependencies-supplychain.md
    ├── methodology.md
    ├── reporting.md
    ├── secrets-detection.md
    └── vulnerability-classes.md
```

### networking-architecture

Vendor-neutral guidance on enterprise network architecture: WAN and SD-WAN, SASE and SSE, zero trust networking, segmentation and microsegmentation, cloud transit, and edge connectivity. It cites the governing standards where they exist — NIST SP 800-207 for zero trust, MEF 70/70.1 for SD-WAN service attributes, Gartner's definitions for SASE/SSE composition.

Its stance, stated in the skill: the architectural principles are durable, the vendor market is not. Gartner redraws categories and vendors consolidate roughly every 18–24 months, so the skill verifies current market details — vendor positioning, product names, pricing — with a search at use time rather than baking them in. Vendor names appear only as illustrations of a category, never endorsements, and the skill never presents a single vendor as "the answer."

For design work, it follows a fixed answer shape — context restated, qualifying gaps, recommendation with the principle behind it, trade-offs, migration sequence, standards mapping — because in real brownfield environments the migration path matters more than the end state.

**Triggers when** you ask about branch or site connectivity, MPLS vs SD-WAN vs DIA decisions, SASE or SSE adoption, ZTNA rollout or VPN replacement, segmentation, cloud transit and hybrid/multi-cloud connectivity, remote access architecture, or want a network design reviewed or recommended. Mentioning SD-WAN, SASE, SSE, SWG, CASB, ZTNA, FWaaS, MPLS, DMVPN, transit gateway, or "modernizing our network" qualifies — the word "architecture" isn't required.

**What it will not hold:** vendor endorsements, static market claims, or absolute pricing — cost figures date fast, so it gives relative economics with caveats and searches before answering anything that depends on current market state.

**Reference files:**

- `wan-fundamentals.md` — transport options, legacy WAN patterns, SD-WAN mechanics, and the migration path off legacy
- `sase-sse.md` — definitions, the SSE component stack, delivery architecture, single- vs multi-vendor, and the VPN-to-ZTNA migration
- `adjacent-architectures.md` — zero trust networking on SP 800-207, segmentation, cloud networking, edge, and how they interlock
- `decision-frameworks.md` — the qualifying questions, selection heuristics, trade-off ledger, migration sequencing, and design red flags

```
networking-architecture/
├── SKILL.md
└── references/
    ├── adjacent-architectures.md
    ├── decision-frameworks.md
    ├── sase-sse.md
    └── wan-fundamentals.md
```

### network-engineering

Networking fundamentals at the level of mechanism, not summary: routing protocol internals for OSPF, EIGRP, BGP, RIP, and IS-IS; the OSI and TCP/IP models with their associated protocols — ARP through TLS — including a layer-by-layer troubleshooting methodology; and topology and design engineering from the three-tier campus model to spine-leaf fabrics.

Its stance is the counterpart to its market-facing sibling: fundamentals age slowly. OSPF's SPF calculation and BGP's path selection don't shift with vendor announcements, so this material is durable in a way the SASE and SD-WAN market is not — with one exception the skill names itself. Standards-revision items — RFC updates, protocol deprecations, IEEE amendment status — get flagged for verification rather than asserted from memory.

The boundary with `networking-architecture` is stated in the frontmatter and holds cleanly: strategy and market questions — SD-WAN vs MPLS adoption, SASE/SSE, ZTNA rollout, cloud transit — belong to `networking-architecture`. Questions that span both, like routing design for an SD-WAN migration, compose the two skills rather than duplicating either.

**Triggers when** you ask how a routing protocol actually works, need a layer-by-layer walk through the OSI or TCP/IP models and their protocols, are troubleshooting by layer, are designing or reviewing a topology from campus three-tier to spine-leaf — and in teaching contexts: Network+ and CCNA study questions trigger it too.

**What it will not do:** it does not answer strategy and market questions — those are `networking-architecture`'s trigger space, and this skill defers to it rather than overlapping.

**Reference files:**

- `routing-protocols.md` — protocol internals for OSPF, EIGRP, BGP, RIP, and IS-IS
- `osi-model-protocols.md` — the OSI and TCP/IP models, their associated protocols, and the layer-by-layer troubleshooting method
- `topology-design.md` — topology and design engineering, three-tier campus through spine-leaf

```
network-engineering/
├── SKILL.md
└── references/
    ├── osi-model-protocols.md
    ├── routing-protocols.md
    └── topology-design.md
```

### executive-reporting

Synthesizes messy mixed sources — email threads, documents, spreadsheets, web data, meeting notes, metrics exports — into executive deliverables in three output types: an email brief, a Word report, or a PowerPoint deck. Document production itself is delegated to the platform's document skills; this skill's job is the synthesis discipline in front of it.

Two disciplines govern it, as the skill states them:

- **Ingestion inventory.** Every source gets logged before synthesis begins, so every claim in the deliverable traces back to a specific input. Nothing appears in the output that can't be pointed to in a source.
- **Answer-first synthesis.** The deliverable leads with the answer, and every number carries a "so what" — a figure without its implication is filler, not reporting.

Behind both sits the hard rule: no invented data. If a number, date, or fact isn't in the sources, it doesn't appear in the deliverable — it appears as a named gap.

**Triggers when** you hand it mixed source material and ask for an executive deliverable — a status brief, a leadership report, a board-ready deck.

**What it will not do:** it will not fabricate figures to fill gaps, and it will not produce a deliverable whose claims can't be traced to the ingested sources.

**Reference files:**

- `email-brief.md` — structure and conventions for the email deliverable
- `word-report.md` — structure and conventions for the report deliverable
- `pptx-deck.md` — structure and conventions for the deck deliverable

```
executive-reporting/
├── SKILL.md
└── references/
    ├── email-brief.md
    ├── pptx-deck.md
    └── word-report.md
```

## Repository layout

One folder per skill, each self-contained: a `SKILL.md` with YAML frontmatter, plus a `references/` directory of files the skill loads on demand. The `packages/` directory holds each skill packaged as an archive for direct install — same content as the folder, in two forms. Future skills land as new top-level folders following the same convention, with a catalog subsection added above by the same rules.

```
.
├── cybersecurity-architecture/
│   ├── SKILL.md
│   └── references/
├── ai-ml-engineering/
│   ├── SKILL.md
│   └── references/
├── application-architecture/
│   ├── SKILL.md
│   └── references/
├── code-security-analysis/
│   ├── SKILL.md
│   └── references/
├── networking-architecture/
│   ├── SKILL.md
│   └── references/
├── network-engineering/
│   ├── SKILL.md
│   └── references/
├── executive-reporting/
│   ├── SKILL.md
│   └── references/
├── packages/
│   ├── cybersecurity-architecture.skill
│   ├── ai-ml-engineering.skill
│   ├── application-architecture.skill
│   ├── code-security-analysis.skill
│   ├── networking-architecture.skill
│   ├── network-engineering.skill
│   └── executive-reporting.skill
├── LICENSE
└── README.md
```

## Installation

Skills install differently per surface. These paths are verified against Anthropic's documentation as of this writing — if one fails, check the current docs linked in each section, because install mechanics change.

### Claude (claude.ai and desktop)

Requires **Code execution and file creation** enabled in Settings, and a ZIP with the skill folder as its root — the `.skill` packages here satisfy that (rename to `.zip` if the picker insists).

1. Go to **Customize → Skills**
2. Click **+**, then **Create skill**
3. Upload the skill's ZIP (skill folder at the archive root, `SKILL.md` inside it)
4. Toggle the skill on

On Team and Enterprise plans an owner must enable Skills in Organization settings first, and can provision skills org-wide. Details: [Use skills in Claude](https://support.claude.com/en/articles/12512180-use-skills-in-claude).

### Claude Code

Filesystem-based — no upload. Clone or copy the skill folder into the skills directory and start a new session; Claude discovers it automatically and loads it when relevant.

```bash
# personal — available in all projects
git clone https://github.com/<owner>/<repo>.git
cp -r <repo>/cybersecurity-architecture ~/.claude/skills/

# project — committed with the repo, shared with the team
cp -r <repo>/cybersecurity-architecture .claude/skills/
```

Details: [Extend Claude with skills](https://code.claude.com/docs/en/skills).

### Claude API

Upload the skill to your workspace via the Skills API, then reference it in requests. A ZIP upload must contain the skill directory as its single top-level entry:

```bash
zip -r cybersecurity-architecture.zip cybersecurity-architecture/

curl https://api.anthropic.com/v1/skills \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -F "files[]=@cybersecurity-architecture.zip"
```

The Python SDK also offers a `files_from_dir` helper that takes the folder path directly. Details and current endpoint specifics: [Using Agent Skills with the API](https://platform.claude.com/docs/en/build-with-claude/skills-guide).

### Other agent products

These skills use only the open Agent Skills format — a `SKILL.md` with standard frontmatter plus reference files, no platform-specific extensions. Per Anthropic's documentation, the standard works across multiple AI tools; the format specification lives at [agentskills.io](https://agentskills.io). Any product that reads the format should load these unchanged.

## Design principles

These are the rules the skills enforce on themselves, and they're the same rules a new skill has to meet before it lands here:

**Framework claims carry identifiers.** A control ID, a technique ID, a requirement number, a CWE ID, a CVSS vector string. A claim that can't be pinned to an identifier gets verified or gets flagged — it doesn't get asserted.

**Version-sensitive facts get verified at use time.** ATT&CK releases, CVSS versions, PCI DSS revisions, CMMC program status, OWASP Top 10 and ASVS editions, language LTS lines, model names and benchmarks, vendor market positions — anything that moves is checked against the canonical source when cited, not baked into the skill to go stale.

**Vendor-neutral throughout.** Categories, representative players, and selection criteria — never a single vendor as the answer.

**Sibling skills partition, they don't overlap.** Related skills draw their trigger boundary explicitly — `network-engineering` owns protocol mechanics, `networking-architecture` owns strategy and market; `ai-ml-engineering` owns ML attack mechanics, `cybersecurity-architecture` owns the ATLAS mapping and review method; `code-security-analysis` owns the hands-on vulnerability finding in a specific file or repo, `application-architecture` owns code quality and design, and `cybersecurity-architecture` owns the AppSec program and framework layer above both — and a question spanning boundaries composes the skills rather than getting a half-answer from either. The cybersecurity-architecture domain files carry these boundary sentences internally too: its network-perimeter file owns security policy and zone architecture while device engineering stays with `network-engineering` and SASE strategy with `networking-architecture`, and its appsec file owns gate placement while coding practice stays with `application-architecture` and per-artifact review stays with `code-security-analysis`.

**Reference files load on demand.** The core `SKILL.md` stays lean; deep material sits in `references/` and loads only when the task calls for it. That's what the format is for.

**Document production delegates.** When a deliverable is a Word file, a deck, or a PDF, the skill hands formatting to the platform's document skills and keeps its own scope to the analysis and synthesis discipline.

**No AI-pattern language in output.** Every deliverable these skills generate is held to a language standard that strips the filler patterns AI text is known for. The output should read like a practitioner wrote it, because the instructions did.

## Roadmap

More skills will land here as I build and test them against the same bar — I don't publish what I haven't used.

## About the author

I'm a senior cybersecurity architect with 25+ years in the field, author of the *Cybersecurity Architect's Handbook*, and a long-time adjunct instructor in cybersecurity, networking, and digital forensics. I write at [secdoc.tech](https://secdoc.tech).

## License and disclaimer

This repository is licensed under the [Apache License 2.0](LICENSE).

These skills reference security frameworks, standards, and compliance regimes, but their output is not legal, audit, or compliance advice, and it is not an attestation of anything. Gap analyses and mappings produced with these skills inform your work; only a qualified assessor can certify it, and GDPR questions with legal weight belong with counsel or a DPO.

MITRE ATT&CK® and MITRE ATLAS are the work of The MITRE Corporation, with ATT&CK's registered mark acknowledged. NIST publications are the work of the National Institute of Standards and Technology. CVSS is maintained by FIRST. The CWE list is maintained by The MITRE Corporation; the OWASP Top 10 and ASVS by the OWASP Foundation. PCI DSS belongs to the PCI Security Standards Council; SOC 2 to the AICPA; CMMC to the U.S. Department of Defense; the GDPR to the European Union. All cited frameworks and standards belong to their respective owners; this repository maps to them, it doesn't reproduce or replace them.

## Contributing

This is a personal library published in the open, so treat it that way: issues and suggestions are welcome, and PRs get considered. The contribution bar is the design principles above — identifiers on framework claims, versions verified not assumed, vendor neutrality, lean core with on-demand references. If a change meets that bar, I want to see it.
