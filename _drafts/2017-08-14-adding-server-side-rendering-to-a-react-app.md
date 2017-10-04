---
layout: post
title: 'Adding Server Side Rendering to a React App'
---
# {{ page.title }}
This is part of a series of posts in which I add features to a web app. The name of the app is Festival Hopper, and its a simple app that uses the Untappd API to create lists and batch check in items.

## Ejecting from Create React App
The app was initially created using Create React App. While Create React App is a great project to use for quick scaffolding and development of React Apps, I wanted to have more fine tuned control over the environment, both for learning purposes and to implement features that Create React App didn't support, like server side rendering. So I ejected useing 'npm run eject'. Next, I made a webpack config for my client-side code from scratch, using the old one from Create React App as inspiration and guidance when needed. Sidenote: I also switched to using ESLint, enforced with AirBnB styles, so I made quite a few edits and started renaming many of my react files to have the .jsx file extension.

## Refactoring for Server Rendering
I decided to use Node and Express for my server code because I'm most familiar with them. I took the base file structure from ejecting my app, and added an index.jsx file (jsx because i'll be including a bit of jsx code related to the untappd app). What index.jsx does:
    - launches a local server
    - listens for all requests
    - checks for login status using cookieParser package
    - redirects the user
    - uses the request url to render my react module. To accomplish this I actually had to create a slightly different webpack bundle for my server. It differs from the bundled client code in that I bundle all the react js as a module that can be required, instead of as code to be executed immediately once the browser loads it.
    - takes that react html and pumps it into a nunjucks template to create the full page's html content
Since i'm using jsx inside of index.js, I need to run it through babel before using it. So in my package.json I add a script command that compiles index.jsx and watches it for further changes. Looks like this: `build:index": "babel index.jsx -w -o index-compiled.js`. At this point, I can launch my server locally and have my Untappd app up and running correctly. Any changes I make to my index files or react project files while developing are being watched and recompiled/bundled when changes are made.

## Putting it on Heroku
I wanted to use Node and Express for my server, and I also didn't want to pay money to get things up and running, so I decided to use Heroku. They are a PAAS (platform as a service) and provide free levels for accounts. I created an account and went through the fairly quick [Getting Started Guide](https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction) for Node. It goes over set up, deployment, scaling, etc of a Heroku app.




- add heroku files 
- re-implement webpack compiling
- using react-dom-server in index.js 
- transpile index.js because it uses react jsx
- adding in session id's for logging in
- refactoring code base to work on either server or client environment
    - when to use local storage, fetch, etc 

