# Content Indexing
**Written**: 2019-04-29<br/>
**Updated**: 2019-05-08

High quality offline-enabled web content is not easily discoverable by users right now. They would have to know which
websites work offline or install a PWA to be able to browse through content while offline. This is not a great user
experience as there is no central point to discover available content. To address this, we propose a new API to allow
developers to tell the browser about their specific content.

The **content index** allows websites to register their offline enabled content in the browser. This allows the browser
to improve their offline capabilities and offer content to users to browse through while offline. This data could also
be used to improve on-device search and augment browsing history.

## Why do we need this?
Unrealiable or even unavailable network connections are very common on mobile devices. Even in connected cities like
London, people have very limited connectivity while travelling underground. This API would allow browsers to show
meaningful content to users in these situations and sites to increase user engagement.

Browser vendors are already looking for content relevant to the user, based on their browsing history, and make it
available to be consumed offline. This is not ideal as it ignores the entity with the most knowledge of that content -
the providers themselves. With this API they can highlight user specific, high quality content through the browser.
Grouping content by a category (e.g. 'article', 'video', 'audio') allows an even richer experience as the browser is
able to understand what kind of content is available and show a relevant UI.

## Goals
- Allow users to easily find content even while **offline**
- Surface high quality content in native spaces (example: [Android Slices](https://developer.android.com/guide/slices))

## Non-goals
- Storage of the offline content itself<br>
  *We expect developers to use more specialized APIs to store content (see
  [Service Worker Caches](https://w3c.github.io/ServiceWorker/#cache-objects) or
  [Web Storage](https://www.w3.org/TR/webstorage/)).*

## Broader API landscape
### Service Worker
We propose to add this API as an extension to [Service Workers](https://w3c.github.io/ServiceWorker/). This allows
browsers to check if the given content is actually available offline. This also makes it easier for developers, as the
entries get removed automatically if the service worker is unregistered (and therefore can not provide the offline
content anymore).

### CacheStorage API
The [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache) allows websites to cache requests and,
for example, use the cached content in the `fetch` event of a service worker. This makes it easy to ensure that some
content is available offline, and is one of the steps to create high quality
[Progressive Web Apps](https://codelabs.developers.google.com/codelabs/your-first-pwapp/#5).

### Web Packaging
[Web Packaging](https://github.com/WICG/webpackage) is a proposed API to bundle resources of a website together, so they
can be shared offline. This also allows them to be securely distributed as a bundle. This API plays nicely together with
**Content Indexing**, making it easier to ensure all necessary content is available offline.

## Security and Privacy
Developers have control over which content they want to make available to the browser. The lifetime of an entry in the
**content index** is comparable to that of Notifications, but with a less intrusive UX and more structured content. When
adding personalized content, websites can simply remove entries on logout (and close all open Notifications). The
storage required to store the entries of the index itself count towards the quota of the origin.
[This document](SECURITY_AND_PRIVACY.md) contains additional answers of the [security & privacy questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/).

## Examples
Please see [this separate document](IDL.md) for the proposed WebIDL additions.

### General usage
```javascript
// Add an article to the content index
await swRegistration.index.add({
  id: 'article-123',
  title: 'Article title',
  description: 'Amazing article about things!',
  category: 'article',
  iconUrl: 'https://website.dev/img/article-123.png',
  launchUrl: 'https://website.dev/articles/123',
});

// Remove an entry from the content index
await swRegistration.index.remove('article-123');

// List all entries in the content index
const entries = await swRegistration.index.list();
```

### Combined with other APIs
Sending breaking news articles via Push API allows websites to keep their users up to date. Adding these articles to the
**content index** allows the browser to highlight them and make them discoverable later on. In this example we make use
of the [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache) to cache content resources, and the
[Indexed Database](https://w3c.github.io/IndexedDB/) to store the structured content.
```javascript
async function handlePush(data) {
  // Fetch additional data about pushed content
  const news = await fetch(`/api/news/${data.id}`);

  // Store content in database and cache resources
  await Promise.all([db.add(news), cache.add(news.iconUrl)]);

  // Add content to content index
  if ('index' in self.registration) {
    await self.registration.index.add({
      id: news.id,
      title: news.title,
      description: news.description,
      category: 'article',
      iconUrl: news.iconUrl,
      launchUrl: `/news/${news.id}`,
    });
  }

  // Display a notification
  return self.registration.showNotification(news.title, {
    tag: news.id,
    body: news.description,
    icon: news.iconUrl,
  });
}

// Handle web push event in service worker
self.addEventListener('push', event => event.waitUntil(handlePush(event.data.json())));
```

When used together with the proposed
[Periodic Background Sync API](https://github.com/beverloo/periodic-background-sync), this allows websites to
automatically sync fresh content and make it available to users.
```javascript
// Add an article to the content index
function addArticleToIndex(article) {
  return self.registration.index.add({
    id: article.id,
    title: article.title,
    description: article.description,
    category: 'article',
    iconUrl: article.iconUrl,
    launchUrl: '/articles/' + article.id,
  });
}

// Fetch new content, cache it and add it to the content index
async function updateLatestNews() {
  const latestNews = await fetch('/latest-news');
  // TODO: cache content
  if ('index' in self.registration) {
    await Promise.all(latestNews.map(addArticleToIndex));
  }
}

// Handle periodic sync event in service worker
self.addEventListener('periodicsync', event => {
  if (event.registration.tag === 'get-latest-news') {
    event.waitUntil(updateLatestNews());
  }
});
```

## Alternatives considered
### Extending the Cache interface
One of the requirements for this API is that the exposed content is available offline. In the case of an article this
could be implemented by simply adding the response to the
[Service Worker Cache](https://developers.google.com/web/ilt/pwa/caching-files-with-service-worker). We could extend
this API to specify that specific cached entries can be exposed to the user.

This would limit some use cases as new content would have to be served from a server. When using the **content index**,
developers could generate and store content offline and then add it to the index, making it available without any
network connection.

## References
  * [Push API](https://w3c.github.io/push-api/)
  * [Background Sync](https://wicg.github.io/BackgroundSync/spec/)
  * [Periodic Background Sync](https://github.com/beverloo/periodic-background-sync)
  * [Notifications API](https://github.com/whatwg/notifications)

Many thanks to [@beverloo](https://github.com/beverloo) and [@jakearchibald](https://github.com/jakearchibald) for their
ideas, input and discussion.
