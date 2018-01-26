---
title: Up And Running With React & Redux & Three.JS Part 3
date: 2018-01-26 14:10:29
tags: react redux three.js
---


## Part 3 ... finally the fun stuff
In the first part of this post, I walked through the basic setup of a React application. Part 2 detailed the setup for Redux to manage our state. Part 3 will now document setup for integrating a three.js scene that is hooked up to state.

A mention should be given to libraries like [react-redux-renderer](https://github.com/toxicFork/react-three-renderer) and [react-three](https://github.com/Izzimach/react-three). A decent amount of work went into each of these, however they're only compatible with React 15. When building this, I didn't want to be dependent on an older version of any particular package or library in case I picked this up again in some time.

The only package we'll be relying on is [three](https://www.npmjs.com/package/three). I suggest crawling around the docs if you haven't already spent some time with the library. I've spent a fair amount of time in the past troubleshooting rendering of scene and elements, so if there's something you feel is not given adequate explanation, I'd love the feedback.
