---
Status: Active
Champions: @justinfagnani
PR: https://github.com/lit/rfcs/pull/16
---

# Lit 3.0

A proposal for the next major release of Lit

## Objective

Allow us to make a few breaking changes that will let us reduce some engineering complexity, and very lightly reduce bundle size.

### Goals
- Require no code changes from developers using non-deprecated Lit 2.0 APIs
- Drop IE support
- Publish ES2021
- Small cleanups (remove Lit pre-2.0 compatibility workarounds)

### Non-Goals
- Drop support for the Shady DOM polyfill
- Not require additional polyfills for older browsers

## Motivation

### IE Support removal

IE11 is now unsupported by Microsoft, most major websites, many frameworks, and apparently most of our users. The times we've asked our users if they need IE support, we haven't heard from anyone that they do at all. While this doesn't mean we don't have any users who require IE11 support, all signs point to it finally being time for us to move on.

Doing so will let us turn down our flaky IE11 test runners and remove some IE-specific workarounds.

### Publishing ES2021

We published Lit 1.0 packages as ES2019 because of spotty browser support for some ES2021 features like logical assignment. The situation is much better now, and publishing native optional chaining and logical assignment expressions will give us a small bundle size and perf improvement. 

### Clean up deprecated APIs

We need to start the habit of regularly removing previously deprecated APIs to keep the codebase clean.

## Detailed Design

### IE11 removal

- Turn down our IE test runners
- Remove workaround for IE normalizing text nodes during clone ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/lit-html.ts#L974))
- Simplify bound attribute handling ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/lit-html.ts#L911))
- Simplify `styleMap()` style removal ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/directives/style-map.ts#L82))
- Use `globalThis` ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/lit-html.ts#L15))
- Use `toggleAttribute()` ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/reactive-element/src/reactive-element.ts#L1126))
- Use `Element.prototype.after()`, `.before()`, `.replaceChildren()` where beneficial

### Publish ES2021

- Update tsconfig target and lib to es2022
- Remove unused catch bindings ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/reactive-element/src/reactive-element.ts#L351))

### Misc / Old API removal
- Remove `UpdatingElement` export from lit-element.ts ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-element/src/lit-element.ts#L84))
- Remove extra exports from lit-element/index.ts ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-element/src/index.ts#L10))
- Move SSR hydration support modules to `@lit-labs/ssr-client`
- Remove `requestUpdateThenable` ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/reactive-element/src/reactive-element.ts#L42))
- Remove old API warnings ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/reactive-element/src/reactive-element.ts#L810))


## Implementation Considerations

### Implementation Plan

#### Branches

If we keep the breaking changes to this version to only dropping IE11 support and publishing ES2020 we may be able to land the changes on `main` in quick succession with no special release branches.

This may make importing into Google harder though - say if we batch the IE11 changes into one PR and that breaks internal Google tests. So we may want to make a `2.x` branch and commit PRs to `main` more incrementally.

### Backward Compatibility

There should be no required _code_ changes for users of non-deprecated* APIs.

\* If we have deprecated everything properly. For the `UpdatingElement` export at least, we did miss deprecating it.

### Testing Plan

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

This should have a very small positive impact on code size and perf due to using more native JS syntax.

### Interoperability

N/A

### Security Impact

N/A

### Documentation Plan

* Blog post
* Update version selector on lit.dev. Since this changes to outward API, v2 and v3 can share the same option.
* Update API docs. The only removals would be `UpdatingElement` and the decorators, etc., exports from `lit-element.js`. Instead of an entirely new set of API docs, maybe we can somehow preserve and mark those as removed.

## Downsides

The breaking changes could effect non-IE browsers that we don't have test coverage for (UC Browser, Cobalt, etc). We really have no way of knowing, since these browsers aren't testable on common CI systems.

Publishing ES2021 could break existing toolchains that don't support the syntax. Sites like unpkg.com have slightly older parsers (only when using the `?module` flag for unpkg). We will need to deal with this eventually as we want to adopt native class fields, decorators, etc. The lit.dev playground will continue to work as it doesn't use the unpkg `?module` flag.

## Alternatives

We don't have to do a breaking change at all, for now.
