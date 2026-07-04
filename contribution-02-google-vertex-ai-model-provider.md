# Contribution #2: Adapter: Google Vertex AI Model Provider

**Contribution Number:** 2
**Student:** Harshavardan Yuvaraj
**Issue:** https://github.com/orthogonalhq/nous-core/issues/305
**Status:** Phase I — Done

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

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

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
