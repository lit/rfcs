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
- get and set dirty, touched, valid states for individual controls
- allow for configuration (e. g. change on `blur` or `change`)
- It should be typed
- It should be possible to set initial values for individual controls. 
- subscribe to value changes and update dependant controls accordinly. 
- validate all controls of a form at the same time. 
- Allow for dynamic forms (e. g. arrays of controls etc.)

### Non-Goals
- The goal is not to provide a set of Validators 

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
The model for a form can be created similar to Angular Reactive Forms through a `FormBuilder` class. Every control must be initialized with an initial value. 



## Implementation Considerations

### Implementation Plan

Is there anything important to note about implementation plan? Can it be done in a single PR or will it need to be staged out across several?

### Backward Compatibility

As this is a new library, it should not. 

### Testing Plan

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

What impact will this proposal have on performance and code size? What benchmarks should we create to evaluate the proposal?

### Interoperability

Is this proposal for a feature that could be interoperable across web components not written in Lit? Does it create a tight coupling between components written in Lit? Could it be a [Web Components Community Group](https://github.com/w3c/webcomponents-cg) [Community Protocol](https://github.com/webcomponents-cg/community-protocols)?

### Security Impact

What impact will this proposal have on security? Does the proposal require a security review? (We have a security team available for reviews)

We especially care about the handling of untrusted user input by library code so that we contnue to prevent XSS vectors.

### Documentation Plan

Do we need to create or update any documentation to complete this proposal? Does related documentation have a clear home in our docs outline? What playground examples or tutorials should be created?

## Downsides

Many proposals involve trade-offs. What are they for this proposal and what are the downsides of this approach?

## Alternatives

What alternatives were considered and rejected? Why?
