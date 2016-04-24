# Statring with React.js

React.js is an JavaScript library providing a view for data rendered as HTML.
React views are typically rendered using components that contain additional
(nested) components specified as custom HTML tags. React promises programmers:

* a model in which subcomponents cannot directly affect enclosing components
  ("data flows down")
* efficient updating of the HTML document when data changes
* a clean separation between components on a modern single-page application

## Main features

* **One-way data flow:** Properties, a set of immutable values, are passed to a
  component's renderer as properties in its HTML tag.
* **Virtual DOM:** React creates an in-memory data structure cache, computes the
  resulting differences, and then updates the browser's displayed DOM
  efficiently.
* **JSX:** React components are typically written in JSX, a JavaScript extension
  syntax allowing easy quoting of HTML and using HTML tag syntax to render
  subcomponents.
* **Architecture Beyond HTML:** The basic architecture of React applies beyond
  rendering HTML in the browser.
* **React Native:** React Native libraries announced by Facebook in 2015 provide
  the React architecture to native iOS and Android applications.

## Starting with React

To start using React.js it is suggested to have a
[CommonJS](https://en.wikipedia.org/wiki/CommonJS) library. CommonJS is a
project to specify an ecosystem for JavaScript out side of the browser. The
React website suggests browserify or webpack. **Browserify** provides a
node-style `require()` to organize your browser code and load modules installed
by `npm`. Browserify will recursively analyze all the `require()` calls in your
application in order to build a **bundle** you can serve up to the browser in a
single `<script>` tag.

Browserify can generate a bundle for React. However Browserify works only with
native JavaScript. It is unable to process JSX. To deal with JSX broweserify
uses [Babel](https://babeljs.io/) compiler through the
[babelify](https://github.com/babel/babelify) browserify transformer.

Install the following packages:

```bash
# React.js
npm install react
# React DOM renderer
npm install react-dom
# Browserify
npm install browserify
# Babel extension for browserify
npm install babelify
# React presets for Babel
npm install babel-preset-react
```

To create a NPM project with the above dependencies:

```bash
npm init
npm install --save react react-dom browserify babelify babel-preset-react
```

To create a simple React.js application use the following example:

```javascript
var React = requre('react')
var ReactDOM = require('react-dom')

var MyElement = React.createObject({
  render: function() {
    return (
      <div class="my-element"></div>
    );
  }
})

ReactDOM.render(
  <MyElement/>,
  document.getElementById('content')
);
```

To compile it into a bundle run `browserify` as follows:

```bash
browserify -t [ babelify [ --presets react ] ] main.js
```

## References

1. https://en.wikipedia.org/wiki/React_(JavaScript_library)
2. https://facebook.github.io/react/docs/tutorial.html
