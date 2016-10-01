# react-validation

React component providing simple form validation. It uses the [Controlled Components](https://facebook.github.io/react/docs/forms.html#controlled-components) approach for validation.

It is not easy to validate forms with React. Due to the one-way data flow style, altering forms based on inputs in a straight-forward way becomes difficult.

React-validation provides several form elements wrapped as React components which are connected to a Form component.

### [DEMO](http://lesha-spr.github.io/react-validation/)

React-validation focuses on providing a validation library without coupling a model or anything similar. To access form-data we recommend FormData or [form-serialize](https://www.npmjs.com/package/form-serialize).

##### NOTE: Be aware to always pass ```name``` prop. It is required.

Additional markup is allowed inside the Validation.Form markup.

Additional props (such as event handlers) may also be passed to components.

If you find any bug or error, please feel free to raise an issue. Pull requests are also welcome.

## Installation

``
npm install react-validation
``

## Example usage

With @2.x, react-validation is no longer dependent on ```validator```. You may use whatever validation strategy you want by extending the ```Validation.rules``` object.
Let's take a look at it's initial state:

```javascript
module.exports = {};
```

That's it, just an empty object literal. We don't have any validation rules out of the box because of an extremely high number of possibilities, but it's still recommended to use a well tested library.

First, let's extend it and add some rules:

```javascript
import React from 'react';
import Validation from 'react-validation';
import validator from 'validator';

// Use Object.assign or any similar API to merge your validation rules
Object.assign(Validation.rules, {
    // Key name maps the rule
    required: {
        // Function to validate value
        rule: (value, component, form) => {
            return value.trim();
        },
        // Function to return hint
        // You may inject the current value into the hint
        hint: value => {
            return <span className='form-error is-visible'>Required</span>
        }
    },
    email: {
        // Example usage with external 'validator'
        rule: value => {
            return validator.isEmail(value);
        },
        hint: value => {
            return <span className='form-error is-visible'>{value} is not an Email.</span>
        }
    },
    // This example shows a way to handle common task - compare two fields for equality
    password: {
        // the rule function can accept 2 extra arguments:
        // component - current checked component
        // form - form component which has 'states' inside native 'state' object
        rule: (value, component, form) => {
            // form.state.states[name] - name of corresponding field
            let password = form.state.states.password;
            let passwordConfirm = form.state.states.passwordConfirm;
            // isUsed, isChanged - public properties
            let isBothUsed = password && passwordConfirm && password.isUsed && passwordConfirm.isUsed;
            let isBothChanged = isBothUsed && password.isChanged && passwordConfirm.isChanged;

            if (!isBothUsed || !isBothChanged) {
                return true;
            }

            return password.value === passwordConfirm.value;
        },
        hint: value => {
            return <span className='form-error is-visible'>Passwords should be equal.</span>
        }
    }
});
```

We've added ```required``` and ```email``` to our rules with provided hints. This might be a separate file where all the rules are registered.

That's it. We can now use it in our react components:

```javascript
import Validation from 'react-validation';
import React, {Component, PropTypes} from 'react';
import rules from './rules';

export default class Registration extends Component {
    render() {
        return <Validation.components.Form>
            <h3>Registration</h3>
            <div>
                <label>
                    Email*
                    <Validation.components.Input value='email@email.com' name='email' validations={['required', 'email']}/>
                </label>
            </div>
            <div>
                <label>
                    Password*
                    <Validation.components.Input type='password' value='' name='password' validations={['required']}/>
                </label>
            </div>
            <div>
                <Validation.components.Button>Submit</Validation.components.Button>
            </div>
        </Validation.components.Form>;
    }
}
```

Note the ```validations``` prop. It's an array of strings which maps to the rules keys we've extended.

## Components and props

```react-validation``` provides the following components within the ```components``` object:
```Form```
```Input```
```Textarea```
```Select```
```Button```

All of them are just custom wrappers around the native components. They accept any valid attributes and a few extra:

|                    | Input | Textarea | Select | Button |                                                                                |
|--------------------|-------|----------|--------|--------|--------------------------------------------------------------------------------|
| validations        | X     | X        | X      |        | Accepts an array of validations strings which refer to the rules object's keys |
| containerClassName | X     | X        | X      |        | Adds a ```className``` to the wrapper around the native component              |
| errorClassName     | X     | X        | X      | X      | Adds a ```className``` on error occur   

##### NOTE: Be aware to always provide a ```name``` prop to ```Input```, ```Select``` and ```Textarea```

### Form component

```
Validation.components.Form
```

At the heart of React-validation is the Form component. It basically clones all of it's children and mixes the binding between the form and the child components.

#### Props

onSubmit
method

#### Public methods

```Form``` provides three public methods:

```validate(name)``` - validates input with passed name. The difference between this method and default validation is that ```validate``` marks input as ```isUsed``` and ```isChanged```.

```name``` - name of corresponding component.

```showError(name [,hint])``` - helps to handle async API errors.

```hint``` - optional hint to show.

```hideError(name)``` - hides a corresponding component's error.

```validateAll()``` - validates all react-validation components. Returns map (key: field name prop, value: non passed validation rule) of invalid fields.


```javascript
export default class Comment extends Component {
    handleSubmit(event) {
        event.preventDefault();

        // Emulate async API call
        setTimeout(() => {
            this.refs.form.showError('username', <span onClick={this.removeApiError.bind(this)} className='form-error is-visible'>API Error. Click to hide out.</span>);
        }, 1000);
    }

    removeApiError() {
        this.refs.form.hideError('username');
    }

    render() {
        return <Validation.components.Form ref='form' onSubmit={this.handleSubmit.bind(this)}>
            <div>
                <label>
                    <Validation.components.Input value='Username' name='username' validations={['required', 'alpha']}/>
                </label>
            </div>
            <div>
                <label>
                    <Validation.components.Textarea value='Comment' name='comment' validations={['required']}/>
                </label>
            </div>
            <div>
                <Validation.components.Button>Submit</Validation.components.Button>
            </div>
        </Validation.components.Form>
    }
}
```

### Input component

```
Validation.components.Input
```

The wrapper around the native ```input```. It accepts a ```validations``` prop - array of strings which refers to rules object keys.

```javascript
<Validation.components.Input name='firstname' validations={['alpha', 'lt8']}/>
```

##### NOTE: For types ```radio``` and ```checkbox```, react-validation will drop the ```value``` to an empty string when it's not checked. This is to avoid validation of non checked inputs.

Both the rules provided to react-validation will fail validation with the first error that occurs. (lt8 - value length less than 8)
In the example above for ```really long value with d1g1t``` input's value the ```alpha``` rule will break validation first. We can control this by ordering rules within the ```validations``` array.

### Textarea component

```
Validation.components.Teaxtarea
```

The wrapper around the native ```textarea```. Like the ```Input``` component, it accepts ```validations``` as a prop. Nothing special here:

```javascript
<Validation.components.Textarea name='comment' value='' validations={['required']}/>
```

### Select component

```
Validation.components.Select
```

The wrapper around the native ```select```. Like the ```Input``` component, it accepts ```validations``` as a prop. Nothing special here:

```javascript
<Validation.components.Select name='city' value='' validations={['required']}>
    <option value=''>Choose your city</option>
    <option value='1'>London</option>
    <option value='2'>Kyiv</option>
    <option value='3'>New York</option>
</Validation.components.Select>
```

### Button component

```
Validation.components.Button
```

The wrapper around the native ```button```. React-validation disables the button (adds ```disabled``` prop) when an error occurs. This behavior may be suppressed by passing the ```disabled``` prop to the component.

## Migration from 1.*

#### extendErrors API
```extendErrors``` no longer exists. Replace it with new approach of validation rules registration. ```hint``` appearance is now fully controlled:

```javascript
Object.assign(Validation.rules, {
    required: {
        rule: value => {
            return value.trim();
        },
        hint: value => {
            return <span className='form-error is-visible'>Required</span>
        }
    }
});
```

#### Defaults
React-validation no longer has any defaults. This is TBD, but for a 2.0.0 please provide ```errorClassName``` and ```containerClassName``` directly to the validation components.

#### Validations
```validations``` prop now accepts array of strings instead of objects. It's made to be more simple and reduce ```render``` code.

#### Components
```Validation.components``` sub-object now provides access to the Components.

Components API moved to Form API. ```forceValidate``` method is no longer exist.
