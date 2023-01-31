---
Status: Active
Champions: {{ rictic }}
PR: {{ update_with_pr_number }}
---

# Hot Elements

A new Lit Labs package for supporting hot module replacement with custom elements.

## Objective

### Goals
- Instantiating a new custom element, post HMR, should use the newest version of the constructor.
- Provide an API for custom element classes so that they can add additional handling of HMR. For example, in LitElement we can patch and rerender existing elements in the page.
  - This API should be implementable by a base class, so that in many cases no changes are needed to an element for it to achieve optimal HMR.
- Support @web/dev-server-hmr specifically and the [esm-hmr](https://github.com/FredKSchott/esm-hmr) protocol more generally.

### Non-Goals
- Support all HMR servers and protocols, at least initially.
- Work perfectly. Need to set expectations that, due to fundamental JS semantics, HMR can't and won't always be perfect.

## Motivation

HMR can be a boon to dev productivity by reducing the latency of the edit/refresh loop, particularly for larger apps.

## Detailed Design

We'll start by building off of [redefine-custom-elements](https://github.com/caridy/redefine-custom-elements), which does some strategic patches to HTMLElement and the Custom Element Registry – in a style similar to the scoped custom element polyfill – in order to get runtime control of control custom element registration and instantiation. With redefine-custom-elements active, re-registering a custom element tagname will result in the new constructor being used whenever that element is instantiated.

On top of that, we're adding a lightweight protocol for custom element classes to participate in HMR. If a custom element class has a static method named `notifyOnHotReplace`, then when that custom element is re-registered, we will call that method on the original version of the class. The method has the signature:

```typescript
static notifyOnHotReplace(tagname: string, updatedClass: typeof this): void;
```

## Implementation Considerations

### Implementation Plan

The initial implementation will just be a single PR. We've been using something similar inside google3 for a little while now, and it seems to work decently well – though the HMR model is somewhat different in Google's proprietary stack.

### Backward Compatibility

This is an experimental API. We can remove the notifyOnHotReplace method from LitElement without breaking anything, because it's only present in dev mode, and it's typed as potentially undefined, so any user should expect that it may not be present.

If we change the HMR element protocol in a non-backwards compatible way, we should rename the static method.

### Testing Plan

To test this properly, we will need integration tests with a real web dev server, using a browser-controlling system like Playwright. The initial PR may not have tests.

### Performance and Code Size Impact

There should be no impact on the Lit prod builds, as all code should be gated behind `if (DEV_MODE) {...}`.

### Interoperability

If we get good feedback, this should absolutely become a community protocol.

### Security Impact

This is a devmode only feature, and should not have any security implications.

### Documentation Plan

After some initial user testing, we should add some info about this to https://lit.dev/docs/tools/development/ and potentially to our starter elements.

## Downsides

Added complexity, and missed expectations on reliability of HMR.

## Alternatives

Live reload works ok for many users, and that will continue to work. This is opt in.
