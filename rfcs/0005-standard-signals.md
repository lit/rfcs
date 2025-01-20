---
Status: Active
Champions: "@justinfagnani"
PR: https://github.com/lit/rfcs/pull/39
---

# Standard Signals

A new labs package that integrates the new TC39 standard signals proposal with Lit.

## Objective

Signals are now [proposed to be standardized as part of JavaScript](https://github.com/proposal-signals/proposal-signals). This would solve an issue we have with previous signals integrations where we have to chose a signals library to integrate with. Standard signals gives us an _interoperable_ primitive for shared observable state.

The signals proposal includes a [polyfill](https://github.com/proposal-signals/proposal-signals/tree/main/packages/signal-polyfill) (actually a ponyfill) that seems fairly ready to use as a userland implementation. This means that we can implement provisional integration today.

Since this is a non-core packages that's versioned indepently from the core libraries, we can ship this integration early and manage breaking changes in the signals proposal with semver major releases.

### Goals
- Enable Lit elements to use signals in their update lifecycle methods, and trigger updates when the signals change.
- Work with standard signals produced by any wrapper library or framework (no proprietary extensions or hooks).
- Allow fine-grained DOM updates from individual signals.
- Integrate signal-driven updates with the ReactElement lifecycle.
- Give real-world, production-tested, feedback to the signals champions.

## Motivation

Signals are taking the web frontend world by storm. While they are but one approach to share observable state, they are an increasingly popular one right now.

Part of the excitement around signals is their ability to improve performance in frameworks that otherwise can have some poor update performance due to expensive VDOM diffs. Signals circumvent this issue by letting data updates skip the VDOM diff and directly induce an update on the DOM.

Lit doesn't have this problem. Instead of a VDOM diff against the whole DOM tree, Lit does inexpensive strict equality checks against previous binding values at dynamic binding sites only, and then only updates the DOM controlled by those bindings. This generally makes re-render performance be fast enough that signals aren't necessary for performance. In fact, one way to look at a Lit template is as a computed signal that depends on the host's reactive properties, and a Lit component as a signal that depends on the properties provided by it's parent.

Where signals can possibly be a major improvement for component authoring is as a shared observable state primitive. This may also have some performance benefits by allowing state updates via signals to bypass the top-down rendering of the component tree.

Lit doesn't have a built-in or endorsed shared *observable* state system. Properties can be passed down a component tree, and the `@lit/context` package allows sharing of values across a tree, but to observe changes to the individual data objects themselves, developers have to choose from a number of possible solutions, such as:

* State management libraries like Redux or MobX
* Observables like RxJS
* Building a custom system on the EventTarget API, or a custom callback.

Signals offer another option with a developer experience that is popular.

## Detailed Design

### User-facing APIs
The signals package will offer three ways to use signals:
- A class mixin that watches the entire update lifecycle and causes the whole element to re-render upon signal updates.
- A directive that applies a single signal to a binding
- A customized `html` template tag that automatically applies the directive to signal-values objects.

### Observing signal updates

To enable observing signal changes, we need to run access to a signal in an _effect_ - a closure that contains the signal access and will be called again when the accessed signals change.

Unlike most userland libraries, the standard signals proposal does not include an `effect()` helper (the intention is to leave effects and scheduling up to "frameworks"). Instead we must use a lower-level [_Watcher_ to watch signals and schedule updates](https://github.com/proposal-signals/proposal-signals/tree/main/packages/signal-polyfill).

We can use a Watcher in two ways:
1. To watch a computed signal that wraps the update lifecycle so that we're notified of updates to any signal read within the lifecycle and can trigger a new update.
2. To watch individual signals bound directly to the DOM with a `watch()` directive to update just that binding.

### SignalWatcher Mixin

Conceptually, we want to run the reactive update lifecycle in an effect so that the signal library observes access to signals and trigger a new update.

We can do this with an override of `performUpdate()` that wraps ReactiveElement's implementation in a watched computed signal:

```ts
abstract class SignalWatcher extends Base {
  // Watcher.watch() doesn't dedupe :(
  private __watching = false;
  private __watcher = new Signal.subtle.Watcher(() => {
    this.requestUpdate();
  });
  private __updateSignal = new Signal.Computed(() => {
    super.performUpdate();
  });

  override performUpdate() {
    if (this.isUpdatePending === false) {
      return;
    }
    this.__updateSignal.get();
  }

  override connectedCallback(): void {
    if (!this.__watching) {
      this.__watching = true;
      this.__watcher.watch(this.__updateSignal);
    }
    super.connectedCallback();
  }

  override disconnectedCallback(): void {
    if (this.__watching) {
      this.__watching = false;
      this.__watcher.unwatch(this.__updateSignal);
    }
    super.disconnectedCallback();
  }
}
```

### watch() directive

The `watch()` async directive accepts a signal and renders its value _asynchronously_ to the containing binding. When the signal changes, the binding value is updated directly.

Usage:
```ts
html`<p>${watch(messageSignal)}</p>`
```

We cannot write to the DOM synchronously like with `@lit-labs/preact-signals` because the standard signals proposal disallows reading signals from withing a watcher callback.

Instead the updates must be asynchronous, but they can still be fine-grained. In cases where we have multiple signals updates and/or a element update in a microtask we will synchronize fine-grained updates and the element's update lifecycle.

#### Static analysis of watch()

`watch()` essentially unwraps a `Signal<T>`. This should be analyzable by template analyzers like lit-analyzer, but we need to check and ensure this is the case.

### Auto-watching template tag

Auto-watching versions of `html` and `svg` template tags will scan a template result's values and automatically wrap them in a `watch()` directive if they are signals.

We should be able to detect signals with `value instanceof Signal`. While the `instanceof` operator is fragile, especially in the presence of multiple copies of a module or multiple realms, the signals proposal doesn't include a `Signal.isSignal()` helper yet.

It may sometimes be neccessary to forward a signal object through a binding, rather than watching it. To support this we will add a special object to box signals to indicate that they should be passed directly to the part. A `signalRef()` function will box the signal at at the binding site. This would only work with property bindings.

## Implementation Considerations

### Implementation Plan

Implementation should be straight forward. We'll create a new `@lit-labs/signals` package with the three APIs proposed here. There is nothing needed in core to support this.

#### lit-analyzer

After the library is launched, we need to check that the `watch()` directive is analyzed correctly and if not, fix it.

Currently, the auto-watching `html` tag will not be analyzed correctly. We should investigate if we could annotate a tag such that the analyzer knows that expressions may be wrapped, so that we don't have to hard-code support for this package.

### Backward Compatibility

No backward compatibility concerns.

### Testing Plan

Unit tests are sufficient for client-side rendering. We should also include server-side tests with the SSR fixture utility.

### Performance and Code Size Impact

No impact on core library size or performance.

### Interoperability

This RFC greatly improves on interoperability compared to the `@lit-labs/preact-signals` package.

### Security Impact

None

### Documentation Plan

This package will initially be documented in its own README. If it stays on track to graduation, we should document this package under the *Managing Data* section on lit.dev. We may end up with multiple signals packages, and either none will graduate, one will, or we'll have to document multiple packages.

## Downsides

None

## Alternatives

None
