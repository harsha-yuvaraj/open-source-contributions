# Contribution #1: Adapter: Groq Model Provider

* **Contribution Number:** 1
* **Student:** Harshavardan Yuvaraj
* **Issue:** https://github.com/orthogonalhq/nous-core/issues/309
* **Status:** Phase I Complete

## Why I Chose This Issue

I chose this issue because adding a model provider adapter is a practical, well-defined backend task. The `nous-core` project integrates with different LLM providers, and adding support for Groq expands the choice of available inference options for users. 

This task matches my interest in backend integrations and type systems. The implementation requires setting up the adapter class, linking it to the framework's model provider registry, mapping configuration schemas, and ensuring both standard text generation and streaming token payloads are handled correctly. It allows me to contribute a clear, functional utility to the codebase, helping me ease into open-source contributions as this will be my very first.

## Understanding the Issue

### Problem Description

The `nous-core` framework manages model coordination through its Subcortex cognitive layer, which routes tasks to various LLM provider adapters. Currently, there is no native adapter for the Groq API within this layer. As a result, users cannot configure their self-hosted personal assistant to utilize Groq's low-latency inference endpoints for running open-weight models.

### Expected Behavior

A user should be able to select `groq` as their model provider in the application configuration. The Subcortex layer should instantiate a Groq provider adapter that reads the necessary credentials (via environment variables or autonomic configuration), maps the requested open-weight models (like Llama 3 or Mixtral), and handles both standard generation and Server-Sent Events (SSE) token streaming. All incoming configurations and payloads must pass strict Zod runtime schema validations.

### Current Behavior

The framework lacks a Groq adapter. Because the core provider types and Zod validation schemas do not recognize `groq`, passing it as a provider option causes runtime configuration validation to fail, blocking agent initialization.

### Affected Components

* **Providers Module:** A new adapter file needs to be added to host the new Groq provider class logic, manage configuration options, and handle the API request shapes.
* **Types / Configuration Schemas:** The TypeScript definitions or configuration schemas that restrict allowed provider names and model strings need to be updated to recognize `groq` and its respective models.

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

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
