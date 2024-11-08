---
Status: Active
Champions: @justinfagnani
PR: {{ update_with_pr_number }}
---

# Form Associated Mixin

A new labs package with helpers for simplifying the creation of form-associated custom elements.

## Objective

Make it easier to build well-behaved and complete form-associated custom
elements.

### Goals

- Reduced boilerplate needed for an element to participate in an HTML form,
  including:
  - Setting the formAssociated flag
  - Managing the form value and form state
  - Check and update element validity
  - Implementing disable, reset, and restore behavior
- Allow an element to have complete control over its public API.
- Optionally, allow an element ot opt-in to best-practice public APIs that match
  built-in form element APIs.

### Non-Goals

- The `FormControl` mixin does not try to emulate the API of a built-in input
  exactly, because native inputs are designed to allow for consumers to define
  custom validation. `FormControl` is instead designed for the element author
  to define validation.
- This RFC does not aim to cover all form-related element use cases, such as
  defining a form and managing form state. It it solely concerned with helpers
  for defining individual form-associated elements, which could be used with
  form management utilities.

## Motivation

Building a form-associated custom element requires a lot of common boilerplate:

- Setting `static formAssociated = true`
- Attaching `ElementInternals`
- Setting a default ARIA role
- Calling `internals.setFormValue()` when needed
- Calling `internals.setValidity()` and `internals.checkValidity()` when needed
- Handling `formDisabledCallback()`
- Handling `formResetCallback()` and `formStateRestoreCallback()`
- Requesting a reactive update when state held in internals changes.

This boilplate makes it much harder to implement a form-associated element.
Element authors should be able to focus on the actually necessary and unique
parts of their element:

- The element's value and state
- Validation logic

## Detailed Design

A new `@lit-labs/forms` package will contain several utilities:
- A `FormAssociated()` mixin that:
  - Makes an element form associated
  - Implements the standard form associated custom element callbacks
  - Provides a validation hook
  - Adds no public APIs to the element
- A set of decorators for designating fields as the form value and state
  - `@formValue()` for the public form value field
  - `@formState()` for the fields that compose part of the form state
  - `@formDefaultValue()` for the public form default value field
  - `@formDefaultState()` for the public form default state field
  - `@formStateGettter()` for private serializion of the form state
  - `@formStateSetter()` for private deserializion of the form state
- A `FormControll()` mixin that implements common public APIs for form controls,
  like `.form`, `.disabled`, `.validity`, etc.

### Why a mixin?

`FormAssociated` needs to be a mixin because we need to implement the form
associated callbacks on the element class. Controllers do not work for this use
case.

### Why decorators?

A key choice in this design is to not require or add any _public_ API of the implementing element. Rather than automatically adding `.value` or `.disabled`
fields, we leave that up to the element authors.

In order for the mixin to function, it needs to perform some actions and read
some element state when form-relate state changes. Decorators let us wrap
arbitrary element accessors to invoke the mixin code paths to update form state,
and let us communicate hooks to the mixin declaratively.

```ts
class MyFormElement extends FormAssociated(LitElement) {

  // Setting this field calls internals.setFormValue()
  @formValue()
  @property({reflect: true})
  accessor value: string = '';

  // When the form resets, the value is set to this field
  @formDefaultValue()
  accessor defaultValue: string = '';

  render() {
    return html`<input .value=${this.value}> @input=${this.#onInput}`;
  }

  #onInput(e) {
    this.value = e.target.value;
  }
}
```

Becauase decorators are used to denote the important form fields, name of the form value field does not need to be `value`. For instance, a checkbox element could
use the field `checked`:


```ts
class MyCheckbox extends FormAssociated(LitElement) {

  @formValue({
    converter: {
      toFormValue(value: boolean) {
        return String(value);
      },
      fromFormValue(value: string) {
        return Boolean(value);
      },
    })
  @property({type: Boolean, reflect: true})
  accessor checked: boolean = false;
}
```

_Note on this example: "boolean" has to be repeated three times here. We can't
deduplicate the TypeScript annotation and the type used by the `@property()`
converter, but we might be able to come up with a way for `@formValue()` to
automatically read the type from the `@property()` property options._

### The FormAssociated mixin

The `FormAssociated()` mixin implements most of the required logic for manging
the form-related state on `ElementInternals`.

#### formAssociated

The `FormAssociated()` mixin defines `static formAssociated = true`

#### ElementInternals

The `FormAssociated()` mixin needs access to `ElementInternals`, so it calls `this.attachInternals()` in the constructor.

Elements applying the mixin may also need access to internals, but
`attachInternals()` is only allowed to be called once so that outside callers
cannot access internals, so a `getInternals()` utility is provided to get them.

To discourage use of `getInternals()` by outside code, it must be called with a
reference to the class and can only be be called once per class.

#### Form Value

A form-associated element's form value must be passed to `internals.setValue()`
whenever the value is changed. This is implemented by the `@formValue()`
decorator, which wrapps an accesor's setter to call `setValue()`.

A form value must be of type `string | File | FormData | null`. To facilitiate
public value fields of different types (ie, numbers, dates, etc) `@formValue()`
accepts a `FormValueConverter` object with `toFormValue` and `fromFormValue`
callbacks.

The initial value of the `@formValue()` decorated field is retained for form
reset.

_TODO: we may only want to store the initial value if the class doesn't have a
`@formDefaultValue()` decorator._

#### Form State

Form state is similar to form value, but is not submitted with the form so it
can contain private, transient state, such as whether the user has edited the
value of the control.

Many element might not need to maintain separate form state. The main reason for
using state is to support form _restoring_. Restoring is when a form is changed
to a previous state, such as when using back- and forward- page navigation.

In a form restore callback the value passed may either be the previous form
_state_ or a new form _value_ depending on the mode. Because of this, state and
value should be derivable from each other.

If state is internal to an element, then the element author can designate a
setter and getter as accessors for state derived from value. The getter should
return an object derived from the value and any internal state, the setter
should accept a state and derive and set the value and any internal state.

This is done with the `@formStateGettter()` and `@formStateSetter()` decorators.

Fields that compose part of the internal state should be decorated with the
`@formState()` decorator so that `internals.setValue()` is called when the state
changes.

#### Disabled elements

The `FormAssociated()` mixin implements `formDisabledCallback()` to set the
`ariaDisabled` state and request a reactive update in case the element renders
differently when disabled.

#### Form reset

When a form is reset, form-associated elements should reset their value and
state to _default_ values.

Default values can usually be provided by the element consumer. ie, in a
particular form, one instance of a control might have a default value of `0` and
another instance of that control might have a default value of `100`, all
depending on the form definition.

The `FormAssociated()` mixin implements `formResetCallback()` with logic to
reset the value and state.

The default value and state can be designated by decorators, in one of two ways:

- Decorators: The `@defaultValue()` and `@defaultState()` decorators let the
  element author designate fields that hold the defaults. Public fields can be
  decoratorated to allow the element consumer to set the defaults.
- Initial fields values: If those default decorators aren't used, then the
  initial values of the `@formValue()` and `@formState()` decorated fields are
  used for defaults.

#### Form Restoration

The `FormAssociated()` mixin implements `formRestoreCallback()` with logic to
restore the value and state.

Restoration is a slightly complicated concept due to form state.
`formRestoreCallback()` has two modes 'restore' and 'autocomplete'. In 'restore'
mode `formRestoreCallback()` is called with a previous _state_, in
'autocomplete' mode it is called with a new _value_.

So we need form associated elements to be able to reset to a new value or state.
When a new value is given, the element should derive (or reset) the state. When
a new state is given, the element should derive the value. Elements implement
these with the `@formStateGetter()` and `@formStateSetter()` decorators.

These are placed on a getter and setter respectively. The `@formStateGetter()`
decorated getter is called whenever the element updates its form value (calls
`internals.setFormValue()`) to retrieve the current form state. The
`@formStateSetter()` decorated setter is called is form reset and restoration in
'restore' mode.

#### Validation

Any time the form value changes it should be validated against any custom
validation logic provided by the element author. To do this the
`FormAssociated()` mixin lets authors define a `_getValidity()` method that
returns a `ValidityResult` object.

`ValidityResult` is an interface that essentially captures the arguments to
`ElementInternals.setValidity()`.

The `FormAssociated()`  mixin implements a `_validate()` method that calls
`_getValidity()` and passes the result to `ElementInternals.setValidity()`.
`_validate()` is called whenever the form value changes, and can also by called
by the element - which is usually a good idea to do to initialize the element's
validitiy state (a required field may start out invalid).

### The FormControl mixin

The `FormControl()` mixin extends `FormAssociated()` to add opinionated and
best-practive public API that consumers likely expect from a form control,
including:

- A `form` getter
- A `disabled` accessor
- `validity`, `validationMessage`, and `willValidate` getters
- `checkValidity()`, and `reportValidity()`

Most of these simply delegate to the same ElementInternals APIs, except the
`disabled` accessor which reflects the `disabled` attribute.

#### Disabled

The actual disabled state of a form associated element is not a simple local
concept. It is determined by both the element's local disabled state and the
element's context. An element is disabled if it has the `disabled` attribute, or
if any of its ancestor `<fieldset>`s or its associated `<form>` is disabled.

It may seem tempting for an element's `disabled` getter to return true if the
element is disabled by its context, instead of just returning the element's
`disabled` attribute, but this would lead to inconsistencies where a user can
set `disabled` to `false`, then immediately read it and get `true`.

Native inputs also only reflect their `disabled` attribute to their `disabled` property.

The way to check whether an element is considered disabled within its form
context is to see if it matched the `:disabled` selector. To help document this
and make it more obvious, we will add the `isDisabled(el)` helper, which is
simply:

```ts
const isDisabled = (element: FormAssociated) => element.matches(':disabled');
```

Unfortunately, there isn't an easy way to be notified when an element is
disabled because one of its form ancestors is disabled. So if an element needs
to structurally rendering different based on disabled state, they will have to
use mutation observers to observe disabled attributes on ancestors. Hopefully,
most elements can use CSS to render disabled states, and check
`isDisabled(this)` in event listeners to ignore user interactions.

If there's demand, we could write a utility to try to detect disabled state
changes, and/or suggest a new browser event or callback for this purpose.

## Implementation Considerations

### Implementation Plan

The utilities outlined here can be implemented in a single, isolated PR.

### Backward Compatibility

This is a new package, so there are no backwards incompatibile changes
initially.

Eventually we will want to ship only standard decorators, so there will be a
future backwards incompatible change to this package.

### Testing Plan

Standard unit tests should be sufficient for most cases. We do not need a server
to capture form data, since we can use the `new FormData(form)` pattern. Form
reset can be tested with the `form.reset()` API.

Form restore is more difficult to test in client-side only tests. We may be able
to test the 'restore' mode with navigation in an iframe, but we likely can't
test 'autocomplete' mode without a page driver (Web Driver, Playwright, or
Puppeteer). We can just call `formRestoreCallback()` manually with the arguments
a browser should pass.

Form-associated custom elements have had some known browser bugs. While the APIs
used directly by these utilities seem likely to be cross-browser compatible,
some of pattterns or arguments passed to them by elements might have issues.

We might want to test for certain scenarios that have triggered browser bugs,
and possibly try to work around them in this library, but given the evolving
implementation status in browsers, it seems prudent for those issues to arrise
from real-world usage and feedback.

### Performance and Code Size Impact

The mixin is optional and not needed for most elements, but we should still ensure that it's as small as possible.

### Interoperability

No interoperability concerns.

### Security Impact

No security concerns.

### Documentation Plan

We will eventually need a section on forms on lit.dev.

## Downsides

The main downside to a form associated mixin would be adding opinionated public
API to elements. We have designed around this with two mixins, with one that
adds no public APIs.

One downside of needing to interact with `ElementInternal`s is that we need to
call `attachInternals()` and provide an alternate way to get internals.

## Alternatives

Reactive controllers were considered, but are not viable.
