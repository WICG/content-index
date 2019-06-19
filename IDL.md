# Proposed WebIDL

## Definition of ContentIndex
```webidl
enum ContentCategory {
    "homepage",
    "article",
    "video",
    "audio",
    // ...
};

dictionary ContentDescription {
    required DOMString id;
    required DOMString title;
    required DOMString description;
    required ContentCategory category;
    required USVString iconUrl;
    required USVString launchUrl;
};

interface ContentIndex {
    Promise<void> add(ContentDescription description);
    Promise<void> delete(DOMString id);
    Promise<sequence<ContentDescription>> getDescriptions();
};
```

## Additions to the Service Worker Registration
```webidl
partial interface ServiceWorkerRegistration {
    [SameObject] readonly attribute ContentIndex index;
};
```
