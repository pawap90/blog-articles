---
title: Content scripts
published: false
description: How to display a notification from a Chrome Extension attaching HTML to the web page with Content Scripts.
tags: chromeextension, chrome, webdev, javascript
//cover_image: https://banner-url.png
series: chrome-extensions
---

In this post, I will focus on **Content Scripts** and how to use them to attach HTML to a web page using Chrome Extensions.

- [About Content Scripts](#about-content-scripts)
- [Our example](#our-example)
- [Let's get coding!](#lets-get-coding)
  - [1. Create a new Command](#1-create-a-new-command)
  - [2. Register the Content Script](#2-register-the-content-script)
  - [3. Display the notification](#3-display-the-notification)
  - [Done!](#done)
- [The repo](#the-repo)

# About Content Scripts
- Content scripts are files that **run in the same context as the web page** the user visited. Within these files, we can use **JavaScript** to access the web page elements, read its contents and make changes. We can also use **CSS** to add new styles to the web page.
- We can also use these scripts to extract information from the page and send it to other scripts. Or receive messages from our extension.
- Finally, content scripts have access to some of the chrome APIs, which allows us to do stuff like get the current URL, access the extension's storage, etc. 

# Our example
For this post's example, I will keep adding features to our base example extension: We will use **Content scripts** to display a notification at the bottom right of the page the user is visiting. 
We will also rely on some of the things we learned previously in this series: A ** command** will trigger the notification that will be handled by our **Background Script**. Finally, the Background script will *message* the Content script, which will display the title of the page at the bottom-right:

![A small portion of a DEV.to's web page appears, and the notification is displayed with the text "DEV Community"](https://i.imgur.com/rD5vZMv.gif)

# Let's get coding!

## 1. Create a new Command
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

Now, we need to handle our command listening to the `onCommand` event. This should be done in the Background Script:

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

> **About Messages**: Content scripts don't run in the context of the extension but in the context of the web page. They need a way to communicating with the extension. That can be done via messages. 
> - To send a message **to** a content script, use `chrome.tabs.sendMessage`  and specify the TabId. The message will be sent to the content script running in that tab.
> - To send a message **from** a content script, use `chrome.runtime.sendMessage`. To receive the message.

So, once the `bark` command is executed, we will send a *message* indicating the title of the currently active tab's title. 
Now our Content Script needs to listen to that message and display the notification.

## 2. Register the Content Script 
To create a Content Script, the first thing we need to do is (yes, you guessed it!) add it to the `manifest.json` file:

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
- `js`: An array of javascript files. Where our Content script's logic will live.
- `css`: An array of CSS files. We can inject our own styles into the web page. We will need it to add the styles for our notification.
- `web_accessible_resources`: A list of resources we will need to use from our content scripts. Since the Content script runs in a different context than the extension, any extension resource we want to access from the Content Script must be explicitly made available here.

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

These first 15 lines of code will ensure that every page we load has the structure for our notification. To finally display the notification, we added a listener for the `chrome.runtime.onMessage`. Let's inspect that code:

- Once we receive the message, the first thing to do is find the notification's structure within the current web page. We use `document.getElementsByClassName` to get the notification's body, and from there we get the `p` element inside it using `getElementsByTagName`.
- Remember that the message that our `background.js` script sent includes the tabTitle, so we use that value from `request.tabTitle` and set it as the content of the notification's text element.
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
