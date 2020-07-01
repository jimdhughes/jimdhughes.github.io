+++
author = "James D Hughes"
categories = ["ReactJS", "nodejs"]
date = 2018-08-29T02:37:01Z
description = ""
draft = false
slug = "fetching-data-with-react"
tags = ["ReactJS", "nodejs"]
title = "Fetching data with react"

+++


What use is an app that can't persist data?

I'm continuing this from my previous post about my efforts to learn about react.  I want to be able to persist my data on a server and display it on a web client.  To my understanding, this should be pretty simple so I'm going to put that theory to the test.

# Firstly!

We're going to use a Node.js scaffold that I covered off in a previous posting as our backend for this app. It's located [here](https://github.com/Daimonos/jdhc-express-stateless-template) and requires you to have Node.js installed and access to a MongoDB Instance.  Follow the instructions in that repo to get up and running.  If you don't want to use that one, you can set up your own REST service (or CouchDB instance?) and follow along with the concepts.

Since I'm not going to cover authentication yet, we need to set up a model and endpoints for our Todo's that are going to be 100% public. To do this add the following code to the `models/todo.model.js` file.

```javascript
// models/todo.model.js
const mongoose = require('mongoose');

const todoSchema = mongoose.Schema({
  todo: { type: String, required: true },
  date: { type: Date, required: true, default: () => new Date() },
  completed: { type: Boolean, required: true, default: false }
});

module.exports = mongoose.model('Todo', todoSchema);

```

Then we load the model in `models/index.js`

```javascript
// models/index.js

// ...
require('./todo.model');
```

Then we need some endpoints to handle our Todos:

```javascript
const router = require('express').Router();
const Todo = require('mongoose').model('Todo');

router.route('/')
  .get(getTodos)
  .post(createTodo)

router.route('/:id')
  .get(getTodo)
  .put(updateTodo)
  .delete(deleteTodo);

module.exports = router;

async function getTodos(req, res, next) {
  try {
    let todos = await Todo.find();
    return res.json(todos);
  } catch (e) { 
    next(e); 
  }
}

async function createTodo(req, res, next) {
  let todo = req.body;
  try {
    todo = await Todo.create(todo);
    return res.status(201).json(todo);
  } catch (e) {
    next(e);
  }
}

async function getTodo(req, res, next) {
  let id = req.params.id;
  try {
    let todo = await Todo.findById(id);
    return res.json(todo);
  } catch(e) {
    next(e);
  }
}

async function updateTodo(req, res, next) {
  let id = req.params.id;
  let todo = req.body;
  try {
    todo = await Todo.findByIdAndUpdate(id, todo, {new: true});
    return res.json(todo);
  } catch(e) {
    next(e);
  }
}

async function deleteTodo(req, res, next) {
  let id = req.params.id;
  try {
    await Todo.findByIdAndRemove(id);
  } catch(e) {
    next(e);
  }
}
```

Then we require it in our `routes/index.js` file:

```javascript
// routes/index.js
module.exports = {
  users: require('./users'),
  authentication: require('./authentication'),
  me: require('./me'),
  assets: require('./assets'),
  todos: require('./todos')
}
```

Then we load it up and map it to an express handler in `app.js`

```javascript
// app.js
// ...
app.use('/api/v1/todos', routes.todos);
// ...
```

Two more things to do here.

1: Enable CORS. Since we are accessing this API from another app that will be hosted on a separate port, we will need to allow access to it.

```javascript
// app.js
/**
 * Enable Cors
 */
app.use(function (req, res, next) {
  res.header('Access-Control-Allow-Origin', 'http://localhost:3000');
  res.header('Access-Control-Allow-Headers', 'Origin,X-Requested-With,Content-Type,Accept,responseType');
  res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS');
  next();
});
```

2. Since I'm not familiar with React configuration, I'm going to just set my Express API to serve on port 3001 instead of 3000. Depending on your OS you can do this differently:

```
// Powershell
$env:PORT="3001"
// Command Line
set PORT=3001&npm start
// Ubuntu
PORT=3001 npm start
```

There we go. API is running on `http://localhost:3001/api/todos` Now it's time to get to the meat and potatoes of this post! Head over to your react app and type `npm start` I'm using the Todo app that I built in a previous post with a few minor modifications. I released this on my github under the release v0.1: [https://github.com/Daimonos/react-todos-blog/releases/tag/v0.1](https://github.com/Daimonos/react-todos-blog/releases/tag/v0.1)

Install it using `npm install` and run with `npm serve` You should see our minimal Todos app that uses local storage for persistence.

Firstly, let's create a new folder to handle our api functionality. Simply called `api`.

In this folder, I'm going to make a helper that will expose FETCH functionality in an easier-to-use way that will set the headers and implement the verbs as required.

So let's create the files `api/api.service.js` with the following structure:

```javascript
// api/api.service.js
const API_URI = "http://localhost:3001/api/v1/";
const headers = {
  "Content-Type":"application/json",
  "Accept":"application/json"
}
class Api {
  
  get(endpoint, params = {}) {
    return fetch(API_URI+endpoint, {
      params: params,
      headers: headers
    })
    .then(r=>r.json());
  }

}

export default Api;
```

Now that we have this neato helper, we can use it in our Todo Service with a little more ease!

```javascript
// api/todo.service.js
import Api from './api.service';

class TodoService extends Api {
  constructor() {
    super();
  }

  getTodos() {
    return this.get('todos');
  }
}
```

We import this service into our TodoList component to make the magic happen! There are going to be a lot of changes to this file so hopefully I don't bungle it up too much in the post.

```javascript
// components/TodoList/TodoList.js
import React from 'react';
import TodoService from '../../api/todo.service';
import TodoInput from '../TodoInput/TodoInput';
import './TodoList.css';

class TodoList extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      todos: [],
      text: ''
    }
    this._service = new TodoService();
  }
  
  componentWillMount() {
    this._loadTodosFromApi();
  }
  
  async _loadTodosFromApi() {
    try {
      let todos = await this._service.getTodos();
      this.setState({ todos });
    } catch(e) {
      // You should handle this better
      console.error(e);
      throw e;
    }
  }
// ... 
}
```

This is pretty straight forward. We import the TodoService, we initialize it in our constructor. In the `componentWillMount` life cycle hook we load our Todos from our api by calling `this._loadTodosFromApi()`.  I think the async/await implementation was the greatest thing to happen to javascript development in the last 10,000 years. Therefore I use it all the time instead of callbacks and promises. We wait for the service to return make our API call and return the JSON body. We then set the state  in our component. Pretty straight forward!

The next logical step would be to amend our "create todo" workflow. Instead of just pushing to a list in the state and persisting to localstorage, we should POST it to the API and then add it into the state assuming everything is successful. We have a few things to do now (todo, get it?)

```javascript

// api/api.service.js
//...
class Api {
  // ...
  post(endpoint, body) {
    return fetch(API_URI+endpoint, {
      method:'POST',
      headers:headers,
      body: body
    })
    .then(r=>r.json());
  }
}


```

```javascript
// api/todo.service.js
class TodoService extends Api {
  // ...
  createTodo(todo) {
    return this.post('todos', todo);
  }
}
```

In our `TodoList` component, we now amend our `onKeypress` function to leverage the service!

```javascript
//components/TodoList/TodoList.js

class TodoList extends React.Component {
  // ...
  onKeypress = (e) => {
    if (e.key === 'Enter') {
      if (this.state.text) {
        this._onSaveTodo();
      }
      else {
        alert('Nothing to do!');
      }
    }
  }

  async _onSaveTodo() {
    try {
      let text = this.state.text;
      let todo = await this._service.createTodo({todo: text});
      this.setState(s=>{
        return {todos: [...s.todos, todo], text:''};
      })
    } catch(e) {
      alert('Error Creating Todo!');
    }
  }
  // ...
}
```

Now when you add a Todo... Explosions Happen! Since we're dealing with objects instead of just text, we need to update our Render function to include a key on the <li> compnent and to display the todo text as opposed to just the {t}.

```javascript
// app/components/TodoList/TodoList.js

class TodoList extends React.Component {
  // ...
  render() {
    return (
      <div>
        <div className="todos">
          <TodoInput
            type="text"
            placeholder="Add a Todo"
            value={this.state.text}
            onChange={this.onChange}
            onKeyPress={this.onKeypress}
          />
        </div>
        <ul className="todo-list">
          {this.state.todos.map((t, index) =>
            <li key = {t._id}>
              <button type="button" onClick={this.onDeleteTodo} value={index}>X</button>
              {t.todo}
            </li>)
          }
        </ul>
      </div>
    );
  }
  // ...
}
```

Once you hit save, you should see your todo pop up. Go ahead and add a few more and you will see it's creating as intended. Hit refresh and you will see that they have been all successfully saved in our API.

On the same token, we will need to update our `onDeleteTodo` function to destroy the asset on the server. To do this, we will have to implement the `delete` function in our `Api` service and then add a `deleteTodo` function in our `TodoService`

```javascript
// api/api.service.js

class Api {
  // ...
  delete(endpoint) {
    return fetch(API_URI+endpoint, {
      method:'DELETE',
      headers: headers
    })
    .then(r=>r.json());
  }

}
```

```javascript
// api/todo.service.js

class TodoService extends Api {
  // ...

  deleteTodo(id) {
    return this.delete('todos/'+id);
  }
}
```

Finally we can update our `onDeleteTodo` function in our `TodoList` component.

```javascript
// components/TodoList/TodoList.js

onDeleteTodo(todo) {
    this._onDeleteTodo(todo);
  }

  async _onDeleteTodo(todo) {
    try {
      let index = this.state.todos.indexOf(todo);
      await this._service.deleteTodo(todo._id);
      this.setState(s => {
        let todos = s.todos;
        todos.splice(index, 1);
        return { todos }
      })
    } catch (e) {
      alert('Error Deleting Todo');
    }
  }

```

I realize the naming might be a little confusing here, but I don't want to change too much.  This still won't work however as we now need to update our render to be able to handle calling this function.

```javascript
// components/TodoList/TodoList.js

render() {
  //...
  <ul className="todo-list">
    {this.state.todos.map((t, index) =>
      <li key={t._id}>
        <button type="button" onClick={() => this.onDeleteTodo(t)}>X</button>
        {t.todo}
      </li>)
    }
  </ul>
  // ...
}
```

For fun, we may as well make use of the other endpoints that we made. Let's add a checkbox to our Todos as well! We will handle the checkbox click by updating the Todos completed field.

Again, we need to implement our API handlers

```javascript
// app/api.service.js
class Api {
  // ...
  put(endpoint, body) {
    return fetch(API_URI+endpoint, {
      method:'PUT',
      headers: headers,
      body: JSON.stringify(body)
    }).then(r=>r.json());
  }
}
```

```javascript
// app/todo.service.js
class TodoService extends Api {
  // ...
  updateTodo(todo) {
    return this.put('todos/'+todo._id, todo);
  }
}
```

Then we add some handlers in our `TodoList` component

```javascript
// components/TodoList/TodoList.js

class TodoList extends React.Compnent {
  
  // ...
  async _onUpdateTodo(todo) {
    try {
      let index = this.state.todos.indexOf(todo);
      todo = await this._service.updateTodo(todo);
      this.setState(s=>{
        let todos = s.todos;
        todos.splice(index, 1, todo);
        return {todos};
      });
    } catch(e) {
      alert('Error Updating Todo!');
    }
  }

  onCheckTodo(todo) {
    todo.completed = !todo.completed;
    this._onUpdateTodo(todo);
  }

  render(){
    return (
      // ...
      <ul className="todo-list">
          {this.state.todos.map((t, index) =>
            <li key={t._id}>
              <button type="button" onClick={() => this.onDeleteTodo(t)}>X</button>
              {t.todo}
              <input type="checkbox" checked = {t.completed} className="todo-checkbox" onChange = {()=>this.onCheckTodo(t)}/>
            </li>)
          }
        </ul>
      // ...
    );
  }

}
```

I have some css you can check out in the project's github.  I'm awful at CSS honestly so I wouldn't suggest taking it too seriously.

There you have it! A Todo app written in full stack javascript using React.js.

Next, I'll dabble with authentication and handling it in this app as well. This code will be released under the v0.2 tag in the same github repository as on the top.

Until Then!

