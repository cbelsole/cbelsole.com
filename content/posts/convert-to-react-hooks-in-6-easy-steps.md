---
title: "Convert to React Hooks in 6 easy steps"
date: "2020-03-21"
description: "In this guide we'll take a look at how to convert an application based on the controller component to one that uses hooks to manage global state."
categories:
  - "Architecture"
tags:
  - "React"
  - "React Hooks"

draft: false
---

[React hooks](https://reactjs.org/docs/hooks-intro.html) were released in version [16.8.0](https://github.com/facebook/react/blob/master/CHANGELOG.md#1680-february-6-2019), and since then [we’ve all](https://dev.to/search?q=react%20hooks) been trying to figure out how to use them effectively and convert our components so that using newer versions of React is not an upgrade nightmare. In this article we are going to discuss the 6 steps for moving your React components to hooks using an [example to-do application](https://github.com/cbelsole/controller_hooks_example) written in Typescript that starts with the controller pattern and [commit by commit](https://github.com/cbelsole/controller_hooks_example/commits/master) lays out my methodology for moving components. There are more complex examples, but this article should give you a good grounding in how to think about migrating your components.

> **Protip:** Using React hooks can be a replacement for Redux bringing down the number of dependencies in your application.

# Step 1: Select the component to convert

Let’s chat for a second about what our example to-do app looks like.

<link to https://raw.githubusercontent.com/cbelsole/controller_hooks_example/master/todo_app_demo.gif>

![to-do app demo](https://raw.githubusercontent.com/cbelsole/controller_hooks_example/master/todo_app_demo.gif "to-do app demo")

As you can see we have a list of to-do’s that can be completed with a button underneath that checks if all to-do’s are complete. For this example we are going to convert the [`Controller.tsx`](https://github.com/cbelsole/controller_hooks_example/blob/cb885358b697ff1096c7de6685d0a7e8ace9da53/src/components/controller.tsx) component which declares an empty list of to-do’s and renders a loading state:

```tsx
export default class Controller extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { todos: [] };
  }
…
  render() {
    const { todos } = this.state;

    if (!todos.length) {
      return <div>loading...</div>;
    }

```

Grabs data from the API via `componentDidMount()` and populates the list of to-do’s:

```tsx
  componentDidMount() {
    getTodos().then(todos => this.setState({ todos }));
  }
```

And renders the [`<TodoList />`](https://github.com/cbelsole/controller_hooks_example/blob/cb885358b697ff1096c7de6685d0a7e8ace9da53/src/components/todoList.tsx) along with the [`All complete?`](https://github.com/cbelsole/controller_hooks_example/blob/cb885358b697ff1096c7de6685d0a7e8ace9da53/src/components/controller.tsx#L47) button while passing down the complete callback:

```tsx
render() {
    const { todos } = this.state;
    ...
    return (
      <div>
        <TodoList completeTodo={this.completeTodo} todos={todos} />
        <button onClick={this.isAllComplete}>All complete?</button>
      </div>
    );
  }

```

Here’s the full code:

```tsx
import * as React from "react";
import { getTodos, completeTodo as completeTodoAPI, iTodo } from "../api/todos";
import TodoList from "./todoList";

interface Props {}
interface State {
  todos: iTodo[];
}

export default class Controller extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { todos: [] };
  }

  componentDidMount() {
    getTodos().then(todos => this.setState({ todos }));
  }

  completeTodo = (item: string) => {
    completeTodoAPI(item).then(todos => this.setState({ todos }));
  };

  isAllComplete = () => {
    const { todos } = this.state;

    for (let i = 0; i < todos.length; i++) {
      if (!todos[i].done) {
        alert("Todos are not complete.");
        return;
      }
    }

    alert("Todos are complete.");
  };

  render() {
    const { todos } = this.state;

    if (!todos.length) {
      return <div>loading...</div>;
    }

    return (
      <div>
        <TodoList completeTodo={this.completeTodo} todos={todos} />
        <button onClick={this.isAllComplete}>All complete?</button>
      </div>
    );
  }
}
```

# Step 2: Convert your class to a function

Here we are changing our class to a function and returning the JSX elements from that function. So we move all of our returns statements outside of the `render()` method. In Typescript `React.FunctionalComponent` (`React.FC`) is the interface for a function component. For Javascript you would just declare a function.

```diff
diff --git a/src/components/controller.tsx b/src/components/controller.tsx
index 7184893..e310613 100644
--- a/src/components/controller.tsx
+++ b/src/components/controller.tsx
@@ -7,7 +7,7 @@ interface State {
   todos: iTodo[];
 }

-export default class Controller extends React.Component<Props, State> {
+const Controller: React.FunctionComponent = () =>  {
   constructor(props: Props) {
     super(props);
     this.state = { todos: [] };
@@ -34,7 +34,6 @@ export default class Controller extends React.Component<Props, State> {
     alert('Todos are complete.');
   };

-  render() {
   const { todos } = this.state;

   if (!todos.length) {
@@ -47,5 +46,7 @@ export default class Controller extends React.Component<Props, State> {
       <button onClick={this.isAllComplete}>All complete?</button>
     </div>
   );
+
 }
-}
+
+export default Controller;
```

# Step 3: Extract class methods to consts

Extracting [static and class methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) to consts is the simplest analogue I've found for the structure of a functional component. Class methods rely on state. So they are inlined with the function. Static methods don't rely on state. So they go outside of the function.

```diff
diff --git a/src/components/controller.tsx b/src/components/controller.tsx
index e310613..4322bf2 100644
--- a/src/components/controller.tsx
+++ b/src/components/controller.tsx
@@ -17,11 +17,11 @@ const Controller: React.FunctionComponent = () =>  {
     getTodos().then(todos => this.setState({ todos }));
   }

-  completeTodo = (item: string) => {
+  const completeTodo = (item: string) => {
     completeTodoAPI(item).then(todos => this.setState({ todos }));
   };

-  isAllComplete = () => {
+  const isAllComplete = () => {
     const { todos } = this.state;

     for (let i = 0; i < todos.length; i++) {
@@ -42,8 +42,8 @@ const Controller: React.FunctionComponent = () =>  {

   return (
     <div>
-      <TodoList completeTodo={this.completeTodo} todos={todos} />
-      <button onClick={this.isAllComplete}>All complete?</button>
+      <TodoList completeTodo={completeTodo} todos={todos} />
+      <button onClick={isAllComplete}>All complete?</button>
     </div>
   );
```

# Step 4: Extract state via useState()

Ahhh, we finally get to use hooks. [`useState()`](https://reactjs.org/docs/hooks-state.html) is our first hook we are going to use to extract the state of our component. This hook works by declaring the default state and returning the first parameter as the state and the second as a function to update the state. Since we inlined all the class methods the new state should be accessible in the functions.

```diff
diff --git a/src/components/controller.tsx b/src/components/controller.tsx
index 4322bf2..000b077 100644
--- a/src/components/controller.tsx
+++ b/src/components/controller.tsx
@@ -1,29 +1,21 @@
-import * as React from 'react';
+import React, { useState } from 'react';
 import { getTodos, completeTodo as completeTodoAPI, iTodo } from '../api/todos';
 import TodoList from './todoList';

 interface Props {}
-interface State {
-  todos: iTodo[];
-}

 const Controller: React.FunctionComponent = () =>  {
-  constructor(props: Props) {
-    super(props);
-    this.state = { todos: [] };
-  }
+  const [todos, setTodos] = useState<iTodo[]>([])

   componentDidMount() {
-    getTodos().then(todos => this.setState({ todos }));
+    getTodos().then(todos => setTodos(todos));
   }

   const completeTodo = (item: string) => {
-    completeTodoAPI(item).then(todos => this.setState({ todos }));
+    completeTodoAPI(item).then(todos => setTodos(todos));
   };

   const isAllComplete = () => {
-    const { todos } = this.state;
-
     for (let i = 0; i < todos.length; i++) {
       if (!todos[i].done) {
         alert('Todos are not complete.');
@@ -34,8 +26,6 @@ const Controller: React.FunctionComponent = () =>  {
     alert('Todos are complete.');
   };

-  const { todos } = this.state;
-
   if (!todos.length) {
     return <div>loading...</div>;
   }
(END)
```

# Step 5: Convert lifecycle methods

Here we have some interesting options depending on what hooks we are converting. Check out [this article](https://dev.to/trentyang/replace-lifecycle-with-hooks-in-react-3d4n) for some common conversions. We only want our [`useEffect()`](https://reactjs.org/docs/hooks-effect.html) function to run when the component mounts. So we'll pass it an empty array (`[]`) in the second argument signifying it should run once and not again since there are no parameters in the array to cause it to refresh.

```diff
diff --git a/src/components/controller.tsx b/src/components/controller.tsx
index 000b077..0f85564 100644
--- a/src/components/controller.tsx
+++ b/src/components/controller.tsx
@@ -1,15 +1,11 @@
-import React, { useState } from 'react';
+import React, { useState, useEffect } from 'react';
 import { getTodos, completeTodo as completeTodoAPI, iTodo } from '../api/todos';
 import TodoList from './todoList';

 interface Props {}

 const Controller: React.FunctionComponent = () => {
-  const [todos, setTodos] = useState<iTodo[]>([])
-
-  componentDidMount() {
-    getTodos().then(todos => setTodos(todos));
-  }
+  const [todos, setTodos] = useState<iTodo[]>([]);

   const completeTodo = (item: string) => {
     completeTodoAPI(item).then(todos => setTodos(todos));
@@ -26,6 +22,10 @@ const Controller: React.FunctionComponent = () =>  {
     alert('Todos are complete.');
   };

+  useEffect(() => {
+    getTodos().then(todos => setTodos(todos));
+  }, []);
+
   if (!todos.length) {
     return <div>loading...</div>;
   }
@@ -36,7 +36,6 @@ const Controller: React.FunctionComponent = () =>  {
       <button onClick={isAllComplete}>All complete?</button>
     </div>
   );
-
-}
+};

 export default Controller;
```

# Step 6: Cleanup unused components

A simple but important step, clean up your code if you have anything left over. Future you will be happy you took the time.

```diff
diff --git a/src/components/controller.tsx b/src/components/controller.tsx
index 0f85564..a4eaac9 100644
--- a/src/components/controller.tsx
+++ b/src/components/controller.tsx
@@ -2,8 +2,6 @@ import React, { useState, useEffect } from 'react';
 import { getTodos, completeTodo as completeTodoAPI, iTodo } from '../api/todos';
 import TodoList from './todoList';

-interface Props {}
-
 const Controller: React.FunctionComponent = () => {
   const [todos, setTodos] = useState<iTodo[]>([]);
```

# (Bonus) Step 7: Move state management to context/provider

At this point you have a working functional component. So why not just stop here and move on to your next task? The answer is a bit complex and touches on architectural principals so first, let's talk a little about [SOLID](https://en.wikipedia.org/wiki/SOLID) principals, state management, and [component coupling](<https://en.wikipedia.org/wiki/Coupling_(computer_programming)>).

## Background

SOLID principals are a set of ideas for building maintainable software acting as guides for making decisions about architecting complex systems. The S in SOLID stands for the [Single Responsibility Principal](https://en.wikipedia.org/wiki/Single-responsibility_principle) which states that "A class should only have one reason to change." In short, things do one thing. Since React is a frontend library it is easy and often required to break this principal since components are often rendering HTML and managing state. This works well enough but it often becomes unmaintainable when you have to rewrite your code for another use case since the state that drives your components is kept at the highest level.

This is where we start talking about local state vs global state. Local state is state local to your component. Think filling out an HTML form or keeping track of button clicks. That info needs to live somewhere, and often that's in the state of the component rendering the HTML. Global state on the other hand is shared across components. Imagine grabbing a user session form your API and storing that somewhere so that you can use the user's name and email to display across your application. If we used a pure component architecture to store global state we have to fetch data in the top level component and then pass it down through all other components to the one that needs it much like the the [`<Controller />`](https://github.com/cbelsole/controller_hooks_example/blob/controller_component_pattern/src/components/controller.tsx) passing the [`completeTodo()`](https://github.com/cbelsole/controller_hooks_example/blob/controller_component_pattern/src/components/controller.tsx#L20) function through the [`<TodoList />`](https://github.com/cbelsole/controller_hooks_example/blob/controller_component_pattern/src/components/todoList.tsx) to the [`<Todo />`](https://github.com/cbelsole/controller_hooks_example/blob/controller_component_pattern/src/components/todo.tsx) component so that the button on the [`<Todo />`](https://github.com/cbelsole/controller_hooks_example/blob/controller_component_pattern/src/components/todo.tsx) component can modify the state of a to-do. We can see in this example that this leads to tight coupling of components.

Why do we want to avoid tight coupling? Imagine writing a complex grocery store application where everything is tied to a single payments processing system. Something happens to them internally and now your payments processing system is shutting down. How are you going to integrate a new payments processing system into your application? You have to rewrite your ordering, refunding, and revenue systems which incurs a lot of risk when these things are so critical to your business. Alternatively, let's think of a scenario where your payments processing system is behind an abstraction. The abstraction is aware of orders and knows how to refund and calculate revenue. Now when you need to rewrite your system to deal with all the new code you only have to rewrite the logic underneath that abstraction. This is also the [D in SOLID](https://en.wikipedia.org/wiki/Dependency_inversion_principle).

Following [SOLID](https://en.wikipedia.org/wiki/SOLID) principals and thinking ahead about how your components are tied together are ideas that make a long lasting application maintainable. It is often faster to write code that works in the now, but if you engrain these concepts into your fingers future you will have a much easier time dealing with bugs and changing your software.

## Let's put the background into pratice

With all that in mind let's dive into the code. First we'll write our [`<TodoProvider />`](https://github.com/cbelsole/controller_hooks_example/blob/master/src/components/todoProvider.tsx) that holds our global state with the capability to get and complete to-do's from the API. Notice that it returns its children wrapped in the provider. This is what allows us to use the context in the component chain.

```typescript
import React, { createContext, useState } from "react";
import {
  getTodos as getTodosAPI,
  completeTodo as completeTodoAPI,
  iTodo
} from "../api/todos";

interface iTodoContext {
  todos: iTodo[];
  completeTodo: (item: string) => void;
  getTodos: () => void;
}

interface TodoProviderProps {
  children?: React.ReactNode;
}

export const todoContext = createContext<iTodoContext>({
  todos: [],
  completeTodo: () => {},
  getTodos: () => {}
});

const TodoProvider: React.FunctionComponent = ({
  children
}: TodoProviderProps) => {
  const [todos, setTodos] = useState<iTodo[]>([]);

  const getTodos = () => getTodosAPI().then(todos => setTodos(todos));

  const completeTodo = (item: string) =>
    completeTodoAPI(item).then(todos => setTodos(todos));

  return (
    <todoContext.Provider value={{ todos, completeTodo, getTodos }}>
      {children}
    </todoContext.Provider>
  );
};

export default TodoProvider;
```

Next, we'll wrap our [`<Controller />`](https://github.com/cbelsole/controller_hooks_example/blob/master/src/components/controller.tsx) in the provider so that we can call [`useContext()`](https://reactjs.org/docs/hooks-reference.html#usecontext) within the component chain:

```diff
diff --git a/src/App.tsx b/src/App.tsx
index f7b1217..83ce739 100644
--- a/src/App.tsx
+++ b/src/App.tsx
@@ -1,10 +1,13 @@
 import React from 'react';
 import Controller from './components/controller';
+import TodoProvider from './components/todoProvider';

 function App() {
   return (
     <div>
+      <TodoProvider>
         <Controller />
+      </TodoProvider>
     </div>
   );
 }
```

We'll then rewrite our [`<Controller />`](https://github.com/cbelsole/controller_hooks_example/blob/master/src/components/controller.tsx) to call [`useContext()`](https://reactjs.org/docs/hooks-reference.html#usecontext) to get to-do's and pass them down to it's children while breaking the dependency of passing down the [`completeTodo()`](https://github.com/cbelsole/controller_hooks_example/blob/master/src/components/todoProvider.tsx#L31) function making the component chain loosely coupled since it still relies on the data but not the interactions.

```diff
diff --git a/src/components/controller.tsx b/src/components/controller.tsx
index a4eaac9..1159fc7 100644
--- a/src/components/controller.tsx
+++ b/src/components/controller.tsx
@@ -1,13 +1,9 @@
-import React, { useState, useEffect } from 'react';
-import { getTodos, completeTodo as completeTodoAPI, iTodo } from '../api/todos';
+import React, { useEffect, useContext } from 'react';
 import TodoList from './todoList';
+import { todoContext } from './todoProvider';

 const Controller: React.FunctionComponent = () => {
-  const [todos, setTodos] = useState<iTodo[]>([]);
-
-  const completeTodo = (item: string) => {
-    completeTodoAPI(item).then(todos => setTodos(todos));
-  };
+  const { todos, getTodos } = useContext(todoContext);

   const isAllComplete = () => {
     for (let i = 0; i < todos.length; i++) {
@@ -21,8 +17,8 @@ const Controller: React.FunctionComponent = () => {
   };

   useEffect(() => {
-    getTodos().then(todos => setTodos(todos));
-  }, []);
+    getTodos();
+  }, [getTodos]);

   if (!todos.length) {
     return <div>loading...</div>;
@@ -30,7 +26,7 @@ const Controller: React.FunctionComponent = () => {

   return (
     <div>
-      <TodoList completeTodo={completeTodo} todos={todos} />
+      <TodoList todos={todos} />
       <button onClick={isAllComplete}>All complete?</button>
     </div>
   );
```

[`<TodoList />`](https://github.com/cbelsole/controller_hooks_example/blob/master/src/components/todoList.tsx) also gets edited to no longer pass down the [`completeTodo()`](https://github.com/cbelsole/controller_hooks_example/blob/master/src/components/todoProvider.tsx#L31) function.

```diff
diff --git a/src/components/todoList.tsx b/src/components/todoList.tsx
index e69edba..4f664b8 100644
--- a/src/components/todoList.tsx
+++ b/src/components/todoList.tsx
@@ -4,15 +4,14 @@ import Todo from './todo';

 interface Props {
   todos: Array<iTodo>;
-  completeTodo: (item: string) => void;
 }

-const TodoList: React.FC<Props> = ({ todos, completeTodo }) => {
+const TodoList: React.FC<Props> = ({ todos }) => {
   return (
     <ul>
       {todos.map(todo => (
         <li>
-          <Todo completeTodo={completeTodo} {...todo} />
+          <Todo {...todo} />
         </li>
       ))}
     </ul>
```

Finally [`<Todo>`](https://github.com/cbelsole/controller_hooks_example/blob/master/src/components/todo.tsx) calls [`useContext()`](https://reactjs.org/docs/hooks-reference.html#usecontext) to get the [`completeTodo()`](https://github.com/cbelsole/controller_hooks_example/blob/master/src/components/todoProvider.tsx#L31) function and update itself.

```diff
diff --git a/src/components/todo.tsx b/src/components/todo.tsx
index 47b0e44..75de4ff 100644
--- a/src/components/todo.tsx
+++ b/src/components/todo.tsx
@@ -1,11 +1,12 @@
-import * as React from 'react';
+import React, { useContext } from 'react';
 import { iTodo } from '../api/todos';
+import { todoContext } from './todoProvider';

-interface Props extends iTodo {
-  completeTodo: (item: string) => void;
-}
+interface Props extends iTodo {}
+
+const Todo: React.FC<Props> = ({ item, done }) => {
+  const { completeTodo } = useContext(todoContext);

-const Todo: React.FC<Props> = ({ item, done, completeTodo }) => {
   return (
     <div>
       task: {item} is {done ? 'done' : 'not done'}{' '}
```

After all that we have an abstracted functional app working off of global and local state where appropriate. I hope you found this guide useful. Please get in contact with me if you have any feedback.
