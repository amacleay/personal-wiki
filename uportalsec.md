[back](index)

# Universal portal security

The relationship between the universal portal web application and athenaNet

---

## Scope

The purpose of this document is to clarify the security design of the universal portal.

* [Overview of the universal portal](#/overview-of-the-universal-portal)
	* [What is it?](#/what-is-the-universal-portal-)
	* [Components](#/components)
	* [Users](#/what-is-a-user-) and [resources](#/what-is-a-resource-)
	* [Mapping to OAuth](#/mapping-to-oauth)
* [Web application security overview](#/web-application-security-overview)
	* [What is the problem?](#/what-is-the-problem-)
	* [What is the goal of the security design?](#/what-is-the-goal-of-the-security-design-)
* [Implementing athenaNet's authorization layer](#/implementing-athenanet-s-authorization-layer)
	* [Major choices](#/major-choices)
	* [As OAuth](#/as-oauth)
* [Web app session security](#/web-app-session-security)

---

# Overview of the universal portal

Introduction to components and roles

---

## What is the universal portal?

The universal portal is a new web application in the athenaNet product suite aimed at the general public.

* Be more patient focused: allow patients to see all their health information in one place
* Create a marketplace for healthcare, starting with the ability to search and schedule appointments across all of athenaNet
* Make baseline capabilities from our existing portal cross-context:
* Leverage what we can that already exists
* Not planning to roll this out at scale until 2017

---

## Components

The universal portal will consist of:
* A database
	* Login information
	* Mappings between a set of login credentials and bundles of patients and permissions in athenaNet
* Web server pool
	* https://my_health.athenahealth.com or something
	* Serves both unauthenticated workflows (shopping) and authenticated (patient portal)
* Job servers

---

## Not components

The universal portal will get most of its data on the fly from _external services_:
* athenaNet practices
* athenaNet Global Scheduling service
* athenaNet Provider Directory service
* athenaNet Master Patient Index service
* Eventually, third party services <!-- .element: class="fragment" data-fragment-index="2" -->
	* Epic?<!-- .element: class="fragment" data-fragment-index="2" -->
	* Cerner?<!-- .element: class="fragment" data-fragment-index="2" -->
	* etc<!-- .element: class="fragment" data-fragment-index="2" -->

---

## What is a user?

### Always ask the question: *a user of what?*

___

## What is a user of the universal portal?

To the universal portal, a user is a person who uses the universal portal.

They may be:

* Shopping for immediate care
	* *"I have a headache and this facebook link says I can find an appointment"*
* Shopping for a new PCP
	* *"I want a woman doctor with great reviews, and this facebook link says I can find her!"*
* Managing their health or healthcare
	* This is the traditional "patient portal experience" we serve today
	* ...but better<!-- .element: class="fragment" data-fragment-index="2" -->


---

## What is a user of athenaNet?

To athenaNet, the *universal portal is the user* requesting a resource.
```
$Global::session{USERNAME} eq 'UPORTAL'
```

Often but not always, the universal portal requests a resource *on behalf of a universal portal user*.

---

## What is a resource?

* Owned by universal portal user
	* Claim data
	* Chart data
	* Demographics
	* Vitals
* Other
	* Appointment slots
	* Department location
	* Messaging settings

---

## How is a resource defined?

A **resource** as currently conceived in athenaNet is *a class of data* to which permission may be granted to add, update, or remove.

```
# The code of today
ResourceSecure($Global::session{USERNAME}, 'MEDICATIONS')
	or confess "You can't mess with meds!";
```

When athenaNet is considering a request from the universal portal, a **resource** is either *a class or an instance of data* to which permission may be granted to add, update, or remove.

Access is granted for the pair `UNIVERSALPORTAL, uportal_user`

```
# The code of the future?
DelegatedResourceSecure({
	ATHENANET_USER => 'UPORTAL',
	DELEGATED_USER => $portal_user,
	RESOURCE => { NAME => 'CHARTMERGE', PATIENTIDS => [21,22] },
});
```

---

## Mapping to OAuth

| OAuth 2.0 concept    | Universal portal concept      |
| ----------           | -----------------             |
| Resource owner       | Universal portal user         |
| Client               | Universal portal application  |
| Resource server      | athenaNet                     |
| Authorization server | athenaNet (this is debatable) |

Missing from this list (and incidentally from the OAuth 2.0 specification) is the authentication mechanism.  <!-- .element: class="fragment" data-fragment-index="2" -->

---

### Questions?

---

# Web application security overview

Trust model between athenaNet and the universal portal

---

## What is the problem?

Vulnerabilities in the universal portal expose sensitive athenaNet PHI.

Without a security apparatus on athenaNet that protects patient resources, it is *hard* to prove that the universal portal doesn't leak PHI and *easy* to write code that exposes PHI.

---

## What is the goal of the security design?

> Let *A* be an attacker, and let *B* be a universal portal user with valid login credentials.  Assume that *A* has complete access to all universal portal code and the contents of all data stores used by the universal portal.

We would like to be able to prove the following:

> The universal portal cannot successfully retrieve a protected resource belonging to user *B* from athenaNet unless *A* can successfully present valid login credentials for *B*.

---

### Questions?

---

# Implementing athenaNet's authorization layer

For fun and profit

---

## Major choices

* Does athenaNet even implement the authorization layer, or is it written in the universal portal request code?
	* If it's not in athenaNet, only the universal portal benefits
	<!-- .element: class="fragment" data-fragment-index="2" -->
	* If it's in athenaNet, the definition of permissions and resources should be generic enough to be used by other applications
	<!-- .element: class="fragment" data-fragment-index="2" -->
* Is the universal portal an internal athenaNet service or an external partner?
	<!-- .element: class="fragment" data-fragment-index="3" -->
	* We could build our application to talk to athenaNet via published MDP APIs or subsystem APIs
	<!-- .element: class="fragment" data-fragment-index="4" -->
	* If an MDP partner, would we be able to use RabbitMQ/athenaWorker?
	<!-- .element: class="fragment" data-fragment-index="4" -->

We have a working theory (trusted service), but that's all.
<!-- .element: class="fragment" data-fragment-index="5" -->

---

## As OAuth

There are a couple of plausible implementation choices within the OAuth 2.0 specification.  We'll consider [three-legged OAuth with Resource Owner Credential authorization](http://tools.ietf.org/html/rfc6749#section-4.3)

* *Resource Owner* is a universal portal user
* *Client* is the universal portal web application
* *Authorization Server* is (probably) athenaNet: one access token requested per athenaNet practice

---

### Overview of OAuth implementation

```
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
```

---

### Overview of OAuth implementation

* Workflow:
	* Universal portal user lands on universal portal landing page and enters username and password
	* After authenticating a universal portal user, the universal portal [requests an access token](http://tools.ietf.org/html/rfc6749#section-4.4.2) from each athenaNet practice to which the user is mapped
		* With the [Resource Owner Credential authorization](http://tools.ietf.org/html/rfc6749#section-1.3.3) scheme, there is no *authorization token* step
	* With every request to athenaNet for PHI, the universal portal passes along the access token and athenaNet always validates that it is valid for the provided resource request

---

### Obtaining an access token

```
     +----------+
     | Resource |
     |  Owner   l
     |          |
     +----------+
          v
          |    Resource Owner
         (A) Password Credentials
          |
          v
     +---------+                                  +---------------+
     |         |>--(B)---- Resource Owner ------->|               |
     |         |         Password Credentials     | Authorization |
     | Client  |                                  |     Server    |
     |         |<--(C)---- Access Token ---------<|               |
     |         |    (w/ Optional Refresh Token)   |               |
     +---------+                                  +---------------+
```

Note that athenaNet is the Authorization Server in this instance.

---

### OAuth questions

* Is the Authorization Server a separate role for which we should create a new service?
	* Is it PingFederate?  Does that mean that all user mapping data would have to be stored in PingFed, and could it handle it?
	* Should authentication be handled by an authentication server?
* How does athenaNet establish the identity of the universal portal?
	* HTTP basic?  SSL certificate?  MAC?

---

# Web app session security

Secure sessions in the universal portal itself

---

## Storage and expiration

Most session data is stored in the database/memcache:

* unstructured session data, which will always include client IP or UA string or other request artifact that should remain the same between requests
* lastmodified (last time the session was committed to storage)

---

## Storage and expiration continued

The session ID and session expiration time live in a cookie.  The format of the cookie is `join('|', $session_id, $expiration_time, MAC($session_id . $expiration_time)` where `MAC` is a secure [MAC function](https://en.wikipedia.org/wiki/Message_authentication_code).  The reason for this is twofold:

- it prevents us from having to store expiration time in the DB, which helps us reduce hits to the DB
- it means the session ID alone is insufficient to recover an active portal user session, which means we can save session IDs in the clear in the database

Database sessions over a certain age will be cleaned up by a periodic cron job.  This will look at all sessions with a `lastmodified` timestamp over a certain age.

---

## Application behavior

- web application receives the session cookie, parses out the session ID and expiration time, and validates the MAC.
- the session is then retrieved from memcache or the database
- consistency validation: validate that IP and/or UA string or other stored request artifact match between the storage version and that from the current request.  If any mismatch, invalidate the session ID, log a suspicious activity event, and issue a new ID
- behavior validation: validate that current expiration time (from cookie) has not expired.  If it has, invalidate the session ID and issue a new ID.
- request proceeds and completes
- Check lastmodified.  If over threshold, force a rewrite of the session to storage (this is to prevent cleanup from wiping active sessions)
- Check if session has been written to, other than expiration time.  If so, force a rewrite of the session to storage
- Update the current expiration time cookie and write it out to the client

---

# Questions?

* Further reading:
	<!-- .element: class="fragment" data-fragment-index="2" -->
	* [Working session design doc](https://intranet.athenahealth.com/wiki/node.esp?ID=103787)
	<!-- .element: class="fragment" data-fragment-index="2" -->

