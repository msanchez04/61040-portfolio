# Exercise 1

1. One invariant of the state related to the aggregation/counts of items is the idea that for every new Request, the count Number must be equivalent to the original request count minus the number of purchases for that item. This ensures the right count gets updated without allowing more purchases than the requested count for an item to occur. Another invaraint related to the requests and purchases is that you can't purchase an item that does not exist (have a request with a count) so that every purchase has at least 1 matching request. The invariant related to the aggregation/counts of items is more important since it ensures gift givers can not only gift gifts that were requested, but also don't buy more than the requested amount meaning the recipient will not have more of an item than originally requested. The action most affected is the purchase action since the purchase count number has to be less than or equal to the requested count and once a purchase is successful, the matching request count goes down ensuring that the recipient will not end up with more of the item than they originally intended.

2. An action that potentially breaks this invariant is the removeItem action since it removes the request for the item meaning the counts are no longer representative and there isn't the original matching request even though the item may have already been purchased. This problem could be fixed if there is an additional requirement that only allows you to remove an item if there are no purchases for that request, that way there aren't any purchased gifts for that item that will cause a conflict with the count.

3. Since the only condition for a registry to be closed/opened is that to close a registry has to be open and vice versa, then it is possible for a registry to be open and closed repeatedly. The reason this is allowed is that it allows for the recipient to evaluate their wishlist between certain time periods/events. They might want to re-evaluate what they might want to add or remove from the list so allowing the option to repeatedly open/close allows for this.

4. In practice this wouldn't matter since closing a registry is essentially ending it until it is ever opened again. It would also allow for tracking/record-keeping of the gifts purchased.

5. A registry owner's query would most likely be related to which items from the registry were pruchased and by which person. A gift giver's query might be related to what items still need to be purchased.

6. In order to preserve the element of surprise, the concept could be updated to have a hidePurchase flag in the registry that would be set to true if a user doesn't want to see the purchased gifts. In that case the recipient cannot query information about purchases until the registry is closed.

**state**
a set of Registrys with

an owner User

an active Flag

a set of Requests

a hidePurchase Flag

**actions**

hide (registry: Registry, hidePurchase: Flag)

**Requires** registry exists and hidePurchase is False

**Effects** hides purchases in the registry

show (registry: Registry, hidePurchase: Flag)

**Requires** registry exists and hidePurchase is True

**Effects** reveals purchases in the registry

7. SKU codes might be preferable for representing items specifically with regards to representation independence since you are able to find the same product from different sources and no constrained to a single store for the purchase. In that case it allows more flexibility for the gift giver to complete the purchase. By avoiding names, descriptions, prices, etc. the gift giver loses a majority of bias with regards to which gift is more expensive/cheaper and ultimately provides a registry of items that are desired by the recipient with no particular preference.

# Exercise 2

**concept** PasswordAuthentication

**purpose** limit access to known users

**principle** after a user registers with a username and a password,
they can authenticate with that same username and password
and be treated each time as the same user

**state**

a set of Users with

a username String

a password String

**actions**

register (username: String, password: String): (user: User)

**requires** username does not already exist

**effects** creates and registers a user with username and password

authenticate (username: String, password: String): (user: User)

**requires** username already exists with matching password

**effects** successful if returns user

3. The essential invariant that must hold on the state is that there can only be one user per username meaning there exists a unique username per user which maps to only one password. This invariant is preserved since register requires a username to not already exist and authenticate verifies that the username maps to the one unique password without mutating the state.

**concept** PasswordAuthentication

**purpose** limit access to known users

**principle** after a user registers with a username and a password,
they can authenticate with that same username and password
and be treated each time as the same user

**state**

a set of Users with

a username String

a password String

a confirmed Flag

<br />

a set of PendingUsers with

a username String

a password String

a token String

**actions**

register (username: String, password: String): (user: User, token: String)

**requires** username does not already exist and there are no pending registrations with the username

**effects** creates a new pending user with username, password, and token. Returns a user with a temporary user id and token that is emailed.

authenticate (username: String, password: String): (user: User)

**requires** username already exists with matching password

**effects** successful if returns user

confirm (username: String, token: String): (user: User)

**requires** a pending registration exists for username with given token

**effects** removes the pending registration. creates and returns a new confirmed user with username and password

# Exercise 3

**concept** PersonalAccessToken

**purpose** allow users to authenticate with automated access for more control other than their password

**principle** Users can generate one or more tokens and each token has permissions and can be revoked independently. Authenticating a token requires the system checks that the token exists, permimssions cover access, and matches the account.

**state**

a set of Users with

a username String

a password String

<br />

a set of Tokens with

an owner User

a token String

a permissions Set

a revoked Flag

**actions**

generateToken (owner: User, scopes: Set): (token: Token)

**requires** owner exists

**effects** creates and returns a new Token for this owner with the given permissions and revoked flag set to false

authenticateWithToken (username: String, tokenString: String, requestedPermissions: Set): (user: User)

**requires** some Token t with matching owner username, tokenString, and revoked flag set to false. The requested permissions must be under the user's alloted permissions.

**effects** returns the corresponding User

revokeToken (owner: User, token: Token)

**requires** token exists with matching token owner and revoked flag set to false

**effects** sets revoked flag equal to true

The key differences between PasswordAuthentication and PersonalAccessTokenAuthentication is that PersonalAccessTokenAuthentication allows a user to have multiple independent tokens rather than one passwords, and each token can have specific/limited permissions compared to default permissions with one password per user. Each token can also be individually revoked limiting access to full account as a password does not do.

Github could make their explanation clearer by not comparing a token to a password at the start. It leaves the reader confused and less open-minded about the distinction between both. Instead they could describe this seperate credential as a way to access different sets of permissions per account without immediately exposing the entire account's information like a password would. Additionally, for clarity, a table comparing and constrasting the use cases and distinctions between tokens and passwords would be more comprehensible. Explaining that tokens would grant similar access as a password without exposing all the information of the account makes it clearer to the user the purpose of PersonalAccessTokenAuthentication.

# Exercise 4

**concept** URLShortener [User]

**purpose** provide shortened links that redirect to longer URLs, supporting both auto-generated and user-defined suffixes

**principle** a user can create a mapping from a short suffix to a long URL and when someone accesses the short link, the system redirects them to the long URL

**state**

a set of Links with

a suffix String

a targetURL String

a creator User

**actions**

shorten (creator: User, targetURL: String): (link: Link)

**requires** no existing link with the same suffix

**effects** creates a new link with a shorter suffix pointing to targetURL

shortenWithSuffix (creator: User, targetURL: String, suffix: String): (link: Link)

**requires** no existing link with the same suffix

**effects** creates a new link with the given suffix pointing to targetURL

convert (suffix: String): (targetURL: String)

**requires** a link with this suffix exists

**effects** return its targetURL

<br />

**concept** BillableHours [User, Project]

**purpose** record billable sessions of work time on projects

**principle** an employee begins a session with a project and description, and ends it when work is done. The system computes duration, even if the employee forgets to end it which is automatically closed after maximum time reached

**state**
a set of Sessions with

an employee User

a project Project

a description String

a startTime DateTime

an endTime DateTime?

a closed Flag

**actions**

startSession (employee: User, project: Project, description: String, startTime: DateTime): (session: Session)

**requires** employee has no open session already

**effects** create a new session with startTime, endTime unset, and closed flag equal to false

endSession (employee: User, endTime: DateTime)

**requires** employee has an open session

**effects** set that session’s endTime to user defined endTime and closed flag equal to true

autoClose (session: Session, endTime: DateTime)

**requires** session exists and not closed and the maxDuration is less than or equal to the endTime-startTime

**effects** set endTime and closed flag equal to true

<br />

**concept** RoomBooking [User, Room]

**purpose** allow users to reserve rooms at particular times while ensuring no double-booking

**principle** a user can create a booking for a room at a specified time interval. The booking guarantees use of that room for the given interval and bookings may be canceled but do not overlap

**state**

a set of Bookings with

a room Room

a user User

a startTime DateTime

an endTime DateTime

**actions**

book (user: User, room: Room, startTime: DateTime, endTime: DateTime): (booking: Booking)

**requires** no existing booking for this room overlapping in the interval [startTime, endTime)

**effects** create and return a new booking

cancel (booking: Booking)

**requires** booking exists

**effects** remove the booking for the interval

### Notes:

- URLShortener concept does not require a user, but included for link editing purposes if a user wants to edit the link they generated in the future
- maxDuration not defined in state since doesn't change per user
