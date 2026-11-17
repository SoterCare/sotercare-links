# CLAUDE.md: sotercare-links

## What this repo is
The link page for **links.sotercare.com**: Next.js 16, React 19, lucide-react, Vercel Analytics. **Unchanged by the B2B rebuild** (D-20). The plan is in `B2B-PLAN.md`.

## Current stage / next work
- Housekeeping only: keep one of `next.config.mjs` / `next.config.ts`, and point links at the B2B pages (Book a demo, the care-home product page) once they exist on `sotercare-website`
- The repo was renamed from `links-sotercare`. Confirm the Vercel project still deploys from `SoterCare/sotercare-links`

## Connections
| Direction | Repo | What |
|---|---|---|
| links to | `sotercare-website` | sotercare.com pages |
| none | the platform repos | standalone |

## Rules
- Pushing to `main` deploys the public site; keep changes small and check the preview

## How the SoterCare B2B platform fits together

**Plan:** `D:\SoterCare\Github-B2B\b2b-plan\` (docs 01–07; not in a repo yet).
- `D-xx` refers to its decision log and `O-x` to its open items (`04-Repos-and-Roadmap.md` §5).
- `07-Old-Repo-Mapping.md` says which old code each repo starts from.

```
sotercare-pcb-designs ── board for ──► sotercare-firmware (thigh band, ~60 per facility)
band ── facility Wi-Fi, local link (transport: open item O-5) ──► sotercare-edge-gateway (1 per facility)
staff phones ◄── local MQTT over WSS (alerts, claims, presence) ── gateway
gateway ── AWS IoT Core → SQS (alert-record, telemetry, heartbeat) ──► sotercare-backend
backend ── record_ack, shadows (roster / bands / policy / keys), cmd/claim ──► gateway
backend ── REST + Socket.IO ──► sotercare-console, sotercare-mobile
backend ── SNS → APNs / FCM push ──► sotercare-mobile
backend ── outbox → SQS events, S3 Parquet export, internal API ──► sotercare-agents
agents ── outputs / proposals / case traces (internal API) ──► backend
sotercare-contracts ── schemas + generated types ──► firmware, gateway, backend, agents, console, mobile
sotercare-infra ── provisions and deploys ──► AWS (IoT Core, ECS, RDS, Cognito, SNS, S3, Bedrock)
sotercare-studio ── records ──► sotercare-iot-dataset ── trains ──► sotercare-ml ── models ──► firmware, gateway
sotercare-website (sotercare.com), sotercare-links (links.sotercare.com): standalone
```

**Platform rules (every repo):**
- **The gateway is the only real-time safety rule engine and claim arbiter** (D-08).
  - The cloud never derives fall / SOS / moisture alerts from telemetry.
  - Agents never create or change alerts.
- **Wellness product** (D-06): no "medical", "diagnosis" or clinical-threshold wording in code, UI or docs.
- **Every cross-repo message is defined in `sotercare-contracts`.** Pin a version and run contract tests. Never keep a local copy of a schema.
- **Tenancy:** every record carries `org_id` (and `facility_id` where relevant). The backend enforces row-level security.
- **AWS `ap-south-1`** (Mumbai). **Amazon Bedrock** is the only LLM provider (D-07).
- **Build from the old repos' work** (D-20). Keep git history when moving code in. Rewrite only what the B2B design changes.

**Git and commits:**
- Trunk-based, protected `main`, PRs with CI (lint, typecheck, tests, contract tests).
- Conventional commits (`feat:`, `fix:`, `docs:`…).
- **Never add `Co-Authored-By:` or any AI-attribution trailer to commits.**
- No secrets in git. Use env files that are git-ignored locally, and AWS Secrets Manager in the cloud.

