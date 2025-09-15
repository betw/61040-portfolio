### Exercise 1: Reading a Concept

1.  **Invariants**.
    One invariant is that the counts of items associated with a `Request` are greater than or equal to 0. Another is that every `Purchase` is for the `Item` associated with a `Request`. The second is more important because the purpose is to "track purchases of requested gifts," and if a `Purchase` is for an `Item` not in the `Request`, that purpose is broken. The `purchase` action preserves this invariant because its precondition requires the registry to have a request for the item and its effect creates a new purchase specifically for that item.
2.  **Fixing an action**.
    The `purchase` action can break the invariant that item counts are non-negative. This happens if the input `count` is negative. Although the precondition requires that the registry contains a request "with at least count," if the existing request has a count of 0, and the input `count` is a negative number (e.g., -1), the effect will decrement the count to a negative value. To fix this, a new precondition should be added to the `purchase` action: `count` must be greater than or equal to 1.
3.  **Inferring behavior**.
    A registry can be repeatedly opened and closed. The `open` action requires the registry to be inactive, which is the post-condition of the `close` action. Conversely, the `close` action requires the registry to be active, which is a post-condition of the `open` action. This allows for a user to temporarily close a registry, perhaps when they don't want people to buy gifts for them, and then open it again later.
4.  **Registry deletion**.
    It wouldn't matter in practice that there is no action to delete a registry. Making a registry inactive effectively "deletes" it for everyone except the owner, as no one else can view it or make purchases.
5.  **Queries**.
    Two likely queries are:
    * The registry owner would want to see the set of `Purchases` associated with a `Request`.
    * A gift giver would want to see the set of `Requests` in a registry.
6.  **Hiding purchases**.
    To support this, a `visibility Flag` could be added to a `Request`. When this flag is `True`, the recipient can see the purchases; when it is `False`, they cannot. Then, two new actions, `makePurchasesVisible` and `makePurchasesHidden`, could be added. These would take a `Request` as input and mirror the `open`/`close` actions by toggling the `visibility Flag`.
7.  **Generic types**.
    Using generic types like `User` and `Item` identified by a unique code is preferable because it's more generalizable. A unique code (like an SKU) can identify many different types of users and items. Representing items by their names, descriptions, or prices is problematic because these attributes can change over time, whereas a unique ID remains constant, preventing the need for additional changes to the concept.

---

### Exercise 2: Extending a Familiar Concept

* **concept**: PasswordAuthentication [User]
* **purpose**: limit access to known users
* **principle**: after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user
* **state**:
    * a set of Users with:
        * a string Username
        * a string Password
        * a string Email
        * a verification Flag
    * a set of Tokens with:
        * a User
        * a number token
* **actions**:
    * `register (username: String, password: String, email: String): (user: User, token: Number)`
        * **requires**: username doesn't already exist in the usernames associated with the set of Users
        * **effect**: create and return a User, add the User and token to set of Tokens, and send the random number token to email
    * `authenticate (username: String, password: String): (user: User)`
        * **requires**: username exists in the set of Users' usernames and verification Flag associated with username is True
        * **effect**: return the User with the matching username and password, Error if the password doesn't match the password associated with the username
    * `confirm (username: String, token: Number): (user: User)`
        * **requires**: the username is associated with some user in the set of Users
        * **effect**: change the verification Flag associated with the username to true if token matches the token associated with a User with username in the set of Tokens

1.  **Complete the definition of the concept state**. (above)
2.  **Write a requires/effects specification for each of the two actions**. (above)
3.  **What essential invariant must hold on the state? How is it preserved?**
    No pair of User in the set of Users has the same username. This is preserved by register, which before creating and returning a User "requires" that the username doesn't already exist in the usernames associated with the set of Users.
4.  **One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality**. (above)

---

### Exercise 3: Comparing Concepts

**A concept specification for PersonalAccessToken and a succinct note about how it differs from PasswordAuthentication and how you might change the GitHub documentation to explain this**
**Note**: PasswordAccessToken differs from PasswordAuthentication in their purposes, as the former is used to control access on the repository-level and the latter is used to register and authenticate a user on a higher-level (above repository-level). The Github documentation can more clearly state that the PasswordAccessToken is for controlling the scope of access a token has **within** a user's github page.
* **concept**: PersonalAccessToken [Repository]
* **purpose**: allow for repository-level access control
* **principle**: given a set of created repositories, a user creates a token that gives it access to some subset of the repositories. Then, a third-party uses a created token to access those repositories.
* **state**:
    * a set of Tokens with:
        * a string name
        * a string token
    * a set of AccessRepositories with:
        * a Token token
        * a set of repositories
* **actions**:
    * `create(name: string): token: Token`
        * **requires**: name is unique among token names
        * **effect**: create and add a random, unique token to Tokens and AccessRepositories and the associated set of repositories are public repositories
    * `addRepository(token: Token, repository: Repository)`
        * **requires**: token exists
        * **effect**: add the repository to set of repositories associated with token token in set of AccessRepositories

---

### Exercise 4: Defining Familiar Concepts

* **concept**: TrackBillableHours [Date]
* **purpose**: track billable hours
* **principle**: an employee marks the beginning of a session by selecting a project and entering a string describing the work to be done, and then marks the end of the session, the time between those actions and the project name are logged
* **state**:
    * a set of Sessions with:
        * a string name
        * a Date date
        * a number time
        * a activate flag
* **actions**:
    * `beginSession(projectName: string, todaysDate: Date): Session`
        * **effect**: if projectName and todaysDate matches with one of the sessions' name or date then modify the activate flag to True. Otherwise, create, return, add a new Session. The time associated with the Session will be incremented second by second, in the background.
    * `resetSession(session: Session)`
        * **requires**: session exists
        * **effect**: sets the time associated with the session to 0 and activate flag to False
    * `endSession(session: Session)`
        * **requires**: session exists and session 's activate flag is True
        * **effect**: stops the time associated with session from incrementing and sets the activate flag to False

**Note**: resetSession is used for when someone forgets to end a session. The two stakeholders are the company and the client being billed, resetSession will make the company pay the cost by not charging the client for an erroneous number of hours.

---

* **concept**: MakingElectronicBoardPasses [Time, Date, Gate]
* **purpose**: create and issue a digital version of a boarding pass that keeps track of a passenger's flight details
* **principle**: create a boarding pass by entering the departure and arrival gates at airports, the carrier code, and departure and arrival times for the flight. Then issue the boarding pass to a passenger.
* **state**:
    * a set of Passes with:
        * a departure time Time and date Date
        * an arrival time Time and date Date
        * a Gate departureGate
        * a Gate arrivalGate
        * a string carrier
    * a set of Passengers with:
        * a string name
        * a set of Passes
* **actions**:
    * `createPass(departureTime: Time, departureDate: Date, arrivalTime: Time, arrivalDate: Date, departureGate: Gate, arrivalGate: Gate, carrier: String): Pass`
        * **effect**: create and add a Pass with departure and arrival time and dates, a departure and arrival gate and the flight carrier code representing by the string carrier
    * `issuePass(name: String, pass: Pass): Passenger`
        * **effect**: create and add a new Passenger with the given name and pass

**Note**: The Gate in "[Time, Date, Gate]" contains: a string gate, and a string airport. I didn't include actions that can modify a state because boarding passes are for viewing purposes; you can't cancel or change a flight from/using a boarding pass.

---

* **concept**: Time-Based-One-Time-PasswordAuth
* **purpose**: reduce the risk of malicious actors authenticating themselves
* **principle**: register multiple devices to send a TOTP to when logging into an application and when a user logs into some application, the authorized devices (not the current one being used) will receive the TOTP, and the user will enter it into the application login when prompted
* **state**:
    * a set of Users with:
        * a string username
        * a string password
    * a set of TwoAuthUsers:
        * a User user
        * a set of registeredDevices
    * a set of RegisteredDevices with:
        * a string device
        * a set of Apps
    * a set of Apps with:
        * a string name
* **actions**:
    * `registerDeviceForApp(user: User, device: string, appName: string)`
        * **requires**: user exists
        * **effect**: create a new TwoAuthUser with User user and registeredDevice with device and appName
    * `registerUser(username: String, password: String): user: User`
        * **requires**: username doesn't already exist in the usernames associated with the set of Users
        * **effect**: create and return a User
    * `login(username: string, password: string, device: string): token: number`
        * **requires**: a User exists with username username and password Password and device is the name of the current device being used
        * **effect**: if the user is a User in TwoAuthUsers then send a random token to the set of registeredDevices, not including the device being used to login

**Note**: If a malicious actor gains access to the username and password and one of the devices in registeredDevices, then this method fails to protect the user. It only serves as a another layer of protection, but DOES not eliminate the risk described.
