---
id: 603
title: 'URLs in img tag aren&#8217;t passed through safe_image.php which lead to exposure of Facebook users IPs.'
date: '2021-02-15T08:05:12+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=603'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This could allow an attacker to embed an external website in an “img” tag. Normally Facebook doesn’t allow the using of external websites for images and even if it did, the URL would be passed to “safe_image.php” proxy that would fetch the content server-side. In this this specific page they allowed any URL to be passed.   
  
This allowed me to embed an URL of my website which host a script to log all incoming requests. Also it’s possible for some browsers to popup a authentication window if the requested external URL returned a 511 or 401 Status codes ( IE ) which asks for username and password to be passed later to the website. This could be used to phish Facebook users. Other attacks are possible too like embedding malformed or long “data” scheme which freezes the browser or embedding local files.

**Impact**

This could leak private user data like his IP and OS based on his user-agent and could in some cases trick users to enter their username and password. Also this bug creates a gap for the attackers to test newly found browser vulnerabilities and maybe escalate this bug to launch more dangerous attacks from the Facebook platform. However this bug could be exploited only if the victim accepts an invitation to be an admin in the attacker page and then visit a special page or by navigating to a link sent by the attacker.

**Reproduction Steps**

1\) Create a Facebook page which will be used later by the attacker.  
2\) Navigate to [https://www.facebook.com/{PAGE_TOKEN}/inbox/](https://www.facebook.com/%7BPAGE_TOKEN%7D/inbox/) and then “Automated Responses” &gt; “Messenger”  
3\) Under “Response and Feedback”, we enable “Page Recommended” and then we click in “  
Edit Message”  
4\) We Upload a random image and then we click “Save” in the top right corner.  
5\) Intercept the request with Burp Suite for example, the request should be made to <span class="has-inline-color has-dark-yellow-color">/api/graphql/</span> endpoint where doc_id=1713290482059875. URL decode the variables parameter and in the “attachment” element we replace the “url” to the attacker domain and “id” to a whatever number.  
6\) Now if we navigate to [](https://www.facebook.com/{PAGE_TOKEN}/inbox/)[https://www.facebook.com/{PAGE_TOKEN}](https://www.facebook.com/%7BPAGE_TOKEN%7D/inbox/)[/inbox?section=automated_responses](https://www.facebook.com/%7BPAGE_TOKEN%7D/inbox?section=automated_responses) , a GET request to the selected URL should be made.  
7\) Finally we give the victim a role in our page with access to page messages and send him the previous link.

**Timeline**

Feb 13, 2019— Report Sent   
Apr 12, 2019— Acknowledged by Facebook  
Apr 15, 2019— Fixed by Facebook  
Apr 30, 2019 — $500 bounty awarded by Facebook.