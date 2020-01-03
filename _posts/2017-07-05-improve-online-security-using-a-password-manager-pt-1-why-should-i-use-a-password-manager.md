---
layout: post
title: "Improve online security using a password manager, pt. 1: Why should I use a password manager?"
tags: [cryptography, encryption, online security, password manager, security, strong password]
categories: [ Password managers, Security ]
redirect_from:
  - /blog/2017/7/4/improve-online-security-using-a-password-manager-pt-1-why-should-i-use-a-password-manager
---

So, you want to keep your online security at a high level? Do you want to  avoid the stress of changing multiple passwords when online services  gets hacked and compromised? Do you want to make remembering passwords a breeze? Why even remember multiple passwords when you can settle with  remembering just one? What you need is a password manager.

What is a password manager, you ask? A password manager is a tool to generate  long, unique, secure and random passwords for your needs and at the same time store them in a secure manner. For each service you access, you  simply create a new, long, unique, secure and random password. If the  service is ever breached and people with malicious intent manage to  recover your password either due to poorly implemented security measures or due to brute-forcing of your securely stored password, the password  will have little to no reuse value for them. Your accounts on other  services will still be safe and secure. At the same time, you will never have to worry about pages implementing password storage in an adequate  way, as you will never hand-over your personal password.

## Ignore risks, use a password manager

But, hey! Don’t all online services implement their password storage  correctly? No, no they don’t. Even large companies fail to secure  themselves properly. Just to bring forward a couple of examples, such as Adobe [(read more)](https://nakedsecurity.sophos.com/2013/11/04/anatomy-of-a-password-disaster-adobes-giant-sized-cryptographic-blunder/) and Sony [(read more)](http://gizmodo.com/sony-pictures-hack-keeps-getting-worse-thousands-of-pa-1666761704). What did Adobe do wrong then? The passwords were not [hashed](https://en.wikipedia.org/wiki/Cryptographic_hash_function) and [salted](https://en.wikipedia.org/wiki/Salt_(cryptography)) adequately, as they should have been. This means that two people with the same  password would have the same encrypted password value stored in Adobe’s  database. Combined with the many password hints, it was possible to  analyze the data and guess the passwords in a fairly easy manner, much  like a giant crossword puzzle. Point being, you need to take your  security into your own hands. One thing is Adobe resetting all users  password, but what if you used the same username and password for your  Facebook account and more, now those are compromised too.

Reusing the password on several services may result in your personal accounts  being compromised on otherwise unbreached services. This is something  Vodafone [(read more)](http://www.esecurityplanet.com/network-security/reused-passwords-expose-1827-vodafone-accounts.html) felt the consequence of in October 2015. Vodafone learned that 1827 accounts had been accessed by unauthorized entities using credentials acquired  from unknown external sources. This is why we need to ensure the  uniqueness of our passwords.

## Don’t memorize passwords using systems

Can’t I just make a system to remember passwords? Sure, a strong system can  be great for remembering semi-unique passwords for different purposes.  There may be several issues though. Firstly, the passwords will likely  never be completely unique and random if you have a pattern for  remembering. And, what if a service is compromised? Will you have to  change your system or append to the rules? If a system can create  several different passwords for the same service, which password is  currently in use? Then comes the places where you have to change a  password periodically, can your system handle that? Where did you get to in your sequence? Oh, and then there’s the different rules for required letters, cases, numbers, symbols and lengths. Your Microsoft account is limited to 16 characters, PayPal limits you to 20 characters and that’s just the upper limits, you will meet lower limits too. Soon you will  need a Lord of the Rings trilogy-sized collection of books to specify  all the rules for your system. Why even bother?

Again, just use a password manager! It’s that simple!