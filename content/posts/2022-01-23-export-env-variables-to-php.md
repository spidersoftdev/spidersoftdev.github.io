---
title: How to pass ENV variables to PHP
author: admin
type: post
date: 2022-01-23T08:30:00+00:00
url: /2021/export-env-variables-to-php
lead: How to export variables to PHP in secure way
categories:
- Technologies
- PHP
tags:
- env

---
It's not a good practice to store your password in the code. It is a well-known fact, and on multiple occasions, people get hacked because they share their code (and passwords) accidentally with others.

Many bots are just browsing GitHub for lost `AWS_SECRET_ACCESS_KEY` or `GCE` credentials. I'm going to show you how to pass your `AWS_SECRET_ACCESS_KEY` to PHP.

<!--more-->

PHP's `getenv()` function is not secure. It's not possible to read variables to PHP from the outside. So, if you want to pass variables to PHP from the outside, you have to use `putenv()` function. Which doesn't make much sense - or you can just set them up in your PHP configuration.

The other solution is to use `.env` files and read variables in PHP. `.env` files are just plain text files with variables. They are a little more secure, but they are still files you can accidentally drop into your source code repository.

Locate the config file and open it.

`nano /etc/php-fpm-7.3.d/www.conf`

Find env section, and drop variables directly there:

```env
env[AWS_ACCESS_KEY_ID]=
env[AWS_SECRET_ACCESS_KEY]=
env[CI_ENV]=production
```

don't forget restart `php-fpm` service.

and now you can reach your variables from PHP code directly: 

```PHP
echo getenv('AWS_ACCESS_KEY_ID');
```

