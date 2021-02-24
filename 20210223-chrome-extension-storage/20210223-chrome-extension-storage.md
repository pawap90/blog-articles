---
title: Chrome extensions: Local storage
published: false
description: Let's explore how to store data locally in our Chrome extension
tags: chromeextension, webdev, javascript, chrome
cover_image: https://i.imgur.com/C7nejI9.png
series: chrome-extensions
---

I'm back with another post about **Chrome extensions**! This time I wanted to explore how to *store data locally* using the `chrome.storage` API.

In this post, we're going to add yet another feature to our original extension (Acho, where are we?). This new feature will **store the Title and URL of the page** each time we call Acho to tell us where we are. We will then **list** all of the pages and allow the user to **navigate** to one of them or **clear** the list.

Here's a quick demo:

![When the user opens the popup, a list of the previously visited pages appears. The user can navigate to each page clicking over it, or they can clear the list using a button](https://i.imgur.com/ZJJgxaC.gif)

So let's get started!

# 1. Add the storage permission to the manifest.json
As usual, the first thing we need to update is our `manifest.json`. This time we're going to add the `storage` permission:

```json
{
    "manifest_version": 2,
    "name": "Acho, where are we?",
    ... 
    "permissions": [
        "tabs",
        "storage"   // 👈
    ]
}
```

This will allow our extension to use the `storage` API.

# 2. Create the Page Service
Since we already know how to reuse code in chrome extensions, we will create the data access logic in a separate class called `PageService`. Here we will add the following methods:
- `getPages`: Will return the list of stored pages.
- `savePage`: Will receive the page data and store it.
- `clearPages`: Will remove all the pages from the storage.

## About the storage API
The `chrome.storage` API allows us to store objects using a *key* that we will later use to retrieve said objects. This API is similar but a bit more robust than the `localStorage`, but it's not as powerful as an actual database, so we will need to manage some things ourselves.

To save an object we will define a *key-value pair* and use the `set` method. Here's an example:

```js
const key = 'myKey';
const value = { name: 'my value' };

chrome.storage.local.set({key: value}, () => {
  console.log('Stored name: ' + value.name);
});
```

And to retrieve our value we will use the `get` method and the *key*:

```js
const key = 'myKey';
chrome.storage.local.get([key], (result) => {
  console.log('Retrieved name: ' + result.myKey.name);
});
```

Finally, to clear the storage we have two options:

```js
// Completely clear the storage. All items are removed.
chrome.storage.local.clear(() => {
    console.log('Everything was removed');
});

// Remove items under a certain key
const key = 'myKey';
chrome.storage.local.remove([key], (result) => {
  console.log('Removed items for the key: ' + key);
});
```

Another thing to have in mind when working with this API is **error handling**. When an error occurs using the `get` or `set` methods, the property `chrome.runtime.lastError` will be set. So we need to check for that value after calling the get/set methods. A few examples:

```js
const key = 'myKey';
const value = { name: 'my value' };

chrome.storage.local.set({key: value}, () => {
    if (chrome.runtime.lastError)
        console.log('Error setting');

    console.log('Stored name: ' + value.name);
});

chrome.storage.local.get([key], (result) => {
    if (chrome.runtime.lastError)
        console.log('Error getting');

    console.log('Retrieved name: ' + result.myKey.name);
});
```

> Don't worry, I promise the actual implementation will be better than a `console.log`.

And, before we move on to the real implementation, I wanted to show you something else. I like to work with `async/await` instead of `callbacks`. So I created a simple function to promisify the callbacks and still handle errors properly. Here it is:

```js
const toPromise = (callback) => {
    const promise = new Promise((resolve, reject) => {
        try {
            callback(resolve, reject);
        }
        catch (err) {
            reject(err);
        }
    });
    return promise;
}

// Usage example: 
const saveData = () => {
    const key = 'myKey';
    const value = { name: 'my value' };

    const promise = toPromise((resolve, reject) => {
        chrome.storage.local.set({ [key]: value }, () => {
            if (chrome.runtime.lastError)
                reject(chrome.runtime.lastError);

            resolve(value);
        });
    });
}

// Now we can await it:
await saveData();
```

> You can replace `chrome.storage.local` with `chrome.storage.sync` to sync the data automatically to any Chrome browser where the user is logged into (if they have the sync feature enabled). But keep in mind that there are [limits, as specified in the official documentation](https://developer.chrome.com/docs/extensions/reference/storage/#property-sync).
> 
> `chrome.storage.local` also has limits in the amount of data that can be stored, but that limit can be ignored if we include the `unlimitedStorage` permission in the `manifest.json` (check [the docs](https://developer.chrome.com/docs/extensions/reference/storage/#property-local)).

Let's move on to our actual implementation!

## PageService class
As I said before, our PageService will have 3 methods to store, retrieve and remove our `pages`. So here they are:

```js
const PAGES_KEY = 'pages';

class PageService {

    static getPages = () => {
        return toPromise((resolve, reject) => {
            chrome.storage.local.get([PAGES_KEY], (result) => {
                if (chrome.runtime.lastError)
                    reject(chrome.runtime.lastError);

                const researches = result.pages ?? [];
                resolve(researches);
            });
        });
    }

    static savePage = async (title, url) => {
        const pages = await this.getPages();
        const updatedPages = [...pages, { title, url }];

        return toPromise((resolve, reject) => {
            
            chrome.storage.local.set({ [PAGES_KEY]: updatedPages }, () => {           
                if (chrome.runtime.lastError)
                    reject(chrome.runtime.lastError);
                resolve(updatedPages);
            });
        });
    }

    static clearPages = () => {
        return toPromise((resolve, reject) => {
            chrome.storage.local.remove([PAGES_KEY], () => {
                if (chrome.runtime.lastError)
                    reject(chrome.runtime.lastError);
                resolve();
            });
        });
    }
}
```

A few things to note about this class:
- We are using the `toPromise` function we talked about earlier.
- We are storing an *array of `pages`*, so every time we add a new page to the storage, we need to **retrieve the entire array**, **add our new element** at the end **and replace the original array** in storage. This is one of a few options I came up with to work with arrays and the `chrome.storage` API since it doesn't allow me to directly push a new element to the array.

# 3. Make our PageService available to our components
As we saw in the previous posts of this series, we need to make some changes to allow our new class to be used by our extension's different components.

First, we will add it as a script to our `popup.html` so we can later use it in `popup.js`:

```html
<!-- popup.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    ...
</head>
<body>
    ...
    <script src='popup.js'></script>
    <script src='acho.js'></script>
    <script src='page.service.js'></script> <!-- 👈 -->
</body>
</html>
```

This will allow us to save pages, retrieve them and clear them from the *browser action*.

And finally, we'll add it as a `background script` in our `manifest.json` so we can also call the `savePage` method from our background script when the user uses the shortcut:

```json
{
    "manifest_version": 2,
    "name": "Acho, where are we?",
    ...
    "background": {
        "scripts": [ 
            "background.js", 
            "acho.js", 
            "page.service.js" // 👈
        ],
        "persistent": false
    },
    ...
}
```

# 4. Update our popup.js
Now let's update our popup.js to add the new features.

```js
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

    // Clear history.
    const clearHistoryBtn = document.getElementById('clear-history');
    clearHistoryBtn.onclick = async () => {
        await PageService.clearPages();
        await displayPages();
    };
});

const displayPages = async () => {
    const visitedPages = await PageService.getPages();
    const pageList = document.getElementById('page-list');
    pageList.innerHTML = '';
    
    visitedPages.forEach(page => {
        const pageItem = document.createElement('li');
        pageList.appendChild(pageItem);
        
        const pageLink = document.createElement('a');
        pageLink.title = page.title;
        pageLink.innerHTML = page.title;
        pageLink.href = page.url;
        pageLink.onclick = (ev) => {
            ev.preventDefault();
            chrome.tabs.create({ url: ev.srcElement.href, active: false });
        };
        pageItem.appendChild(pageLink);
    });
}
```

So in the previous code, we are using our three methods from `PageService` to add the current page to the storage, list the pages on the screen and allow the user to navigate them, and clear the list. 

We use the `displayPages` method to display the pages, where we retrieve the list of pages and generate a `<li>` element and an `<a>` element for each page. It's important to notice that we need to override the `onclick` event on our `<a>` element because if we leave the default functionality, the extension will try to load the page *inside our popup*, which it's not what we want and it will cause an error. Instead, we create a new tab and navigate to the link using `chrome.tabs.create`.

That's all we need to do to add the new feature to our popup.

# 5. Saving tha page from the background script
 ow let's make sure the pages are also stored when we use the command shortcut. To achieve that all we need to do is call the `savePage` method when the user executes the command:

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

const barkTitle = async () => {
    const acho = new Acho();
    const tab = await acho.getActiveTab();

    chrome.tabs.sendMessage(tab.id, {
        tabTitle: tab.title
    });

    await PageService.savePage(tab.title, tab.url); // 👈
}
```

That's it!

# The repo
You can find this and all of the previous examples of this series in my repo:

{% github pawap90/acho-where-are-we no-readme %}

# Let me know what you think! 💬
Are you working on or have you ever built a Chrome extension? 
How do you manage data storage?