---
layout: post
title:  "My trip of using Formik + Yup"
date:   2018-11-06 10:30:00 -0500
categories:
tags: [react, es6]
---

* Build a React form with Formik + Yup.

By using [Formik](https://github.com/jaredpalmer/formik), you can save time from building React forms from scratch. Formik gives us a boilplate with a bunch of APIs. [Yup](https://github.com/jquense/yup) is a form validation library. What's look like when we use them to build a form? What's kind of issues I have experienced?


Here are some Tips
### 1. You need a smart component inside the Formik component to handle Formik events

Here is a Formik form

```javascript
<Formik
    validation={...}
    renders={props => (
        <form onSubmit={props.onSubmit}>
            <input type="text" name="firstName" />
            {props.touch.name && props.errors.name props.errors.name}
            ...
            ...
            <button type="submit">Submit</button>
        </form>
    )
/>
```

What to do if you want to automatically scroll the form up to the first field with an validation error when you click the Submit button?

You can use Web API [`Element.scrollIntoView`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView). Probably you want to do the scroll up inside `componentDidUpdate` so that the page can be scrolled till the React render is done.

Here is the [reference link of the solution](https://github.com/jaredpalmer/formik/issues/146)

```javascript

// formik-container.js

class FormikContainer extends Component {
  render() {
    return (
      <Formik
        validation={...}
        renders={props => (
          <Form {...props} />
        )}
      />
    )
  }
}

// form.js

class Form extends Component {
  componentDidUpdate(prevProps) {
    // Here is the scroll up function
    if (prevProps.isSubmitting && !this.props.isSubmitting && !isEmpty(this.errors)) {
      const errorElem = document.getElementsByClasses('your-error-class')[0];
      errorElem.scrollIntoView();
    }
  }

  render() {
    return (<form onSubmit={props.onSubmit}>
        <input type="text" name="firstName" />
        {props.touch.name && props.errors.name props.errors.name}
        ...
        ...
        <button type="submit">Submit</button>
      </form>)
  }
}

```

### 2. Making Http Api calls inside Yup schema is not recommended

When you use Yup for form validation in Formik, you need to define a Yup schema and call the schema in the `validationSchema` function. There is a validation schema example on [this page](https://jaredpalmer.com/formik/docs/guides/validation)

This is a Yup schema

```javascript
const SignupSchema = Yup.object().shape({
  firstName: Yup.string()
    .min(2, 'Too Short!')
    .max(50, 'Too Long!')
    .required('Required'),
  lastName: Yup.string()
    .min(2, 'Too Short!')
    .max(50, 'Too Long!')
    .required('Required'),
  email: Yup.string()
    .email('Invalid email')
    .required('Required'),
});
```

The validation function will be triggered on an input change event / a blur event / a form submit event, based on your configuration. Everytime the validation function is called, it will execute validation on all fields on the form. That means even you only change first name, it will run validation on last name and email as well.

Think about this

```javascript
const SignupSchema = Yup.object().shape({
  firstName: Yup.string()
    .min(2, 'Too Short!')
    .max(50, 'Too Long!')
    .required('Required'),
  lastName: Yup.string()
    .min(2, 'Too Short!')
    .max(50, 'Too Long!')
    .required('Required'),
  email: Yup.string()
    .checkDuplicatedEmail() // a customized validation function which makes api call from it
    .email('Invalid email')
    .required('Required'),
});

// See the Yup.addMethod doc at https://github.com/jquense/yup#yupaddmethodschematype-schema-name-string-method--schema-void
yup.addMethod(yup.string, 'checkDuplicatedEmail', => {
  return Yup.string().test('checkDuplicatedEmail', 'The email address has been taken', email => {
    // make an api call to the server
  });
});
```

The api call will be fired while user is typing by key strokes. Consider to cache the api call or separate the email validation from this schema.

### 3. Don't use self-controlled input components inside Formik

Think about this case

I have a self-controlled component

```js
// This component isself-controlled since it has its own state
  class FirstNameInput extends Component {
    constructor(props){
      ...
      this.state = { value: this.props.value };
    }

    onChange = (e) =>
      this.setState({ value: e.target.value })

    render() {
      return
      <div>
        <label>First Name</label>
        <input type="text" name="firstName" value={this.state.value} onChange={this.onChange} />
      </div>
    }
  }

// Formik has a state called values, which controls all values of its children inputs
  class FormikContainer extends Component {
    render() {
      copyFirstName = (e, setFieldValue) => {
        // set firstName by a default value
      }

      return (
        <Formik
          validation={...}
          renders={props => (
            <FirstNameInput value={props.values.firstName} />
            <LastNameInput />
            <input type="checkbox" name="copyFirstName" onChange={e => { this.copyFirstName(e, props.setFieldValue) }}>Use the same first name above</input>
            <button>Submit</button>
          )}
        />
      )
    }
  }
```

It looks good, right? What will happen if Formik wants to change the value of first name? e.g. automatically change the first name when users fill out other fields.

Since the `FirstNameInput` component controls its value by itself, when users click copy first name, users won't see any changes on the first name field.

There are a couple solutions

* Add a key prop to the `FirstNameInput`, e.g. `<FirstNameInput key={props.values.copyFirstName} />`
* (Recommend) Remove the state value from `FirstNameInput` component and make it a dump component

### 4. How to write async tests for Formik (in Mocka)?

When you want to write tests around the function of submitting a Formik form or change input values inside a form, you need to force the test suites to wait for a few million seconds after triggering a event. We need async js tests in order to not block the main test process.

```js
  it('shows an error when submit the form', async () => {
    submitTheForm();

    const promise = new Promise(resolve =>
      // create a recusive function to do a loop with an exit condition
      const timerId = setInterval(() => {
        if (!isEmpty(mountedForm.errors)) {
          // check the validation errors appear
          resolve(timerId);
        }
      }, 100)
    )

    promise.then(timerId => clearInterval(timerId))

    expect(mountedForm.text()).to.include('The first name is blank');
  })
```