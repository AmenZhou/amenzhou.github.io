---
layout: post
title:  "3 Ways to Compose CSS in React Js"
date:   2018-11-22 17:30:00 -0500
categories: css
tags: [css, reactjs]
---

### CSS files without Webpack

This is the same life prior to having Webpack, create CSS files in Rails assets pipe line and call class names or ids in the react component.

`app/assets/stylesheet/main.css`
```css
  .my-class {
    width: 100%;
    margin: 2% auto;
    ...
  }
```

`web/components/my-component.js`
```js
  const MyComponent = props =>
    <div className="my-class">Hello World</div>

  export default MyComponent;
```

### CSS-in-Js

**Example 1**
`web/components/my-component.js`
```js
  const MyComponent = props =>
    <div style="width: 100% margin: 2%, auto;">Hello World</div>

  export default MyComponent;
```

**Example 2**
Have a separate Js style file

`web/components/my-styles.js`
```js
  import { style } from 'typestyle';

  export const helloWorld = style({
    width: 100%,
    ...
  });
```

`web/components/my-component.js`
```js
  import { helloWorld } from './my-styles';

  const MyComponent = props =>
    <div className={helloWorld}>Hello World</div>

  export default MyComponent;
```

### CSS files with Webpack compilation

`web/components/my-styles.css`

```css
  .myClass {
    width: 100%;
    margin: 2% auto;
    ...
  }
```

`web/components/my-component.js`
```js
  import style from './my-styles.css';

  const MyComponent = props =>
    <div className={style.myClass}>Hello World</div>

  export default MyComponent;
```

`webpack.config.js`
```js
  module: {
    plugins: [
      // extract css file and export to rail pipe line
      new ExtractTextPlugin('app/assets/stylesheets/main.css')
    ],
    rules: [
      {
        test: /\.css$/
        use: ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: ['css-loader', 'postcss-loader']
        })
      }
    ]
  }
```
