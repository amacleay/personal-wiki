[back](index)

# Universal portal security

The relationship between the universal portal web application and athenaNet

---

## Scope

The purpose of this document is to clarify the security design of the universal portal.  There are the following communications channels which need to be addressed:

* Browser to universal portal server
	* HTTP
* Universal portal to athenaNet
	* Primarily synchronous
	* Some asynchronous
* athenaNet to universal portal
	* Primarily asynchronous
	* Some synchronous

---

## Limit to scope

We will assume the following:
* The universal portal and athenaNet communicate over secure channels
* The universal portal and athenaNet have a mechanism for establishing trust about identiy

<section data-markdown="example.md" data-separator="^\n\n\n" data-separator-vertical="^\n\n" data-separator-notes="^Note:"></section>

# Title
## Sub-title

Here is some content...

Note:
This will only display in the notes window.

---

Issues for each:

* Transport layer (rabbit? http?)
* Trust model (basic? OAuth? SAML?)

![simple](https://devshare.athenahealth.com/~amacleay/vimwiki/uportal_simple_diagram.png)
