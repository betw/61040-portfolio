# Problem Set 2: Composing Concepts

### Contexts
The **NonceGeneration** concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

Answer: The contexts generally in **NonceGeneration** are for the ability to possibly use the same set or subset of unique strings across **different** contexts. A context in the URL shortening app will be the targetURL for which mutliple shortURLs can link to, thus requiring targetURL to have a set of unique strings to allocate to the shortURLs.

---

### Storing Used Strings
Why must the **NonceGeneration** store sets of used strings? One simple way to implement the **NonceGeneration** is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

Answer: **NonceGeneration** must store sets of used strings because the purpose of the concept is to "generate unique strings within a context," so you need a set of strings "generate" has already made to be able to return a unique string. The counter in the implementation represents "counter" number of unique used strings.

---

### Words as Nonces
One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the **NonceGeneration** concept to realize this idea?

Answer: One advantage is that the user can easily remember the shortening because it's a common dictionary word. However, the word being common, the user might confuse it for another nonce or confuse another nonce for it. To incorporate this change in **NonceGeneration**, one can add a static state storing the most common dictionary words and generate randomly selects a word from that static state at most once.

### Synchronization Questions

1. **Partial matching.** In the first sync (called **generate**), the **Request.shortenUrl** action in the `when` clause includes the **shortUrlBase** argument but not the **targetUrl** argument. In the second sync (called **register**) both appear. Why is this?

The "generate" action only requires input "context" whereas the "register" action's inputs include shortUrlSuffix and shortUrlBase. In the latter case's "when" section "NonceGeneration.generate" requires a "context" input, which is shortUrlBase, and its output "nonce" is the shortUrlSuffix passed into "register".

3. **Omitting names.** The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?
   
This convention is not used in every case because it won't allow for clear dilenation between actions belonging to different concepts, for example in sync register's "when" section generate's output is nonce, which is the input to register as shortUrlSuffix is assigned to it. Additionally, some variables take on concrete values, as in the case of "seconds" in "sync setExpiry" where 3600 is assigned to seconds.

5. **Inclusion of request.** Why is the **request** action included in the first two syncs but not the third one?

   The user of "UrlShortening" doesn't need to manually set the expiration time for a resource. That should be handeled internally.

7. **Fixed domain.** Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?
```
sync generate
  when Request.shortenUrl (shortUrlBase: "bit.ly")
  then NonceGeneration.generate(context: shortUrlBase)

sync register
  when
    Request.shortenUrl (targetUrl, shortUrlBase: "bit.ly")
    NonceGeneration.generate (): (nonce: ".ly")
  then UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: "bit", targetUrl)

sync setExpiry
  when UrlShortening.register (): (shortUrl: "bit.ly")
  then ExpiringResource.setExpiry (resource: shortUrl, seconds: 3600)
```

9. **Adding a sync.** These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the **ExpiringResource** and **URLShortening** concepts.
```
sync expireResource
  when ExpiringResource.expireResource(): resource
  then UrlShortening.delete(shortUrl: resource)

```


### Extending the design
Design a couple of additional concepts to realize this extension, and write them out in full (but including only the essential actions and state).
```
concept: countAnalytics [User]
purpose: allow a user to view analytics about some service usage
principle: for each general (not only the user) access to the resource, the count associted with the resource
increases by one
state:
   a set of Analytics with
      a User user
      a set of CountAccess
   a set of UserData with
      a User user
      a set of Associations
   a set of Associations with
      a String base
      a set of Strings derivative
   a set of CountAccess with
      a String base
      a String derivative
      a number count
actions:
   addString(user: User, base: String, derivative: String)
      requires: base and derivative are unique among users
      effect: add derivative to set associated with base belonging to user
   incrementAccessCount(derivative: String)
      requires: derivative is associated with some base belonging to some user
      effect: increments the count associted with derivative belonging to some user
   accessCounts(user: User, base: String): count: number
      requires: base exists in Associations belonging to user
      effect: return the accumulated counts belonging to each derivative associated with base

concept: userAuth
purpose: allow users to signup and authenticate themselves
principle: users register with a username and password and use those
credentials from then on for authentication
state:
   a set of Users with
      a String username
      a String password
actions:
   register(username: String, password: String): User
      requires: username not already in use
      effect: add a new User with username username and password password
   authenticate(username: String, password: String): User
      requires: username exists and password matches
      effect: return the User with that username and password
```
Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics.
```
sync 
   when
       Request.registerUser(username, password)
       Request.register(shortUrlSuffix, shortUrlBase, targetUrl)
       UrlShortening.register(shortUrlSuffix, shortUrlBase, targetUrl): shortUrl
   then
      userAuth.register(username, password): User
      countAnalytics.addString(user, targetUrl, shortUrl)

sync
   when UrlShortening.lookup(shortUrl): targetUrl
   then countAnalytics.incrementAccessCount(shortUrl)

sync
   when Request.examineAnalytics(username, password, targetUrl)
   then
      userAuth.authenticate(username, password): user
      countAnalytics.accessCounts(user, base: targetUrl): count
        
```
As a way to assess the modularity of your solution, consider each of the following feature requests, to be included along with analytics. For each one, outline how it might be realized (eg, by changing or adding a concept or a sync), or argue that the feature would be undesirable and should not be included:

Allowing users to choose their own short URLs;

Add a new sync where **NonceGeneration's** generate is not called. Instead, the user will provide their own shortUrlSuffix and Base, not already taken, and the sync will include a call to UrlShortening.register.

Using the “word as nonce” strategy to generate more memorable short URLs; 

One way to incorporate this change is to add a state to the NonceGeneration concept that includes the most common dictionary words and generate only selects words from that static state.

Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL;

I have already included this in my concepts.

Generate short URLs that are not easily guessed;

One can change the generate action in **NonceGeneration** to create a random string, including letters, numbers, and other symbols.

Supporting reporting of analytics to creators of short URLs who have not registered as user;

This feature should NOT be included because without identification, anyone can view the analytics of the shortUrl's creator, which might be undesirable as potential competitors might glean some insight from it.
