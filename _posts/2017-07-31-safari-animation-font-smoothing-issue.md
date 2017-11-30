---
layout: post
title: 'Solving the Safari font smoothing issue during animations'
---
# {{ page.title }}
I've often come across an issue when adding animations to elements on a page that I can never quite seem to get the answer to. Basically, in Safari, and occasionally Chrome, I'll add an animation to an element and all the text inside that element will change from subpixel-antialiased font smoothing to antialised font smoothing for the duration of the animation, and then switch back once the animation is complete. This is common for animation handled by the GPU. However, I noticed that it wasn't happening to every element I was animating, some elements with text inside appeared just fine. So, I embarked to find out why the behavior was so inconsistent.

## The Fix?
After some random testing, I found that the change to antialiased from subpixel-antialiased text doesn't occur is the element has a background color specified.

I simply set a background-color, and the text animates fine now. I seriously need to remember this for future cases where this occurs. 