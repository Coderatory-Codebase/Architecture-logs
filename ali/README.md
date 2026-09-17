# System Architecture Documentation — base_server E-commerce Platform

Next.js frontend + Express/TypeScript backend architecture using `base_server`, documented arc42-style. Each section is its own file so it's easy to read, link to, and update independently.

## Contents

| # | Section | File |
|---|---|---|
| 1 | Introduction & Goals | [`01-introduction-and-goals.md`](./01-introduction-and-goals.md) |
| 2 | Constraints | [`02-constraints.md`](./02-constraints.md) |
| 3 | Context & Scope | [`03-context-and-scope.md`](./03-context-and-scope.md) |
| 4 | Solution Strategy | [`04-solution-strategy.md`](./04-solution-strategy.md) |
| 5 | Building Block View | [`05-building-block-view.md`](./05-building-block-view.md) |
| 6 | Runtime View | [`06-runtime-view.md`](./06-runtime-view.md) |
| 7 | Deployment View | [`07-deployment-view.md`](./07-deployment-view.md) |
| 8 | Crosscutting Concepts | [`08-crosscutting-concepts.md`](./08-crosscutting-concepts.md) |
| 9 | Architecture Decisions (index) | [`09-architecture-decisions.md`](./09-architecture-decisions.md) |
| 10 | Quality Requirements | [`10-quality-requirements.md`](./10-quality-requirements.md) |
| 11 | Risks & Technical Debt | [`11-risks-and-technical-debt.md`](./11-risks-and-technical-debt.md) |
| 12 | Glossary | [`12-glossary.md`](./12-glossary.md) |

## Architecture Decision Records

| ADR | Decision | File |
|---|---|---|
| 0001 | Database choice — PostgreSQL + Redis | [`adr/0001-database-choice.md`](./adr/0001-database-choice.md) |
| 0002 | API style — REST vs GraphQL | [`adr/0002-api-style.md`](./adr/0002-api-style.md) |
| 0003 | Search technology | [`adr/0003-search-technology.md`](./adr/0003-search-technology.md) |
