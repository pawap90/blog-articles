---
title: Chrome Extensions: Migrating to Manifest v3
published: false
description: In this post, we'll go through a brief overview of Manifest v3, and learn everything we need to know to migrate our sample extension.
tags: chromeextension, javascript, chrome, manifestv3
cover_image: https://i.imgur.com/MqB2Ckh.png
series: chrome-extensions
---

**Manifest v3** has been available since the release of **Chrome 88** earlier this year. If you're planning on building a Chrome extension or if you're currently building one, you should learn about this new version of the Chrome Extensions Manifest in order to **benefit from the new features and vision.**

In this post, we'll go through a **brief overview of Manifest v3**, then we'll take a look at the **Migration Checklist** to learn everything we'll need to change to migrate our sample extension. Finally, we'll **apply the changes step by step** so at the end, our sample extension will be successfully migrated to Manifest v3!

# 1. Manifest v3 Overview
Chrome Extensions launched a decade ago, and, according to the docs, Manifest V3 represents one of the biggest shifts in the extensions platform since then. It includes many changes that bring Chrome Extensions closer to the modern web (like promises and service workers!).

## 1.1. Three pillars
As stated in the docs, Manifest v3 is a step forward in Chrome Extensions' strategic direction. The main focus of this vision is in the following 3 pillars:

- **Privacy**: The idea here seems to be to let the user know about the extension's activities and how their information is used. And also reduce the need for extensions to have access to user data persistently.
- **Security**: Extensions will be required to follow stricter protocols, and, for example, they won't be allowed to access scripts from outside the extension context.
- **Performance**: Keep good performance in all devices and avoid performance issues when extensions are installed.

They also state that they will preserve the **"webbiness"** of Chrome extensions to keep the barriers for developers low and benefit from the advances of the web.

Finally, they say the idea is to keep the platform **capable**, powerful, and feature-rich so developers can keep delivering value to users through it.

## 1.2. Main changes

### Background pages/scripts are replaced by **Service workers**.
Much like background pages, service workers are scripts that run in the background and are independent of web pages. They don't need interaction with the website or a user.

> ℹ️ Fun fact! Service workers were inspired by the background pages in Chrome Extensions.

### The new `declarativeNetRequest` API handles **Network request modification**.

This new API is focused on privacy. The request will still be able to be modified and blocked, but in a privacy-preserving way. 

This API is an improvement from the old `webRequest` API that fixes privacy, performance, and compatibility issues.

### Remotely-hosted code is no longer allowed
This change came to improve security. Since all the code will be available in the extension package, extensions will be more reliably and efficiently reviewed before they are made available for the users.

The alternative recommended for extensions that require some feature to be handled remotely is using *remote configuration files*.

### Added **Promise support** for many APIs

We can finally use promises in some of `chrome` APIs! 🎈 This was something I was really looking forward to. 

Callbacks are still supported, so you don't need to refactor all your code right away. 

### Other minor changes
- The `browserAction` API and `pageAction` API are now unified in a single API called `action`.
- The **Web-accessible resources** are no longer available to all websites, which allowed extensions to use fingerprinting to track users.
- The method `executeScript()` was moved from the `tabs` API into a new `scripting` API and no longer allows string scripts. You must provide a script file path or a function.
- Host permissions are specified separately from the `permissions` property in the `manifest.json`.
- The `content_security_policy` used to be a string, now it's an object, and you must specify the extension pages (HTML files and service workers) covered by the policy.

# 2. Migrating "Acho, where are we?" to Manifest v3

Now that we know the highlights of Manifest v3 and its vision, we can move on to migrate our sample extension.

## 2.1. Migration checklist

When migrating our extension to manifest v3, the first thing we should do is check the [Manifest V3 migration checklist](https://developer.chrome.com/docs/extensions/mv3/mv3-migration-checklist/). I'll mark each bullet with ✅ when the change applies to our extension or ❌ when it doesn't:

❌ Do you have host permissions in your manifest?

✅ Are you using background pages?
- Replace background.page or background.scripts with background.service_worker in manifest.json. Note that the service_worker field takes a string, not an array of strings.
- Remove `background.persistent` from `manifest.json`.
- Update background scripts to adapt to the service worker execution context.

✅ Are you using the browser_action or page_action property in manifest.json?
- Since these two APIs were unified into a single action API, we must replace these properties with action.

✅ Are you using chrome.browserAction or chrome.pageAction JavaScript API?
- Migrate to the chrome.action API.

❌ Are you currently using the blocking version of chrome.webRequest?

❌ Are you using these scripting/CSS methods in chrome.tabs API?

❌ Are you executing remote code or arbitrary strings?

❌ Are you executing functions that expect an MV2 background context?

❌ Are you making CORS requests in content scripts?

❌ Are you using a custom content_security_policy in manifest.json?

## 2.2. Applying the changes described in the Checklist
Let's review each point from the previous section in-depth and apply the appropriate changes.

### 2.2.1. Set the Manifest version to 3

In the `manifest.json` file, set the value of `manifest_version` to "3".

### 2.2.2. Replacing background pages with service workers

As we replace our background page with a service worker, we must remember two things:
- Service workers are **terminated when inactive** and **restarted when they're needed** again.
- Service workers ** don't have access to the DOM**.

This won't be a problem for us since when I created our background script, I already knew this change was coming, and so I made sure to keep those 2 things in mind in the original design of my background script. 

The first change we need to do is rename the `background.js` script to `service-worker.js`. 

Now we'll set our new service worker in the `manifest.json` file. To do that, we must replace the old `background` property with the following:

```json
"background": {
    "service_worker": "service-worker.js"
},
```

Now, notice that **the `service_worker` property is a string**. So *we can't declare more than one file* there (as far as I know, I didn't find much about this issue in the docs). Because of this change, I couldn't add the other two scripts I needed: `acho.js` and `page.service.js`. So I found a new way to include them and call them from `service-worker.js`: Simply use the `importScripts()` method at the top of the `service-worker.js` script:

```js
// service-worker.js
importScripts('acho.js', 'page.service.js');

/* More code */
```

You can see all the changes I applied to replace my background script with a service worker in [this commit](https://github.com/pawap90/acho-where-are-we/commit/185c939637ffea25db083b3381a84cf5c72d4ab9#diff-9d9dda0262cb14fb69febe39e1fcc19e6b2e02d6560efad8639ec5d2f152e9db).

### 2.2.3. Replacing "browser_action" by "action" in the manifest
Since these two APIs were unified into a single `action` API, we must change the property `browser_action` to `action` in our `manifest.json` file:

```json
{
    "action": {
        "default_popup": "popup.html",
        "default_icon": {
            "16": "images/icon16.png",
            "24": "images/icon24.png",
            "32": "images/icon32.png"
        }
    }
}
```

See the [commit](https://github.com/pawap90/acho-where-are-we/commit/9bfae96327a8be8a368012af398521013dbcab0c?branch=9bfae96327a8be8a368012af398521013dbcab0c).

### 2.2.4. Use the "action" API instead of the "browserAction" API
Similarly to the previous section, we must use the new unified `action` API. 

In our sample extension, we had only used the `browserAction` API to set the badge color and text, so we'll replace those lines:

```js
// acho.js

class Acho {
    
    /* More code */

    growl = () => {
        chrome.action.setBadgeBackgroundColor({ color: '#F00' }, () => {
            chrome.action.setBadgeText({ text: 'grr' });
        });
    }

    quiet = () => {
        chrome.action.setBadgeText({ text: '' });
    }

    /* More code */
}
```

> Note: The `text` property is now required in the `setBadge` method, so we can no longer use `chrome.action.setBadgeText({ });` to clear the badge text.

See the [commit](https://github.com/pawap90/acho-where-are-we/commit/e745baff590c7c55c8ae2a4976964c69acf893b2#diff-9d9dda0262cb14fb69febe39e1fcc19e6b2e02d6560efad8639ec5d2f152e9db).

### 2.2.5. Specify an URL pattern for Web-accessible resources
This one wasn't in the Checklist, but I realized I needed to make a change because when I tried the extension, I got an error that said: "Invalid value for 'web_accessible_resources[0]'. Entry must be a dictionary value".

So, I figure out we must explicitly define which pages will have access to our resources. This is done via the `matches` property (similarly to content scripts). Here's how the new `web_accessible_resources` property looks like in the `manifest.json`:

```json
{
    "web_accessible_resources": [
        {
            "matches": ["<all_urls>"],
            "resources": ["images/icon32.png"]
        }
    ]
}
```

See the [commit](https://github.com/pawap90/acho-where-are-we/commit/4056176cf8cfbd654ee178bccc71aa8f162c82be#diff-9d9dda0262cb14fb69febe39e1fcc19e6b2e02d6560efad8639ec5d2f152e9db).

### 2.2.6. Replace the command "_execute_browser_action" with "_execute_action"
This one wasn't in the Checklist either, and I also couldn't find anything related to this change in the docs, but I figure out the change through my own intuition 😂.

We used to have a `command` defined in our `manifest.json` called `_execute_browser_action` that automatically (without adding any extra code) will trigger our extension's popup (browser action). 

After updating to Manifest v3, this command wasn't working, and I figured it was because of the merge between `browserAction` and `pageAction` into the new `action` API. So I changed `_execute_browser_action` to `_execute_action`, and it worked 🎉.

```json
{
    "commands": {
        "_execute_action": {
            "suggested_key": {
                "default": "Alt+Shift+1"
            }
        }
    }
}
```

### 2.2.7. Refactor to use promises
Finally, after everything else was working, I decided to refactor my code to use promises in the APIs that support them.

Here are some examples:
```js
// Using callback:
chrome.action.setBadgeBackgroundColor({ color: '#F00' }, () => {
    chrome.action.setBadgeText({ text: 'grr' });
});

// Using promises:
await chrome.action.setBadgeBackgroundColor({ color: '#F00' });
await chrome.action.setBadgeText({ text: 'grr' });
```

```js
// Optional callback:
chrome.tabs.create({ url: ev.srcElement.href, active: false });

// Using promises:
await chrome.tabs.create({ url: ev.target.href, active: false });
```

```js
// Using callback:
    chrome.tabs.query(query, (tabs) => {
        // callback logic
    });
});

// Using promises:
const tabs = await chrome.tabs.query(query);
```

> Remember: You don't need to refactor your code to use promises right away. Callbacks are still supported in Manifest v3. 

One thing to notice is that I couldn't make promises to work with the `chrome.storage` API. This may be one of the APIs that don't support promises yet, but I couldn't find more information on the subject in the docs.

Here's [the commit](https://github.com/pawap90/acho-where-are-we/commit/c5234a43794cb38f4316f9af136eeac07457382f#diff-9d9dda0262cb14fb69febe39e1fcc19e6b2e02d6560efad8639ec5d2f152e9db) if you're interested.

### Done!
Our sample extension was successfully migrated to Manifest v3. 

![Charlie Chaplin audience cheering in black and white](https://media.giphy.com/media/GLViT0g0w1NhC/giphy.gif)

# The repo
You can find this and all of the examples of this series in my repo:

{% github pawap90/acho-where-are-we no-readme %}

Hope you found this article useful! 

💬 Let me know what you think in the comments! 