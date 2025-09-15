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

***

### Exercise 2: Extending a Familiar Concept

1.  **Complete the definition of the concept state.**
    The state should include a set of `Users` with `Username`, `Password`, `Email`, and a `verification Flag`. It should also include a set of `Tokens` with a `User` and a `Number` for the token.
2.  **Write a requires/effects specification for each of the two actions.**
    * `register (username: String, password: String, email: String): (user: User, token: Number)`
        * **requires**: `username` doesn't already exist in the `Usernames` associated with the set of `Users`.
        * **effect**: Create and return a new `User`, add the `User` and a `token` to the set of `Tokens`, and send the random number `token` to the `email`.
    * `authenticate (username: String, password: String): (user: User)`
        * **requires**: `username` exists in the set of `Users`' usernames and the `verification Flag` associated with that `username` is `True`.
        * **effect**: Return the `User` with the matching `username` and `password`. If the password doesn't match, return an `Error`.
    * `confirm (username: String, token: Number): (user: User)`
        * **requires**: The `username` is associated with some `User` in the set of `Users`.
        * **effect**: Change the `verification Flag` associated with the `username` to `true` if the `token` matches the `token` associated with the `User` in the set of `Tokens`.
3.  **What essential invariant must hold on the state? How is it preserved?**
    An essential invariant is that no two `Users` in the set can have the same `username`. This is preserved by the `register` action, which has a `requires` clause to ensure a `username` doesn't already exist before creating a new `User`.
4.  **One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality.**
    The concept has been extended to include this. The `register` action now sends a token to the user's email, and a new `confirm` action is added. The `authenticate` action's `requires` clause now checks if the `verification Flag` is `True`, ensuring a user cannot authenticate until their email is confirmed.

***

### Exercise 3: Comparing Concepts

A concept specification for **`PersonalAccessToken`** and a succinct note about how it differs from `PasswordAuthentication`.

* **concept**: `PersonalAccessToken [Repository]`
* **purpose**: Allow for repository-level access control.
* **principle**: Given a set of created repositories, a user creates a token that gives access to a subset of those repositories. A third-party then uses this token to access those repositories.
* **state**:
    * a set of `Tokens` with:
        * a string `name`
        * a string `token`
    * a set of `AccessRepositories` with:
        * a `Token` `token`
        * a set of `repositories`
* **actions**:
    * `create(name: string): token: Token`
        * **requires**: `name` is unique among token names.
        * **effect**: Create and add a random, unique token to `Tokens` and `AccessRepositories`. The associated set of repositories will be the public repositories.
    * `addRepository(token: Token, repository: Repository)`
        * **requires**: `token` exists.
        * **effect**: Add the `repository` to the set of repositories associated with the input `token` in `AccessRepositories`.

**Note:** `PasswordAuthentication` is used for user-level authentication, confirming a user's identity to grant general access. `PersonalAccessToken`, on the other hand, is for fine-grained, repository-level access control. It allows a user to grant a third party **limited scope** of access to their repositories without sharing their main account password.

To improve the GitHub documentation, it could more clearly state that a `PersonalAccessToken` is for controlling the **scope of access** a token has **within** a user's GitHub account. This distinction is crucial for security and access management.

***

### Exercise 4: Defining Familiar Concepts

#### Track Billable Hours

* **concept**: `TrackBillableHours [Date]`
* **purpose**: Track billable hours.
* **principle**: An employee marks the beginning of a session by selecting a project and describing the work. They then mark the end of the session, and the elapsed time and project name are logged.
* **state**:
    * a set of `Sessions` with:
        * a string `projectName`
        * a `Date` `date`
        * a number `time` (in seconds)
        * an `active` flag
* **actions**:
    * `beginSession(projectName: string, todaysDate: Date): Session`
        * **effect**: If a `Session` with `projectName` and `todaysDate` already exists, set its `active` flag to `True`. Otherwise, create, return, and add a new `Session` with `time` initialized to 0 and `active` set to `True`.
    * `resetSession(session: Session)`
        * **requires**: `session` exists.
        * **effect**: Set the `time` associated with the `session` to 0 and the `active` flag to `False`.
    * `endSession(session: Session)`
        * **requires**: `session` exists and its `active` flag is `True`.
        * **effect**: Set the `active` flag to `False`. The time associated with the session is not incremented further.

#### Making Electronic Boarding Passes

* **concept**: `MakingElectronicBoardingPasses [Time, Date, Gate]`
* **purpose**: Create and issue a digital boarding pass that tracks a passenger's flight details.
* **principle**: Create a boarding pass by entering flight details (departure/arrival gates, times, carrier code). Then, issue the pass to a passenger.
* **state**:
    * a set of `Passes` with:
        * a `Time` and `Date` for departure
        * a `Time` and `Date` for arrival
        * a `Gate` for departure and a `Gate` for arrival
        * a string `carrier`
    * a set of `Passengers` with:
        * a string `name`
        * a set of `Passes`
* **actions**:
    * `createPass(departureTime: Time, departureDate: Date, arrivalTime: Time, arrivalDate: Date, departureGate: Gate, arrivalGate: Gate, carrier: String): Pass`
        * **effect**: Create and add a new `Pass` with the provided flight details.
    * `issuePass(name: String, pass: Pass): Passenger`
        * **effect**: Create and add a new `Passenger` with the given `name` and `pass`.

#### Time-Based-One-Time-Password Authentication

* **concept**: `Time-Based-One-Time-PasswordAuth`
* **purpose**: Reduce the risk of malicious actors authenticating themselves.
* **principle**: Register multiple devices to receive a TOTP (Time-Based One-Time Password) when logging into an application. When a user logs in, the authorized devices (excluding the current one) receive the TOTP, which the user then enters to complete the login.
* **state**:
    * a set of `Users` with:
        * a string `username`
        * a string `password`
    * a set of `TwoAuthUsers`:
        * a `User` `user`
        * a set of `registeredDevices`
    * a set of `RegisteredDevices` with:
        * a string `device`
        * a set of `Apps`
    * a set of `Apps` with:
        * a string `name`
* **actions**:
    * `registerDeviceForApp(user: User, device: string, appName: string)`
        * **requires**: `user` exists.
        * **effect**: Create a new `TwoAuthUser` with the `User` and a `RegisteredDevice` with the specified `device` and `appName`.
    * `registerUser(username: String, password: String): user: User`
        * **requires**: `username` doesn't already exist.
        * **effect**: Create and return a new `User`.
    * `login(user: User, device: string): token: number`
        * **requires**: `user` exists and `device` is the name of the current device being used.
        * **effect**: If the user is in `TwoAuthUsers`, send a random token to the set of `registeredDevices`, not including the device currently being used for login.
