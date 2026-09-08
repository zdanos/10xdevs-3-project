---
starter_id: 10x-astro-starter
package_manager: npm
project_name: 10x-cards
hints:
  language_family: js
  team_size: solo
  deployment_target: cloudflare-pages
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: first-class
  path_taken: standard
  quality_override: false
  self_check_answers: null
  has_auth: true
  has_payments: false
  has_realtime: false
  has_ai: true
  has_background_jobs: false
---

## Why this stack

A solo learner shipping an AI-flashcard MVP in six after-hours weeks needs
accounts, a database, and an LLM generation step without building any of them
from scratch. 10x Astro Starter is the recommended default for a JS web app and
carries all three: Supabase supplies email/password auth (FR-001, FR-002) and
Postgres, with row-level security as the enforcement point for the PRD's
"no cross-account visibility" guardrail; Astro plus Cloudflare gives a single
deploy path for the paste-and-generate flow (FR-003, FR-004). TypeScript
end-to-end with Zod at boundaries clears all four agent-friendly gates, which
matters most for a solo build leaning on an agent. Scaffolding confidence is
first-class — registered with a valid CLI, expect occasional manual steps. Two
known frictions to plan around: the edge runtime constrains long-running
generation calls, which ties into the PRD's unanswered latency and input-limit
questions, and RLS must be configured with the first table rather than after.
CI runs on GitHub Actions with auto-deploy on merge.
