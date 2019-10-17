---
layout: post
title:      "Redirecting with React Router"
date:       2019-10-17 21:54:28 +0000
permalink:  redirecting_with_react_router
---


There are various ways to redirect a user after an event, but I’ll be using React Router’s `<Redirect>` in this example. 

Let’s start by importing Redirect in the file we’ll be using it in. 

```
import { Redirect } from 'react-router-dom';
```

I want the user to be redirected to another page (or component) after submitting a form. We can trigger the redirect at the end of a form submission function. 

Let’s say we have a form to submit a new post. In our React class component, we can set up local state like so:

```
state = {
        post: {
            ...
        },
        submit: false
    };
```

In our `handleSubmit`, after the action for creating a new post is dispatched, we can reset the form’s local state and set the submit key to true.

```
handleSubmit = (event) => {
        event.preventDefault();
        this.props.createPost(this.state.post);
        this.setState({
            post: {
                ...
            },
            submit: true
        });
    };
```

Lastly, before the return statement in our `render()`, we need to add a conditional statement that will redirect the user if the `this.state.submit` is true. 

```
render() {

        if (this.state.submit) {
            return <Redirect to="/posts" />
        }

        return (
            <Form onSubmit={this.handleSubmit} >
      ...
           </Form>
        )

    }

```

In React, when a component’s state is changed, a re-render is triggered. We changed the component’s state using `setState`, which caused the component to re-render and evaluate our conditional statement. If the statement is true, the user will be redirected to the path specified in the React Router Redirect tag. 
