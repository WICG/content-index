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
    DOMString id;
    DOMString title;
    DOMString description;
    ContentCategory category;
    USVString iconUrl;
    USVString launchUrl;
};

interface ContentIndex {
    Promise<void> add(ContentDescription description);
    Promise<void> remove(DOMString id);
    Promise<sequence<ContentDescription>> list();
};
```

## Additions to the Service Worker Registration
```webidl
partial interface ServiceWorkerRegistration {
    [SameObject] readonly attribute ContentIndex index;
};
```
