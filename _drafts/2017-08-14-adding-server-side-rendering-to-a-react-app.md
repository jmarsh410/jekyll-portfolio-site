---
layout: post
title: 'Adding Server Side Rendering to a React App'
---
# {{ page.title }}
This is part of a series of posts in which I add features to a web app. The name of the app is Festival Hopper, and its a simple app that uses the Untappd API to create lists and batch check in items.

## Ejecting from Create React App
The app was initially created using Create React App. While Create React App is a great project to use for quick scaffolding and development of React Apps, it does not include server side rendering. So first I had to eject the project. 

## Refactoring for Heroku
I wanted to use Node and Express for my server, and I also didn't want to pay money to get things up and running, so I decided to use Heroku. They are a PAAS (platform as a service) and provide free levels for accounts. This involves creating 



- add heroku files 
- re-implement webpack compiling
- using react-dom-server in index.js 
- refactoring code base to work on either server or client environment
    - when to use local storage, fetch, etc 
- transpile index.js because it uses react jsx
- 




### Request and response multiplexing
Basically, allowing multiple resources to be requested through the same TCP connection, where previously only 4-8 resources could be retreived at a time. This lead to slower speeds and congestion before. now, instead of using all of those 4-8 connections, 1 connection does everything. From google: 'All communication is performed over a single TCP connection that can carry any number of bidirectional streams'. So, 1 TCP connection handles the communication of requests and responses within individual items called BINARY-ENCODED FRAMES. Requests and responses live within a stream, which is made up of Binary-Encoded Frames. And the streams live within the tcp connection. and ONE TCP connection now handles all of these streams instead of 4-8. HTTP/1.1 used TCP connections inefficiently, and only allowed one response to be delivered at a time and each TCP connection handles only one request.

<br/>

### Prioritize more important assets
The server can specify a prioritization tree that delivers more important files first, and can also specify dependencies on other files. the prioritization tree looks like a tree data structure, with nodes and each node having a value. parent nodes are loaded first with the idea that the child nodes depend on the parent node, then the child nodes are allocated resources according to their value. the key word here is PRIORITIZATION. this does not mean the files will be delivered in the exact order you specified, that would not be desired in case a file was blocked for some reason.

