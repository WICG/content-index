# Content Indexing
**Written**: 2019-04-29<br/>
**Updated**: 2019-04-29

High quality offline-enabled web content is not easily discoverable by users right now. They would have to know which websites work offline or install a PWA to be able to browse through content while offline. This is not a great user experience. To address this, we propose a new API on the service worker registration so developers can expose fresh content to their users.

The **content index** allows websites to register their offline enabled content in the browser. This allows the user to browse through content while offline and potentially search on-device for a specific article.

## Why do we need this?
Unrealiable or even unavailable network connections are very common on mobile devices. Even in connected cities like London, people have very limited connectivity while travelling underground. This API would allow browsers to show meaningful content to users in these situations and sites to increase user engagement.

## Goals
- Allow users to easily find content even while **offline**
- Surface high quality content in native spaces (example: [Android Slices](https://developer.android.com/guide/slices))

## Non-goals
- Storage of the offline content itself

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
  launchUrl: 'https://website.dev/articles/123'
});

// Remove an entry from the content index
await swRegistration.index.remove('article-123');

// List all entries in the content index
const entries = await swRegistration.index.list();
```

### Combined with other APIs
When used together with the proposed [Periodic Background Sync API](https://github.com/beverloo/periodic-background-sync), this allows websites to automatically sync fresh content and make it available to users.
```javascript
// Add article to content index
function addArticleToIndex(article) {
  return self.index.add({
    id: article.id,
    title: article.title,
    description: article.description,
    category: 'article',
    iconUrl: article.icon,
    launchUrl: '/articles/' + article.id
  });
}

// Fetch new content, cache it and add reference to content index
async function updateLatestNews() {
  const latestNews = await fetch('/latest-news');
  // TODO: cache content
  for (const article : latestNews) {
    await addArticleToIndex(article);
  }
}

// Handle periodic sync event in service worker
self.addEventListener('periodicsync', (event) => {
  if (event.registration.tag === 'get-latest-news') {
    event.waitUntil(updateLatestNews());
  }
});
```

## Alternatives considered
### Extending the Cache interface
One of the requirements for this API is that the exposed content is available offline. In the case of an article this could be implemented by simply adding the response to the [Service Worker Cache](https://developers.google.com/web/ilt/pwa/caching-files-with-service-worker). We could extend this API to specify that specific cached entries can be exposed to the user.

This would limit some use cases as new content would have to be served from a server. When using the **content index**, developers could generate and store content offline and then add it to the index, making it available without any network connection.
