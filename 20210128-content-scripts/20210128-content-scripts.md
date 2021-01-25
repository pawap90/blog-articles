---
title: Chrome Extensions: Making changes to a web page
published: false
description: Using content scripts to make changes to a web page.
tags: chromeextension, chrome, webdev, javascript
//cover_image: https://banner-url.png
series: chrome-extensions
---

In this post, I will focus on **content scripts** and how to use them to make changes to a web page. 

The **main concepts** we will explore are:
- Using content scripts to make changes to a web page.
  - Attaching HTML
  - Adding new styles
- Sending messages between the background script and the content script
- Accessing our extension's resources from the content script

--- 
**Table of contents**
- [The example](#the-example)
- [About content scripts](#about-content-scripts)
- [Let's get coding!](#lets-get-coding)
  - [1. Create a new command](#1-create-a-new-command)
  - [2. Register the content script](#2-register-the-content-script)
  - [3. Display the notification](#3-display-the-notification)
  - [Done!](#done)
- [The repo](#the-repo)

# The example
For this post's example, I will keep adding features to our initial sample extension: We will use **content scripts** to display a notification at the bottom right of the currently active page. 
We will also rely on what we learned previously in this series: A **command** will trigger the notification to be handled by our **background script**. Finally, the background script will *message* the **content script**, to activate the notification showing the title of the page at the bottom-right of the screen:

![A small portion of a DEV.to's web page appears, and the notification is displayed with the text "DEV Community"](https://i.imgur.com/rD5vZMv.gif)

# About content scripts
- Content scripts are files that **run in the same context as the web page** the user visited. 
- They share access with the page's DOM.
- Within these scripts, we can use **JavaScript** to access the web page elements, read its contents and make changes. And we can use **CSS** to add new styles to the web page.
- They allow you to extract information from the page and send it to other scripts or receive messages from our extension.
- Finally, content scripts have access to some of the chrome APIs, which allows us to do stuff like get the current URL, access the extension's storage, etc. 

# Let's get coding!

## 1. Create a new command
In the previous post of this series, we added two commands to our example extension. Now we are going to add a third one.
To do that, first, we will define the command and it's suggested shortcut in the `manifest.json` file:

```json
{
    "manifest_version": 2,
    "name": "Acho, where are we?",
    // ...
    "commands": {
        "bark": {
            "suggested_key": {
                "default": "Alt+Shift+3"
            },
            "description": "Makes Acho bark the title at the bottom right of the current page."
        },
        // .... Other commands
    }
}
```

Now, we need to handle our command by listening to the `onCommand` event. This should be done in the background script:

```js
// background.js

chrome.commands.onCommand.addListener(function (command) {
    switch (command) {
        case 'bark':
            barkTitle();
            break;
        default:
            console.log(`Command ${command} not found`);
    }
});

function barkTitle() {
    const query = { active: true, currentWindow: true };
    chrome.tabs.query(query, (tabs) => {
        chrome.tabs.sendMessage(tabs[0].id, {
            tabTitle: tabs[0].title
        });
    });
}
```

So, once the `bark` command is executed, we will send a *message* indicating the currently active tab's title. 
Now our content script needs to listen to that message and display the notification.

> **About Messages**: Content scripts don't run in the context of the extension but in the context of the web page. They need a way to communicating with the extension. We can do that using *messages*. 
> - To send a message **to** a content script, use `chrome.tabs.sendMessage`  and specify the `TabId`. The message will be sent to the content script running in that tab.
> - To send a message **from** a content script, use `chrome.runtime.sendMessage`. 

## 2. Register the content script 
To create a content script, the first thing we need to do is (yes, you guessed it!) add it to the `manifest.json` file:

```json
{
    "manifest_version": 2,
    "name": "Acho, where are we?",
    // ... 
    "content_scripts": [
        {
            "matches": ["<all_urls>"],
            "js": ["content.js"],
            "css": ["content.css"]
        }
    ],
    "web_accessible_resources": [
        "images/icon32.png"
    ],
}
```
- `content_scripts`: An array of content scripts. We can register multiple scripts, each with different configurations.
- `matches`: An array of string expressions that specify which pages will this particular content script be injected into. You can use `"matches": ["<all_urls>"]` to inject it in any URL.
- `js`: An array of javascript files. These files will handle the logic for our content scripts.
- `css`: An array of CSS files. In this case, we will use a CSS file to define our notification styles.
- `web_accessible_resources`: A list of resources we will need to access from our content scripts. Since the content script runs in a different context than the extension, any extension resource we want to access must be explicitly made available here.

## 3. Display the notification
Let's start by adding the logic to `content.js`:

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

    notificationText.innerHTML = request.tabTitle;

    notification.style.display = 'flex';

    setTimeout(function () {
        notification.style.display = 'none';
    }, 5000);
    
    return true;
});
```

Now let's inspect the previous code more carefully:
- First, we create a `div`, which will be our notification body. We also assign it a `class` that will help us define styles for it later.
- Then, we attach an `img` to the previous `div`. This will add the icon of our extension to the notification box. 
  - To get the icon of our extension, we must use `chrome.runtime.getURL`. Remember, the content script doesn't run in the same context as the extension, so we can't just access our extension's resources directly. That's also why we added the `icon32.png` file to the `manifest.json` as a *web-accessible resource*.
- Next, we add a `p` element where we will later attach the notification text.
- Finally, we append the notification to the web page's body.

These first 15 lines will ensure that every page loaded has our notification structure. To finally display the notification, we added a listener for the `chrome.runtime.onMessage`. Let's inspect that code:

- Once we receive the message, the first thing to do is find the notification's structure within the current web page. We use `document.getElementsByClassName` to get the notification's body, and from there we get the `p` element inside it using `getElementsByTagName`.
- Remember that the message sent by our `background.js` script includes the `tabTitle`, so we use that value from `request.tabTitle` and set it as the content of the notification's text element.
- We make sure our notification is visible by setting the `display` property to `flex`.
- Finally, we use `setTimeout` to hide the notification again after 5 seconds.

Great! We are almost done. Let's add some styles to the notification inside the `content.css` file:

```css
.acho-notification {
    background-color: white;
    border: rgb(242, 105, 77) 1px solid;
    border-radius: 5px;
    font-size: medium;
    width: 320px;
    display: none;
    padding: 8px;
    position: fixed;
    bottom: 30px;
    right: 30px;
    align-items: center;
}

.acho-notification > img {
    margin-right: 12px;
}
```

## Done!
And that's it! This is how our notification will look when the user presses `Alt+Shift+3`:

![After a second, the notification appears at the bottom-right displaying the title of the page](https://i.imgur.com/ep4CUYC.gif)

# The repo

I'm keeping this repo updated with all my Chrome Extensions examples:

{% github pawap90/acho-where-are-we no-readme %}
