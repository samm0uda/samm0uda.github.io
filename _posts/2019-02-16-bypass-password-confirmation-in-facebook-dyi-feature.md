---
id: 240
title: 'Bypass password confirmation in Facebook &#8220;DYI&#8221; feature'
date: '2019-02-16T16:58:30+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=240'
categories:
    - Uncategorized
---

This could allowed an attacker who has access to a user’s web session to use DYI to download all of the user’s information, without re-entering the user’s password.  
  
**Description**  
  
If the attacker was able to steal the victim cookies and access his Facebook account by exploiting another bug or maybe he had a temporary access, he’s normally unable to download the information of the victim account through the “DYI” feature because of the required password confirmation but using this bug he can do it.   
Some would say that this could easily be bypassed by adding a new phone/email to the account and then resetting the password but this action could trigger Facebook detection systems or may alert the owner about this change and expose the attack.

**Demonstration**  
  
After generating files in the DYI dashboard located at https://www.facebook.com/dyi/, we try to download the file generated but then a password confirmation is required to complete the action .  
In the request to the https://www.facebook.com/dyi/download2/ endpoint which is responsible of returning a valid download URL for the generated file, we get the parameter **“id”** which represent the generated file id. Then instead of using this endpoint we try the same request to this one below which is responsible of downloading user albums but without asking for password confirmation.

```
https://www.facebook.com/photos/album/download/file/
?album_id=1
&dyi_job_id={JOB_ID}
&notif_t=photo_album_download
```

This request should redirect the user to a valid URL to download the file containing all victim information. The attacker then could remove the information request from the “DYI” dashboard and the victim will never know that his information has been stolen. (No password was changed)

  
**Timeline**  
Feb 4, 2019 — Report Sent  
Feb 8, 2019— Acknowledged by Facebook  
Feb 12, 2019— Fixed by Facebook  
Feb 15, 2019 —Bounty Awarded by Facebook