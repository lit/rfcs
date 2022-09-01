---
Status: Active
Champions: christian24
PR: {{ update_with_pr_number }}
---

# @lit-labs/forms

Add advanced form validation for web components. This is a followup to the discussion in the lit repository: https://github.com/lit/lit/discussions/2489 

## Objective

Provide an easy to use dev experience for validating forms with web components in them. 

### Goals
- Provide a common API, so developers can quickly start using the @lit-labs/forms package without changing their existing controls. 
- Allow cross field validation
- Allow async validation
- Get and change state for individual controls
- Allow for configuration (e. g. when should changes be reflected in the model (when the element looses focus vs when the value of the element changes.))
- The API should be type safe. 
- It should be possible to set initial values for individual controls. 
- Subscribe to value changes of a control
- Subscribe to state changes of a control.
- Update the value of a control. 
- Organize controls into groups
- Validate or update groups 
- Form states should be serializable and can be used to update a form.
- Allow for dynamic forms (e. g. arrays of controls or multi step wizards etc.)

### Non-Goals
- The goal is not to provide a set of Validators. Validators should just be pure functions. They can be shared across projects, but it is not a goal to provide a library of common validators with this project.  

## Motivation

In enterprise applications forms with complex validations often play a huge part. In React and Angular a lot of easy to use solutions exist (Formik, React Hook Form and Angular Reactive Forms). With web components several alternatives exist as well (@vaadin/form or https://github.com/realiarthur/lite-form), but none have really gained traction. It would be awesome to have lightweight standard in the Lit community or hopefully even beyond.  

## Detailed Design

This is heavily influenced by Angular Reactive Forms. A prototype has been implemented by @UserGalileo here: https://github.com/UserGalileo/lit-reactive-forms

### General Overview
In general there are three types of form elements:

#### Form group: 
A form group is the most high level concept. It is a group of form controls. It can also hold form arrays. It combines values and state into single objects. Form groups can be nested. It allows to run validation on all children. State of the children can also be set.  

#### Form control: 
This is the most low level concept. This model holds the value and state of an individual control. Changes to the model should be reflected in the rendered control. Validation can be run on this. 

#### Form array: 
This allows dynamic lists to be part of a form. They act as aggregators similar to a form group, but new children can be added or deleted at runtime. Form array children can either be form controls or form groups. (Form array would also be possible as a child, but not sure this would make sense, it might be a quick win API wise though.)   

All of these should have listeners for changes to value and status, so that users can patch the values or status of dependant controls when necessary. 

### Validation

Validation should allow for async validators. All validators can be async by default Validators are pure functions.  There are two types of validators:

#### Form control level validator

This would be just a function declared as `(value) => string`. `value` being the current value of the control and the returned string being the potential error message describing the validation error. If an empty string is returned the control is valid. This is inspired by https://developer.mozilla.org/en-US/docs/Web/API/HTMLSelectElement/setCustomValidity

Validators should only be run for controls that are connected. 

#### Form group level validator

This validator runs on a form group and gets passed all values of the children and returns an object of error messages for the children. 

### How to define a form

#### Defining the model 
The model for a form can be created similar to Angular Reactive Forms through a `FormBuilder` class. Every control must be initialized with an initial value. The following example is taken from @UserGalilelo's prototype:

```js
fb = new FormBuilder(this);

form = this.fb.group({
    user: this.fb.group({
        name: this.fb.control(''),
        surname: this.fb.control(''),
    }),
    consent: this.fb.control(false)
}, {
    validators: [],
    asyncValidators: [],
});


// Binding (dotted syntax for nested FormGroups)
render() {
  const { bind } = this.form;
  
  return html`
    <input type="text" ${bind('user.name')}>
    <input type="text" ${bind('user.surname')}>
    <input type="text" ${bind('consent')}>
  `
}
```

The model and the elements are connected by use of a directive. There is an additional directive which simplifies rendering a form array, similar to `repeat` which takes an instance of form array as a parameter and as a second parameter takes a render function, which the directive uses to render an individual item of the form array. 

The `bind` directive subscribes to changes to the element via event listeners and updates the model accordingly. There is an abstraction called an accessor, which can be defined for web components specifying details like
- change event
- how to trigger an element's builtin validation (if exists)
- how to get and set the element's current value
- how to set an error message on the element (if available)
- how to change the element's state

Similarly, if the model is updated, the directive will update the element. 

## Implementation Considerations
So far the only feature that really is Lit-dependant is the use of directives. Maybe directives can be provided, but potentially an API along the lines of `formControl.registerElement(htmlElement)` can be provided as a substitute to support none Lit environments?

The prototype uses `rxjs` heavily. Is an external dependency a problem / show stopper?  

### Implementation Plan

Is there anything important to note about implementation plan? Can it be done in a single PR or will it need to be staged out across several?

### Backward Compatibility

As this is a new library, it should not. 

### Testing Plan

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

What impact will this proposal have on performance and code size? What benchmarks should we create to evaluate the proposal?

### Interoperability

This could potentially be implemented to be usable in any web component environment. 

### Security Impact

Since we only read and write value and states to existing element's this shouldn't have much of a security implication. 

### Documentation Plan

A pretty good README should be part of the npm package. 

## Downsides

This approach is quite oppinionated. It doesn't neccessarily live close to the web standards. It more tries to improve developer experience in enterprise contexts. 

## Alternatives

An approach similar to Formik, where the model is defined in the template and the validation is a simple function (that's not part of the model) was considered, but was after an initial proof of concept abandoned due to complexities involving nesting of controls, grouping controls together or having a dynamic list of form controls. 
