---
Status: Proposed
Champions: @AndrewJakubowicz
PR: 0
---

# @lit-labs/observers directive based element observing

Extend [@lit-labs/observers](https://www.npmjs.com/package/@lit-labs/observers) to provide a declarative API for observing
child elements with `IntersectionController`, `ResizeController`, and
`MutationController`.

## Objective

Propose a small directive based API that will remove the imperative boilerplate required to observe child elements.

### Goals
- Provide a declarative API for observing elements in the html tag function.
- Make it easy to return a different DirectiveResult based on the last observed entry for an element.
- Provide the option of observing a Ref to enable observing at a distance.

### Non-Goals
- Provide an API that works outside of Lit.
- Modify PerformanceController which doesn't observe elements.

## Motivation

Our observers controllers currently shine when observing the host, but fall short when observing child elements. For example, currently observing an element other than the host requires:

1. Instantiating the observers controller on the host.
2. After the first update, query the rendered elements in the shadowRoot and imperatively `observe` them.
3. Implement bookkeeping logic in the `callback` to handle changes on the observed elements.

This code does not provide much (or maybe any) benefit to using the labs/observers package over using a native observer.

This is a significant change for the observers package, and by creating an RFC there is an opportunity for feedback, iterating on the design, and enabling multiple use cases.

## Detailed Design

TODO: Describe the change in detail. Include API changes and usage examples. Add your own sub-headings as necessary.

## Overview

The two proposed instance methods are:

- `lastEntryFor(target: Ref | Element): ObserverEntry | undefined`
- `observeDirective(cb?: (entry?: ObserverEntry) => unknown)`

## Implementation Considerations


## TODO - add implementation and design.


### Implementation Plan

This change can be made as a single PR.

### Backward Compatibility

This proposal does not break backward compatibility. It's additive only.

### Testing Plan

Extending the existing tests with the existing infrastructure should be sufficient.

### Performance and Code Size Impact

What impact will this proposal have on performance and code size? What benchmarks should we create to evaluate the proposal?

Currently labs/observers does not depend on anything. Thus this proposal adds some code size for the implementation but also pulls in a dependency on directives.

Performance of this proposal is mostly related to memory, as the observers will need to cache the last seen entry.


### Interoperability

This proposal leverages a lit-html feature called [Directives](https://lit.dev/docs/templates/custom-directives/), which are functions that can extend Lit by customizing how a template expression renders. It cannot be a community protocol and is tightly coupled to Lit.

### Security Impact

N/A

### Documentation Plan

Do we need to create or update any documentation to complete this proposal? Does related documentation have a clear home in our docs outline? What playground examples or tutorials should be created?

The labs/observers README should be expanded with these use cases.

## Downsides

Many proposals involve trade-offs. What are they for this proposal and what are the downsides of this approach?

- Directives are lit-html only. (is there an option to make them optional?)
- TODO - explore if the same element can appear in the callback entry array. This could be a slight implementation wrinkle and may need exploration of the stateful object.

## Alternatives

What alternatives were considered and rejected? Why?

TODO