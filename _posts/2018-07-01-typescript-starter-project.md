---
layout: post
title: Typescript starter project for Node.js
category: Node.js
tags: [Node.js, Linux, typescript]
---

### What is Node.js?

Node.js is a server side interpreter for JavaScript. Using it you can write server 
side application and scripts in JavaScript.  

### What is a typescript?

Typescript is a typed superset of JavaScript which provides static typing (optional), classes and 
interfaces. It provides a lot better support for tools and at least in my opinion is much more 
readable than JavaScript. 

### Howto

Here is a quick instruction howto setup a starter project that uses Typescript for Node.js

```bash
mkdir typescript-starter
npm init -y
npm install --save typescript
npm install --save ts-node
npm install --save-dev @types/node
npx tsc --init
```

If you want to use more modern JavaScript features you should edit ```tsconfig.json``` and replace
following lines:
```json
"compilerOptions": {
    /* Basic Options */
    "target": "ES5",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */
     ...

```

With desired version:
```json
"compilerOptions": {
    /* Basic Options */
    "target": "ES2015",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */
     ...

```

Create and execute test script:
```bash
echo "console.log('Hello World!')" > main.ts
npx ts-node main.ts
Hello World!
```

Having that starter project we should be able to write scripts using Typescript.

Here is a starter project on GitHub: [typescript-starter](https://github.com/DariuszOstolski/typescript-starter)  


