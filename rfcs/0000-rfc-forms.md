---
Status: Active
Champions: christian24
PR: {{ update_with_pr_number }}
---

# @lit-labs/forms

Add advanced form validation for web components. This is a followup to the discussion in the lit repository: https://github.com/lit/lit/discussions/2489 

## Objective

Provide an easy-to-use dev experience for validating forms with web components in them. 

### Goals
- Provide a common API, so developers can quickly start using the @lit-labs/forms package without changing their existing controls. 
- Allow cross field validation
- Allow async validation
- Get and change state for individual controls
- Allow for configuration (e.g. when should changes be reflected in the model (when the element looses focus vs when the value of the element changes.))
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
- Build an entire form solution from scratch. The Lit team has voiced no interest in doing this. 

## Motivation

In enterprise applications forms with complex validations often play a huge part. In React and Angular a lot of easy to use solutions exist (Formik, React Hook Form and Angular Reactive Forms). With web components several alternatives exist as well (@vaadin/form or https://github.com/realiarthur/lite-form), but none have really gained traction. It would be awesome to have lightweight standard in the Lit community or hopefully even beyond.  

## Detailed Design

The idea is to wrap an existing form library. 

### General Overview for forms
In general there are three types of form elements:

#### Form group: 
A form group is the highest level concept. It is a group of form controls. It can also hold form arrays. It combines values and state into single objects. Form groups can be nested. It allows to run validation on all children. State of the children can also be set.  

#### Form control: 
This is the lowest level concept. This model holds the value and state of an individual control. Changes to the model should be reflected in the rendered control. Validation can be run on this. 

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

There are multiple competing API suggestions:
#### Separation of template and form configuration
```ts
const form = new FormApi(...);

// Validation logic
const fieldA = new FieldApi({
  form,
  ...whateverElse
});

const fieldB = new FieldApi({
  form,
  ...whateverElse
});

return html`
  <form @submit=${() => form.handleSubmit()}>
    <!-- Templating logic -->
    <label for=${fieldA.name}>Field A</label>
    <input name=${fieldA.name}>

    <label for=${fieldB.name}>Field B</label>
    <textarea name=${fieldB.name}></textarea>
  </form>
`;
```

#### Field configuration inside the template
This is closer to how other Tanstack Form adapters currently work:
```ts
const form = new FormApi(...);

return html`
  <form @submit=${() => form.handleSubmit())}>
    <!-- Templating logic AND validation logic in one place -->
    ${form.Field({onChange: z.string().min(1), render: (fieldA) => html`
    <label for=${fieldA.name}>Field A</label>
    <input name=${fieldA.name}>
`})}
  </form>
`;
```

### bind directive 
To make syncing form elements and the form model easier, the idea is to create a `bind`directive that automatically connects the two:

```js
// field is the instance of the form field
<input type="text" ${bind(field)}>
```

The model and the elements are connected by use of the directive. 

The `bind` directive subscribes to changes to the element via event listeners and updates the model accordingly. There is an abstraction called an accessor, which can be defined for web components specifying details like
- change event
- how to trigger an element's builtin validation (if exists)
- how to get and set the element's current value
- how to set an error message on the element (if available)
- how to change the element's state

Similarly, if the model is updated, the directive will update the element. 

An additional idea is that the directive could be cross-form-wrapper: An interface how to represent a model could be defined that works across multiple form-integrations. 

### Potential field array directive
Potentially an additional directive could be created which simplifies rendering a form array, similar to `repeat` directive. It would take the field instance that is an array as a parameter. As a second parameter it would take a render function, which the directive uses to render an individual item of the form array.
## Implementation Considerations
A form library to base the wrapper on needs to be decided. So far POCs for `final-form` and `@tanstack/form-core` have been created.

With `final-form`a lack of type safety of the defined form was apparent.

`@tanstack/form-core` is still in heavy development with a prospective version 1.0 release in December or early next year.

If the API `Field configuration inside the template` is chosen the wrapper could be an official Tanstack Form adapter that would live in the Tanstack Form repo. If the other API is chosen they adapter would have to live in the `Lit`-repo.

Other form solutions can be suggested in this RFC. 


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

A wrapper probably does mean that some compromises about implementation or developer experience need to be made.

## Alternatives

A previous version of this RFC explored building an entirely lit-focused library from scratch. Due to missing interest and expertise this was ultimately abandoned. 

