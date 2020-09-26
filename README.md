# Torus (`torus-dom`)

[![npm torus-dom](https://img.shields.io/npm/v/torus-dom.svg)](http://npm.im/torus-dom)
[![TypeScript types](https://img.shields.io/npm/types/torus-dom.svg)](https://github.com/thesephist/torus/tree/master/types)
[![Build Status](https://travis-ci.com/thesephist/torus.svg?branch=master)](https://travis-ci.com/thesephist/torus)
[![gzip size](http://img.badgesize.io/https://unpkg.com/torus-dom/dist/index.min.js?compression=gzip)](https://unpkg.com/torus-dom/dist/index.min.js)
[![install size](https://packagephobia.now.sh/badge?p=torus-dom)](https://packagephobia.now.sh/result?p=torus-dom)

Torus is an event-driven model-view UI framework for the web, focused on being **tiny, efficient, and free of dependencies**.

You can find the 📄 **[full documentation for Torus here, on Github pages](https://thesephist.github.io/torus/api-documentation.html)**.

Torus also has an annotated, easy to read version of the entire (pretty concise) codebase, also on [Github Pages](https://thesephist.github.io/torus/). Check it out if you want to learn more about how the frameworks is designed, and how the virtual DOM and templating works!

Here's what Torus looks like in action! This is a fully functional counter app, no compilation or bundling steps needed.

Drop this script tag in your HTML

```html
<script src="https://unpkg.com/torus-dom/dist/index.min.js"></script>
```

... and load this script.

```javascript
// every view extends Component or StyledComponent
class App extends Torus.StyledComponent {

    init() {
        // initialize our local state
        this.count = 0;
    }

    styles() {
        // we can define dynamically and efficiently injected
        //  styles for our component with styles(), like this.
        //  These styles are also automatically scoped to the
        //  component, and we can use the full, nested SCSS syntax.
        return css`
        font-family: system-ui, sans-serif;
        h1 {
            text-align: center;
        }
        button {
            padding: 4px 8px;
            &:hover {
                opacity: .8;
            }
        }
        `;
    }

    compose() {
        // We define the structure of our component in compose(),
        //  using a JSX- and lit-html-like syntax as a template string.
        return jdom`<main>
            <h1>Hi! You clicked ${this.count} times.</h1>
            <button onclick="${() => {
                this.count ++;
                this.render();
            }}">Count up!</button>
        </main>`;
    }

}

// mount the app to the page
document.body.appendChild(new App().node);
```

## Features

### 👌 Tiny without compromises

Torus has no production dependencies, requires no build step to take advantage of all of its features, and weighs in at under 5kB gzipped including the templating engine, renderer, component and event system, and CSS-in-JS wrapper. This makes it simple to adopt and ship, for anything from rendering a single component on the page to building full-scale applications.

### 🏃‍️ Fast and responsive by default

Torus isn't designed to be the fastest virtual DOM library (there are great alternatives like `inferno`), but performance and responsiveness are among the primary goals of the project. While remaining tiny, Torus tries to be as fast and responsive as possible, _especially_ in rendering. Combined with the small bundle size, this makes Torus great for building web applications for anywhere, on any device.

### 💻 Portable across the web platform

Torus's architecture encapsulates all of the rendering and updating logic within the component itself, so it's safe to take `Component#node` and treat it as a simple pointer to the root DOM element of the component. You can move it around the page, take it in and out of the document, embed it in React or Vue components or even web components, and otherwise use it anywhere a traditional DOM element can be used. This allows you to include Torus components and apps in a variety of frontend architectures.

Combined with the small size of Torus, this makes it reasonable to ship torus with only one or a few components for a larger project that includes elements from other frameworks, if you don't want to or can't ship an entire Torus application.

### 🌍 Internationalization and extensibility

Torus doesn't concern itself with internationalization, but as developers, we can use the APIs available to us make internationalization possible inside our Torus components. Torus exposes much of the rendering process and the virtual DOM to you, the developer, and importantly allows us create a `preprocessor` that can take in JDOM, and modify it before it reaches the renderer, so we can make modifications to the DOM that the renderer sees with our own code. This makes Torus highly extensible and ideal for i18n. In fact, the component preprocessor API is what makes Torus's `Styled()` components possible. (`Styled()` adds a new class name to the JDOM before the component is rendered.)

For example, we might make an `I18nComponent`, which can act as a base component class for an internationalized project, like this.

```javascript
class I18nComponent extends Component {

    // The default preprocess method just returns the jdom as-is. We can override it
    //  to modify the JDOM given by component's `#compose()` method before it reaches the
    //  virtual DOM renderer.
    preprocess(jdom, _data) {
        // Here, we might recursively traverse the JDOM tree of children
        //  and call some custom `translate()` function on each string child
        //  and any displayed props like `placeholder` and `title`.
        //  As a trivial example, if we only cared about text nodes on the page,
        //  we could write...
        const translate = jdom => {
            if (typeof jdom === 'string') {
                // translate text nodes
                return yourImplementationOfTranslateString(jdom);
            } else if (Array.isArray(jdom.children)) {
                // it's an object-form JDOM, so recursively translate children
                jdom.children = jdom.children.map(yourImplementationOfTranslateString);
                return jdom;
            }
            return jdom;
        }

        // In production, we'd also want to translate some user-visible properties,
        //  so we may also detect and translate attrs like `title` and `placeholder`.
        return translate(jdom);
    }

}
```

## Examples

I (Linus) use Torus for most of my personal projects when I need a client-side UI library. Some of these projects include:

- [Codeframe, an online code editor](https://github.com/thesephist/codeframe)
- [Mira, a contacts app](https://github.com/thesephist/mira)
- [Unim.press, a newspaper-style Reddit reader](https://unim.press/)
- [Graphing calculator](https://graphcalc.surge.sh)
- [A Hacker News reader](https://torushn.surge.sh)
- [1989.style, a quiz about Taylor Swift songs](https://1989.style/)
- [Sounds from Places](https://soundsfromplaces.surge.sh)

## Influences

Torus's API is a mixture of declarative interfaces for defining user interfaces and views, and imperative patterns for state management, which I personally find is the best balance of the two styles when building large applications. As a general practice, components should try to remain declarative and idempotent, and interact with data models / state via public, stable imperative APIs exposed by data models.

Torus's design is inspired by React's component-driven architecture, and borrows common concepts from the React ecosystem, like the idea of diffing in virtual DOM before rendering, composition with higher order components, and mixing CSS and markup into JavaScript to separate concerns for each component into a single class. But Torus builds on those ideas by providing a more minimal, less opinionated lower-level APIs, and opting for a stateful data model rather than a view/controller layer that strives to be purely functional.

Torus also borrows from [Backbone](http://backbonejs.org) in its data models design, for Records and Stores, for having an event-driven design behind how data updates are bound to views and other models.

Lastly, Torus's `jdom` template tag was inspired by [htm](https://github.com/developit/htm) and [lit-html](https://github.com/Polymer/lit-html), both template tags to process HTML markup into virtual DOM.

### Runtime over compilation

Frontend development tooling has been on a trend to doing more and more at build time / compile time, making richer syntaxes and features possible, like adopting proposal-track JavaScript features and JSX. [Svelte](https://svelte.dev) is a wonderful example of how compile-time tooling can create a categorically difference way of thinking about building user interfaces.

Torus doesn't try to be another build-time tool. One of Torus's goals is to be as useful as possible _at runtime_ while sacrificing as little performance and overhead as possible, so we can eliminate the compile step in development. As a result, Torus is the gold-standard experience of prototyping user interface ideas: just drop a `<script>` tag in the document and start writing. The downside of this conscious prioritization of runtime over compile-time is that some features that aren't a part of JavaScript syntax just aren't possible without a compile step. Notably, the [ECMAScript Decorator](https://github.com/tc39/proposal-decorators) syntax and JSX syntax are not possible, but would be useful, were there to be a compile step in building Torus apps. We could write JDOM templates in JSX, which is syntactically very similar, and wrap state-updating methods and event listeners in `@render` and `@bind` decorators rather than calling `this.render()` and `.bind(this)` at every instance.

Adding compilation support is not currently on the roadmap, but it should be straightforward, since Torus is a subset of modern JavaScript. We may come back to addressing these marginal benefits of compilation in the future, especially if decorators show no progress in the proposal track.

### A note on fragments

Similar declarative UI frameworks like React and Preact have introduced the notion of [Fragments](https://reactjs.org/docs/fragments.html), which is syntax sugar for rendering an array of (virtual) DOM nodes from a function. This is because while having a component return an array of nodes doesn't make sense on its own, it's often useful to have internal functions that return parts of components and views as nodes without a wrapper element. Torus natively supports an array representation of a list of nodes -- just wrap JDOM objects in an array! Although, unlike in React, a component cannot render more than one node, most use cases of fragments are covered by simply being able to pass around a representation of a list of nodes in an array internally inside a component, and this is intuitively supported out-of-the box in Torus.

I've toyed with the idea of modifying the `jdom` template tag to be able to turn template representations of fragments like `<>...</>` into arrays of nodes. `jdom` is also capable of simply parsing adjacent top-level elements in the template and return them in a single array. However, I decided not to ship these features for now, because I believe these use cases are adequately covered by being able to return the `.children` of a jdom template, even perhaps one wrapped inside the `<>...</>` fragment markers for readability, or simply returning an array of JDOM objects. I appreciate the explicitness of the extra step involved in returning a nontraditional array from a rendering operation, and I think the occasional cost of returning arrays as intermediate representations of parts of a view is not worth the extra feature cost.

## Compatibility

Torus uses Symbols, Maps, and Sets, so is compatible with the latest versions of all major browsers except Internet Explorer 11. On older browsers that don't support e.g. Array spread operators, you may need to transpile the library to ES5 using a tool like Babel.

## Installation and usage

You can install Torus from NPM as `torus-dom`. Torus is still considered _beta_, and not to a 1.0 release yet. I believe the API is stable now and most of the major bugs have been squashed, but no guarantees until 1.0.

```sh
npm install --save torus-dom
# or
yarn add torus-dom
```

```javascript
import { StyledComponent, Record, Store, Router } from 'torus-dom';
```

Alternatively, you can also just import Torus with:

```html
<script src="https://unpkg.com/torus-dom/dist/index.min.js"></script>
```

Torus will export all of its default globals to `window.Torus`, so they're accessible as global names to your scripts. This isn't recommended in production apps, but great for experimenting.

## Contributing

If you find bugs, please open an issue or put in a pull request with a test to recreate the bug against what you expected Torus to do. If you have feature requests, I might not necessarily honor it, because Torus is being built mostly to suit my personal workflow and architecture preferences. But I'm open to hearing your opinion! So feel free to open an issue, with the expectation that I might not decide to add the feature to Torus (especially if it'll inflate the bundle size or require a transpiler.)

### Builds

You can use both npm and Yarn to develop Torus, but the npm scripts use Yarn, and Yarn is officially supported as it's what I use to develop and build Torus.

To build Torus from source, run

```sh
yarn build
```

This will run `./src/torus.js` through a custom toolchain, first removing any debug function calls and running that result through Webpack, through both `development` and `production` modes. Both outputs, as well as the vanilla version of Torus without Webpack processing, are saved to `./dist/`. Running `yarn clean` will delete any such build artifacts, as well as any generated coverage reports.

### Generating documentation from comments

Torus has a unique system for generating documentation from code comments that begin with `//>`. To generate comment docs, run

```sh
yarn docs
```

Docs files will be generated at `./docs/` and are viewable on a web browser. Check out [the Github page for this project](https://thesephist.github.io/torus/) for an example of what this script generates.

### Running tests

To run Torus's unit tests and generate a coverage report to `coverage/`, run

```sh
yarn test
```

This will run the basic test suite on a development build of Torus. More comprehensive integration tests using full user interfaces like todo apps is on the roadmap.

We can also run tests on the production build, with:

```sh
yarn test:prod
```

This **won't generate a coverage report**, but will run the tests against a minified, production build at `dist/torus.min.js` to verify no compilation bugs occurred.

### Linting

Torus lints with ESLint, using a custom configuration. To run the linter, run

```sh
yarn lint
```

or consider using an editor plugin for ESLint.
