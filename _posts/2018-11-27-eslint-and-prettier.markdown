---
layot: post
title:  "Integration of Eslint and Prettier"
date:   2018-11-22 17:30:00 -0500
categories: eslint
tags: [eslint, prettier]
comments: true
---

### This post is not done yet

## Why do I need both `Eslint` and `Prettier`?

`Eslint` is a language linter and `Prettier` is a formatter. They are having some intersections but not duplicated.

## How can I make them work together?

### First of all, you need to install these 3 packages

* eslint
* prettier
* [prettier-eslint-cli](https://github.com/prettier/prettier-eslint-cli)

### Config `Eslint` and `Prettier` rules

### Make it work

`prettier-eslint` does the jobs of both `eslint --fix` and `prettier` and always regards `eslint` rules above `prettier` if there are conflicts rules present.

A simple command to kickoff the formatting `prettier-eslint [path/to/files]`
