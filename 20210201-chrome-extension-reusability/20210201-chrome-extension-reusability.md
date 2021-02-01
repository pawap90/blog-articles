---
title: Chrome extensions: Reusing code
published: true
description: A strategy to share code across a Chrome extension's different components.
tags: chromeextension, webdev, javascript, bestpractices
cover_image: https://i.imgur.com/wXwuE9u.png
series: chrome-extensions
---

If you're new to this series and don't want to read the previous posts, here's a **quick recap**: 

- I started this series building a very simple chrome extension that I've been updating and improving in every post. 
- The chrome extension we're working with is called "Acho, where are we?" 
- Acho is the name of my dog 🐶, and in this extension, he will bark and **tell you the Title** of the page you're currently navigating: 
  - Through a **browser action** (a popup that appears at the right of the navigation bar in Chrome) 
  - Or through a keyboard shortcut that shows a **notification** at the bottom-right of the screen.

**Table of contents**
- [Introduction](#introduction)
- [Centralize the shared logic in a separate file](#centralize-the-shared-logic-in-a-separate-file)
- [Accessing the reusable code](#accessing-the-reusable-code)
  - [From the Browser Action](#from-the-browser-action)
  - [From the content script](#from-the-content-script)
  - [From the background script](#from-the-background-script)
- [Conclusion](#conclusion)
- [The repo](#the-repo)
- [Let me know what you think! 💬](#let-me-know-what-you-think-)

# Introduction

So far, our extension has the following **features**:
- Display a browser action (popup) with the title of the active tab
- A command that duplicates the current tab
- A command that shows a notification at the bottom-right of the screen with the active Tab title.

And these are the components we built to manage the logic of these features:

![Three components (popup.js, background.js, and content.js) with their primary functions. The popup.js has the following functions: Listen OnLoad, Get active tab, and Show tab title. The background.js has the following functions: Listen OnCommand, Duplicate tab, Send a message to the content script, Get active tab. The content.js has the following functions: Build notification and Show tab title.](https://i.imgur.com/yyeX546.png)

The functions "Get active tab" and "Show tab title" are used by multiple components, but right now, their logic is duplicated inside each of the components. As you may have imagined, we need to find a way to write that logic a single time and share it across our project.

> ℹ️ **About reusability**: Reusing code allows us to save time and reduce redundancy in our project. By avoiding writing the same code multiple times, we're also making our project easier to maintain since our code gets cleaner and updates to the shared logic can be done in a single place.

So, a better version of our app would look something like this:

![The functions Get active tab and Show tab title appear a single time inside a new file called acho.js and are shared with popup.js, background.js, and content.js](https://i.imgur.com/7GrZjlB.png)

In this version, our components are only responsible for their particular logic, and the shared logic is separated in the `acho.js` file, where it can be easily maintained and shared. There's also no duplicated logic.

Let's see how to achieve that in our sample chrome extension.

# Centralize the shared logic in a separate file

For starters, we need our reusable logic to be centralized in a separate file. So we are going to create a new file called `acho.js`. Here we will create a class named Acho and add the methods that will later be called from each component. 

> In a real example, you'd probably use more than one file for your shared logic. We are using just one to keep the example simple.

Here's how the `acho.js` file looks like:
```js
/** Shared logic */
class Acho {

    /**
     * Gets the active Tab
     * @returns {Promise<*>} Active tab
     */
    getActiveTab = async () => {
        const query = { active: true, currentWindow: true };
        const getTabTitlePromise = new Promise((resolve, reject) => {
            chrome.tabs.query(query, (tabs) => {
                resolve(tabs[0]);
            });
        });
        return getTabTitlePromise;
    }

    /**
     * Concatenates the tab title with Acho's barks.
     * @param {String} tabTitle Current tab title
     * @returns {String} 
     */
    getBarkedTitle = (tabTitle) => {
        const barkTitle = `${this.getRandomBark()} Ahem.. I mean, we are at: <br><b>${tabTitle}</b>`
        return barkTitle;
    }
    
    /**
     * Array of available bark sounds
     * @private
     * @returns {String[]}
     */
    getBarks = () => {
        return [
            'Barf barf!',
            'Birf birf!',
            'Woof woof!',
            'Arf arf!',
            'Yip yip!',
            'Biiiirf!'
        ];
    }

    /**
     * Returns a random bark from the list of possible barks.
     * @private
     * @returns {String}
     */
    getRandomBark = () => {
        const barks = this.getBarks();
        const bark = barks[Math.floor(Math.random() * barks.length)];
        return bark;
    }
}
```

We have two public methods:
- `getActiveTab` returns the active tab. 
- `getBarkedTitle` generates a string concatenated with a random bark sound and the tab title. We'll use this both in the browser action (the popup) and the notification.

Then we have a few private methods just to simplify the logic in our public methods.

# Accessing the reusable code 

Great. Now our reusable logic is ready to be used by many components, but that's not all. We need to figure out *how to access this logic* from each component:
- Background script (`background.js`)
- Content script (`content.js`)
- Browser action script (`popup.js`)

To approach this issue it's important to remember that, even though all of these components are part of the same extension, **they run in different contexts**:

- The `popup.js` runs in the context of our Browser Action
- The content script runs in the context of the web page.
- The background script handles events triggered by the browser and is only loaded when needed. It works independently from the current web page and the browser action.

So how can we make our reusable code available to all of these different contexts?

## From the Browser Action
This one will probably feel familiar to you since the solution we are going to implement it's what we do in static HTML + JS websites: We are going to add the file `acho.js` as a script in our browser action HTML file (`popup.html`) using the `<script>` tag:

Open the `popup.html` file and add the script at the bottom of the `<body>` tag, like so:

```html
<body>
    <!-- the rest of the body -->

    <script src='popup.js'></script> 
    <script src='acho.js'></script> <!-- 👈 -->
</body>
```

Done! Now we can use the `Acho` class from `popup.js`, and our code will be significantly reduced:

```js
document.addEventListener('DOMContentLoaded', async () => {
    
    const dialogBox = document.getElementById('dialog-box');
    const query = { active: true, currentWindow: true };
    
    const acho = new Acho(); // 👈
    const tab = await acho.getActiveTab();
    const bark = acho.getBarkedTitle(tab.title);

    dialogBox.innerHTML = bark;
});
```

## From the content script
The solution here may not be as obvious, but it's pretty simple: Just add `acho.js` to the `js` array inside our current content script object in the `manifest.json` file:

```json
{
    "manifest_version": 2,
    "name": "Acho, where are we?",
    ... 
    "content_scripts": [
        {
            "matches": ["<all_urls>"],
            "js": ["content.js", "acho.js"], // 👈
            "css": ["content.css"]
        }
    ],
}
```

And now we can instantiate and use the `Acho` class in `content.js` to generate the "barked title" string:

```js
// Notification body.
const notification = document.createElement("div");
notification.className = 'acho-notification';

// Notification icon.
const icon = document.createElement('img');
icon.src = chrome.runtime.getURL("images/icon32.png");
notification.appendChild(icon);

// Notification text.
const notificationText = document.createElement('p');
notification.appendChild(notificationText);

// Add to current page.
document.body.appendChild(notification);

chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {

    const notification = document.getElementsByClassName('acho-notification')[0];
    const notificationText = notification.getElementsByTagName('p')[0];

    // 👇👇👇
    const acho = new Acho();
    notificationText.innerHTML = acho.getBarkedTitle(request.tabTitle); 

    notification.style.display = 'flex';

    setTimeout(function () {
        notification.style.display = 'none';
    }, 5000);
    
    return true;
});
```

## From the background script
Here the solution is similar: We need to add `acho.js` to the `scripts` array of our `background` object in the `manifest.json`:

```json
{
    "manifest_version": 2,
    "name": "Acho, where are we?",
    ... 
    "background": {
        "scripts": [ "background.js", "acho.js" ], // 👈
        "persistent": false
    }
}
```

And just like that, we can now access the `Acho` class from `background.js`:

```js
chrome.commands.onCommand.addListener(async (command) => {
    switch (command) {
        case 'duplicate-tab':
            await duplicateTab();
            break;
        case 'bark':
            await barkTitle();
            break;
        default:
            console.log(`Command ${command} not found`);
    }
});

/**
 * Gets the current active tab URL and opens a new tab with the same URL.
 */
const duplicateTab = async () => {
    const acho = new Acho(); // 👈 
    const tab = await acho.getActiveTab();

    chrome.tabs.create({ url: tab.url, active: false });
}

/**
 * Sends message to the content script with the currently active tab title.
 */
const barkTitle = async () => {
    const acho = new Acho(); // 👈 
    const tab = await acho.getActiveTab();

    chrome.tabs.sendMessage(tab.id, {
        tabTitle: tab.title
    });
}
```

> I had to make the functions `async` so I could await the promise from `acho.getActiveTab()`. You can use `acho.getActiveTab().then((tab) => { })` instead if you like.

That's it! Now all our components are reusing the logic from `acho.js`.

# Conclusion

We managed to remove our duplicated code and apply reusability by creating a separate file containing the shared logic and using different strategies to make that file available in every component. 

Now our extension's code is easier to read and maintain 👌

# The repo

You can find all my Chrome Extensions examples in this repo:

{% github pawap90/acho-where-are-we no-readme %}

# Let me know what you think! 💬
Are you working on or have you ever built a Chrome extension? 

Do you know any other strategies for code reusability in Chrome extensions?