## Intro

CSS-in-JS is a powerful construct for styling with React based applications. In some small cases it can be unnecessary, however as applications grow CSS can become difficult to maintain. Even more so within applications that do themeing and or need run-time evaluation CSS, normal CSS can be difficult to use for these cases.

There are a lot of rules around CSS when it comes to specificity, and even the order at which a `.css` file was included on the page to determine whether or not a style is applied/overridden. These are generally things you don't really want to have to worry about when building out a large dynamic application. This is where CSS-in-JS libraries like Emotion come into play.

## Emotion

To get going with emotion in a React application we need to install `@emotion/styled`.

```
yarn add @emotion/styled
```

To use styled we import it into our React component.

```js
import styled from "@emotion/styled/macro";
```

We use the `macro` version which will allow `create-react-app` to use the [babel-macros](https://github.com/kentcdodds/babel-plugin-macros) babel plugin provided by emotion to run on this file. This gives additional power to emotion when it comes to compiling your app, you can read more about what it allows for you to do here [babel-plugin-emotion](https://emotion.sh/docs/babel-plugin-emotion).

## Background

Get the [background here](https://github.com/codedailyio/teach/blob/hoverCardEmotion/public/bg.jpg) and toss it in the `public` directory.

This is an arbitrary background that I pulled from [Unsplash](https://unsplash.com/) which is an amazing site for free photos from the community!

```js
const Background = styled.div({
  backgroundSize: "cover",
  backgroundRepeat: "no-repeat",
  color: "#FFF",
  position: "relative",
  width: "500px",
  height: "350px",
  cursor: "pointer",
  backgroundImage: "url(/bg.jpg)",
});
```

We use the `styled.div` call which is provided from Emotion that will actually create a React component that will be a `<div>`, create a css class, and apply that `className` to the div with those styles.

If you were to re-use this `Background` the css class won't be re-created and will be reused. So Emotion is smart about creating classes.

Now that we've created it we can just use it like a normal React component.

```js
class App extends Component {
  render() {
    return (
      <div className="App">
        <Background />
      </div>
    );
  }
}
```

![](https://images.codedaily.io/lessons/emotionHover/Background.png)

## Create Initial Display

Now that we have a background created we can create the content that will be displayed when we aren't hovering. We setup our `Background` above as a `position: 'relative'` so that the rest of our content that is contained in the `DisplayOver` will just cover the entire card.

I won't go into any depth of explanation with this css but the key piece to point out is the `backgroundColor` is transparent, and we've create a `transition` for our background color to transition in the event it changes.

```js
const DisplayOver = styled.div({
  height: "100%",
  left: "0",
  position: "absolute",
  top: "0",
  width: "100%",
  zIndex: 2,
  transition: "background-color 350ms ease",
  backgroundColor: "transparent",
  padding: "20px 20px 0 20px",
  boxSizing: "border-box",
});

const BigTitle = styled.h2({
  textTransform: "uppercase",
  fontFamily: "Helvetica",
});
```

We also setup our `BigTitle` which is just an `h2` and apply some css styling. Then once again we use the styled components as normal React components.

```js
<Background>
  <DisplayOver>
    <BigTitle>Really Cool Title!</BigTitle>
  </DisplayOver>
</Background>
```

![](https://images.codedaily.io/lessons/emotionHover/Title.png)



## Hover Section

Now onto the section to appear when we hover. We create a wrapper called `Hover` with an opacity of `0` but with a transition so when we hover all the children content will fade in.

```js
const Hover = styled.div({
  opacity: 0,
  transition: "opacity 350ms ease",
});

```

Here is the heart of the animated effects that we see. We position the components by default translated away. This is a `translate3d` but only animating the `Y` portion of the translate.

With our `opacity` on `Hover` set to `0` we won't see these so we just move them to the position where they should start. Then later when we hover we can animate them to `0`. So they fade in and slide in from the bottom.

```js
const SubTitle = styled.h4({
  fontFamily: "Helvetica",
  transform: "translate3d(0,50px,0)",
  transition: "transform 350ms ease",
});

const Paragraph = styled.p({
  transform: "translate3d(0,50px,0)",
  transition: "transform 350ms ease",
});

const CTA = styled.a({
  position: "absolute",
  bottom: "20px",
  left: "20px",
});
```

Here have our final DOM structure with our styled components. As you can see there is not a single `className`. Just descriptive components. These are components that could have come from anywhere. By that I mean you could have a library of these components for all your text/title/paragraph styling needs.

```js
<Background>
  <DisplayOver>
    <BigTitle>Really Cool Title!</BigTitle>
    <Hover>
      <SubTitle>You could vacation here!</SubTitle>
      <Paragraph>
        More description about this really cool random desert photo from unsplash!
      </Paragraph>
      <CTA>View More +</CTA>
    </Hover>
  </DisplayOver>
</Background>
```

One cool point I want to make is knowing exactly what styling is being used. Generally with `.css` files you're referencing random global names from a file that may come from anywhere.

If you go to the `.css` file you may or may not know what or if any of those styles are still being used. This can cause you to hesitate when it comes to refactoring your CSS.

Emotion and other CSS-in-JS libraries utilize JS, and thus the JS import system. So if a specific component isn't being used it becomes very easy to determine that. Not only that with JS bundling systems it wouldn't even include that component in your JS bundle automatically.

![](https://images.codedaily.io/lessons/emotionHover/Hovered.png)


## Emotion and `:hover`

Because we're using the babel macros version of emotion this allows us to reference React components as normal classes. This means `:hover` will work as expected. So you wouldn't need to apply any sort of mouse over/mouse out code in React to create this effect.

So because `DisplayOver`, `SubTitle`, and other components are children of `Background` we can configure our style changes when we `:hover` over the `Background`.

```js
const Background = styled.div({
  // Other background code
  [`:hover ${DisplayOver}`]: {
    backgroundColor: "rgba(0,0,0,.5)",
  },
  [`:hover ${SubTitle}, :hover ${Paragraph}`]: {
    transform: "translate3d(0,0,0)",
  },
  [`:hover ${Hover}`]: {
    opacity: 1,
  },
});
```

On our previous components we had applied `transitions` for these specific styles we are now changing. So on hover, these styles will change and animate accordingly.


![](https://images.codedaily.io/lessons/emotionHover/CoolHover.gif)
