
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

Answer: One advantage is that the user can easily remember the shortening because it's a common dictionary word. However, the word being common, the user might confuse it for another nonce or confuse another nonce for it. To incorporate this change in **NonceGeneration**, one can add a static state storing the most common dictionary words and generate only reads from that static state for its nonce generation.

### Synchronization Questions

1. **Partial matching.** In the first sync (called **generate**), the **Request.shortenUrl** action in the `when` clause includes the **shortUrlBase** argument but not the **targetUrl** argument. In the second sync (called **register**) both appear. Why is this?

The "generate" action only requires input "context" whereas the "register" action's inputs include shortUrlSuffix and shortUrlBase. In the latter case's "when" section "NonceGeneration.generate" requires a "context" input, which is shortUrlBase, and its output "nonce" is the shortUrlSuffix passed into "register".

3. **Omitting names.** The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?
   
This convention is not used in every case because it won't allow for clear dilenation between actions belonging to different concepts, for example in sync register's "when" section generate's output is nonce, which is the input to register as shortUrlSuffix is assigned to it. Additionally, some variables take on concrete values, as in the case of "seconds" in "sync setExpiry" where 3600 is assigned to seconds.

5. **Inclusion of request.** Why is the **request** action included in the first two syncs but not the third one?

   The user of "UrlShortening" doesn't need to manually set the expiration time for a resource. That should be handeled internally.

7. (????) **Fixed domain.** Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?
```
sync generate
  when Request.shortenUrl (shortUrlBase)
  then NonceGeneration.generate(context: shortUrlBase)

sync register
  when
    Request.shortenUrl (targetUrl, shortUrlBase)
    NonceGeneration.generate (): (nonce: ".ly")
  then UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: "bit", targetUrl)

sync setExpiry
  when UrlShortening.register (): (shortUrl)
  then ExpiringResource.setExpiry (resource: shortUrl, seconds: 3600)
```

9. **Adding a sync.** These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the **ExpiringResource** and **URLShortening** concepts.
```
sync expireResource
  when ExpiringResource.expireResource(): resource
  then 

```
