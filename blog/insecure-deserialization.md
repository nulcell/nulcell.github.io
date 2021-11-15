---
layout: blog
type: blog
title: Web Application Security - Insecure Deserialization
permalink: blog/web-insecure-deserialization
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - cookies
  - php
  - java
  - python
---

## Serialization and Deserialization

 __*Serialization*__ is the process of converting complex data structures, such as objects and their fields, into a "flatter" format that can be sent and received as a sequential stream of bytes. Serializing data makes it much simpler to:

- Write complex data to inter-process memory, a file, or a database
- Send complex data, for example, over a network, between different components of an application, or in an API call

__*Deserialization*__ is the process of restoring this byte stream to a fully functional replica of the original object, in the exact state as when it was serialized. The website's logic can then interact with this deserialized object, just like it would with any other object. 

__Note:__ When working with different programming languages, serialization may be referred to as marshalling (Ruby) or pickling (Python). These terms are synonymous with "*serialization*" in this context. 

## Insecure Deserialization

Insecure deserialization is when user-controllable data is deserialized by a website. This potentially enables an attacker to manipulate serialized objects in order to pass harmful data into the application code.

## How to Identify

During auditing, you should look at all data being passed into the website and try to identify anything that looks like serialized data. Serialized data can be identified relatively easily if you know the format that different languages use. Once you identify serialized data, you can test whether you are able to control it. 

## Serialization Formats

### PHP

 PHP uses a mostly human-readable string format, with letters representing the data type and numbers representing the length of each entry. For example, consider a User object with the attributes:
 
```txt
$user->name = "carlos";
$user->isLoggedIn = true;
```

When serialized, this object may look something like this:

```txt
O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
```


This can be interpreted as follows:

- O:4:"User" - An object with the 4-character class name "User"
- 2 - the object has 2 attributes
- s:4:"name" - The key of the first attribute is the 4-character string "name"
- s:6:"carlos" - The value of the first attribute is the 6-character string "carlos"
- s:10:"isLoggedIn" - The key of the second attribute is the 10-character string "isLoggedIn"
- b:1 - The value of the second attribute is the boolean value true

The native methods for PHP serialization are `serialize()` and `unserialize()`. If you have source code access, you should start by looking for `unserialize()` anywhere in the code and investigating further. 

### Java

 Some languages, such as Java, use binary serialization formats. This is more difficult to read, but you can still identify serialized data if you know how to recognize a few tell-tale signs. For example, serialized Java objects always begin with the same bytes, which are encoded as `ac ed` in hexadecimal and `rO0` in Base64.

Any class that implements the interface `java.io.Serializable` can be serialized and deserialized. If you have source code access, take note of any code that uses the `readObject()` method, which is used to read and deserialize data from an InputStream. 

### Python


## Manipulating Serialized Data

There are two approaches you can take when manipulating serialized objects. 

- Edit the object directly in its byte stream form
- Write a short script in the corresponding language to create and serialize the new object yourself.

The latter approach is often easier when working with binary serialization formats.

### Modifying object attributes

As a simple example, consider a website that uses a serialized `User` object to store data about a user's session in a cookie. If an attacker spotted this serialized object in an HTTP request, they might decode it to find the following byte stream:

`O:4:"User":2:{s:8:"username";s:6:"carlos";s:7:"isAdmin";b:0;}`

The `isAdmin` attribute is an obvious point of interest. An attacker could simply change the boolean value of the attribute to `1` (true), re-encode the object, and overwrite their current cookie with this modified value. In isolation, this has no effect. However, let's say the website uses this cookie to check whether the current user has access to certain administrative functionality:

```php
$user = unserialize($_COOKIE);  
if ($user->isAdmin === true) {  
// allow access to admin interface  
}
```

### Modifying data types

PHP-based logic is particularly vulnerable to this kind of manipulation due to the behavior of its loose comparison operator (`==`) when comparing different data types. For example, if you perform a loose comparison between an integer and a string, PHP will attempt to convert the string to an integer, meaning that `5 == "5"` evaluates to `true`.

Unusually, this also works for any alphanumeric string that starts with a number. In this case, PHP will effectively convert the entire string to an integer value based on the initial number. The rest of the string is ignored completely. Therefore, `5 == "5 of something"` is in practice treated as `5 == 5`.

This becomes even stranger when comparing a string the integer `0`:

`0 == "Example string" // true`

Why? Because there is no number, that is, 0 numerals in the string. PHP treats this entire string as the integer `0`.

Consider a case where this loose comparison operator is used in conjunction with user-controllable data from a deserialized object. This could potentially result in dangerous logic flaws #business_logic 

```php
$login = unserialize($_COOKIE)  
if ($login['password'] == $password) {  
// log in successfully  
}
```

Let's say an attacker modified the password attribute so that it contained the integer `0` instead of the expected string. As long as the stored password does not start with a number, the condition would always return `true`, enabling an authentication bypass. Note that this is only possible because deserialization preserves the data type. If the code fetched the password from the request directly, the `0` would be converted to a string and the condition would evaluate to `false`.

Be aware that when modifying data types in any serialized object format, it is important to remember to update any type labels and length indicators in the serialized data too. Otherwise, the serialized object will be corrupted and will not be deserialized.

## Using application functionality

As well as simply checking attribute values, a website's functionality might also perform dangerous operations on data from a deserialized object.

For example, as part of a website's "Delete user" functionality, the user's profile picture is deleted by accessing the file path in the `$user->image_location` attribute. If this `$user` was created from a serialized object, an attacker could exploit this by passing in a modified object with the `image_location` set to an arbitrary file path. Deleting their own user account would then delete this arbitrary file as well.

### Magic methods

Magic methods are a special subset of methods that you do not have to explicitly invoke. Instead, they are invoked automatically whenever a particular event or scenario occurs.

One of the most common examples in PHP is `__construct()`, which is invoked whenever an object of the class is instantiated, similar to Python's `__init__`.

Most importantly in this context, some languages have magic methods that are invoked automatically **during** the deserialization process. For example, PHP's `unserialize()` method looks for and invokes an object's `__wakeup()` magic method.

In Java deserialization, the same applies to the `readObject()` method, which essentially acts like a constructor for "re-initializing" a serialized object. The `ObjectInputStream.readObject()` method is used to read data from the initial byte stream. However, serializable classes can also declare their own `readObject()` methods as follows:

```java
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {...};
```

## Injecting Arbitrary Objects

 Deserialization methods do not typically check what they are deserializing. This means that you can pass in objects of any serializable class that is available to the website, and the object will be deserialized. This effectively allows an attacker to create instances of arbitrary classes. The fact that this object is not of the expected class does not matter. The unexpected object type might cause an exception in the application logic, but the malicious object will already be instantiated by then.

If an attacker has access to the source code, they can study all of the available classes in detail. To construct a simple exploit, they would look for classes containing deserialization magic methods, then check whether any of them perform dangerous operations on controllable data. The attacker can then pass in a serialized object of this class to use its magic method for an exploit. 

Classes containing these deserialization magic methods can also be used to initiate more complex attacks involving a long series of method invocations, known as a "gadget chain".

For PHP files, you can sometimes read source code by appending a tilde (~) to a filename to retrieve an editor-generated backup file.

```txt
Exploit: 
Generate a serialized object of a vulnerable class to exploit their magic methods (constructors and destructors using interesting functions).
```

## Gadget Chain

A "gadget" is a snippet of code that exists in the application that can help an attacker to achieve a particular goal. An individual gadget may not directly do anything harmful with user input. However, the attacker's goal might simply be to invoke a method that will pass their input into another gadget. By chaining multiple gadgets together in this way, an attacker can potentially pass their input into a dangerous "sink gadget", where it can cause maximum damage. 

 It is important to understand that, unlike some other types of exploit, a gadget chain is not a payload of chained methods constructed by the attacker. All of the code already exists on the website. The only thing the attacker controls is the data that is passed into the gadget chain. This is typically done using a magic method that is invoked during deserialization, sometimes known as a "kick-off gadget".

In the wild, many insecure deserialization vulnerabilities will only be exploitable through the use of gadget chains. This can sometimes be a simple one or two-step chain, but constructing high-severity attacks will likely require a more elaborate sequence of object instantiations and method invocations. Therefore, being able to construct gadget chains is one of the key aspects of successfully exploiting insecure deserialization. 

### Pre-built gadget chains

 Manually identifying gadget chains can be a fairly arduous process, and is almost impossible without source code access. Fortunately, there are a few options for working with pre-built gadget chains that you can try first.

There are several tools available that provide a range of pre-discovered chains that have been successfully exploited on other websites. Even if you don't have access to the source code, you can use these tools to both identify and exploit insecure deserialization vulnerabilities with relatively little effort. This approach is made possible due to the widespread use of libraries that contain exploitable gadget chains. For example, if a gadget chain in Java's Apache Commons Collections library can be exploited on one website, any other website that implements this library may also be exploitable using the same chain. 

#### Java [ysoserial](https://github.com/frohoff/ysoserial) 

One such tool for Java deserialization is "ysoserial". This lets you choose one of the provided gadget chains for a library that you think the target application is using, then pass in a command that you want to execute. It then creates an appropriate serialized object based on the selected chain. This still involves a certain amount of trial and error, but it is considerably less labor-intensive than constructing your own gadget chains manually. 

Note: Install dependencies first

```shell
java -jar ~/tools/ysoserial.jar CommonsCollections4 "rm /home/carlos/morale.txt" 2>/dev/null | base64
```

#### PHP Generic Gadget Chains [PHPGGC](https://github.com/ambionics/phpggc)

Most languages that frequently suffer from insecure deserialization vulnerabilities have equivalent proof-of-concept tools. For example, for PHP-based sites you can use "PHP Generic Gadget Chains" (PHPGGC)

```shell
./phpggc Symfony/RCE4 exec "rm /home/carlos/morale.txt" | base64 -w 0
```

### Documented gadget chains

It's always worth taking a look online to see if there are any documented exploits that you can adapt to attack your target website. Even if there is no dedicated tool for automatically generating the serialized object, you can still find documented gadget chains for popular frameworks and adapt them manually. 

If you can't find a gadget chain that's ready to use, you can still gain valuable knowledge, which you can use to create your own custom exploit. 

Major languages with deserialization exploits are PHP, Java, Python and Ruby

