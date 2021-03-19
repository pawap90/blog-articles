---
title: Chrome Extensions: Adding a badge
published: true
description: How to add a badge to your extension's icon
tags: chromeextension, javascript, chrome, webdev
cover_image: https://i.imgur.com/zyfqHe2.png
series: chrome-extensions
---

I thought I should add some new fun features to our sample extension to explore a few more things that can be done with Chrome extensions. I was curious about **badges** because they seem like an interesting tool to *communicate changes* in the state of our extension to our users.

# About badges
Badges appear over the Browser Action Icon and **include a short text**. 

The text can include a number of characters, but the badge *will only show the amount that fits* in that tiny space (the docs say around 4, but I could fit a maximum of 6). The **characters** that **don't fit won't be visible**, so try to keep your badge text short.

To create a badge all we need to do is **set the text**, like so:

```js
chrome.browserAction.setBadgeText({ tabId: myTabId, text: 'grr' });
// or to add it to all tabs:
chrome.browserAction.setBadgeText({ text: 'grr' });
```

- The `tabId` is optional, but when included, the text specified for the badge will only be visible when that particular tab is active.
- The `text` is optional too, but the badge won't be visible if we don't include it.

So, to hide the badge all we need to do is set the text for that particular tab to `null`:

```js
chrome.browserAction.setBadgeText({ tabId: myTabId, text: null });
// or, the shorter version:
chrome.browserAction.setBadgeText({ tabId: myTabId });
// or to remove it from all tabs:
chrome.browserAction.setBadgeText({ });
```

For extra customization, we can also **change the background color** of the badge (the default is blue):

```js
chrome.browserAction.setBadgeBackgroundColor({ color: '#F00' }, () => {
    // callback
});
```

# Updating our sample extension
You see, 🐶 Acho gets impatient whenever a new page or tab is loaded and we don't ask him about it right away (I mean, it's his job!). So we'll give him a tool to express himself:
- When a **new tab** is created, or the **active tab gets updated**, Acho will let us know he's ready to work by *creating a badge*.
- After his job is done, *the badge will disappear*.

Here's how it will look:

![A badge with the text "grr" shows up over the icon 🐶 every time a new page is loaded](https://i.imgur.com/ntmu5wL.gif)

So first, we'll update the Acho class `acho.js` to give him the ability to growl and be quiet:

```js
// acho.js
class Acho {

    growl = () => {
        chrome.browserAction.setBadgeBackgroundColor({ color: '#F00' }, () => {
            chrome.browserAction.setBadgeText({ text: 'grr' });
        });
    }

    quiet = () => {
        chrome.browserAction.setBadgeText({});
    }
}
```

Then we'll listen for the `tabs.onCreated` and `tabs.onUpdated` events in our `background.js`, and when they're fired we'll let Acho growl using the `growl` method we just added:

```js
// background.js
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
    new Acho().growl();
});

chrome.tabs.onCreated.addListener(tab => {
    new Acho().growl();
});
```

And finally, we'll ask Acho to be quiet when he fulfills his job. This must be done both in the `background.js` file and the `popup.js` file since Acho can do his job through the browser action (popup) or a command handled in the background script.

In the background script, we must add a new line at the end of our `barkTitle` method. So, once the notification is sent, we can remove the badge:
```js
// background.js
const barkTitle = async () => {
    const acho = new Acho();
    const tab = await acho.getActiveTab();

    chrome.tabs.sendMessage(tab.id, {
        tabTitle: tab.title
    });

    await PageService.savePage(tab.title, tab.url);

    acho.quiet();   // 👈
}
```

In the `popup.js`, we will remove the notification after loading all the info in the popup:

```js
// popup.js
document.addEventListener('DOMContentLoaded', async () => {
    
    const dialogBox = document.getElementById('dialog-box');
    
    const acho = new Acho();
    const tab = await acho.getActiveTab();
    const bark = acho.getBarkedTitle(tab.title);

    dialogBox.innerHTML = bark;
    
    // Store page.
    await PageService.savePage(tab.title, tab.url);

    // Display history.
    await displayPages();

    // Clear history
    const clearHistoryBtn = document.getElementById('clear-history');
    clearHistoryBtn.onclick = async () => {
        await PageService.clearPages();
        await displayPages();
    };

    acho.quiet();   // 👈
});
```
# Done!
That's it! We learned **how to add a badge, hide it and change its color**, and now Acho can express his frustration when we don't let him fulfill his purpose 😂.

# The repo
You can find this and all of the examples of this series in my repo:

{% github pawap90/acho-where-are-we no-readme %}