# Contribution #2: Adapter: Google Vertex AI Model Provider

**Contribution Number:** 2
**Student:** Harshavardan Yuvaraj
**Issue:** https://github.com/orthogonalhq/nous-core/issues/305
**Status:** Phase II (Solution Approach) — Done. Investigation redirected the issue, now an architectural contribution via #421.

---

## Why I Chose This Issue

This is my second contribution to orthogonalhq/nous-core, following the merged Groq provider leaf (PR #404). I asked the maintainer (@atlamors) directly on the issue and was assigned it on 2026-06-30.

Groq was mostly a metadata exercise (static API key, one endpoint). I wanted to step up to a harder problem for contribution #2, and Vertex AI delivers: real OAuth2/service-account auth with token refresh, and a templated per-project/region endpoint — neither of which fits the existing leaf contract cleanly. It's also directly relevant to my interest in cloud/GCP infra, and pushes me to read runtime code (not just definitions) to understand where the contract's assumptions break down.

---

## Understanding the Issue

### Problem Description

Nous has no Google Vertex AI model provider. There's no `self/subcortex/providers/src/providers/vertex/` leaf, so users can't route requests to Gemini models via Vertex AI — nothing missing is "broken," it's a vendor gap in the provider catalog.

### Expected Behavior

A certified provider leaf for Vertex (`definition.ts`, `provider.ts`, `adapter.ts`, `index.ts`, likely + `implementation.ts`) that: registers `vertex` as a known vendor, resolves credentials/endpoint correctly for Vertex's auth model, and lets a configured Vertex provider entry actually send/receive chat completions — appearing correctly in the generated catalogs and passing the roster tests other leaves must satisfy.

### Current Behavior

No `vertex` vendor exists anywhere in the codebase (confirmed via grep — no `providers/vertex/` dir, no `vertex` in `provider-definitions.ts`/`KNOWN_PROVIDER_VENDORS`). If someone hand-configured a provider entry with `vendor: 'vertex'` today, `resolveProviderFactory` would find nothing, `createProvider` would fall back to a plain `ChatCompletionsProvider`, and it would fail: that class sends a static `Authorization: Bearer <key>` header and a single static endpoint, but Vertex needs a refreshing OAuth2 token and a project/region-templated URL — so the fallback path is silently wrong, not just absent.

### Affected Components

- New leaf: `self/subcortex/providers/src/providers/vertex/` (definition, provider factory, adapter, index, probably a custom `implementation.ts` for auth/URL logic)
- Generated catalogs (regenerated, not hand-edited): `provider-adapters.ts`, `provider-definitions.ts`, `provider-factories.ts`
- Roster tests that enumerate all vendors: `provider-codegen.test.ts`, `provider-definitions/provider-definition-types.test.ts`, `provider-definitions/provider-definitions.test.ts`, `provider-pipeline-integration.test.ts`, `adapter-resolver.test.ts`
- Possibly `schemas/provider-definition.ts` (the `auth`/`defaultEndpoint` contract), pending the maintainer's answer on whether to extend it or keep auth/URL logic inside a custom provider class
- Possibly `package.json` (new dependency: `google-auth-library`, not currently in the monorepo)

---

## Reproduction Process

### Environment Setup

Reused the fork/remote setup from the Groq contribution (`origin` = my fork, `upstream` = official repo). Created `feat/google-vertex-ai-model-provider` off the latest `upstream/feat/contributor-friendly-inference-provider-surface` tip — no new environment issues since the Groq round already resolved local build quirks.

### Steps to Reproduce

This is a missing-feature gap, not a runtime bug, so "reproducing" it means confirming the gap and confirming *why* the obvious workaround fails:

1. Search `self/subcortex/providers/src/providers/` for a `vertex` (or `google-vertex`) directory — none exists.
2. Search `provider-definitions.ts` / `KNOWN_PROVIDER_VENDORS` for `vertex` — absent, so the runtime has no registered vendor to resolve.
3. Trace what happens if a user manually configures a provider entry with `vendor: 'vertex'` anyway: `resolveProviderFactory('vertex')` returns nothing, so `ProviderRegistry.createProvider` falls back to `new ChatCompletionsProvider(...)`. That class authenticates with a single static Bearer token and a fixed endpoint — neither matches Vertex's refreshing OAuth2 token or project/region-templated URL, so the fallback would fail at request time, not just be "missing."

### Reproduction Evidence

- **Commit showing reproduction:** N/A — no code changes yet; this is a static-analysis gap, confirmed via `grep`/read-through of `provider-definitions.ts`, `provider-runtime.ts`, and `schemas/provider-definition.ts`, not a runnable repro.
- **My findings:**  Key finding: Vertex's OpenAI-compatible `/chat/completions` endpoint is OAuth-only (no static API key), while the only static-key path (Express Mode) only works against the native `generateContent` endpoint — so auth and API surface are coupled, and no combination fits the existing thin-leaf (metadata-only) pattern used by Groq. Posted two clarifying questions to @atlamors on #305 (which path to build, and contract-vs-custom-code) and am waiting on a reply before deciding the solution approach.

---

## Solution Approach

### Analysis

The "root cause" here turned out to be a mis-scoped issue, not a code bug. Digging into Vertex's auth model against the leaf contract surfaced two hard mismatches:

1. **Auth and API surface are coupled.** Vertex's OpenAI-compatible `/chat/completions` endpoint accepts *only* a short-lived OAuth2 / service-account bearer token (~1h, needs refresh). Static API keys (Express Mode) work *only* on the native `:generateContent` endpoint. There is no static-key + OpenAI-compatible combination — so the thin metadata-only leaf that worked for Groq is impossible for Vertex.
2. **The endpoint is project/region-templated**, not a single static `defaultEndpoint`; the runtime overwrites `config.endpoint` with the leaf constant, which can't hold per-user project/region values.

The deeper root cause: cloud platform providers (Vertex, Bedrock, Azure AI Foundry) are categorically different from BYOK static-key providers. They carry account/project identity, region-aware endpoints, server-side credential control, quota, and billing — which is a *managed connector* concern, not a client-side provider leaf.

### Proposed Solution

The solution changed shape as a result of the investigation, and that pivot **is** this contribution:

1. **Documented the constraint and presented options — no recommendation.** I posted the coupling finding to the maintainer (@atlamors) on #305 with a neutral 3-path table (OpenAI-compat + OAuth / native + Express key / native + OAuth) and two questions (which path; contract-vs-custom-code + a `google-auth-library` dependency), deliberately letting him choose the direction rather than steering it.
2. **Maintainer reframed the issue.** He agreed Vertex doesn't fit the static-key leaf shape and moved it under the managed **NueOS Cloud inference relay** (issue #421), alongside Bedrock and Azure — a server-side managed connector, not a client-side leaf. He credited the analysis and invited me into the architecture work.
3. **My read going into the session (not yet confirmed by the maintainer):** #421 appears to split into a public, leaf-shaped **client-side `nue-cloud` / `nue-ai-api` provider leaf + session/token schema** (buildable and mergeable in the open repo) versus a closed, commercial server side (control plane, regional relay, provider connectors). Based on that, the client-leaf slice looks like the piece I'm best positioned to own — but which piece is actually mine gets decided in the design session, not before it.
4. **Confirmed next step:** the maintainer is assigning us both to #421 and running a collaborative design session to "design backward from the goal line" and settle the provider-leaf patterns and schemas. My goal for that session is to come out with a concrete, mergeable first slice I can implement (ideally the client leaf), then build it in small PRs.

### Implementation Plan

**Understand:** Vertex can't be a static-key provider leaf (auth/API-surface coupling + templated endpoint). It belongs in the managed-inference-relay architecture (#421); the client-facing `nue-cloud` leaf + its schema *appears* to be the public, mergeable piece I could own — to be confirmed in the design session.

**Match:** Client-leaf shape follows the existing OpenAI-compatible protocol-wrapper pattern (`protocols/openai-api/` + thin leaves like `groq/`, `ollama/`); for anything needing custom construction/auth logic, `providers/anthropic/` (with its `implementation.ts`) is the reference. The generated-catalog + roster-test workflow is the same one I used for the merged Groq leaf.

**Plan:** 
1. Co-define the client-facing session/token protocol + leaf schema with the maintainer.
2. Implement the `nue-cloud` client provider leaf against that contract (stub/mock the relay initially so it's testable without the closed backend).
3. Regenerate the provider catalogs and add the leaf to the roster tests.
4. Land it as incremental PRs rather than one large change.

**Implement:** Pending the design session; branch to be defined once the client-leaf slice is scoped (working branch `feat/google-vertex-ai-model-provider` may be repurposed/renamed).

**Review:** Conventional commits scoped to the package; `pnpm typecheck && pnpm lint && pnpm test && pnpm build` green; `check:generated` in sync; leaf files match the certified-leaf anatomy.

**Evaluate:** The leaf resolves through `resolveProviderFactory` / `createProvider`, the roster/definition tests pass, and a configured `nue-cloud` entry can round-trip a request against the stubbed relay.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week Progress (Solution Approach)

- Researched Vertex's auth against the leaf contract (runtime code, not just definitions) and found the auth/API-surface coupling that makes a Groq-style thin leaf impossible.
- Posted the finding to the maintainer as neutral options (3-path table + 2 questions), no recommendation — let him choose the direction.
- Maintainer agreed the issue was mis-scoped and reframed Vertex under the managed-inference-relay architecture (#421); credited the analysis and invited me into it.
- Analyzed #421 + the relay design doc; my read is that it splits into a public, leaf-shaped client piece (`nue-cloud` leaf + schema) versus a closed server side — which piece is mine is still to be decided in the session, not settled.
- Accepted involvement, being assigned to #421; a design session is pending to define the client-facing protocol and carve out a first implementable slice.
- **Key decision:** rather than force Vertex into the current leaf model (which would have shipped something silently wrong), the contribution pivoted to shaping the correct architecture and taking the one slice I can actually build and merge. Kept a parallel track of separate leaf issues as the reliable near-term PR deliverable.

### Code Changes

- **Files modified:** None yet — this phase is investigation + design (no implementation until the design session defines the client-leaf contract).
- **Key commits:** N/A
- **Approach decisions:** See "Key decision" above and Solution Approach.

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
