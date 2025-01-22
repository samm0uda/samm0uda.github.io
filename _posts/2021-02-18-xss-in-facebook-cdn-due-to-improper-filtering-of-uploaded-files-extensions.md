---
id: 632
title: 'XSS in Facebook CDN due to improper filtering of uploaded files extensions'
date: '2021-02-18T08:15:09+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=632'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This could allow an attacker to upload html files to Facebook CDNs. This happens because the uploading endpoint only checks for the Mime type supplied in “Content-Type” but not the file extension. This would result to a HTML file being uploaded to scontent.\*.fbcdn.net CDNs.

**Reproduction Steps**

1\) Login to [https://www.facebookrecruiting.com/](https://l.facebook.com/l.php?u=https%3A%2F%2Fwww.facebookrecruiting.com%2F%3Ffbclid%3DIwAR2xYdF5PDG8I1ZSH9aTqFV_AEGU669Df7SXoEiRLt0a8p5Uf37ByDU4kVs&h=AT0mUe8lc9e6dTh4ggllNZlvDXcNmgPsXnCBo04YpU96ilHs3yJN787AQS60jh9BCloepswPzKFX4bv7032h3TwJKbhLZqLn89aSqFlisPMNcUNUFVPaUHKxUwCKaQ)

2\) Upload a html file named as file.pdf

3\) Intercept the request and change the name from **file.pdf** to **file.html** and keep the content-type as application/pdf

4\) You should get a URL to the uplaoded file in the request response

This is an example file: **https://scontent.ftun3-1.fna.fbcdn.net/v/t39.29810-6/76190145_2675021082561187_2889406825675882491_n.jpg.html/h1.html?_nc_cat=101&amp;_nc_oc=X&amp;_nc_ht=scontent.ftun3-1.fna&amp;oh=74e85a9cc7e9452c3e9bd3979028ce31&amp;oe=5E541E53**

**Impact**

The impact of this bug is limited since the XSS would be in the CDN domains however this could be used when exploiting other bugs like capturing a token since fbcdn.net subdomains are whitelisted in Linkshim.

**Timeline**

Nov 4, 2019— Report Sent   
Nov 5, 2019— Acknowledged by Facebook  
Nov 6, 2019— Fixed by Facebook  
Nov 13, 2020 — $500 bounty awarded by Facebook.