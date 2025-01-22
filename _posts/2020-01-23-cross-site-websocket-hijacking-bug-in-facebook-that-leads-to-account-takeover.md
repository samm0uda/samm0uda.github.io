---
id: 363
title: 'Cross-Site Websocket Hijacking bug in  Facebook that leads to account takeover'
date: '2020-01-23T21:45:12+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=363'
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to hijack the Websocket connection of a Facebook user who is using the new Facebook platform (name REDECATED) and then send different websocket messages to takeover the account.   
  
**Explanation**  
  
This is possible because this new web application hosted under facebook.com subdomain allowed local IP addresses ( like 10.0.0.1/8 and 192.168.1.1/8) to be included as an Origin header. This application sued a nonce based login (generated in the login page and sent later in a websocket message) that should be different and correct for each user even after establishing a websocket connection to get a valid session (cookies are ignored in this case).  
However while testing/finding another bug in the authentication system, i noticed a change in the platform ( authentication mechanism exactly) which was using a nonce to login but now started to use cookies to identify the user (of course the cookies are Facebook cookies because this was a subdomain of facebook.com) .

<div class="wp-block-image"><figure class="aligncenter size-large">[![](http://127.0.0.1:4000/assets/img/2020/01/te-1-1024x207.png)](http://127.0.0.1:4000/assets/img/2020/01/te-1.png)Successfully establishing a Websocket connection with a local IP as origin</div>  
This could allowed an attacker in the same local network of the victim to takeover his account by injecting malicious link using DNS spoofing attacks or by simply sending it to the victim ( This is possible locally of course because the Origin header is restricted to local IPs) .   
It was also possible to a malicious app installed in the victim phone to start a HTTP server ( no extra persmissions required) and send a malicious URL with the phone local IP address to the victim Facebook app’s Webview via a deep-link.

**Reproduction steps**

***“This new platform is still in beta/testing phase and only limited number of researchers could access it. For that, the reproduction explanation here is mostly redacted and exploit scripts weren’t supplied.”***

1\) To demonstrate this, i visited REDACTED.facebook.com and saved the index page with the Javascript files handling the websocket communication locally ( The same Javascript files were used in the attack with some simple modification because the Websocket messages were encrypted and the Javascript files were obfuscated and hard to understand). Then i started a HTTP server containing these files.

2\) After sending a link to the victim in the same network, the script would establish a Websocket connection with REDACTED.facebook.com server and valid Facebook cookies are used to identify the user and log him in (This was possible from the local IP of course because Websocket is not bound by SOP and CORS) . The victim should see himself logged-in to his Facebook account while only visiting a local IP. The javascript file contained a payload to send specific websocket messages after logging-in to add an email or a phone number. Same attack could be launched from an Android app hosting the scripts( iOS wasn’t tested)

**Timeline**

Dec 10, 2019 — Report Sent  
Dec 10, 2019 — Acknowledged by Facebook  
Dec 17, 2019 — Fixed by Facebook  
Jan 2, 2020 — 12,500$ Bounty Awarded by Facebook