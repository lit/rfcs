---
Status: Accepted
Champions: "@justinfagnani"
PR: https://github.com/lit/rfcs/pull/20
---

# Decorator Roadmap

This RFC describes how Lit can migrate to standard decorators with the smoothest upgrade path possible for developers.

## Objective

Migrate to using standard decorators as the only decorator implementation and unify Lit's syntax around decorators as the only way to declare reactive properties. When the migration is complete, Lit will require a decorator-supporting environment or toolchain.

### Goals

- Make Lit decorators work without transpilation
- Make the migration as easy as possible on developers
- Have a single implementation for decorators
- Unify Lit's reactive property syntax
- Simplify ReactiveElement's property handling code
- Increase component performance due to more static class initialization
- Decrease code size with simpler decorator implementations

### Non-Goals

- Change our decorators API drastically (ie, split `@property()` into `@input()`, `@output()`, `@attribute`, etc)

## Motivation

As [standard decorators](https://github.com/tc39/proposal-decorators) are starting to ship in [compilers](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html#decorators) and soon in VMs, we need to prepare for migrating Lit to use them.

Standard decorators will allow us to unify our surface syntax which is currently different in plain JS and compiled sources using TypeScript or Babel. This will let us eventually use decorators as the one way of declaring reactive properties, instead of also offering the `static properties` feature. Removing `static properties` in turn lets us remove the infrastructure for dynamically creating reactive properties, simplifying ReactiveElement.

## Detailed Design

Detailed design of the new decorators should be covered in pull requests and reviews, given that they follow the constraints in this RFC. This RFC focuses on:

- That standard decorators become the only way to declare reactive properties
- That breaking changes are made to remove support for dynamically adding reactive properties, since that will not be used by the new decorators
- A plan so that developers can incrementally migrate to standard decorators

### Audiences

If we group our developers by how they relate to decorators, we have four distinct audiences:

- JavaScript/TypeScript developers not yet using decorators
- JavaScript developers using decorators with Babel
- TypeScript developers sticking to using legacy experimental decorators
- TypeScript developers using standard decorators

This plan needs to address all audiences during the migration.

### End State

After all stages of the implementation plan are complete:

- Standard decorators are the only decorator implementation
- Decorators are the one way to define reactive properties
- Remove `static properties`, `static createProperty()` and associated APIs
- Remove `static addInitializer()` since the standard decorator API provides this
- Core decorators (`@property`, `@state`) are exported from the main `reactive-element`, `lit-element` and `lit` modules.

### Standard Decorators

Standard decorators have a much different design than TypeScript's experimental decorators. Rather than getting access to the class and making dynamic, imperative changes to the class, standard decorators have no direct access to the class or the class member they're decorating and must instead return an object describing a replacement for the class member. For instance, class accessor decorators return an object with `get` and `set` methods.

This makes standard decorator usage much more static than experimental decorators. They can't add new class members, and can't change the kind of member they decorate, and ReactiveElement's `static createProperty()` API is simply not callable.

The good news is that our decorator implementations are likely to be smaller and simpler. For example, the two main functions of our `@property()` decorator are to replace a field with a getter/setter pair that calls `requestUpdate()` in the setter, and to store metadata about the field for use in attribute reflection.

For example, a simplified decorator that only replaces an accessor might look like:

```ts
export const property =
  (options?: PropertyDeclaration) =>
  <C extends ReactiveElement, V>(
    _target: ClassAccessorDecoratorTarget<C, V>,
    context: ClassAccessorDecoratorContext<C, V>
  ): ClassAccessorDecoratorResult<C, V> => {
    const { access, name } = context;
    return {
      get(this: C) {
        return access.get(this);
      },
      set(this: C, v: V) {
        const oldValue = access.get(this);
        access.set(this, v);
        this.requestUpdate(name, oldValue);
      },
    };
  };
```

This decorator would require the new "auto accessor" feature and `accessor` keyword to use:

```ts
class MyElement extends LitElement {
  @property()
  accessor foo = 42;
}
```

### Potential and required changes in behavior with standard decorators

We intend to keep the new standard decorators implementations mostly compatible with the existing legacy decorators, but the new decorator standard does force us or allow us to make some breaking changes in behavior.

#### `accessor` keyword is required

As mention already, the `accessor` keyword will be required for all formerly field-decorators like `@property()`, `@query()`, etc.

#### Initial values don't reflect for `@property()`

The new decorator spec passes field and accessor initial values through a separate callback from `set()`. This allows us to know when we are receiving an initial value and not reflect it to an attribute. This is part of a [long-standing issue](https://github.com/lit/lit/issues/1476) where we would like to not create any attributes spontaneously on an element.

We will _not_ do this initially, as it will make migrating more challenging. We will instead open another RFC on how to opt out of initial value reflection for both experimental and standard decorators.

#### Restoring default property values when attributes are removed

We could also remember the initial property value in order to restore it when an associated attribute is removed.

We will _not_ do this, however, since there is a decent chance of harmful memory leaks from retaining initial values. We will instead investigate allowing a default value to be specified in property options. This will act as an opt-in for the behavior and retain only one object per class instead of per instance, at the cost of duplicating a default and initializer.

### Hybrid Decorators

For an incremental migration path, Lit experimental decorators should be _use site_ compatible
with standard decorators. We're calling this the "hybrid" decorator approach, and
these decorators "hybrid decorators".

To enable hybrid decorators, the existing Lit experimental decorators need some minor
breaking changes. These should most likely not impact many users.

 - `requestUpdate()` will be called automatically for `@property` and `@state` decorated accessors.
 - The value of an accessor is read on first render and used as the initial value for `changedProperties` and attribute reflection.
 - No longer support version `"2018-09"` of `@babel/plugin-proposal-decorators`.

Additionally, the existing experimental decorators must also work with auto-accessors and the `accessor` keyword, so they can be syntactically identical to standard decorators.

Hybrid decorators make migrating from experimental decorators to standard decorators incremental for existing codebases by enabling
each decorator to be independently updated to match standard decorator syntax. Once all decorators are syntactically identical to
standard decorator usage, the `experimentalDecorators` TypeScript flag can be turned off, and semantics will remain unchanged.

#### Example migration

1. Existing codebase using experimental decorators

```ts
// tsconfig uses `experimentalDecorators: true`.
class El extends LitElement {
  @property()
  reactiveProp = "initial value";

  @state()
  secondProperty = 0;
}
```

2. Incrementally make decorator usage _use site_ compatible by adding `accessor`

```ts
// tsconfig uses `experimentalDecorators: true`.
class El extends LitElement {
  @property()
  accessor reactiveProp = "initial value";

  @state()
  accessor secondProperty = 0;
}
```

3. Finally, change `experimentalDecorators` to `false` to complete the migration.

## Implementation Considerations

### Implementation Plan

The plan is to make the Lit decorators work in both experimental decorator
environments and standard decorator environments.

#### I. Make Lit decorators hybrid (breaking)

This stage adds support for the new decorators standard for both TypeScript and JavaScript developers,
and ensures experimental decorators are semantically identical to standard decorators.

##### Requirements

- TypeScript ships decorator metadata (TypeScript 5.2, scheduled for 2023-08-22)
- Babel ships decorator proposal plugin with metadata support (Babel 7.23.0, 2023-09-25)

##### Changes

- Make Lit decorators "hybrid", working in either experimental or standard environments.
- Non-core packages with decorators (`@lit/localize`, `@lit-labs/context`) must follow a similar plan
- Update `static elementProperties` to read from `[Symbol.metadata]`
  - Standard decorators cannot call into the `static createProperty()` API, so they must place property options into `[Symbol.metadata]`. We should use that as the source-of-truth going forward.

#### II. Deprecate experimental decorators

##### Requirements

- Stage I has landed.
- Preferred: At least one browser has shipped decorators

##### Changes

- Deprecate the `noAccessor` property option.
- Deprecate experimental decorators, `static createProperty()`, `static getPropertyDescriptor()`, etc. but _not_ `static properties`.
  - Since there are no native implementations of decorators yet, we don't quite yet want to deprecate the main developer-facing API for property declaration.


#### III. Remove experimental decorators (breaking)

This stage requires decorators for creating properties. It is no longer usable in non-decorator-supporting environments without transpilation.

##### Requirements

- Stage II has landed
- Native decorators are shipping in all major browsers.

##### Changes

- Remove previously deprecated APIs
- Deprecate `static properties`
- Re-export `@customElement()`, `@property()` and `@state()` from the main reactive-element, lit-element, and lit modules.
  - This increases the core module size, which is paid for by removing the deprecated APIs.
  - Other decorators are more optional and remain in their own modules.

#### IV. Cleanup (breaking)

When all developers can use the standard decorators API we can remove the experimental decorators fully as well as `static properties`.

##### Requirements

- Stage III has landed
- Native decorators are have been shipping in all major browsers for last 2 major versions.

##### Changes

- Remove all non standard decorator APIs, including `static properties`

### Backward Compatibility

See Implementation Plan.

### Testing Plan

Hybrid decorators requires testing in three configurations:

1. Experimental decorator syntax (no `accessor` keyword), with `experimentalDecorators: true`. This reflects the existing tests.
2. Standard decorator syntax, with `experimentalDecorators: true`. Tests that experimental decorators can be incrementally migrated.
3. Standard decorator syntax, with `experimentalDecorators: false`. Test that standard decorators match experimental decorator behavior exactly.

In the future, as we cleanup experimental decorators, we'll be able to remove tests in configurations `1.`, and `2.`.

### Performance and Code Size Impact

Code size will ultimately be reduced due to simpler decorators and smaller API in ReactiveElement.

The code size of components needs to be investigated since the TypeScript emit for standard decorators appears to be larger than experimental decorators. We might want to point this out and not recommend them as the default until browsers ship natively.

### Interoperability

No interoperability concerns.

### Security Impact

N/A

### Documentation Plan

We will need to document both standard and legacy decorators for a while, but since the set of decorators are the same, hopefully there is just common decorator overview and config documentation that covers each, and individual decorators are documented once.

## Downsides

This is a bit of churn on the ecosystem, but it's unavoidable and we accepted this when we shipped experimental decorators.

## Alternatives

None
