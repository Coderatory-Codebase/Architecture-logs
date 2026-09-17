# 2. Constraints

*(arc42 Section 2 — Technical and Organizational Constraints)*

## 2.1 Technical Constraints

| Constraint | Reason |
|---|---|
| Backend must build on the existing `base_server` scaffold (Express + TypeScript, layered architecture, existing auth/middleware patterns) | Given starting point — not a free technology choice for the HTTP/framework layer. |
| Frontend must be Next.js | Given, non-negotiable. |
| Payments via Stripe (primary) and PayPal (secondary) | Given target market is international; both are the industry-standard processors for card + wallet payments outside a specific local market. |
| Single currency at launch (assume USD) | Multi-currency (FX rates, rounding, display formatting per locale) is a real feature, not a default — introducing it without a stated requirement would be scope creep. Flagged here as an explicit, deliberate scope boundary, not an oversight. |

## 2.2 Organizational Constraints

| Constraint | Reason |
|---|---|
| Solo developer, no dedicated DevOps/SRE/DBA | Every operational decision (databases, caching, deployment) must be realistically operable by one person — this rules out architectures that assume a platform team. |
| Learning project, no hard external deadline | Allows iterative, section-by-section documentation (this doc itself) rather than a rushed, single-pass design. |
| "Production-scale simulation," not literal hyperscale | Scaling and caching decisions should be justified against a realistic mid-size store's growth path (thousands–low millions of monthly visits), not Amazon-scale assumptions. Designing for imaginary Amazon-scale load would itself be a violation of YAGNI. |

## 2.3 Conventions

- Architecture discussion and reasoning happens in Roman Urdu (working language between developer and this document's author).
- The document artifacts themselves (this file, ADRs, diagrams) are written in English, since that is the standard for architecture documentation that might be shared, referenced, or read by others later.
- Format: Markdown, arc42-structured, kept as docs-as-code (versionable alongside the eventual code repo).
- Individual significant decisions are recorded separately as ADRs under `/adr`, referenced from the relevant arc42 section rather than duplicated inline.
