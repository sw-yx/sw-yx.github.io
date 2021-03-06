---
layout: post
date: 2018-07-27
tags: react
feelings: neutral
title: reactrally grind 1
comments: true
description: prep for talk
---

i am in hardcore grind mode now for the talk. first is to flesh out reactive react.

what we need from zpao https://www.youtube.com/watch?v=_MAD4Oly9yg

- createElement
- Component
- render

within Component

- constructor
- render
- setState
- this.props
- this.state

---

also reviewd pomber's writeup https://engineering.hexacta.com/didact-instances-reconciliation-and-virtual-dom-9316d650f1d0

---

also evan's talk on many flavors of FRP https://www.youtube.com/watch?v=Agu6jipKfYw


---

had to take a nap


right now the main task is to make didact use zen-observable so that i am reactive by default. i start off with the pure element creation bit: https://engineering.hexacta.com/didact-element-creation-and-jsx-d05171c55c56

---

3am. finally made some headway - wrapping addEventListener into a fromEvent:

```js
function fromEvent(el, eventType) {
  return new Observable(observer => {
    el.addEventListener(eventType, e => observer.next(e))
    // on unsub, remove event listener
    return () => console.log('not implemented yet')
  })
}
```

this lets me make change handlers into streams:

```js
function LabeledSlider() {
  return <input type="range" min={20} max={80} value={state} 
  onInput={e$ => {
    e$.subscribe(e => {
      state = e.target.value;
      console.log(state)
      render(appElement(), document.getElementById("app"));
    })
  }}/>
}
```

but now i have to think about what `setState` means when we are talking about streams.


---

4am i have found a perf issue too - you cant just subscribe at the point of render - you have to create the stream at instatiation or you will have multiple streams.

heres the reconciler:

```js
  // Add event listeners
  Object.keys(nextProps).filter(isEvent).forEach(name => {
    const eventType = name.toLowerCase().substring(2);
    // dom.addEventListener(eventType, nextProps[name]);
    // nextProps[name](fromEvent(dom, eventType))
    const stream = fromEvent(dom, eventType)
    stream.subscribe(nextProps[name])
    dom[name + '$'] = stream
  });
```

and usage

```js
function LabeledSlider() {
  return <input type="range" min={20} max={80} value={state} 
  onInput={e => {
      state = e.target.value;
      console.log(state)
      render(appElement(), document.getElementById("app"));
  }}
  />
}
```

this causes multiple subscriptions, bad.

this may mean i have to go to class based components sooner than planned.

4.05 lol i solved it by storing the stream on the dom

```js
if (!dom[name + '$']) { // note - bug if handler mutates
      const stream = fromEvent(dom, eventType)
      stream.subscribe(nextProps[name])
      dom[name + '$'] = stream
    }
```
