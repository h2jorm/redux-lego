# redux-lego
Redux-lego is a simplified building strategy of redux app. Like lego, redux could be composed from redux bricks.

## redux brick
Every redux brick is an object with three specific attribute names.
* name: brick name
* defaultState: default state of the redux brick
* mutation: a generator function wrapping redux action creator and reducer together

Here are two examples of redux brick.
```js
// src/store/countApp.js
module.exports = {
  name: 'countApp',
  defaultState: {
    count: 0
  },
  mutation: {
    add: function *() {
      yield type => {
        return () => ({type});
      };
      yield (state, action) => {
        return Object.assign({}, state, {
          count: state.count + 1
        });
      };
    }
  }
};

// src/store/todoApp.js
module.exports = {
  name: 'todoApp',
  defaultState: {
    todos: []
  },
  mutation: {
    add: function *() {
      yield type => {
        return todo => ({
          todo, type
        });
      };
      yield (state, action) => {
        return Object.assign({}, state, {
          todos: [
            ...state.todos,
            action.todo
          ]
        });
      };
    }
  }
};
```
Here is the benefits from decomposing the whole root redux into small bricks.

1. Redux brick helps to save the declaration of lots of constants to make sure the unique of action type. Instead, just take care of brick name.
2. Considering the fact that almost every action has its counterpart in reducer, action creators and reducer should be in same file for a more efficient coding. Leave no more separated actions and reducers and save two directories and a lot of files.

## compose
`compose` function from `redux-lego` helps to compose redux. It takes redux bricks as arguments and returns an object containing `actions` and `reducer`. Then it will go back the traditional redux way to apply middle wares, create store and so on.

Here is an example using two bricks declared before.
```js
// src/store/index.js
import {
  createStore,
  combineReducers,
} from 'redux';
import {compose} from 'redux-lego';

import countApp from './countApp';
import todoApp from './todoApp';

// compose bricks
const {actions, reducer} = compose(
  countApp,
  todoApp
);

// for now, `reducer` is an object.
// Use `combineReducers` to convert it to a valid redux reducer.
const store = createStoreWithMiddleware(
  combineReducers(reducer)
);

// dispatch some actions
store.dispatch(actions.countAppAdd());
expect(store.getState().countApp.count).toBe(1);// true

// the mutation name scope are isolated in different bricks
const newTodo = {
  title: 'hello',
  done: false
};
store.dispatch(actions.todoAppAdd(newTodo));
expect(store.getState().countApp.count).toBe(1);// true
expect(store.getState().todoApp.todos.length).toBe(1);// true
expect(store.getState90.todoApp.todos[0]).toEqual(newTodo);// true
```
After composing bricks, `actions` and `reducer` will be generated. `redux-lego` only provides a sugar wrapper and do not break into the structure of origin redux. `actions` and `reducer` are still combined by a unique action name, but maintained automatically instead of declared in a long list of constants.

A unique action name is composed by two parts.
* a unique redux brick name
* a mutation name

`type` is the reserved attribute name of a redux action where a unique action name recorded.

So, for example, mutation `add` in `countApp` will be `countAppAdd`. Mutation `add` in `todoApp` will be `todoAppAdd`. Every brick has its own scope and will not disturb with each other.