---
Status: Active
Champions: @justinfagnani @43081j
PR: 
---

# @lit-labs/eslint-plugin-lit

An TypeScript ESLint plugin for Lit projects that performs type-aware linting and augmented type-checking of LitElement components and standalone lit-html templates.

## Objective

Assist developers in writing correct Lit components and templates with a fully-supported, first-party tool.

This RFC proposes that we bring James Garbut's eslint-plugin-lit work into the Lit monorepo, with some major changes, and combining it with our other first-party tooling:

- Use the typescript-eslint infrastructure
- Base the analysis on @lit-labs/analyzer
- Add template type-checking rules such as those found in runem's [lit-analyzer](https://github.com/runem/lit-analyzer/tree/master/packages/lit-analyzer)

### Goals
- Type-check lit-html template bindings
- Check lit-html templates for well-formedness and other errors
- Check LitElemenent & ReactiveElement property and style declarations
- Offer value to IDE users that only have an ESLint plugin, and not an otherwise Lit-specific IDE plugin
- Make it easy to add new rules to lint usage of other first-party Lit libraries like Context or Task

### Non-Goals
- Take over the non-linting feature of runem's lit-analyzer work - such as syntax-highlighting, auto-complete - which belong in a Language Server

## Motivation

### Tool maintenance and ownership

We want to start bringing Lit tooling into the Lit monorepo to keep it more up-to-date, make it easier for the Lit team to contribute to, and hopefully expand the set of community contributors.

### Convergence of linting tools

Right now there are two main linters available for Lit: eslint-plugin-lit and lit-analyzer. They bother have different sets of rules. lit-analyzer performs TypeScript type-checking on templates, and eslint-plugin-lit has more syntactic checks.

We would like to merge the functionality into one linter. and have that be a plugin for typescript-eslint. This will let existing ESLint users add the plugin to easily check Lit projects locally, in CI, and in editors with no additional IDE plugins of CI scripts.

## Detailed Design

_TODO_

## Implementation Considerations

### Implementation Plan

_TODO:_

There are two main options:

1. Copy the eslint-plugin-lit codebase into a package in the monorepo
2. Start with a clean typescript-eslint project since there are some tasks about getting typescript-eslint and @lit-labs/analyzer integrated that might be easier in a new project.

### Backward Compatibility

_TODO_:

It's probably tractable to maintain backwards-compatibility with eslint-plugin-lit, though some rules could possibly be deprecated.

It might be harder to maintain backwards-compatibility with lit-analyzer which has its own configuration and comment syntax for disabling rules. We could likely approach a 1-1 mapping or rules so that migration is easy.

### Testing Plan

We should copy the test suites from both eslint-plugin-lit and lit-analyzer.

### Performance and Code Size Impact

No impact on the client code.

### Interoperability

These tools are Lit-specific and not intended to be used on other web component libraries.

Some of the rules will apply to ReactiveElement (checking properties, etc). If we see a new ReactiveElement-based library out there, we can consider supporting it somehow.

Interoperability is a consideration for analyzing _dependencies_. We do not wish to create a meta-analyzer that can host plugins to analyze web component dependencies from arbitrary libraries. We instead want to consumer custom element manifests from dependencies and expect dependencies to publish them.

Being strict about this could be a regression to users if dependencies have not yet published manifests. We may want a (temporary?) option to analyze Lit dependencies until publishing manifests is a norm in the ecosystem.

### Security Impact

No negative impact on client-side code.

We may be able to add security-related rule like disallowing `javascript:` in attributes.

### Documentation Plan

We need the tools documented on lit.dev in the **Tools and Workflows** section.

## Downsides

It is more work to move this project into the Lit monorepo.

## Alternatives

We could keep Lit tooling as-is. The current situation is a maintenance burden and confusing to users.
