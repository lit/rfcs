---
Status: Active
Champions: "@justinfagnani @rictic"
PR: https://github.com/lit/rfcs/pull/19
---

# Preact Signals

A new labs package with Preact signals integration for Lit.

## Objective

We want to enable use of signal libraries with Lit, as an option for observable shared state. There are many signals libraries out there, and they are not generically compatible, so for now Lit would need integration with each library developers want to use. This RFC proposes [Preact Signals](https://preactjs.com/guide/v10/signals/) as one such library that we should provide integration for.

### Goals
- Enable Lit elements to use signals in their update lifecycle methods, and trigger updates when the signals change.

### Non-Goals
- Build a Lit-specific signal library

## Motivation

Signals are taking the web frontend world by storm. While they are but one approach to share observable state, they are an increasingly popular one right now.

Part of the excitement around signals is their ability to improve performance in frameworks that otherwise can have some poor update performance due to expensive VDOM diffs. Signals circumvent this issue by letting data updates skip the VDOM diff and directly induce an update on the DOM.

Lit doesn't have this problem. Instead of a VDOM diff against the whole DOM tree, Lit does inexpensive strict equality checks against previous binding values at dynamic binding sites only, and then only updates the DOM controlled by those bindings. This generally makes re-render performance be fast enough that signals aren't necessary for performance. In fact, one way to look at a Lit template is as a computed signal that depends on the host's reactive properties, and a Lit component as a signal that depends on the properties provided by it's parent.

Where signals can possibly be a major improvement for component authoring is as a shared observable state primitive. This may also have some performance benefits by allowing state updates via signals to bypass the top-down rendering of the component tree.

Lit doesn't have a built-in or endorsed shared *observable* state system. Properties can be passed down a component tree, and the `@lit-labs/context` package allows sharing of values across a tree, but to observe changes to the individual data objects themselves, developers have to choose from a number of possible solutions, such as:

* State management libraries like Redux or MobX
* Observables like RxJS
* Building a custom system on the EventTarget API, or a custom callback.

Signals offer another option with a developer experience that is popular.

## Detailed Design

To enable observing signal changes, we need to run access to a signal in an _effect_ - a closure that contains the signal access and will be called again when the accessed signals change.

Preact signals exports the `effect()` method for this:

```ts
import {effect} from '@preact/signals-core';

export const logSignal = (s: Signal<unknown>) =>
  effect(() => {
    // Run when someSignal changes
    console.log(s.value);
  });
```

We intend to offer three ways to automatically run signal access inside an effect:
- A class mixin that runs the entire update lifecycle in an effect and causes the whole element to re-render upon changes.
- A directive that applies a single signal to a binding
- A customized `html` template tag that automatically applies the directive to signal-values objects.

### SignalWatcher Mixin

Conceptually, we want to run the reactive update lifecycle in an effect so that the signal library observes access to signals and trigger a new update.

We can do this with an override of `performUpdate()` that wraps ReactiveElement's implementation in an effect:

```ts
    private _disposeEffect?: () => void;

    override performUpdate() {
      // ReactiveElement.performUpdate() also does this check, so we want to
      // also bail early so we don't erroneously appear to not depend on any
      // signals.
      if (!this.isUpdatePending) {
        return;
      }
      // If we have a previous effect, dispose it
      this._disposeEffect?.();

      // We create a new effect to capture all signal access within the
      // performUpdate phase (update, render, updated, etc) of the element.
      this._disposeEffect = effect(() => {
        // When Signals change we need to re-render, but we need to get past
        // the isUpdatePending in performUpdate(), so we set it to true.
        this.isUpdatePending = true;
        // We call super.performUpdate() so that we don't create a new effect
        // only as the result of the effect running.
        super.performUpdate();
      });
    }
```

An issue with this approach is that we bypass the reactive update lifecycle when signals change. We would like to integrate updates from signals and reactive properties into one batch. To do this we need separate watch and update code paths. Preact Signals don't have such an API, but we should be able to simulate it:

```ts
    override performUpdate() {
      if (!this.isUpdatePending) {
        return;
      }
      this._disposeEffect?.();
      let updateFromLit = true;
      this._disposeEffect = effect(() => {
        if (updateFromLit) {
          super.performUpdate();
        } else {
          // This branch is an effect run from Preact signals.
          // This will cause another call into performUpdate, which will
          // then create a new effect watching that update pass.
          this.requestUpdate();
        }
        updateFromLit = false;
      });
    }
```

### watch() directive

The `watch()` async directive accepts a signal and renders its value synchronously to the containing binding. When the signal changes, the binding value is updated directly.

Usage:
```ts
html`<p>${watch(messageSignal)}</p>`
```

```ts
class WatchSignal extends AsyncDirective {
  render<T>(signal: Signal<T>): T {
    let updateFromLit = true;
    effect(() => {
      // Access the signal unconditionally to watch it
      const value = signal.value;
      if (!updateFromLit) {
        this.setValue(value);
      }
      updateFromLit = false;
    });
    return signal.peek();
  }

  // disconnected / reconnected too
}
export const watch = directive(WatchSignal);
```

#### Static analysis of watch()

`watch()` essentially unwraps a `Signal<T>`. This should be analyzable by template analyzers like lit-analyzer, but we need to check and ensure this is the case.

### Auto-watching template tag

Auto-watching versions of `html` and `svg` template tags will scan a template result's values and automatically wrap them in a `watch()` directive if they are signals.

We should be able to detect signals with `value instanceof Signal`. While the `instanceof` operator is fragile, especially in the presence of multiple copies of a module, it appears that the Preact Signals modules are _already_ fragile in this respect due to module-scoped state: all signals, computed signals, and effects, must use the same module instance to work. We have filed an [issue for a more robust check](https://github.com/preactjs/signals/issues/402) and will revist this topic when that issue is addressed.

## Implementation Considerations

### Implementation Plan

Implementation should be straight forward. We'll create a new `@lit/labs/preact-signals` package with the three APIs proposed here. There is nothing needed in core to support this.

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

Signals implementations are unfortunately not interoperable with each other. Signals are most appropriate for the internal state of a component, or share state amongst already tightly-coupled components, as in an app.

### Security Impact

None

### Documentation Plan

This package will initially be documented in its own README. If it stays on track to graduation, we should document this package under the *Managing Data* section on lit.dev. We may end up with multiple signals packages, and either none will graduate, one will, or we'll have to document multiple packages.

## Downsides

Implementing this RFC may appear as endorsing Preact Signals above other signals packages. This will be true to the extent that we will have built and tested the integration ourselves, so know it works. The downside is that other signals implementations may be as good or better choices, and our intention of Preact Signals integration only being the first integration could be overlooked causing people to not use the other packages. Something has to go first however.

## Alternatives

We could leave it to the community to build an integration. The Lit team is most familiar with the nuances of ReactiveElement update lifecycle, so we can build this quickly.
