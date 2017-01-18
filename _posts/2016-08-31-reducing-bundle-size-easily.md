---
layout: post
title:  "How I reduced my bundle size by 400kBs by changing only a few lines of code"
date: 2016-08-31
categories: "Web Development"
tags: react webpack optimisation
---

Like a lot of people, I use [Slack](https://slack.com/) to remain involved in the software developer communities in my city & country. On a particular Slack I frequent, we were discussing a code sample and how I had imported [lodash](https://lodash.com/). I was importing lodash like this:

`import {forEach} from 'lodash';`

There were a number of suggestions to change this to:

`import forEach from 'lodash/forEach';`

Eventually, this led on to a question about the bundle size of [SimpleRM](https://simplerm.co) and so I did a bit of exploration.

##### ---- Warning: big bundle size incoming. Optimisation fiends, avert your eyes ----
 My production bundle size came in at 1.7 MBs which caused some of the others to be alarmed. At this point, someone suggested refactoring _just_ how lodash was being imported and see if that made a difference.

I took them up on this and took the following screenshot of the dev `app.js` build before changing anything:

![Pre change](https://i.gyazo.com/2d8414e554f707c33a710afc8f413636.png)

So it comes it at 5.96MBs in development.

I then did a find in my [IDE of choice](https://www.jetbrains.com/webstorm/) and found 15 files which were using lodash. In some cases I even had imports which weren't actually being used (oops). I changed all of these to the way mentioned above. Ultimately it resulted in 15 changed files with 42 additions and 36 deletions.

Here's the result:

![Post change](https://i.gyazo.com/11076c09ed59e631a2669ac2796453bc.png)

So, basically by just changing how I was importing the lodash functions I managed to save ~400kBs but I had no idea **why**.

#### The Why

I tried for quite some time but my Google-fu wasn't strong enough and I couldn't find a really clear answer on why, but I did find this on [Stackoverflow](http://stackoverflow.com/questions/34239731/how-to-minimize-the-size-of-webpacks-bundle):
> You are probably importing the react-bootstrap and material-ui components using the library way...
This is nice and handy but it does not only bundles Button and FlatButton (and their dependencies) but the whole libraries.

So, I'm assuming that importing it this way was bringing in way more lines of code then we needed. Judging by the comments for that SO question, I'm not the only one who doesn't understand this fully.

#### Next steps?
Minimizing bundle size is important, especially for a single page app where the JavaScrip all needs to be loaded before the user can do anything.

I'm planning on doing the same thing that I did for `lodash` with `react-bootstrap`, `react-router`, and some of the other packages I'm using. I'm not going to do this in one big go but rather bit by bit, simply because I have other features that I want to be working on rather than solely an optimisation activity.

I also want to better understand Webpack and how I can use some of the things listed in [this blog post](http://moduscreate.com/optimizing-react-es6-webpack-production-build/) and others to further reduce the bundle size.

If you have any comments or ideas, I'd love to hear them.

Patrick.
