---
layout: blog
type: blog
title: Web Application Security - Authentication Vulnerabilities
permalink: blog/web-authentication
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - Passwords
  - mfa
---

# Introduction

## What is Authentication?
Authentication is the process of verifying the identity of a given user or client. In other words, it involves making sure that they really are who they claim to be. At least in part, websites are exposed to anyone who is connected to the internet by design. Therefore, robust authentication mechanisms are an integral aspect of effective web security.

There are three authentication factors into which different types of authentication can be categorized:

-   Something you **know**, such as a password or the answer to a security question. These are sometimes referred to as "knowledge factors".
-   Something you **have**, that is, a physical object like a mobile phone or security token. These are sometimes referred to as "possession factors".
-   Something you **are** or do, for example, your biometrics or patterns of behavior. These are sometimes referred to as "inherence factors".

Authentication mechanisms rely on a range of technologies to verify one or more of these factors.

## What is the difference between authentication and authorization?

Authentication is the process of verifying that a user really **is who they claim to be**, whereas authorization involves verifying whether a user **is allowed to do something**.

In the context of a website or web application, authentication determines whether someone attempting to access the site with the username `Carlos123` really is the same person who created the account.

Once `Carlos123` is authenticated, his permissions determine whether or not he is authorized, for example, to access personal information about other users or perform actions such as deleting another user's account.

## How do authentication vulnerabilities arise?

Broadly speaking, most vulnerabilities in authentication mechanisms arise in one of two ways:

-   The authentication mechanisms are weak because they fail to adequately protect against brute-force attacks.
-   Logic flaws or poor coding in the implementation allow the authentication mechanisms to be bypassed entirely by an attacker. This is sometimes referred to as "**broken authentication**".

In many areas of web development, [logic flaws](https://portswigger.net/web-security/logic-flaws) will simply cause the website to behave unexpectedly, which may or may not be a security issue. However, as authentication is so critical to security, the likelihood that flawed authentication logic exposes the website to security issues is clearly elevated.

## What is the impact of vulnerable authentication?

The impact of authentication vulnerabilities can be very severe. Once an attacker has either bypassed authentication or has brute-forced their way into another user's account, they have access to all the data and functionality that the compromised account has. If they are able to compromise a high-privileged account, such as a system administrator, they could take full control over the entire application and potentially gain access to internal infrastructure.

Even compromising a low-privileged account might still grant an attacker access to data that they otherwise shouldn't have, such as commercially sensitive business information. Even if the account does not have access to any sensitive data, it might still allow the attacker to access additional pages, which provide a further attack surface. Often, certain high-severity attacks will not be possible from publicly accessible pages, but they may be possible from an internal page.

# Password Based Authentication

## Username Enumeration
Note: turbo intruder is best for speed but lacks response time feedback. ffuf is also good if command line is being used

### Via Different Responses
Notes: check Status codes and Error messages very carefully
```shell
#https login error message bruteforce 
hydra -L auth-lab-usernames -p password acfb1faa1e3ad0298188614f00e500a6.web-security-academy.net https-post-form "/  
login:username=^USER^&password=^PASS^:Invalid username" -o username-enum-different-responses.txt

#format
hydra -L <username_list> -P <password_list> <host-ip-address> https-post-form "</directory>:<payload e.g. username=^USER^&password=^PASS^>:<error message>" -o <output_file>
```
#authentication_http-post-form

### Via Response Timing
- Test the timing of a correct username with a very long password
- Test the timing of a wrong username with a very long password
- if the difference in response time is reasonably long enough then the system may be vulnerable to a response timing bruteforce attack
- burpsuite intruder can be used to get response time

#authentication_response_timing

### IP-based Brute-force Protection
The following headers could be used to spoof your IP Address
```txt
1.  X-Forwarded-For : IP
2.  X-Forwarded-Host : IP
3.  X-Client-IP : IP
4.  X-Remote-IP : IP
5.  X-Remote-Addr : IP
6.  X-Host : IP
```
Change the IP whenever the Request gets Blocked Again.
Try adding Multiple headers sometimes can bypass a rate limit too.

[Bypassing rate limit abusing misconfiguration rules](https://infosecwriteups.com/bypassing-rate-limit-abusing-misconfiguration-rules-dcd38e4e1028)

#authentication_rate_limiting

## Flawed Brute-force Protection

### Blocking User's IP
In some implementations, the counter for the number of failed attempts resets if the IP owner logs in successfully. This means an attacker would simply have to log in to their own account every few attempts to prevent this limit from ever being reached.

In this case, merely including your own login credentials at regular intervals throughout the wordlist is enough to render this defense virtually useless.

### Account Locking
One way in which websites try to prevent brute-forcing is to lock the account if certain suspicious criteria are met, usually a set number of failed login attempts. Just as with normal login errors, responses from the server indicating that an account is locked can also help an attacker to enumerate usernames. 

Account locking also fails to protect against credential stuffing attacks. This involves using a massive dictionary of username:password pairs, composed of genuine login credentials stolen in data breaches.

Note: Burpsuite Cluster Bomb attack using null payload to repeat the request a certain number of times.

OR

```shell
ffuf -X POST -u https://ac1d1f611ec12ca180132d4f00d8006b.web-security-academy.net/login -d "username=W1&password=W2"  
-w auth-lab-usernames:W1,null_list:W2 -mode clusterbomb -fw 1145
```

#credential_stuffing 
#authentication_account_locking

### User Rate Limiting
In this case, making too many login requests within a short period of time causes your IP address to be blocked. Typically, the IP can only be unblocked in one of the following ways:

- Automatically after a certain period of time has elapsed
- Manually by an administrator
- Manually by the user after successfully completing a CAPTCHA

This method is generally safe from username enumeration but not password brute-force.

If the data is in json format, then multiple passwords could potentially be sent at once using the format

```txt
{
"username": "user"
"password": [
	"pass1",
	"pass2",
	...
]
}
```

## Re-registration
Assuming `"nullcell"` is already a registered user, try registering `" nullcell"` and logging in.

# Two-Factor Authentication

## Bypassing two-factor authentication
At times, the implementation of two-factor authentication is flawed to the point where it can be bypassed entirely.

If the user is first prompted to enter a password, and then prompted to enter a verification code on a separate page, the user is effectively in a "logged in" state before they have entered the verification code. In this case, it is worth testing to see if you can directly skip to "logged-in only" pages after completing the first authentication step. Occasionally, you will find that a website doesn't actually check whether or not you completed the second step before loading the page. 

## Flawed two-factor verification logic
Sometimes flawed logic in two-factor authentication means that after a user has completed the initial login step, the website doesn't adequately verify that the same user is completing the second step. 

- the user logs in with their normal credentials in the first step
- They are then assigned a cookie that relates to their account, before being taken to the second step of the login process
- When submitting the verification code, the request uses this cookie to determine which account the user is trying to access
- an attacker could log in using their own credentials but then change the value of the account cookie to any arbitrary username when submitting the verification code

Steps:
- Get the GET request that requests for a 2FA code and use burp repeater to request a code for the new user by editing the user selection parameter.
- Use burp intruder to bruteforce the mfa code and then open the redirect in a browser from bup.