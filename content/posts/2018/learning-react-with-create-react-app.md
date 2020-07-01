+++
author = "James D Hughes"
categories = ["ReactJS"]
date = 2018-08-17T20:34:30Z
description = ""
draft = false
slug = "learning-react-with-create-react-app"
tags = ["ReactJS"]
title = "Learning React with create-react-app"

+++


I'm an Angular kind of guy.

I've built several front end applications using Angular 1.x and 2+.  I've built hybrid mobile apps using the Ionic Framework.  In this time, I seem to have just completely ignored the phenomenon that is React js.  There are so many frameworks that I've not even touched but if any of the job postings I'm seeing are an indication, the appetite for React and React Native developers is increasing rapidly.

So I'm going to go through my experience building the quintessential "todo" app to help me learn some things.  Feel free to follow along!

First, from the docs, I'm going to bootstrap my app with the 'create-react-app' installation from npm.  You can install this by opening a terminal and typing

```terminal
npm i -g create-react-app
```

Once that's started. You literally do just that.  Create a react app called my-todos or something of the like by entering a command such as
```terminal
create-react-app my-todos
```

Pretty straight forward so far?  Good.

Once everything is installed, go into your app folder and host it. You'll see a pretty minimalized application.

```terminal
cd my-todos
npm start
```

Congrats.  You've gone through the docs without reading the docs.  Some super entertaining content so far, no?

Now the first thing I'm going to do is essentially nuke out EVERYTHING in the `src/App.js` file as I don't care about their scaffold whatsoever. I'll then replace it with something like this:

```javascript
import React, { Component } from 'react';
import './App.css';

class App extends Component {
  render() {
    return (
      <div>
        <h3>Hello, World</h3>
      </div>
    );
  }
}

export default App;

```

That's because we "Hello, World" our learning here! Your browser should have live updated to display this purest of HTML greetings.

For our app we're going to need very few things.
1. A header for awesome UI ness
2. A todos-list component
3. An input to add new Todos

We're not going to worry about persisting data on a website for now as that may come in a part 2 (spoiler alert!)

First and foremost, we need to consider how we plan on managing state.  This has been done a load of ways now apparently using `react-redux` however right now I'm not totally sure what that does, so I'm just going to use this from what I've read in the docs as well:

```javascript
class App extends Component {

  constructor(props) {
    super(props);
    this.state = {
      todos: ['Learn React', '???', 'Profit'],
      text: ''
    }
  }
  // Omitted for brevity
}
```

This should give us an active state now. Which is pretty neat.  We've created some Todos that we will now want to display in some way.

Do do this, let's go ahead and update our render function to show our cool list!

```javascript
  render() {
    return (
      <div>
        <h3>Todo</h3>
        <ul>
          {this.state.todos.map(t => <li>{t}</li>)}
        </ul>
      </div>
    );
  }
}
```

This should do the trick.  We create some HTML components and then add in some react code to rip through the todos list and return some list items with the todo information in it.  Once you hit save, you should have a snazzy bullet list of our todos.

Next, what's a todo list without having the ability to enter in some new todos?  Surely you don't want me to just display a list that you can never add to! Needless to say, we're going to require an input.  For this part we are going to be editing the state so, if you're using chrome, it'd be worth installing the React DevTools offered up by Facebook.  This will help with any issues you may have debugging.  It's a pretty neat extension.

Let's add an input above our todo list (I'm a add from the top sort of guy I guess)

```javascript
  render() {
    return (
      <div>
        <h3>Todo</h3>
        <div>
          <input 
            type="text"
            value={this.state.text}
            onChange={this.onChange} 
          />
        </div>
        <ul>
          {this.state.todos.map(t => <li>{t}</li>)}
        </ul>
      </div>
    );
  }
```

The onChange function will be called as a user types.  This is so that we can update our state variables as a way of databinding.  Angular does this automatically with the [(ngModel)] directive, but for React, we need to handle it ourselves!  So let's add that function into our class then!

```javascript
  onChange = (event) => {
    this.setState({ text: event.target.value });
  }
```

Now this part confused me a load when I first started working in react.  The defining of a function in this way, that is.  React requires you to bind functions in order to call them from a component.  You can do this by invoking your function like this or by declaring your function and binding it in your constructor.  In lieu of having to make sure that I expose everything in my constructor, I elected this method as I think it may be easier, however not as straight forward.  FYI you could have accomplished this as well by doing something like this.

```javascript
class App extends Component {

  constructor(props) {
    super(props);
    this.state = {
      todos: ['Learn React', '???', 'Profit'],
      text: ''
    }
    this.onChangeBound = this.onChangeBound.bind(this);
  }

  onChangeBound(event) {
    this.setState({ text: event.target.value });
  }
  ...
}
```
Update the `onChange` in your `<input/>` to `onChangeBound` and you've got functionality again.  I'm still sort of confused on this syntax, but I'm not about to fight the framework.  Learn and move on!

We've got an input and it's updating our state variables.  Next we should probably handle when a user presses the enter key.  Traditionally you'd do this with a form and send some data to the server and it'd do its thing and redirect you to reload the page.  We're not going to do that though.  I plan on just using an input container. To do this, we will need to implement a keypress handler and listen for the 'enter' key.  Ready?

```javascript
  onKeypress = (e) => {
    if (e.key === 'Enter') {
      // validate the text
      if (this.state.text) {
        // if it's truthy
        this.setState(state => {
          let todos = state.todos;
          todos.push(state.text);
          return { todos: todos, text: '' }
        });
      }
      else {
        // Or else there's nothing to do I guess
        alert('Nothing to do!');
      }
    }
  }
```

Next we amend our input to handle keypresses...

```javascript
<input
  type="text"
  value={this.state.text}
  onChange={this.onChangeBound}
  onKeyPress={this.onKeypress}
/>
```
I'll let you test this on your own.  It should work. It worked on my machine!

Now that we have some functionality, go ahead and dump those placeholder todos.  We can make this go on our own!

At minimum, you should be able to get rid of todos from your list when you're done. So we're going to implement an 'x' button. We'll do this by amending our todos iteration to include the index, add in a button with an 'x' in it, and implement an onDelete function!

First let's adjust our list items to include both the todo and the index of the todo. We'll then add a button that has an onClick function to handle the delete.
```javascript
render(){
// ...

        <ul>
          {this.state.todos.map((t, index) =>
            <li>
              {t}
              <button 
                type="button" 
                onClick={this.onDeleteTodo} 
                value={index}>X</button>
            </li>)
          }
        </ul>
// ...
```
You'll probably have errors since onDeleteTodo isn't implemented, so let's build that function next.

```javascript
  onDeleteTodo = (e) => {
    let index = e.target.value;
    this.setState(state => {
      let todos = state.todos;
      todos.splice(index, 1);
      return { todos: todos };
    });
  }
```

One final piece of functionality we will add is some persistence.  We'll use localStorage and a `React.Component` lifecycle hook.

First, we need to persist the todos.  Let's make a function called `persistTodos`
```javascript
  persistTodos(){
    localStorage.setItem("todos", JSON.stringify(this.state.todos));
  }
```
That was easy.  We need to call this function every time we update our todos list. So add this into your `onKeypress` and `onDeleteTodo` functions.

```javascript
  onKeypress = (e) => {
    if (e.key === 'Enter') {
      if (this.state.text) {
        this.setState(state => {
          let todos = state.todos;
          todos.push(state.text);
          this.persistTodos();
          return {todos: todos, text:''};
        });
      }
      else {
        alert('Nothing to do!');
      }
    }
  }

  onDeleteTodo = (e) => {
    let index = e.target.value;
    this.setState(state => {
      let todos = state.todos;
      todos.splice(index, 1);
      this.persistTodos();
      return { todos: todos };
    });
  }
```

Finally, we need to load the state before we mount the component. We will do this using the `onComponentWillMount` lifecycle hook.  We will read the localStorage and set the todos should they exist.
```javascript
  componentWillMount() {
    let todos = localStorage.getItem("todos")
    if(todos) {
      this.setState({todos:JSON.parse(todos)});
    }
  }
```

Voila - a Todo App in Reactjs with some naive persistence and 0 styling. Only 80 lines of code!  Next, I'm going to explore styling (with frameworks. I hate writing CSS) and storing our data with a REST api.  

If you've got any questions or suggestions for improvement, feel free comment below!

