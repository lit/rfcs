---
Status: Accepted
Champions: @justinfagnani
PR:
---

# {{ TITLE }}

Brief summary paragraph of the RFC.

## Objective

What is the RFC trying to accomplish?

### Goals
- Specific goals of the RFC

### Non-Goals
- Specific non-goals of the RFC

## Motivation

Why is the RFC neccessary? What background information is needed to understand why?

## Detailed Design

Describe the change in detail. Include API changes and usage examples. Add your own sub-headings as necessary.

### Implementation Plan

Is there anything important to note about implementation? Can it be done in a single PR or will it needs to be staged out across several.

### Backward Compatibility

Backwards compatibility is extremely important to Lit, especially the core libraries. Does this proposal break backwards compatibility? Are there workarounds?

### Testing Plan

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

What impact will this proposal have on performance and code size? What benchmarks should we create to evaluate the proposal?

### Interoperability

Is this proposal for a feature that could be interoperable across web components not written in Lit? Does it create a tight coupling between components written in Lit? Could it be a Web Components Community Group Protocol?

### Security Impact

What impact will this proposal have on security? Does the proposal require a security review? (We have a security team available for reviews)

We especially care about the handling of untrusted user-inputs by library code so that we contnue to prevent XSS vectors.

### Documentation Plan

## Downsides

Many proposals involve trade-offs. What are they for this proposal and what are the downsides of this approach.

## Alternatives

What alternatives were considered and rejected? Why?
