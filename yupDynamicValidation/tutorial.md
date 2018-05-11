## Intro

[Yup](https://github.com/jquense/yup) is a JavaScript object schema validator and object parser inspired by Joi ( a validator for node).
It has many powerful features like async validation, custom rules, stacking conditions based on other values through out your data, and dynamic run-time schema validation.

We'll be talking about run-time schema validation. Essentially this lets you define validation rules on the fly when you are attempting to validate your data. This will let you implement all the crazy validation logic you'll ever need for your forms!

## Scenario

I encountered a scenario where sections of a form I was making were optional if omitted but required if they existed.

Essentially the data payload could be and empty object

```js
{}
```

or

```js
{
  data1: {
    name: "",
    items: [""]
  },
  data2: {
    orgName: "",
  },
  data3: undefined
}
```

Where `data1`, or `data2`, or `data3` could be defined as objects or just completely `undefined`. This made validation tricky as it required knowledge of the value at the given key location before you knew whether you needed to validate it. There are other intricacies that could exist as well. With more information certain sub-trees of data may or may not be required based on previous answers.

Overall I needed a quick way to use a value to determine whether or not data at a key needed validation.


## Quick Setup

So we're using React because why not? Yup can be used with any front end framework.
We'll setup a basic `yup.object()` and run `isValidSync()` and pass in our `this.state`.

The `yup.object` doesn't have any schema validations so `valid` is going to be true.

```js
import React, { Component } from "react";
import yup from "yup";

const optionalRequiredSchema = yup.object();

class App extends Component {
  state = {};
  render() {
    const valid = optionalRequiredSchema.isValidSync(this.state);

    return (
      <div className="container">
        <div className="text" style={{ backgroundColor: valid ? "green" : "red" }}>
          Valid Data: {"" + valid}
        </div>
        <button>Add Optional</button>
      </div>
    );
  }
}
```

The `isValidSync` will only return `true` or `false`. To get the actual errors you'll need to use the `validateSync` in combination with a `try/catch`.

## Defining Validation Schema

In order to define validation rules we define a validation schema. This is a series of `yup` calls to build out an entire schema that will then be passed the data to validate.

Our first step is to define our outer wrapper. In our case because the wrapper is an `object` with nested data we use the `yup.object()` call and then tell `yup` we are going to define the `shape` of the data below.

```js
const optionalRequiredSchema = yup.object().shape({
    
});
```

Next we define a `key` on this shape called `optionalObject`. This will then match up with any data like so `{ optionalObject: {} }`.
We then define another `yup.object().shape()`. If we just did `yup.object()` it would just be looking for an object with no data.

```js
const optionalRequiredSchema = yup.object().shape({
  optionalObject: yup.object().shape({

  }),
});
```

Finally we define our required data. We'll just call it `otherData` and indicate that it's a required string.

```js
const optionalRequiredSchema = yup.object().shape({
  optionalObject: yup.object().shape({
    otherData: yup.string().required(),
  }),
});
```

If we were to run this against our data it would not be valid.

## Make it Optional

If we passed in our data that is just an empty object it would attempt to validate and tell us that we `otherData` string is required.

```js
optionalRequiredSchema.isValidSync({});
```

Well what if we wanted this to be valid? Then if we passed in `optionalObject` actually defined then we would trigger our validation. This sort of strategy can make optional form validation easier since we can just add an `{}` or set whole subtrees to `undefined`.

```js
optionalRequiredSchema.isValidSync({
  optionalObject: {
    otherData: ""
  }
});
```

To make this work we use the `yup.lazy()` function. It will get passed the value of the key you've defined it on. When the `yup.lazy()` is run you are guaranteed the value to exist.

```js
optionalObject: yup.lazy(value => {

})
```

We must always return at least some validation rule. So first off if `value !== undefined` then we'll return our previous validation schema. 
If it is `undefined` then we'll use the `yup.mixed().notRequired()` which will just inform `yup` that nothing is required at the `optionalObject` level.

```js
  optionalObject: yup.lazy(value => {
    if (value !== undefined) {
      return yup.object().shape({
        otherData: yup.string().required(),
      });
    }
    return yup.mixed().notRequired();
  }),
```

## Wire it Up

Now if we just have our `button` add in our `optionalObject` subtree. We can 

```js
 <button onClick={() => this.setState({ optionalObject: {} })}>Add Optional</button>
```

Once toggled our validation re-runs, and we now see that `optionalObject` is set thus our validation will be returned from within the `if` statement of `yup.lazy()` and we'll now have a required `otherData` string.

![](http://images.codedaily.io/lessons/yupDynamicValidationToggle.gif)


## Final

[Yup](https://github.com/jquense/yup) is a very powerful validation library with a ton of features. It can be used for the simplest of validations and accommodate anything you need to get done including async validation.