---
layout: post
title: "Series: React, Redux, and API's"
subtitle: "Part One: React 
Only"
date: 2018-03-20
category: webdev
tags: react redux API
image: react_plus_api_part_one.svg
---

When building a web application using React, there's a very strong likelihood you will need to interface with an API. It may be one that you or your company have built, or it might be one provided by a 3rd party. Regardless, you're going to want to retrieve data from it and display that data to your users.

This series will cover the different ways for your React and, later, React/Redux app to talk to an API, as well as the and the pros and cons of each approach.

This first post will be very simple and cover using only React.

# Fetching in a Container

The first way you might try to retrieve data is to make your fetch call directly inside your [container component](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.1k4schs31). Suppose you want to display a list of Posts and your container renders a `PostList` component which takes a prop of `posts`.

This might look something like this:

```javascript
import React, { Component } from "react";

import PostList from "./PostList";

class Posts extends Component {
	state = {
		posts: []
	}

	async componentDidMount() {
		const fetchConfig = {
			method: "GET",
			headers: new Headers({ "Content-Type": "application/json" }),
			mode: "cors"
		}

		const response = await fetch("https://jsonplaceholder.typicode.com/posts/", fetchConfig);
		
		if (response.ok) {
			const posts = await response.json();
			this.setState({ posts });
		} else {
			console.log("error!", error);
		}
	}

	render() {
		const { posts } = this.state;

		return (
			<PostList posts={posts} />
		)
	}
}
```

#### Here's a working jsFiddle of this example: [https://jsfiddle.net/patrickgordon/69z2wepo/142592/](https://jsfiddle.net/patrickgordon/69z2wepo/142592/)

In the above example we are first defining our components local state with an empty array for `posts`. This is what we will use to pass to our `PostList` component. We're then using the lifecycle method `componentDidMount` to fire off a fetch with some standard configuration. If you're unfamiliar with promises and async/await, this part may look confusing but essentially we are waiting to get a response from the API and then using `this.setState` to update the component state with the JSON response from the API. This triggers a re-render and the data is then displayed in the list to the user.

## Pros and Cons
This approach is great if you are only in need of a quick and easy way to access data in a small React app. However, it will start to get pretty frustrating as your app grows. For example, suppose you want to add multiple routes to your app using something like [react-router](https://github.com/ReactTraining/react-router). When you navigate away from this page, and then navigate back, every time you hit the page it will fire the fetch inside of `componentDidMount`. This would be undesirable for many reasons, but not limited to:
* Unncessary network requests
* Causing multiple re-renders of your app
* Lots of duplicate code

Additionally, the data retrieved from this API is only good for this component heirarchy. What if you wanted to have a Navigation bar which had a number which counted how many posts you have? You would also need to make it fetch and retrieve all your posts too. Which means lots of wasted network requests and lots of duplicated code.

# Up Next
In the next part of this series I will explain how to use utility functions to ensure code-reuse when dealing with APIs in React only.
