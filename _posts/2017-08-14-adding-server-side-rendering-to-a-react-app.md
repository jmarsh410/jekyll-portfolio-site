---
layout: post
title: 'Adding Server Side Rendering to a React App'
---
# {{ page.title }}
This is part of a series of posts in which I add features to a web app. The name of the app is Festival Hopper, and its a simple app that uses the Untappd API to create lists and batch check in items.

## Ejecting from Create React App
The app was initially created using Create React App. While Create React App is a great project to use for quick scaffolding and development of React Apps, I wanted to have more fine tuned control over the environment, both for learning purposes and to implement features that Create React App didn't support, like server side rendering. So I ran 'npm run eject' and re-created the build process from scratch. I started off by making a webpack config for my client-side code, using the old one from Create React App as inspiration and guidance when needed. I also switched to using ESLint, enforced with AirBnB styles, which meant making quite a few edits and renaming many of my react files to have the .jsx file extension.

## Adding Server Rendering
Server side rendering refers to the process of building out the webpages of an application on the server, filling them with all necessary content for the requested url, and sending it to the client. This is unlike the usual way, which is to send  a mostly empty html page to the client and wait until the app's javascript file loads and populates the page with content. Server side rendering allows you to cut out the middleman and present a page more quickly, with the downside being that it takes a bit longer to implement.

I decided to use Node and Express for my server code because I'm most familiar with them. I took the base file structure from ejecting my app, and added an index.jsx file (jsx because i'll be including a bit of jsx code related to the untappd app). What index.jsx does:
    - launches a local server
    - listens for all requests
    - checks for login status using cookieParser package
    - redirects the user when necessary
    - uses the request url to render appropriate react modules. To accomplish this I actually had to create a slightly different webpack bundle for my server. It differs from the bundled client code in that I bundle all the react js as a module that can be required, instead of as code to be executed immediately once the browser loads it.
    - takes that react html and pumps it into a nunjucks template to create the complete page.
Since i'm using jsx inside of index.js, I need to run it through babel, so in my package.json I add a script command that compiles index.jsx and watches it for further changes. Looks like this: `build:index": "babel index.jsx -w -o index-compiled.js`.

Another requirement when moving from strictly client-side development is that you have to make your code either aware of its environment or agnostic of it. Certain API's that I was using on the client side don't exist on the server, so I had to go through my codebase and add environment checks to do specific things depending on the environment.

At this point, I can launch my server locally and have Festival Hopper up and running correctly. Any changes I make to my index files or react project files while developing are being watched and recompiled/bundled when changes are made. Most importantly though, when I request a page, i receive a fully flushed out html page from the server.

## TODOS
- Production build: Right now I don't have a production webpack build. So I'd like to make one that makes sure I'm bundling everything to be as lightweight as possible.