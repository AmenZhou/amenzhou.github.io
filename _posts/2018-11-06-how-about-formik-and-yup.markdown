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

class FormikContainer extends React {
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

class Form extends React {
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

### 3. Don't use self-controlled input components inside Formik

### 4. How to write tests?