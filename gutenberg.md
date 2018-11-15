---
title: Gutenberg
category: React
layout: 2017/sheet
updated: 2018-01-17
weight: -3
---

### Creating a store

```js
import { registerStore } from '@wordpress/data';
```
{: .-setup}

```js
// Reducer
function counter ( state = { value: 0 }, action ) {
  switch ( action.type ) {
  case 'INCREMENT':
    return { value: state.value + 1 };
  case 'DECREMENT':
    return { value: state.value - 1 };
  default:
    return state;
  }
}
```

```js
let store = registerStore( 'counter', {
  reducer: counter,
} );
```

A store is made from a reducer function, which takes the current `state`, and
returns a new `state` depending on the `action` it was given.

### Using a store

```js
import { registerStore } from '@wordpress/data';
let store = registerStore( 'counter', {
  reducer: counter,
} );
```
{: .-setup}

```js
// Dispatches an action; this changes the state
store.dispatch({ type: 'INCREMENT' })
store.dispatch({ type: 'DECREMENT' })
```

```js
// Gets the current state
store.getState()
```

```js
// Listens for changes
store.subscribe(() => { ... })
```

### The better way

```js
import { registerStore, dispatch } from '@wordpress/data';
registerStore( 'counter', {
  actions: {
    increment: () => ( { type: 'INCREMENT' } ),
    decrement: () => ( { type: 'DECREMENT' } ),
  },
  reducer: counter,
  selectors: {
    getValue: state => state.value,
  }
} );
```
{: .-setup}

```js
// Dispatches an action; this changes the state
dispatch( 'counter' ).increment();
dispatch( 'counter' ).decrement();
```

```js
// Gets the current state using a selector
select( 'counter' ).getValue();
```

```js
// Listens for changes
subscribe( () => { /* no way to get the state from here, can use the object returned by `registerStore` */ } )
```

Dispatch actions to change the store's state.

## React Gutenberg

```js
registerStore( 'events', {
  actions: {
    click: ( message ) => ( { type: 'CLICK', message } ),
  },
  reducer: ( state = { value: 0, message: '' }, action ) {
    switch ( action.type ) {
    case 'CLICK':
      return { value: state.value + 1, message: action.message };
    default:
      return state;
    }
  },
  selectors: {
    getMessage: state => state.message,
  }
} );
```

### Mapping state

```js
import { withDispatch, withSelect } from '@wordpress/data';
```
{: .-setup}

```js
// A functional React component
function App ({ message, onMessageClick }) {
  return (
    <div onClick={() => onMessageClick('hello')}>
      {message}
    </div>
  );
}
```

```js
// Maps `state` to `props`:
// These will be added as props to the component.
function mapState ( select ) {
  const { getMessage } = dispatch( 'events' );
  return {
    message: getMessage(),
  };
}

// Maps `dispatch` to `props`:
function mapDispatch ( dispatch, ownProps ) {
  const { click } = dispatch( 'events' );
  return {
    onMessageClick: click,
  };
}

// Connect them:
export default withDispatch( mapDispatch )( withSelect( mapState )( App ) );

// Equivalent to:
import { compose } from '@wordpress/compose';
export default compose( withSelect( mapState ), withDispatch( mapDispatch ) )( App );
```

### Shorthand

```js
export default compose(
  withSelect( ( select ) => {
    const { getMessage } = dispatch( 'events' );
    return {
      message: getMessage(),
    };
  } ),
  withDispatch( ( dispatch ) => {
    const { click } = dispatch( 'events' );
    return {
      onMessageClick: click,
    };
  } )
)( App );
```

Same as above, but shorter.
