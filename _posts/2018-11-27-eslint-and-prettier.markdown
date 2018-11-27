---
layot: post
title:  "Integration of Eslint and Prettier"
date:   2018-11-22 17:30:00 -0500
categories: eslint
tags: [eslint, prettier]
---

### Install packages and plugins

* eslint
* prettier
* eslint-plugin-prettier
* eslint-config-prettier

*[reference link](https://prettier.io/docs/en/eslint.html)*

### Tips

The so called integration of `Eslint` and `Prettier` is to use `eslint cli` to check `prettier` rules, or allow editors to show `prettier` errors on the files.

`Prettier` is for styling tool, `eslint` is a javascript linter. You should avoid using eslint to do styling which may cause conflict rules between `prettier` and `eslint`