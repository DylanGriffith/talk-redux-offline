# Redux Offline

---

# What is Redux?

Redux is a predictable state container for JavaScript apps.

Works with:
  - ReactJS
  - React Native
  - Angular 2
  - Probably any JS app

---

```javascript
import { createStore } from 'redux'

function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT': return state + 1
  case 'DECREMENT': return state - 1
  default: return state
  }
}

let store = createStore(counter)

store.subscribe(() =>
  // Re-render your views...
  console.log(store.getState())
)

store.dispatch({ type: 'INCREMENT' })
store.dispatch({ type: 'DECREMENT' })
```

---

# Redux: The Good

- Easier to understand how UI state is modified (only by dispatching)
- Single state for frontend app makes it easier for multiple UI changes to be applied when the state changes.

---

# Redux: The Bad

- Javascript doesn't enforce immutability so new developers can easily make mistakes that lead to confusing behaviour.
- Some immutable updates can be messy code and possibly really innefficient
- ImmutableJS solves(?) the above, however is yet another API to learn

---

# Example

```javascript

// Reducer
export default function reducer(state = {widgets: []}, action = {}) {
  switch (action.type) {
    case 'CREATE': return {
      ...state,
      widgets: [...state.widgets, action.data.widget]
    }
    default: return state;
  }
}

// Action Creators
export function createWidget(widget) {
  return { type: 'CREATE', data: { widget } };
}
```

---

# What about side effects

### Thunk:

```javascript
export function createWidget(widget) {
  return (dispatch) => {
    fetch('/widgets', method: 'POST', body: JSON.stringify(widget))
      .then(() => { dispatch({ type: 'CREATE', data: { widget } }) })
      .catch(() => { dispatch({type: 'CREATE_FAIL'}) })
  }
}
```

---

# What about optimistic UI Updates for slow connections?

---

```javascript
export function createWidget(widget) {
  return (dispatch) => {

    dispatch({ type: 'CREATE', data: { widget } }) // <====

    fetch('/widgets', method: 'POST', body: JSON.stringify(widget))
      .then(() => { dispatch({ type: 'CREATE_SUCCESS' }) })
      .catch(() => { dispatch({type: 'CREATE_FAIL' }) })
  }
}
```

---

# Problems

- Thunk API is kinda hard to get your head around. (ie. function returning function that gets passed functions. Why?)
- Not resilient to network errors (ie. no retries)
- Need to be online. If you want an app that works offline you'll need to figure out how to do that
- Turned our easily testable pure function into some hard to test impure function

---

# Enter Redux Offline

```javascript
export function createWidget(widget) {
  return {
    type: 'CREATE',
    data: { widget },
    meta: {
      offline: {
        effect: { url: '/widgets', method: 'POST', body: { widget } },
        commit: { type: 'CREATE_SUCCESS' },
        rollback: { type: 'CREATE_FAIL' },
      },
    }
  };
}
```

---

# Why Redux Offline

- Now we've got a pure function again
- Network resilience because the `effect` is automatically retried by `redux-offline`
- Works offline because `redux-offline` will only make requests when you have an internet connection.
- Resilient to somebody closing the tab due to all state being persisted to localStorage (in theory)
- Works with web and mobile (ie. React Native)

---

# Idempotence

What is it?

---

# Not Idempotent

```
POST /widgets
```

```
def create
  Widget.create(params[:widget])
end
```

---

# Idempotent

```
def update
  widget.find_or_create_by(uuid: params[:uuid])
  widget.update(params[:widget])
end
```

---

# Idempotent

```javascript
export function createWidget(widget, uuid) {
  return {
    ...
    meta: {
      offline: {
        effect: { url: `/widgets/${uuid}`, method: 'PUT', body: { widget } },
        ...
      },
    }
  };
}
```

---

# Smaller Updates (even better)

```javascript
export function updateWidgetTitle(title, uuid) {
  return {
    ...
    meta: {
      offline: {
        effect: { url: `/widgets/${uuid}`, method: 'PATCH', body: { title } },
        ...
      },
    }
  };
}
```

---

# Considerations

- Redux Offline is very new (and maybe not really well maintained)
- Quite a few open issues that have received few responses
- Redux Offline is mainly built for side effects (creates/updates) but that pattern would actually generalise nicely to fetching data too
- Redux Offline codebase is incredibly simple and could be forked if something doesn't work for you.

---

# Functional Programming

- Redux Offline demonstrates how to write purely functional code but still "do something"
- Testing is a breeze
- This API actually looks a lot like Elm

---

# Links

https://github.com/jevakallio/redux-offline

https://github.com/erikras/ducks-modular-redux
