---
layout: post
title:  "Dealing with API Calls in React and Redux, Part One."
date: 2017-10-27
category: webdev
tags: react redux API
---

When building a web application using React, there's a very strong likelihood of needing to interface with an API. It may be one that you or your company have built, or it might be one provided by a 3rd party. Regardless, you're going to want to retrieve and display data from it to your users.

This post will discuss a common pattern in React for retrieving data from an API as well as common Redux patterns which you might like to introduce to improve this flow.

# React Only

The first way you might try to retrieve data is to make your fetch call directly inside your [container component](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.1k4schs31). Suppose you want to display a list of Posts and your container renders a `PostList` component which takes a prop of `posts`.

This might look something like this:

```javascript
import React, { Component } from "react";

import PostList from "./PostList";

class Posts extends Component {
	state = {
		posts: []
	}

	componentDidMount() {
		const fetchConfig = {
			method: "GET",
			headers: new Headers({ "Content-Type": "application/json" }),
			mode: "cors"
		}

		fetch("www.example.com/posts", fetchConfig)
			.then(response => {
				if (response.ok) {
					const result = response.json().then(json => {
						this.setState({ posts: json })
					});
				}
			})
			.catch(error => console.log(error))
	}

	render() {
		const { posts } = this.state;

		return (
			<PostList posts={posts} />
		)
	}
}
```

In the above example we are first defining our components local state with an empty array for `posts`. This is what we will use to pass to our `PostList` component. We're then using the lifecycle method `componentDidMount` to fire off fetch with some standard configuration. If you're unfamiliar with promises, this part may look confusing, but essentially we are waiting to get a response from the API and then using `this.setState` to update the component state with the JSON response from the API.

## Pros and Cons
This approach is great if you are only in need of a quick and easy way to access data in a small React app. However, it will start to get pretty frustrating as your app grows. For example, suppose you want to add multiple routes to your app using something like [react-router](https://github.com/ReactTraining/react-router). When you navigate away from this page, and then navigate back, every time it your hit the page it will fire the fetch inside of `componentDidMount`. This would be undesirable for many reasons, but not limited to:
* Unncessary network requests 
* Causing multiple re-renders of your app
* Lots of duplicate code

Additionally, the data retrieved from this endpoint is only good for this component heirarchy. What if you wanted to have a Navigation bar which had a number which counted how many posts you have? You would also need to make it fetch and retrieve all your posts too. Which means lots of wasted network requests and lots of duplicate code.

# Redux
That last example with the Navigation bar is where [Redux](http://redux.js.org/) really shines. It empowers you to lift your state outside of just local components up to the entire application level. You can then share it across components in different heirarchies without the need for much duplication.

Redux introduces the use of "reducers" and "actions". I won't spend time going into the detail of what they are or how they work, but you should have a look at the Redux documentation if you find yourself lost. The documentation is _fantastic_.

Following our Posts example from before, let's see how this might look with Redux.

```javascript
// posts.js whehre your redux logic is going to live.

const fetchPosts = () => {
	const fetchConfig = {
		method: "GET",
		headers: new Headers({ "Content-Type": "application/json" }),
		mode: "cors"
	}

	return fetch("www.example.com/posts", fetchConfig)
		.then(response => {
			if (response.ok) {
				return response.json().then(json => {
					return {
						type: "FETCH_POSTS",
						payload: json
					}
				})
			}
		})
		.catch(error => console.log(error))
}

```

```javascript
// TODO: Post List

```