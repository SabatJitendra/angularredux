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

export interface IAppState{}

export function rootReducer(state, action){
    return state;
}
13)Lets go to app.module.ts and import redux based stuffs and configure it
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
This completes the configuration of redux with Angular.




