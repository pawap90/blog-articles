---
title: Create games using TypeScript, Snowpack and ESLint with this template
published: true
description: I created an open-source template to develop games using Phaser 3, TypeScript, Snowpack and ESLint.
tags: showdev, gamedev, typescript
cover_image: https://i.imgur.com/nPuFFZj.png
---

If you follow me on [Twitter](https://twitter.com/pauxdsantamaria) or [Instagram](https://www.instagram.com/pau.codes/), you may know I've been trying out [Phaser](https://phaser.io/) lately. 

**Phaser is an open-source game framework** that allows you to develop games using HTML and Javascript. 

I've always been interested in Game development, but I hadn't found the time in the past to really get into it. I tried some things with Unity over the years, but I was focused on University and work, so those projects always got left behind at some point. Now that I graduated, I have more time to explore these kinds of things.

So I started going through some tutorials and doing some experiments with the framework. To get started, all you need is a local web server and getting Phaser through NPM or a CDN, for example. Browsing online, I found some templates that used No.jsde as a web server and installed Phaser through NPM, so I decided to make my own template using a modern tech stack.

And here it is:

{% github pawap90/phaser3-ts-snowpack-eslint %}


# About the template

This template allows you to develop your game using **TypeScript**, keep your codebase clean with **ESLint**, and enjoy lightning-fast live updates thanks to **Snowpack**.

I decided to use TypeScirpt instead of JavaScript after doing some tutorials. I tried both options, and I just felt more comfortable with TypeScript.

## Build it with Snowpack
When the time came to choose a bundler, I decided to give Snowpack a try since I've heard many good things about it. And honestly, I was blown away by how **fast** it is. The development server starts in 14ms on my machine, and the live updates are fantastic. I can play around with Phaser and see every update on the browser almost instantly.

I also used **Snowpack's built-in optimizations** to minify and bundle the build and make it lighter for production. This Snowpack feature is not yet production-ready (according to the docs), but I've done a few tests, published some sample games, and I haven't had any issues with it so far.

## Minimalistic
The template is very minimalistic, meaning that it only has what's needed to run and build the project, plus a Sample scene to check everything is working correctly. 

## Project structure

```
├───public/                         Public static files
│   ├───assets/                     Sample assets
│   │   ├───banner.png
│   │   ├───acho.png
│   │   └───ground.png
│   └───index.html                  HTML file where our game will be loaded
├───src/                            Game logic goes here
│   ├───scenes/                     Game scenes
│   │   ├───InitialScene.ts         Initial sample scene
│   │   └───PreloaderScene.ts       Scene preloader
│   └───Main.ts                     Phaser game configuration
├───.eslintignore                   Files that should be ignored by ESLint	
├───.eslintrc.js                    ESLint configuration file
├───.gitignore                      Files that should not be pushed to the repo
├───package.json                    Project scripts, dependencies and metadata
├───snowpack.config.js              Snowpack configuration file
└───tsconfig.json                   Typescript configuration file
```

> You can remove everything in the `public/assets` folder. But I recommend you first run the project once and make sure everything is installed and running properly.

## Clean and safe
It's focused on keeping the codebase clean and safe, thanks to TypeScript and ESLint.

TypeScript's configuration contains some flags that set some checks like "strict" and "noImplicitAny" to reduce errors. You can always change that in the `tsconfig.json` file if you prefer different settings.

I also added a `prebuild` script that uses `tsc` and `ESLint` to compile and lint the project before building it to ensure the build is safe to publish.

So when you run `npm run build`, the `prebuild` will be executed first and fail if there are errors. This script could be easily executed in a CI pipeline, for example, to make sure you don't merge or deploy the project if there are issues.

### ESLint

I added ESLint to keep the codebase clean and consistent. The configuration can be found in `.eslint.js` in case you want to add your own rules or modify something. 

There are also a few scripts I added to the `package.json` to check for errors or styling issues:

```sh
# Print the list of problems found
npm run lint

# Some of the issues can be automatically fixed using:
npm run lint:fix
```

# Beautiful and classy 🧐🎩

Once you clone the template and install the dependencies, you'll be blessed with this beauty:

![Acho the pup bouncing around](https://i.imgur.com/bYVcrSr.gif)

--- 

If you want to know more about my Game Dev journey, follow me on [Instagram](https://www.instagram.com/pau.codes/) or [Twitter](https://twitter.com/pauxdsantamaria). I'll be sharing my progress, tips, and tricks, and probably lots of bloopers and fails.