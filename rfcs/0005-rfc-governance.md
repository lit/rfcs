---
Status: Active
Champions: {{ your_github_username }}
PR: {{ update_with_pr_number }}
---

# {{ TITLE }}

Brief summary paragraph describing the RFC.

## Objective

What is the RFC trying to accomplish?

### Goals
- Specific goals of the RFC

### Non-Goals
- Specific non-goals of the RFC

## Motivation

Why is the RFC necessary? What background information is needed to understand why?

## Detailed Design

Describe the change in detail. Include API changes and usage examples. Add your own sub-headings as necessary.

## Implementation Considerations

### Implementation Plan

Is there anything important to note about implementation plan? Can it be done in a single PR or will it need to be staged out across several?

### Backward Compatibility

Backwards compatibility is extremely important to Lit, especially in the core libraries. Does this proposal break backwards compatibility? Are there workarounds?

### Testing Plan

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

What impact will this proposal have on performance and code size? What benchmarks should we create to evaluate the proposal?

### Interoperability

Is this proposal for a feature that could be interoperable across web components not written in Lit? Does it create a tight coupling between components written in Lit? Could it be a [Web Components Community Group](https://github.com/w3c/webcomponents-cg) [Community Protocol](https://github.com/webcomponents-cg/community-protocols)?

### Security Impact

What impact will this proposal have on security? Does the proposal require a security review? (We have a security team available for reviews)

We especially care about the handling of untrusted user input by library code so that we contnue to prevent XSS vectors.

### Documentation Plan

Do we need to create or update any documentation to complete this proposal? Does related documentation have a clear home in our docs outline? What playground examples or tutorials should be created?

## Downsides

Many proposals involve trade-offs. What are they for this proposal and what are the downsides of this approach?

## Alternatives

What alternatives were considered and rejected? Why?
