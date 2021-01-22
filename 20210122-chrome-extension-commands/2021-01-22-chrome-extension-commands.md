---
title: Adding shortcuts to your Chrome Extension
published: false
description: Learn how to add keyboard shortcuts to your chrome extension using Commands.
tags: webdev, javascript, chromeextension, chrome
cover_image: https://i.imgur.com/jCgXmcU.png
series: chrome-extensions
---

Last week I wrote an article explaining [how to create a simple chrome extension](https://dev.to/paulasantamaria/creating-a-simple-chrome-extension-36m). Today I'm going to add a new feature to the same extension we built last week: **Keyboard shortcuts**.

We will add two shortcuts: One to open the browser action we created in the previous article and one to duplicate the current tab.  The shortcuts will be `Alt + Shift + 1` and `Ctrl/Command + Shift + 2`.
Using these 2 examples, we will learn how to add different types of shortcuts to our extension. 

![A diagram displaying the two shortcuts we will add to our extension](https://i.imgur.com/ABHf9Dt.jpg)

# Table of contents
- [Table of contents](#table-of-contents)
- [Commands API](#commands-api)
- [Let's get coding](#lets-get-coding)
  - [1. Define the commands in the manifest.json file](#1-define-the-commands-in-the-manifestjson-file)
  - [2. Add a background script](#2-add-a-background-script)
  - [3. Listen for the command event](#3-listen-for-the-command-event)
  - [Done!](#done)
- [The repo](#the-repo)
- [Was this useful?  💬](#was-this-useful--)

# Commands API

To create a keyboard shortcut for our extension, we must use the **commands API.** Through this API, we can define *commands* for our extension and bind them to *keyboard shortcuts*. When someone uses the shortcut, the command will be triggered, and the logic we added to the command will be executed.

Commands must be added to the `manifest.json` file along with their suggested keyboard shortcut.  Multiple commands can be defined in the `manifest.json`. However, only 4 shortcuts can be *suggested* by our extension. The user can bind the other commands to a keyboard shortcut from the browser (`chrome://extensions/shortcuts`).

> **Available keys**
> Any keyboard shortcut must use either `Ctrl` (`Command` in Mac) or `Alt` but cannot include both. `Shift` can also be used. 

>Other supported keys: `A-Z`, `0-9`, `Comma`, `Period`, `Home`, `End`, `PageUp`, `PageDown`, `Space`, `Insert`, `Delete`, Arrow keys (`Up`, `Down`, `Left`, `Right`) and the Media Keys (`MediaNextTrack`, `MediaPlayPause`, `MediaPrevTrack`, `MediaStop`).

> **Examples:** `Ctrl + Shift + L`, `Alt + Shift + L`  `Command + ,`  `Ctrl + Shift + 1`

Keep in mind that you cannot use commands *reserved* by the browser, like `Ctrl + T` (which in Chrome opens a new tab).

The logic that should be executed once the user runs a command should be handled in a *background script*. I will explain more about this later.

# Let's get coding

## 1. Define the commands in the manifest.json file

To define a command, we should use the `commands` property in our `manifest.json` file, like so:

```json
{
    "manifest_version": 2,
    "name": "Acho, where are we?",
    ...
    "commands": {
        "_execute_browser_action": {
            "suggested_key": {
                "default": "Alt+Shift+1"
            }
        },
        "duplicate-tab": {
            "suggested_key": {
                "default": "Ctrl+Shift+2",
                "mac": "Command+Shift+2"
            },
            "description": "Duplicates the currently active tab because... why not?"
        }
    }
}
```

In the previous code, we defined 2 different commands:

- `_execute_browser_action`: This is a **reserved** command that will be handled by chrome directly. By defining it in our `manifest.json`, we are telling chrome to open our browser action when the user executes the shortcut. We do not need to handle the logic for this command ourselves.
- `duplicate-tab`: This is a custom command that our extension must handle manually. Notice that this command also includes a `description`, which is not required for reserved commands (chrome just displays a default description that cannot be overridden).

## 2. Add a background script

To handle the logic for our `duplicate-tab` command, we will need a *background script.* Using this script, we will listen for the `onCommand` method and execute the appropriate logic.

>**About background scripts**

>Background scripts are used mainly to listen to events from the browser and trigger a reaction.  These scripts will stay running while they perform their tasks after some event was fired, and then they will be unloaded.

To include our background script, we must modify our `manifest.json` file and define the script using the `background` property, like so:

```json
{
    "manifest_version": 2,
    "name": "Acho, where are we?",
    ...
    "background": {
        "scripts": [
            "background.js"
        ],
        "persistent": false
    }
}
```

Finally, let's add a new file called `background.js` in our project's root.

## 3. Listen for the command event

For our command to be properly handled, we need to listen to the `onCommand` event in our background script and execute the appropriate logic once our command is called.

So we will listen to the event and call the `duplicateTab` function when the `duplicate-tab` command is called:

```js
chrome.commands.onCommand.addListener(function (command) {
    switch (command) {
        case 'duplicate-tab':
            duplicateTab();
            break;
        default:
            console.log(`Command ${command} not found`);
    }
});

/**
* Gets the current active tab URL and opens a new tab with the same URL.
*/
function duplicateTab() {
    const query = { active: true, currentWindow: true };
    chrome.tabs.query(query, (tabs) => {
        chrome.tabs.create({ url: tabs[0].url, active: false });
    });
}
```

> Notice that I did not include a `case` for the `_execute_browser_action` command because, as we said before, that command is handled automatically by chrome.

## Done!
Now when the user executes `Alt + Shift + 1`, the browser action will be open, and when they use the shortcut `Ctrl/Command + Shift + 2`, the current tab will be duplicated.

# The repo

I'm keeping this repo updated with all my Chrome Extensions examples:

{% github pawap90/acho-where-are-we no-readme %}

---

# Was this useful?  💬

Let me know what you think about this article in the comments!