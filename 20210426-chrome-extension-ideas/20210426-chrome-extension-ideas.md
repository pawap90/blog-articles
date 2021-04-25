---
title: 5 Ideas to set your portfolio apart
published: false
description: Tired of building TO-DO lists and e-commerce sites? One of these Chrome extension ideas could set your portfolio apart!
tags: chromeextension, javascript, portfolio, beginners
cover_image: https://i.imgur.com/4pKqrb8.png
series: chrome-extensions
---

Tired of building TO-DO lists and e-commerce sites? One of these Chrome extension ideas could help you set your portfolio apart! 👩‍💻

All you need to know to build your first Chrome Extension is the **basics of HTML, JavaScript, and CSS**! These are a few fun ideas I came up with that can be created using only those technologies.

If you're having doubts, check out my video [Creating a simple Chrome Extension in 2 minutes](https://www.instagram.com/tv/CLIRDc4gWyz/).

# 1. Dark mode for your favorite website

![Dark mode icon - The moon, some clouds and a few stars in the sky](https://i.imgur.com/pd7BYKa.png)

Do you regularly use a website that you love but doesn't have dark mode? 

With Chrome extensions, you can create your own CSS stylesheet that overrides some styles in a particular website. This allows you to apply dark mode styles over the website when your extension is enabled.

To achieve that, you should use **Content Scripts**.

> Tip: Set `"run_at": "document_end"`  in the `content_scripts` object in the `manifest.json` so your styles override the site's.

For this project, you'll need to learn about:
- [Content Scripts](https://developer.chrome.com/docs/extensions/mv3/content_scripts/)

# 2. Weather App

![Weather icon - A cloud, rain and the sun behind it](https://i.imgur.com/7XO2kWk.png)

You could create an extension that shows the current weather for a particular location on your browser. 

You'll need to call an API to retrieve the weather info: [OpenWeatherMap API](https://openweathermap.org/api/one-call-api) is a good option. They have a free plan.

Get the latest weather updates every few minutes in the background using an alarm from the `chrome.alarms` API handled by a Service Worker.

Store the results using the `chrome.storage` API and display them in your extension's Action Popup.

To call the API, you can use `fetch`, and you'll need to add something like the following to your `manifest.json`: 

```json
"host_permissions": ["https://some-weather-api.com/*"]
```

For this project you'll need to learn about:
- [`chrome.storage` API](https://developer.chrome.com/docs/extensions/reference/storage/)
- [`chrome.alarms` API](https://developer.chrome.com/docs/apps/app_codelab_alarms/)
- [`chrome.action` API](https://developer.chrome.com/docs/extensions/reference/action/)
- [Service Workers](https://developer.chrome.com/docs/extensions/mv3/background_pages/)

# 3. Show today's calendar.

![Calendar icon - A calendar with a clock](https://i.imgur.com/1N4uIDX.png)

You could get easy access to your Calendar events for the day, showing them in your Browser through an Extension. 

As with the previous idea, you'll need to get the data from an external API (Google Calendar's or your favorite calendar API) every few minutes in the background using an alarm from the `chrome.alarms` API and a Service Worker. 

You'll also need to store the results of the API request using `chrome.storage`. Remember that Service Workers can be unloaded when the extension goes idle, so you can't just store the calendar events in a variable declared in your Service Worker.

Display the events with their title, time, guests, etc., in your extension's Action Popup.

You can also use `chrome.notifications` API to show a message to the user a few minutes before the event.

For this project you'll need to learn about:
- [`chrome.storage` API](https://developer.chrome.com/docs/extensions/reference/storage/)
- [`chrome.alarms` API](https://developer.chrome.com/docs/apps/app_codelab_alarms/)
- [`chrome.action` API](https://developer.chrome.com/docs/extensions/reference/action/)
- [`chrome.notifications` API](https://developer.chrome.com/docs/extensions/reference/notifications/)
- [Service Workers](https://developer.chrome.com/docs/extensions/mv3/background_pages/)

# 4. Daily coffee

![Coffee cup icon - A coffee cup with coffee](https://i.imgur.com/IaSq7MB.png)

Show the user a random coffee recipe to try every day.

In this case, you don't need to call an external API (at least at first). You could just store a list of your favorite recipes in a JSON array in your extension and use a `Math.random()` to get a random item from the array every day.

Display the coffee recipe with a nice picture in the Action Popup.

To make it more robust, you could use `chrome.storage` to store the last coffee recipe shown and the date to make sure of two things:
- A single coffee recipe is shown every day.
- We don't get the same recipe two days in a row.

For this project you'll need to learn about:
- [`chrome.action` API](https://developer.chrome.com/docs/extensions/reference/action/)
- [`chrome.storage` API](https://developer.chrome.com/docs/extensions/reference/storage/) (optional)

# 5. E-Commerce wishlist

![Wishlist icon - A page with a list of items and a heart](https://i.imgur.com/yTvftWv.png)

Allow users to add products from Amazon, eBay, etc., to their wishlist: When a user finds a product they're interested in, they add it to their wishlist using a keyboard shortcut or from the **Action popup**.

The Action popup also shows the previously added items and allows the user to remove them.

The user should also be able to navigate to each product page in case they want to buy it. For this, you should use `chrome.tabs.create`.

You'll need to store links, name, and price for each product using `chrome.storage` API. 

For this project you'll need to learn about:
- [`chrome.action` API](https://developer.chrome.com/docs/extensions/reference/action/)
- [`chrome.storage` API](https://developer.chrome.com/docs/extensions/reference/storage/) 
- [`chrome.tabs` API](https://developer.chrome.com/docs/extensions/reference/tabs) - Particularly the [`create` method](https://developer.chrome.com/docs/extensions/reference/tabs/#method-create)

---

I hope this post was helpful and inspires you to work on your next project! If you're interested in learning about Chrome extensions, check out [my series about it](https://dev.to/paulasantamaria/creating-a-simple-chrome-extension-36m) where I create a simple chrome extension and gradually improve it. At the same time, I explore most of the `chrome` APIs and concepts required to build the ideas mentioned in this post.

You can also check out my video [Creating a simple Chrome Extension in 2 minutes](https://www.instagram.com/tv/CLIRDc4gWyz/) to get a grasp of the process.

*Icons by Smashicons & Freepik on Flaticon.com*