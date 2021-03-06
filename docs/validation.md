## Form data validation

### Live validation

By default, form data are only validated when the form is submitted or when a new `formData` prop is passed to the `Form` component.

You can enable live form data validation by passing a `liveValidate` prop to the `Form` component, and set it to `true`. Then, everytime a value changes within the form data tree (eg. the user entering a character in a field), a validation operation is performed, and the validation results are reflected into the form state.

Be warned that this is an expensive strategy, with possibly strong impact on performances.

To disable validation entirely, you can set Form's `noValidate` prop to `true`.

### HTML5 Validation

By default, required field errors will cause the browser to display its standard HTML5 `required` attribute error messages and prevent form submission. If you would like to turn this off, you can set Form's `noHtml5Validate` prop to `true`, which will set `noValidate` on the `form` element.

### Custom validation

Form data is always validated against the JSON schema.

But it is possible to define your own custom validation rules. This is especially useful when the validation depends on several interdependent fields.

```js
function validate(formData, errors) {
  if (formData.pass1 !== formData.pass2) {
    errors.pass2.addError("Passwords don't match");
  }
  return errors;
}

const schema = {
  type: "object",
  properties: {
    pass1: {type: "string", minLength: 3},
    pass2: {type: "string", minLength: 3},
  }
};

render((
  <Form schema={schema}
        validate={validate} />
), document.getElementById("app"));
```

> Notes:
> - The `validate()` function must **always** return the `errors` object
>   received as second argument.
> - The `validate()` function is called **after** the JSON schema validation.

### Custom error messages

Validation error messages are provided by the JSON Schema validation by default. If you need to change these messages or make any other modifications to the errors from the JSON Schema validation, you can define a transform function that receives the list of JSON Schema errors and returns a new list.

```js
function transformErrors(errors) {
  return errors.map(error => {
    if (error.name === "pattern") {
      error.message = "Only digits are allowed"
    }
    return error;
  });
}

const schema = {
  type: "object",
  properties: {
    onlyNumbersString: {type: "string", pattern: "^\\d*$"},
  }
};

render((
  <Form schema={schema}
        transformErrors={transformErrors} />
), document.getElementById("app"));
```

> Notes:
> - The `transformErrors()` function must return the list of errors. Modifying the list in place without returning it will result in an error.

### Error List Display

To disable rendering of the error list at the top of the form, you can set the `showErrorList` prop to `false`. Doing so will still validate the form, but only the inline display will show.

```js
render((
  <Form schema={schema}
        showErrorList={false} />
), document.getElementById("app"));
```

> Note: you can also use your own [ErrorList](advanced-customization.md#error-list-template)

### The case of empty strings

When a text input is empty, the field in form data is set to `undefined`. String fields that use `enum` and a `select` widget will have an empty option at the top of the options list that when selected will result in the field being `undefined`.

One consequence of this is that if you have an empty string in your `enum` array, selecting that option in the `select` input will cause the field to be set to `undefined`, not an empty string.

If you want to have the field set to a default value when empty you can provide a `ui:emptyValue` field in the `uiSchema` object.
