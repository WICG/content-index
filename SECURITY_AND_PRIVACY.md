# Answers to [Security and Privacy Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/)

### 2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

This feature does not give access to any other information than what the Web site itself supplies to this API. The supplied information can then be used to surface offline available content to the user to be consumed when there is no network connection.

### 2.2 Is this specification exposing the minimum amount of information necessary to power the feature?

No extra information exposed.

### 2.3 How does this specification deal with personal information or personally-identifiable information or information derived thereof?

This feature does not give access to any other information than what the Web site supplies to this API.

### 2.4 How does this specification deal with sensitive information?

This feature does not give access to any other information than what the Web site supplies to this API. Furthermore, the data is tied to a service worker registration, and therefore to encrypted and authenticated origins.

### 2.5 Does this specification introduce new state for an origin that persists across browsing sessions?

Yes, however, this data is tied to a service worker registration and its lifetime. See [service worker privacy section](https://www.w3.org/TR/service-workers-1/#privacy).

### 2.6 What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

None.

### 2.7 Does this specification allow an origin access to sensors on a user’s device?

No.

### 2.8 What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

This feature only exposes data that the origin itself supplied to the API.

### 2.9 Does this specification enable new script execution/loading mechanisms?

No new script loading mechanism enabled. The user agent might load new resources (image data) when a new entry is added with this API.

### 2.10 Does this specification allow an origin to access other devices?

No.

### 2.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

The user agent might display information from an origin that uses this API. This includes text and image / icon data. However, the user agent is in full control over if, how and when this data should be displayed in its UI.

### 2.12 What temporary identifiers might this this specification create or expose to the web?

All identifiers are provided by the Web site itself.

### 2.13 How does this specification distinguish between behavior in first-party and third-party contexts?

The data will only be available to first-party contexts.

### 2.14 How does this specification work in the context of a user agent’s Private \ Browsing or "incognito" mode?

It will work the same way. During an active "incognito" session, the user agent might show the passed data in its UI, however, as soon as that session ends, the data will be removed together with all other data tied to the service worker registration.

### 2.15 Does this specification have a "Security Considerations" and "Privacy Considerations" section?

Yes. See the [explainer](README.md#security-and-privacy).

### 2.16 Does this specification allow downgrading default security characteristics?

No.
