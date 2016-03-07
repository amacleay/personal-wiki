[back](index)

# Universal portal security

The relationship between the universal portal web application and athenaNet

---

## Scope

The purpose of this document is to clarify the security design of the universal portal.

* Overview of the universal portal
	* What is it?
	* The components
	* Outline of communications channels
	* Users and resources
	* Overview of communication types
* Web application security overview
	* What is the problem?
	* Definition of _resource_
	* A proposition
	* Relationship with existing protocols

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

First question: *a user of what?*

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

## What is a user?

To athenaNet, the *universal portal is the user* requesting a resource.  The universal portal requests a resource *on behalf of a universal portal user*.

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
ResourceSecure($Global::session{USERNAME}, 'CHARTMERGE')
	or confess "You can't merge charts!";
```

When athenaNet is considering a request from the universal portal, 
a **resource** is either *a class or an instance of data* to which permission may be granted to add, update, or remove.

```
# Requesting a specific resource on behalf of a user
ExternalResourceSecure('UPORTAL', $portal_user, $requested_args)
	or confess "You can't merge that person's chart!";
# Requesting a class of behavior
ExternalResourceSecure('UPORTAL', $portal_user, 'PROVIDERSEARCH')
	or confess "You can't merge charts, silly universal portal";
```

---

## Communications channels

* Web application
	* User's browser to the universal portal server
	* Universal portal server to _external services_
		* Right now, this is code for athenaNet web services <!-- .element: class="fragment" data-fragment-index="2" -->
* Offline communications
	* athenaNet practices to universal portal

---

Issues for each:

* Transport layer (rabbit? http?)
* Trust model (basic? OAuth? SAML?)

![simple](https://devshare.athenahealth.com/~amacleay/vimwiki/uportal_simple_diagram.png)
