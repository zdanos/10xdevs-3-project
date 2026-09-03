---
project: "10xCards"
context_type: greenfield
created: 2026-09-03
updated: 2026-09-03
checkpoint:
  current_phase: 8
  phases_completed: [1, 2, 3, 4, 5, 6, 7]
  gray_areas_resolved:
    - topic: "primary persona"
      decision: "self-directed adult learner (technical/professional material from docs, courses, articles)"
    - topic: "pain category"
      decision: "compound: workflow friction + missing capability + decision paralysis"
    - topic: "insight vs status quo"
      decision: "existing tools bury generation behind heavy UX; paste-and-go is the product"
    - topic: "trigger moment"
      decision: "just finished reading something worth retaining; pastes while tab is still open"
    - topic: "auth strategy"
      decision: "email + password; server-side accounts"
    - topic: "role model"
      decision: "flat - single role, no admin"
    - topic: "card visibility"
      decision: "strictly private to owner; no cross-account visibility"
    - topic: "MVP scope vs timeline"
      decision: "kept full 8-step flow; committed to longer timeline (6 weeks, after-hours) with cost acknowledged"
    - topic: "collection organization"
      decision: "cards grouped into named decks; create/rename/delete in MVP; every card belongs to a deck from creation; no move-between-decks"
    - topic: "study scope"
      decision: "learner picks one deck to study"
    - topic: "generation failure"
      decision: "atomic - error with retry, nothing saved"
    - topic: "domain rule shape"
      decision: "extraction + classification: decide which claims are card-worthy, emit one atomic Q/A pair each"
    - topic: "card quality criteria"
      decision: "atomic; answerable standalone; faithful to source; non-duplicative within a batch"
    - topic: "product type"
      decision: "web app"
    - topic: "target scale"
      decision: "dozens to a hundred learners"
    - topic: "timeline"
      decision: "6 weeks, after-hours, no hard deadline"
  frs_drafted: 14
  quality_check_status: accepted
---

# Shape notes — 10xCards

## Seed idea (verbatim, as provided)

> ## 10xCards - MVP
>
> ### Główny problem
> Manualne tworzenie wysokiej jakości fiszek edukacyjnych jest czasochłonne, co zniechęca do
> korzystania z efektywnej metody nauki jaką jest spaced repetition.
>
> ### Najmniejszy zestaw funkcjonalności
> - Generowanie fiszek przez AI na podstawie wprowadzonego tekstu (kopiuj-wklej)
> - Manualne tworzenie fiszek
> - Przeglądanie, edycja i usuwanie fiszek
> - Prosty system kont użytkowników do przechowywania fiszek
> - Integracja fiszek z gotowym algorytmem powtórek
>
> ### Co NIE wchodzi w zakres MVP
> - Własny, zaawansowany algorytm powtórek (jak SuperMemo, Anki)
> - Import wielu formatów (PDF, DOCX, itp.)
> - Współdzielenie zestawów fiszek między użytkownikami
> - Integracje z innymi platformami edukacyjnymi
> - Aplikacje mobilne (na początek tylko web)
>
> ### Kryteria sukcesu
> - 75% fiszek wygenerowanych przez AI jest akceptowane przez użytkownika
> - Użytkownicy tworzą 75% fiszek z wykorzystaniem AI

## Vision & Problem Statement

Creating high-quality flashcards by hand is time-consuming. The cost is high enough that
people who know spaced repetition works abandon the method rather than pay it. The pain is
compound: it is workflow friction (the typing and formatting is tedious), a missing
capability (splitting material into atomic question/answer pairs is a skill most learners
don't have), and decision paralysis (facing a long text, the learner can't decide what is
worth a card at all).

The insight: existing tools bury card generation behind heavy UX. Anki, Quizlet and RemNote
all exist and some already ship AI generation, but it sits behind plugins, subscriptions, or
complex setup. A stripped-down paste-and-go flow is the whole product.

## User & Persona

Primary persona: a **self-directed adult learner** — someone learning technical or
professional material on their own from documentation, courses, and articles. The input text
is prose they already have in front of them.

The moment: they have just finished reading something worth retaining. They know re-reading
won't make it stick, and they paste it while the tab is still open.

## Access Control

Multi-user with server-side accounts. Registration and sign-in by **email + password**.

Flat user model — there is exactly one role. Every account is identical in capability; there
is no admin, moderator, or guest role in the MVP.

Cards are strictly private to their owner. There is no cross-account visibility of any kind,
by any user. This is the access-control expression of the "no shared decks" non-goal.

Unauthenticated visitors have no access to cards; card storage requires an account.

> Not decided in this session: account deletion / data export behaviour. Routed to Open Questions.

## MVP first flow (locked)

1. Learner registers, then signs in
2. Pastes a block of text they just finished reading
3. App generates candidate cards from it
4. Learner reviews each candidate: accept / edit / reject
5. Accepted cards land in their collection
6. Learner opens a study session
7. App serves the cards due now, per the repetition algorithm
8. Learner rates each recall; the algorithm reschedules

Manual card creation and full edit/delete on the collection sit alongside this flow.

### Timeline acknowledgment

Estimated MVP: 6 weeks, after-hours.

Acknowledged on 2026-09-03: 6-week MVP requires sustained dedication; user accepted.
The scope cost was surfaced explicitly (8 user actions before value, 4 subsystems, 2 external
dependencies) along with concrete scope-down moves; the user chose the longer timeline
eyes-open rather than cutting the study loop, manual creation, or accounts.

## Success Criteria

### Primary
- 75% of AI-generated cards are accepted by the user.
- Users create 75% of their cards using AI (rather than manually).

### Secondary
- Learners come back for a second session: a meaningful share of users who generate cards
  once return within a week to study or generate more.

### Guardrails
- No card is saved without explicit user approval. The review gate is load-bearing; auto-saving
  generated cards would destroy the premise of user control and make the acceptance metric
  meaningless.
- Pasted source text is not retained or reused. Text submitted for generation leaves no trace
  in operator-accessible storage once the request that consumed it completes.
- A user never sees another user's cards. Any cross-account leak is a regression regardless of
  other outcomes.
- Existing cards survive — no silent data loss. Cards a learner accepted are never lost or
  altered without their action.

## Functional Requirements

### Accounts
- FR-001: Learner can register an account with email and password. Priority: must-have
  > Socrates: Counter-arguments considered (accounts delay the metric-bearing work; passwords
  > are the costliest auth to own; sign-up before value is a conversion wall). Resolution:
  > stands as written — cross-device persistence of cards requires accounts and the seed names
  > them explicitly.
- FR-002: Learner can sign in and sign out. Priority: must-have
  > Socrates: Counter-arguments considered (sign-out is dead weight on a personal device;
  > session handling is the hidden cost). Resolution: stands as written — it is the unavoidable
  > other half of FR-001.

### AI generation
- FR-003: Learner can paste a block of text, within an explicit length limit, and request card
  generation from it. Priority: must-have
  > Socrates: Counter-argument accepted: "unbounded text hides a cliff — cost, latency, and card
  > quality break differently between a paragraph and a 40-page chapter." Resolution: FR revised
  > to carry an explicit input length limit, enforced and communicated to the learner before they
  > submit. The specific limit is an open question.
- FR-004: Learner can review generated candidates and accept, edit, or reject each one, and can
  accept all candidates at once. Priority: must-have
  > Socrates: Counter-argument accepted: "per-card review reintroduces the work you're removing —
  > judging 30 candidates replaces writing with reviewing." Resolution: FR revised to add bulk
  > accept alongside per-card review; editing remains available afterward. Noted cost: bulk accept
  > weakens the 75%-acceptance metric, since a bulk-accepted batch is not a per-card judgement.
  > Routed to Open Questions.
- FR-005: Learner can save accepted candidates into a deck in their collection. Priority: must-have
  > Socrates: Counter-arguments considered (forcing a deck choice at save time is friction; saving
  > is a consequence of accepting, not a capability). Resolution: stands as written — explicit save
  > into a chosen deck is what makes the review gate real and the organization deliberate.

### Collection management
- FR-006: Learner can create a card manually. Priority: nice-to-have
  > Socrates: Counter-argument accepted: "highest-cost, lowest-signal FR in the list — a full manual
  > authoring UI tests none of the hypotheses." Resolution: demoted to nice-to-have and removed from
  > MVP scope; recorded in Non-Goals. This departs from the seed's stated minimum feature set,
  > deliberately.
- FR-007: Learner can browse their collection of cards. Priority: must-have
  > Socrates: Counter-arguments considered ("browse" understates search/filter/pagination at volume;
  > nobody browses a flashcard collection). Resolution: stands as written — the seed names review of
  > cards, and edit/delete need a surface to act on.
- FR-008: Learner can edit an existing card; editing preserves the card's existing schedule.
  Priority: must-have
  > Socrates: Counter-argument accepted: "editing a card in mid-schedule corrupts its history — the
  > scheduling state describes a card that no longer exists." Resolution: FR revised — edits preserve
  > the schedule, treating an edit as a correction to the same card. Accepts that a heavy rewrite
  > keeps a schedule it did not earn.
- FR-009: Learner can delete a card. Priority: must-have
  > Socrates: Counter-arguments considered (hard delete sits in tension with the no-data-loss
  > guardrail; deletion papers over a weak review gate). Resolution: stands as written — learners
  > change what they study, and an append-only collection becomes unusable.

### Decks
- FR-012: Learner can create a named deck. Priority: must-have
  > Socrates: Counter-arguments considered (decks were added in this session, not the seed; naming a
  > deck interrupts paste-and-go; auto-grouping by paste would have been free). Resolution: stands as
  > written — a multi-subject learner needs deliberate grouping, and deck-scoped study depends on it.
- FR-014: Learner can rename a deck. Priority: must-have
- FR-015: Learner can delete a deck. Priority: must-have
  > Socrates (covering FR-014 and FR-015, formerly one FR): Counter-argument accepted: "rename and
  > delete are unrelated operations bundled together — renaming is cosmetic and safe, deleting is
  > destructive and needs a rule." Resolution: split into two FRs. FR-015 must state what happens to
  > the cards inside a deleted deck; that rule is an open question and is the MVP's most dangerous
  > operation against the no-silent-data-loss guardrail.

> Dropped: FR-013 (assign cards to a chosen deck).
> Socrates: Counter-argument accepted: "redundant with FR-005 — a separate assignment capability only
> matters for cards that exist outside a deck." Resolution: dropped. Every card belongs to a deck from
> creation; no deckless state exists.

### Study
- FR-010: Learner can start a study session for a chosen deck, which serves the cards due now plus a
  capped batch of cards not yet seen. Priority: must-have
  > Socrates: Counter-argument accepted: "'due now' is empty on day one — a learner who just generated
  > their first deck has nothing due." Resolution: FR revised — a session mixes due cards with a
  > bounded number of never-seen cards. The cap is an open question.
- FR-011: Learner can rate their recall of a card so that it is rescheduled. Priority: must-have
  > Socrates: Counter-arguments considered (self-rating is the weakest link in spaced repetition; the
  > rating scale is an unmade decision hiding in the FR). Resolution: stands as written — it is the
  > input the repetition algorithm requires; without it there is no scheduling at all.

## User Stories

### US-01: Learner turns a text they just read into a reviewed deck

- **Given** a signed-in learner with a block of text they have just finished reading
- **When** they paste it and request card generation
- **Then** they are shown candidate cards one at a time and can accept, edit, or reject each
  one, and only the ones they accept are saved into a deck they choose

#### Acceptance Criteria
- No candidate card is persisted before the learner accepts it, whether accepted one at a time
  or in bulk.
- Text exceeding the input length limit is rejected before submission, with the limit made
  visible to the learner.
- An edited candidate is saved as the learner edited it, not as generated.
- Rejected candidates are discarded and are not saved anywhere.
- If generation fails or returns nothing usable, the learner sees an error and can retry;
  nothing is saved (generation is atomic — no partial state).
- The pasted source text is not retained after the generation request that consumed it completes.

## Business Logic

Given a block of text, the product decides which claims in it are worth learning and turns each
one into a single atomic question/answer pair.

The rule consumes one input the learner supplies: a block of prose they have just read, bounded
by a stated length limit. Its output is a set of candidate cards, each carrying exactly one
question and its answer.

Four criteria define a good candidate, and the acceptance target is a bet on all four holding:
each card is **atomic** (one idea, never compound); **answerable from the card alone** (the
question carries enough context to be answered without the source text at hand); **faithful to
the source** (every answer is supported by the pasted text — no outside knowledge, no
fabrication); and **non-duplicative within a batch** (no two candidates from one paste test the
same fact in different words).

The learner encounters the rule immediately after pasting: the candidates are presented for
accept, edit, or reject, and only what they approve is kept. The rule never persists anything on
its own.

## Non-Functional Requirements

- Source text submitted for card generation leaves no trace in operator-accessible storage after
  the request that consumed it completes.
- The product remains usable on the latest two major versions of the mainstream desktop browsers.

## Product framing

- Product type: web app (browser-based).
- Target scale: dozens to a hundred learners.
- Timeline: 6 weeks, after-hours (evenings and weekends). No hard deadline.

Scale insight (Socratic probe at 100x): generation cost would force a per-learner cap. At ten
thousand learners the per-paste cost dominates, and the rule would need a quota or a cheaper
path — which changes what "paste and go" means. Not an MVP concern at a hundred users, but it
is the first thing that breaks on growth.

## Non-Goals

Functional non-goals:

- **Manual card creation.** Demoted during the Socratic round as the highest-cost, lowest-signal
  capability; it tests none of the success metrics. Departs from the seed's stated minimum
  feature set, deliberately.
- **Building our own repetition algorithm.** The MVP integrates a ready-made algorithm and never
  writes scheduling logic. (From the seed.)
- **Moving cards between decks.** A card's deck is chosen at creation and stays there.
- **Search, filter, or pagination over the collection.** The MVP assumes collections small enough
  to browse as a list, and says so rather than discovering it later.
- **Importing other formats (PDF, DOCX, etc.).** Paste is the only input. (From the seed.)
- **Sharing decks between users.** (From the seed; also expressed as an access-control rule.)
- **Integrations with other learning platforms.** (From the seed.)
- **Mobile applications.** Web only. (From the seed.)

Non-functional non-goals:

- **Offline use.** Generation needs a network; the MVP makes no offline guarantee, including for
  studying.
- **Full WCAG accessibility conformance.** Sensible baseline markup only; no conformance target
  or audit in the MVP.
- **Uptime or latency guarantees.** No SLA.
- **Localization beyond a single interface language.** One UI language, even though pasted source
  text may be in any language.

## Open Questions

1. **What is the input length limit for a paste?** — Owner: user / downstream. Needed to make
   FR-003 enforceable.
2. **What happens to the cards inside a deleted deck?** — Owner: user. Blocks FR-015; this is the
   MVP's most dangerous operation against the "no silent data loss" guardrail.
3. **How many not-yet-seen cards enter a single study session?** — Owner: downstream; likely
   dictated by the chosen repetition algorithm. Needed for FR-010.
4. **Does bulk accept undermine the 75%-acceptance metric?** — Owner: user. A bulk-accepted batch
   is not a per-card judgement, so the primary success metric may measure something weaker than
   intended. Decide whether the metric counts only per-card decisions.
5. **Account deletion and data export behaviour** — Owner: user. Not decided in this session;
   likely a baseline expectation and a GDPR-shaped requirement for EU learners.
6. **What is the acceptable generation wait time?** — Owner: user. No latency NFR was captured, so
   the product currently makes no commitment about how long a learner waits after pasting.

## Quality cross-check

Run 2026-09-03. All six greenfield elements present; status: accepted.

- Access Control: present.
- Business Logic (one-sentence rule): present.
- Project artifacts: present.
- Timeline-cost acknowledged: present (6 weeks, accepted 2026-09-03).
- Non-Goals: present (8 functional, 4 non-functional).
- Preserved behavior: n/a (greenfield).

Two weak spots noted, not gate failures — both already carried in Open Questions:

- Bulk accept (FR-004) may hollow out the primary success metric: a bulk-accepted batch is not a
  per-card judgement, so "75% of AI-generated cards accepted" could measure impatience rather
  than card quality. See Open Question 4.
- No generation-latency NFR was captured. The product makes no commitment about how long a
  learner waits after pasting — the single moment where the paste-and-go premise either holds or
  does not. See Open Question 6.
