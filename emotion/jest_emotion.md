## Intro

CSS-in-JS libraries like emotion are great, however because they are indeed generating CSS accessing a CSS value in a test may be difficult. Access a CSS value in with normal stylesheets would also be very difficult.

Generally you might apply a specific class name that you can test for or a `style` prop.

When using a CSS-in-JS library like emotion some of your dynamic logic might be locked away inside of a styled component that is generating the dynamic styling from props.

## The Scenario

Lets examine a scenario. One common practice with the web is scaling an item in and out. React can make this a little bit difficult when it comes to animating out. Unmounting will remove it from the DOM immediately. Bringing in react-transition-group, or other animation libraries is overkill from a single scale animation.

One of the use cases I've encountered is displaying a mute badge on a persons display avatar. We want it to gracefully animate in and out and don't care that it's always rendered. However we want a test to verify that indeed when the user mutes an icon is visible.

Lets setup a quick use case. Here is our `App` it has a show toggle on state, and passes that prop to a `DiplayPiece`.

```js
class App extends Component {
  state = {
    show: false,
  };
  render() {
    return (
      <div className="App">
        <DisplayPiece show={this.state.show}>HI</DisplayPiece>
        <button
          onClick={() => {
            this.setState(state => {
              return {
                show: !state.show,
              };
            });
          }}
        >
          Toggle
        </button>
      </div>
    );
  }
}
```

Our `DisplayPiece` is a circle with some content. The key part is the ending where we toggle the `transform` based upon our `show` prop that we passed in.

Our transition is setup to 1 second so as you toggle the `show` value it will scale in or out over a 1 second period.

```js
const DisplayPiece = styled.div(
  {
    width: "50px",
    height: "50px",
    borderRadius: "25px",
    backgroundColor: "tomato",
    transition: "transform 1s ease",
    color: "#FFF",
    display: "flex",
    alignItems: "center",
    justifyContent: "center",
  },
  ({ show }) => {
    return {
      transform: show ? "scale(1)" : "scale(0)",
    };
  },
);
```

![](https://images.codedaily.io/lessons/general/jestEmotion/single.png)

## The Issue

The `DisplayPiece` is always rendered. In some cases without animation or without animation on unmount you would check if the `DisplayPiece` existed in the DOM or not.

With it always being rendered, and our style buried in Emotion we can't write a test to check if it's in the DOM or not. We have no confidence that when we ship our code that it will appear and hide correctly.

## The Test and the Fix

Lets fix it and gain a little confidence. One library that will help us is [https://www.npmjs.com/package/jest-emotion](https://www.npmjs.com/package/jest-emotion). It allows us to add in specific matchers to our Jest setup and actually query. A long with `react-testing-library` we can render our component.

Using the provided querying abilities from `react-testing-library` we can find our elements and pass them directly to expect. Using `toHaveStyleRule` which is one of the provided matchers from `jest-emotion` we can grab the `transform` value of `DisplayCircle` and check if it's scaled in or out.

```js
import React from "react";
import { render } from "react-testing-library";

import App from "./App";

it("should be visible when clicked", () => {
  const { getByText } = render(<App />);

  expect(getByText("HI")).toHaveStyleRule("transform", "scale(0)");

  getByText("Toggle").click();

  expect(getByText("HI")).toHaveStyleRule("transform", "scale(1)");
});
```

Now if we run our `test` command we can see our test is passing. Everything is working as we expect.

![](https://images.codedaily.io/lessons/general/jestEmotion/test.png)


## Ending

Don't go too overboard with this. The point of tests is to have confidence that your code is operating as expected. This generally revolves around business logic that is crucial to your customers going about their day to day business.

If you have some business logic like this locked away in Emotion exposing and testing it is an easy way to gain confidence your code is working as expected.

![](https://images.codedaily.io/lessons/general/jestEmotion/toggle.gif)