---
title: >-
  Up And Running With React & Redux & Three.JS Part 2
date: 2018-01-25 21:13:40
tags: react redux three.js
---

## Part 2 ... introducing redux
In the first part of this post, I walked through the basic setup of a React application. Part 2 will detail the setup for Redux to manage our state, and Sagas to handle our actions' side effects (asynchronous calls and whatnot).


### Setting up the store
Setting up our Store will require us first to create our root reducer file. If you only have a single reducer, you can simply pass it to the store upon creation. However, as the application grows, keeping it separated is cleaner.


In our reducer, we'll require combineReducers from redux with will allow us to combine all of our future reducers that we'll pass to Store upon creation/initialization.


##### src/Reducer.js
```
import { combineReducers } from 'redux';
import counterReducer from '../counter/CounterReducer;

const RootReducer = combineReducers({
    counter: counterReducer
});

export default RootReducer;
```

Next we'll create our Store also in src. Our store is also where we'll configure our middleware, and require our reducer.

##### src/Store.js
```
import { createStore, applyMiddleware } from 'redux';
import { createLogger } from 'redux-logger';
import createSagaMiddleware from 'redux-saga';
import RootReducer from 'Reducer';

const logger = createLogger()
const sagaMiddleware = createSagaMiddleware();

const store = createStore(
    RootReducer,
    applyMiddleware(sagaMiddleware, logger)
  )

export default store
```

At present we don't have any actions warranting the use of a saga, so we don't have a rootSaga that we would then call to run (this will come in a bit). Next we'll want to hook up our store to our application by passing it to the Provider which will now wrap our application in index.js. We need to make sure to import Provider from react-redux and our store.

```
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import Store from './Store';
import App from './App';
import './index.css';

ReactDOM.render(
  <Provider store={Store}>
    <App />
  </Provider>,
  document.getElementById('main'));
```

### Setting up a Connected Component
Before we can run our application, we'll need to go ahead and create the reducer that our root reducer imports. From there we'll build out our connected component. For the purpose of simplicity, the connected component will be a simple counter.

Start by creating a new directory in src/, counter/, and touching CounterReducer.js, CounterActions.js and CounterActionTypes.js

CounterActionTypes.js
```
export const INCREMENT_COUNTER = 'INCREMENT_COUNTER';
```
For now it might seem overkill to give the action type its own file but I find this helps to illustrate the responsibility of each part of our component's data flow by distinguishing our action types from the actual action that will be dispatched...

CounterActions.js
```
export const incrementCounter = () => {
  return {
    type:  'INCREMENT_COUNTER'
  }
}
```

The counterReducer only has one value, and the only action it will be listening for is an increment action. As the function of the reducer grows, it's good house keeping to import the action types from a separate actions file.

CounterReducer.js
```
import { INCREMENT_COUNTER } from './CounterActionTypes';

const initialState = {
  counterValue: 0
}

const counterReducer = (state = initialState, action) => {
  switch(action.tyoe){
    case INCREMENT_COUNTER:
      let newValue = state.counterValue +=1
      return {...state, counterValue: newValue}
    default:
      return state
  }
}

export default counterReducer;
```

Next we'll set up the Counter component. For now it won't do anything aside from return some html.

```
import React, { Component } from 'react';

export default class Counter extends Component{

  render(){
    return(
      <div>
        <h1>Counter Component</h1>
      </div>
  )}
}
```

We're ready to hook up our CounterContainer with our component and reducer/actions. The CounterContainer leverages connect from react-redux to connect the redux store to our component. Map state to props will map the part of state that the component is concerned with to the props passed down to it, and map dispatch to props will do the same with our action creators, in this case, 'incrementCounter'.

```
import { Component } from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import { incrementCounter } from './CounterReducer';
import Counter from './Counter.jsx';

const mapStateToProps = (state) => ({
  counterValue: state.counterReducer.counterValue
});

const mapDispatchToProps = (dispatch) => bindActionCreators({
  incrementCounter: incrementCounter }, dispatch);

const CounterContainer = connect(mapStateToProps, mapDispatchToProps)(Counter);

export default CounterContainer;
```

Now we can return to the component, and use the props we've connected and passed down in the html that we're returning

Counter.jsx
```
render(){

    const {incrementCounter, counterValue} = this.propsMode

    return(
      <div>
        <button className="increment-button" onClick={incrementCounter}>+</button>
        <h1>{counterValue}</h1>
      </div>
  )}
```

Two last things before we build and run the application. Make sure that the new reducer is added to the root reducer, and that the CounterContainer is added in App.js

Reducer.js
```
const RootReducer = combineReducers({
  counterReducer
});
```

App.js
```
import CounterContainer from './counter/CounterContainer';

....
render(){
    return(
      <div className="App">
        <h1>Hello World</h1>
        <CounterContainer />
      </div>);
  }
```

When we click the increment button, now we'll see the counterValue increase as a result.


Check out Part 3 coming soon to setup a Three.js scene component and hook it up to you redux state.
