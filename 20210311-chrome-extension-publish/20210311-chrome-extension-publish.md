---
title: Publishing a Chrome Extension
published: true
description: Steps to publish a Chrome Extension
tags: chromeextension, webdev, javascript, chrome
cover_image: https://i.imgur.com/5TAZR1T.png
series: chrome-extensions
---

This series wouldn't be complete without a post about how to **publish** a Chrome Extension, so here it is!

# 1. Prepare the extension to be published
We need to create a **.zip file** containing the source code for our extension. The only required file is the `manifest.json`, but we'll need to include the whole project if we want everything to work correctly.

We'll later upload this file to the Chrome Developer Dashboard.

# 2. Create a Chrome developer account

To register as a Chrome Web Store developer, we'll need to access the [developer console](https://chrome.google.com/webstore/devconsole).
Once we do that, we'll have to accept the Developer Agreement and Privacy Policies and pay the $5 registration fee (a one-time payment).

>Don't forget to access the Account panel and fill in your email address. If you don't, you won't be able to publish an extension.

# 3. Publish the extension

We'll go to the "Items" panel and click over the "New Item" button to publish our extension.
We'll see a modal where we'll drop our .zip file (the one created in step 1).

![](https://i.imgur.com/qVULKua.gif)

After uploading the file, we'll be redirected to the "Store Listing" form. Here we'll have to fill all required fields, which include:
- Name
- Description
- Category
- Language
- Small Icon (128 x 128 px)
- At least one Screenshot

After filling all the required fields, we should go ahead and do the same on the "Privacy practices" form. Here we'll need to explain the **purpose** of the extension and **justify why we need each of the permissions** we listed in our `manifest.json`.

> **Host Permission Justification**
> Given that we added a content script with access to *all the web pages* visited by our users in this example, we'll need to justify why we need that, and **our extension will take longer to get verified**.

![](https://i.imgur.com/EVrd3BA.gif)

After completing all fields in both forms, check the buttons at the top-right of the screen:

![The buttons "Save Draft" and "Submit for review". The latter is greyed out.](https://i.imgur.com/wZLX8AG.png)

If the button "Submit for review" is greyed out, click over "Why can't I submit?" to learn what's missing.

Once we've met all the requirements, click "Submit for review":

![](https://i.imgur.com/HdRRpe3.png)

Now our extension is submitted, and we just need to wait for it to be reviewed and approved!

In this case, since we added a content script that requires access to all web pages, we'll need to wait a little longer for the review. 

# The repo
You can find all of the examples of this series in my repo:

{% github pawap90/acho-where-are-we no-readme %}
