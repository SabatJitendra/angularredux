******************************************************************************************************************************************************
Angular Redux in action
******************************************************************************************************************************************************
1)Redux is a state management library based on flux architecture.Instead of different JS objects and data scattered across the application,redux keeps 
data and state in a single JS object(single source of truth) called store. This way it's more manageable,maintainable and predictable.If a specific
data is being referred in multiple places,changing single state object will result in updating the UI in all the places.This way we will avoid 
duplication and better code reusablility.Some of the advantages are
            ->Predictable application state
            ->Decoupled architecture
            ->Increases testability of application
            ->Great tooling(Redux developer tools)
            ->Undo/redo
2)When to use Redux?
            ->Independent copies of same data in multiple places
            ->Multiple views that need to work with the same data and be in sync
            ->Data can be updated by multiple users
3)Basic building blocks of Redux.
            ->Store
            ->Actions
            ->Reducers                    
4)Store - It's a single JS object that contains the state of application.It can be considered as a local client side database.Store is the single source
of truth for our application state and is accessible from anywhere in UI application.In an ecommerce app a store can have properties like categories, 
products,cart etc.We can store anything in our store depending on the application need.We can not modify or mutate the store
{
    categories: [],
    products: [],
    cart: {},
    user: {}
}
5)Action - Plain JS objects that represent something that has happened.Semantically they are more like events.
e.g.   { type: 'MARK_AS_READ' }
       { type: 'POST_MESSAGE', body: '...'}
6)Reducer - A function that specifies how the state changes in response to an action.It's an action handler or event handler that determines how the 
state is changed.A reducer does not modify the state, it returns a new state.Then the store will internally update the state.In redux our reducers should
be pure functions.
7)Pure function - A function is pure if we will get the same output for given set of same input.No matter how many times we call the function it will 
always result in same output.
-------------------------------------------
Examples of impure functions
-------------------------------------------
function increment(input){
    input.count++;//mutating arguments so it's impure
}

function incremenr(input){
    service.addMessage(...);//making back end calls
}

function increment(input){
    input.count += Math.random(); //output always changes as we are using an impure function like random()
}
-------------------------------------------
Example of pure functions
-------------------------------------------
function increment(input){
    return { count: input.count+1 };
}
8)A reducer will always take 2 parameters.
        ->state i.e. current state
        ->action i.e. an event that has happened recently in application
So a basic reducer will look something like as follows

function reducer(state,action){
    switch(action.type){
        case 'INCREMENT':
            return { count: state.count + 1 };
        case 'DECREMENT':
            return { count: state.count - 1 };
    }
}
9)Why are we not allowed to mutate or modify the state? Rather we always return a new state?What is the benefit by doing so?
->Easy testability and it's easy to do an assertion because of predictable behavior
->It's easy to implement undo/redo functionality by this approach,because we always keep previous state instead of modifying it
->Time travel debugging- a tool to debug application state and how the state has changed subsequently because of several actions.This makes debugging
    easy.We can easily find bug and fix them.
10)Redux implementations for Angular
    ->ngrx/store - reimplement the redux from the scratch in Anguar friendly way.It's not compatible with other libraries built for Redux.
    ->ng2-redux - built on top of real redux library and compatible with much of redux eco system.It adds binding for Angular;so we can easily connect
    Angular components with Redux.We will be using this for our example.
11)Install redux
npm install redux ng2-redux rxjs-compat --save
rxjs-compat is a package used to resolve the dependency error.We don't have to do any implementation with this package
12)Under Angular app folder add a new file store.ts
---------------------------------------------------
store.ts
---------------------------------------------------
export interface IAppState{}

export function rootReducer(state, action){
    return state;
}
13)Lets go to app.module.ts and import redux based stuffs and configure it
---------------------------------------------------------------
app.module.ts
---------------------------------------------------------------
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { NgRedux, NgReduxModule } from 'ng2-redux';

import { AppComponent } from './app.component';
import { IAppState, rootReducer } from './store';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    NgReduxModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { 
  constructor(private ngRedux: NgRedux<IAppState>){
    ngRedux.configureStore(rootReducer,{});
  }
}
This completes the configuration of redux with Angular.Now lets see how to handle actions
14)On click of button increment the counter
--------------------------------------
app.component.html
--------------------------------------
<div>
  <h1>
    {{title}}
  </h1>
  <p>Counter: {{ counter }}</p>
  <button (click)="increment()">Increment</button>
</div>
--------------------
app.component.ts
--------------------
export class AppComponent {
  title = 'reduxApp';
  counter = 0;
  increment(){
    ++this.counter;//This is how we modify state in a typical Angular app.When applying Redux we can't mutate state like this directly.
  }
}
We dispatch an action for every event in application. Action goes in to store.Store knows rootReducer and passes the action to rootReducer.Reducer
looks at the action and based on the type of action,it will return a new state.Then the store will update the state internally.
                ____________        ___________         _____________
                |          |        |         |         |           |
                |          |        |         |         |           |
                |  Action  |------->|  Store  |-------->|  Reducer  |
                |          |        |         |         |           |
                |          |        |         |         |           |
                ------------        -----------         -------------
                                       /\                       |
                                        |_______________________|
                                            New State
15)Changed implementation with redux version should be
-------------------------------------------------
app.component.ts
-------------------------------------------------
import { Component } from '@angular/core';
import { NgRedux } from 'ng2-redux';
import { IAppState } from './store';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'reduxApp';
  counter = 0;

  constructor(private ngRedux: NgRedux<IAppState>){}

  increment(){
    //++this.counter;

    //dispatch() is taking an action 'INCREMENT'.Every time we take an action,we need to go our reducer and determine how the state will change in 
    //response to the said action.So add case in rootReducer() to handle action type 'INCREMENT'.
    this.ngRedux.dispatch({ type: 'INCREMENT' });
  }
}
---------------------------------------
app/store.ts
---------------------------------------
export interface IAppState{
    counter: number;
}

export function rootReducer(state: IAppState, action):IAppState{
    switch(action.type){
        case 'INCREMENT':
            return { counter: state.counter + 1 };
            break;
        default:
            break;
    }
    return state;
}
reducer takes in a state which is of type IAppState and depending on action type returns a new state which is of type IAppState.In our app.component.ts
we are dispatching an event 'INCREMENT', which eventually will increment a counter variable.So we have requirement of introducing a counter property to
our IAppState which is of type number.This property will be equivalent to counter property in app.component.ts level counter variable.Now as it's part of
IAppState, we can refer it from any where in application.Now rootReducer will simply return a brand new object with counter property incremented by 1.
As we introduced a counter property to IAppState so code in app.module.ts would be breaking where we passed an empty object.Update that one
-----------------------------------
app.module.ts
-----------------------------------
export class AppModule { 
  constructor(private ngRedux: NgRedux<IAppState>){
    ngRedux.configureStore(rootReducer,{ counter: 0 });
  }
}
16)in app folder add a file actions.ts.All string type values will be centrally maintained here. It will ensure less bug and better code reusability and
maintainability.
-------------------------------
actions.ts
-------------------------------
export const INCREMENT = 'INCREMENT';
Now refer this where ever we use string 'INCREMENT'.
17)app.module.ts needs a initial state { counter: 0 } with some default value.Lets create a separate entry for the same in store.ts
------------------------
store.ts
------------------------
export const INITIAL_STATE: IAppState = {
    counter: 0
}
Now refer this in app.module.ts
------------------------
app.module.ts
------------------------
import { IAppState, rootReducer, INITIAL_STATE } from './store';

export class AppModule { 
  constructor(private ngRedux: NgRedux<IAppState>){
    ngRedux.configureStore(rootReducer,INITIAL_STATE);
  }
}
At this stage also our button won't increment.We will do that in next step
18)we have ngRedux available as constructor parameter in app.component.ts. ngRedux is an observable, subscribe to it.
------------------------------
app.component.ts
------------------------------
constructor(private ngRedux: NgRedux<IAppState>){
  ngRedux.subscribe(() => {
    console.log(ngRedux.getState());
  });
}
With above change on click of button in UI we will see state object being logged to console with incremented counter value
{counter: 1}//clicked once
{counter: 2}//clicked twice
We can use this to update counter value in UI
-----------------------------
app.component.ts
-----------------------------
export class AppComponent {
  title = 'reduxApp';
  counter = 0;

  constructor(private ngRedux: NgRedux<IAppState>){
    ngRedux.subscribe(() => {
      var store = ngRedux.getState();
      this.counter = store.counter;
    });
  }

  increment(){    
    this.ngRedux.dispatch({ type: INCREMENT });
  }
}