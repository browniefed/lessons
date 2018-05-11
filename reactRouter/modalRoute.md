## Intro

A common pattern on Facebook and other applications is to show pictures in a gallery, this gallery may be an interactive modal. However if the user visits the page from just a URL you might want to display the gallery outside of the modal. React Router makes this simpler by allowing you to pass in a state object when routing.

![](https://images.codedaily.io/lessons/reactRouter/modalView.gif)

## Create Routes

First off lets setup our routes, we'll have our base `Home` route component. Then we'll also setup a `Photo` component. This will render a div with a random image in it.

```js
const Home = () => <div>You're on the Home Tab</div>;
const Photo = () => {
  return (
    <div>
      <div>
        <img src="https://source.unsplash.com/random" />
      </div>
    </div>
  );
};
```

## Setup Routes

Now we need to setup our app routes and also be able to navigate between them. We have 2 links, and then 2 routes. Our `/` route for `Home` needs the `exact` prop otherwise it will render even when we visit the `/photo` route.

```js
class App extends Component {
  render() {
    return (
      <div>
        <div className="links">
          <Link to="/" className="link">
            Home
          </Link>
          <Link to="/photo" className="link">
            View Photo
          </Link>
        </div>
        <div className="tabs">
          <Switch>
            <Route path="/" exact component={Home} />
            <Route path="/photo" component={Photo} />
          </Switch>
        </div>
      </div>
    );
  }
}
```

## Add Link State

Rather than just passing in a simple string to `Link` you can pass in an object. This will be accessible in the `location` prop that is passed into the component rendered on our `<Route>`.

We'll add in `{modal: true}` because if the user clicks on it they are active in the application and so we know it's not a fresh visit from someone just visiting a URL.

```js
<Link
  to={{
    pathname: "/photo",
    state: { modal: true },
  }}
  className="link"
>
  View Photo
</Link>
```

## Show the Modal

Now we need to adjust our `Photo` route component to respond when we indicate we want a modal. We destructure `location` from our props that are passed in. We default `state` to an empty `{}` because in the event that state isn't set it will just be undefined.

Then we grab our `modal` value off of state.

```js
const Photo = ({ location }) => {
  const { state = {} } = location;
  const { modal } = state;
  return (
    <div>
      <div>
        <img src="https://source.unsplash.com/random" />
      </div>
    </div>
  );
};
```

Now with our knowledge of whether or not we are going to render a `modal` we can make adjustments. For example we add a `modal` `className` to our outer div. We also render a `Close` button that is just a link to the home route which will render a different route, thus causing the modal to close.

```js
const Photo = ({ location }) => {
  const { state = {} } = location;
  const { modal } = state;
  return (
    <div className={modal ? "modal" : undefined}>
      {modal && <Link to="/">Close</Link>}
      <div>
        <img src="https://source.unsplash.com/random" />
      </div>
    </div>
  );
};
```

## Ending

Passing state is an effective method to change the experience for users as they flow through your application while altering the experience for users that visit your application from a direct URL. Both experiences should be relatively identical as to not confuse the user from visit to visit, but route state should be used to enhance the users experience.

![](https://images.codedaily.io/lessons/reactRouter/modalView.gif)



![](https://images.codedaily.io/lessons/reactRouter/modalViewCode.png)