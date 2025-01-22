---
id: 510
title: 'Bad regex used in Facebook Javascript SDK leads to account takeovers in websites that included it'
date: '2020-12-31T18:11:04+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=510'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

This bug could allow an attacker to target websites that included Facebook Javascript SDK. The attacker would trick a certain website user to visit his website which would exploit a bug in the implementation used in the SDK to allow cross-origin communication between the website that included the SDK, and Facebook main website. The Facebook JS SDK is widely used by developers for various reasons like logging-in to the website using Facebook or to embed Share or Like plugins which left thousands of websites vulnerable to various attacks like account takeovers.

**Technical Details**  
  
The Facebook JS SDK could be implemented in a website for various reasons like to Login with Facebook, Sharing, Embedding. Also the same SDK could is used to communicate with Facebook website to authenticate and serve content for Canvas/Tab applications that would be accessed via apps.facebook.com or page tabs. This is done by including the SDK in a page in your website which would have the content for the canvas application, then you would add the link of the page in your Facebook canvas application settings. If you visit apps.facebook.com/APP_ID/, you should find that the page selected would be served in an iframe and Facebook website would communicate with your website and vice versa by using postMessage method and EventListeners for cross-origin window messages.   
The main key for secure cross-origin communication is to always verify the origin and destination of a certain message. Facebook failed to do that in the SDK which is included in a third party website because it was using a wrong regular expression to verify the origin of a certain cross-origin message received :  
  
This is a snippet of the code responsible for cross-origin communication in the SDK:

```javascript
__d("sdk.XD", ["JSSDKXDConfig", "Log", "QueryString", "Queue", "UrlMap", "guid", "isFacebookURI", "resolveWindow", "sdk.Event", "sdk.feature", "sdk.RPC", "sdk.Runtime", "sdk.Scribe", "sdk.URI"], (function(a, b, c, d, e, f) {
                     var g = new(b("Queue"))(),
                         h = "parent",
                         i = null,
                           j = /^https:\/\/.*facebook.com$/;
...

window.addEventListener("message", function(a) {
                             var c = a.data,
                                 d = a.origin || "native";
                             if (!/^(https?:\/\/|native$)/.test(d)) {
                                 b("Log").debug("Received message from invalid origin type: %s", d);
                                 return
                             }
                             if (!j.test(d)) return;
                             if (typeof c === "string") r(c, d);
                             else {
                                 if (a.source == parent && a.data.xdArbiterRegisterAck && j.test(d)) {
                                     typeof a.data.xdArbiterRegisterAck === "string" && a.data.xdArbiterRegisterAck !== "" && p(a.data.xdArbiterRegisterAck);
                                     g.isStarted() || g.start(function(a) {
                                         if (a == null) {
                                             b("Log").warn("Discarding null message from %s to %s", m, d);
                                             return
                                         }
                                         var c = parent;
                                         typeof a === "object" && typeof a.relation === "string" && (c = b("resolveWindow")(a.relation));
                                         ((c = c) != null ? c : parent).postMessage({
                                             xdArbiterHandleMessage: !0,
                                             message: a,
                                             origin: m
                                         }, d)
                                     });
                                     return
                                 }
                                 b("Log").warn("Received message of type %s from %s, expected a string. %s", typeof c, m, ES("JSON", "stringify", !1, c));
                                 return
                             }
                         });
```

If we examine the code, we find that there is one Eventlistener for cross-origin messages inside the SDK. Once a message is received, it would check if the origin starts with https (or native) , and if it checks against the regex “**/^https:\\/\\/.\*facebook\\.com$/**”   
As you may notice the regex was meant to target facebook.com and its subdomains however an escaped dot is missing before facebook.com ( correct one would be **/^https:\\/\\/.\*\\.facebook\\.com$/** ).   
This allows a domain like testpocfacebook.com to pass the origin check done here.  
The message sent by testpocfacebook.com would be **{“xdArbiterRegisterAck”:”canvas”}** and the SDK would return an FB_RPC json formatted message to the parent window. The message would include “fallback_redirect_uri” which is the value of window.location.href.   
Here comes the exploitation of this bug. Like explained above, the SDK includes code for all possible functionalities, a website could use the SDK to only use the sharing functionality however the code for the canvas functionality would still be included which means that the website would have an EventListener for cross origin messages which would now accept messages from the attacker owned website by exploiting the bug . As you may noticed in the code above, the source of the message is checked and it should be the parent window which means that for this to work, the targeted page must also allow itself to be iframed.   
To conclude, If we can find a page in a website which includes the Facebook Javascript SDK and can be iframed, we can leak the window.location.href of that page.   
  
**How to abuse this** **?**

This could be used to leak information after redirects in the targeted website. For example, if we iframe **https://example.com/me** page which would redirect to **https://example.com/profile/PROFILE_ID**, the /profile/ page can be iframed, we send the message after the redirect and we would receive the logged-in user profile id.   
  
Same attack could apply to oauth or other authorization mechanisms’ callback endpoints which includes the Facebook SDK. Since many websites includes the Facebook JS SDK in every page ( same way the google analytics tag is included), this would enables account takeovers or the stealing of sensitive tokens. This would work the same way as the previous example, we assume that the website uses Google or Facebook for authentication, we would iframe https://accounts.google.com/o/oauth2/auth or https://www.facebook.com/dialog/oauth with parameters that includes the targeted website Google or Facebook application details (redirect_uri and app_id), if the victim is already logged in to his/her Facebook or Google accounts, these urls would redirect to the callback endpoints in the targeted website, if the callback endpoints could be iframed, then we would send the cross-origin message and we would receive the access_token or code ( since we’re receiving current window.location.href inside the iframe)

This attack could easily be launched against websites that hosts canvas apps or page tabs. The page that is used in canvas app can be iframed to allow itself to be included in an iframe inside apps.facebook.com, and also the same page ,most of the time, would be used as a Facebook oauth callback endpoint.   
We would embed an iframe inside https://testpocfacebook.com/ with the following src ‘**https://www.facebook.com/dialog/oauth?client_id=APP&amp;redirect_uri=REDIRECT_URI&amp;response_type=token**‘. If the user authorised this app before, this would directly redirect to REDIRECT_URI which would have the Facebook SDK script embedded. At that point, the parent window would sent the cross-origin message and receive the access_token embedded inside fallback_redirect_uri.   
  
The attacker website could have 100 iframes with each iframe pointing to different website hosting a canvas application which would increase the chance of having a visitor that used one of these applications before and we would finally receive an access_token with permissions selected in the application scope.

**Quick Demonstration**

This PoC was used in the report sent to Facebook (Since i wasn’t able to include PoCs for third-party websites) . Although this didn’t work since www.instagram.com callback endpoint didn’t allow itself to be iframed (X-Frame-Options: deny header was set), however the endpoint included the Facebook JS SDK. If a bypass was found for the iframing problem, this could have lead to a full Facebook account takeover since the Instagram.com Facebook application is a first-party application and generated access_tokens would have access to https://graph.facebook.com/graphql endpoint.

```javascript
<html>
<head>
window.addEventListener("message", (event) => {console.log(event.data); }, false);
</head>
<body>
<script>function fun() {
var ifr = document.getElementById('ifr');
ifr.contentWindow.postMessage({"xdArbiterRegisterAck":"canvas"}, '*');
}</script>
<iframe src="https://www.facebook.com/dialog/oauth?client_id=124024574287414&redirect_uri=https://www.instagram.com/accounts/signup/&scope=email&response_type=token" onload="fun(this)"/>
</body>
</html>
```

  
If a logged-in Facebook user visits the attacker website, https://www.facebook.com/dialog/oauth would be requested in the iframe, a redirect would be made to https://www.instagram.com/accounts/signup/ with an access_token in the hash fragment part of the URL. The endpoint /accounts/signup/ includes the Facebook SDK ( although it can’t be iframed , for the sake of the example we’ll assume it can). A cross-origin message would be send to the iframe window and we would receive a message with the Facebook user access_token included. We may then takeover the account :’).

Many famous websites that used the Facebook Javascript SDK were vulnerable to account takeovers of their users using the same attack logic. I was happy that i found this bug and reported it in time before it was misused.

**Timeline**

Nov 20, 2020— Report Sent   
Nov 27, 2020— Acknowledged by Facebook  
Dec 4, 2020— Fixed by Facebook  
Dec 31, 2020 — $10K bounty awarded by Facebook.

</body></html>