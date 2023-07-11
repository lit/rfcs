---
Status: Active
Champions: @AndrewJakubowicz
PR: https://github.com/lit/rfcs/pull/21
---

# @lit-labs/compiler

A new labs package for preparing Lit templates at build time.

## Objective

Provide an optional build time performance improvement for the first render of a lit template.


### Goals

- Provide a build time transform that replaces lit `html` tagged template expressions with compiled templates.
- Compiled and Uncompiled template results can be mixed and matched transparently.
- Provide a TypeScript transformer that can be used with a rollup plugin such as [@rollup/plugin-typescript](https://www.npmjs.com/package/@rollup/plugin-typescript).
- Compiler can be run on JavaScript and TypeScript files.

### Non-Goals

- The compiled template does not have to be syntactically identical to a runtime prepared template, only semantically identical.
- Other template transformations. This is only for performance.

## Motivation

`lit-html` has [three internal rendering phases](https://github.com/lit/lit/blob/main/dev-docs/design/how-lit-html-works.md#summary-of-lit-html-rendering-phases) which are all optimized to cache and reuse previous work done. However, the very first `render()` must still execute all three phases.

The @lit-labs/compiler will provide a way to opt into precompiling the templates, moving one of the internal rendering phases to build time.

By precompiling `html` tagged template literals, the runtime only needs to execute two internal rendering phases reducing the first render performance. As a nice benefit, any subsequent updates that encounter a new compiled `html` tagged template literal will also benefit from the performance improvement and can skip the prepare phase.

## Detailed Design

At a high level, the `@lit-labs/compiler` package provides a transformer that converts:

```js
const hi = (name) => html`<h1>Hello ${name}!</h1>`;
```

into:

```js
const compiled_html = (s) => s;
const lit_template_1 = { h: compiled_html`<h1>Hello <?></h1>`, parts: [{ type: 2, index: 1 }] };
const hi = (name) => ({ _$litType$: lit_template_1, values: [name] });
```

`lit-html` version 2.7.5 and up already support a `CompiledTemplateResult` type. This transform converts the `` html`<h1>Hello ${name}!</h1>` `` `TemplateResult` into a `CompiledTemplateResult` which semantically behaves the same but with greater initial render performance. The update performance is unchanged.

### Transformer Design

The transformer should be completely syntactic. Initially only compiling any `html` template expressions if `html` was imported from `lit` or `lit-html` directly.

Each individual `TemplateResult` is split into two declarations, a `CompiledTemplate` and `CompiledTemplateResult`. The `CompiledTemplate` is hoisted to the module scope, and contains the prepared HTML and template parts for the original `TemplateResult`. The `CompiledTemplateResult` replaces the `TemplateResult` and binds the static `CompiledTemplate` with the dynamic `values` array.

If a part is used that requires a constructor, the compiler
will also insert an import such as:

```js
// ...
import { _$LH as litHtmlPrivate_1 } from "lit-html/private-ssr-support.js";
const { AttributePart: _$LH_AttributePart, PropertyPart: _$LH_PropertyPart, BooleanAttributePart: _$LH_BooleanAttributePart, EventPart: _$LH_EventPart } = litHtmlPrivate_1;
```

## Implementation Considerations

### Implementation Plan

Create a new `@lit-labs/compiler` package which exports a Typescript transformer.

### Backward Compatibility

Compiled templates require the version of lit-html to be greater than 2.7.5.

### Testing Plan

Unit tests are sufficient for testing that the transform is applied correctly. An additional test target `test:dev:compiled` will be added to lit-html to compile the test suite using the transformer ensuring that most lit-html tests behave the same with a `CompiledTemplate`.

> Note: Some lit-html tests need to be skipped, for example dev mode warnings that will no longer execute if the template has been compiled.

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

This RFC has been prototyped: https://github.com/lit/lit/pull/3984 to gather benchmarks. Note that these are all subject to change based on the final implementation. These initial benchmarks support that compiling templates do result in a measurable performance improvement, with greater performance improvements occurring in template heavy contexts.

**Kitchen sink** average tachometer benchmark results:

- [**6.7% faster**]: `render-compiled`: 7.93ms - 8.10ms, `render-uncompiled`: 8.50ms - 8.69ms
- [roughly same]: `update-compiled`: 17.92ms - 18.56ms, `update-uncompiled`: 18.08ms - 19.00ms
- [roughly same]: `nop-update-compiled`: 5.53ms - 5.66ms, `nop-update-uncompiled`: 5.64ms - 5.80ms
 - [**9.3% larger gzipped file size**]: `index.js` uncompiled is 2216 bytes gzipped, or compiled it is 2443 bytes gzipped.

**Template Heavy** average tachometer benchmark results:

 - [**46% faster**]: `render-compiled`: 9.51ms - 9.67ms, `render-uncompiled`: 17.66ms - 18.03ms
 - [**21% faster**]: `update-compiled`: 26.40 - 26.91ms, `update-uncompiled`: 33.62 - 34.51ms
 - [**5.6% larger gzipped file size**]: `template-heavy.js` uncompiled is 202,544 bytes gzipped, or compiled it is 214,495 bytes gzipped.

**repeat** average tachometer benchmark results:

 - [**14% faster**]: `render-compiled`: 7.40ms - 7.62ms, `render-uncompiled`: 8.56ms - 8.94ms
 - [**8% faster**]: `update-compiled`: 189.57ms - 197.54ms, `update-uncompiled`: 206.51ms - 218.68ms
 - [**7.1% larger gzipped file size**]: `index.js` uncompiled is 3072 bytes gzipped, or compiled it is 3307 bytes gzipped.

### Interoperability

This labs package is not interoperable as it specifically optimizes Lit template results using specific knowledge of the lit-html internal render phases.

### Security Impact

The prepared compiled template must be a tagged template literal to defend JSON injection attacks. Otherwise the `@lit-labs/compiler` package is only a transform and has no security impact.

### Documentation Plan

This package will initially be documented in its own README. If it stays on track to graduation, we should document this package under the *Tools and Workflows* section on lit.dev. The package on GitHub will also include a TypeScript and JavaScript example of using `@lit-labs/compiler`.

## Downsides

The downside of this approach is that it requires the code to have a build step which can add complexity and build specific setups. There are also slight mismatches in the precompiled prepared html, such as lit markers not required in comment nodes to reduce file size.

## Alternatives

No alternatives were considered and rejected.
