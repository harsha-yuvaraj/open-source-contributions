# Contribution #3: Add Oracle Cloud OCI Storage Backend

**Contribution Number:** 3
**Student:** Harshavardan Yuvaraj
**Issue:** https://github.com/cortexproject/cortex/issues/7593
**Status:** Phase I Complete

---

## Why I Chose This Issue

I wanted a feature contribution in Go that touches a real, production system. This issue asks to add Oracle Cloud (OCI) as an object-storage backend for Cortex, which is a well-scoped, self-contained addition: it follows an existing, repeatable pattern (each backend is a small config + client wrapper registered in a central switch), so I can learn the codebase's storage abstraction deeply without the risk of touching core query or ingestion paths.

It also matches what I want to learn: how a large project cleanly abstracts multiple cloud providers behind one interface, how Cortex vendors and wraps the Thanos `objstore` library, and how configuration/flags/validation propagate across components (blocks, ruler, alertmanager). Because the upstream `objstore` library already ships an OCI provider, the work is about integration and following conventions correctly — a realistic taste of contributing a "supported backend" to an open-source infra project.

---

## Understanding the Issue

### Problem Description

This is a missing-feature issue, not a bug. Cortex can store its long-term data (blocks, ruler configs, alertmanager configs) in object storage and today supports five backends: S3, GCS, Azure, Swift, and Filesystem. There is no native support for Oracle Cloud Infrastructure (OCI) Object Storage, which blocks organizations running on Oracle Cloud from using their native storage. The upstream Thanos `objstore` library that Cortex already depends on ships an OCI provider (`providers/oci`), so the gap is purely on the Cortex integration side.

### Expected Behavior

A user should be able to select `oci` as a storage backend — e.g. `-blocks-storage.backend=oci` (and the equivalent for ruler/alertmanager) — configure it with OCI-specific flags (`-<prefix>oci.*`), and have Cortex read/write objects to an OCI bucket, exactly like any other supported backend.

### Current Behavior

`oci` is not a known backend. It is absent from `bucket.SupportedBackends`, so configuring it fails validation with `ErrUnsupportedStorageBackend` ("unsupported storage backend") in `Config.Validate()`, and `NewClient` would hit its `default` case and return the same error. No `oci.*` flags exist.

### Affected Components

- `pkg/storage/bucket/client.go` — the central backend registry (backend constants, `SupportedBackends`, `Config` struct, flag registration, and the `NewClient` switch). Primary change site.
- `pkg/storage/bucket/oci/` — new package to add (`config.go` + `bucket_client.go`), mirroring `pkg/storage/bucket/swift/`.
- Downstream consumers that validate against `bucket.SupportedBackends` automatically inherit OCI support: `pkg/alertmanager/alertstore/config.go`, plus ruler and blocks storage config.
- `vendor/` + `go.mod`/`go.sum` — must vendor `objstore/providers/oci` and the `github.com/oracle/oci-go-sdk/v65` SDK.
- Docs (generated config reference) and `CHANGELOG.md`.

---

## Reproduction Process

> Note: This is a feature request, not a bug, so there is no failing behavior to reproduce. Instead I verified the gap — confirming OCI is genuinely unsupported and that the integration points are as expected — through code inspection.

### Environment Setup

Work is on a Windows 11 machine with the Cortex repo cloned locally. Verification of the gap was done by reading the source directly; the build/vendor/test steps will be run in an environment with the Go toolchain.

### Steps to Verify the Gap

1. Searched the backend registry `pkg/storage/bucket/client.go` and confirmed `SupportedBackends = []string{S3, GCS, Azure, Swift, Filesystem}` — no OCI constant, config field, flag registration, or `NewClient` case.
2. Traced that `Config.Validate()` and the `NewClient` `default` branch both return `ErrUnsupportedStorageBackend`, so `backend=oci` is rejected today.
3. Confirmed downstream components validate against the same slice (e.g. `pkg/alertmanager/alertstore/config.go:35` calls `slices.Contains(bucket.SupportedBackends, cfg.Backend)`), so adding OCI in one place propagates to ruler/alertmanager/blocks.
4. Confirmed the pinned Thanos objstore version (`v0.0.0-20250804093838-71d60dfee488` in `go.mod`) provides `providers/oci`, but it is **not** in `vendor/` yet (only azure/filesystem/gcs/s3/swift are vendored), so it will need vendoring.

### Verification Evidence

- **Findings:** OCI is absent from every backend integration point in `pkg/storage/bucket`. The `swift` backend is the closest existing template (thin wrapper over an objstore provider), so the implementation is a well-understood, low-risk addition following an established pattern. The only real dependency work is vendoring the OCI provider and the `oracle/oci-go-sdk/v65` SDK.
- **Screenshots/logs:** N/A (feature addition — no runtime error to capture beyond the config-validation path described above).

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
