# Concept Questions

1. The context in the NonceGeneration are for ensuring that uniqueness is only guaranteed within that existing context. Essentially, the same string could exist in two different contexts. In the URL shoterning app, a context will end up being the short-url base. The serparate contexts would be a different base domain (e.g tinyurl.com) and this prveents collisions with nonces that are generate for another base.

2. The NonceGeneration stores sets of used strings because the concept describes the return of a string that has not been returned before for a context. The concept needs a way to be able to tell what it has already produced. In this case, the set of used strings in the specification is related to the counter in the implementation in the sense that in the implementation version of the concept every context increases its used count by 1 for every generate called. In the abstraction function, the counter from the implementation would map to the set of used strings for a context. The set of used used strings corresponds to all strings that were generated up to but not including the current count value in the implementation form for a given context.

3. An advantage of using this sceheme is that the generated shortenings are more user-friendly and can be easily inputted to the search bar. A disadvantage of using this scheme is that the word selected may be ambiguous to the user and there is not enough uniqueness as there are with random strings meaning collisions are bound to happen whereas multiple words would guarantee more uniqueness. A way to resolve this issue would be to limit the selection pool of words using a dictionary. I would extend the concept as follows:

<br />

**concept** NonceGeneration [Context]

**purpose** generate unique strings within a context

**principle** each generate returns a string not returned before for that context

**state**

a set of Contexts with

a used set of Strings

a dictionary set of Strings

**actions**

generate (context: Context) : (nonce: String)

**effect** returns a nonce that is not already used by this context, drawing from dictionary

# Synchronization Questions

1. In the generate sync no targetUrl is required since the context is the only required information to generate a nonce. In the register sync both targetUrl and shortUrlBase are needed since the registration of the url essentially maps the shortUrl to the targetUrl therefore needing the targetUrl argument.

2. This convention isn't used in every case since it is possible this would cause ambiguity if two bindings with the same name exist from different actions. In this case, it is only convenient to omit names only when it is guaranteed that there is no risk of any name collisions.

3. The request action is included in the first two syncs but not the third one because at that point the shortened url has been generated since it is only called upon the completion of the registration of the shortened url.

4. If there was a fixed base/domain, the modification to the shortUrlBase constant would be as follows (bit.ly constant used as example but would be specified fixed domain):

**sync** generate

**when** Request.shortenUrl (targetUrl)

**then** NonceGeneration.generate (context: "bit.ly")

<br />

**sync** register

**when**

Request.shortenUrl (targetUrl)

NonceGeneration.generate (): (nonce)

**then** UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: "bit.ly", targetUrl)

<br />

**sync** setExpiry

**when** UrlShortening.register (): (shortUrl)

**then** ExpiringResource.setExpiry (resource: shortUrl, seconds: 3600)

<br />

5. The sync I would add for post expiration would be

**sync** postExpire

**when** ExpiringResource.expireResource (): (resource)

**then** UrlShortening.delete (shortUrl: resource)

# Extending the Design

## Additional Concepts:

**concept** ShortOwner [ShortUrl, User]

**purpose** record creator of designated shortening (restrcited to creator) for analytics

**principle** after user registers shortening they become the owner

**state**

a set of ShortUrls with

an owner User

**actions**

assign (user: User, shortUrl)

**effect** records user as owner of shortUrl

owner (shortUrl) : (user: User)

**requires** a ShortOwner exists for shortUrl

**effect** returns the owner user of shortUrl

<br />

**concept** ShortAnalytics [ShortUrl]

**purpose** maintain and retrieve the access count for every shortened url

**principle** after short url is accessed, count increases by one and only owner can view

**state**

a set of ShortUrls with

an accessCount Number

**actions**

recordAccess (shortUrl)

**effect** increment accessCount for shortUrl by 1

getCount (user: User, shortUrl): (n: Number)

**requires** ShortOwner.owner(shortUrl) == user

**effect** returns accessCount for shortUrl

<br />

## Additional Syncs:

**sync** onRegister

**when**

URLShortening.register (): (shortUrl)

Request.createShortening (): (user)

**then**

ShortOwner.assign(u: User, s:ShortUrl)

ShortAnalytics.recordAccess(s: shortUrl)

<br />

**sync** onLookup

**when** UrlShortening.lookup (shortUrl)

**then** ShortAnalytics.recordAcess (s: shortUrl)

<br />

**sync** onViewAnalytics

**when** Request.viewAnalytics (user, shortUrl)

**where** ShortOwner.owner(shortUrl) == user

**then** ShortAnalytics.getCount (u: user, s: shortUrl)

## 3.

### Allowing users to choose their own short URLs:

This feature is already supported by the shortUrlSuffix parameter when registering (UrlShortening.register(shortUrlSuffix: nonce, shortUrlBase, targetUrl)). Even though the original precondition that no shortening exists for that specific shortUrlBase, an additional sync could be added to check this.

### Using the “word as nonce” strategy to generate more memorable short URLs:

To incorporate this feature, we could extend the NonceGeneration concept similar to the addition of the dictionary. The generate sync would be updated to call this new version as well.

**concept** NonceGeneration [Context]

**state**

a set of Contexts with

a used set of Strings

a dictionary set of Strings

**actions**

setDictionary (context: Context, dict: set String)

**effect** associates a dictionary of words with the context

generate (context: Context): (nonce: String)

**effect** returns a nonce from the dictionary if one is provided that is not already used in this context

<br />

**sync** generate

**when** Request.shortenUrl (shortUrlBase)

**then** NonceGeneration.generate (context: shortUrlBase)

### Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL:

To include this feature, a new concept can be added, which would require a state and sync addition.

**concept** TargetAnalytics [TargetUrl]

**state**

a set of TargetUrls with

an accessCount Number

**actions**

recordAccess(targetUrl: String)

**effect** increment accessCounr for targetUrl

getCount (targetUrl: String): (n: Number)

**effect** returns accessCount for targetUrl

<br />

**sync** onLookupTarget

**when** URLShortening.lookup(shortUrl): (targetUrl)

**then** TargetAnalytics.recordAccess(targetUrl)

### Generate short URLs that are not easily guessed:

To incorporate this feature the NonceGeneration concept can be updated as follows in order to generate a more secure generated url if secure mode is on (adding a flag):

**concept** NonceGeneration [Context]

**state**
a set of Contexts with

a used set of Strings

a secureMode Flag

**actions**

setSecureMode (context: Context, secure: Boolean)

**effect** turns on/off random generation

generate (context: Context): (nonce: String)

**effect** if secureMode then generate random string, else normal generation

### Supporting reporting of analytics to creators of short URLs who have not registered as user:

This feature would be undesireable since this would be a breach to the private aspect of the concept. In this case, registering an owner wouldn't make sense anymore since the analytic would be visible to everyone which could result in a violation of privacy rules.
