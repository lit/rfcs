---
Status: Active
Champions: {{ your_github_username }}
PR: {{ update_with_pr_number }}
---

# New Language Server and VS Code extension

This RFC proposes that we create a new language server and VS Code extension that are based on `@lit-labs/analyzer` to be a first-party project.

## Objective

Deliver a great developer experience for IDE users writing Lit code.

### Goals
- Enable standard DX features for Lit templates in VS Code:
  - Syntax highlighing
  - Type-checking
  - Intellisense: jump-to-definition, hover-over docs, etc.
- Enable those features in other editors via a Language Server
- Enable type-checking in build processes via TypeScript or typescript-eslint plugins

### Non-Goals
- Linting - which should be done via an ESLint plugin

## Motivation

The currently reccomended Lit VS Code extension, lit-plugin, suffers from a few problems:
1. It's not owned by the Lit team, and we don't have owner rights on the extension marketplace.
2. It isn't based on our `@lit-labs/analyzer`, and sometimes falls out-of-date with new Lit features.
3. It has a very high degree of abstraction in its implementation.

All of this makes it difficult for us to maintain.

## Detailed Design

We will write and publish a new VS Code extension (name TBD). Because we want it to be based on the first-party Lit Analyzer, it will be written mostly from scratch, and not fork the existing lit-plugin.

## Implementation Considerations

### Implementation Plan

There are several libraries that are required to come together to compose a fully functional extension. We will build a language server so that Lit IDE features can be used by developers using editors other than VS Code. And we will build the analysis features in packages outside of the extension and language server so that they can be used in other tools if needed.

### Backward Compatibility

In the underlying type-checking and linting libraries we will strive to keep a opt-in backcompat mode for users migrating from the lit-plugin.

### Testing Plan

Since developers will have varying versions of VS Code of TypeScript, we will need to test against multiple versions of TypeScript. Setting this up is a bit complicated, but we have experience from lit-plugin on how to do this.

### Performance and Code Size Impact

No performance or code size impact on core libraries.

### Interoperability

This extension will be specifically for developing Lit components, and not designed to support other web component libraries.

Support for consuming arbitrary web compompents will be implemented by supporting the Custom Elements Manifest standard. Any web component that publishes a manifest will be able to participate in type-checked templates, hover-over docs, etc.

### Security Impact

This extension will not run user code.

### Documentation Plan

We will need documentation in the lit.dev tooling section.

## Downsides

This is a lot of work to create and maintain.

## Alternatives

We could fork the lit-plugin. This would require a lot of refactoring to use the `@lit-labs/analyzer` and to factor out a language server.
