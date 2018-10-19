# React Primer

This is a concise introduction to the core elements of
[React](https://reactjs.org) development, intended for developers who are
already familiar with HTML, CSS, and basic JavaScript.

- [Components](#components)
- [State](#state)
- [Props](#props)
- [Stateful components](#stateful-components)
- [Stateless components](#stateless-components)
- [Events](#events)
- [Common rendering patterns](#common-rendering-patterns)
- [styled-components](#styled-components)
- [Modern JavaScript](#modern-javascript)
- [Immutability](#immutability)
- [Routing](#routing)
- [Container components](#container-components)
- [HTTP requests](#http-requests)
- Type safety
- MobX

## Components

A React app is a collection of many individual components. A component is a
*visual* building block that serves a particular purpose, such as displaying a
button or a certain web page.

In the long run, it's recommended to split larger components into smaller
ones. Small components are easier to reuse and debug, whereas large components
are challenging to maintain and understand.

## State

Some components have their own internal state that dictates how they should
behave (e.g. whether a dropdown menu should be open or closed).

Avoid duplicating state and, instead, distribute relevant parts of the
state to other components as props.

When you reach a point where many components across your application depend
on a shared state, you should look for a global state management strategy.
[MobX](https://mobx.js.org) is a minimal third-party library that provides a
clean way to create and manage encapsulated stores. React also has a built-in
API called [Context](https://reactjs.org/docs/context.html) which allows you to
create globally accessible states.

## Props

Components can pass read-only data to child components as props.

You can also pass entire functions to child components. This is useful
when, for example, you want a button component to call the parent
component's save method when clicked.

## Stateful components

Stateful components are classes that have their own key–value stores.
You can change the state by calling `this.setState` and providing the
key and a new value. The state is readable via `this.state`.

```js
class Dropdown extends Component {
  // Default state
  state = {
    isOpen: false,
  }

  render() {
    return (
      <div onClick={() => this.setState({isOpen: true})}>
        {
          this.state.isOpen &&
          <div>
            Dropdown content goes here
          </div>
        }
      </div>
    )
  }
}
```

By default, **every state and prop change triggers a re-render**.

Stateful components contain
[lifecycle methods](https://reactjs.org/docs/react-component.html#the-component-lifecycle)
or hooks that can be used to influence a component's behavior under certain
conditions. For example, by implementing the lifecycle method
`componentDidMount` you can tell a component to fire an HTTP request on
mount (after loading up the component) that fetches a list of blog posts to be
shown on the page:

```js
...

componentDidMount() {
  this.fetchBlogPosts()
}

fetchBlogPosts = async () => {
  this.setState({
    isLoading: true, // render a spinner while this.state.isLoading is true
  })

  const blogPosts = await axios.get('https://example.com/api/blog-posts')

  this.setState({
    blogPosts,
    isLoading: false,
  })
}

render() {
  ...
}

...
```

## Stateless components

Stateless components, also referred to as dumb components, are essentually just
render functions that take any number of props as arguments and return JSX
(markup). They do not have their own state so their behavior is solely
influenced by props.

```js
// Here, the 'type' prop is given a default value of 'button'
const Button = ({type = 'button', onClick, label}) =>
  <button type={type} onClick={onClick}>
    {label}
  </button>

...

<Button label="Click Me" onClick={() => window.alert('Hi!')} />
```

## Events

Many elements support built-in [events](https://reactjs.org/docs/handling-events.html)
that can be used to interact with your components. Here are some examples:

```js
<input
  type="text"
  value={this.state.username}
  onChange={event => this.setState({username: event.target.value})}
/>
```

```js
<button onClick={() => this.save()}>
  Save
</button>
```

```js
<form onSubmit={event => {
  event.preventDefault()
  
  this.save()
}}>
  ...
  <button type="submit">
    Save
  </button>
</form>
```

## Common rendering patterns

### Conditional rendering

Render a spinner while `this.state.isLoading` is true:

```js
<div>
  {
    this.state.isLoading &&
    <Spinner />
  }
</div>
```

Remember to convert truthy/falsy values into proper booleans, because otherwise
React will render the values themselves!

```js
<div>
  {
    Boolean(this.state.blogPosts.length) &&
    <ul>
      ...
    </ul>
  }
</div>
```

### Lists of items

Iterate over an array of items with `Array.prototype.map`:

```js
<div>
  {
    this.state.blogPosts.map(blogPost =>
      <article key={blogPost.id}>
        <h2>{blogPost.title}</h2>
        <div>{blogPost.body}</div>
      </article>
    )
  }
</div>
```

Every iteratee needs a unique `key` prop to allow React to figure out which
item (not) to update during the next re-render.

## styled-components

[styled-components](https://www.styled-components.com) is a helper
library that provides a convenient interface for styling your React
components in plain JavaScript, outputting presentational React
components.

The CSS rules are written inside template literals (backticks) allowing
you to write any JavaScript expressions like variables and function
calls. Styled components can be extended and referenced by other styled
components.

```js
import styled from 'styled-components'
import {PRIMARY_COLOR, SECONDARY_COLOR} from './styles/colors'

const Button = styled.button`
  padding: 20px;
  transition: transform 100ms;

  &:hover {
    transform: scale(1.05);
  }
`

const PrimaryButton = styled(Button)`
  background-color: ${PRIMARY_COLOR};
  border: none;
  color: ${SECONDARY_COLOR};
`

...

<PrimaryButton onClick={() => window.alert('Hi!')}>
  Click Me
</PrimaryButton>
```

## Modern JavaScript

### const vs let

Always prefer `const`. Use `let` if absolutely necessary.

`const` is better because it disallows variable reassignment and reduces
unnecessary side effects.

### Array spread

Array spread syntax merges arrays.

```js
const girls = ['Tiffany', 'Wendy', 'Alice']
const boys = ['Caleb', 'Tim', 'Rick']

// Now let's merge both arrays into one
const everyone = [
  ...girls,
  ...boys,
]
```

### Object spread

Object spread syntax merges/overrides objects.

```js
const personal = {
  name: 'Wendy McDuff',
  email: 'wendy@example.com',
}

const social = {
  twitter: 'wendymcduff',
  facebook: 'wendy.mcduff',
  instagram: 'wendywendy',
}

// Now let's pull them all into a single object
const profile = {
  ...personal,
  ...social,
}
```

### Implicitly returning functions

Without curly braces, arrow functions will by default return whatever comes
after the arrow.

The following functions produce the same output:

```js
const sayHi = () => 'Hi!'

const sayHi = () => {
  return 'Hi!'
}
```

However, if you want to implicitly return an **object**, you'll need to
remember to wrap the object in parentheses:

```js
const giveMeAnObject = () => ({
  foo: 'bar',
})

// Which is equivalent to:

const giveMeAnObject = () => {
  return {
    foo: 'bar',
  }
}
```

### Async/await

`async/await` functions are a more convenient way to use `Promise`s.
Internally, they still behave as promises and are fully compatible with other
non-`async/await` functions that return `Promise`s.

```js
const fetchBlogPosts = async () => {
  const blogPosts = await axios.get('https://example.com/api/blog-posts')

  return blogPosts
}
```

You can call `async` functions anywhere in your code, but you always need to
declare a function as an `async` function if you want to use the `await`
keyword.

## Immutability

Instead of modifying existing objects and arrays, try to create new copies.
Spread syntax is great because it only reads the inputs and returns completely
new values. `Array.prototype.map` and `Array.prototype.filter` also produce
new values, without touching the original arrays.

The main problem with mutations is that changing the shape of existing things
can solve your immediate problem but it can break other parts of the
application without your knowing it.

## Routing

Routing lets you split your app into separate page components based on the
requested URL path. For example, you can render a Home component at `/` and a
Blog component at `/blog`.

[react-router-dom](https://reacttraining.com/react-router/web/example/basic) is
the de facto routing framework for React.

To enable app-wide routing, define your routes in your root (top-most)
component:

```js
import React, {Fragment} from 'react'
import {BrowserRouter, Switch, Route} from 'react-router-dom'

const Router = () =>
  <BrowserRouter>
    <Switch>
      <Route exact path="/" component={Home} />
      <Route path="/blog" component={Blog} />
      <Route path="/blog/:id" component={BlogPost} />
      <Route component={NotFound} />
    </Switch>
  </BrowserRouter>
```

`Switch` ensures that only one of the enclosed route components gets mounted at
a time.

Having a colon in a route (e.g. `/blog/:id`) forwards the wildcard `id` to the
component as `this.props.match.params.id` so that you are able to fetch a blog
post by that ID.

The last `Route` entry without a path is a fallback route which usually works
as a 404 page.

Having `BrowserRouter` as the routing adapter requires that the web server
(e.g. Amazon S3/CloudFront) be capable of serving the same `index.html` entry
point regardless of path: `https://example.com/foo/bar` should serve
`https://example.com/index.html`. If your web server does not support it, you
can replace `BrowserRouter` with `HashRouter` which prefixes paths with `/#/`.

You can link to these routes with the `Link` component:

```js
import {Link} from 'react-router-dom'

...

<Link to="/blog">Check out my blog</Link>
```

## Container components

Container components are just an architectural concept referring to any
stateful component that has a "master"-like responsibility.

A container component typically holds data and methods which it delegates to
child components to form a coherent unit, like a single web page.

In our previous routing example, the `Home` page component could be seen as a
container component that's in charge of maintaining the home page related state
and then combining smaller components to make up the actual page.

To avoid state duplication, the container component's state is the single
source of truth that gets distributed to child components.

## HTTP requests

A React app is often just a hollow front-end client so it needs to fetch
content (e.g. blog posts) from somewhere. Since web browsers are built around
HTTP, we can use the very same protocol to transfer information between a React
app and back-end services.

We prefer to use [axios](https://github.com/axios/axios) for HTTP requests
because it's simpler and more convenient than using the native
[`fetch` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
directly.

This is how you'd `GET` a list of blog posts from a hypothetical RESTful API
and show them in a list:

```js
import React, {Component} from 'react'
import axios from 'axios'

class BlogPosts extends Component {
  state = {
    blogPosts: [],
  }

  fetchBlogPosts = async () => {
    const blogPosts = await axios.get('https://example.com/api/blog-posts')

    this.setState({blogPosts})
  }

  componentDidMount() {
    this.fetchBlogPosts()
  }

  render() {
    return (
      <ul>
        {
          this.state.blogPosts.map(
            blogPost => <li key={blogPost.id}>{blogPost.title}</li>
          )
        }
      </ul>
    )
  }
}
```
