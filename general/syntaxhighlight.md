# Intro

Determining what markdown parser, renderer, syntax highlighter to choose can be a tricky thing. One caveat that I had was that I wanted to work with React and wanting to render to React elements rather than just rendering to a big string and setting innerHTML.

There is one main reason for this and it's security. With innerHTML or dangerouslySetInnerHTML this opens up for user injected script tags that can compromise your users. This is generally an issue if you're accepting content from users to display to other users.

# Syntax highlighting

One specific case that is generally an area of conflict is syntax highlighting. Syntax highlighting is hard because you need to parse and understand different languages. That parsing needs to be turned into different DOM elements with css classes applied so that your CSS theme you've added will be applied.

There are some optiosn but 2 main options out there are [https://highlightjs.org/](https://highlightjs.org/) and [https://prismjs.com/](https://prismjs.com/). I found that `highlightjs` has more support and examples but in general has very poor support for JSX. So I decided to go with PrismJS but ran into issues supporting that within a React environment.

# Unified, Remark and Rehype

That's where Unified, Remark, and Rehype comes into play. Lets look at what each of these does.

[Unified](https://unified.js.org/) is an interface for processing text with syntax trees and transforming between them. Creating an AST of the text it can be transformed by plugins. [Remark](https://github.com/remarkjs/remark) is one of those plugins that will parse the markdown text into an AST that will let us apply other plugins and create an output. In our case we then transform remark to [Rehype](https://github.com/rehypejs/rehype) which turns that AST into HTML. We finally use the `rehype-react` component to turn that HTML into React rather than a raw HTML string.

This ends up being a series of functions that you pipe your content to.

# Syntax Highlighting with Rehype

Now where does syntax highlighting fit into that flow? Well there is a plugin out there called [rehype-prism](https://github.com/mapbox/rehype-prism) however it's not recommended for browser usage because it includes syntax highlighting for every language via [refractor](https://github.com/wooorm/refractor). It would increase your bundle size 352kb (128kb GZipped). That's a lot!

So we'll create our own refractor prism highlighter!

# Refractor Prism

This module was 100% pulled from [https://github.com/mapbox/rehype-prism](https://github.com/mapbox/rehype-prism) and modified to be able to support just the languages that I needed to support. However I'll still step through and talk about each bit that goes into creating it.

First step lets follow the recommendation from `refractor` and just pull in `refractor/core` and the languages we want to support.

```js
import refractor from "refractor/core";

import jsx from "refractor/lang/jsx";
import javascript from "refractor/lang/javascript";
import css from "refractor/lang/css";
import cssExtras from "refractor/lang/css-extras";
import jsExtras from "refractor/lang/js-extras";
import sql from "refractor/lang/sql";
import typescript from "refractor/lang/typescript";
import swift from "refractor/lang/swift";
import objectivec from "refractor/lang/objectivec";
import markdown from "refractor/lang/markdown";
import json from "refractor/lang/json";

refractor.register(jsx);
refractor.register(json);
refractor.register(typescript);
refractor.register(javascript);
refractor.register(css);
refractor.register(cssExtras);
refractor.register(jsExtras);
refractor.register(sql);
refractor.register(swift);
refractor.register(objectivec);
refractor.register(markdown);

refractor.alias({ jsx: ["js"] });
```

Some of these have overlap. For example `jsx` actually extends the `javascript` module. So you get the benefits of `js` and `jsx`. I aliased the `jsx` module to just `js` so that I get benefits of JSX without changing much of my existing markdown.

This name will refer to the language you apply after the first triple back tick for your code block ` ```js `.


Next we create our module. I won't go through the whole thing but essentially an AST is provided. With the `visit` module we visit each thing declared an `element` and call the `visitor` function.

The `visitor` function checks if we're dealing with a `pre` or `code` tag which is an indicator we're dealing with code. Otherwise we return and move on to the next nodes. Then we parse out the language that we're dealing with. Finally with `refractor` we can apply the class names and highlight the string of code with the language we've found.

```jsx
const getLanguage = (node) => {
  const className = node.properties.className || [];

  for (const classListItem of className) {
    if (classListItem.slice(0, 9) === "language-") {
      return classListItem.slice(9).toLowerCase();
    }
  }

  return null;
}

const rehypePrism = options => {
  options = options || {};

  return tree => {
    visit(tree, "element", visitor);
  };

  function visitor(node, index, parent) {
    if (!parent || parent.tagName !== "pre" || node.tagName !== "code") {
      return;
    }

    const lang = getLanguage(node);

    if (lang === null) {
      return;
    }

    let result;
    try {
      parent.properties.className = (parent.properties.className || []).concat("language-" + lang);
      result = refractor.highlight(nodeToString(node), lang);
    } catch (err) {
      if (options.ignoreMissing && /Unknown language/.test(err.message)) {
        return;
      }
      throw err;
    }

    node.children = result;
  }
};

export default rehypePrism;
```

# Putting it all Together

Okay we've got our custom plugin created, now lets put together a Markdown rendering component.

We'll bring in `unified` to pipe all our plugins through.
Then we register `remark-parse` to parse our markdown. Next we use `remark-rehype` to convert our markdown AST to rehype.
Then we apply our plugin we created `rehypePrism` to do our virtual syntax highlighting.
Finally we use `rehype-react` to turn our final AST into React components.

The processor looks something like this.

```jsx
import React, { Fragment } from "react";
import unified from "unified";
import parse from "remark-parse";
import rehypePrism from "./rehype-prism";
import remark2rehype from "remark-rehype";
import rehype2react from "rehype-react";

const processor = unified()
  .use(parse)
  .use(remark2rehype)
  .use(rehypePrism)
  .use(rehype2react, {
    createElement: React.createElement
  });
```

Nothing will happen until we supply it with markdown to process. So we create a `Markdown` component, agree on a prop we'll call `source`.
We then take our `processor` we created and call `processSync` since we want to synchronously process the markdown in our render. Then return the `contents`.

I added `Fragment` surrounding the output as I'm unsure if there is ever a case where `contents` would return multiple roots. So wrapping it in `Fragment` is just cautionary.

```jsx
import React, { Fragment } from "react";
import unified from "unified";
import parse from "remark-parse";
import rehypePrism from "./rehype-prism";
import remark2rehype from "remark-rehype";
import rehype2react from "rehype-react";

const processor = unified()
  .use(parse)
  .use(remark2rehype)
  .use(rehypePrism)
  .use(rehype2react, {
    createElement: React.createElement
  });

const Markdown = props => {
  return <Fragment>{processor.processSync(props.source).contents}</Fragment>;
};

export default Markdown;
```

# CloudFlare Syntax Highlighting

You can grab your themes from where ever but I grabbed it from CloudFlare CDN. Which you can find here [https://cdnjs.com/libraries/prism](https://cdnjs.com/libraries/prism). Additionally some themes are shipped with `prism` however it was easier to link to a hosted solution versus adding a CSS Loader for my existing setup.

```jsx

<link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.16.0/themes/prism-okaidia.min.css"
    integrity="sha256-Ykz0nNWK7w4QWJUYR7OraN4773aMB/11aMt1nZyrhuQ="
    crossorigin="anonymous"
/>

```

# Ending

Yes all of that just to highlight and parse markdown but there are many other powerful features that are supported with unified and operating in this environment. For example parsing out a table of contents from your markdown.

You can find many plugins for the rehype/unified system here [https://github.com/rehypejs/rehype/blob/master/doc/plugins.md](https://github.com/rehypejs/rehype/blob/master/doc/plugins.md).
