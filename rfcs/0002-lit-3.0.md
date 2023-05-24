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
- Remove APIs deprecated in Lit 2.0

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

### 2.x deprecations

* `UpdatingElement` is not currently deprecated and needs to be.
* All other moved APIs are deprecated.

### IE11 removal

- Turn down our IE test runners
- Remove workaround for IE normalizing text nodes during clone ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/lit-html.ts#L974))
- Simplify bound attribute handling ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/lit-html.ts#L911))
- Simplify `styleMap()` style removal ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/directives/style-map.ts#L82))
- Use `globalThis` ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/lit-html.ts#L15))
- Use `toggleAttribute()` ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-html/src/lit-html.ts#L1893))
- Use `Element.prototype.after()`, `.before()`, `.replaceChildren()` where beneficial
- Remove test conditional on IE11 (and Chrome 41) ([example](https://github.com/lit/lit/blob/main/packages/lit-html/src/test/directives/style-map_test.ts#L14))
- Use `for..of` loops instead of `forEach()`.

### Publish ES2021

- Update tsconfig target and lib to es2022
- Remove unused catch bindings ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/reactive-element/src/reactive-element.ts#L351))
- Verify code size against regressions

### Misc / Old API removal
- Remove `UpdatingElement` export from lit-element.ts ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-element/src/lit-element.ts#L84))
- Remove extra exports from lit-element/index.ts ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/lit-element/src/index.ts#L10))
- Move SSR hydration support modules to `@lit-labs/ssr-client`
- Remove `requestUpdateThenable` ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/reactive-element/src/reactive-element.ts#L42))
- Remove old API warnings ([source](https://github.com/lit/lit/blob/f2eb97962c7e77373b3b8861ab59639de22da3d0/packages/reactive-element/src/reactive-element.ts#L810))


## Implementation Considerations

### Implementation Plan

#### Branches

As recommended by [changesets prereleases documentation](https://github.com/changesets/changesets/blob/main/docs/prereleases.md) we will work on a separate `3.0` branch, and regularly merge `main` into it. When 3.0 is ready, we will create a `2.x` branch, then merge `3.0` into `main`.

### Backward Compatibility

There should be no required _code_ changes for users of non-deprecated* APIs.

There are potential interop considerations between versions of Lit, though only when sharing objects between version, like a directive from one version and a template from another. We should be ok because of the work we did to eliminate `instanceof` checks and ensure stable APIs on implementation objects that could be shared between versions.

\* _If we have deprecated everything properly. For the `UpdatingElement` export at least, we did miss deprecating it._

### Testing Plan

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

This should have a very small positive impact on code size and perf due to using more native JS syntax.

### Interoperability

Since there are no new features, there is no impact on web component ecosystem interoperability.

### Security Impact

No changes

### Documentation Plan

* Prerelease blog post
* Blog post
* Add version selector for v3 docs
* Update tooling and workflow docs
  * Clarify and simplify the support policy.
  * Describe exactly which versions are supported (tested) in a table
  * Remove reverences to IE
* Update API docs. The only removals would be `UpdatingElement` and the decorators, etc., exports from `lit-element.js`. Instead of an entirely new set of API docs, maybe we can somehow preserve and mark those as removed.

## Downsides

The breaking changes could effect non-IE browsers that we don't have test coverage for (UC Browser, Cobalt, etc). We really have no way of knowing, since these browsers aren't testable on common CI systems.

Publishing ES2021 could break existing toolchains that don't support the syntax. Sites like unpkg.com have slightly older parsers (only when using the `?module` flag for unpkg). We will need to deal with this eventually as we want to adopt native class fields, decorators, etc. The lit.dev playground will continue to work as it doesn't use the unpkg `?module` flag.

## Alternatives

We don't have to do a breaking change at all, for now.
