# secdoc skills

![License](https://img.shields.io/badge/license-Apache%202.0-blue)
![Skills](https://img.shields.io/badge/skills-4-blue)
![Format](https://img.shields.io/badge/format-Agent%20Skills%20(open%20standard)-informational)

Practitioner-built skills for AI assistants — instruction sets that hold an assistant's output to the same standards I hold my own work to: framework claims with identifiers, versions verified before citing, vendor neutrality, and no invented data. I'm a working security architect; these were built for my own use, tested in real architecture and reporting work, and published here for anyone to install. Each one is an Agent Skill in the open skill format — a `SKILL.md` with YAML frontmatter plus optional reference files that load on demand — so they run in Claude and in any agent product that supports the format.

## Contents

- [Skill catalog](#skill-catalog)
  - [cybersecurity-architecture](#cybersecurity-architecture)
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
| [cybersecurity-architecture](#cybersecurity-architecture) | Architecture-grade security work anchored to authoritative frameworks | MITRE ATT&CK/ATLAS, NIST SPs, CVSS, SOX/SOC 2/PCI DSS | [`packages/cybersecurity-architecture.skill`](packages/cybersecurity-architecture.skill) |
| [networking-architecture](#networking-architecture) | Vendor-neutral enterprise network architecture guidance | WAN/SD-WAN, SASE/SSE, zero trust, segmentation, cloud/edge | [`packages/networking-architecture.skill`](packages/networking-architecture.skill) |
| [network-engineering](#network-engineering) | Mechanically precise networking fundamentals | Routing protocols, OSI/TCP-IP models, troubleshooting, topology design | [`packages/network-engineering.skill`](packages/network-engineering.skill) |
| [executive-reporting](#executive-reporting) | Turns messy mixed sources into traceable executive deliverables | Email briefs, Word reports, PowerPoint decks | [`packages/executive-reporting.skill`](packages/executive-reporting.skill) |

### cybersecurity-architecture

Security architecture work that a senior practitioner could defend in front of an audit committee, a CISO, or an assessor. It grounds analysis in MITRE ATT&CK and ATLAS, the NIST Special Publications (800-53, 800-37, 800-30, 800-171, 800-207, and the CSF), CVSS scoring, and the compliance regimes I work with most — SOX, SOC 2, and PCI DSS.

Two principles govern everything the skill produces, and they're stated in the skill itself:

- **Cite specifically, never vaguely.** "NIST recommends MFA" is worthless in architecture work. Every framework claim carries an identifier — a control ID (AC-2(1)), a technique ID (T1078.004), a requirement number (PCI DSS 8.3.6), a Trust Services criterion (CC6.1). If the identifier can't be named, the skill says so and verifies before asserting.
- **Verify versions before citing.** These frameworks move — ATT&CK ships two releases a year, CVSS v4.0 coexists with v3.1 in the wild, PCI DSS v4.0.1 superseded v4.0. When a citation carries version-sensitive weight, the skill confirms currency against the canonical source before presenting it, and flags the version when it can't.

The workflow maps to the adversary view first and controls second — compliance-first analysis produces checkbox architectures that pass audits and fail incidents — and every recommendation set ends with residual risk stated in writing.

**Triggers when** you ask for security architecture reviews, threat modeling, control selection or mapping, risk assessment, vulnerability prioritization, compliance gap analysis, zero trust design, AI/ML system security, or audit prep — or when you hand it an artifact like a system design, a finding to score, an SoA, or a controls matrix, even without saying "architecture."

**What it will not do:** it does not certify compliance — gap analyses inform, but only a QSA, CPA firm, or authorized assessor can attest. It does not resolve genuinely contestable scoping questions (PCI scope, SOX materiality) unilaterally; it flags them as assessor conversations. And it does not produce exploit code or offensive tooling — ATT&CK mapping stays on the defensive side of that line.

**Reference files:**

- `mitre-attack.md` — ATT&CK structure and how to apply it in architecture work without the common mapping pitfalls
- `mitre-atlas.md` — AI/ML threat modeling organized by system lifecycle
- `nist-sp.md` — which SP answers which question, working with 800-53 Rev. 5 and 800-30, and crosswalking
- `cvss.md` — scoring conventions (full vector strings, always), v3.1 vs v4.0, and prioritization beyond the number
- `compliance.md` — SOX, SOC 2, and PCI DSS, the crosswalk pattern, and boundary discipline
- `architecture-best-practices.md` — principles, reference models, review method, and the recommendation quality bar

```
cybersecurity-architecture/
├── SKILL.md
└── references/
    ├── architecture-best-practices.md
    ├── compliance.md
    ├── cvss.md
    ├── mitre-atlas.md
    ├── mitre-attack.md
    └── nist-sp.md
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

**Framework claims carry identifiers.** A control ID, a technique ID, a requirement number, a vector string. A claim that can't be pinned to an identifier gets verified or gets flagged — it doesn't get asserted.

**Version-sensitive facts get verified at use time.** ATT&CK releases, CVSS versions, PCI DSS revisions, vendor market positions — anything that moves is checked against the canonical source when cited, not baked into the skill to go stale.

**Vendor-neutral throughout.** Categories, representative players, and selection criteria — never a single vendor as the answer.

**Sibling skills partition, they don't overlap.** Related skills draw their trigger boundary explicitly — `network-engineering` owns protocol mechanics, `networking-architecture` owns strategy and market — and a question spanning both composes the two rather than getting a half-answer from either.

**Reference files load on demand.** The core `SKILL.md` stays lean; deep material sits in `references/` and loads only when the task calls for it. That's what the format is for.

**Document production delegates.** When a deliverable is a Word file, a deck, or a PDF, the skill hands formatting to the platform's document skills and keeps its own scope to the analysis and synthesis discipline.

**No AI-pattern language in output.** Every deliverable these skills generate is held to a language standard that strips the filler patterns AI text is known for. The output should read like a practitioner wrote it, because the instructions did.

## Roadmap

More skills will land here as I build and test them against the same bar — I don't publish what I haven't used.

## About the author

I'm a senior cybersecurity architect with 25+ years in the field, author of the *Cybersecurity Architect's Handbook*, and a long-time adjunct instructor in cybersecurity, networking, and digital forensics. I write at [secdoc.tech](https://secdoc.tech).

## License and disclaimer

This repository is licensed under the [Apache License 2.0](LICENSE).

These skills reference security frameworks, standards, and compliance regimes, but their output is not legal, audit, or compliance advice, and it is not an attestation of anything. Gap analyses and mappings produced with these skills inform your work; only a qualified assessor can certify it.

MITRE ATT&CK® and MITRE ATLAS are the work of The MITRE Corporation, with ATT&CK's registered mark acknowledged. NIST publications are the work of the National Institute of Standards and Technology. CVSS is maintained by FIRST. PCI DSS belongs to the PCI Security Standards Council; SOC 2 to the AICPA. All cited frameworks and standards belong to their respective owners; this repository maps to them, it doesn't reproduce or replace them.

## Contributing

This is a personal library published in the open, so treat it that way: issues and suggestions are welcome, and PRs get considered. The contribution bar is the design principles above — identifiers on framework claims, versions verified not assumed, vendor neutrality, lean core with on-demand references. If a change meets that bar, I want to see it.
