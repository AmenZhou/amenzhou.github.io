---
layout: post
title:  "3 Ways to Compose CSS in React Js"
date:   2018-11-22 17:30:00 -0500
categories: css
tags: [css, reactjs]
---

### CSS files without Webpack

This is the same life prior to having Webpack, create CSS files in Rails assets pipe line and call class names or ids in the react component.

`rails_path/app/assets/stylesheet/main.css`
```css
  .my-class {
    width: 100%;
    margin: 2% auto;
    ...
  }
```

`rails_path/web/components/my-component.js`
```js
  const MyComponent = props =>
    <div className="my-class">Hello World</div>

  export default MyComponent;
```

### CSS-in-Js

**Case 1**
`rails_path/web/components/my-component.js`
```js
  const MyComponent = props =>
    <div style="width: 100% margin: 2%, auto;">Hello World</div>

  export default MyComponent;
```

**Case 2: use typestyle**
`rails_path/web/components/my-styles.js`
```js
  import { style } from 'typestyle';

  export const helloWorld = style({
    width: 100%,
    ...
  });
```

`rails_path/web/components/my-component.js`
```js
  import { helloWorld } from './my-styles';

  const MyComponent = props =>
    <div className={helloWorld}>Hello World</div>

  export default MyComponent;
```

### CSS files with Webpack compilation
