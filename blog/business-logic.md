---
layout: blog
type: blog
title: Web Application Security - Business Logic
permalink: blog/web-business-logic
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - logic
---

## Excessive trust in client-side controls

Generally passing values in parameters that contain important distinguishing factors

## Failing to handle unconventional input

- check parameters involving intergers for negative
- check integer limit and web app's reaction to it
- check the character limit on parameters

## Making flawed assumptions about user behavior

### Not always supply mandatory input

 When probing for logic flaws, you should try removing each parameter in turn and observing what effect this has on the response. You should make sure to:

- Only remove one parameter at a time to ensure all relevant code paths are reached.
- Try deleting the name of the parameter as well as the value. The server will typically handle both cases differently.
- Follow multi-stage processes through to completion. Sometimes tampering with a parameter in one step will have an effect on another step further along in the workflow.

This applies to both URL and POST parameters, but don't forget to check the cookies too. This simple process can reveal some bizarre application behavior that may be exploitable. 

### Unintended sequence

To identify these kinds of flaws, you should use forced browsing to submit requests in an unintended sequence. For example, you might skip certain steps, access a single step more than once, return to earlier steps, and so on. Take note of how different steps are accessed. Although you often just submit a GET or POST request to a specific URL, sometimes you can access steps by submitting different sets of parameters to the same URL. As with all logic flaws, try to identify what assumptions the developers have made and where the attack surface lies. You can then look for ways of violating these assumptions. 

You could also just drop certain points along the sequence.

## Domain-specific flaws

To identify these vulnerabilities, you need to think carefully about what objectives an attacker might have and try to find different ways of achieving this using the provided functionality. This may require a certain level of domain-specific knowledge in order to understand what might be advantageous in a given context.

