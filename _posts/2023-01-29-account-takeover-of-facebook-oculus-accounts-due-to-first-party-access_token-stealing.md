---
id: 777
title: 'Account takeover of Facebook/Oculus accounts due to First-Party access_token stealing'
date: '2023-01-29T13:04:37+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=777'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

A malicious actor could steal a first-party access token of the Oculus application which he could use to access the Facebook/Oculus accounts.   
  
This was possible because the Oculus application in Facebook, which was used to login to Oculus using Facebook accounts has **[auth.oculus.com/login/](http://auth.oculus.com/login/)** endpoint as a valid redirect_uri. However, Oculus has switched to using Meta Accounts for login. This means that upon visiting **[auth.oculus.com/login/](http://auth.oculus.com/login/)**, the endpoint would redirect to [auth.meta.com/oidc/](http://auth.meta.com/oidc/) for login using Meta Accounts and then come back to the auth.oculus.com.   
  
We can choose in www.facebook.com OAuth the response_type=token and the token would be passed to the next redirect URL until it reaches again auth.oculus.com. The problem here was that before, [auth.oculus.com/login](http://auth.oculus.com/login) was protecting against token leakage through redirects by having the redirects being made using Javascript , however after the oculus login being changed to Meta accounts and not with Facebook , this protection disappeared and now it directly redirects to the URL initially found in [auth.oculus.com/login/?redirect_uri=Redirect_Here](http://auth.oculus.com/login/?redirect_uri=Redirect_Here). Redirect_Here could be any subdomain of oculus.com and some of them like forums.oculus.com which would redirect to a third party application which can have an open redirect to leak the token ( this was not the open redirect used though ). This bug was simple to find and exploit.

**Details**

Setup:

1\. Victim is logged-in to [Facebook.com](https://facebook.com/)  
2\. Victim is not logged-in to [Oculus.com](https://l.facebook.com/l.php?u=http%3A%2F%2FOculus.com%2F%3Ffbclid%3DIwAR0iBRvQMbk-2rLkCcZdJVietJdBU28tY_1v_AP6XdvUN3NkJl89eSDB9nM&h=AT3MKIAPmmotcNSrKVJ8Rpqvw5CIbxOhQooIA9Gj8Hbqp9jCiomjG1qQv1qyzrAtk8xUp5YP0DgSzDj4t0E_0KRqTMy3g-kBKOFI4YpDHPQ0ZRbzOVVyq9RTd1KDjxrX9XmHOx8r2kI) ( this is not necessary since we can use a logout CSRF here )

Attack:  
  
1\) Login CSRF the victim to his Meta account by redirecting to this page [https://auth.meta.com/login/facebook/](https://l.facebook.com/l.php?u=https%3A%2F%2Fauth.meta.com%2Flogin%2Ffacebook%2F%3Ffbclid%3DIwAR03K8axPxMWJXxKSS5BUbUtTHa8CE1rPm-M6nQts2JLbOBUpNP06pEytC4&h=AT0y-zHscHbdw102lWpuzOXh0q-YanPXBnMkoOVPf-vjqsPLIAWFFBAVUijoL7ja6TEZDOFJ6njX79wUsdJ0Oqk_y_wUV3TvfY_AK7fxWFOB_9VfAF45T9kEhKlvBTwo519AtQEKMA)  
  
2\) Open **[https://www.facebook.com/v3.1/dialog/oauth?app_id=1517832211847102&amp;redirect_uri=https://auth.oculus.com/login/?redirect_uri=https://forums.oculus.com/openredirect&amp;response_type=token](https://www.facebook.com/v3.1/dialog/oauth?app_id=1517832211847102&redirect_uri=https://auth.oculus.com/login/?redirect_uri=https://forums.oculus.com/openredirect&response_type=token)**  
  
3\) After the OAuth flow we can notice that the token ended up in https://forums.oculusvr.com/openredirect#access_token=TOKEN

4\) Eventually the access_token would be leaked to <https://ysamm.com>

  
The open redirect here was not fully disclosed since it’s still not fully fixed.

Timeline

Aug 27, 2022— Report Sent   
Sep 25, 2022— Acknowledged by Meta  
Sep 25, 2022— Fixed by Meta  
Sep 25, 2022 — $44250 bounty awarded by Meta. ( Including BountyCon bonuses and bonus for **Highest Impact Report** )