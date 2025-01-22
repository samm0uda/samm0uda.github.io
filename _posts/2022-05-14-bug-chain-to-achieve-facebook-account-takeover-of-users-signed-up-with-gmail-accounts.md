---
id: 763
title: 'Multiple bugs chained to takeover Facebook Accounts which uses Gmail.'
date: '2022-05-14T15:32:44+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=763'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

  
  
**<mark class="has-inline-color" style="background-color:#88e1db">Description</mark>**

This bug could allow a malicious actor to takeover a Facebook account after stealing a Gmail OAuth id_token/code used to login to Facebook. This happened due to multiple bugs that were chained. Main ones were an intended-by-design XSS in a Facebook sandbox domain and a bug that caused the sharing of sensitive data with this sandbox domain.

**<mark class="has-inline-color" style="background-color:#88e1db">Details</mark>**

The exploitation of the bugs was developed to only target Facebook users who have signed-up using a Gmail account which has an OAuth Flow that Facebook could use to log them in to Facebook using their account. It was possible to target all Facebook users but that was more complicated to develop an exploit for and this was actually enough to demonstrate the impact.

Let’s first take a look on each weakness i found and later how i chained it all together:

**<mark class="has-inline-color" style="background-color: #ffa751;">1. Facebook Checkpoint </mark>**

Facebook has extra security mechanism after login which is responsible of securing the account / verifying the real identity of the user / Ensuring valid MFA. Also, in case your Facebook account was deleted for misuse or temporary blocked you’ll end up in this “Checkpoint”.

In some cases during this Checkpoint process, you’ll need to complete a Captcha to ensure rate limitation and number of tries:

<figure class="wp-block-image size-large">![](http://127.0.0.1:4000/assets/img/2022/05/captcha-1-1024x416.png)https://www.facebook.com/checkpoint/CHECKPOINT_ID/Here, they’ve used Google Captcha and as an extra security feature to avoid adding third-party code from Google into main domain, they just included the Google Captcha in a sandbox domain and establish a communication between it and the main domain after putting it in an Iframe.

Everything is fine until now, the strange part in this implementation that they, for unknown reasons ( maybe logging ), decided to share the current URL at the checkpoint with the sandbox domain as follow:

- If for example the current URL is **https://www.facebook.com/checkpoint/CHECKPOINT_ID/?test=test**,   
    the iframe page being loaded would have this URL **https://www.fbsbx.com/captcha/recaptcha/iframe/?referer=**https%3A%2F%2Fwww.facebook.com%2Fcheckpoint%2FCHECKPOINT_ID%2F%3Ftest%3Dtest

As you can notice the parent window URL was included with parameters.

- Moreover, since we’re stuck in this checkpoint any URL visited in www.facebook.com domain would get us redirected to the checkpoint again BUT with the previous URL included in a **next** parameter :  
      
    1 – ( Visited ) **https://www.facebook.com/eg_oauth_callback_endpoint?code=code**   
      
    2 – ( After Server-Side Redirect ) **https://www.facebook.com/checkpoint/CHECKPOINT_ID/?next=**https%3A%2F%2Fwww.facebook.com%2Feg_oauth_callback_endpoint%3Fcode%3Dcode  
      
    3 – ( Inside Iframe ) **https://www.fbsbx.com/captcha/recaptcha/iframe/?referer=**https://www.facebook.com/checkpoint/CHECKPOINT_ID/?next=https%3A%2F%2Fwww.facebook.com%2Feg_oauth_callback_endpoint%3Fcode%3Dcode

At this point, we can leak any visited endpoint and its parameters in www.facebook.com to www.fbsbx.com. Next step is to find a way to leak it from the sandbox domain to us. This shouldn’t be hard since sandbox domains security is loosen compared to main ones and an XSS was easily found there.

**<mark class="has-inline-color" style="background-color:#ffa751">2. XSS in www.fbsbx.com </mark>**

I already had one in pocket for this. I wouldn’t call it a bug since Facebook actually used the sandbox domain to allow some users to test a certain feature that have HTML files uploading possible. Even after this report, it was kept as it is since like i said in the beginning it’s intended-by-design.

It’s simply going to **https://developers.facebook.com/tools/playable-preview/** while having a developer account and upload any HTML file with your script and eventually you’ll have it uploaded to www.fbsbx.com like this : **https://www.fbsbx.com/developer/tools/playable-preview/preview-asset/?handle_str=STR**

A good thing to learn here and Facebook actually was doing it to some extent is to have a separate sandbox subdomain for each feature. This ensure that third-party code hosted and configured by Facebook but not fully trusted, is not hosted in the same domain as user uploaded code which is mainly to ensure SOP ( of course document.domain shouldn’t be set to the top level domain in the sandbox domains hosting Facebook code )

**<mark class="has-inline-color" style="background-color:#ffa751">3. Logout CSRF &amp; Login CSRF</mark>**

Normally the Facebook account to be targeted in not in the checkpoint state, because of that we can’t leverage the bug above.   
  
In this case, we should log out the current user and log him in to the attacker account which is in the Checkpoint state. BUT if we do that how we’re going to Takeover his Facebook account?   
  
The answer here to actually target a third-party OAuth provider that Facebook uses which is Gmail.  
Gmail sends back the OAuth code/token to www.facebook.com if the user is logged in to Gmail, and since we can steal anything that is coming to www.facebook.com we can use the Google OAuth code to login to the Facebook account that has that Gmail account linked to it ( meaning any account that has a gmail account )

**<mark class="has-inline-color" style="background-color:#ffa751">3. SOP &amp; COOP</mark>**

To steal the URL passed to the iframe inside www.facebook.com, we first should establish a relation between the two windows:  
The window (1) that is having the page with URL **https://www.facebook.com/checkpoint/?next=Pevious_Page_With_Gmail_Code** and the window (2) that have the page in **https://www.fbsbx.com/developer/tools/playable-preview/preview-asset/?handle_str=** with XSS and attack script.  
  
This way, from window (2) we can for example access **opener.frames\[0\]**.location.href :   
– **Opener** here is window (1).   
– **frames\[0\]** is the iframe inside which has **https://www.fbsbx.com/captcha/recaptcha/iframe/?referer=**https://www.facebook.com/checkpoint/?next=Pevious_Page_With_Gmail_Code  
  
Since frames\[0\] and window (2) are same-origin we have access to read the page **location.href** which contains the stolen code.  
  
The only protection that could have prevented this was COOP ( Cross-Origin-Opener-Policy ) which was being heavily applied in Facebook in previous months but likely was not in the callback&amp;checkpoint endpoints.

**<mark class="has-inline-color" style="background-color:#88e1db">Attack Steps</mark>**

The full attack below would be delivered via an uploaded script to www.fbsbx.com by exploiting the XSS explained in step 2) and by having a code that does the following steps :

1\) Logout CSRF: **REDACTED**  
  
2\) Login CSRF: ****REDACTED****

3\) Open **https://accounts.google.com/o/oauth2/auth?redirect_uri=https://www.facebook.com/oauth2/redirect/&amp;response_type=permission%20code%20id_token&amp;scope=email%20profile%20openid&amp;client_id=15057814354-80cg059cn49j6kmhhkjam4b00on1gb2n.apps.googleusercontent.com** using window.open (Opened window noted **TARGET**) . It would redirect to www.facebook.com/checkpoint/CHECKPOINT_ID/?next=URL_WITH_CODE where URL_WITH_CODE is www.facebook.com/oauth2/redirect/?code=Gmail_Code

4\) Wait for few seconds and access TARGET.frames\[0\].location.href. We should have the Google OAuth code and id_token.

In the attacker side, we can extract the email address from id_token, then we can start a password recovery process in www.facebook.com ( to set the right sfui cookie ) and later choose to connect to Gmail ( openid ) and we would be redirected to a similar URL like Step 3 but with an extra state parameter.

We use the state from the previous step ( state in the redirect to accounts.google.com ) and we put it here to construct this final URL   
**https://www.facebook.com/oauth2/redirect/?state=STATE&amp;code=CODE_FROM_STEP_4.**   
If we visit this URL, we should find in the response body a link to access the victim account ( contains a nonce in the /recover/password/ URL )

**<mark class="has-inline-color" style="background-color:#88e1db">Timeline</mark>**

Feb 16, 2022— Report Sent   
Feb 22, 2022— Acknowledged by Facebook  
Mar 21, 2022— Fixed by Facebook  
May 14, 2022 — $44625 bounty awarded by Facebook. (7000 Diamond League Bonus, 2625 Bounty Delay Bonus)