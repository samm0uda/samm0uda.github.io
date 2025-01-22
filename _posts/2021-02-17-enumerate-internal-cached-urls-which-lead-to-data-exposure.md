---
id: 629
title: 'Enumerate internal cached URLs which lead to data exposure'
date: '2021-02-17T10:19:43+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=629'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to enumerate cached URLS in Facebook servers. This vulnerable endpoint would accept any URL given and check if it’s cached or not so the attacker could determine if a certain endpoint or path exists Facebook domains including internal ones ( [phabricator.intern.facebook.com](http://phabricator.intern.facebook.com/) , [our.intern.facebook.com](http://our.intern.facebook.com/) , sandcastle domains …). What make this bug more dangerous is that the the URL supplied could be unfinished which allow us to bruteforce each path character by character ( eg: we can check if https://our.intern.facebook.com/intern/arg is valid and continue by adding another character like “u” which is valid until we reach <https://our.intern.facebook.com/intern/argus/> ) This method works also with ids which could lead to disclosure of internal objects ids.

**Impact**

The impact here is serious since the attacker could use this bug to get a huge number of information. The information includes:  
– Internal endpoints ( by checking <https://our.intern.facebook.com/intern/> and then adding a character each time)  
– Internal files ( by doing the same thing to <https://phabricator.intern.facebook.com/diffusion/> eg (<https://phabricator.intern.facebook.com/diffusion/E/browse/tfb/trunk/www/flib/> … )) . This could also lead to enumerating Controllers by looking for corresponding files (\*Controller.php)  
– Enumerate JS files by doing the attack against <https://s-static.intern.facebook.com/> or <https://s-static.internalfb.com/>  
– Finding an endpoint which was leaking access_tokens/password in GET requests could lead to account takeover  
– Other possible scenarios that could disclose information about users or employees in URLs.

**Reproduction Steps**

Send a POST request to [https://developers.facebook.com/tools/debug/sharing/batch_submit/?elem_id=u_91_0](https://developers.facebook.com/tools/debug/sharing/batch_submit/?elem_id=u_91_0) with valid cookies and those parameters in the request body:

__a=1&amp;  
fb_dtsg=CSRF_TOKEN&amp;  
q=URL

where URL is the desired url to enumerate. You can put multiple URLs here separated by a new line (%0A). After sending the request you should look in the response for “No Cached Information” which means that the character selected in the bruteforcing attack is wrong. If the character is valid then you should get “No Issues”. You can reproduce this without building the request using <https://developers.facebook.com/tools/debug/sharing/batch/> however you would be limited to manual trying.  
  
Also i listed above examples of Facebook domains however please notice that other domains could be targeted. For example, if a link is shared privately in a Facebook conversation between two users, it would automatically be scraped and cached which allow an attacker in this scenario to enumerate a certain domain to leak sensitive data.

**Examples:**  
  
https://our.intern.facebook.com/intern/si/bouncer/explore/?src_type=instagram_user&amp;src_name=5884754650  
https://our.intern.facebook.com/intern/scuba/query/\*  
https://our.intern.facebook.com/intern/profile/\*  
https://our.intern.facebook.com/intern/wiki/\*  
https://www.facebook.com/webgraphql/mutation/?doc_id\*  
https://www.facebook.com/webgraphql/query/?doc_id\*  
https://graph.facebook.com/\* ( or upload to target potential cached apps/user access_tokens)

**Timeline**

Oct 11, 2020— Report Sent   
Oct 12, 2020— Acknowledged by Facebook  
Dec 11, 2020 — $4800 bounty awarded by Facebook. (including bonus)  
Feb 8, 2021— Fixed by Facebook