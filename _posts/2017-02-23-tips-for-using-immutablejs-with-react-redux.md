---
layout: post
title:  "Tips for using ImmutableJS with React, Redux"
date: 2017-02-23
category: webdev
tags: react redux immutablejs
---

I recently started overhauling the frontend for [SimpleRM](http://simplerm.co) because it wasn't performant and I have improved my React & Redux work a lot since I started building it. I decided that this would be a good opportunity to explore the popular library [ImmutableJS](link). I have had many recommendations from other FE devs and was eager to give it a go.

This post will cover, briefly, some of the reasons you might be interested in ImmutableJS with your react/redux projects as well as some tips I have from using it.

## Why ImmutableJS?

As the name suggests, ImmutableJS is focused around immutable objects. That is, an object whose state cannot be modified after it is created.

Instead, ImmutableJS takes your data, mutates it, and if there is any changes returns a new reference. If there isn't, however, it just returns the initial reference. 

Ultimately, this means that it will speed up your app. By only returning a new reference if the data has changed, your react/redux app will limit the number of unncessary re-renders. 

This is loosely touched on in the [redux documentation](http://redux.js.org/docs/recipes/reducers/PrerequisiteConcepts.html#prerequisite-reducer-concepts):

> For React Redux, connect checks to see if the props returned from a mapStateToProps function have changed in order to determine if a component needs to update. To improve performance, connect takes some shortcuts that rely on the state being immutable, and uses shallow reference equality checks to detect changes. This means that changes made to objects and arrays by direct mutation will not be detected, and components will not re-render.

## Tips

### 1. You can use `.filter()`, `.map()`, `.reduce()`, etc. on all of the ImmutableJS collections
Sad that you can only use `.filter()` on arrays? Well, you can use them on all of the data structures that ImmutableJS brings to the table. That includes `Map()` which is similar to an object.

### 2. Make sure to use `.get()` in `mapStateToProps`
I had the following code:

```javascript
const mapStateToProps = state => ({
  isFetching: state.contacts.isFetching,
  items: state.contacts.items,
});
```

I couldn't for the life of me figure out why, after a successful redux action, my components were not re-rendering. Instead, I needed to have:

```javascript
const mapStateToProps = state => ({
  isFetching: state.contacts.get('isFetching'),
  items: state.contacts.get('items'),
});
```

### 3. Creating initialState with ImmutableJS
Here's an example of how to use initialState with ImmutableJS and in your reducer.

```javascript
// SomeModule.js

// ... other imports
import Immutable from 'immutable';

// other stuff

const { Map, List } = Immutable;

const initialState = Map({
  foo: 'bar',
  items: List(),
});

// then use it in your reducer.
export default function someReducer(state = initialState, action) {
  switch (action.type) {
    default:
      return state;
  }
}

```

### 4. Using ImmutableJS in a redux reducer:
Use `.merge` and `.mergeDeep` to ensure immutablity:

```javascript
const initialState = Map({
  isFetching: false,
  meta: Map({
    recordCount: 0,
    offset: 0,
  }),
  items: List(),
});

export default function contactsReducer(state = initialState, action) {
  switch (action.type) {
    case FETCH_ALL_REQUEST:
      return state.merge({
        isFetching: true,
      });
    case FETCH_ALL_SUCCESS:
      return state.mergeDeep({
        isFetching: false,
        meta: action.payload.meta,
        items: action.payload.data,
      });
    default:
      return state;
  }
}
```

### 5. Using `.getIn()` to retrieve from nested Maps or Lists
If you want to get a value that is nested you can use `.getIn()`:

```javascript
recordCount: state.contacts.getIn(['meta', 'recordCount']),
```

### 6. Use `.find()` to retrieve a single value from an ImmutableJS collection
Using `.filter()` on an ImmutableJS collection will return a new iterable of the same type. i.e. `.filter()` on a Map will result in a Map and `.filter()` on a List will result in a List.

If you have a List of items which consists of Maps you may want to not pass a List and instead just one Map.

You can use `.find()` instead as it will return the first value for which the predicate returns true. This is great for use with unique keys such as an `id`.

```javascript
const mapStateToProps = (state, ownProps) => ({
  isFetching: state.contacts.get('isFetching'),
  item: state.contacts.get('items')
                      .find(item => item.get('id') === ownProps.params.id),
});
```

# Further Reading:
* http://reactkungfu.com/2015/08/pros-and-cons-of-using-immutability-with-react-js/
* https://www.toptal.com/react/react-redux-and-immutablejs