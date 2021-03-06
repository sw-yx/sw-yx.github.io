---
layout: post
date: 2018-04-14
tags: react
feelings: happy
title: react amsterdam notes
comments: true
description: notes rom amsterdam
---

Schedule

- [Tracy](#tracy): Reactive Programming Demystified
- [Michele](#michele): setState Machine 
- [Michel](#michel): There and back again: grokking state and data
- [Shirley](#sxywu): D3 and React, Together
- [Lightning talks](#lightning)
- [Kitze](#kitze): React State Management In a GraphQL Era
- [Richard](#richard): GraphQL at scale with AWS
- [Manjula](#manjula): Rethinking With React 16
- [Ken](#kanye): Mixed Mode React

---

<a id="tracy"></a>

---

# Tracy Lee: Reactive Programming Demystified

increasing adoption of reactive programming

- promises considered reactive??
- TC39 Observable proposal - added in apr 30 2015
- rxjs is at 8m monthly downloads now
- WHATWG Event Target Observable psoposed dec 12 2017  eg `button.on('click')` gives you an observable <https://github.com/whatwg/dom/issues/544>

Libraries

- RxJS: DSL for reacting to events, most popular FRP lib
- d3: reacting to the propagation of change - you describe what happens and it just happens under the hood
- react: reactive programming doesnt have to be pull based. calling `setState` causes reactions
- redux-observable
- mobx
- angular: rxjs is first class citizen.
  - onPush change detection strategy
  - ngrx
  - angular router
  - forms
- vuejs watchers

We see reactive programming patterns in cases where there is natrual fit for events to be modeled as values over time

- web sockets
- user events
- animations
- http

How to start thinking reactively?

- think of modeling everything as an event
- web apps are event driven
- everything can be represented as *a set of values* **over time**

code examples:

- drag and drop in react with a few lines of rxjs
- adding inputs and chaining with little refactoring in angular
- merging google vision + web speech + typed input to pipe into a search

No need to Rx all the things. Easy wins:

- click handlers
- ajax requests (easy cancellation)
- anything async

---

<a id="michele"></a>

---

# Michele Bertoli: setState Machine 

State is hard

- have to keep track of it over time
- make sure users can only perform valid operations

eg. Infinite Scroll

- attach event handler to scroll
- if scroll > 90% of page
- fetch data and append to end

problems

- keep track of whether you're fetching: `this.state.isFetching`
- no way to stop fetching if `data.hasMore` is falsy
- dealing with errors - `this.state.hasError`
- retry functionality
- nested ternaries of state flags

this is called the bottom-up approach

- focusing on linear transitions
- turns out, you need context
- edge cases can come from combinations of state flags that are impossible

solution: state machines!

- we want to constrain ourselves from infinite state turing machines to deterministic finite automata
- <Q: all states, Sigma: all inputs, delta: f(input) => state, q0: inital, F: final>
- visualize with state transition diagrams

If too many states, use Harel statecharts!

- clustering - wrapping shared transitions
- orthogonality - indepedent states
- guards - conditional blocking of transitions
- actions: side effects of entry/exit events

workflow: event -> statechart (evaluates guards etc) -> actions

"A statechart is a magic box; you tell it what happened, it tells you what to do" @lmatteis

Code example: 

- `yarn add xstate` :) i know this already
- `react-automata` - layer on top of xstate to work with it in React. 
  - infinite scroll example:
  - start -> ready -> fetching
  - fetching - SUCCESS[hasMore] - listening
  - listening - SCROLL[scrollPercentage>0.9] - fetching
  - listening - ERROR - fetching
  - fetching - SUCCESS[!hasMore] - finish
  
Workflow when using `react-automata`

- send transition (event, data)
- react-automata calculates
- react-automata fires action methods for you

React-automata API:

- `withStatechart(statechart, {devTools: true})(MyComponent)` connects to redux dev tools
- `<Action>` component shows on condition!! very nicely declarative!
- `testStatechart({statechart}, MyComponent)` tests all the paths in statechart at once for free, taking snapshot

Benefits

- fewer bugs (80% less bugs when A/B tested at facebook)
- easy to understand
- separating what happens (in component) vs when it happens (in statechart)

sample: <http://bit.ly/automata-calculator> no if statements or state flags

takeaways:

- think about states more than transitions. redux encourages you to think about actions that mutate your states, xstate tells you to think about states first
- this is a top down approach
- downside is youre goign to spend a lot of time thinking about how your components work before writing them
- read more papers! :)
- <bit.ly/statecharts-book>
- <bit.ly/statecharts-paper>

---

<a id="michel"></a>

---

# Michel Weststrate: There and back again: grokking state and data

Why is state management hard: 

- Organizing state
- Updating state
- Distributing changes

You can choose to treat JS object as a value or an entity.

Object as value 

- to represent a fact, immutable

Object as entity

- represent something stateful, changes over time

Object properties: composition vs association

- composition: composes dependent entities, proper life cycle handling - eg in React component tree
- association: relates 2 independent entities
- derivation: aggregations and transformations of existing state

Distributing Change

- 2 ways to do it: manually `entity.name="frodo; this.forceUpdate()` or with subscriptions `entity.on('change', callback)` (or transparently "just using the data" eg in mobx)

React's problem - no association with "outside state" eg association with sibling components, so have to pull up state

- Redux allows your state to live outside component tree - one store, event emitter.
- downsides of Redux - only one update = global update so need to use sCU/reselect, hard to compose/use references. Have to manually map values onto entities

What Immer solves

- State -> Draft -> Next state

Comparison with Rx

- streams = entities, they capture state
- easily reason about the past
- you dont ask for current value, you subscribe to the values

MobX

- turns all objects into entities
- you can produce beautiful derivation trees from them

MobX State tree

- adds structure to mobx
- react for data
- composition & association - transparently reactive

How can these state mgmt approaches make your life easier?

- do you mainly reason about events that happened? => values
- ... or the present? => identities
- can you derive your information? better than introducing new information
- consider that entities change over time, values might become irrelevant over time
- can the receiver subscribe to changes?
  - yes: send identity - eg smart redux container comopnent, mobx @observer
  - no: create subscription, send value on each change - eg pass plain props/data

his advice

- make every field either read only or observable

Applying this to React: context vs sCU

---

<a id="sxywu"></a>

---

# Shirley Wu: D3 and React, Together

Where D3 conflicts with React: when it needs the dom

- transitions
- axes
- brushes

D3 doesnt need the DOM for:

- layouts
- geo
- shape

### Why

For most of our use cases, we can have d3 just doing calcs, and react doing rendering. Only when you need transitions/axes/brushes do you have d3 doing both calc and rendering.

- film flowers: only needs a click interaction
- travel photos: only a hover interaction -> shows details

D3 is more than enough for just simple interactions. But looking at more complex viz:

- expense app: drag to link dates and expense, creation
- brainsights: linking multiple viz with brushes
- hamilton: persistent dots: manage scroll and state of final exploratory tool

use React + d3 to componentize viz and manage state as user interacts.

### How

Weather data example

LIVE. CODING.

- use cWRP to get nextProps
- update scales, and pass it into the `domain()` function
- re-form the lineGenerator using the new `xScale` and `yScale`
- setState with the lineGenerator result
- inside `render()` -  pass it into `<path d={this.state.highs} {...} />`

More? 

More.

- add axes!
- inside cDU: call `d3.select(this.refs.xAxis).call(this.Axis)` <- not sure what this does
- `<g ref='xAxis' transform{'translate(0, ${height - margin.bottom})}/>`

More?

More.

- bar charts!
- same process, calculate from scale, and then plot inside an `svg` tag inside `render`.
- `this.state.bars.map(...)`
- throw colorScale on and put it into the `bars` state as a `fill`

More? 

More.

- radius scale chart!
- create a radiusScale and arcGenerator (`d3.arc()`)
- `const perSliceAngle = (2 * Math.PI) / data.length`
- use `arcGenerator({startAngle: ..., endAngle: ..., innerRadius: ..., outerRadius: ...})`



---

<a id="lightning"></a>

---

# Lightning talk 1: Structure Your App's Story With Sagas and Selectors (Rebecca Hill)

using selectors/ reselect

- contain all knowledge about state shape in reducers and components only , refactoring becomes easier

conditionals

- action creators/ redux-thunk

demo using games

- redux-saga : uses generator functions

whole file of selectors

main takeaway: decouple biz logic from rest of application

# Lightning talk 2: How to “Reactify” Your Existing UI Components (Olga Petrova)

Wrapping ExtJS Component with a React Wrapper

- i missed most of this haha sorry
- sencha repo

# Lightning talk 3: Inclusive React: A Survival Guide (Almero Steyn)

Accessibility tips

- dont do `outline: 0` or outline: none
- use `label` with your `input`
- use `buttton` with `aria-`
- use html `button` not a `div`
- use `a href` instead of `a onClick`
- semantic html: use `<nav>`, `<aside>`, `<button>`, not just div tag
- good contrast

resources:
- <https://reactjs.org/docs/accessibility.html>
- <https://webaim.org/standards/wcag/checklist>
- <http://www.w3.org/TR/wai-aria-practices-1.1>
- use Semantic HTML where possible <http://developer.mozilla.org/en-US/docs/Web/HTML/Element>

a11y tests for your app:

- navigate by tab, enter, and arrow keys, no mouse
- eslint-plugin-jsx-a11y
- <http://www.deque.com/axe> runs a11y reports - can also just use react-axe: <http://github.com/dequelabs/react-axe>
- screen reader tests


---

<a id="kitze"></a>

---

# Kitze: React State Management In a GraphQL Era

Data is #1 reason why state management is hard

- data fetching
- caching
- reading
- cache invalidating

love-hate redux

on to mobx

thought leading 101

comparison of redux vs mobx vs apollographql - apollo has less code

basic stacks

- apollo/router/setstate
- apollo/router/setstate/form library
- apollo/router/apollo-cache-persist
- apollo/mobx/mobx-react-form/mobx-router
- apollo/nextjs/setstate
- next level - full page reloads? every route a mini app
- apollo/apollo-link-state

before you follow someone's advice, evaluate the context and your situation

Don't be scared of the network, dont overengineer.

Don't feel bad for not using latest & greatest

Delete twitter


---

<a id="richard"></a>

---

# Richard: GraphQL at scale with AWS

AWS AppSync
AWS Amplify

Hello world with GraphQL

What comes after hello world?

define schema on AWS AppSync

trend: inspecting key schema in database ,autogenerating common graphql

amplify is category based way of programming

images and rich content

create s3object butcket and key - both writes and reads

graphql subscriptions - for realtime updates

goal: start cost effectively - use graphql to swap out from dynamodb to elastic

**what else comes after hello world??**

today appsync is live! new features

- test/debug resolver flows
- cloudwatch
- cloudformation
- subscription resolvers
- batch delete/read/put on individual resolvers-> IOT use case
- AWS ampllify graphql
- awsmobile cli + graphql
- HIPPA compliance

some demos of the interface. very clean.



---

<a id="manjula"></a>

---

# Manjula: Rethinking With React 16

a recap of the react 16 changes

- fragments
- error boundaries
- new context api
- portals
- new refs api


---

<a id="kanye"></a>

---

# ken: Mixed Mode React

last year: [Fiber Renderers](https://www.youtube.com/watch?v=oPofnLZZTwQ)

- custom react renderers
- you can do an emoji renderer
- react synth

what if you want to render two things?

- two renders, not good

renderless components

- ui doesnt just mean ui
- non dom UI
  - web audio
  - canvas
  - webgl
  - non dom api's
  
simple example

- `<Redirect />` component
- `<Log />` component

intermediate example

- d3 example

renderless component checklist

- didnt catch it

complex renderless components

- arbitrarily nested sub components
- relationally nested sub components
- multi component non dom composition

writing declaratively - using context (best part of React API)

- theme example of new context API
- you can pass antyhing in context, and you can pass it up too!

registration pattern: 

- initialize module in top level
- pass function to children w context
- children use the function to register with the parent
- child component calls `this.props.register(uuid)`

registration pattern 2:

- initalize in top level
- register callback w parent from the child
- call the callback
- parent calls the child's callback

demo example

- canvas renderer with react
- translating declarative api with imperative one
- with fallback input 

demo 2 - threejs

- clicking balls

mixing the demos

- bringing in the `react-music` api
