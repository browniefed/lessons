## Intro

When dealing with forms there comes a time when you need multiple pages. With this comes state across pages, validation and preventing users from destroying the data they entered. We won't tackle validation in this article, but we'll walk through a simple setup of an onboarding form, and touch on life cycle methods with React Router, and finally how to prevent users from destroying their data.

![](https://images.codedaily.io/lessons/reactRouter/formWizardCode.png)

## Setup

There is a bit too much to walk through in a simple setup but lets look at our folder structure and how that translates to routing.

![](https://images.codedaily.io/lessons/reactRouter/formWizardFolderStructure.png)

Our `index.js` is treated as our master App file. It is the wrapping `BrowserRouter` 2 top level routes, our `/` home route, and our `/form` route that will render our form wizard.

```js
const routes = (
  <BrowserRouter>
    <div>
      <div className="links">
        <Link to="/" className="link">
          Home
        </Link>
        <Link to="/form" className="link">
          Wizard
        </Link>
      </div>
      <div className="tabs">
        <Switch>
          <Route path="/" exact component={Home} />
          <Route path="/form" component={Form} />
        </Switch>
      </div>
    </div>
  </BrowserRouter>
);
```

The rest of the forms will be rendered inside of the `Form` component, and additionally the state of the form will live in there as well.

## The Wizard Form

The one thing we need to take care of is to ensure that even when our route changes at the `/form` endpoint that our URL matches, and our same `WizardForm` renders again. This is where all of our state will live. It will actually live inside of `Formik` but as long as this same `WizardForm` component renders the state will be preserved.

```js
import React, { Component } from "react";
import { Switch, Route, Prompt, Redirect, matchPath } from "react-router-dom";
import { Formik, Form } from "formik";

import BasicPage from "./form/basic";
import LocationPage from "./form/location";
import SubmitPage from "./form/submit";

class WizardForm extends Component {
  render() {
    return (
      <div>
        <Formik
          initialValues={{
            email: "",
            firstName: "",
            lastName: "",
            address: "",
            city: "",
            state: "",
            zipCode: "",
            tos: false,
          }}
        >
          <Form>
            <Switch>
              <Redirect from="/form" exact to="/form/basic" />
              <Route path="/form/basic" component={BasicPage} />
              <Route path="/form/location" component={LocationPage} />
              <Route path="/form/submit" component={SubmitPage} />
            </Switch>
          </Form>
        </Formik>
      </div>
    );
  }
}

export default WizardForm;
```

This is why we need to have our route setup based off of the `/form`. You can see here that we have `/form/basic`, `/form/location`, `/form/submit`.

```js
<Switch>
  <Redirect from="/form" exact to="/form/basic" />
  <Route path="/form/basic" component={BasicPage} />
  <Route path="/form/location" component={LocationPage} />
  <Route path="/form/submit" component={SubmitPage} />
</Switch>
```

Here we setup a `Redirect` so that if anyone visits just `/form` we change the URL to `/form/basic` to match our first page which is our `BasicPage`.

So whenever our URL changes out hierarchy of components will be the same.

So for example at `/form/basic` it would look like

```
index
- <WizardForm />
-- <BasicPage />
```

Then when we change to `/form/location` it would look like

```
index
- <WizardForm />
-- <LocationPage />
```

So with the way React and React Router works it's going to see the same `WizardForm`. It's going to reuse it and the form state inside of `Formik` will be preserved. Then it's going to see a new component down below and render something new.

Thankfully though we have all the state for the form up a level and preserved. So if the user presses the back button we'll render the `BasicPage` and the form will render and we won't have lost their data.

## Form Page

I'm not going to post them all but this is what a form would look like

```js
import React, { Component } from "react";
import { Link } from "react-router-dom";
import { Field } from "formik";

class LocationPage extends Component {
  render() {
    return (
      <div>
        <div>
          <Field type="text" name="address" placeholder="Address" />
        </div>
        <div>
          <Field type="text" name="city" placeholder="City" />
        </div>
        <div>
          <Field type="text" name="state" placeholder="State" />
        </div>
        <div>
          <Field type="text" name="zipCode" placeholder="Zip Code" />
        </div>
        <Link to="/form/submit" className="next">
          Next
        </Link>
      </div>
    );
  }
}

export default LocationPage;
```

Nothing too crazy, we use `Field` from `Formik` that will read the data of the form from `context` and then additionally just use `Link` to navigate between forms.

## Add Prompt

Now we have a wizard that our users can edit, click next and navigate through to the end and submit. What if they accidentally click the `Home` link, or another route. They're going to lose their hard entered data and you might lose a user.

What we can do is add in a `Prompt`, however our prompt is going to alert every single time we change the URL even when navigating between `/form/basic` to `/form/location`.

```js
render() {
    return (
      <div>
        <Prompt
          when={true}
          message={"Are you sure you want to navigate away?"}
        />
        <Formik
          initialValues={{
            email: "",
            firstName: "",
            lastName: "",
            address: "",
            city: "",
            state: "",
            zipCode: "",
            tos: false,
          }}
          onSubmit={this.handleSubmit}
        >
          <Form>
            <Switch>
              <Redirect from="/form" exact to="/form/basic" />
              <Route path="/form/basic" component={BasicPage} />
              <Route path="/form/location" component={LocationPage} />
              <Route path="/form/submit" component={SubmitPage} />
            </Switch>
          </Form>
        </Formik>
      </div>
    );
  }
```

## Use matchPath

We need to use 2 features of React Router. The first is `matchPath`. It's what the `Route` component uses internally. Then second is the the `message` of `Prompt` can take a function. If it returns a string it will prompt the user if they'd like to navigate away from the page. If it receives `false` then it rejects the navigation and doesn't prompt. However if it receives `true` then it will allow navigation without a prompt.

It also receives the URL of the route the user is navigating too. So we can use `matchPath` to attempt to match the `pathname` of where the user is headed and see if it matches `/form`. If `/form` is the base we know that our `WizardForm` component will keep rendering and thus no user form data is going to be destroyed.

```js
<Prompt
  when={true}
  message={({ pathname }) => {
    return matchPath(pathname, { path: "/form" })
      ? true
      : "Are you sure you want to navigate away?";
  }}
/>
```
The `matchPath` call takes the `pathname` to examine, and then an object to match against. Here we just pass in that we are looking for `path: "/form"`.

## Turning off Prompt

However when we submit we want to push them to another page. Our `Prompt` is going to check that they're going to another page and throw up a prompt even though the user already submitted their form. We don't want that.

To combat this we setup some state to track whether or not the form has been successfully submitted. Now because we aren't doing anything and things aren't async, we need to use the componentDidUpdate callback function of `setState` before triggering our navigation to the home page.

```js
class WizardForm extends Component {
  state = {
    submitted: false,
  };
  handleSubmit = () => {
    this.setState(
      {
        submitted: true,
      },
      () => this.props.history.push("/"),
    );
  };
  render() {
  }
}
```

Now we fix our `when` prop of `Prompt` which controls whether or not `Prompt` is active or not. So it will be active when we haven't submitted, but once we have it gets disabled.

```js
<Prompt
  when={!this.state.submitted}
  message={({ pathname }) => {
    return matchPath(pathname, { path: "/form" })
      ? true
      : "Are you sure you want to navigate away?";
  }}
/>
```

## Ending

There are a lot of cases where you may want to validate and check `when` but I only covered a very simple case of `submitted`. The key to all of this is that our `WizardForm` rendering at `/form` will manage our state and live through route changes, then our child routes can remain stateless and be destroyed each route change. They'll simply rehydrate between routes from the state that `Formik` holds onto in our `WizardForm`.

![](https://images.codedaily.io/lessons/reactRouter/formWizard.gif)
