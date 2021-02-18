# Proposed WebIDL

## Definition of ContentIndex
```webidl
enum ContentCategory {
    "",
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
    ContentCategory category = "";
    sequence<ImageResource> icons = [];
    required USVString url;
};

interface ContentIndex {
    Promise<void> add(ContentDescription description);
    Promise<void> delete(DOMString id);
    Promise<sequence<ContentDescription>> getAll();
};
```

## Additions to the Service Worker Registration
```webidl
partial interface ServiceWorkerRegistration {
    [SameObject] readonly attribute ContentIndex index;
};
```

## Additions to the Service Worker Global Scope
```webidl
dictionary ContentIndexEventInit : ExtendableEventInit {
    required DOMString id;
};

interface ContentIndexEvent : ExtendableEvent {
  constructor(DOMString type, ContentIndexEventInit init);
  readonly attribute DOMString id;
};

partial interface ServiceWorkerGlobalScope {
    attribute EventHandler oncontentdelete;
};
```
