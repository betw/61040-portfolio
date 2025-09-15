# Exercise 1: Reading a concept
1. **Invariants** (?). **What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases)**
     One invariant is that the counts of items associated with a Request is greater than or equal to 0. Another invariant is that every set of Purchases is for the item associated with a Request. The second invariant is more important because the purpose of the concept is to "track purchases of requested gifts," but if the set of Purchases associated with a Request is for an item not associated with the Request, then one can't properly track the purchases of requested gifts. Though, the action "purchase" preserves the above invariant as the precondition requires the registry contains a request for the item with at least count and creates a new purchase specifically for that item.
2. **Fixing an action**. **Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?**
   The "purchase" action can break the invariant that counts of items associated with a Request is greater than or equal to 0, as the precondition only requires that the registry has a request for the input item "with at least count". The input count could be negative, and if the counts of items associated with the request for the input item is 0, then the effect of "purchase" will decrement the count in request resulting in a negative count. To fix this problem, require that the input count into "purchase" is greater than or equal to 1.
3. **Inferring behavior**. **The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?**
   Based on "open" and "close" action specs, one can repeatedly open/close a registry, since if an existing reqistry started as inactive the preconditions of open are the postconditions of close and the preconditions of close the postconditions of open. There exists a good reason for this feature, namely a user, at some point, might not want people to buy their requested items in the registry so they close it for a time, and they open it when they want people to buy things for them again.
4. **Registry deletion**. **There is no action to delete a registry. Would this matter in practice?**
    Making the registry inactive is effectively "deleting" the registry in practice because no one, other than the owner of the registry, can see it or make purchases.
5. **Queries**. **What are two common queries likely to be executed against the concept state?**
     The registry owner would want to query the set of Purchases associated with a Request. A giver of a gift would want to query the set of Requests of a registry.
6. **Hiding purchases**. **A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?**
   One can add "a visiblitiy Flag" to a "Request" so that when the flag is True, the recipient can see the purchases, and when it is false, the reciipient can't. Then, add two additional actions "makePurchasesVisible"/"makePurchasesHidden" which mirror how "open/close" work, but their input types is of type Request.
7. **Generic types**. **The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc**
     Specifying User and Item types as generic parameters, identified with some unique code is more generalizable, in the sense it can be used to identify many different types of Users (organizations, individuals) and Items. Additionally, prices change for Items so a unique code will allow us to still identify some Item irrespective of its current price without additional changes.
   
# Exercise 2: Extending a familiar concept
- concept: PasswordAuthentication [User]
- purpose: limit access to known users
- principle: after a user registers with a username and a password,
    they can authenticate with that same username and password
    and be treated each time as the same user
- state
  - a set of Users with:
       - a string Username
       - a string Password
       - a string Email
       - a verification Flag
  - a set of Tokens with:
       - a User
       - a number token
- actions
  - register (username: String, password: String, email: String): (user: User, token: Number)
       - requires: username doesn't already exist in the usernames associated with the set of Users
       - effect: create and return a User, add the User and token to set of Tokens, and send the random number token to email
  - authenticate (username: String, password: String): (user: User)
       - requires: username exists in the set of Users' usernames and verification Flag associated with username is True
       - effect: return the User with the matching username and password, Error if the password doesn't match the password associated
         with the username
  - confirm (username: String, token: Number): (user: User)
       - requires: the username is associated with some user in the set of Users
       - effect: change the verification Flag associated with the username to true if token matches the token
               associated with a User with username in the set of Tokens
1. **Complete the definition of the concept state**. (above)
2. **Write a requires/effects specification for each of the two actions**. (above)
3. **What essential invariant must hold on the state? How is it preserved?**
   No pair of User in the set of Users has the same username. This is preserved by register, which before creating and returning a User "requires" that the username doesn't already exist in the usernames
   associated with the set of Users.
4. **One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality.** (above)

# Exercise 3: Comparing Concepts
**A concept specification for PersonalAccessToken and a succinct note about how it differs from PasswordAuthentication and how you
might change the GitHub documentation to explain this**
Note: PasswordAccessToken differes from PasswordAuthentication in their purposes, as the former is used to control access on the repository-level and the latter is used to register and authenticate a user
on a higher-level (above repository-level). The Github documentation can more clearly state that the PasswordAccessToken is for controlling the scope of access a token has **within** a user's github page.
- concept: PersonalAccessToken [Repository]
- purpose: allow for repository-level access control
- principle: given a set of created repositories, a user creates a token
           that gives it access to some subset of the repostories. Then,
           a third-party uses a created token to access those
           repositories.
- state:
     - a set of Tokens with:
          - a string name
          - a string token
     - a set of AccessRepositories with:
          - a Token token
          - a set of repositories
- actions:
     - create(name: string): token: Token
          - requires: name is unique among token names
          - effect: create and add a random, unique token to Tokens and AccessRepositories and the associated set of repositories are public repositories
     - addRepository(token: Token, repository: Repository)
          - requires: token exists
          - effect: add the respository to set of repositories associated with token token in set of AccessRepositories

# Exercise 4: Defining familiar Concepts

- concept: TrackBillableHours [Date]
- purpose: track billable hours
- principle: an employee marks the beginning of a session by selecting a project
             and entering a string describing the work to be done, and then marks
             the end of the session, the time between those actions and the project name are logged 
- state:
     - a set of Sessions with:
            - a string name
            - a Date date
            - a number time
            - a activate flag
     
- actions:
        - beginSession(projectName: string, todaysDate: Date): Session
            - effect: if projectName and todaysDate matches with one of the sessions' name or date then
                 modify the activate flag to True. Otherwise, create, return, add a new Session.
                 The time associated with the Session will be incremented second by second, in the background.
        - resetSession(session: Session)
            - requires: session exists
            - effect: sets the time associated with the session to 0 and activate flag to False
        - endSession(session: Session)
            - requires: session exists and session 's activate flag is True
            - effect: stops the time associated with session from incementing and sets the activate flag to False
          
Note: resetSession is used for when someone forgots to end a session. The two stakeholders are the company and the client being billed, resetSession will make the company pay the cost by not charging the client for an erroneous number of hours.

- concept: MakingElectronicBoardPasses [Time, Date, Gate]
- purpose: create and issue a digital version of a boarding pass that keeps track of
            a passanger's flight details
- principle: create a boarding pass by entering the departure and arrival gates at airports,
            the carrier code, and departure and arrival times for the flight. Then
            issue the boarding pass to a passenger. 
- state:
       - a set of Passes with:
            - a departure time Time and date Date
            - an arrival time Time and date Date
            - a Gate departureGate
            - a Gate arrivalGate
            - a string carrier
       - a set of Passengers with:
            - a string name
            - a set of Passes
            
- actions:
       - createPass(departureTime: Time, departureDate: Date, arrivalTime: Time, arrivalDate: Date, departureGate: Gate,
                      arrivalGate: Gate, carrier: String): Pass
            - effect: create and add a Pass with departure and arrival time and dates, a departure and arrival gate and
                      the flight carrier code representing by the string carrier
       - issuePass(name: String, pass: Pass): Passenger
            - effect: create and addd a new Passenger with teh given name and pass
Note: The Gate in "[Time, Date, Gate]" contains: a string gate, and a string airport. I didn't include actions that can modify an state because boarding passes are for viewing purposes; you can't cancel or change a flight from/using a boarding pass.

- concept: Time-Based-One-Time-PasswordAuth
- purpose: reduce the risk of malicious actors authenticating themselves
- principle: register multiple devices to send a TOTP to when logging into an application
             and when a user logs into some application, the authorized devices (not the current one being used)
             will receive the TOTP, and the user will enter it into the applicaiton login when prompted
- state:
  
- actions:
       - register(user: Usesr, device: Device, app: string)
       - login(username: string, password: string): User
       - logout(user: User)
       
     
     
   
