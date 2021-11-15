---
layout: blog
type: blog
title: Web Application Security - Authorization (Access Control)
permalink: blog/web-authorization
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - IDORs
  - access control
  - priv esc
---

## Types of Access Control
- Vertical 
- Horizontal
- Context-dependent

## Vertical Access Control
- Unprotected functionality
	- For example, administrative functions might be linked from an administrator's welcome page but not from a user's welcome page.
	- In some cases, sensitive functionality is not robustly protected but is concealed by giving it a less predictable URL: so called security by obscurity.
- Parameter-based access control methods
	- Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location, such as a hidden field, cookie, or preset query string parameter.
- Platform misconfiguration
	- Some applications enforce access controls at the platform layer by restricting access to specific URLs and HTTP methods based on the user's role.
	- Some application frameworks support various non-standard HTTP headers that can be used to override the URL in the original request, such as `X-Original-URL` and `X-Rewrite-URL`. 
	```
	POST / HTTP/1.1
	X-Original-URL: /admin/deleteUser
	... 
	```
	- If an attacker can use the GET (or another) method to perform actions on a restricted URL, then they can circumvent the access control that is implemented at the platform layer.

## Horizontal Privilege Escalation
- Horizontal privilege escalation arises when a user is able to gain access to resources belonging to another user, instead of their own resources of that type
- attacks may use similar types of exploit methods to vertical privilege escalation. For example, a user might ordinarily access their own account page using a URL like the following:
	```
	https://insecure-website.com/myaccount?id=123
	```
	Now, if an attacker modifies the id parameter value to that of another user, then the attacker might gain access to another user's account page, with associated data and functions. 
- In some applications, the exploitable parameter does not have a predictable value. For example, instead of an incrementing number, an application might use globally unique identifiers (GUIDs) to identify users. Here, an attacker might be unable to guess or predict the identifier for another user. However, the GUIDs belonging to other users might be disclosed elsewhere in the application where users are referenced, such as user messages or reviews. 
- In some cases, an application does detect when the user is not permitted to access the resource, and returns a redirect to the login page. However, the response containing the redirect might still include some sensitive data belonging to the targeted user, so the attack is still successful. 

## Horizontal to Vertical Privilege Escalation
Often, a horizontal privilege escalation attack can be turned into a vertical privilege escalation, by compromising a more privileged user.


## Insecure Direct Object References (IDORs)
Insecure direct object references (IDOR) are a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly. 
IDOR vulnerabilities are most commonly associated with horizontal privilege escalation, but they can also arise in relation to vertical privilege escalation. 
#idor

## Access control vulnerabilities in multi-step processes
Sometimes, a web site will implement rigorous access controls over some of these steps, but ignore others. For example, suppose access controls are correctly applied to the first and second steps, but not to the third step. Effectively, the web site assumes that a user will only reach step 3 if they have already completed the first steps, which are properly controlled. Here, an attacker can gain unauthorized access to the function by skipping the first two steps and directly submitting the request for the third step with the required parameters. 

## Referer-based access control
Some websites base access controls on the Referer header submitted in the HTTP request. The Referer header is generally added to requests by browsers to indicate the page from which a request was initiated.
