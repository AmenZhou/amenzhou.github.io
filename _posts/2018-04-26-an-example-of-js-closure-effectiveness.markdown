---
layout: post
title:  "A Show Case of Js Closure and Destructing Assignment"
date:   2018-04-27 10:30:00 -0500
categories: javascript
tags: [js_closure, es6]
comments: true
---

As a Js beginner, it is quite hard to understand the effectiveness of Js closure. Today I am going to use an example to demo how we can leverage Js closure for object destructing in real life.

In Redux world, a typical reducer function is like this

```javascript
 const todoReducer = (state, action) => {
    let new_list;
    switch(action.type) {
      case ADD_ITEM:
        return {
          list: [
            ...state.list,
            action.payload.data.item
          ]
        }
      case DELETE_ITEM:
        newList = state.list.filter(item => action.payload.data.itemKey !== item.key)

        return {
          list: [
            ...newList
          ]
        }
      case EDIT_ITEM:
        newList = state.list.filter(item => action.payload.data.item.key !== item.key)

        return {
          list: [
            ...newList,
            action.palyload.data.item
          ]
        }
    }
  }
```

Do you find that the code is not neat? There are too many chain fetchings on the `action` object.

What is the best part of ES6? The destructing syntax is one of my favorite ones. Let's use destructing to power this code.

```javascript
 const todoReducer = ({ list }, { payload, type }) => {
    let new_list;
    switch(type) {
      case ADD_ITEM:
        return {
          list: [
            ...list,
            payload.data.item
          ]
        }
      case DELETE_ITEM:
        newList = list.filter(item => payload.data.itemKey !== item.key)

        return {
          list: [
            ...newList
          ]
        }
      case EDIT_ITEM:
        newList = state.list.filter(item => payload.data.item.key !== item.key)

        return {
          list: [
            ...newList,
            palyload.data.item
          ]
        }
    }
  }
```

Looks better? But I want to move one step further. I am going to add some spices - Js closure, to destruct all chained objects.

```javascript
 const todoReducer = ({ list }, { payload, type }) => {
    switch(type) {
      case ADD_ITEM: {
        const { data: { item } } = payload;

        return {
          list: [
            ...list,
            item
          ]
        }
      }
      case DELETE_ITEM: {
        const { data: { itemKey } } = payload;

        const newList = list.filter({ key } => itemKey !== key)

        return {
          list: [
            ...newList
          ]
        }
      }
      case EDIT_ITEM: {
        // To avoid duplication of key, I use alias name itemKey
        const { data: { item: { key: itemKey }, item } } = payload;

        const newList = state.list.filter(({ key }) => key !== itemKey)

        return {
          list: [
            ...newList,
            item
          ]
        }
      }
    }
  }
```

**As you can see, I added curly brace pairs to wrap each `case` blocks so that the destructing constants are all in their own closures. Even the destructed consts are having same names in each block, but they only affect inside their own blocks **
