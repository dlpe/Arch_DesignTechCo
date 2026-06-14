# DesignTechCo AI Creative Studio — Architecture Submission

Technical architecture for the Superside Lead Product Engineer assessment.

## Document index

| # | Document | Assessment deliverable |
|---|----------|------------------------|
| — | [05-executive-summary.md](./05-executive-summary.md) | Overview |
| 1 | [01-component-diagram.md](./01-component-diagram.md) | Component diagram |
| 2 | [02-data-flow.md](./02-data-flow.md) | Data flow: localise copy & replace images |
| 3 | [03-tech-choices.md](./03-tech-choices.md) | Technology choices & trade-offs |
| 4 | [04-roadmap.md](./04-roadmap.md) | Phased roadmap, risks & mitigations |
| 5 | [06-epics-sprints.md](./06-epics-sprints.md) | MVP epics & sprint plan (Jira / Trello) |
| — | [00-goals-figma-plugin.md](./00-goals-figma-plugin.md) | Goals → Figma plugin scope → services |

## Architecture summary

**Clients:** Figma plugin (designers), Admin UI (Creative Ops)

**Backend:** API Gateway, Brand Service, Copy Service, Image Service

**Data:** PostgreSQL, S3, Auth0

**AI:** OpenAI GPT-4o (copy, translation, vision); DALL-E 3 (image placeholders, F4)

## Requirements coverage

| Requirement | Addressed in |
|-------------|--------------|
| F1 Copy variants | [00](./00-goals-figma-plugin.md), [02](./02-data-flow.md) |
| F2 8 locales | [00](./00-goals-figma-plugin.md), [02](./02-data-flow.md) |
| F3 Brand PDF ingest | [00](./00-goals-figma-plugin.md), Brand Service |
| F4 Image placeholders | [00](./00-goals-figma-plugin.md), [02](./02-data-flow.md) |
| N1 ≤2s latency | [00](./00-goals-figma-plugin.md) (copy sync), [02](./02-data-flow.md) |
| N2 SSO/OAuth | [01](./01-component-diagram.md), [03](./03-tech-choices.md) |
| N3 5× scale | [01](./01-component-diagram.md), [04](./04-roadmap.md) |
| N4 Usage metering | [00](./00-goals-figma-plugin.md), [03](./03-tech-choices.md) |
| N5 Availability / deploy | [04](./04-roadmap.md) |
