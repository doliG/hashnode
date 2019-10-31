## Event based communication between micro frontends

## Summary

- [What is it?](#what-is-it-)
- [How it works?](#how-it-works-)
- [How to use?](#how-to-use-)
  * [App side: Creating and dispatching event](#app-side-creating-and-dispatching-event)
  * [Analytics side: Catching and processing events](#analytics-side-catching-and-processing-events)
- [Under the hood](#under-the-hood)

## What is it?

The problem: When you have multiple application, such as in micro front-end, you need to find a way to communicate between them.

This is an **event based** proof of concept (POC), of how three application and an analytics layer can communicate.

This had been inspired by [MicroFrontend](https://micro-frontends.org/) and [Eev](https://github.com/chrisdavies/eev)

![monolith-frontback-microservices.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1572258097832/HKKluEQ7n.png)

## How it works?

There is three applications:
- `Header app`
- `Product app`
- `Log app`

And there is on analytics layer called `tracking`

> Note: I simplified it for demo purpose. In a full micro front-end app, you'll probably have more apps like basket, checkout, catalog...

The tracking layer expose an object called `TrackingEvent` which will be dispatched.

The tracking layer is responsible to catch and handle ALL dispached trackingEvents, and sync it with the analitycs side.


## How to use?

### App side: Creating and dispatching event

When you want to track an event, you have to:
1. Create a new TrackingEvent and providing it the data you need
2. Dispatch your custom event

Example:

```js
const type = trackingEventName;
const payload = { your: 'datas', goes: 'here' };

const e = new TrackingEvent(type, payload);
e.dispatch();
```

Easy, isn't it ?

### Analytics side: Catching and processing events

It works this way:

```js
document.addEventListener('trackingEvent', function (event) {
  /**
   * Process the event here.
   * Please note that event is the CustomEvent object.
   * Your data are stored into event.detail
   */
  const { type, payload } = event.detail;
  
  axios
    .post('analytics-api' { type, payload })
    .then(res => /* */)
    .catch(err => /* */);
});
```

## Under the hood

The `TrackingEvent` object is just some syntaxic sugar added on top of [CustomEvent](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Creating_and_triggering_events#Adding_custom_data_%E2%80%93_CustomEvent) api.

Its main purpose is to simplify the developer by abstracting the not so complicated CustomEvent api and reducing boilerplate.

It does two things:

- Wrap your data into one object (by convention, we have a `type` and a `payload`, juste like redux store pattern),
- Create a CustomEvent, set your customData into this event, and dispatch it.


When you catch a CustomEvent, you will need to access to event.details key to retrieve the data you store in the custom event.

And because sometimes, code is more expressive than word, check this out:

```js
class TrackingEvent {
  constructor(type, payload) {
    this.type = type;
    this.payload = payload;
  }

  dispatch() {
    const { type, payload } = this;
    const dispatchedAt = new Date();
    const detail = { type, payload, dispatchedAt };
    const event = new CustomEvent('trackingEvent', { detail });
    document.dispatchEvent(event);
  }
}
```

---

Want to know more ? Feel free to follow the project or read the source code on [GitHub](https://github.com/doliG/poc_events) .

Cover photo by Kyle Hinkson on Unsplash.