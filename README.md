# Learning Redux
Notes from Dan Abramov's lesson on Redux: https://egghead.io/series/getting-started-with-redux

Great to learn for users of React, and now Angular 2 and Ember.

## Pure vs Impure Functions
Some of the functions you write in Redux have to be pure and you have to be mindful of that.

**Pure Functions** - Return values solely based on the arguments given to the function instead of having any observable side effects such as network and database calls. They also do not the value passed from them. It returns a new variable. Examples:

    function square(x) {
        return x * x;
    }

    function squareAll(items) {
        return items.map(square);
    }
**Impure Functions** - May call the database or network, have side effects, operate on the DOM, and override the values passed on to them. Examples:

    function square(x) {
        x = x * x;
        return x;
    }

    function squareAll(items) {
        items = items.map(square);
        return items;
    }

## Three Principals of Redux

1. Everything that changes in your app, including the data and the UI state, is contained in a single object that we call the **state** or **state tree**.

    **State** or **State Tree** - the whole state of your application:
    
        [object Object] {
            todos: [
                [object Object] {
                    completed: true,
                    id: 0,
                    text: “hey”
                }, [object Object] {
                    completed: false,
                    id: 1,
                    text: “ho”
                }],
            visibilityFilter: “SHOW_COMPLETED”
        }
2. The state tree is redundant, meaning you can not write or modify to the state tree. Anytime you want to change the state, you have to dispatch an action.

    **Action** - a plain javascript object describing the minimal change to the app. Only requirement is that the action have a “type” property that is not undefined. Recommendation that “type” property is a string value:
    
        [object Object] {
            type: “INCREMENT”
        }
3. To describe state mutations, you have to write a function called the **Reducer** that takes the previous state of the app, the action being dispatched, and returns the next state of the app.

    **Reducer** - describes the state mutation as a pure function. Takes the previous state and the action being dispatched as arguments and returns the next state of your application. An example of the return:

        [object Object] {
            filter: “SHOW_COMPLETED”,
            type: “SET_VISIBILITY_FILTER”
        }

## Writing a Reducer
If we dispatch an action that the reducer does not understand, it should return the previous state of the application. The reducer should handle all unknown actions. This is why the testing is so important. Also notice that if the state is not given, the initial is set to 0 in this example. The state can not be undefined, therefore this part is important.

    const counter = (state = 0, action) => {
      switch (action.type) {
        case ‘INCREMENT’:
          return state + 1;
        case ‘DECREMENT’:
          return state - 1;
        default:
          return state;
      }
    }

## Store Methods: getState(), dispatch(), and subscribe()
When using the external script for Redux with Webpack or Browserfy, we are given three store methods. Before using the methods, you want to define the store methods and the store itself with your reducer. In this example, we create the store with our counter reducer.

**Store** - Binds together the three principles of Redux. It holds the app’s state. It lets you dispatch actions. When you create it, you need to specify the reducer that tells how state is updated with actions.

    // createStore(reducer) binds your reducer to the new store
    import { createStore } from “Redux";
    const store = createStore(counter);
    
    // getState() binds UI to your state.
    const render = () => {
      document.body.innerText = store.getState();
    };
    
    // subscribe(whateverFunction) registers a callback that the Redux chore will call any time an action has been dispatched
    store.subscribe(render);
    render;
    
    // dispatch() dispatches an action so that the reducer can be automatically ran
    document.addEventListener(‘click’, () => {
      store.dispatch({ type: ‘INCREMENT’ });
    });

## What would createStore look like if you made one from scratch?

    // Reducer is only argument passed when declaring variable
    const createStore = (reducer) => {

      // Current state
      let state;

      // To keep track of all the changed listeners aka history.
      // This helps a bunch in the future with debugging.
      let listeners = [];

      // Just return current state
      const getState = () => state;

      const dispatch = (action) => {
        state = reducer(state, action);
        listeners.forEach(listener => listener());
      };

      const subscribe = (listener) => {
        // Keep 
        listeners.push(listener);
        return () => {
          listeners = listeners.filter(l => l !== listener);
        };
      };

      dispatch({});

      return { getState, dispatch, subscribe };
    };

## Redux + React Example

1. We are going to keep the same **reducer** from earlier:

        const counter = (state = 0, action) => {
          switch (action.type) {
            case ‘INCREMENT’:
              return state + 1;
            case ‘DECREMENT’:
              return state - 1;
            default:
              return state;
          }
        }
        
        const { createStore } = Redux;
        const store = createStore(counter);

2. To make the render function of the root called everytime the **store** updates, subscribe the root render function to the store:

        const counter = (state = 0, action) => {
          switch (action.type) {
            case ‘INCREMENT’:
              return state + 1;
            case ‘DECREMENT’:
              return state - 1;
            default:
              return state;
          }
        }
    
        const { createStore } = Redux;
        const store = createStore(counter);
    
        const render = () => {
          ReactDOM.render(
            document.getElementById('root')
          );
        };
    
        // Hey root component render function, be a callback for whenever our store changes!
        store.subscribe(render);
        render();

3. Now we can safely pass the current **state** changes, we pass it to our counter as a prop:

        const counter = (state = 0, action) => {
          switch (action.type) {
            case ‘INCREMENT’:
              return state + 1;
            case ‘DECREMENT’:
              return state - 1;
            default:
              return state;
          }
        }
    
        // Make that component
        const Counter = ({ value }) => (
          <h1>{value}</h1>
        );
    
        const { createStore } = Redux;
        const store = createStore(counter);
    
        const render = () => {
          ReactDOM.render(
            // Pass the current state as a prop to every component
            <Counter value={store.getState()} />,
            document.getElementById('root')
          );
        };
    
        store.subscribe(render);
        render();

3. Now we can safely pass the current **state** changes, we pass it to our counter as a prop:

        const counter = (state = 0, action) => {
          switch (action.type) {
            case ‘INCREMENT’:
              return state + 1;
            case ‘DECREMENT’:
              return state - 1;
            default:
              return state;
          }
        }
    
        // Bind buttons to dispatches back in the root as callbacks
        const Counter = ({
          value,
          onIncrement,
          onDecrement
        }) => (
          <h1>{value}</h1>
          <button onClick={onIncrement}>+</button>
          <button onClick={onDecrement}>-</button>
        );
    
        const { createStore } = Redux;
        const store = createStore(counter);
    
        const render = () => {
          ReactDOM.render(
            <Counter
              value={store.getState()}
    
              // Here are the actual dispatches that the button callbacks trigger to hit the store
              onIncrement={() =>
                store.dispatch({
                  type: 'INCREMENT'
                })
              }
              onDe crement={() =>
                store.dispatch({
                  type: 'DECREMENT'
                })
              }
            />,
            document.getElementById('root')
          );
        };
    
        store.subscribe(render);
        render();
