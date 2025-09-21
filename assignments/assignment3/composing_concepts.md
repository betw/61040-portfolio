Here are the questions formatted in markdown.

### Contexts
(???) The **NonceGeneration** concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

Answer: A context in the URL shortening app will be the targetURL for which mutliple shortURLs can link to, thus requiring targetURL to have a set of unique strings to allocate to the shortURLs.
---

### Storing Used Strings
Why must the **NonceGeneration** store sets of used strings? One simple way to implement the **NonceGeneration** is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

Answer: **NonceGeneration** must store sets of used strings because the purpose of the concept is to "generate unique strings within a context," so you need a set of strings "generate" has already made to be able to return a unique string. The counter in the implementation represents "counter" number of unique used strings.

---

### Words as Nonces
One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the **NonceGeneration** concept to realize this idea?

Answer: One advantage is that the user can easily remember the shortening because it's a common dictionary word. However, the word being common, the user might confuse it for another nonce or confuse another nonce for it. To incorporate this change in **NonceGeneration**, one can add a static state storing the most common dictionary words and generate only reads from that static state for its nonce generation.
