# Router

This RFC describes the new API that is proposed for the `@lit-labs/router` package.

## Objective

The goal of this new proposal is to add a more intuitive API for defining and navigating routes, with a significantly low learning curve. This is achieved by incorporating the best features from popular libraries.

## Goals

- Backwards Compatibility.
- Define routes using a declarative API.
- Programmatic navigation.
- Support nested routes.
- Lazy loading.
- Route params & query.
- Route guards.

## Non-Goals

- Support for SSR.
- Transition animations.

## Motivation

Currently the `@lit-labs/router` package has the following limitations:

- An unfriendly and confusing API when you want to nest routes. This is because the routes are defined within the component that contains them. This makes it difficult to read and maintain. Because instead of seeing the entire route tree, you must navigate through each component to see the routes it contains.
- When you need to do lazy loading it is confusing since it is done by adding the import of the component inside a function that is executed when you try to access it, when you would expect a declaration similar to how they do it in `angular`, `vue` or similar.
- An API that makes programmatic navigation difficult.

Currently, the most popular package for routing our client applications is `@vaadin/router`. This package has an intuitive API and is easy to use. However, it has limitations such as the point three of the previous list.

The proposed solution is to create a new API that incorporates the best features of popular libraries, but with the ability to define routes declaratively.

## Detailed design

The new API will be based on popular libraries. The idea is to create an API that will contain all the mentioned features and that will also be compatible with the existing one.

> [!IMPORTANT]  
> Below I share a repository where you can find an implementation of what would be the new API. https://github.com/thebug404/lit-router. In it you can find a more detailed explanation of the API. There is a very cool DEMO. To enjoy!

### Basic usage

```ts
// Import package
import { Route } from '@lit-labs/router'

// Import your Pages/Views
import { HomePage } from './pages/home-page.js'
import './pages/about-page.js'

// Define your routes
const routes: Route[] = [
  { path: '/', component: HomePage },
  { path: '/about', component: 'about-page' },
  {
    path: '/terms',
    component: () => import('./pages/terms-page.js').then((m) => m.TermsPage)
  }
]

// Get a router
const router = document.querySelector('lit-router')

// Register your routes
router.setRoutes(routes)
```

**HTML**

```html
<body>
  <lit-router></lit-router>
</body>
```

### Dynamic Routes Matching

`Lit Router` also provides a simple API to define dynamic routes. We can define dynamic routes of many types. Below are some examples:

```ts
import { UserPage } from './pages/user-page.js'

const routes: Route[] = [
  { path: '/users/:id', component: UserPage }
]
```

> **Info**
> The API used to define dynamic route is [URLPattern](https://developer.mozilla.org/en-US/docs/Web/API/URLPattern).

### Lazy Loading

Lazy loading is a technique for loading pages on demand. This means that we only load the page when the user navigates to it. This is very useful when we have a large application and we want to improve the performance of our application. Here's an example:

```ts
const routes: Route[] = [
  {
    path: '/',
    component: () => import('./pages/home-page.js').then((m) => m.HomePage)
  }
]
```

### Nested routes

Many times we need to define nested because we have a page that has a sidebar and we want to render the content of the sidebar in a specific place. For this we can use the `children` property. Here's an example:

```ts
const routes: Route[] = [
  {
    path: '/',
    component: DashboardPage,
    children: [
      { path: '/settings', component: SettingsPage }
    ]
  }
]
```

### Displayed 404 Page

To display a 404 page, we can define a route with the `*` path. Here's an example:

```ts
const routes: Route[] = [
  { path: '/', component: HomePage },
  { path: '/about', component: AboutPage },
  { path: '*', component: NotFoundPage }
]
```

### Programmatic Navigation

`Lit Router` also provides a simple API for programmatic navigation. You can use the `navigate` method to navigate to a specific route. Here's an example:

```ts
import { router } from './index.js'

// Navigate by path
router.navigate({ path: '/about' })
```

#### Navigate for history

You can also use the `forward` & `back` method to navigate for history. Here's an example:

```ts
import { router } from './index.js'

// Navigate forward
router.forward()

// Navigate back
router.back()
```

#### Navigation utilities

`Lit Router` also provides some utilities to help you with navigation. Here's an example:

```ts
import { navigate, forward, back } from '@lit-labs/router'

// Navigate by path
navigate({ path: '/about' })

// Navigate forward
forward()

// Navigate back
back()
```

### Query & Params

`Lit Router` also provides a simple API to get the `query` and `params` of the current route. Here's an example:

```ts
import { router } from './index.js'

// Get all queries. Example: /users?name=Ivan&age=23
router.qs() // { name: 'Ivan', age: '23' }

// Get a specific query. Example: /users?name=Ivan&age=23
router.qs('name') // Ivan

// Get all params. Example: /users/1
router.params() // { id: '1' }

// Get a specific param. Example: /users/1
router.params('id') // 1
```

### Guards

The guards are functions that are executed before entering a route. They are very useful when we want to validate that the user has the necessary permissions to enter a route. Here's an example:

```ts
// Define your guard
const isAdminGuard = (router) => {
  const user = localStorage.getItem('user')

  if (user && user.role === 'admin') return true

  router.navigate({ path: '/login' })

  return false
}

const routes: Route[] = [
  {
    path: '/admin',
    component: AdminPage,
    // Execute the guard before entering the route
    beforeEnter: [isAdminGuard]
  }
]
```

Or you can use destructuring to get the router. Here's an example:

```ts
const isAdminGuard = ({ navigate }) => {
  const user = localStorage.getItem('user')

  if (user && user.role === 'admin') return true

  navigate({ path: '/login' })

  return false
}
```

## Implementation Considerations

### Implementation Plan

Is there anything important to note about implementation plan? Can it be done in a single PR or will it need to be staged out across several?

### Backward Compatibility

Since it is a new API to router our SPA applications, there will be no compatibility problems since it is an addition.

### Testing Plan

How will this proposal be tested? Are unit tests sufficient, or do we need integration tests? Is any unique testing infrastructure required?

### Performance and Code Size Impact

What impact will this proposal have on performance and code size? What benchmarks should we create to evaluate the proposal?

### Interoperability

Is this proposal for a feature that could be interoperable across web components not written in Lit? Does it create a tight coupling between components written in Lit? Could it be a [Web Components Community Group](https://github.com/w3c/webcomponents-cg) [Community Protocol](https://github.com/webcomponents-cg/community-protocols)?

### Security Impact

- `Route Guards`: Implementing faulty route guards could allow unauthorized access to certain parts of the application.
- `Lazy Loading`: Lazy loading of modules could introduce vulnerabilities if not handled properly.

### Documentation Plan

Due to the simplicity (but powerful) of the API, a README file will suffice.

## Downsides

By significantly modifying the API, it will be necessary for users to migrate the definition of their routes to this new proposal. Likewise, it is not mandatory since they can use the existing version.

Although if you want more features, more intuitive API and a lower learning curve, it is recommended to use this new proposal.

## Alternatives

What alternatives were considered and rejected? Why?
