---
Status: Active
Champions: @43081j
PR: #17
---

# Official postcss/tailwind support

As part of unifying the standard tools around Lit, it would make sense to move
the [postcss-lit](https://github.com/43081j/postcss-lit) project into the
Lit monorepo as an official solution to inline CSS transforms (e.g. tailwind).

## Objective

A strategy for how we will move the project into the monorepo.

### Goals
- Moving the postcss-lit project into the Lit monorepo
- Potentially documenting the project in an appropriate place (or a blog post)

### Non-Goals
- Transfer of responsibility (it is expected I will continue being the primary
maintainer to avoid extra burden on the Lit team)

## Motivation

The project already functions well and has high usage, but could benefit hugely
in terms of discovery and support by being in the monorepo.

- **Discovery:** we are regularly asked by users how to integrate Lit with
tailwind and other such CSS solutions. Having an officially supported and
maintained integration will make this clearer
- **Support:** we are more likely to receive community contributions through
issues and pull requests

It will also become much easier to keep the integration in sync with the
most current version of Lit, as we are likely to update it in line with
any breaking changes elsewhere in the monorepo.

## Detailed Design

A new `packages/postcss-lit` directory can be created in the monorepo which
is initially an exact copy of the existing `postcss-lit` package.

Once the sources have been copied across, we can continue publishing the package
under the existing `postcss-lit` name for backwards compatibility.

After the move has been made, we should also do the following:

- Migrate the CI workflow to be better aligned with other pipelines in the
monorepo
- Add a new `examples/` directory with full examples of how and when to use
`postcss-lit`
  - In particular, we should include a tailwind example, and a basic example

## Implementation Considerations

### Implementation Plan

A single PR should be enough to begin with: copying the existing sources
to the agreed directory and ensuring CI is setup correctly.

### Backward Compatibility

N/A

### Testing Plan

Existing tests should be enough once copied across.

### Performance and Code Size Impact

N/A

### Interoperability

N/A

### Security Impact

N/A

### Documentation Plan

We should create articles explaining the two ways users can transform
CSS:

- Inline CSS via postcss-lit
- External CSS via rollup plugins (using CSS imports)

These could be linked in the Lit website.

In addition to these articles, we should publish some working examples (as
mentioned earlier) in the repo itself. Particularly tailwind, a basic setup,
and possibly an example of using postcss transforms.

## Downsides

N/A

## Alternatives

N/A
