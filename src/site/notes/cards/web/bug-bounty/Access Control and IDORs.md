---
{"dg-publish":true,"permalink":"/cards/web/bug-bounty/access-control-and-ido-rs/"}
---

[[+/Bug Bounty\|Bug Bounty]]

Restricting access to specific resources and referencing sensitive objects using user controlled inputs such as IDs or username such as:

- As a parameter `?=12345`
- At the end of a URL `/aqz-KE-bpKQ`
- Hidden POST URLs
- Hidden Field names.
- APIs
	- /pokemons - list all
	- /pokemons/:id - show specific pokemon

#### UUID = No Bug?
---
It is basically a bunch of random characters thrown in together to make it unpredictable.

As long as you can find some IDs it is still technically vulnerable.

### Testing Questions
---
1. Can we still access the resource even if we are logged out? delete cookies.
2. Can we log in and then access other objects owned by another user?
3. Can we as a regular user use/see some admin functionality?
4. _Cross-Tenant_ access another tenant's resource while using another tenant account.

![Access Control and IDORs.png](/img/user/cards/web/images/Access%20Control%20and%20IDORs.png)
#### Finding Acces Control Issues
---
1. Create an account then log in
2. Perform a bunch of request
3. Go to repeater and remove the cookie
	- Effectively treating the request as if they came from no particular user
4. Check what happen to our account. 

##### Accessing another user's resources
---
1. Create an account as Account A.
2. Perform bunch of request.
3. Create an account as Account B
4. Go to repeater and change A's cookie to the new cookie (B's)
5. Check account A

**Note:** If the actions performed on A happens to B instead it is not a **VULNERABILITY** since what we did is an equivalent of logging using the login form.

##### Admin function as a regular user
---
1. Create admin account: A
2. Perform a bunch of admin functions
3. Create a normal account: B
4. Go to repeater and change A's cookie with B's cookie
5. Check and see if they caused an admin action to happen