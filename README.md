# TLDR - ReactJS

## Contents:

- [General React principles](#general-react-principles)
- [Setup](#setup)
- [Importing react](#importing-react)
- [Functional components](#functional-components)
- [Class components](#class-components)
- [PropType (property validation)](#proptype-property-validation)
- [Managing state](#managing-state)
- [Composing components](#composing-components)
- ["Lifting state"](#lifting-state)
- [Component lifecycle](#component-lifecycle)
- [Improve performance](#improving-performance)
- [Accessing the regular DOM, using `refs`](#accessing-the-regular-dom-using-refs)
- [Further reading](#further-reading)

---

## General React principles

> In the typical React dataflow, props are the only way that parent components interact with their children.
>
> A component must never modify its own props.
>
> To modify a child, you re-render it with new props.
>
> Any data shared by components should live in the state of the parent component.

## Setup

```console
npm install @babel/core babel-loader @babel/preset-env @babel/preset-react react-dom react prop-types
```

In `.babelrc`:

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

Webpack config:

```diff
module.exports = {

  //...

  module: {
    rules: [
      {
-       test: /\.js$/,
+       test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      }
    ]
  },
  resolve: {
    extensions: [
      '.js',
+     '.jsx'
    ]
  }

  // ...

}
```

## Importing React

Generally:

```javascript
import React from "react";
import ReactDOM from "react-dom";
```

Specific packages:

```javascript
import React, { Component } from "react";
import ReactDOM from "react-dom";
```

or

```javascript
import { useState } from "react";
```

## Functional components

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

## Class components

```javascript
class MyComponent extends React.Component {

  // Set state as a class field (preferred method)
  // This requires Babel option `@babel/plugin-proposal-class-properties`

  this.state = {
    foo: 'initialValue',
    bar: this.props.bar,   // 'props' is passed into the constructor (see below)
  }

  constructor(props) {    // fires before component is mounted
    // makes 'this' refer to this component, not React.Component
    super(props);

    // Set state (alternative method the using class fields, above)
    this.state = {
      foo: 'initialValue',
      bar: this.props.bar,
    }

    // class methods should normally be bound to 'this'
    this.fooExists = this.fooExists.bind(this);
  }

  fooExists() {
    return !!this.state.foo;
  }

  render() {
    // 1. Components must return 1 element only
    // 2. Encase multiple lines in parentheses
    // 3. Embed JS in the JSX using `foo={some js}`
    // 4. In JSX, HTML attributes are in camelCase
    return (
      <div>
        <h1>Hello {this.state.foo}</h1>
        <input
          className='someClass'
          type='text'
          value={this.state.bar}
        />
      </div>
    )
  }
}
```

## PropType (property validation)

Enable validation of component 'props' (requires `prop-types`).

```diff
import React from 'react';
+import PropTypes from "prop-types";

class MyComponent extends React.Component {

  // ...

}

+ MyComponent.propTypes = {
+
+   // required
+   foo: PropTypes.string.isRequired,
+
+   // optional
+   bar: PropTypes.string,
+
+   // an object of a particular shape
+   someObject: PropTypes.shape({
+     colour: PropTypes.string,
+     fontSize: PropTypes.number
+   }),
+
+ };
+

export default MyComponent;
```

## Managing state

- Your state is encapsulated: It is only accessible _to the component that owns and sets it_.
- However, a component can pass its state down as props to its child components
- Components re-render each time receives new props

Let's add a `ResultPanel` component to some other one:

```javascript
render(){
  return !!this.state.gssid
    ? <ResultPanel gssid={this.state.gssid} />
    : '<p>Choose a place</p>';
}
```

The `ResultPanel` component would receive the gssid in its `props`:

```javascript
function ResultPanel(props) {
  return (
    <div>
      <h2>Location:</h2>
      <p>{props.gssid}</p>
    </div>
  );
}
```

Never modify `this.state` directly, instead use:

- `this.setState({ foo: 'bar' });`, or
- `this.setState({ this.state..., foo: 'bar' });`

## Composing components

..how to add components together, to make a larger complex component:

```javascript
// define a re-usable component (re-usable by passing in 'props')
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Tom" />
      <Welcome name="Dick" />
      <Welcome name="Sally" />
    </div>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

## "Lifting state"

For when you need two components to be in sync with each other..

> In React, sharing state is accomplished by moving it up to the closest common ancestor of the components that need it.

Update the parent component:

```diff
class MainComponent extends React.Component {

  constructor(props) {
    super(props);
    this.state = { foo: 'bar' };

    // class methods need to be bound to 'this'
    this.updateFoo = this.updateFoo.bind(this);
  }

  updateFoo() {
    this.setState({this.state..., foo});
  }

  render() {
    return (
-      <ChildOne />
-      <ChildTwo />
+      <ChildOne foo={this.state.foo} />
+      <ChildTwo foo={this.state.foo}/>
    )
  }
}
```

...and then update the children components:

```diff
class ChildOne extends React.Component {
  contructor(props) {
    super(props);
-    this.state.foo 'AAA';
  }

-  updateFoo() {
-    this.setState({this.state..., foo});
-  }

-  render() {
+  render(props) {
-    return <p>{this.state.foo}</p>
+    return <p>{this.props.foo}</p>
  }
}

class ChildTwo extends React.Component {
  contructor(props) {
    super(props);
-    this.state.foo 'BBB';
  }

-  updateFoo() {
-    this.setState({this.state..., foo});
-  }

-  render() {
+  render(props) {
-    return <p>{this.state.foo}</p>
+    return <p>{this.props.foo}</p>
  }
}
```

## Component Lifecycle

In applications with many components, it’s very important to free up resources taken by the components when they are destroyed.

Example: unsetting event handlers, killing timers, etc.

```javascript
class myComponent extends React.Component {
  constructor() {
    // fires once on component init
  }

  componentWillMount() {
    // fires immediately before the initial render
  }

  componentDidMount() {
    // fires immediately after the initial render
    //
    // Most common use case for componentDidMount:
    // start AJAX calls to load in data
    //
    // You can also setup up event handlers, timers, etc here
  }

  componentWillReceiveProps() {
    // fires when component is receiving new props
  }

  shouldComponentUpdate() {
    // fires before rendering with new props or state
  }

  componentWillUpdate() {
    // fires immediately before rendering
    // with new props or state
  }

  componentDidUpdate() {
    // fires immediately after rendering with new P or S
  }

  componentWillUnmount() {
    // fires immediately before component is unmounted
    // from DOM (removed)..
    //
    // This is a good place for clearInterval(foo),
    // removeEventListener, window.cancelAnimationFrame, etc
  }

  render() {
    // fires when props or state is updated
  }
}
```

## Improve performance

By default React will, re-render the component each time its `props` change.

Here's the default behavior of a component, deciding when to update (re-render):

```
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

If you know of any cases where you your component does not need to re-render,
then you should return `false` in `shouldcomponentUpdate` in those cases.

Example: only re-render the component when `props.colour` changes:

```
shouldComponentUpdate(nextProps, nextState) {
  if (this.props.colour !== nextProps.colour) {
    return true;
  }
  return false;
}
```

Keep in mind that it can cause major problems if you set it and forget it,
because your React component will not update normally. So use with caution.

## Accessing the regular DOM, using `refs`

For accessing regular DOM elements, using the regular DOM API, use `refs`.

When you might need refs:

- you might need to set focus on a `<input type="text">` element
- you might need to integrate a non-React, third party library

Creating a `ref`:

```diff
class myInput extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      foo: 'initialValue',
      bar: this.props.bar,
    }
+    this.inputElemref = React.createRef();
  }

  ...

  render() {
    return (
      <input
        className='someClass'
        type='text'
+       ref={this.inputElemRef}
        value={this.state.bar}
      />
    )
  }
}
```

Accessing the `ref`:

```javascript
  focusInput() {
    // You must access "current" to get the DOM node using the **raw DOM API**
    this.inputElemRef.current.focus();
  }
```

## Further reading:

- [Comonents and Props](https://reactjs.org/docs/components-and-props.html#props-are-read-only)
- [State and Lifecycle](https://reactjs.org/docs/state-and-lifecycle.html)
- [Lifting State up](https://reactjs.org/docs/lifting-state-up.html)
- [Refs and the DOM](https://reactjs.org/docs/refs-and-the-dom.html)
- [react-16-lifecycle-methods-how-and-when-to-use-them](https://blog.bitsrc.io/react-16-lifecycle-methods-how-and-when-to-use-them-f4ad31fb2282)
- [React lifecycle-simulators](https://reactarmory.com/guides/lifecycle-simulators)
- [PropTypes](https://reactjs.org/docs/typechecking-with-proptypes.html)
