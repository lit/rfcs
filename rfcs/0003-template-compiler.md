---
Status: Active
Champions: "@AndrewJakubowicz"
PR: "https://github.com/lit/rfcs/pull/21"
---

# @lit-labs/compiler

A new labs package for preparing Lit templates at build time.

## Objective

Provide an optional build time performance improvement for the first render of a lit template.

### Goals

- Provide a build time transform that replaces lit `html` tagged template expressions with compiled templates.
- Compiled and uncompiled template results can be mixed and matched transparently, and should be semantically identical.
- Provide a rollup plugin to compile templates.
- Compiler can be run on JavaScript and TypeScript files.
- Compilation preserves source-maps.

### Non-Goals

- The compiled template does not have to be syntactically identical to templates prepared at runtime. For example comment markers will not be the same as runtime preparation.
- Other template transformations. This is only for performance.

## Motivation

`lit-html` has [three internal rendering phases](https://github.com/lit/lit/blob/main/dev-docs/design/how-lit-html-works.md#summary-of-lit-html-rendering-phases) which are all optimized to cache and reuse previous work. However, the very first `render()` must still execute all three phases.

The @lit-labs/compiler will provide a way to opt into precompiling the templates, moving the first internal rendering phase to build time.

By precompiling `html` tagged template literals, the runtime only needs to execute two internal rendering phases reducing the first render performance. As a nice benefit, any subsequent updates that encounter a new compiled `html` tagged template literal will also benefit from the performance improvement and can skip the prepare phase.

## Detailed Design

At a high level, the `@lit-labs/compiler` package provides a transformer that converts:

```js
const hi = (name) => html`<h1>Hello ${name}!</h1>`;
```

into:

```js
const compiled_html = (s) => s;
const lit_template_1 = {
  h: compiled_html`<h1>Hello <?></h1>`,
  parts: [{ type: 2, index: 1 }],
};
const hi = (name) => ({ _$litType$: lit_template_1, values: [name] });
```

This transform converts the `` html`<h1>Hello ${name}!</h1>` `` `TemplateResult` into a `CompiledTemplateResult` which semantically behaves the same but with greater initial render performance. The update performance is unchanged.

### Transformer Design

The transformer should be completely syntactic. Initially only compiling any `html` template expressions if `html` was imported from `lit` or `lit-html` directly.

Each individual `TemplateResult` is split into two declarations, a `CompiledTemplate` and `CompiledTemplateResult`. The `CompiledTemplate` is hoisted to the module scope, and contains the prepared HTML and template parts for the original `TemplateResult`. The `CompiledTemplateResult` replaces the `TemplateResult` and binds the static `CompiledTemplate` with the dynamic `values` array.

If a part is used that requires a constructor, the compiler
will also insert an import such as:

```js
// ...
import { _$LH as litHtmlPrivate_1 } from "lit-html/private-ssr-support.js";
const {
  AttributePart: _$LH_AttributePart,
  PropertyPart: _$LH_PropertyPart,
  BooleanAttributePart: _$LH_BooleanAttributePart,
  EventPart: _$LH_EventPart,
} = litHtmlPrivate_1;
```

This transformer will be exported from `@lit-labs/compiler`, and can be used anywhere TypeScript transformers are accepted, which depends on build setup. Below is an example using [Rollup](https://rollupjs.org/) with [@rollup/plugin-typescript](https://www.npmjs.com/package/@rollup/plugin-typescript):

```js
// File: rollup.config.js
import typescript from "@rollup/plugin-typescript";
import { compileLitTemplates } from "@lit-labs/compiler";

export default {
  // ...
  plugins: [
    typescript({
      transformers: {
        before: [compileLitTemplates()],
      },
    }),
    // other rollup plugins
  ],
};
```

#### What templates will not be compiled

The initial version of the compiler will not handle compilation of all templates. In the future, handling of all cases would allow us to completely remove the lit-html prepare phase from runtime.

| Name                                 | Example                                       | Why                                                                                                                                                                                                                                                                           |
| ------------------------------------ | --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Re-exported `html` tag               | `import {html} from 'my-rexported-lit';`      | Any templates using this `html` tag will not be compiled, as it isn't imported from `lit` or `lit-html`.                                                                                                                                                                      |
| `svg` tag                            | `` svg`<line />` ``                           | `svg` tags would require a change in the create phase of `lit-html`.                                                                                                                                                                                                          |
| Invalid dynamic element name         | `` html `<${'A'}></${'A'}>` ``                | It's invalid.                                                                                                                                                                                                                                                                 |
| Invalid octal literals               | `` html`\2` ``                                | It's invalid.                                                                                                                                                                                                                                                                 |
| Raw text elements with inner binding | `<textarea>${'not ok'}</textarea>`            | Raw text elements (`script`, `style`, `textarea`, or `title`) with bindings get prepared at runtime by creating adjacent text nodes and markers. The `CompiledTemplate` prepared HTML cannot represent this so the `TemplateParts` nodeIndex cannot be correctly represented. |
| `<template>` with internal binding   | `` html `<template>${'not ok'}</template>` `` | Expressions are not supported inside `<template>`. [See Invalid Locations docs](https://lit.dev/msg/expression-in-template).                                                                                                                                                  |

### Rollup plugin

An additional `@lit-labs/compiler-rollup` package will be usable as a rollup plugin that operates on TypeScript and JavaScript. An example usage looks like:

```js
// rollup.config.js (file truncated)
import minifyHTML from "rollup-plugin-minify-html-literals";
import litTemplateCompiler from "@lit-labs/compiler-rollup";

// ...
plugins: [
  sourcemaps(),
  // optional: TypeScript compilation
  nodeResolve(),
  minifyHTML(),
  litTemplateCompiler(/** options **/), // <- @lit-labs/compiler-rollup plugin
  terser(),
];
// ...
```

The compiler plugin should be ordered after all other plugins that operate on
`html` tags, (such as `minifyHTML` and `@lit/localize`), but before any manglers
or source code minifiers. The placement is required because once the compiler transforms
the templates into compiled templates, `minifyHTML` will no longer work – as it only minifies
uncompiled templates. Source code minifiers such as `terser` may rename the `html` tag
function preventing the locating of the templates to compile, so must come after.

By operating on Javascript, this package should be less prone to breakages due to TypeScript version changes while still allowing the compiler to work in both JavaScript and TypeScript projects.

The following options can be passed into `litTemplateCompiler`:

| Option    | Description                                                                     |
| --------- | ------------------------------------------------------------------------------- |
| `include` | Input file glob patterns, used to include files to compile.                     |
| `exclude` | File glob patterns to exclude from compilation. Take precedence over `include`. |

## Implementation Considerations

### Implementation Plan

Create a new `@lit-labs/compiler` package which exports a [TypeScript transformer](https://github.com/itsdouges/typescript-transformer-handbook#the-basics), and `@lit-labs/compiler-rollup` which vends a rollup plugin. No major changes in Lit core are required.

### Backward Compatibility

lit-html supports compiled templates on and above version 2.7.5.

### Testing Plan

Unit tests are sufficient for testing that the transform is applied correctly. An additional test target will be added to lit-html to compile the test suite using the transformer ensuring that all lit-html tests behave the same with a `CompiledTemplate`.

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

This labs package is interoperable as it can be safely run on any code, and will only optimize Lit templates.

### Security Impact

The prepared compiled template must be a tagged template literal to defend against JSON injection attacks. Otherwise the `@lit-labs/compiler` package is only a source code transform. Because a compiled template is processed in the same way that an uncompiled template is, we're just moving some of the work to build time. This should preserve the security invariants of the code being compiled.

### Documentation Plan

This package will initially be documented in its own README. If it stays on track to graduation, we should document this package under the _Tools and Workflows_ section on lit.dev. The package on GitHub will also include a TypeScript and JavaScript example of using `@lit-labs/compiler`.

## Downsides

The downside of this approach is that it requires the code to have a build step which can add complexity and build specific setups. There are also slight mismatches in the precompiled prepared HTML, such as lit markers which are not required with pre-compilation. The change in compiled comment markers broke one test in `lit_html` which was introspecting comment markers – something that should be uncommon in production use cases.

## Alternatives

We could not release this package, as the package will have initial implementation cost, and ongoing maintenance cost. The performance wins are large enough that it seems worth it.
We expect the code to need only occasional maintenance from TypeScript updates, as Lit's template result API is very stable.
