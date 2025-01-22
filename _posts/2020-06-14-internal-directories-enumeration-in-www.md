---
id: 458
title: 'Internal directories enumeration in www'
date: '2020-06-14T19:41:33+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=458'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to enumerate internal “www” or “flib” directories by finding sub-directories and “potentially” files names.

**Reproduction**

Bruteforcing **X** in https://www.facebook.com/**X** with a wordlist of files or directories names would reveal sub-directories of www.

You can distinguish between the existence of a directory or not by comparing the status code returned. If it’s a “301 redirect”, then the directory exist in www.   
  
For example, if this URL “https://www.facebook.com/flib/third-party/3PC” returned 301 status code and redirected us to “https://www.facebook.com/flib/third-party/3PC***/***” ( with a slash ) , it means /var/www/flib/third-party/3PC/ directory exist.

Example found directories by exploiting the bug:

www.facebook.com/flib/core/smc/   
www.facebook.com/flib/third-party/tfpdf/  
www.facebook.com/conf/build/

**Impact**

It is possible to infer the existence of arbitrary directories in www which would allow an attacker to find third-party libraries used by Facebook or many other information that could be useful for other attacks or bugs.

**Timeline**

Mar 14, 2020 — Report Sent  
Mar 16, 2020 — Acknowledged by Facebook  
Apr 16, 2020 — Bounty awarded by Facebook  
May 1, 2020 — Fixed by Facebook