# Truly painless UI state for components.

### Install

```
npm i react-redux-ui-state -S
```

```
yarn add react-redux-ui-state
```

## Goals of this library

* Make it easy to have component state in Redux
* Make it possible to have "internal" state with or without using classes
* Reduce Redux boilerplate for properties like UI flags
* Easily allow multiple instances of the same component to get their own unique
  state
* Get the benefits of Redux without most of the work for UI state

## How it works

* Create a config object with an optional name property and a function (which
  gets passed props) which returns a state object consisting of your initial
  state.
* Wrap your component in the HOC.
* An action gets dispatched like `UI_STATE_ADD:some-component_<uuid>` on
  `componentWillMount`
* The initial state gets added to a `uiState` slice of the store with the
  name/state.
* Call `setUiState` with a new state object.
* An action gets dispatched like `UI_STATE_SET:some-component_<uuid>`
* Component state gets removed from store on `componentWillUnmount` (optional)

`setUiState`: Simple update function that accepts an object that will be shallow
merged with current state. Accepts an optional callback called with the state
change.

`resetUiState`: Function to reset component to initial state. Accepts an
optional callback called with the initial state.

## uiState Options

#### `name {String | Function}`

If a string is provided a uuid will be suffixed in order not to overwrite the
state in the store. Providing a function gives you the ability to create your
own unique name, with the ability to use the component's `props`, e.g:

```js
const uiStateConfig = {
    ...
    name: props => `MyComponent__${props.className}`,
    ...
};
```

#### `state {Function}`

Extend the wrapper component's props, e.g.

```js
const uiStateConfig = {
    ...
    state: (props) => ({
        uiProp: 'uiProp',
        uiActive: props.active,
    }),
    ...
};
```

#### `persist {Boolean}`

Defaults to `false`. If `true` the component's state will not be removed from
the `uiState` domain.

```js
const uiStateConfig = {
    ...
    persist: true,
    ...
};
```

## Usage:

```javascript
// rootReducer.js
import { combineReducers } from 'redux';
import { uiStateReducer } from 'react-redux-ui-state';

export default function createRootReducer (reducers) {
    return combineReducers({
        ...reducers,
        uiState: uiStateReducer,
    });
}

// someComponent.js
import React from 'react';
import { uiState } from 'react-redux-ui-state';

export function someComponent ({
    someUiFlag,
    somePassedThing,
    resetUiState,
    setUiState,
    uiStateName,
}) {
    return (
        <button onClick={() => setUiState({
                    someUiFlag: !someUiFlag
                },
                updatedState => console.log(updatedState),
            )}>
            Toggle flag for {uiStateName}
        </button>

        <button
            onClick={() =>
                resetUiState(resetState => console.log(resetState))}
        >
            reset state
        </button>
    );
}

const uiStateConfig = {
    name: 'some-component',
    persist: true, // optional
    state: (props) => ({
        someUiFlag: false,
        somePassedThing: props.x ? props.x : null;
    }),
};

export default uiState(uiStateConfig)(someComponent);
```

## Example app

There is a simple example app made with `create-react-app` in the `./example`
folder. Simple `cd` into that directory, `yarn install && yarn start`. You can
edit the `App.js` file to add more instances of the `Button` and `Text`
components.

![App screenshot](example-app.png)

Feel free to open a PR!
