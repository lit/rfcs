---
Status: Active
Champions: @justinfagnani
PR: {{ update_with_pr_number }}
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

As [standard decorators](https://github.com/tc39/proposal-decorators) are starting to ship in [compilers](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html#decorators) and soon in VMs, we need to prepare for migrating Lit to use them

Standard decorators will allow us to unify our surface syntax which is currently different in plain JS and compiled sources using TypeScript or Babel. This will let us use decorators as the one way of declaring reactive properties, instead of also offering the `static properties` feature. Removing `static properties` in turn lets us remove the infrastructure for dynamically creating reactive properties, simplifying ReactiveElement.

## Detailed Design

Detailed design of the new decorators should be covered in another RFC. This RFC focuses on:
* That standard decorators become the only way to declare reactive properties
* That breaking changes are made to remove support for dynamically adding reactive properties, since that will not be used by the new decorators
* A multi-stage plan for making the changes so that developers can incrementally migrate

### Audiences

If we group our developers by how they relate to decorators, we have four distinct audiences:

- JavaScript developers not yet using decorators
- TypeScript developers using legacy experimental decorators
- TypeScript developers not yet using decorators because of forward compatibility concerns
- TypeScript developers stuck using legacy experimental decorators because of backward compatibility concerns

This plan needs to address all audiences during the migration.

### End State

After all stages of the implementation plan are complete:

- Standard decorators are the only decorator implementation
- Decorators are the one way to define reactive properties
- Remove `static properties`, `static createProperty()` and associated APIs
- Remove `static addInitializer()` since the standard decorator API provides this
- Core decorators (`@property`, `@state`) are exported from the main `reactive-element`, `lit-element` and `lit` modules.

### Standard Decorators

Standard decorators have a much different design than TypeScript's experimental decorators. Rather than getting access to the class and making dynamic, imperative changes to the class, standard decorators have no direct access to the class or the class member they're decorating and must instead return an object describing a replacement for the class member. For instance class accessor decorators return an object with get and set methods.

This makes standard decorator usage must more static than experimental decorators. They can't add new class members, and can't change the type of member they decorate, and ReactiveElement's `static createProperty()` API is simply not callable.

The good news is that our decorator implementations are likely to be smaller and simpler. For example, the two main functions of our `@property()` decorator are to replace a field with a getter/setter pair that calls `requestUpdate()` in the setter, and to store metadata about the field for use in attribute reflection.

For example, a simplified decorator that only replaces an accessor might look like:

```ts
export const property =
  (options?: PropertyDeclaration) =>
  <C extends ReactiveElement, V>(
    _target: ClassAccessorDecoratorTarget<C, V>,
    context: ClassAccessorDecoratorContext<C, V>
  ): ClassAccessorDecoratorResult<C, V> => {
    const {access, name} = context;
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

## Implementation Considerations

### Implementation Plan

The plan is divided into stages which must be shipped as releases.

#### I. Add standard decorators (non-breaking)

This stage adds support for the new decorators standard in a backward-compatible way for both TypeScript and JavaScript developers.

##### Requirements
* TypeScript ships decorator metadata

##### Changes
* Add new module with standard decorators
    * Place them in a new module: `'lit/std-decorators.js'`
* Re-export legacy decorators from `'lit/legacy-decorators.js'`
* Add new module with a new implementation of _experimental_ decorators that work the same way standard decorators do:
    * A _third_ set of decorators: `'lit/experimental-decorators.js'`
    * They require `accessor`, replace the accessors on the prototype, do not call `static createProperty()`, store metadata in `[Symbol.metadata]`.
    * This is so that users stuck on the experimental decorators setting (ie, google3) have a path forward when we remove the infrastructure that supports them.
    * This decorator set will also be _use site_ compatible with standard decorators. Only the import will need to change to upgrade to standard decorators.
    * There is no great way to emulate the standard `addInitializer()`, so we keep our `static addInitializer()` un-deprecated.
* Non-core packages with decorators (@lit/labs) must follow a similar plan
    * We can skip the legacy-decorators step with more aggressive breaking changes for labs packages.
* Deprecate legacy experimental decorators, `static createProperty()`, `static getPropertyDescriptor()`, etc. but _not_ `static properties`.
    * Since there are no native implementations of decorators yet, we don't quite yet want to deprecate the main developer-facing API for property declaration.
* Deprecate the `noAccessor` property option.
* Update `static elementProperties` to read from `[Symbol.metadata]`
    * Standard decorators cannot call into the `static createProperty()` API, so they must place property options into `[Symbol.metadata]`. We should use that as the source-of-truth going forward.
* Vend codemods to upgrade decorator usage:
    * One to migrate to the new standard decorators
    * One to migrate to the new experimental decorators

#### II. Prefer standard decorators (breaking)

This stage is still usable in non-decorator environments via `static properties`, but we move decorator modules around so that legacy use must use the non-default module names (`'lit/legacy-decorators.js'`) \

##### Requirements

* Stage I has landed.
* Preferred: At least one browser has shipped decorators

##### Changes

* Move standard decorators implementation to <code>'lit/decorators.js'</code>
* Re-export standard decorators from <code>'lit/std-decorators.js'</code>
* Deprecate <code>static properties</code>
* Deprecate <code>static addInitializer()</code>
* Vend codemod to migrate standard decorator imports

#### III. Remove legacy decorators (breaking)

This stage requires decorators for creating properties. It is no longer usable in non-decorator-supporting environments without transpilation.

##### Requirements

* Native decorators are shipping in all major browsers.
* Stage II has landed

##### Changes

* Remove previously deprecated APIs
* Deprecate `'lit/std-decorators.js'` module
* Re-export `@customElement()`, `@property()` and `@state()` from the main reactive-element, lit-element, and lit modules.
    * This increases the core module size, which is paid for by removing the deprecated APIs.
    * Other decorators are more optional and remain in their own modules.


#### IV. Cleanup (breaking)

When all developers can use the standard decorators API we can remove the experimental decorators fully.

##### Requirements

* Native decorators are shipping in all major browsers.
* Stage II has landed

##### Changes

* Remove <code>'lit/std-decorators.js'</code>
* Remove <code>'lit/experimental-decorators.js'</code>
* Vend codemod to move experimental decorator imports to standard decorators

### Backward Compatibility

See Implementation Plan

### Testing Plan

N/A

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
